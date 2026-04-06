# Music Assistant Alexa Skill Add-on

This add-on provisions and hosts the Music Assistant Alexa skill from inside Home Assistant.

## Install

1. Add this repository as a custom add-on repository in Home Assistant.
2. Install `Music Assistant Alexa Skill`.
3. Configure the add-on options.
4. Start the add-on.
5. Open the add-on Web UI and complete `/setup`.

ASK CLI credentials are stored persistently under `/data/.ask`, so re-authentication is not required after every restart.

## Configuration

| Option | Required | Default | Description |
| --- | :---: | :---: | --- |
| `SKILL_HOSTNAME` | Yes | — | Public HTTPS hostname used in the Alexa skill manifest and endpoint validation. |
| `MA_HOSTNAME` | No | — | Public Music Assistant hostname. Required on non-APL devices and for externally reachable artwork URLs. |
| `APP_USERNAME` | No | — | Basic-auth username for the add-on UI and API endpoints. |
| `APP_PASSWORD` | No | — | Basic-auth password for the add-on UI and API endpoints. |
| `LOCALE` | No | `de-DE` | Alexa locale used for the skill manifest and interaction models. |
| `AWS_DEFAULT_REGION` | No | `us-east-1` | AWS region used by ASK CLI. |
| `TZ` | No | `America/Chicago` | Timezone for container logs and timestamps. |
| `SKIP_URL_VALIDATION` | No | `false` | Skip outgoing validation of the rewritten Music Assistant stream URL. |

## Notes

- The add-on listens internally on port `5000`.
- The Web UI opens `/setup`; visiting `/` redirects to `/status`.
- `SKILL_HOSTNAME` must be publicly reachable by Amazon over HTTPS.
