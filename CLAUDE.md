# CLAUDE.md — dotfiles-template

Guidance for Claude Code / people adopting or maintaining this template.

## What this is

A minimal chezmoi source dir meant to be **forked** and made your own. Adopters init chezmoi from
their fork; upstream changes flow in via the Upstream Sync workflow. Don't use
`sonic-com/dotfiles-template` directly.

## chezmoi conventions

Source filenames encode target attributes:

- `dot_X` → `~/.X`; `private_X` → mode `0600` (dir `0700`); `symlink_X` → a symlink whose file
  contents are the link target; `*.tmpl` → rendered with chezmoi data.
- `.chezmoiexternal.toml` pulls in external content; `.chezmoi.toml.tmpl` is the per-machine
  config generated at `chezmoi init`.

Pre-commit hooks ship **disabled** (see `.pre-commit-config.yaml`) so they don't fight chezmoi's
`autoCommit`/`autoPush` flow or require a toolchain on every clone. Enable what you want, then run
`pre-commit install`.

## SSH

`~/.ssh/config` (`private_dot_ssh/private_config`) includes, in precedence order (first match wins):

1. `Include config.d/*.conf` — your **local, per-machine** drop-ins in `~/.ssh/config.d/`.
2. `Include ~/.local/share/dotfiles-ssh/config.d/*.conf` — **team-wide** snippets from dotfiles-ssh.
3. the `Host *` defaults in `private_config`.

The `.conf` suffix is deliberate: editor backups/temp files (`foo.conf~`, `.conf.swp`) are never
sourced. Add host stanzas as drop-ins rather than editing `private_config`, so upstream syncs
don't conflict.

### The host CA

`~/.ssh/known_hosts_ca` is a symlink to `~/.local/share/dotfiles-ssh/known_hosts_ca`, wired in via
`UserKnownHostsFile ~/.ssh/known_hosts ~/.ssh/known_hosts_ca`. `known_hosts` stays first so your
own accepted keys still land there; the CA file is read-only lookup. Because Sonic hosts present
CA-signed certificates, `StrictHostKeyChecking yes` is safe — add any non-CA hosts to
`known_hosts` explicitly.

## Private vs public dotfiles-ssh

`dotfiles-ssh` is **private** by default and cloned over SSH (needs `github.com/sonic-com` access).
If it is ever made public, switching is a one-line comment toggle in `.chezmoiexternal.toml` — the
`type = "archive"` alternative fetches the same content over HTTPS. The landing path is identical,
so the symlink and ssh config don't change.

## Staying in sync with upstream

`.github/workflows/upstream-sync.yml` fast-forwards your fork's `main` from upstream on a schedule
(and on demand via "Run workflow"). To use it in your fork:

1. **Settings → Actions → General → allow Actions** (forks disable them by default).
2. Only if you want workflow-file changes to sync too: add a repo secret `UPSTREAM_SYNC_TOKEN` = a
   PAT with `workflow` scope (the default `GITHUB_TOKEN` can't push under `.github/workflows/`).

The workflow no-ops in the upstream repo itself (`if: github.repository != 'sonic-com/dotfiles-template'`).
