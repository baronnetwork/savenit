# savenit - proposed changelog

- History and current status: /Users/hbaron/claude-busywork/inprogress-nit-2026-10-13-16-36.md
- Always use interactive `read -r -e` on a terminal; shell rejects metachar args before savenit runs
- Split heredoc into quoted static sections (`<< 'MARKER'`) to prevent content expansion
- Use `printf '%s\n'` for all variable content (COMPLAINT, AGENT_INSTRUCTIONS) to handle any text
- Remove unquoted heredoc that expanded backticks/`$(...)` in user-supplied complaint text
