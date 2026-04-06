# Music Assistant Alexa Skill - Home Assistant Add-on

This folder contains the Home Assistant add-on wrapper for the `music-assistant-alexa-skill-prototype` project. It is intended for development and local testing inside Home Assistant Supervisor.

## How to Run

### Prerequisites

- Home Assistant Supervisor running with add-on support
- A repository containing this `addons/music-assistant-skill` folder added to the Supervisor add-on store
- A public HTTPS endpoint for the skill, if you plan to test the full Alexa flow

### 1. Install the add-on

1. Add this repository as a custom add-on repository in Home Assistant Supervisor.
2. Open Supervisor > Add-on Store and install the "Music Assistant Alexa Skill" add-on.
3. Open the add-on configuration and set the options below as needed.
4. Start the add-on and open the Web UI. The add-on opens `/setup` directly.
5. ASK CLI credentials are stored in the add-on data directory under `/data/.ask` so they survive restarts.

### 2. Access the service

- The web UI is available on the configured `PORT` (default `5000`).
- The `/setup` page guides you through ASK CLI setup and skill provisioning.
- The `/status` page shows a quick health check for the local service and skill configuration.

### Configuration Options

Set these in the Supervisor add-on configuration:

| Variable             | Required | Default           | Description                                                                                             |
| -------------------- | :------: | ----------------- | ------------------------------------------------------------------------------------------------------- |
| `SKILL_HOSTNAME`     |   Yes    | —                 | Public HTTPS hostname used in the Alexa skill manifest and endpoint validation.                         |
| `MA_HOSTNAME`        |    No    | —                 | Music Assistant server hostname. Required if your device does not support APL or if you want album art. |
| `APP_USERNAME`       |    No    | —                 | Username for the Music Assistant web UI and API basic authentication.                                   |
| `APP_PASSWORD`       |    No    | —                 | Password for the Music Assistant web UI and API basic authentication.                                   |
| `LOCALE`             |    No    | `en-US`           | Skill locale used by setup and interaction model operations.                                            |
| `AWS_DEFAULT_REGION` |    No    | `us-east-1`       | AWS region used by ASK CLI operations when needed.                                                      |
| `TZ`                 |    No    | `America/Chicago` | Container timezone used for logs and timestamps.                                                        |
| `SKIP_URL_VALIDATION`|    No    | `false`           | Skip stream URL reachability checks when Docker/network routing prevents the add-on from validating the public HTTPS URL. |

### Notes

- The add-on is configured to pull the prebuilt container image from GHCR (`ghcr.io/alams154/music-assistant-skill:<version>`).
- The image includes the packages needed to build Python dependencies from `requirements.txt`.
- The add-on always listens on the internal port `5000`; the published Home Assistant port mapping must stay aligned with that fixed port.
- The add-on Web UI entry opens `/setup`, while a plain GET request to `/` redirects to `/status`.
