# convoynit - proposed changelog

- History and current status: /Users/you/claude-busywork/inprogress-nit-2026-10-13-17-24-40.md
- Resolved by: log=/Users/you/claude-busywork/done-nit-2026-10-13-17-24-40-agent.log finished=2026-10-13 at 17:45
- New script: convoy queue orchestrator; loops queue, fires fixnit at newest eligible nit, tracks PIDs
- Eligibility: target files from complaint parsing ('Target repo file:' hints, absolute paths) must not overlap in-flight nits; same-file order preserved
- Skips DONE/QA/PENDINGHUMAN nits (PENDINGHUMAN never fired); adopts orphaned in-progress nits with no live agent
