---
name: discover-apis
description: Runs api-discovery agent to scan all endpoints
allowed-tools: Read, Glob, Grep, Write, Agent
---

# Discover APIs

Scan the entire codebase and generate a comprehensive API endpoint inventory.

## Instructions

1. Launch the `api-discovery` agent to scan this project
2. The agent will:
   - Auto-detect the framework (Spring Boot, Express, FastAPI, etc.)
   - Find all controller/route files
   - Extract every endpoint with full metadata
   - Flag issues (path conflicts, missing auth, naming issues)
3. Output is written to `.claude/docs/api-inventory.md`

After the agent completes, display a summary:
- Total controllers found
- Total endpoints discovered
- Number of issues flagged
- Top 3 most critical issues

If `.claude/docs/api-inventory.md` already exists, note that it will be overwritten with fresh data.
