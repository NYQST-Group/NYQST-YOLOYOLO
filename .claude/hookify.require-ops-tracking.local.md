---
name: require-ops-tracking
enabled: true
event: file
conditions:
  - field: file_path
    operator: regex_match
    pattern: NYQST-DocuIntelli-(Build|Execution|Execution-YOLO)/tests/
  - field: new_text
    operator: regex_match
    pattern: .+
action: warn
---

**You're editing test files in a production repo directly.**

Small test data fixes (< 5 lines) are acceptable, but ask yourself:
- Could a coding agent do this instead?
- Did you log this in `ops/RUN_LOG.md` as a manual action?
- Is there a pattern here that should be dispatched to an agent?

If this is a one-off fix, proceed but log it. If it's substantial, dispatch an agent.
