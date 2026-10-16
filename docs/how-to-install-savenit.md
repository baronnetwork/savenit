# How to Install savenit + fixnit

A two-command nit pipeline. `savenit` captures an annoyance (your complaint plus
whatever is on the clipboard) into a timestamped nit file. `fixnit` fires a
background Sonnet agent with `--dangerously-skip-permissions` at the newest
unfinished nit, one per run. You save the nit and move on; the agent does the fixing.

```mermaid
flowchart LR
    A[Something annoys you] -->|Cmd+C the output| B[savenit "complaint"]
    B -->|writes nit file| C[~/.local/state/savenit/new-nit-DATE-TIME.md]
    C -->|you run| D[fixnit]
    D -->|background Sonnet agent| E[Agent fixes it, logs, marks DONE]
    E --> F[done-nit-DATE-TIME.md]
```

## File paths

| What | Path |
|---|---|
| Scripts | In the repo's `bin/`, e.g. `~/consulting/savenit/bin/savenit` (also `fixnit`, `checknit`, `convoynit`, `auditnit`) |
| Nit template (reference) | `~/.local/state/savenit/new-nit-template.txt` |
| Nit working directory | `~/.local/state/savenit/` |
| New nits | `~/.local/state/savenit/new-nit-YYYY-MM-DD-HH-MM-SS.md` (collision suffix `-N` if the same second matches) |
| In-progress nits | `~/.local/state/savenit/inprogress-nit-YYYY-MM-DD-HH-MM-SS.md` |
| Finished nits | `~/.local/state/savenit/done-nit-YYYY-MM-DD-HH-MM-SS.md` (checknit shelves these into `done-nits/`) |
| Agent logs | `~/.local/state/savenit/inprogress-nit-<timestamp>-agent.log`, renamed `done-nit-...` on completion |

## Prerequisites

1. macOS (uses `pbpaste` and `sw_vers`).
2. The `claude` CLI installed and authenticated.
3. The repo's `bin/` directory on your `$PATH`, exported where each shell family actually
   looks. Check with `echo $PATH | tr ':' '\n' | grep savenit`.

   **zsh: `~/.zshenv` is the canonical file.** It is sourced by every zsh:
   interactive shells, `zsh -c` scripts, cron jobs, agent subprocesses. Do NOT
   also add the export to `~/.zshrc` or `~/.zprofile` — interactive shells
   inherit from `.zshenv` anyway, and duplicate PATH exports are drift seeds
   (this is where every "works in my terminal but not in scripts" bug is born).

   **bash: `~/.bashrc`, chained from the login file.** macOS Terminal opens login
   shells that read `~/.bash_profile` (or `~/.profile`) and skip `.bashrc`
   entirely, so put the export in `.bashrc` and one `source ~/.bashrc` line in
   the login file.

   **GUI apps are a separate universe.** Apps launched from Dock/Spotlight (e.g.
   Claude desktop) read no rc files; their environment comes from launchd. Only
   `launchctl setenv PATH ...` reaches them, and it dies at logout, so treat it
   as optional.

   Idempotent export line for both shells:

   ```bash
   if [[ ":$PATH:" != *":$HOME/consulting/savenit/bin:"* ]]; then export PATH="$HOME/consulting/savenit/bin:$PATH"; fi
   ```

## Install

```bash
git clone git@github-baron:baronnetwork/savenit.git ~/consulting/savenit   # github-baron = baronnetwork SSH alias (see spicebaron-git skill)
mkdir -p ~/.local/state/savenit
```

The scripts ARE the repo files; PATH points at the checkout's `bin/` directly, so there
are no symlinks or copies to keep in sync. The full script snapshots inline in
sections 1/2/4 below are historical reference — the repo files are authoritative.

With the `$PATH` entry from the Prerequisites, the scripts work everywhere:
interactive shells, scripts, subshells, agent subprocesses.

## 1. ~/bin/savenit

