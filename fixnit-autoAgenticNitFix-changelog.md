# fixnit proposed changelog

- History and current status: /Users/hbaron/claude-busywork/inprogress-nit-2026-10-13-17-10-01.md
- Resolved by: log=/Users/hbaron/claude-busywork/done-nit-2026-10-13-17-10-01-agent.log finished=2026-10-13 17:25
- Health-check launch: roll back claim-rename if agent exits within 5s; strand bug fixed
- Audit stamp now written only after agent survives the 5s liveness window
- Scrub inherited ANTHROPIC_AUTH_TOKEN/ANTHROPIC_BASE_URL before spawning claude or lite claude (gateway 403 fix)
- Selection adopts orphaned inprogress-nit-*.md with no live agent when no new-nit is pending

---

# fixnit proposed changelog

- History and current status: /Users/hbaron/claude-busywork/inprogress-nit-2026-10-13-17-10-01-2.md
- Resolved by: log=/Users/hbaron/claude-busywork/done-nit-2026-10-13-17-10-01-2-agent.log finished=2026-10-13 17:15
- WORKING detection pgrep matched only 'claude.*new-nit', missing inprogress-nit agents
- fixnit renames nit to inprogress-nit before launch, so running agents were invisible
- pgrep pattern broadened to 'claude.*(new|inprogress)-nit', argv regex likewise

---

# fixnit proposed changelog

- History and current status: /Users/hbaron/claude-busywork/inprogress-nit-2026-10-13-16-52.md
- Default MODE changed from "standard" to "lite" (LiteLLM path is now the default)
- Add -sonnet/--sonnet flag to select direct claude launch; replaces "standard" label
- Remove implicit "standard" as the else-branch; rename to "sonnet" for clarity
- Invert if/else blocks so MODE==sonnet triggers direct claude, else triggers LiteLLM
- Fix all-mode recursion: pass --sonnet only when sonnet, else bare call (lite default)
