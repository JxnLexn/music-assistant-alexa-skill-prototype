# Music Assistant Alexa Skill Add-on

This add-on hosts the Alexa custom skill endpoint for Music Assistant inside Home Assistant.

## Quick Start

1. Add the repository `https://github.com/JxnLexn/music-assistant-alexa-skill-prototype` as a custom add-on repository.
2. Install `Music Assistant Alexa Skill`.
3. Configure at least:

```yaml
SKILL_HOSTNAME: muas-api.example.com
MA_HOSTNAME: muas-stream.example.com
APP_USERNAME: alexa_ui
APP_PASSWORD: change-me
LOCALE: de-DE
```

4. Start the add-on.
5. Open the Web UI and complete `/setup`.
6. Push current Music Assistant playback metadata to:
   `POST https://<SKILL_HOSTNAME>/ma/push-url`

## Important Notes

- `SKILL_HOSTNAME` must be publicly reachable by Amazon over HTTPS.
- `MA_HOSTNAME` must publicly serve the rewritten Music Assistant stream and artwork URLs.
- The add-on stores ASK CLI credentials persistently under `/data/.ask`.
- `/status`, `/simulator`, and `/invocations` are available for debugging.

For the full installation and troubleshooting guide, see the repository root [`README.md`](../../README.md).
