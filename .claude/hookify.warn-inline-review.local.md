---
name: warn-inline-review
enabled: true
event: bash
conditions:
  - field: command
    operator: regex_match
    pattern: git\s+diff.*\|\s*(head|tail|less|more|cat)|git\s+show|git\s+log.*-p
---

**You're reading diffs directly — are you reviewing code yourself?**

If you're about to give review feedback, STOP. Dispatch a reviewer agent instead:

- `superpowers:code-reviewer` — production readiness (probation trust level)
- `pr-review-toolkit:review-pr` — comprehensive multi-agent review (untested)
- `pr-review-toolkit:silent-failure-hunter` — error handling gaps (untested)
- `feature-dev:code-reviewer` — project convention adherence (untested)

Reading diffs to understand state is fine. Reviewing code and giving feedback is not your job.
