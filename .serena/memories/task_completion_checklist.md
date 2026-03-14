# Task Completion Checklist

Before claiming any task is complete:

1. **Verify tests pass** — run actual test commands, read output, confirm 0 failures
2. **Log in RUN_LOG.md** — every agent dispatch recorded with outcome
3. **Update PIPELINE_STATE.md** — current state, new blockers, updated next actions
4. **Update AGENT_LEDGER.md** — if agent trust level should change
5. **No direct code edits** — all coding was dispatched to agents
6. **No inline reviews** — all reviews were dispatched to reviewer agents
7. **Second verification** — dispatched a verifier agent to check the first agent's work
8. **Git state clean** — no uncommitted drift in production repos
