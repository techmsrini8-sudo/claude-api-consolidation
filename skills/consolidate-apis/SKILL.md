---
description: Produce a phased consolidation plan with exact code fixes for all API issues
allowed-tools: Read, Glob, Grep, Write, Agent
---

# Consolidate APIs

Analyze discovered issues and produce a phased consolidation plan with exact before/after code changes for every fix.

## Prerequisites

Both files must exist:
- `.claude/docs/api-inventory.md` — if missing, tell user to run `/api-consolidation:discover-apis` first
- `.claude/docs/api-issues.md` — if missing, tell user to run `/api-consolidation:validate-apis` first

If either is missing, tell the user which command to run and stop.

## Instructions

1. Launch the `api-consolidation:api-consolidation` agent
2. The agent will:
   - Read the inventory and issues reports completely
   - Design a 4-phase remediation plan:
     - **Phase 1:** Critical Security Fixes (missing auth, path conflicts, PCI/PII)
     - **Phase 2:** Correctness Fixes (status codes, validation, response wrapping)
     - **Phase 3:** Architecture Improvements (layer violations, DTO separation, error handling)
     - **Phase 4:** Convention Normalization (path renaming, naming consistency)
   - Produce exact before/after code blocks for every change
   - Map all breaking changes with migration notes
3. Output is written to `.claude/docs/consolidation-plan.md`

After the agent completes, display:
- Number of changes per phase
- Count of breaking changes
- Files that will be modified
- Implementation order recommendation
