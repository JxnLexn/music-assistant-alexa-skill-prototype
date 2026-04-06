# Music Assistant Alexa Skill Add-on Repository

This repository is a Home Assistant custom add-on repository for hosting an Alexa custom skill endpoint for Music Assistant.

The add-on does three things:

- hosts the public Alexa skill endpoint on `POST /`
- provides a setup UI that creates or updates the Alexa skill through ASK CLI
- accepts current Music Assistant playback metadata on `/ma/push-url`

What it does not do automatically:

- discover your Music Assistant instance on its own
- create the Music Assistant -> `/ma/push-url` bridge for you
- host the audio stream itself

You must provide a public HTTPS hostname for the Alexa endpoint and a public HTTPS hostname that exposes your Music Assistant stream and image URLs.

## What You Need

- Home Assistant with Supervisor / add-on support
- a Music Assistant instance that is already working locally
- an Amazon developer account
- Skill Access Management enabled in the Amazon developer account:
  [https://developer.amazon.com/alexa/console/ask/settings/access-management](https://developer.amazon.com/alexa/console/ask/settings/access-management)
- two public HTTPS hostnames:
  - `SKILL_HOSTNAME`: points to this add-on on port `5000`
  - `MA_HOSTNAME`: points to the Music Assistant stream / image endpoints

If you use Cloudflare or another proxy in front of the public hostnames:

- do not put Cloudflare Access, Managed Challenge, Bot Fight, or similar login/challenge protection in front of `SKILL_HOSTNAME`
- keep minimum TLS at `1.2`
- make sure Amazon can reach `https://<SKILL_HOSTNAME>/` directly

## Required Public Routing

The working request flow looks like this:

1. Amazon calls `https://<SKILL_HOSTNAME>/`
2. Your add-on handles the Alexa request and asks its local `/ma/latest-url` cache for the current track
3. The add-on rewrites internal Music Assistant URLs to `https://<MA_HOSTNAME>/...`
4. Alexa plays the public MP3 stream from `MA_HOSTNAME`

In other words:

- `SKILL_HOSTNAME` is the Alexa skill API host
- `MA_HOSTNAME` is the public stream / artwork host

In practice, `MA_HOSTNAME` must expose at least the same paths Music Assistant generates for:

- stream URLs from port `8097`
- image proxy URLs from port `8095`

## Install In Home Assistant

1. Open `Settings -> Add-ons -> Add-on Store -> Repositories`.
2. Add this repository:
   `https://github.com/JxnLexn/music-assistant-alexa-skill-prototype`
3. Install `Music Assistant Alexa Skill`.
4. Open the add-on configuration and set at least:

```yaml
SKILL_HOSTNAME: muas-api.example.com
MA_HOSTNAME: muas-stream.example.com
APP_USERNAME: alexa_ui
APP_PASSWORD: change-me
LOCALE: de-DE
AWS_DEFAULT_REGION: us-east-1
TZ: Europe/Berlin
SKIP_URL_VALIDATION: false
```

5. Start the add-on.
6. Open the Web UI. It opens `/setup`.
7. Complete the ASK CLI authentication flow.

During the first setup run, the add-on prints a no-browser authorization URL.

Do this exactly once:

1. Open the URL in a browser.
2. Log in with your Amazon developer account.
3. Copy the authorization code back into the setup flow.
4. When ASK CLI asks about linking an AWS account for hosting, answer `No`.

Important:

- do not create an Alexa-hosted Node.js skill for this project
- the add-on provisions a custom skill endpoint that points to your own `SKILL_HOSTNAME`

The add-on stores ASK CLI credentials persistently under `/data/.ask`, so you normally do not need to authenticate again after a restart.

## Add-on Options

| Option | Required | Default | Description |
| --- | :---: | :---: | --- |
| `SKILL_HOSTNAME` | Yes | — | Public HTTPS hostname used in the Alexa skill manifest and endpoint validation. |
| `MA_HOSTNAME` | Yes in practice | — | Public HTTPS hostname used to rewrite Music Assistant stream and artwork URLs. |
| `APP_USERNAME` | No | — | Basic-auth username for the Web UI and API helper endpoints. |
| `APP_PASSWORD` | No | — | Basic-auth password for the Web UI and API helper endpoints. |
| `LOCALE` | No | `de-DE` | Alexa locale used for manifest and interaction model upload. |
| `AWS_DEFAULT_REGION` | No | `us-east-1` | Region used by ASK CLI / AWS tooling. It does not control playback latency. |
| `TZ` | No | `America/Chicago` | Timezone for container logs and timestamps. |
| `SKIP_URL_VALIDATION` | No | `false` | Skip runtime validation of the rewritten public stream URL. |

Notes:

- `APP_USERNAME` / `APP_PASSWORD` protect the UI and helper APIs, but Alexa can still reach `POST /` without Basic Auth.
- Leave `AWS_DEFAULT_REGION` at `us-east-1` unless you have a specific reason to change it.
- Only set `SKIP_URL_VALIDATION` to `true` if the public stream URL is correct but validation fails from inside the container.

## Connect Music Assistant

This is the most important integration step.

The add-on does not poll Music Assistant by itself. It expects the currently playing track to be pushed into:

- `POST https://<SKILL_HOSTNAME>/ma/push-url`

Expected JSON payload:

```json
{
  "streamUrl": "http://192.168.1.10:8097/single/abc/EchoDot/xyz/EchoDot.flac",
  "title": "Crazy",
  "artist": "Aerosmith",
  "album": "Best Of",
  "imageUrl": "http://192.168.1.10:8095/imageproxy?...etc..."
}
```

The add-on rewrites those internal Music Assistant URLs to the public `MA_HOSTNAME` at runtime.

Quick manual test:

```bash
curl -u alexa_ui:change-me \
  -H 'Content-Type: application/json' \
  -d '{
    "streamUrl":"http://192.168.1.10:8097/single/test/EchoDot/test/EchoDot.flac",
    "title":"Test Track",
    "artist":"Test Artist",
    "album":"Test Album",
    "imageUrl":"http://192.168.1.10:8095/imageproxy?example=true"
  }' \
  https://muas-api.example.com/ma/push-url
```

If this works, `/status` will show `Music Assistant API reachable (200)` with the payload.

## Verify The Setup

After setup, open `/status`.

What you want to see:

- `Music Assistant Skill interaction model found; endpoint matches (...); testing enabled`
- `Music Assistant API reachable (200)` after your Music Assistant bridge has posted track metadata
- `Alexa API reachable (200)` after a successful LaunchRequest / playback request

Useful built-in pages:

- `/setup`: create or update the skill
- `/status`: health and setup checks
- `/simulator`: local request simulator
- `/invocations`: captured incoming Alexa requests and responses

## Test The Skill

1. Start playback in Music Assistant.
2. Confirm `/status` shows current track data under `Music Assistant API`.
3. In the Alexa Developer Console `Test` tab, or on a real device, say:
   `Alexa, öffne Music Assistant`
4. If playback starts, refresh `/status` and check `Alexa API`.

If you test with the built-in simulator:

- use `/simulator`
- do not send a plain manual `POST /` without the simulator headers

Otherwise the request will fail signature verification and you will see log messages like `Missing Signature/Certificate for the skill request`.

## Troubleshooting

### Manifest Build Fails During `/setup`

If setup stops during the manifest phase:

- read the setup log in the Web UI
- the add-on now logs the manifest status dump directly when Amazon rejects it
- common causes are invalid icon URLs or malformed manifest content

### `Music Assistant API` Stays Empty

If `/status` shows that no stream metadata has been pushed yet:

- your Music Assistant -> `/ma/push-url` bridge is missing or broken
- the add-on itself is not the source of current track data

### `Alexa API` Stays Empty

If `/status` shows no successful playback request yet:

- no successful Alexa LaunchRequest / play request has completed
- check the Developer Console test tab or `/invocations`

### Alexa Says There Was A Communication Problem

Check these in order:

1. `/status` shows skill, endpoint, and testing all green
2. `Music Assistant API reachable (200)` already contains valid data
3. `MA_HOSTNAME` publicly serves the rewritten MP3 and image URLs
4. no Cloudflare Access / challenge is in front of `SKILL_HOSTNAME`
5. you are on a build at least as new as the current repository version

### Cloudflare / Reverse Proxy Notes

- `GET /` redirects to `/status`
- Alexa uses `POST /`
- do not require human login, challenge, or captcha on `POST /`
- keep `SKILL_HOSTNAME` and `MA_HOSTNAME` externally reachable over HTTPS

## Repository Layout

- [`repository.yaml`](repository.yaml): Home Assistant custom repository metadata
- [`addons/music-assistant-skill/`](addons/music-assistant-skill): add-on source, runtime code, and assets
- [`scripts/`](scripts): repository maintenance scripts

## Additional Notes

- The add-on image is built from [`addons/music-assistant-skill/Dockerfile`](addons/music-assistant-skill/Dockerfile).
- GitHub Actions publishes `ghcr.io/jxnlexn/music-assistant-skill`.
- Versioning is driven by [`VERSION`](VERSION) and synced into the add-on config.

See [`COMPATIBILITY.md`](COMPATIBILITY.md), [`LIMITATIONS.md`](LIMITATIONS.md), and [`DISCLAIMER.md`](DISCLAIMER.md) for further details.
