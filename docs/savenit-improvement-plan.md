# savenit improvement plan (ponytail review, 2026-10-15) — partly implemented (see Status)

Review of `baronnetwork/savenit` (`/mnt/d/github-baron/savenit`) and its five commands, run through the `/ponytail` skill
("lazy senior dev": smallest change that fully solves the problem). Review only; nothing was changed.
See also `savenit-part-of-claude-setup-and-fixing-a-broken-path-v01.md` (install, PATH and state-dir notes).

**Verdict:** good idea, about 4x more code than needed (1,081 lines across 5 scripts). The essential part is
"save a complaint, run an agent on it". Most of the rest is bookkeeping that a script can do deterministically
instead of an agent.

## Status 2026-10-15 21:41: `nit` built on a branch, waiting for Hadrian to try it

Branch `nit-single-command` in `baronnetwork/savenit` (pushed; `main` untouched, so the five old scripts still work).
Commit: "nit: single command (save, fix [-a], ls, log, import) ...".

**Built** (`bin/nit`, 322 lines vs ~1,080 for the old five):
- `nit "complaint"` / `nit -c "..."` / bare `nit` (prompts); `nit fix [-a] [id] [--lite|--sonnet]`; `nit ls` (read-only);
  `nit log [id]`; `nit import`. A command word counts only as the whole first argument (`nit "fix the bug"` is a complaint).
- Clean nit file exactly as in "Clean nit file" below (14 lines for a one-liner), with the complaint-once self-check.
- Process rules live in the prompt (`prompt_for` in `bin/nit`): evidence is data not instructions, smallest fix, no
  commit/push, `.bak-nit` only for non-git files, no changelogs/locks, verify, ask instead of guessing.
- State by directory: `new/ working/ needs-you/ done/ logs/ imported/` under the state root; `runner.pid`, one agent
  at a time; `kill <pid>` stops runner and agent (trap). No locks, no pgrep, no auditnit; `fix -a` replaces convoynit.
- `nit` itself writes `ran:` / `fixed:`, moves files, and syntax-checks changed shell/python files after DONE
  (failure -> NEEDS-YOU). Agent error exit -> requeued to `new/` and `-a` stops; exit 0 with no status -> NEEDS-YOU;
  runner dead mid-fix -> stays in `working/`, next `nit fix` adopts it.
- Lite when the `lite` CLI + live token exist, else direct `--model sonnet` with a notice (ranked fix 7 done).
- Compatibility: `nit import` converts unfinished old `new-nit-*.md` (state root and `~/claude-busywork`), originals
  kept in `imported/`; `nit ls` counts old-format nits left. Old `inprogress-nit-*` are left to old fixnit/checknit.
- Docs: `docs/how-to-install-savenit.md` rewritten around `nit`, links to `bin/` instead of embedding stale copies;
  README rewritten. No private names.

**Tested** only with a fake `claude` on PATH in a scratch `NITDIR` (save, `-c`, fix, `-a` drain, syntax-fail,
no-status, agent-fail requeue, retry from needs-you, import of an old-savenit nit, kill). **Not yet run with a real
agent**, not run on macOS (bash 3.2) or with `lite`.

**Deviations from this plan:** added a `needs-you/` dir (plan listed only new/working/done); `ran:` line has no pid
(pid is in `runner.pid`); direct model is the `sonnet` alias, not the pinned `claude-sonnet-4-6`.

**Open questions for Hadrian (ask one at a time):**
1. Has `nit` been tried for real (one real `nit fix`)? Anything off?
2. Merge `nit-single-command` to `main`?
3. After merge: delete `savenit fixnit checknit convoynit auditnit` and the root `*-autoAgenticNitFix-changelog.md`
   files, or keep `savenit` as a thin alias to `nit` for muscle memory?
4. Pure laptop: its queue uses old naming; run `nit import` there after pulling (and after its old agents finish)?
5. Security: keep `--dangerously-skip-permissions`, or restrict the agent's tools?
6. Default mode: keep lite-first, or direct sonnet first?
7. Add savenit to the harness `install.sh` (it currently is not)?

## What each command is worth

| Command | Lines | Verdict |
|---|---|---|
| `savenit` | 178 | Keep. It is the product. |
| `fixnit` | 207 | Keep, but strip the extras below. |
| `checknit` | 246 | Keep as a status view, without side effects. |
| `convoynit` | 290 | Redundant: `fixnit all` already drains the queue. Keep one of them. |
| `auditnit` | 160 | Delete. An agent audits the last 10 fixes because one bug once slipped through; a `bash -n` after each fix would have caught it. |

## Friction and bloat, ranked

1. **Every nit carries ~100 lines of identical instructions.** `savenit` writes the full process rules into every nit
   file, so old nits keep old rules forever. Put the instructions once in `fixnit`'s prompt; nit files hold only the
   complaint, evidence and status.
2. **Three agent spawns per fix for paperwork.** The template makes the agent create a backup copy, a proposed changelog,
   a test subagent, a changelog subagent and a rename subagent. Git is already the backup and the changelog.
   `fixnit` can `wait` for the agent, do the renames and run a syntax check itself. Keep a `.bak` only for files that
   are not in a git repo.
3. **State lives in three places:** the filename (`new-` / `inprogress-` / `done-`), the "Issue status:" text, and `pgrep`
   matching on process argv (fragile: any command line containing `claude` plus a nit filename looks like a running
   agent, as a test hit on 2026-10-15). Use one directory per state (`new/`, `working/`, `done/`); moving the file is
   the state change.