```bash
#!/usr/bin/env bash
set -euo pipefail

if [[ $# -eq 0 ]]; then
    # Interactive prompt mode -- avoids shell parsing of parens/special chars
    # when the user types: savenit (some thing in parens) without quotes
    if [[ -t 0 ]]; then
        printf 'What is annoying you? '
        read -r -e COMPLAINT
        if [[ -z "$COMPLAINT" ]]; then
            echo "Usage: savenit \"what is annoying you\"" >&2
            echo "  or:  savenit   (then type at the prompt)" >&2
            exit 1
        fi
    else
        echo "Usage: savenit \"what is annoying you\"" >&2
        echo "  or:  savenit   (then type at the prompt)" >&2
        exit 1
    fi
else
    COMPLAINT="$*"
fi
DATE=$(date '+%Y-%m-%d')
TIME=$(date '+%H:%M')
TIMESTAMP="${DATE}-$(date '+%H-%M')"
HOST=$(hostname)
OS_TYPE=$(sw_vers -productVersion 2>/dev/null || uname -rs)
CAPTURED=""
if [[ -t 0 ]]; then
    printf 'Grab clipboard? [Y/n] '
    read -r -n 1 REPLY
    printf '\n'
    if [[ ! "$REPLY" =~ ^[nN]$ ]]; then
        CAPTURED=$(pbpaste 2>/dev/null || true)
    fi
else
    # Non-interactive stdin: keep original copy-then-command behavior
    CAPTURED=$(pbpaste 2>/dev/null || true)
fi
OUTFILE="${XDG_STATE_HOME:-$HOME/.local/state}/savenit/new-nit-${TIMESTAMP}.md"

AGENT_INSTRUCTIONS="${COMPLAINT}"
if [[ -n "$CAPTURED" ]]; then
    AGENT_INSTRUCTIONS="${COMPLAINT}

Captured output for context:
${CAPTURED}"
fi

cat > "$OUTFILE" << TEMPLATE
# NitFile - for fixing the pesky things while bored

# Human complaint:

${COMPLAINT}

## Issue status: NEW

## When it happened: ${DATE} at ${TIME} on ${HOST} (${OS_TYPE})

## Where it happened (invoked from): ${PWD}

## When was it fixed:

# Human prompt for the AI agent on what action to take:

${AGENT_INSTRUCTIONS}

# AI agent question back to the human

Maximum two questions, pick the most important 1 or 2 questions needed to unblock.  Small issues, I leave it to the AI to decide what is best.

(In bullets, no more than 2 bullets, no more than 5-20 words per bullet)

# AI resolution summary

(In bullets, no more than 5 bullets, no more than 5-20 words per bullet)

# List of files to be edited and their backups, and sizes

filename.sh - 2600 bytes
filename-autoAgenticNitFix.sh - 2600 bytes

# List of files that were edited and their new sizes, and their backups and their sizes, and the changelog of what was changed

filename.sh - 3821 bytes
filename-autoAgenticNitFix-backup.sh - 2600 bytes
filename-autoAgenticNitFix-changelog.md


# Steps for the AI agent to take after addressing the prompt

## Change issue status
status states allowed are:
NEW -> REVIEW -> INPROGRESS -> PENDINGHUMAN -> DONE
                            -> QA -> DONE

if the AI can self validate the issue is resolved via some minimal QA, it can move to DONE

# Make a plan and resolve the issue

## If human questions are needed, AI agent is to ask advisor if available to see if it can make an informed decision on how to proceed, assuming it is safe

## Any changes to existing shell scripts, or configuration files, are to be backed up first

backup name:

filename.sh
copied to filename-autoAgenticNitFix-Backup.sh

## Validate the backup exists and matches file size

## Document the list of files to be edited and their backup in this file

## Create a filename-autoAgenticNitFix-proposed-changelog.md which has no more than 5 bullets of 5-20 words each about what is to be changed and why.  First bullet says "History and current status: <absolute path to nitfile>"

## Implement the changes

## After implementing the change, spawn a sonnet subagent to test the change or validate the fix to the best of the ability
update the nitFile with the # AI resolution summary

## After implementing the change, spawn a sonnet subagent to update the proposed_changelog.md as needed if something changed in the plan, and update the nitFile with the # AI resolution summary

## After the issue is fully resolved and tested

spawn a subagent to:
1/ Rename filename-autoAgenticNitFix-proposed-changelog.md to filename-autoAgenticNitFix-changelog.md
2/ Update the NitFile status field issueStatus to DONE
3/ Update the Nitfile status field whenFixed to ${DATE} at ${TIME} by <model>
4/ Rename the Nitfile from new-nit-${TIMESTAMP}.md to done-nit-${TIMESTAMP}.md
TEMPLATE

LINE_COUNT=$(echo "$CAPTURED" | wc -l | tr -d ' ')
if [[ -n "$CAPTURED" ]]; then
    echo "Validated pbcopy got your clipboard: ${LINE_COUNT} lines"
fi
echo "Saved your nit.  Fix it with fixnit."
```

## 2. ~/bin/fixnit

