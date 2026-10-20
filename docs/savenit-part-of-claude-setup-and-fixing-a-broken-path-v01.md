# savenit: part of the Claude setup (install it, and how to fix a broken PATH) — v01, 2026-10-15

## What it is and why it belongs here
`savenit` (public repo `baronnetwork/savenit`, clone at `/mnt/d/github-baron/savenit` on the home box) is the nit pipeline:
complain with `savenit "..."`, walk away, background agents fix it (`fixnit`, `checknit`, `convoynit`, `auditnit`).
It is **part of the Claude setup that should be installed on every machine** that runs css (the pure laptop, Omarchy,
WSL), the same way the harness skills are. It is *not* installed by `install.sh` (yet). Many css fixes start as nits.

## Install (any machine)
```bash
git clone git@github-baron:baronnetwork/savenit.git ~/consulting/savenit     # github-baron = baronnetwork SSH alias
export PATH="$HOME/consulting/savenit/bin:$PATH"                             # add to ~/.zshenv (mac) / ~/.bashrc (linux)
```
Full details: `savenit/docs/how-to-install-savenit.md`. Check with `command -v savenit fixnit checknit`.

## What changed on 2026-10-15 (and why things may be broken now)
1. **Scripts moved into `bin/`.** Before, PATH pointed at the repo root (`~/consulting/savenit`). Now it must point at
   `~/consulting/savenit/bin`. **If `savenit: command not found` after a `git pull`, this is why.**
   Fix: change the PATH line in `~/.zshenv` / `~/.bashrc` to end in `/savenit/bin`, open a new shell.
2. **Nit state moved** from `~/claude-busywork/` to `${XDG_STATE_HOME:-~/.local/state}/savenit/` (override with `NITDIR`).
   The scripts migrate the old dir automatically on first run, but only when no nit agent is running; while one is,
   they keep using the old dir. If nits seem to vanish: look in both places, finish/kill running agents
   (`pgrep -fl 'claude.*(new|inprogress)-nit'`), then run `checknit` once to migrate. Nits are runtime state and are
   deliberately NOT in the repo (public repo; nit text can mention private projects).
3. **`convoynit` default `FIXNIT`** is now the sibling `fixnit` (was a hardcoded `/Users/hbaron/...` path).
   `FIXNIT=<path>` still overrides. Also scrubbed `/Users/hbaron/` from changelogs.
4. **Docs fix:** the install doc told people to clone via plain `github.com` (authenticates as ghostnote, wrong
   identity). It now uses `git@github-baron:...`.

## Gotchas
- The "is an agent in flight" check is `pgrep -f 'claude.*(new|inprogress)-nit'`; a command line containing both
  strings (even a test harness) counts as in flight and defers the migration.
- Pure laptop shares `css-v3.sh` with this setup; keep savenit paths home-relative (no `/mnt/d`).
- Related: `docs/shared-claude-across-machines-plan-v01.md` (how skills/instructions are shared across machines).
