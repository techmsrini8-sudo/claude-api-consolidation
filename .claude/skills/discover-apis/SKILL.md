---
description: Scan the codebase to discover all REST API endpoints and generate an inventory report
allowed-tools: Read, Glob, Grep, Write, Agent
---

# Discover APIs

Scan the entire codebase and generate a comprehensive API endpoint inventory.

## Instructions

1. Launch the `api-consolidation:api-discovery` agent to scan this project
2. The agent will:
   - Auto-detect the framework by reading build files (pom.xml, package.json, requirements.txt, go.mod)
   - Find all controller/route files using framework-specific patterns
   - Extract every endpoint with 12 metadata fields (method, path, handler, auth, request body, response type, status code, path variables, query params, validation, OpenAPI annotations)
   - Flag 9 issue types inline (PATH_CONFLICT, MISSING_AUTH, WRONG_STATUS, DUPLICATE, LAYER_VIOLATION, MISSING_VALIDATION, NAMING_ISSUE, SECURITY_RISK, PATH_AMBIGUITY)
3. Output is written to `.claude/docs/api-inventory.md`

After the agent completes, display a summary:
- Framework detected
- Total controllers found
- Total endpoints discovered
- Number of issues flagged by severity
- Top 3 most critical issues

If `.claude/docs/api-inventory.md` already exists, it will be overwritten with fresh data.

**Supported frameworks:** Spring Boot, Express.js, FastAPI, Django, Go (Gin/Mux/Echo)
