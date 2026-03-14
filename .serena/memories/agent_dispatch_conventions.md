# Agent Dispatch Conventions

## Core Rules
1. NEVER code/review/test directly — always dispatch an agent
2. Log EVERY dispatch in ops/RUN_LOG.md
3. Verify EVERY agent output with a SECOND agent
4. Update ops/PIPELINE_STATE.md after every significant action
5. Update ops/AGENT_LEDGER.md trust levels after evaluating output

## Agent Selection
- Small coding task → Claude subagent (general-purpose, sonnet model for speed)
- Complex coding → Claude subagent (general-purpose, opus model)
- Code review → superpowers:code-reviewer (probation trust)
- PR review → pr-review-toolkit:review-pr (untested)
- Test analysis → pr-review-toolkit:pr-test-analyzer (untested)
- Error handling review → pr-review-toolkit:silent-failure-hunter (untested)
- Research → Explore subagent
- Architecture → Plan subagent or feature-dev:code-architect

## Trust Levels
- untested → probation: first successful run
- probation → trusted: 3+ successful runs
- trusted → promoted: handles complex/critical tasks
- Any → demoted: produces incorrect output

## Dispatch Template (for RUN_LOG.md)
```
### RUN-{YYYYMMDD}-{seq} | {agent_type} | {status}
- **Task:** what was dispatched
- **Target:** repo/branch/PR
- **Agent ID:** returned agent ID
- **Worktree:** if isolated
- **Duration:** approx
- **Outcome:** pass/fail/partial + summary
- **Action taken:** what we did with the result
- **Lessons:** what we learned about this agent type
```
