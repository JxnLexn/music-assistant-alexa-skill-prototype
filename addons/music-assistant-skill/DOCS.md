# Music Assistant Alexa Skill

This add-on provisions and hosts the Music Assistant Alexa skill from inside Home Assistant.

## Setup

1. Install the add-on from this custom repository.
2. Configure at least `SKILL_HOSTNAME`.
3. Start the add-on.
4. Open the Web UI and complete `/setup`.

ASK CLI credentials are stored persistently under `/data/.ask`.

## Options

| Option | Required | Default | Description |
| --- | :---: | :---: | --- |
| `SKILL_HOSTNAME` | Yes | — | Public HTTPS hostname used in the Alexa skill manifest and endpoint validation. |
| `MA_HOSTNAME` | No | — | Public Music Assistant hostname for stream and artwork URL rewriting. |
| `APP_USERNAME` | No | — | Basic-auth username for the add-on UI and API endpoints. |
| `APP_PASSWORD` | No | — | Basic-auth password for the add-on UI and API endpoints. |
| `LOCALE` | No | `de-DE` | Alexa locale used for the skill manifest and interaction models. |
| `AWS_DEFAULT_REGION` | No | `us-east-1` | AWS region used by ASK CLI. |
| `TZ` | No | `America/Chicago` | Timezone for logs and timestamps. |
| `SKIP_URL_VALIDATION` | No | `false` | Skip outgoing validation of the rewritten Music Assistant stream URL. |

## Notes

- The add-on listens internally on port `5000`.
- The Web UI opens `/setup`; visiting `/` redirects to `/status`.
- `SKILL_HOSTNAME` must be publicly reachable by Amazon over HTTPS.
- If you use Cloudflare, set the minimum TLS version to `1.2`.
