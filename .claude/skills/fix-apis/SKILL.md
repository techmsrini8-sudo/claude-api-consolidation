---
description: Apply all planned consolidation changes to the actual source code
allowed-tools: Read, Glob, Grep, Write, Edit, Agent
---

# Fix APIs

Apply all planned consolidation changes to the actual source code files.

## Prerequisites

`.claude/docs/consolidation-plan.md` must exist. If not, tell the user to run `/api-consolidation:consolidate-apis` first.

## Safety Notice

This command **modifies source code**. Before proceeding:
1. Confirm the user has reviewed the consolidation plan in `.claude/docs/consolidation-plan.md`
2. Recommend committing current changes first (git add and git commit)
3. Ask if they want to apply all phases or a specific phase (1, 2, 3, or 4)

## Instructions

1. Launch the `api-consolidation:api-fixer` agent
2. The agent will:
   - Parse every code change from the consolidation plan into a work queue
   - Apply changes one phase at a time (Phase 1 → 2 → 3 → 4)
   - For each change: read file → locate code → apply edit → verify result
   - Handle edge cases (code drift, new files, dependent changes)
   - Mark completed items as `[x]` in the consolidation plan
   - Track and report progress
3. Source code files are modified in place

After the agent completes, display:
- Total changes: applied / skipped / failed
- Files modified and new files created
- Recommend running `/api-consolidation:validate-apis` to verify fixes
- Remind about breaking changes that need API consumer notification
