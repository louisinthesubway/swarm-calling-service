# SWARM changes to Signal-Calling-Service

This repository is SWARM's fork of
[signalapp/Signal-Calling-Service](https://github.com/signalapp/Signal-Calling-Service)
(AGPL-3.0-only, like upstream). It is the pinned build source of the group-call server (SFU) that
SWARM Messenger's staging stack runs at `sfu.chat.swarm.green`. The deployment itself (Dockerfile,
configuration, secrets, firewall, runbook) lives in
[louisinthesubway/swarm-messenger-server](https://github.com/louisinthesubway/swarm-messenger-server),
`deploy/staging/sfu/` and `docs/STAGING.md`, section 5d.

Branches:

| Branch | What it is |
|---|---|
| `main` | upstream `main` as forked on 2026-09-29, `56da39e` ("Bump version to v141") |
| `swarm-main` | the build source: upstream `56da39e` plus the one change below |

No cryptography, no media or protocol code and no API is changed. The group-call credentials are
still checked by upstream code (HMAC-SHA256 group tokens, zkgroup call-link credentials).

## 1. The frontend may read its secrets from the environment (2026-09-29)

`frontend/src/config.rs`, `frontend/Cargo.toml` (clap feature `env`).

Upstream takes the frontend's two secrets only as command-line options: `--authentication-key` (the
hex key that checks group-call tokens) and `--zkparams` / `--old-zkparams` (base64
`GenericServerSecretParams` that check call-link credentials); its own `docker-entrypoint.sh` passes
them that way. A process's command line is readable by every local user of the host through
`/proc/<pid>/cmdline` and shows up in any `ps` listing, and SWARM's staging host runs other service
users and is operated by people and agents who list processes. With this change each of the three
options can also be given as an environment variable, which only the process's own user and root can
read:

| Option | Environment variable |
|---|---|
| `--authentication-key` | `CALLING_AUTH_KEY` |
| `--zkparams` | `CALLING_ZKPARAMS` |
| `--old-zkparams` | `CALLING_OLD_ZKPARAMS` |

A command-line option still wins over the variable, so upstream's entrypoint and every existing
invocation behave exactly as before. `hide_env_values` keeps the values out of `--help`. The
frontend never logged these values (`print_config` prints only whether `old_zkparams` is set) and
still does not.

Nothing else differs from upstream `56da39e`: `git diff 56da39e swarm-main` shows these two files and
this document.
