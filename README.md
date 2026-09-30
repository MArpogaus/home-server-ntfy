# home-server-ntfy

ntfy in a rootless Podman pod, with an Ansible role that deploys it. ntfy sends
push notifications to the phone. No other service needs it.

| Container | Job | Default memory ceiling |
|---|---|---|
| ntfy-server | Push notifications on the loopback port | 128M |

## Configuration

The service follows the configuration interface in
`home-server-template/README.md`, "Configuration interface".

| Variable | Default | Controls |
|---|---|---|
| `ntfy_service_password` | required | Login of the phone, user `ntfy`: topic `alerts`, and `agent` with an agent token |
| `ntfy_service_token` | required | Token of Alertmanager, user `alertmanager`, which may only publish to `alerts`: `tk_` plus 29 lowercase letters or digits |
| `ntfy_service_agent_token` | empty | Token of a coding agent, user `agent`, which reads and publishes on `agent` only; empty: no user `agent` |
| `ntfy_service_hostname` | empty | The public hostname; links in a notification use it, or the loopback port without it |
| `ntfy_service_config` | `{}` | ntfy's environment (`NTFY_*`), merged over `ntfy_service_config_defaults` |
| `ntfy_service_memory` | `{}` | Memory ceilings per container |
| `ntfy_service_server_image` | see `defaults/main.yml` | The image |

The role keeps the listen port, the base URL, the database paths and the
access rules; the config cannot change them. The users and the tokens reach
ntfy as Podman secrets.

A token: `echo "tk_$(openssl rand -hex 15 | cut -c1-29)"`.

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
  internet when the inventory adds its site. ntfy does not work under a
  subpath, so it needs a hostname of its own.
- Every start syncs the users, their tokens and their access into
  `data/user.db`. `NTFY_AUTH_DEFAULT_ACCESS=deny-all` is the only gate of the
  public site: anonymous clients can neither read nor publish.
- By default, the message cache keeps 72 hours, in `data/cache.db`.

## Role contract

The contract is in `home-server-template/README.md`.

## LLM coding tools

LLM-based coding tools write most of the code and documentation of this
project. The maintainer sets the goals and the design, reviews every change and
is responsible for it. Each change runs on a VM before it reaches a host.

## License

MIT
