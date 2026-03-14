# Agent Dispatch Templates

## Claude Subagent (coding task)
```
Agent tool:
  subagent_type: "general-purpose"
  model: "sonnet"
  prompt: |
    You are implementing [task description].

    REQUIRED: Before writing any code, invoke the Skill tool with skill
    "superpowers:test-driven-development" and follow RED-GREEN-REFACTOR exactly.

    Working directory: cd [worktree path or repo path]
    Python venv: /Users/markforster/NYQST-DocuIntelli-Build/.venv/bin/python
    Test command: [venv path] -m pytest tests/unit/ -q

    FILE OWNERSHIP: You may ONLY modify these files:
    - [list exact files]
    Do NOT modify any other files.

    GitHub issue: #[number]
    Acceptance criteria: [from issue body]

    When done report: DONE / DONE_WITH_CONCERNS / NEEDS_CONTEXT / BLOCKED
```

## Codex CLI (coding task)
```bash
cd ~/NYQST-DocuIntelli-Build
codex exec "
You are implementing [task description]. GitHub issue #[number].

MANDATORY TDD PROTOCOL:
1. Write failing test FIRST
2. Run: .venv/bin/python -m pytest tests/unit/[file] -v
3. Confirm FAILS (show output)
4. Write MINIMAL implementation
5. Run test again, confirm PASSES
6. Commit with: git commit -m 'feat: [desc] (#[issue])'

FILE OWNERSHIP: Only modify [list files].
"
```

## Review dispatch
```
Agent tool:
  subagent_type: "superpowers:code-reviewer"
  prompt: [use requesting-code-review template with BASE_SHA, HEAD_SHA]
```

## TDD checker (haiku, cheap)
```
Agent tool:
  model: "haiku"
  prompt: |
    Read the git diff: cd [repo] && git diff [base]..[head]
    Verify: every new function has a test, tests describe behavior, no code without coverage.
    Report violations only.
```

## Worktree setup (run before dispatching coding agent)
```bash
cd ~/NYQST-DocuIntelli-Build
git worktree add /tmp/pkt-{issue}-wt -b feat/{issue}-{desc} {base_branch}
```

## Worktree cleanup (run after merge or discard)
```bash
cd ~/NYQST-DocuIntelli-Build
git worktree remove /tmp/pkt-{issue}-wt
```
