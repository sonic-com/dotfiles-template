# dotfiles-template

A [chezmoi](https://www.chezmoi.io/) starter for Sonic workstations (your own login, not shared
server accounts). It gives you Sonic's SSH host-CA trust and team-wide SSH config snippets out of
the box, and stays updatable as the template evolves.

## Bootstrap — fork it first

**Fork this repo, then init chezmoi from your fork.** That is the only supported path — don't
`chezmoi init` straight from `sonic-com/dotfiles-template`: you can't push back to it, and you'd
have nowhere to keep your own changes.

```bash
# 1. Fork sonic-com/dotfiles-template on GitHub (e.g. to <you>/dotfiles).
# 2. Init chezmoi from YOUR fork:
chezmoi init --apply https://github.com/<you>/dotfiles.git
```

Preview with `chezmoi diff`, apply with `chezmoi apply`. Then do the one-time
[fork setup](#set-up-your-fork-one-time) below to turn on auto-sync and secret scanning.

## Set up your fork (one-time)

After forking and `chezmoi apply`, a couple of things need enabling on your fork's GitHub repo and
in your chezmoi source:

1. **Enable Actions (for auto-sync).** On your fork: **Settings → Actions → General → Allow all
   actions** — forks ship with Actions disabled. The `Upstream Sync` workflow then merges
   `sonic-com/dotfiles-template` into your `main` daily (and on demand via **Actions → Upstream
   Sync → Run workflow**).
2. **(Optional) Sync workflow-file changes too.** The default `GITHUB_TOKEN` can't push under
   `.github/workflows/`. If you want updates to the workflow itself to sync, add a repo **secret**
   `UPSTREAM_SYNC_TOKEN` = a personal access token with `workflow` scope (**Settings → Secrets and
   variables → Actions**).
3. **Turn on secret scanning locally.** In your chezmoi source (`chezmoi cd`), run
   `pre-commit install`. The always-on `gitleaks` hook then blocks commits containing secrets.

## What you get

- **`~/.ssh/config`** — a small, sane starter: strict host-key checking (safe because Sonic hosts
  present CA-signed certs), `~/.ssh/known_hosts_ca` trust, and two `Include`s for drop-in configs.
- **Sonic host-CA trust** — no more host-key prompts/warnings for `*.sonic.net` / `*.sontrak.net`.
- **Team-wide SSH snippets** — anything in the shared
  [`dotfiles-ssh`](https://github.com/sonic-com/dotfiles-ssh) repo (e.g. agent-forwarding to
  build and monitoring hosts) is `Include`d automatically.
- **Auto-sync** — a GitHub Action merges upstream into your repo on a schedule.
- **Safer commits** — an always-on `gitleaks` pre-commit hook blocks secrets, and new git repos
  you create inherit a pre-commit setup automatically (via `~/.config/git/template`).

## More

- [SSH config layout & the host CA](CLAUDE.md#ssh)
- [Public vs private dotfiles-ssh](CLAUDE.md#private-vs-public-dotfiles-ssh)
- [Handling secrets](CLAUDE.md#secrets) and [git defaults](CLAUDE.md#git-defaults)
- [How upstream sync works](CLAUDE.md#staying-in-sync-with-upstream)
- [chezmoi conventions](CLAUDE.md#chezmoi-conventions)
