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

Preview with `chezmoi diff`, apply with `chezmoi apply`. Your fork can auto-track upstream — see
[Staying in sync](CLAUDE.md#staying-in-sync-with-upstream).

## What you get

- **`~/.ssh/config`** — a small, sane starter: strict host-key checking (safe because Sonic hosts
  present CA-signed certs), `~/.ssh/known_hosts_ca` trust, and two `Include`s for drop-in configs.
- **Sonic host-CA trust** — no more host-key prompts/warnings for `*.sonic.net` / `*.sontrak.net`.
- **Team-wide SSH snippets** — anything in the shared
  [`dotfiles-ssh`](https://github.com/sonic-com/dotfiles-ssh) repo (e.g. agent-forwarding to
  build and monitoring hosts) is `Include`d automatically.
- **Auto-sync** — a GitHub Action fast-forwards your fork from upstream on a schedule.

## More

- [SSH config layout & the host CA](CLAUDE.md#ssh)
- [Public vs private dotfiles-ssh](CLAUDE.md#private-vs-public-dotfiles-ssh)
- [Staying in sync with upstream](CLAUDE.md#staying-in-sync-with-upstream)
- [chezmoi conventions](CLAUDE.md#chezmoi-conventions)
