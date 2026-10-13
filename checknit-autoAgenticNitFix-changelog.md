# checknit autoAgenticNitFix proposed changelog

- History and current status: /Users/hbaron/claude-busywork/new-nit-2026-10-13-16-16.md
- Directory listing at bottom of checknit output showed all files including done-nit-* entries
- Fix: pipe ls output through grep -v to exclude done-nit- filenames from the display
- Add a parenthetical count of hidden done-nits so user knows they exist but aren't cluttering view
- No behavioral change to Working/Backlog sections; only the ls listing at the bottom is affected
