# Suggested Commands

## Pipeline State
```bash
cat ops/PIPELINE_STATE.md                    # Current state
cat ops/RUN_LOG.md                           # Agent dispatch history
cat ops/AGENT_LEDGER.md                      # Agent capabilities

## GitHub Project 11
gh project item-list 11 --owner NYQST-Group --format json --limit 100

## Issue Counts
gh issue list --repo NYQST-Group/NYQST-DocuIntelli-Build --state open --limit 500 --json number,title,labels | python3 -c "import json,sys; print(len(json.load(sys.stdin)))"
gh issue list --repo NYQST-Group/NYQST-DocuIntelli-Execution-YOLO --state open --limit 500 --json number,title,labels | python3 -c "import json,sys; print(len(json.load(sys.stdin)))"

## Open PRs
gh pr list --repo NYQST-Group/NYQST-DocuIntelli-Build --state open
gh pr list --repo NYQST-Group/NYQST-DocuIntelli-Execution-YOLO --state open

## Git Status Across Repos
for d in ~/NYQST-DocuIntelli-Build ~/NYQST-DocuIntelli-Execution ~/NYQST-DocuIntelli-Execution-YOLO; do echo "=== $(basename $d) ===" && cd $d && git fetch --all 2>&1 | grep -v "^$" && git status -sb && cd -; done

## Build Repo Tests
cd ~/NYQST-DocuIntelli-Build && .venv/bin/python -m pytest tests/unit/ -q
cd ~/NYQST-DocuIntelli-Build/ui && npm run test
cd ~/NYQST-DocuIntelli-Build/ui && npx tsc --noEmit

## Lint
cd ~/NYQST-DocuIntelli-Build && .venv/bin/python -m ruff check src/ tests/
```
