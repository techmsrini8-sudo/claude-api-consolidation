---
description: Run 9 automated quality checks against all discovered API endpoints
allowed-tools: Read, Glob, Grep, Write, Agent
---

# Validate APIs

Run 9 automated quality checks against every discovered API endpoint and produce a severity-rated issues report.

## Prerequisites

Check if `.claude/docs/api-inventory.md` exists. If not, run the `api-consolidation:api-discovery` agent first automatically, then proceed with validation.

## Instructions

1. Launch the `api-consolidation:api-testing` agent on this project
2. The agent runs these 9 checks:
   - **Check 1:** Path Conflict Detection (Critical) — duplicate paths, variable overlap, prefix collision
   - **Check 2:** HTTP Status Code Correctness (High) — validates against REST conventions
   - **Check 3:** Response Wrapper Consistency (Medium) — ResponseEntity usage consistency
   - **Check 4:** Input Validation Coverage (High) — @Valid and bean validation annotations
   - **Check 5:** Authentication Consistency (Critical) — sensitive endpoints require auth
   - **Check 6:** Naming Convention Consistency (Low) — plural nouns, no verbs, kebab-case
   - **Check 7:** Layer Violation Detection (Medium) — no repos in controllers
   - **Check 8:** Error Handling Completeness (Medium) — global handler, custom exceptions
   - **Check 9:** Security Vulnerability Scan (Critical) — PCI-DSS, PII, mass assignment, CORS
3. Output is written to `.claude/docs/api-issues.md`

After the agent completes, display:
- Executive summary with severity breakdown (Critical/High/Medium/Low)
- Top 5 priority fixes with effort estimates
- Most problematic controller
- Pass/fail percentage
