---
name: warn-missing-run-log
enabled: true
event: stop
pattern: .*
action: warn
---

**Pre-completion checklist — verify before stopping:**

1. Did you dispatch any agents this session? If yes:
   - [ ] Each dispatch is logged in `ops/RUN_LOG.md`
   - [ ] Each agent's outcome is recorded
   - [ ] `ops/AGENT_LEDGER.md` trust levels updated if needed

2. Did the pipeline state change? If yes:
   - [ ] `ops/PIPELINE_STATE.md` is updated
   - [ ] Next actions are clearly listed

3. Did you write any code directly? If yes:
   - **That's a violation.** Note it in the run log and dispatch agents next time.

4. Did you review code yourself instead of dispatching a reviewer agent?
   - **That's a violation.** Use `superpowers:code-reviewer` or `pr-review-toolkit:review-pr`.