```bash
#!/usr/bin/env bash
set -euo pipefail

NITDIR="${XDG_STATE_HOME:-$HOME/.local/state}/savenit"
MODE="standard"
[[ "${1:-}" == "-lite" || "${1:-}" == "--lite" ]] && MODE="lite"
if [[ "$MODE" == "lite" ]]; then
    AGENT_DESC="Kimi-K3-lab-claude-compatible[1m] / effort high / --bare / via LiteLLM"
else
    AGENT_DESC="claude-sonnet-4-6 / direct"
fi

# Nits currently assigned to a running agent (same trick checknit uses)
WORKING=$(pgrep -fl 'claude.*new-nit' 2>/dev/null | grep -oE "$NITDIR/new-nit-[0-9-]+\.md" || true)

# Find most recent new-nit-*.md not internally marked DONE and not already being worked
NITFILE=""
while IFS= read -r f; do
    grep -q "Issue status: DONE" "$f" 2>/dev/null && continue
    [[ -n "$WORKING" ]] && grep -qF "$f" <<< "$WORKING" && continue
    NITFILE="$f"
    break
done < <(ls -r "$NITDIR"/new-nit-*.md 2>/dev/null)

if [[ -z "$NITFILE" ]]; then
    echo "No pending nit files. Either you fixed everything or you forgot to complain."
    exit 0
fi

LOGFILE="${NITFILE%.md}-agent.log"
PROMPT="Read the nit file at ${NITFILE} and implement all agent instructions in it autonomously. The nit file is your complete specification. The 'Where it happened' directory marks where the complaint was filed, not necessarily where the fix belongs; follow the evidence."

# Ground the agent in the directory the complaint was filed from; old nits or
# vanished directories fall back to the nit directory
WORKDIR=$(grep -m1 '## Where it happened' "$NITFILE" | sed 's/.*: *//' || true)
[[ -d "${WORKDIR:-}" ]] || WORKDIR="$NITDIR"

echo "Mode:  $MODE"
echo "Nit:   $NITFILE"
echo "Log:   $LOGFILE"
echo "From:  $WORKDIR"
echo "Agent: starting..."

cd "$WORKDIR"

if [[ "$MODE" == "lite" ]]; then
    # Same launch shape as css-v3.sh -lite: LiteLLM proxy, Kimi K3, effort high.
    if ! command -v lite >/dev/null 2>&1; then
        echo "Missing 'lite' CLI. Install: pipx install 'litellm[proxy]', then: lite login" >&2
        exit 1
    fi
    if lite auth print-token 2>&1 | grep -qiE 'expired|not logged|run.*login|login.*again|no.*token|unauthorized'; then
        echo "LiteLLM token expired. Run: lite login" >&2
        exit 1
    fi
    lite claude --bare --dangerously-skip-permissions \
        --model "Kimi-K3-lab-claude-compatible[1m]" --effort high \
        --settings '{"model":"Kimi-K3-lab-claude-compatible[1m]"}' \
        -p "$PROMPT" \
        >> "$LOGFILE" 2>&1 &
else
    claude --model claude-sonnet-4-6 --dangerously-skip-permissions \
        -p "$PROMPT" \
        >> "$LOGFILE" 2>&1 &
fi

echo "PID:   $!"

# fixnit writes the audit stamp itself; agent self-reports are decorative
printf '\n## fixnit audit stamp: mode=%s agent=%s pid=%s started=%s\n' \
    "$MODE" "$AGENT_DESC" "$!" "$(date '+%Y-%m-%d %H:%M')" >> "$NITFILE"
```

The `-lite` mode mirrors the `css-v3.sh -lite` launch path: `lite claude` routes
through the LiteLLM proxy to `Kimi-K3-lab-claude-compatible[1m]` at effort `high`,
with a `--settings` JSON override so a stale model entry in `~/.claude/settings.json`
cannot win. It also passes `--bare`: no hooks, plugins, CLAUDE.md, or auto-memory
load for nit agents. Auth survives because the proxy injects credentials via env,
not the keychain (bare never reads OAuth/keychain). Requires the `lite` CLI
(`pipx install 'litellm[proxy]'`) and a live token (`lite login` when
`lite auth print-token` says otherwise).

## 3. ~/.local/state/savenit/new-nit-template.txt

The reference template the nit file format is based on. `savenit` embeds this
structure in its heredoc, so this file is documentation and a design reference,
not a runtime requirement.

````text
# NitFile - for fixing the pesky things while bored

# Human complaint:
```
$1 (first parameter)
```

