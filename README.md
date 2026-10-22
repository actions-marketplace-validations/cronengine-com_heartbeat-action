# CronEngine Heartbeat

Get an alert when a GitHub Actions workflow fails, or when a scheduled workflow quietly stops running.

GitHub's `schedule:` trigger can start runs late or skip them, and a failed nightly job is easy to miss in a long list of runs. This action pings a [CronEngine](https://cronengine.com/cron-monitoring?utm_source=github&utm_medium=action) heartbeat from your workflow. CronEngine knows when the next ping is due, and tells you by email, webhook, Slack, Discord or Telegram when it does not arrive, or when the run reports a failure.

## Setup

1. In [CronEngine](https://cronengine.com/?utm_source=github&utm_medium=action), create a **heartbeat** with the same schedule as your workflow. The free plan includes heartbeats, no credit card needed.
2. Copy its ping URL.
3. In your repository, open **Settings › Secrets and variables › Actions** and add a secret named `CRONENGINE_PING` with that URL.

## Usage

### The short version

Ping once, at the end, when everything succeeded. If the workflow fails or never runs, the ping goes missing and you get an alert.

```yaml
name: Nightly export

on:
  schedule:
    - cron: '0 3 * * *'

jobs:
  export:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: ./scripts/export.sh

      - uses: cronengine-com/heartbeat-action@v1
        with:
          ping-url: ${{ secrets.CRONENGINE_PING }}
```

### Start, success and failure

Report the start as well, to see how long every run takes, and report a failure straight away instead of waiting for the missing ping.

```yaml
    steps:
      - uses: cronengine-com/heartbeat-action@v1
        with:
          ping-url: ${{ secrets.CRONENGINE_PING }}
          status: start

      - uses: actions/checkout@v4
      - run: ./scripts/export.sh

      - if: always()
        uses: cronengine-com/heartbeat-action@v1
        with:
          ping-url: ${{ secrets.CRONENGINE_PING }}
          status: ${{ job.status }}
```

`job.status` is `success`, `failure` or `cancelled`; the last two are reported as a failure.

## Inputs

| Input | Default | Description |
|---|---|---|
| `ping-url` | required | The heartbeat's ping URL. Keep it in a secret. |
| `status` | `success` | `start`, `success` or `fail`. Also accepts `${{ job.status }}`. |
| `timeout` | `10` | Seconds to wait for one attempt. |
| `retries` | `3` | Retries when the network fails. |
| `fail-on-error` | `false` | Fail the step when the ping cannot be sent. |

## Safe by default

Monitoring should never be the reason a build fails. If CronEngine cannot be reached, this action logs a warning and lets the workflow carry on, unless you set `fail-on-error: true`. The ping URL is masked in the logs.

It uses only `bash` and `curl`, so it works on Linux, macOS and Windows runners.

## Want the workflow to start on time too?

GitHub's scheduled runs can start late at busy times. CronEngine can also start the workflow itself through the dispatch API, on time and in your own timezone. See [GitHub Actions cron running late? How to start workflows on time](https://cronengine.com/guides/github-actions-schedule?utm_source=github&utm_medium=action).

## License

MIT
