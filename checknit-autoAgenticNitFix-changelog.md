- History and current status: /Users/you/claude-busywork/inprogress-nit-2026-10-13-17-07-16.md
- Resolved by: log=/Users/you/claude-busywork/done-nit-2026-10-13-17-07-16-agent.log finished=2026-10-13 17:33
- Reorder checknit output: directory listing first, then the Cleanup line, then the queue/recent-fixes summary
- Cleanup (shelving done-nits) now runs quietly before printing so the listing reflects post-shelve state
- Clemency prompt stays last; detection logic and single-line Cleanup format unchanged

---

- History and current status: /Users/you/claude-busywork/inprogress-nit-2026-10-13-17-12-04.md
- Resolved by: log=/Users/you/claude-busywork/done-nit-2026-10-13-17-12-04-agent.log finished=2026-10-13 17:26
- Replace verbose Working/Backlog sections with one-line-per-nit queue table (in-flight/waiting, short stamp, complaint)
- In-flight lines show PID from ps plus mode from fixnit audit stamp (sonnet shown as standard); STALE kept inline at 10 min
- Add needs-you flag for real questions or PENDINGHUMAN status; add 3-newest done-nits recent fixes section
- Scope is display only; detection (pgrep matching, backlog glob) untouched per nit instructions
# checknit proposed changelog

- History and current status: /Users/you/claude-busywork/inprogress-nit-2026-10-13-17-10-01-2.md
- Resolved by: log=/Users/you/claude-busywork/done-nit-2026-10-13-17-10-01-2-agent.log finished=2026-10-13 17:15
- Working section pgrep matched only 'claude.*new-nit', missing inprogress-nit agents
- fixnit renames nit to inprogress-nit before launch, so running agents were invisible
- pgrep pattern broadened to 'claude.*(new|inprogress)-nit', argv regex likewise

---

# Proposed Changelog

- History and current status: /Users/you/claude-busywork/inprogress-nit-2026-10-13-16-35.md
- `checknit` directory listing showed uid/gid/modbits from `ls -l`; user wants size/date/name only
- Pipe `ls -lhA` output through `awk 'NF>=9{print $5,$6,$7,$8,$9}'` to strip perms/owner/group
- The `total NNN` header line (2 fields) is silently dropped by the `NF>=9` guard
- No behavioral change to any other section of the script; one-line fix at ls output line

---

# checknit proposed changelog

- History and current status: /Users/you/claude-busywork/inprogress-nit-2026-10-13-17-05-47.md
- Resolved by: log=/Users/you/claude-busywork/done-nit-2026-10-13-17-05-47-agent.log finished=2026-10-13 17:28
- Cleanup section echoed every shelved filename; user wants a single summary line only
- Drop per-file listing loop; print one line: "Cleanup: shelved N file(s) -> <done-nits path>"
- Keep WARNING for done-nit files lacking DONE status; an anomaly is not verbose noise
- Scope is display only; move/shelve logic (mv of nits, logs, changelogs) untouched