## Issue status: NEW

## When it happened: $date and $time on $hostname ($osType)

## When was it fixed: $date and $time by $model

# Human prompt for the AI agent on what action to take:
```
$2 (second parameter) 
```

# AI agent question back to the human

Maximum two questions, pick the most important 1 or 2 questions needed to unblock.  Small issues, I leave it to the AI to decide what is best.

(In bullets, no more than 2 bullets, no more than 5-20 words per bullet)

# AI resolution summary

(In bullets, no more than 5 bullets, no more than 5-20 words per bullet)

# List of files to be edited and their backups, and sizes

```
# example
filename.sh - 2600 bytes
filename-autoAgenticNitFix.sh - 2600 bytes
```

# List of files that were edited and their new sizes, and their backups and their sizes, and the changelog of what was changed
```
# example
filename.sh - 3821 bytes
filename-autoAgenticNitFix-backup.sh - 2600 bytes
filename-autoAgenticNitFix-changelog.md
```


# Steps for the AI agent to take after addressing the prompt

## Change issue status
status states allowed are:
NEW -> REVIEW -> INPROGRESS -> PENDINGHUMAN -> DONE
                            -> QA -> DONE

if the AI can self validate the issue is resolved via some minimal QA, it can move to DONE

# Make a plan and resolve the issue

## If human questions are needed, AI agent is to ask advisor if available to see if it can make an informed decision on how to proceed, assuming it is safe

## Any changes to existing shell scripts, or configuration files, are to be backed up first

backup name:

filename.sh
copied to filename-autoAgenticNitFix-Backup.sh

## Validate the backup exists and matches file size

## Document the list of files to be edited and their backup in this file

## Create a filename-autoAgenticNitFix-proposed-changelog.md which has no more than 5 bullets of 5-20 words each about what is to be changed and why.  First bullet says "History and current status: <absolute path to nitfile>"

## Implement the changes

## After implementing the change, spawn a sonnet subagent to test the change or validate the fix to the best of the ability
update the nitFile with the # AI resolution summary

## After implementing the change, spawn a sonnet subagent to update the proposed_changelog.md as needed if something changed in the plan, and update the nitFile with the # AI resolution summary

## After the issue is fully resolved and tested

spawn a subagent to:
1/ Rename filename-autoAgenticNitFix-proposed-changelog.md to filename-autoAgenticNitFix-changelog.md
2/ Update the NitFile status field issueStatus to DONE
3/ Update the Nitfile status field whenFixed to `$date and $time by $model`
4/ Rename the Nitfile from new-nitFile.txt to done-nitFile.txt
````

## 4. ~/bin/checknit

Shows which fixnit agents are running against which nits, and what is waiting in
the queue. No state files: the process table is the source of truth.

