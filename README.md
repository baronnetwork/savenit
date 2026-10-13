# savenit

The nit pipeline. Complain, walk away, agents fix it.

Three commands, all in `~/bin`, one directory of state (`~/claude-busywork/`):

| Command | What it does |
|---|---|
| `savenit "complaint"` | Saves a timestamped nit file: your complaint, optional clipboard evidence, hostname, OS, invocation PWD. Never opens an editor. Save and move on. |
| `fixnit` | Fires one background Sonnet agent at the newest unclaimed nit. `fixnit -lite` uses Kimi via LiteLLM in `--bare` mode. fixnit stamps its own audit line into the nit, launches from the complaint's PWD, and never doubles an in-flight nit. |
| `checknit` | Reports working agents with PIDs and runtimes, the backlog, a full directory listing, tails logs of agents past 10 minutes, and offers to kill agents past 5. |

## Quick start

```bash
savenit "zshrc alias stopped loading on new terminals"
fixnit -lite
checknit
```

A fixed nit renames itself from `new-nit-*` to `done-nit-*` with backup, changelog,
and resolution summary attached.

## Install

Full scripts, template, prerequisites, and behavior notes:

[docs/how-to-install-savenit.md](docs/how-to-install-savenit.md)
