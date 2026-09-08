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

## Secrets

**Never commit secrets** (passwords, tokens, API keys, private keys) here — this is a git repo,
and once pushed a secret is leaked. The `gitleaks` pre-commit hook is left **on** to catch
accidents (`pre-commit install` to activate it locally), but don't rely on it. Handle secrets one
of these chezmoi-supported ways instead:

- **Encrypt in the repo (age or GnuPG).** Configure your recipient/key
  ([chezmoi encryption](https://www.chezmoi.io/user-guide/encryption/)), then
  `chezmoi add --encrypt ~/.some-secret`. The source is stored as `encrypted_…` ciphertext and
  decrypted only on `chezmoi apply`.
- **Fetch from a secret manager at apply time.** Reference the secret inline in a `*.tmpl` file
  rather than storing it — e.g. HashiCorp Vault (`{{ (vault "path").data.data.value }}`),
  1Password (`onepasswordRead`), or `pass`. The secret lives in the manager, never in git.
- **Keep it out of chezmoi** — list it in `.chezmoiignore` and manage it by hand.

At Sonic, use the vault-path conventions in the internal wiki (personal-only secrets under
`secret/vaultPass/<user>/...`, or 1Password).

## Git defaults

`dot_config/git/config` sets `init.defaultBranch = main` and
`init.templateDir = ~/.config/git/template`. That template dir ships a `hooks/pre-commit` shim, so
**new** repos you `git init` / `git clone` get pre-commit auto-installed — it no-ops until a repo
has a `.pre-commit-config.yaml`. Combined with the always-on `gitleaks` hook, secret scanning is
on by default across the repos you create.

## Staying in sync with upstream

`.github/workflows/upstream-sync.yml` merges upstream `main` into your `main` on a schedule (and on
demand via "Run workflow"), and no-ops in the upstream repo itself. It merges rather than
fast-forwards so it works once you have your own commits, and for "Use this template" repos, which
share no history with upstream; those need one manual first merge (the failed run prints the
commands). **Per-fork setup steps
are in the [README](README.md#set-up-your-fork-one-time).**