```bash
#!/usr/bin/env bash
set -euo pipefail

NITDIR="${XDG_STATE_HOME:-$HOME/.local/state}/savenit"

etime_human() {
    # ps etime formats: mm:ss, hh:mm:ss, dd-hh:mm:ss
    local e="$1" d=0 h=0 m=0 a b c
    if [[ "$e" == *-* ]]; then
        d="${e%%-*}"
        e="${e#*-}"
    fi
    IFS=: read -r a b c <<< "$e"
    if [[ -n "${c:-}" ]]; then
        h="$a"; m="$b"
    else
        m="$a"
    fi
    d=$((10#${d:-0})); h=$((10#${h:-0})); m=$((10#${m:-0}))
    if (( d > 0 )); then echo "${d}d ${h}h"
    elif (( h > 0 )); then echo "${h}h ${m}m"
    else echo "${m} min"; fi
}

etime_seconds() {
    # prefer ps etimes; derive from etime string if the platform lacks it
    local pid="$1" secs e d=0 h=0 m=0 s=0 a b c
    secs=$(ps -o etimes= -p "$pid" 2>/dev/null | tr -d ' ')
    if [[ -n "$secs" ]]; then echo "$secs"; return; fi
    e=$(ps -o etime= -p "$pid" 2>/dev/null | tr -d ' ')
    [[ "$e" == *-* ]] && { d="${e%%-*}"; e="${e#*-}"; }
    IFS=: read -r a b c <<< "$e"
    if [[ -n "${c:-}" ]]; then h="$a"; m="$b"; s="$c"
    else m="$a"; s="$b"; fi
    echo $(( 10#${d:-0}*86400 + 10#${h:-0}*3600 + 10#${m:-0}*60 + 10#${s:-0} ))
}

# Working: claude processes with a nit path in argv
# S_PIDS: agents past 5 minutes, offered for killing at the end
declare -a W_LINES=()
declare -a W_NITS=()
declare -a S_PIDS=()
while IFS= read -r line; do
    pid="${line%% *}"
    nit=$(grep -oE "$NITDIR/new-nit-[0-9-]+\.md" <<< "${line#* }" | head -1 || true)
    [[ -z "$nit" ]] && continue
    etime=$(ps -o etime= -p "$pid" 2>/dev/null | tr -d ' ')
    esecs=$(etime_seconds "$pid")
    (( esecs > 300 )) && S_PIDS+=("$pid")
    log="${nit%.md}-agent.log"
    W_LINES+=("PID $pid  running $(etime_human "${etime:-00:00}")  $nit")
    W_LINES+=("    log:   $log")
    if (( esecs > 600 )); then
        W_LINES+=("    STALE: over 10 min, last 5 log lines:")
        while IFS= read -r tl; do
            W_LINES+=("      | $tl")
        done < <(tail -5 "$log" 2>/dev/null || true)
    fi
    W_LINES+=("")
    W_NITS+=("$nit")
done < <(pgrep -fl 'claude.*new-nit' 2>/dev/null || true)

# Backlog: non-DONE nits nobody is working
declare -a B_LINES=()
while IFS= read -r f; do
    grep -q "Issue status: DONE" "$f" 2>/dev/null && continue
    skip=0
    for w in ${W_NITS[@]+"${W_NITS[@]}"}; do
        [[ "$w" == "$f" ]] && { skip=1; break; }
    done
    (( skip )) && continue
    base=$(basename "$f")
    stamp=${base#new-nit-}; stamp=${stamp%.md}
    B_LINES+=("${stamp:0:10} ${stamp:11:2}:${stamp:14:2}  $f")
done < <(ls -r "$NITDIR"/new-nit-*.md 2>/dev/null || true)

echo
echo "# Working (${#W_NITS[@]})"
echo
for l in ${W_LINES[@]+"${W_LINES[@]}"}; do echo "$l"; done
echo "# Backlog (${#B_LINES[@]})"
echo
for l in ${B_LINES[@]+"${B_LINES[@]}"}; do echo "$l"; done
echo

if (( ${#W_NITS[@]} > 0 || ${#B_LINES[@]} > 0 )); then
    echo "Interrupt an agent:  kill <PID>"
    echo "Work the next nit:   fixnit"
else
    echo "Nothing pending. Either you're out of complaints or you've gone numb."
fi

echo
echo "# Directory ($NITDIR)"
echo
ls -lhA "$NITDIR"

# Offer clemency hearing for agents past 5 minutes (interactive only)
if [[ -t 0 && ${#S_PIDS[@]} -gt 0 ]]; then
    echo
    printf 'Do you want me to kill the processes older than 5 minutes?  Y/N '
    KREPLY=""
    read -t 10 -r -n 1 KREPLY || true
    if [[ "$KREPLY" =~ ^[yY]$ ]]; then
        printf '\n'
        for p in "${S_PIDS[@]}"; do
            kill "$p" 2>/dev/null && echo "Killed PID $p" || echo "PID $p already gone"
        done
    else
        printf '\n\n%s\n' "You are too kind sir, to give us this chance."
    fi
fi
```

## Commands

### savenit

```bash
# 1. Copy the offending output first (Cmd+C), then:
savenit "zshrc alias is not loading on new terminals"

# Instruction-only nit (clipboard is never read unless -c):
savenit "add a cust alias for customer-projects"

# Bare invocation asks once for the complaint (the metachar-safe path):
savenit
```

Zero prompts when args are given. The clipboard is read only with `-c` / `--clip` (via
`pbpaste`, `wl-paste`, `xclip`, `xsel` or `powershell.exe`; `-n` is accepted and ignored). Output is at most one line —
`Attached clipboard (-c): N lines`, only when the clipboard had
content. Fire-and-forget by design: save the nit, move on.

Behavior notes:

- Filenames carry seconds (`new-nit-YYYY-MM-DD-HH-MM-SS.md`) plus a `-N`
   collision suffix when two saves land in the same second, so a new nit never
   silently overwrites an older one.
