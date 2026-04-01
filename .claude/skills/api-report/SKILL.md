---
description: Run the full API consolidation pipeline — discover, validate, and plan in one command
allowed-tools: Read, Glob, Grep, Write, Agent
---

# Full API Report

Master orchestrator that runs the complete API consolidation pipeline: discover → validate → plan.

## Pipeline

Execute these three stages **sequentially** (each depends on the previous):

### Stage 1: Discovery
- Launch the `api-consolidation:api-discovery` agent
- Wait for `.claude/docs/api-inventory.md` to be generated
- Display: "{X} endpoints found across {Y} controllers"

### Stage 2: Validation
- Launch the `api-consolidation:api-testing` agent
- Wait for `.claude/docs/api-issues.md` to be generated
- Display: "{X} issues found ({critical} critical, {high} high, {medium} medium, {low} low)"

### Stage 3: Consolidation Planning
- Launch the `api-consolidation:api-consolidation` agent
- Wait for `.claude/docs/consolidation-plan.md` to be generated
- Display: "{X} changes planned across {Y} phases"

## Final Report

After all 3 stages complete, display a comprehensive summary:

```
===================================================
        API CONSOLIDATION REPORT
===================================================

Discovery:  {X} endpoints across {Y} controllers
Validation: {X} issues ({critical} critical)
Plan:       {X} changes across 4 phases

Top 3 Recommendations:
1. {highest priority fix}
2. {second priority fix}
3. {third priority fix}

Generated Files:
  .claude/docs/api-inventory.md
  .claude/docs/api-issues.md
  .claude/docs/consolidation-plan.md

Next Step: Run /api-consolidation:fix-apis to apply the plan
===================================================
```

**Note**: This does NOT apply any code changes. Run `/api-consolidation:fix-apis` separately after reviewing the plan.
