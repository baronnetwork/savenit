# savenit

The nit pipeline. Complain, walk away, an agent fixes it.

One command, `nit`, in the repo's `bin/` (put `bin/` on your PATH). State lives in `~/.local/state/savenit/`.

| Command | What it does |
|---|---|
| `nit "complaint"` | Saves a short nit file: your complaint, host, OS, the directory you were in. `-c` also attaches the clipboard. Never prompts when given args. |
| `nit fix [-a] [id]` | Runs one background Claude agent on the newest nit (or `id`); `-a` works through the whole queue one at a time. Lite (LiteLLM) when available, else direct sonnet. |
| `nit ls` | Read-only status: working, needs you, queued, recent fixes. |
| `nit log [id]` | Tails an agent's log. |
| `nit import` | Moves old-format `new-nit-*.md` files into the new queue. |

## Quick start

```bash
nit "zshrc alias stopped loading on new terminals"
# → Saved. Fix with: nit fix
nit -c "this build error"        # attaches your clipboard (terminal output) as evidence
nit fix
nit ls
```

A nit moves through `new/` → `working/` → `done/` (or `needs-you/` when the agent has a question or the syntax check
after the fix fails). The agent writes a short resolution and the list of files it changed into the nit; git is the
backup and the changelog.

The older five commands (`savenit`, `fixnit`, `checknit`, `convoynit`, `auditnit`) are still in `bin/` while `nit` is
being tried out.

## Install

[docs/how-to-install-savenit.md](docs/how-to-install-savenit.md)
