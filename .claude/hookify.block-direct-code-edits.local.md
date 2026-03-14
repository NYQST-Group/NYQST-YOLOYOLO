---
name: block-direct-code-edits
enabled: true
event: file
action: block
conditions:
  - field: file_path
    operator: regex_match
    pattern: NYQST-DocuIntelli-(Build|Execution|Execution-YOLO)/src/|NYQST-DocuIntelli-(Build|Execution|Execution-YOLO)/ui/src/
---

**BLOCKED: Direct code edit to production repo.**

You are a build pipeline manager. You do NOT write production code directly.

**What to do instead:**
1. Dispatch a coding agent: `Agent` tool with `subagent_type: "general-purpose"`
2. Or dispatch Codex CLI: `codex --model o4-mini --approval-mode full-auto "task"`
3. Log the dispatch in `ops/RUN_LOG.md`
4. Verify the agent's output with a reviewer agent

The only files you may edit directly are:
- `ops/` tracking files
- `CLAUDE.md` files
- Test data fixes (small, < 5 lines)
- Git/CI configuration
