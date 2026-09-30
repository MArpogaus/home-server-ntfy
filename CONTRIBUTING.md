# Contributing

The branch flow, the hooks, the releases and the house style are in
`home-server/CONTRIBUTING.md`.

## Tags

A tag names the ntfy version that the release deploys, such as `2.28.0`. A later
release on the same version adds a counter: `2.28.0-1`, `2.28.0-2`.

## Checks in this repository

- Hooks: the basics, ansible-lint and commitizen.
- Ansible variables are `<role>_*`.
- Renovate updates the container image tags in the role defaults, through the
  preset that `.github/renovate.json` extends.
- `home-server` checks this repository out at `services/ntfy`. Work on it
  there. `home-server/CONTRIBUTING.md` says how a change here reaches the pin.
