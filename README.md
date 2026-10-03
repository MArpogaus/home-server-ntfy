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
| `ntfy_service_users` | required | The users by name, each with a `password`, a `token` or both, and its `access`: topic pattern to `rw`, `ro`, `wo` or `deny` |
| `ntfy_service_hostname` | empty | The public hostname; links in a notification use it, or the loopback port without it |
| `ntfy_service_config` | `{}` | ntfy's environment (`NTFY_*`), merged over `ntfy_service_config_defaults` |
| `ntfy_service_memory` | `{}` | Memory ceilings per container |
| `ntfy_service_server_image` | see `defaults/main.yml` | The image |

The role keeps the listen port, the base URL, the database paths and the
default access `deny-all`. While a user has access, it also keeps the access
rules. The config cannot change them. The users and the tokens reach ntfy as
Podman secrets.

```yaml
ntfy_service_users:
  ntfy:                        # the phone
    password: "..."
    access: {alerts: rw, "agent*": rw}
  alertmanager:
    token: tk_...
    access: {alerts: wo}
  agent:                       # coding agents, one topic each: agent-<name>
    token: tk_...
    access: {"agent*": rw}
```

Make a token with `echo "tk_$(openssl rand -hex 15 | cut -c1-29)"`. A token user without a
password logs in with its token as the password.

## Alerts from home-server-monitoring

While ntfy is in `base_setup_services`, the deployment directory points
Alertmanager's webhook at `/alerts?template=alertmanager` with the token of
the user `alertmanager`. The template, `quadlets/configs/templates/alertmanager.yml`,
makes the alert name the title and the summary and the description the
message, and it maps the severity to a priority:

| Severity | ntfy priority |
|---|---|
| `critical` | 5 |
| `warning` | 3 |
| `info` | 2 |

## Specifics

- The pod publishes only on the host loopback. The proxy puts it on the
  internet when the deployment directory adds its site. ntfy does not work under a
  subpath, so it needs a hostname of its own.
- Every start syncs the users, their tokens and their access into
  `data/user.db`. `NTFY_AUTH_DEFAULT_ACCESS=deny-all` is the only gate of the
  public site: anonymous clients can neither read nor publish.
- A leaked password or token can make more tokens through the API, and a new
  secret does not revoke them. To revoke a leak, rename the user in
  `ntfy_service_users`: ntfy then drops the old user with all its tokens. A
  renamed `alertmanager` needs the new name in the deployment directory's
  `monitoring_service_alert_webhook_token`.
- By default, the message cache keeps 72 hours, in `data/cache.db`.

## Role contract

The contract is in `home-server-template/README.md`.

## LLM coding tools

LLM-based coding tools write most of the code and documentation of this
project. The maintainer sets the goals and the design, reviews every change and
is responsible for it. Each change runs on a VM before it reaches a host.

## License

MIT
