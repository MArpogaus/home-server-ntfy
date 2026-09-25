# home-server-ntfy

ntfy in a rootless Podman pod, with an Ansible role that deploys it. ntfy sends
push notifications to the phone. No other service needs it.

| Container | Job | Memory ceiling |
|---|---|---|
| ntfy-server | Push notifications on `127.0.0.1:8081` | 128M |

## Configuration

| Variable | Default | Controls |
|---|---|---|
| `ntfy_service_server_image` | see `defaults/main.yml` | The image |
| `ntfy_service_password` | required | Login of the phone, user `ntfy`, topic `alerts` |
| `ntfy_service_token` | required | Token of the publisher, such as Alertmanager: `tk_` plus 29 lowercase letters or digits |
| `ntfy_service_base_url` | `http://127.0.0.1:8081` | The address that links in a notification use |

The token: `echo "tk_$(openssl rand -hex 15 | cut -c1-29)"`.

## Alerts from home-server-monitoring

While ntfy is in `base_setup_services`, the inventory of `home-server` points
Alertmanager's webhook at `/alerts?template=alertmanager` with
`ntfy_service_token`. The template, `quadlets/configs/templates/alertmanager.yml`,
makes the alert name the title and the summary and the description the
message, and it maps the severity to a priority:

| Severity | ntfy priority |
|---|---|
| `critical` | 5 |
| `warning` | 3 |
| `info` | 2 |

## Specifics

- The pod publishes only on the host loopback. The proxy puts it on the
  internet when the inventory adds its site; ntfy does not work under a
  subpath, so it needs a hostname of its own.
- Every start syncs the phone's user and the publisher's token into
  `data/user.db`. Anonymous clients can neither read nor publish.
- The message cache keeps 72 hours, in `data/cache.db`.

## Role contract

The contract is in `home-server-template/README.md`.

## LLM coding tools

This project is developed with LLM-based coding tools. They write most of the
code and documentation. The maintainer sets the goals and the design, reviews
every change and is responsible for it. Changes are tested on a VM before they
reach a host.

## License

MIT
