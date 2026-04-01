---
name: consolidate-apis
description: Runs api-consolidation agent to produce consolidation plan
allowed-tools: Read, Glob, Grep, Write, Agent
---

# Consolidate APIs

Analyze discovered issues and produce a phased consolidation plan with exact code fixes.

## Prerequisites

Both files must exist:
- `.claude/docs/api-inventory.md` — if missing, run `/discover-apis` first
- `.claude/docs/api-issues.md` — if missing, run `/validate-apis` first

If either is missing, tell the user which command to run and stop.

## Instructions

1. Launch the `api-consolidation` agent
2. The agent will:
   - Read the inventory and issues reports
   - Design a 4-phase remediation plan (Critical → High → Medium → Low)
   - Produce exact before/after code for every fix
   - Map all breaking changes
3. Output is written to `.claude/docs/consolidation-plan.md`

After the agent completes, display:
- Number of changes per phase
- Count of breaking changes
- Files that will be modified
- Estimated effort per phase (based on change count)