- The bare-interactive prompt exists because unquoted complaints with shell
   metacharacters (', `, <, >, \) never survive the shell long enough to become
   arguments; typing at a prompt sidesteps that entire failure class.
- The file is never opened. Save the nit, move on, let the agent earn its keep.
- Issue status starts as `NEW`; "When was it fixed" stays blank for the agent.
- Clipboard content lands in the agent-instructions block as captured context, so
  the agent gets the complaint and the raw evidence in one place.
- With no argument, savenit asks `What is annoying you?` interactively, sidestepping
  shell parsing of unquoted parens and other special characters.
- Every nit records `Where it happened (invoked from): <PWD>` so the agent knows
  which project directory the complaint came from.

### fixnit

```bash
fixnit      # fires one background agent at the newest unfinished nit
fixnit      # run again for the next one
```

Output:

```text
Mode:  standard
Nit:   /Users/you/.local/state/savenit/inprogress-nit-2026-10-13-14-47-12.md
Log:   /Users/you/.local/state/savenit/inprogress-nit-2026-10-13-14-47-12-agent.log
From:  /Users/you/some-project
Agent: starting...
PID:   84231
```

If nothing is pending:

```text
No pending nit files. Either you fixed everything or you forgot to complain.
```

Selection logic:

1. Lists `new-nit-*.md` in reverse name order (filename sort equals time sort, newest first).
2. Skips any file whose contents contain `Issue status: DONE` (handles the case
   where the agent finished but had not renamed the file yet).
3. Skips any nit already assigned to a running agent (`pgrep` over claude argv,
   same detection as checknit), so rerunning fixnit mid-flight claims the NEXT nit
   instead of doubling the current one.
4. Takes the first survivor, one per run.
5. After launch, fixnit appends an authoritative audit stamp to the nit file
   (`## fixnit audit stamp: mode=... agent=... pid=... started=...`). Trust this
   line over the template's self-reported `by <model>` field: agents fill that in
   from self-image, and a Kimi instance once stamped itself "Claude Opus 4.8".
6. The agent launches from the nit's stamped PWD when it still exists, else from
   the busywork directory. This is grounding, not confinement: file access is
   global because `--dangerously-skip-permissions` bypasses outside-workspace
   checks (proven by nit 15-34 rewriting `~/.zshenv`). If you ever want real
   confinement, drop the skip flag and gate on permission modes instead.

### checknit

```bash
checknit
```

Output, live example:

```text
# Working (1)

PID 50892  running 2 min  /Users/you/.local/state/savenit/new-nit-2026-10-13-15-38.md
    log:   /Users/you/.local/state/savenit/new-nit-2026-10-13-15-38-agent.log

# Backlog (2)

2026-10-13 15:37  /Users/you/.local/state/savenit/new-nit-2026-10-13-15-37.md
2026-10-13 15:36  /Users/you/.local/state/savenit/new-nit-2026-10-13-15-36.md

Interrupt an agent:  kill <PID>
Work the next nit:   fixnit
```

Behavior notes:

- Working comes from `pgrep` over running `claude` processes whose argv contains a
  nit path (fixnit puts the path in its `-p` argument). Elapsed time comes from `ps`.
- Logs are listed for reference only. `claude -p` buffers output until exit, so the
  elapsed runtime is the live signal, not the log contents.
- Backlog lists non-DONE nits nobody is working on, newest first. A killed agent's
  nit falls back into the Backlog automatically.
- When nothing is pending, it says so and declines to print the kill hint.
- A working agent running over 10 minutes triggers a STALE marker plus the last 5
  lines of its log, inline under that entry. The threshold exists because `claude -p`
  logs are buffered and silent mid-run, so runtime is the only stuck-detector.
- The `# Directory` section always prints a full `ls -lhA` of the nit directory:
  template, pending nits, done nits, and agent logs in one glance.
- After all output, if any agent is past 5 minutes and stdin is a terminal, checknit
  asks `Do you want me to kill the processes older than 5 minutes?  Y/N`. `y` kills
  (SIGTERM) every such PID; `n`, Enter, or 10 seconds of silence spares them with
  "You are too kind sir, to give us this chance." Non-interactive runs never prompt.

## Safety note

`fixnit` runs with `--dangerously-skip-permissions`, which means the agent edits,
deletes, and executes without asking. Eyeball a nit file before pointing an
autonomous agent at your machine. Auto-firing the agent inside `savenit` was
deliberately rejected for this reason; revisit once the workflow earns trust.

## Uninstall

```bash
rm -rf ~/consulting/savenit          # the repo checkout holds all three scripts
# remove the PATH export from ~/.zshenv and ~/.bashrc
# keep or delete ~/.local/state/savenit as you see fit
```
