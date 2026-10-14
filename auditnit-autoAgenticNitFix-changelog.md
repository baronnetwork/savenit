# auditnit - proposed changelog

- History and current status: /Users/hbaron/claude-busywork/inprogress-nit-2026-10-13-17-24-40.md
- Resolved by: log=/Users/hbaron/claude-busywork/done-nit-2026-10-13-17-24-40-agent.log finished=2026-10-13 at 17:45
- New script: wintermute batch auditor; fires ONE lite-mode agent over K most recent done-nits (default 10)
- Agent reads complaints, changelogs, git diffs of touched files; runs only cheap structural checks (bash -n)
- Verdicts written as NEW nits prefaced 'REGRESSION from done-nit-<stamp>:' for convoy, or PENDINGHUMAN; silent when clean
