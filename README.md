# API Consolidation Plugin for Claude Code

Automated REST API discovery, validation, and consolidation for any backend project. Scans your codebase, finds every endpoint, runs 9 quality checks, produces a phased fix plan, and applies code changes — all from within Claude Code.

**Version:** 1.0.0 | **License:** MIT | **Author:** [techmsrini8-sudo](https://github.com/techmsrini8-sudo)
**Repository:** https://github.com/techmsrini8-sudo/claude-api-consolidation

---

## Table of Contents

1. [Supported Frameworks](#supported-frameworks)
2. [Installation](#installation)
3. [How It Works — Architecture & Diagrams](#how-it-works--architecture--diagrams)
4. [Commands](#commands)
5. [Pipeline Overview](#pipeline-overview)
6. [Agents — Detailed Behavior](#agents--detailed-behavior)
   - [api-discovery](#api-discovery)
   - [api-testing](#api-testing)
   - [api-consolidation](#api-consolidation)
   - [api-fixer](#api-fixer)
7. [The 9 Quality Checks](#the-9-quality-checks)
8. [Generated Files](#generated-files)
9. [Hooks](#hooks)
10. [Permissions](#permissions)
11. [Plugin Structure](#plugin-structure)
12. [Customization](#customization)
13. [Troubleshooting](#troubleshooting)

---

## Supported Frameworks

Auto-detected from your build file — no configuration required.

| Framework | Detected Via | Controller/Route Files Scanned |
|-----------|-------------|-------------------------------|
| Spring Boot (Java) | `pom.xml` containing `spring-boot-starter-web` | `**/*Controller.java`, `**/*Resource.java`, `**/*Endpoint.java` |
| Spring Boot (Gradle) | `build.gradle` containing `spring-boot` | Same as above |
| Express.js (Node) | `package.json` containing `express` | `**/routes/**/*.js`, `**/router/**/*.ts` |
| FastAPI (Python) | `requirements.txt` / `pyproject.toml` containing `fastapi` | `**/routers/**/*.py`, `**/*router*.py` |
| Django (Python) | `requirements.txt` / `pyproject.toml` containing `django` | `**/urls.py`, `**/views.py` |
| Go (Gin/Mux/Echo) | `go.mod` containing `gin`, `mux`, or `echo` | `**/*.go` |

---

## Installation

### From GitHub (recommended)

```bash
# Step 1: Add the marketplace (one-time setup)
/plugin marketplace add techmsrini8-sudo/claude-api-consolidation

# Step 2: Install the plugin
/plugin install api-consolidation@api-consolidation-marketplace
```

### From local directory (development / testing)

```bash
claude --plugin-dir ./api-consolidation-plugin
```

---

## How It Works — Architecture & Diagrams

### Diagram 1: Overall Architecture

Three layers sit between what you type and your fixed source code.

```
╔══════════════════════════════════════════════════════════════════════╗
║                          USER INTERFACE                              ║
║                                                                      ║
║   Explicit slash commands          Auto-triggered via Skills         ║
║   ──────────────────────           ─────────────────────────         ║
║   /api-consolidation:              "scan my APIs"                    ║
║     discover-apis                  "validate endpoints"              ║
║     validate-apis                  "consolidate APIs"                ║
║     consolidate-apis               "apply fixes"                     ║
║     fix-apis                       "run full audit"                  ║
║     api-report                                                       ║
╚══════════════════════════════════╦═══════════════════════════════════╝
                                   ║ invokes
                                   ▼
╔══════════════════════════════════════════════════════════════════════╗
║                         COMMANDS LAYER                               ║
║                                                                      ║
║  ┌───────────────┐  ┌──────────────┐  ┌────────────────────────┐    ║
║  │ discover-apis │  │ validate-apis│  │    consolidate-apis     │    ║
║  └───────┬───────┘  └──────┬───────┘  └────────────┬───────────┘    ║
║          └─────────────────┴──────────────────────┐ │               ║
║                  api-report (orchestrator) ◄───────┘ │               ║
║                                            ┌─────────┘               ║
║                                            │  fix-apis               ║
╚════════════════════════════════════════════╬════════════════════════╝
                                             ║ launches
                                             ▼
╔══════════════════════════════════════════════════════════════════════╗
║                          AGENTS LAYER                                ║
║                                                                      ║
║  ┌───────────────┐  ┌─────────────┐  ┌──────────────────────────┐   ║
║  │ api-discovery │  │ api-testing │  │   api-consolidation      │   ║
║  │  (read-only)  │  │ (read-only) │  │      (read-only)         │   ║
║  └───────┬───────┘  └──────┬──────┘  └────────────┬─────────────┘   ║
║          │                 │                       │                 ║
║          └─────────────────┴───────────────────────┘                 ║
║                        api-fixer                                     ║
║                  (only agent that writes source code)                ║
╚══════════════════════════════════╦═══════════════════════════════════╝
                                   ║ produces
                                   ▼
╔══════════════════════════════════════════════════════════════════════╗
║                             OUTPUT                                   ║
║                                                                      ║
║   .claude/docs/  (audit trail)        Your source files              ║
║   ┌─────────────────────────┐         ┌──────────────────────────┐   ║
║   │ api-inventory.md        │         │ *Controller.java  ──────► │   ║
║   │ api-issues.md           │         │ *router.ts        fixed   │   ║
║   │ consolidation-plan.md   │         │ New DTOs created          │   ║
║   └─────────────────────────┘         └──────────────────────────┘   ║
╚══════════════════════════════════════════════════════════════════════╝
```

---

### Diagram 2: Pipeline Data Flow

Each stage reads the output of the previous stage. The pipeline can only flow forward.

```
  YOUR CODEBASE                 STAGE 1                .claude/docs/
  ─────────────                 ───────                ─────────────
                             ┌─────────────┐
  pom.xml                    │             │
  *Controller.java  ────────►│ api-        │──────►  api-inventory.md
  *router.ts        (scanned) │ discovery  │          ┌──────────────────────┐
  views.py                   │ agent       │          │ 45 endpoints         │
  *.go                       │             │          │ 12 fields each       │
                             └─────────────┘          │ 9 inline issue flags │
                                                      └──────────┬───────────┘
                                                                 │
                                  STAGE 2                        │
                                  ───────                        ▼
                             ┌─────────────┐
  *Controller.java  ────────►│             │
  (re-read to       (verify) │ api-testing │──────►  api-issues.md
  verify findings)           │ agent       │          ┌──────────────────────┐
                             │             │          │ 52 findings          │
                             └─────────────┘          │ grouped by 9 checks  │
                                                      │ + by controller      │
                                                      └──────────┬───────────┘
                                                                 │
                                  STAGE 3                        │
                                  ───────                        ▼
                             ┌─────────────────┐
                             │                 │
  (reads both docs) ────────►│ api-            │──────►  consolidation-plan.md
                             │ consolidation   │          ┌──────────────────────┐
                             │ agent           │          │ 4 phases             │
                             │                 │          │ exact before/after   │
                             └─────────────────┘          │ code per change      │
                                                          │ breaking change map  │
                                                          └──────────┬───────────┘
                                                                     │
                                  STAGE 4            review plan     │
                                  ───────        ◄───────────────────┘
                             ┌─────────────┐     then run /fix-apis
  *Controller.java ◄─────────│             │
  (modified)        (writes) │ api-fixer   │
  *router.ts ◄───────────────│ agent       │
  New DTOs ◄─────────────────│             │
                             └─────────────┘
```

---

### Diagram 3: Agent Internals

What happens inside each agent when it runs.

```
api-discovery (5 phases, read-only)
─────────────────────────────────────────────────────────────────────
  Phase 1        Phase 2         Phase 3         Phase 4       Phase 5
  ───────        ───────         ───────         ───────       ───────
  Detect         Find all        Extract         Flag          Write
  framework      controller/     12 metadata     9 issue       api-
  from build     route files     fields per      types         inventory
  files          by glob         endpoint        inline        .md
  │              │               │               │             │
  pom.xml        *Controller     HTTP method     PATH_         .claude/
  package.json   *router.ts      Full path       CONFLICT      docs/
  go.mod         views.py        Auth required   MISSING_
  requirements   urls.py         Request body    AUTH ...
  .txt                           Response type
                                 Status code
                                 Path variables
                                 Query params
                                 @Valid present
                                 OpenAPI annot.

api-testing (9 checks, read-only)
─────────────────────────────────────────────────────────────────────
  Read               Run checks                       Write
  api-inventory ──►  1. Path conflicts          ───►  api-issues.md
  .md                2. HTTP status codes
                     3. Response consistency
  Re-read source     4. Input validation
  files to verify    5. Auth consistency
  each finding       6. Naming conventions
                     7. Layer violations
                     8. Error handling
                     9. Security scan

api-consolidation (4 phases, read-only)
─────────────────────────────────────────────────────────────────────
  Read both docs     Apply 5 planning           Write
  api-inventory ──►  principles             ──► consolidation-
  api-issues         │                          plan.md
                     ├─ Critical first
                     ├─ Group by controller      Each change:
                     ├─ Flag breaking changes    • file + line
                     ├─ One concern per change   • current code
                     └─ Phase = deployable unit  • fixed code
                                                 • breaking flag
                                                 • issue ref

api-fixer (5 steps, modifies source files)
─────────────────────────────────────────────────────────────────────
  Step 1     Step 2          Step 3           Step 4      Step 5
  ──────     ──────          ──────           ──────      ──────
  Parse      Apply by        Handle           Track       Output
  plan into  phase           edge cases       progress    summary
  work       (1→2→3→4)       │                │
  queue      per file:       ├─ Code drift:   ├─ Mark     Applied /
             read →          │  adapt fix     │  [x] in   Skipped /
             locate →        ├─ New file:     │  plan     Failed
             edit →          │  write full    └─ Add      counts
             verify →        │  file + import    date
             log             └─ Dependent:
                                re-read between
                                edits
```

---

### Diagram 4: Hook Firing Timeline

When each of the 7 hooks fires during a typical full-pipeline run.

```
  ACTION                               HOOKS THAT FIRE
  ──────                               ───────────────

  Session starts ───────────────────►  [7] Plugin active banner
                                           "Commands: /api-consolidation:..."

  /api-consolidation:api-report
    │
    ├── api-discovery agent runs
    │     └── Writes api-inventory.md ─►  [5] "API Consolidation: Report updated."
    │
    ├── api-testing agent runs
    │     └── Writes api-issues.md ────►  [5] "API Consolidation: Report updated."
    │
    └── api-consolidation agent runs
          └── Writes consolidation-plan.md ► [5] "API Consolidation: Report updated."
              Agent completes ────────────►  [6] "Agent completed. Check .claude/docs/"

  git commit (before fix) ──────────►  [3] "X critical issues remain. Run /fix-apis."

  /api-consolidation:fix-apis
    │
    ├── Before editing OrderController.java
    │     ─────────────────────────────────►  [1] "Modifying endpoint file.
    │                                              Run /validate-apis after changes."
    ├── Edit applied to OrderController.java
    │     ─────────────────────────────────►  [4] "Endpoint file modified.
    │                                              Inventory may be stale."
    ├── Before editing CartController.java
    │     ─────────────────────────────────►  [1] (fires again per file)
    │
    ├── Edit applied to CartController.java
    │     ─────────────────────────────────►  [4] (fires again per file)
    │
    └── api-fixer agent completes
          ───────────────────────────────────►  [6] "Agent completed. Check .claude/docs/"

  /api-consolidation:validate-apis (verify fixes)
    └── Writes api-issues.md ────────────►  [5] "API Consolidation: Report updated."

  Adding a new controller file
    └── Write *Controller.java ──────────►  [2] "New endpoint file detected.
                                                Run /discover-apis to update inventory."

  Hook reference:
  [1] PreToolUse(Edit)   — endpoint file about to be edited
  [2] PreToolUse(Write)  — new endpoint file about to be written
  [3] PreToolUse(Bash)   — git commit with unresolved critical issues
  [4] PostToolUse(Edit)  — endpoint file was just edited
  [5] PostToolUse(Write) — api doc file was just written
  [6] PostToolUse(Agent) — any agent just completed
  [7] Notification       — session started
```

---

### Diagram 5: Prerequisite Chain

Which commands depend on which files. Run them in order — or use `/api-consolidation:api-report` to run stages 1–3 automatically.

```
  /api-consolidation:discover-apis
            │
            │ produces
            ▼
      api-inventory.md
            │
            │ required by
            ▼
  /api-consolidation:validate-apis  ◄── (auto-runs discover-apis if missing)
            │
            │ produces
            ▼
       api-issues.md
            │
            │ required by (along with api-inventory.md)
            ▼
  /api-consolidation:consolidate-apis
            │
            │ produces
            ▼
    consolidation-plan.md
            │
            │ required by
            ▼
  /api-consolidation:fix-apis
            │
            │ produces
            ▼
     Modified source files  +  consolidation-plan.md (checkboxes ticked)
```

---

## Commands

After installation, five slash commands are available. All are namespaced under `api-consolidation:` to avoid conflicts with other plugins.

| Command | Agent Launched | Prerequisite Files | Output |
|---------|---------------|-------------------|--------|
| `/api-consolidation:discover-apis` | `api-discovery` | None | `.claude/docs/api-inventory.md` |
| `/api-consolidation:validate-apis` | `api-testing` | `api-inventory.md` (auto-generated if missing) | `.claude/docs/api-issues.md` |
| `/api-consolidation:consolidate-apis` | `api-consolidation` | Both `api-inventory.md` and `api-issues.md` | `.claude/docs/consolidation-plan.md` |
| `/api-consolidation:fix-apis` | `api-fixer` | `consolidation-plan.md` | Modified source files |
| `/api-consolidation:api-report` | All three sequentially | None | All three doc files |

> `/api-consolidation:fix-apis` is the only command that modifies source code. All others are read-only.

### What each command displays after completing

**`/api-consolidation:discover-apis`** shows:
- Total controllers found
- Total endpoints discovered
- Number of issues flagged
- Top 3 most critical issues

**`/api-consolidation:validate-apis`** shows:
- Executive summary (critical / high / medium / low counts)
- Top 5 priority fixes with effort estimates
- Which controllers have the most issues

**`/api-consolidation:consolidate-apis`** shows:
- Number of changes per phase
- Count of breaking changes
- Files that will be modified
- Estimated effort per phase (based on change count)

**`/api-consolidation:fix-apis`** shows:
- Total changes applied vs skipped vs failed
- List of files modified
- Reminder to run `/validate-apis` to verify fixes
- Reminder about breaking changes that need API consumer notification

**`/api-consolidation:api-report`** shows:
- Combined summary: endpoint count, issue count by severity, change count by phase
- Top 3 recommendations
- Paths to all three generated files
- Prompt to run `/fix-apis` next

---

## Pipeline Overview

The plugin runs as a sequential 4-stage pipeline. Each stage produces a file that the next stage reads.

```
/api-consolidation:api-report (or run stages individually)
          │
          ├─── Stage 1: api-discovery agent
          │         Scans codebase
          │         → writes .claude/docs/api-inventory.md
          │
          ├─── Stage 2: api-testing agent
          │         Reads api-inventory.md, reads source files
          │         → writes .claude/docs/api-issues.md
          │
          └─── Stage 3: api-consolidation agent
                    Reads api-inventory.md + api-issues.md
                    → writes .claude/docs/consolidation-plan.md

(separately, after reviewing the plan)

/api-consolidation:fix-apis
          │
          └─── Stage 4: api-fixer agent
                    Reads consolidation-plan.md
                    → edits actual source files
                    → marks [x] in consolidation-plan.md as changes are applied
```

### Quick start: full pipeline in one command

```
/api-consolidation:api-report
```

Runs stages 1–3. Review `.claude/docs/consolidation-plan.md`, then apply:

```
/api-consolidation:fix-apis
```

### Step-by-step (recommended for first use)

```
/api-consolidation:discover-apis     # 1. Build endpoint inventory
/api-consolidation:validate-apis     # 2. Find all issues
/api-consolidation:consolidate-apis  # 3. Generate fix plan
                                     #    → review .claude/docs/consolidation-plan.md before continuing
/api-consolidation:fix-apis          # 4. Apply fixes to source code
/api-consolidation:validate-apis     # 5. Verify all issues are resolved
```

### Continuous improvement loop

Repeat until the issues report shows zero critical findings:

```
/api-consolidation:validate-apis     # Check current state
/api-consolidation:consolidate-apis  # Plan next round of fixes
/api-consolidation:fix-apis          # Apply
/api-consolidation:validate-apis     # Confirm resolved
```

---

## Agents — Detailed Behavior

Each agent is a focused sub-process with a single responsibility. They run autonomously and produce a specific output file.

---

### api-discovery

Performs a 5-phase read-only scan of the codebase. Does not modify any files.

#### Phase 1 — Framework Detection

Reads build/config files to identify the project stack. Records:
- Framework name and version
- Language and version
- Build tool (Maven, Gradle, npm, pip, etc.)
- Server port (from `application.properties`, `.env`, config files)

#### Phase 2 — Controller/Route Discovery

Globs for all endpoint-defining files using the patterns in the [Supported Frameworks](#supported-frameworks) table.
Looks for framework-specific annotations/decorators: `@RestController`, `router.get()`, `@app.get()`, `path()`, `http.HandleFunc()`, etc.

#### Phase 3 — Endpoint Extraction

For every controller/route file found, extracts 12 metadata fields per endpoint:

| Field | Description |
|-------|-------------|
| HTTP Method | GET, POST, PUT, DELETE, PATCH |
| Full Path | **Base path + method path resolved** (e.g., `@RequestMapping("/api/v1")` + `@GetMapping("/users")` → `/api/v1/users`) |
| Controller | Class or file name |
| Handler Method | Function/method name |
| Auth Required | Yes / No + mechanism (token header, JWT, session, none) |
| Request Body | DTO / model class name, or inline params |
| Response Type | Return type (ResponseEntity, raw object, etc.) |
| HTTP Status Code | Declared or inferred status |
| Path Variables | Any `{id}`, `{name}`, etc. in the path |
| Query Params | Any `@RequestParam` or query string parameters |
| Validation | Whether `@Valid` or equivalent is present |
| OpenAPI Annotations | Whether Swagger/OpenAPI annotations exist |

> **Path resolution rule:** Both class-level and method-level annotations are always checked and combined into the full path. A controller with `@RequestMapping("/orders")` and a method with `@GetMapping("/{id}")` produces the full path `/orders/{id}`.

#### Phase 4 — Issue Flagging

While extracting, flags known problem patterns inline on each endpoint:

| Flag | Severity | Description |
|------|----------|-------------|
| `PATH_CONFLICT` | Critical | Two endpoints that could match the same URL |
| `PATH_AMBIGUITY` | Critical | Path variable overlaps with a literal segment (e.g., `/users/{id}` vs `/users/me`) |
| `MISSING_AUTH` | Critical | Sensitive operation (DELETE, PII, orders) without authentication |
| `WRONG_STATUS` | High | Incorrect HTTP status code for the operation type |
| `DUPLICATE` | High | Same functionality at different paths |
| `LAYER_VIOLATION` | Medium | Repository/DAO injected directly in a controller |
| `MISSING_VALIDATION` | Medium | Request body without `@Valid` or equivalent |
| `NAMING_ISSUE` | Low | Non-RESTful path (verbs in URL, singular/plural mismatch) |
| `SECURITY_RISK` | Critical | PCI/PII data in request/response body without protection |

#### Phase 5 — Output

Writes `.claude/docs/api-inventory.md` with:
- Project metadata header
- Summary table (controller count, endpoint count, auth breakdown, issues count)
- Per-controller endpoint tables (8 columns: method, path, handler, auth, request body, response, status, issues)
- Aggregated issue summary by type
- Flagged endpoints list with explanations

Safe to re-run at any time — overwrites the previous inventory with fresh data.

---

### api-testing

Runs 9 automated quality checks against every endpoint. Reads actual source files (not just the inventory) to verify findings. Does not modify any files.

**Prerequisite:** `api-inventory.md` must exist. `/api-consolidation:validate-apis` will auto-run discovery first if the file is missing.

See [The 9 Quality Checks](#the-9-quality-checks) for full check details.

Writes `.claude/docs/api-issues.md` with:
- Executive summary with 🔴/🟠/🟡/🟢 severity breakdown
- Findings grouped by each of the 9 checks (pass/fail status per check)
- Findings grouped by controller (developer-friendly parallel view)
- Top 10 priority fixes with effort estimates
- Statistics: pass rate, most-problematic controller, most common issue type

---

### api-consolidation

Reads both the inventory and issues files, then designs a phased remediation plan. Does not modify any source code.

**Prerequisites:** Both `api-inventory.md` and `api-issues.md` must exist. If either is missing, the agent stops and tells you which command to run.

#### 5 Planning Principles

The agent follows these principles when designing the plan:

1. **Fix critical issues first** — security and path conflicts before style or architecture
2. **Minimize blast radius** — changes are grouped by controller to limit risk per deployment
3. **Preserve backwards compatibility** — all breaking changes are flagged explicitly with migration notes
4. **One concern per change** — unrelated fixes are never bundled into a single change block
5. **Test-friendly** — each phase is designed to be independently deployable and testable

#### 4 Consolidation Phases

| Phase | Priority | Scope |
|-------|----------|-------|
| Phase 1: Critical Security Fixes | Immediate | Missing auth, path conflicts, PCI/PII violations |
| Phase 2: Correctness Fixes | Next sprint | Wrong status codes, missing `@Valid`, inconsistent response wrapping |
| Phase 3: Architecture Improvements | Backlog | Layer violations, DTO separation, error handling gaps |
| Phase 4: Convention Normalization | Backlog | Path renaming, singular/plural fixes, verb removal from paths |

#### Every change in the plan includes

- Target file path and line number
- Exact current code block (copied from the live file)
- Exact replacement code block (complete, compilable, import-ready)
- Breaking change flag with migration notes for API consumers
- Reference to the originating issue number from `api-issues.md`

#### The generated `consolidation-plan.md` ends with

An **Implementation Checklist** for tracking progress:

```
- [ ] Phase 1: Security fixes (deploy immediately after)
- [ ] Phase 2: Correctness fixes
- [ ] Phase 3: Architecture improvements
- [ ] Phase 4: Convention normalization
- [ ] Run /api-consolidation:validate-apis to verify all issues resolved
- [ ] Update API documentation / Swagger annotations
- [ ] Notify API consumers of breaking changes
```

The api-fixer agent checks these boxes (`[x]`) as it completes each section.

---

### api-fixer

Reads `consolidation-plan.md` and applies every change to actual source code. **This is the only agent that modifies files.**

**Prerequisite:** `consolidation-plan.md` must exist. Run `/api-consolidation:consolidate-apis` first.

**Safety prompt:** Before starting, `/api-consolidation:fix-apis` asks you to:
1. Confirm you have reviewed the consolidation plan
2. Commit your current changes first (`git add . && git commit`)
3. Choose whether to apply all phases or a specific phase only

#### 5-Step Execution Strategy

**Step 1 — Parse the plan**
Reads `consolidation-plan.md` and builds an ordered work queue of every code change: file path, phase number, current code block ("before"), replacement code block ("after"), and whether it's an edit, a new file, or a deletion.

**Step 2 — Apply changes phase by phase**
Processes phases in order (1 → 2 → 3 → 4). Within each phase, processes one file at a time:
1. Reads the target file completely
2. Locates the exact code block to change
3. Applies the change with the Edit tool
4. Re-reads the modified section to verify correctness
5. Logs the change as completed

**Step 3 — Handle edge cases**

| Situation | What the agent does |
|-----------|---------------------|
| Current code doesn't match the plan | Reads the actual file, adapts the fix to match what's there, logs the adaptation |
| New file needed (e.g., a DTO class) | Checks the target directory exists, writes the complete file with package declaration and imports |
| Changes are dependent on each other | Applies them in order within the same file, re-reads between dependent edits |

**Step 4 — Track progress**
After each section completes, updates `consolidation-plan.md` by changing `- [ ]` to `- [x]` and adding `Applied on {date}`. This lets you resume safely if the run is interrupted.

**Step 5 — Final summary**
Outputs a table of every change with applied / skipped / failed status, plus statistics and next steps.

#### api-fixer Rules

- Apply changes **exactly** as specified — the agent does not improvise improvements beyond the plan
- If a change cannot be applied cleanly, it is **skipped and reported** — never partially applied
- Code is **never deleted** unless the plan explicitly calls for deletion
- Existing code formatting and style conventions are **preserved**
- If the plan references new DTO classes, **complete files** are written with proper package declarations and all imports

---

## The 9 Quality Checks

All 9 checks apply to every supported framework. The agent reads actual source files to verify, not just the inventory.

| # | Check | Severity | What it catches |
|---|-------|----------|----------------|
| 1 | Path Conflict Detection | 🔴 Critical | Exact duplicate paths; variable overlap (e.g., `/products/{id}` vs `/products/{catenum}` — both match `/products/123`); prefix collision (e.g., `/customer/{id}` vs `/customer/current`) |
| 2 | HTTP Status Code Correctness | 🟠 High | POST returning 200 instead of 201; GET returning 302; DELETE returning 200 instead of 204; synchronous operations returning 202 |
| 3 | Response Wrapper Consistency | 🟡 Medium | Some endpoints returning raw objects while others use `ResponseEntity<>`; inconsistent error response format across controllers |
| 4 | Input Validation Coverage | 🟠 High | `@RequestBody` params missing `@Valid`; DTOs without JSR-380 constraints (`@NotNull`, `@Size`, `@Email`, etc.); `@RequestParam` without validation |
| 5 | Authentication Consistency | 🔴 Critical | DELETE or PUT endpoints without auth; endpoints returning emails, addresses, or order data without authentication; admin-level listing endpoints (all users, all orders) without auth |
| 6 | Naming Convention Consistency | 🟢 Low | Verbs in paths (`/cart/add` → `/cart/items`); inconsistent singular/plural across controllers; non-kebab-case multi-word paths; action words in paths (`/register/customer` → `POST /customers`) |
| 7 | Layer Violation Detection | 🟡 Medium | Repository or DAO interfaces injected directly into controllers (bypassing service layer); business logic in controllers instead of services; controllers calling other controllers |
| 8 | Error Handling Completeness | 🟡 Medium | No `@ControllerAdvice` global exception handler; no custom exceptions per domain; endpoints that don't handle not-found cases; inconsistent error response structure; catch-all handlers that swallow exceptions silently |
| 9 | Security Vulnerability Scan | 🔴 Critical | PCI-DSS: credit card numbers in request/response bodies; PII exposure: SSNs or passwords in responses; mass assignment: JPA entities used directly as request bodies; missing CORS configuration; no rate limiting on auth/payment endpoints; SQL injection vectors |

Severity guide: 🔴 Critical — fix immediately | 🟠 High — fix this sprint | 🟡 Medium — schedule for backlog | 🟢 Low — style, address when convenient

---

## Generated Files

All output is written to `.claude/docs/`. These files are regenerated on each run and are safe to gitignore.

### `.claude/docs/api-inventory.md`

Generated by `/api-consolidation:discover-apis`.

| Section | Contents |
|---------|----------|
| Header | Framework, language, version, port, generation date |
| Summary table | Total controllers, total endpoints, authenticated vs unauthenticated count, issues flagged |
| Per-controller tables | One table per controller: method, path, handler, auth, request body, response type, status code, inline issue flags |
| Issue summary | Aggregated issue count by type across all controllers |
| Flagged endpoints | Each flagged endpoint with the issue type and a plain-English explanation |

### `.claude/docs/api-issues.md`

Generated by `/api-consolidation:validate-apis`.

| Section | Contents |
|---------|----------|
| Executive summary | 🔴/🟠/🟡/🟢 breakdown with counts |
| Findings by check | Pass/fail status per check; detailed finding table with endpoint, issue, and recommendation |
| Findings by controller | Same findings reorganized by controller (for developer-friendly review) |
| Top 10 priority fixes | Ordered list with effort estimates (Low / Medium / High) |
| Statistics | Pass rate (X/Y endpoints), most-problematic controller, most common issue type |

### `.claude/docs/consolidation-plan.md`

Generated by `/api-consolidation:consolidate-apis`.

| Section | Contents |
|---------|----------|
| Change impact summary | Files to be modified, endpoints changed, breaking changes count, new files to be created |
| 4 phases | Each change has: file path, severity, issue reference, current code block, replacement code block, breaking change flag + migration notes |
| Path migration map | Table of old path → new path for all renamed endpoints, with breaking change flag |
| Implementation checklist | Checkboxes for each phase + post-fix steps (validate, update docs, notify consumers) — checked off by api-fixer as it runs |

---

## Hooks

The plugin installs 7 hooks that fire automatically. They provide real-time reminders without requiring any manual action.

### Hook execution environment

Hooks run shell commands. The variable `$CLAUDE_TOOL_INPUT` is available in every hook and contains the path or command passed to the triggering tool.

### PreToolUse hooks (fire before a tool runs)

**1. Endpoint file edit warning**
- **Trigger:** Edit tool on any file matching `(Controller|Resource|Endpoint).(java|ts|py|go)`
- **Message:** `"API Consolidation: Modifying an endpoint file. Run /api-consolidation:validate-apis after changes."`

**2. New endpoint file warning**
- **Trigger:** Write tool on any file matching `(Controller|Resource|Endpoint|router|route).(java|ts|js|py|go)`
- **Message:** `"API Consolidation: New endpoint file detected. Run /api-consolidation:discover-apis to update inventory."`

**3. Pre-commit critical issues gate**
- **Trigger:** Any `git commit` command
- **Action:** Reads `.claude/docs/api-issues.md` and counts occurrences of "critical" (case-insensitive). If count > 0:
- **Message:** `"API Consolidation: {N} critical API issues remain. Run /api-consolidation:fix-apis before committing."`
- Silently passes (no message) if `api-issues.md` doesn't exist yet or if there are zero critical issues.

### PostToolUse hooks (fire after a tool runs)

**4. Endpoint file modified reminder**
- **Trigger:** Edit tool on any file matching `(Controller|Resource|Endpoint|router|route).(java|ts|js|py|go)`
- **Message:** `"API Consolidation: Endpoint file modified. API inventory may be stale."`

**5. API docs updated confirmation**
- **Trigger:** Write tool on any file whose name matches `api-(inventory|issues|consolidation)`
- **Message:** `"API Consolidation: Report updated."`

**6. Agent completion notification**
- **Trigger:** Any Agent tool call completing
- **Message:** `"API Consolidation: Agent completed. Check .claude/docs/ for reports."`

### Notification hooks (fire on session events)

**7. Plugin active banner**
- **Trigger:** Session start notification
- **Message:** `"API Consolidation Plugin active. Commands: /api-consolidation:discover-apis, /api-consolidation:validate-apis, /api-consolidation:consolidate-apis, /api-consolidation:fix-apis, /api-consolidation:api-report"`

### Hook flow example

```
User runs /api-consolidation:fix-apis
    │
    ├─ PreToolUse(Edit) fires → "Modifying an endpoint file..."
    │
    ├─ Edit tool runs on OrderController.java
    │
    ├─ PostToolUse(Edit) fires → "Endpoint file modified. Inventory may be stale."
    │
    ├─ ... (repeats for each file edited) ...
    │
    └─ PostToolUse(Agent) fires → "Agent completed. Check .claude/docs/ for reports."
```

---

## Permissions

Defined in `settings.json`. All permissions are granted at install time — no runtime prompts.

### Allowed

| Permission | Needed by | Purpose |
|------------|-----------|---------|
| `Read(**)` | All agents | Read source files during scanning and validation |
| `Glob(**)` | api-discovery | Find controller/route files by pattern |
| `Grep(**)` | api-discovery, api-testing | Search file contents for endpoint patterns and annotations |
| `Write(.claude/docs/*)` | api-discovery, api-testing, api-consolidation | Write generated report files |
| `Edit(src/**/*.java)` | api-fixer | Apply code changes to Java source files |
| `Agent(api-discovery)` | All commands | Commands can launch the discovery agent |
| `Agent(api-testing)` | All commands | Commands can launch the testing agent |
| `Agent(api-consolidation)` | All commands | Commands can launch the consolidation agent |
| `Agent(api-fixer)` | fix-apis command | The fix command can launch the fixer agent |

### Denied (safety guardrails)

| Denied operation | Prevents |
|-----------------|---------|
| `Bash(rm -rf *)` | Recursive deletion of project files |
| `Bash(git push --force*)` | Force-push that can overwrite remote history |
| `Bash(git reset --hard*)` | Hard reset that discards uncommitted work |

### Adapting `Edit` permission for non-Java projects

Update the `Edit` permission in `settings.json` to match your source file paths before running `/api-consolidation:fix-apis`:

```json
"Edit(src/**/*.{java,kt})"    // Java + Kotlin
"Edit(**/*.ts)"                // TypeScript / Express
"Edit(app/**/*.py)"            // Python / FastAPI / Django
"Edit(**/*.go)"                // Go
"Edit(src/**/*)"               // Any file under src/
```

---

## Plugin Structure

```
api-consolidation-plugin/
├── .claude-plugin/
│   ├── plugin.json              # Plugin manifest (name, version, author, repository)
│   └── marketplace.json         # Marketplace entry for distribution
├── agents/
│   ├── api-discovery.md         # 5-phase endpoint scanner agent
│   ├── api-testing.md           # 9-check quality validator agent
│   ├── api-consolidation.md     # 4-phase fix planner agent
│   └── api-fixer.md             # Code change applier agent
├── commands/
│   ├── api-report.md            # Orchestrator command (discover + validate + plan)
│   ├── consolidate-apis.md      # Consolidation command
│   ├── discover-apis.md         # Discovery command
│   ├── fix-apis.md              # Fixer command (includes safety prompt)
│   └── validate-apis.md         # Validation command (auto-discovers if needed)
├── skills/
│   ├── api-consolidation/
│   │   └── SKILL.md             # Auto-trigger: "consolidate APIs", "plan fixes"
│   ├── api-discovery/
│   │   └── SKILL.md             # Auto-trigger: "scan APIs", "find endpoints"
│   ├── api-fixer/
│   │   └── SKILL.md             # Auto-trigger: "apply fixes", "execute plan"
│   ├── api-report/
│   │   └── SKILL.md             # Auto-trigger: "full API audit", "run pipeline"
│   └── api-testing/
│       └── SKILL.md             # Auto-trigger: "validate APIs", "check quality"
├── hooks/
│   └── hooks.json               # All 7 hook definitions with shell commands
├── settings.json                # Permissions (allow + deny lists)
└── README.md                    # This file
```

**`commands/` vs `skills/`:** Commands define what a slash command does (the full agent prompt and instructions). Skills define auto-trigger conditions — they tell Claude when to invoke a command automatically based on what the user says (e.g., "scan my APIs" triggers `api-discovery` without requiring the explicit `/api-consolidation:discover-apis` invocation).

---

## Customization

### Add a new validation check

Edit `agents/api-testing.md` and add a Check 10 section following the existing format:

```markdown
### Check 10: Rate Limiting Coverage (Medium)

Check whether sensitive endpoints (auth, payment, search) have rate limiting indicators.
Flag endpoints that handle authentication or financial operations without visible rate limiting middleware.
```

Update the output format table in the same file to include Check 10 in the results.

### Change a severity level

Edit the severity label in the check heading in `agents/api-testing.md`:

```markdown
### Check 6: Naming Convention Consistency (Medium)    # was (Low)
```

### Change consolidation phase order or priorities

Edit `agents/api-consolidation.md` to reorder phases, merge phases, or add new ones.
Current order (from highest to lowest priority): Critical Security → Correctness → Architecture → Conventions.

### Add a new hook

Edit `hooks/hooks.json` and add an entry to the relevant section:

```json
{
  "matcher": "Bash(git push*)",
  "hooks": [
    {
      "type": "command",
      "command": "bash -c \"echo 'API Consolidation: Pushing — ensure /api-consolidation:validate-apis passed first.'\""
    }
  ]
}
```

**Available matchers:** `"Edit"`, `"Write"`, `"Bash"`, `"Bash(git commit*)"`, `"Bash(git push*)"`, `"Agent"`, `""` (empty = all notifications).
**Available env var:** `$CLAUDE_TOOL_INPUT` — the file path or command string passed to the triggering tool.

---

## Troubleshooting

### "No API inventory found"

**Cause:** Running `/validate-apis` or `/consolidate-apis` before `/discover-apis`.
**Fix:** Run `/api-consolidation:discover-apis` first, or use `/api-consolidation:api-report` which runs all stages in order. Note: `/validate-apis` will also auto-run discovery automatically if the inventory is missing.

### "No consolidation plan found"

**Cause:** Running `/fix-apis` before `/consolidate-apis`.
**Fix:** Run the full pipeline in order: discover → validate → consolidate → fix.

### Agent finds 0 endpoints

**Cause:** Framework not detected, or controller files are in a non-standard location.
**Fix:** Verify your build file (`pom.xml`, `package.json`, etc.) is in the project root. Check that controller files follow standard naming patterns (`*Controller.java`, `routes/*.js`, etc.).

### Fixer skips a change

**Cause:** The "current code" block in the consolidation plan no longer matches the actual file — the file was modified after the plan was generated.
**Fix:** Re-run `/api-consolidation:consolidate-apis` to regenerate the plan from the current code state, then run `/api-consolidation:fix-apis` again.

### Edit permission denied when running fix-apis

**Cause:** The `Edit` permission in `settings.json` uses the Java pattern (`src/**/*.java`) but your project uses a different language or directory structure.
**Fix:** Update the `Edit(...)` pattern in `settings.json` to match your source files. See [Adapting Edit permission for non-Java projects](#adapting-edit-permission-for-non-java-projects).

### Hook not firing

**Cause:** The matcher string in `hooks/hooks.json` doesn't match the actual tool call pattern, or the file extension isn't in the grep pattern.
**Fix:** Open `hooks/hooks.json` and check the `grep -qE` pattern for your file extension. The default pattern covers `.java|.ts|.js|.py|.go`. Add your extension if it's missing. Use `""` (empty matcher) to match all tool calls for debugging.

### Pre-commit hook shows wrong critical count

**Cause:** The hook counts all occurrences of the word "critical" in `api-issues.md`, including occurrences in headings or descriptions, not just issue findings.
**Note:** This is by design — the count may be slightly higher than the actual issue count. It's a conservative warning, not an exact count.

### Fix applied but issue still shows in validate-apis

**Cause:** The source file was changed but the inventory or issues report was generated before the fix. The reports are snapshots in time.
**Fix:** Re-run `/api-consolidation:validate-apis` after applying fixes to regenerate the issues report from the updated source code.

---

## Repository

- **Source:** https://github.com/techmsrini8-sudo/claude-api-consolidation
- **Issues:** https://github.com/techmsrini8-sudo/claude-api-consolidation/issues
- **License:** MIT
