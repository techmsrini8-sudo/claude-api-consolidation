---
name: fix-apis
description: Runs api-fixer agent to apply all consolidation changes to source code
allowed-tools: Read, Glob, Grep, Write, Edit, Agent
---

# Fix APIs

Apply all planned consolidation changes to the actual source code.

## Prerequisites

`.claude/docs/consolidation-plan.md` must exist. If not, tell the user to run `/consolidate-apis` first.

## Safety Notice

This command **modifies source code**. Before proceeding:
1. Confirm the user has reviewed the consolidation plan
2. Recommend committing current changes first (`git add . && git commit`)
3. Ask if they want to apply all phases or a specific phase

## Instructions

1. Launch the `api-fixer` agent
2. The agent will:
   - Parse every code change from the consolidation plan
   - Apply changes one phase at a time (Phase 1 → 2 → 3 → 4)
   - Verify each change after applying
   - Mark completed items in the plan
3. Progress is tracked in the consolidation plan itself

After the agent completes, display:
- Total changes applied vs skipped vs failed
- Files modified
- Recommend running `/validate-apis` to verify fixes
- Remind about breaking changes that need API consumer notification