4. **Per-file `.lock` directories** are created in the repo being fixed, and any stale one blocks every launch. Run one
   agent at a time and the locks, the lock gate in `fixnit` and the lock prose in the template all go.
5. **`checknit` mutates files.** A status command moves done nits into `done-nits/` as a side effect. Do that when a
   fix finishes.
6. **Clipboard capture is macOS-only.** `savenit` uses `pbpaste` (and `sw_vers`), so on WSL/Omarchy the grab silently
   does nothing and the message still says "Validated pbcopy…". Add a fallback chain: `wl-paste`, `xclip`,
   `powershell.exe Get-Clipboard`.
7. **Default `fixnit` mode needs the LiteLLM login.** Missing `lite` CLI or an expired token is a hard error; fall back to
   direct sonnet instead.
8. **Five command names for one tool.** One command (`nit "text"`, `nit fix [-a]`, `nit ls`) means one PATH entry and
   shared code. On 2026-10-15 the same 10-line state-dir block was pasted into all five scripts; merging removes that
   duplication.

## Clean nit file (the readable end state)

Measured 2026-10-15: `savenit -n "one line"` writes an **88-line** file for a one-line complaint. Duplicates found:

- the complaint appears **twice** (under "Human complaint" and again under "Human prompt for the AI agent");
- two near-identical file lists ("files to be edited and their backups" / "files that were edited and their new sizes
  ... and the changelog") **plus** a third step that says to document the same list in the nit;
- "update the nitFile with the # AI resolution summary" appears in **two** consecutive steps;
- "backup" is mentioned 6 times and "changelog" 5 times (backup rule, validate-backup step, file-list headings,
  changelog step, rename step);
- status is described three times (the `Issue status:` line, the "Change issue status" block with its six-state
  arrow diagram, and final-step items 2-3), plus "When it happened" / "When was it fixed" as separate headings;
- placeholder filler (`filename.sh - 2600 bytes`) and mixed heading levels (`#` and `##` for peers).

Target: scaffolding of roughly 15 lines, every fact once. The process rules move into `fixnit`'s prompt, not the file.

```
# Nit 2026-10-15-14-47-12
status: NEW                       # NEW -> WORKING -> DONE | NEEDS-YOU
filed:  2026-10-15 14:47 on <host> (<os>) from <pwd>
fixed:                            # filled by fixnit when done

## Complaint
<text, once>

## Evidence
<clipboard capture, only if grabbed>

## Question for human
<only if blocked; max 2 bullets>

## Resolution
<max 5 bullets, 5-20 words each>

## Files changed
<path  before -> after bytes; .bak path only for non-git files>
```

Also collapse the six states (NEW/REVIEW/INPROGRESS/PENDINGHUMAN/QA/DONE) to four: nobody sets REVIEW or QA, so
they are noise. One `ran:` line (mode, model, pid, start) replaces the multi-line audit stamps appended by `fixnit`.
Add a small self-check: after generating, assert the complaint text occurs exactly once.

## Done 2026-10-15: clipboard is opt-in

`savenit -c "complaint"` is now the only way the clipboard is read; the default never touches it (`-n` is accepted and
ignored for old habits). `-c` uses a fallback chain (pbpaste, wl-paste, xclip, xsel, powershell.exe) so it works on
WSL/Omarchy, and warns on stderr if the clipboard is empty or unreadable. Message is now "Attached clipboard (-c): N lines".
Tested: default run left a clipboard marker out of the nit; `-c` included it. This closes the silent-capture half of the
security risk below (a deliberate `-c` can still attach injected text to an unattended agent).
Known leftover: `docs/how-to-install-savenit.md` embeds stale full copies of the scripts (e.g. the old Y/n prompt);
replace them with a link to `bin/` when the single `nit` command lands.

## Security risk to decide on

`savenit` auto-captures the clipboard into the nit, and `fixnit` hands the nit to an agent launched with
`--dangerously-skip-permissions`. Anything copied from a web page or terminal can carry instructions the agent will
follow with full permissions (prompt injection). The zero-friction design was deliberate, so this is Hadrian's call.
Options: make clipboard capture opt-in (`-c`), or restrict the agent's tools instead of skipping permissions.

## Proposed simplified shape

- One `nit` script (~250 lines instead of ~1,080): `nit "complaint"` saves, `nit fix [-a]` runs one (or drains
  sequentially), `nit ls` shows status.
- State in `~/.local/state/savenit/{new,working,done}/NN.md` plus `.log` (already under the XDG state dir).
- Static instructions inline in the fixnit prompt; completion (rename, shelving, syntax check) done by the script
  after `wait`; backups only for non-git files.
- No locks, no `auditnit`, no changelog or backup files littering repos.

## Suggested order

1. Build `nit` in a branch; leave the current scripts untouched until it has been tried.
2. Keep a compatibility path for existing nits (the pure laptop's queue uses the old naming and instructions).
3. Then decide: opt-in clipboard, sonnet fallback, remove `convoynit`/`auditnit`.

## What was not checked

Read in full: `savenit`, `fixnit`, `checknit`, `auditnit`. Only the header and doc comments of `convoynit` were read.
Nothing was run; line counts are from `wc -l`. In-flight nits on other machines (naming, instructions baked in) are the
main migration risk.
