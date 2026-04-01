---
name: validate-apis
description: Runs api-testing agent to validate endpoints against 9 checks
allowed-tools: Read, Glob, Grep, Write, Agent
---

# Validate APIs

Run 9 automated quality checks against all discovered API endpoints.

## Prerequisites

Check if `.claude/docs/api-inventory.md` exists. If not, run the `api-discovery` agent first automatically, then proceed with validation.

## Instructions

1. Launch the `api-testing` agent on this project
2. The agent runs these 9 checks:
   - Path Conflict Detection (Critical)
   - HTTP Status Code Correctness (High)
   - Response Wrapper Consistency (Medium)
   - Input Validation Coverage (High)
   - Authentication Consistency (Critical)
   - Naming Convention Consistency (Low)
   - Layer Violation Detection (Medium)
   - Error Handling Completeness (Medium)
   - Security Vulnerability Scan (Critical)
3. Output is written to `.claude/docs/api-issues.md`

After the agent completes, display:
- Executive summary (critical/high/medium/low counts)
- Top 5 priority fixes with effort estimates
- Which controllers have the most issues
