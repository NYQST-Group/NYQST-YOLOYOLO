# Skill Chain Reference

Loaded on demand. CLAUDE.md references this but doesn't include it.

## Stage-to-Skill Mapping

| Pipeline Stage | Required Skill | Key Rules |
|---|---|---|
| UNDERSTAND | `superpowers:brainstorming` | One question per message. 2-3 approaches. Spec review loop. HARD GATE: no code until design approved. |
| SPEC | `superpowers:writing-plans` | 2-5 min steps with exact code and file paths. Plan review loop. |
| BRANCH | `superpowers:using-git-worktrees` | Create isolated workspace before implementation. |
| IMPLEMENT | `superpowers:subagent-driven-development` | Fresh subagent per task. TWO-STAGE review: spec FIRST, then quality. |
| IMPLEMENT (per task) | `superpowers:test-driven-development` | RED→GREEN→REFACTOR. No code before failing test. Delete code written before test. |
| VERIFY | `superpowers:verification-before-completion` | Run command, read output, confirm 0 failures. "Should pass" = lying. |
| REVIEW | `superpowers:requesting-code-review` | Dispatch reviewer with BASE_SHA, HEAD_SHA. Fix Critical immediately. |
| FIX (parallel) | `superpowers:dispatching-parallel-agents` | One agent per independent problem. Focused prompt, clear constraints. |
| MERGE | `superpowers:finishing-a-development-branch` | Verify tests → 4 options → execute. Never proceed with failing tests. |

## Skill Enforcement by Agent Type

**Claude subagents (CAN load skills):**
Tell them: "Before writing code, invoke the Skill tool with skill 'superpowers:test-driven-development'"

**Codex CLI (CANNOT load skills):**
Embed the protocol directly in the prompt. Include: RED→GREEN→REFACTOR steps, iron law, exact test commands.

## Validation Chain

```
DOER (follows TDD) → TDD CHECKER (sonnet) → SPEC REVIEWER (sonnet) → QUALITY REVIEWER (sonnet) → MERGE
```

If validator fails → re-dispatch doer with feedback.
If doer fails twice → escalate to more capable model.
If capable model fails → escalate to user.
