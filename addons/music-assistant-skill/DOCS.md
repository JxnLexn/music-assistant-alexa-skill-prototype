# Music Assistant Alexa Skill

This add-on hosts an Alexa custom skill endpoint for Music Assistant and provides the setup flow to create or update the skill in your Amazon developer account.

## Setup

1. Install the add-on from this repository.
2. Configure at least:

```yaml
SKILL_HOSTNAME: muas-api.example.com
MA_HOSTNAME: muas-stream.example.com
APP_USERNAME: alexa_ui
APP_PASSWORD: change-me
LOCALE: de-DE
```

3. Start the add-on.
4. Open the Web UI and complete `/setup`.
5. During first setup, open the ASK CLI authorization URL, sign in, paste the code back, and answer `No` when ASK asks about AWS hosting.
6. Push current Music Assistant playback metadata to:
   `POST https://<SKILL_HOSTNAME>/ma/push-url`

## Required Public Hosts

- `SKILL_HOSTNAME`: public HTTPS hostname that points to this add-on
- `MA_HOSTNAME`: public HTTPS hostname that points to Music Assistant stream and image endpoints

If you use Cloudflare:

- no Access / challenge / bot protection in front of `SKILL_HOSTNAME`
- minimum TLS `1.2`

## Options

| Option | Required | Default | Description |
| --- | :---: | :---: | --- |
| `SKILL_HOSTNAME` | Yes | — | Public Alexa skill endpoint hostname. |
| `MA_HOSTNAME` | Yes in practice | — | Public Music Assistant stream / artwork hostname. |
| `APP_USERNAME` | No | — | Basic-auth username for the UI and helper APIs. |
| `APP_PASSWORD` | No | — | Basic-auth password for the UI and helper APIs. |
| `LOCALE` | No | `de-DE` | Locale used for manifest and interaction models. |
| `AWS_DEFAULT_REGION` | No | `us-east-1` | ASK CLI / AWS tooling region. |
| `TZ` | No | `America/Chicago` | Container timezone. |
| `SKIP_URL_VALIDATION` | No | `false` | Skip public stream URL validation. |

## Verify

Open `/status` and check:

- skill exists, endpoint matches, testing enabled
- `Music Assistant API reachable (200)` after `/ma/push-url` receives metadata
- `Alexa API reachable (200)` after a successful LaunchRequest

Useful pages:

- `/setup`
- `/status`
- `/simulator`
- `/invocations`

For the full walkthrough, see the repository root `README.md`.
