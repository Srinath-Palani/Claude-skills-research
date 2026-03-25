---
name: unified-mcp-skill
user-invocable: true
description: >
  All-in-one unified MCP skill: research MCP servers with security scoring, document attributes,
  conditionally set up locally, diagnose errors inline, and audit projects. Single optimized
  execution path for research, deployment, error recovery, and compliance auditing. Faster,
  fewer tokens, full functionality.
---

# Unified MCP Skill — Research, Deploy, Fix, Audit

**All-in-one MCP server management:** Research with security scoring, document attributes, conditional local setup (clone+install), inline error recovery, and project compliance auditing — all in one unified skill with optimized execution paths.

---

## Core Capabilities

| Capability | Description | Invocation Signal |
|-----------|-------------|------------------|
| **MCP Research** | Research and security-score MCP servers | "Research the GitHub MCP server", "Analyze this MCP: https://..." |
| **Attribute Docs** | Fill every MCP attribute with evidence | "Document attributes for...", "Catalogue this MCP server" |
| **Local Setup** | Clone, install, configure MCP servers | "Set up this server locally", "Clone and run..." |
| **Error Recovery** | 7-phase inline diagnosis and auto-fix | Automatic on connection fail |
| **Project Audit** | Audit project and skills for compliance | "Review the project", "Is this ready to commit?" |

---

## Feature Overview

✓ **3 input types:** Remote endpoint / GitHub URL / Server name (auto-detect)
✓ **Protocol version verification** before research
✓ **Conditional local setup** — only clone+install if user wants local deployment
✓ **4 connection methods** — STDIO / Published Package / Docker / Remote
✓ **Configurable paths** (report save + clone paths, stored for future)
✓ **Logic enforcement** — STDIO↔Local, HTTP/SSE↔Remote auto-set
✓ **7-phase inline error recovery** — automatic diagnosis on connection fail
✓ **10-attribute security scoring** (0–53 scale)
✓ **Evidence-backed attribute documentation**
✓ **CSV report output** with full traceability
✓ **Project compliance audit** with PASS/FAIL/WARN checks
✓ **Skill 2.0 self-learning** — learned-fixes.md for error patterns
✓ **Token-optimized** — single execution path, no redundant processing

---

## Security Mandate — Always Enforced

- **Never ask for credentials in chat** — absolute rule, no exceptions
- **Always use `<PLACEHOLDER>` values** in config examples
- **Direct user to edit files directly** — credentials only via filesystem
- **On accidental credential paste:**
  ```
  Security alert: Do not share credentials here. Treat as compromised.
  Revoke and add new one directly to settings file at <path>. Do NOT paste here.
  ```
- **Never store credentials** in committed files (README, CSV, .env)

---

## Usage

**Invoke with:** `/start-mcp`

```
RESEARCH
/start-mcp "Research the GitHub MCP server"
/start-mcp "Get full report on the Slack MCP server"
/start-mcp "Document this MCP server: https://github.com/org/repo"
/start-mcp "What does the Linear MCP server support?"
/start-mcp "Analyze and score this MCP: https://mcp.example.com"

LOCAL SETUP
/start-mcp "Set up the GitHub MCP server locally"
/start-mcp "Clone and get the Slack MCP running"
/start-mcp "Install this MCP server from GitHub"

ERROR RECOVERY (automatic)
/start-mcp "The MCP server won't connect"
/start-mcp "I'm getting a 401 error from my MCP server"
/start-mcp [paste stack trace or error message]

PROJECT AUDIT
/start-mcp "Review the project"
/start-mcp "Audit all skills for alignment"
/start-mcp "Is this ready to commit?"
/start-mcp "Check if my changes meet requirements"
```

---

## Unified Workflow: 8 Steps + Conditional Local Setup + Inline Error Recovery

### Step 0 — Configuration & Input Classification

**First-time path configuration (simple 2-step):**

```
Where to save MCP reports?
[Open Folder Picker Button]

User selects → Used for this session
On next use: "Use saved default? [Yes/No]"
If Yes → Saved to .claude/settings.json
```

Same flow for clone location: `/Users/yourname/MCP_repos/`

**Classify input:**

| Input Type | Signal | Transport | Deployment | Next |
|-----------|--------|-----------|-----------|------|
| **Remote Endpoint** | `https://`, `/sse` or `/mcp` path | HTTP/SSE or StreamableHttp | Remote=Yes | TLS verify, skip local setup |
| **GitHub URL** | Contains `github.com` | STDIO=Yes | Local=Yes | Offer local setup or research only |
| **Server Name Only** | Plain text (e.g., "GitHub MCP") | N/A | N/A | Search vendor docs, present options |

---

### Step 1 — Protocol Version Verification

**Run BEFORE filling attributes:**

1. Locate SDK version: `uv.lock` → `package-lock.json` → `pyproject.toml` → `package.json` → `requirements.txt`
2. Extract pinned version, fetch latest from PyPI/npm
3. If outdated by 1+ minor: ask user (Option 1: Update / 2: Keep / 3: Cancel)
4. Map to protocol version:
   - mcp ≥1.24 → protocol 2025-11-25
   - mcp 1.15–1.23 → protocol 2025-06-18
   - mcp <1.15 → protocol 2024-11-05 or earlier
5. Record both SDK and protocol in report

---

### Step 2 — Type A: Remote Endpoint URL

1. Extract protocol from path (`/sse` → HTTP/SSE, `/mcp` → StreamableHttp)
2. Verify TLS: `curl checks, document results`
3. Test connection: `curl -s -o /dev/null -w "%{http_code}" <url>`
4. Detect auth (WWW-Authenticate header, /.well-known/oauth, vendor docs)
5. Ask config location: Option 1: Global (~/.claude/settings.json) / 2: Project
6. Build config with placeholders, guide user to edit directly
7. **Test initialize request** — if fails → Error Recovery Phase 1

---

### Step 3 — Type B: GitHub Repository URL (Conditional Local Setup)

**First, ask deployment preference:**

```
How do you want to run this server?

Option 1: Locally (clone + install + configure)
          → Full setup: clone, install deps, test connection
Option 2: Use remote endpoint (if available)
          → Check for remote endpoint in README
Option 3: Just research attributes (no setup)
          → Skip local setup, only research

Select option (1/2/3):
```

**If Option 1 (Local Setup):**

| Step | Action | Check |
|------|--------|-------|
| 3.1 | Clone repo to configured path | Verify dir exists, expected files present |
| 3.2 | Determine connection method | Identify STDIO/Docker/Published from README |
| 3.3 | Get approval for install plan | Show deps + env vars, wait for "yes" |
| 3.4 | Install dependencies | Python: venv+pip / Node: npm+build / Docker: docker build |
| 3.5 | Extract configuration | Read command, args, env vars from README + source code |
| 3.6 | Test STDIO/connection | Send initialize JSON-RPC, verify response |

**Connection Methods (Step 3.2):**

| Method | Indicator | Setup | Deps Install |
|--------|-----------|-------|--------------|
| **STDIO** | stdin/stdout, subprocess mention | Clone + install locally | Python: venv+pip / Node: npm+build |
| **Published Package** | PyPI/npm registry, pip/npx | No clone needed | `pip install <pkg>` or `npx <pkg>` |
| **Docker** | Dockerfile present | Clone + docker build | `docker build -t <name> .` |
| **Remote** | vendor URL, no local setup | Skip all setup | N/A |

**Dependency Installation (Step 3.4):**

| Language | Command | Verification |
|----------|---------|--------|
| Python | `python3 -m venv .venv && source .venv/bin/activate && pip install -r requirements.txt` | `python -c "import mcp"` |
| Node/TS | `npm install && npm run build` (if build in scripts) | Check dist/ exists |
| Docker | `docker build -t <repo-name> .` | `docker images \| grep <repo-name>` |
| uv | `uv sync` | `uv run python -c "import mcp"` |
| Published | `pip install <package-name>` or `npx <package>` | Version check or --help |

**Configuration Extraction (Step 3.5):**

After install, extract from local disk:
- README.md → start command, args, env var names
- pyproject.toml / package.json → exact entry point
- .env.example → all env vars with descriptions
- uv.lock / package-lock.json → dependency versions
- Main script → @mcp.tool() decorators, env reads

Determine:
- **`command`**: executable to run (e.g. uv, node, python3, docker)
- **`args`**: argument list (e.g. ["--directory", "<clone-path>", "run", "server.py"])
- **`env`**: env var names and their auth types

**If Option 3 (Research Only):**
Skip local setup, proceed to Step 4 (attribute research).

**If Option 2 (Check Remote):**
Look for remote endpoint in README, offer as alternative to local setup.

---

### Step 4 — Type C: Server Name Only

1. Search vendor documentation for remote endpoint
2. Search for GitHub repository
3. If both found: ask (Option 1: Remote / 2: GitHub local / 3: Help me decide)
4. Continue with chosen Type A or B workflow

---

### Step 5 — Attribute Filling with Logic Enforcement

**Transport ↔ Deployment Rules:**
- STDIO=Yes → Deployment: Local=Yes (auto-set)
- HTTP/SSE OR StreamableHttp=Yes → Deployment: Remote=Yes (auto-set)

**Key Attributes (Mutually Exclusive):**

| Attribute | Exclusive | Rule |
|-----------|-----------|------|
| Distribution | Yes | Official OR Community only |
| Pricing | Yes | Free OR Paid only |
| Hosting | Yes | One of: SaaS/GitHub/GitLab/3rdParty |
| Transport | Yes | One of: STDIO/HTTP-SSE/StreamableHttp |
| Tools Operations | Yes | Highest level only (Read-only / R+U / R+U+D) |

**Attributes to Document (Evidence-backed):**

| Attribute | Type | Source Priority |
|-----------|------|-----------------|
| Name | Text | GitHub repo or vendor docs |
| Description | Text | README or vendor site |
| Category | Choice | mcp.md or vendor classification |
| Git Repo | URL | GitHub link |
| Git Version | Version | Latest tag/release |
| Distribution | Choice | Official / Community |
| Pricing | Choice | Free / Paid |
| Authentication | Choice | OAuth 2.1 / Bearer / PAT / Token / None |
| Hosting | Choice | SaaS / GitHub / GitLab / Self-hosted |
| Transport | Choice | STDIO / HTTP/SSE / StreamableHttp |
| Deployment | Choice | Remote / Local / Hybrid |
| Tools Operations | Choice | Read-only / R+U / R+U+D |
| Capabilities | List | Text/Resource/Prompt/Server |

**Source Priority (highest trust first):**
1. Official vendor documentation
2. GitHub repository & source code
3. PyPI / npm package page
4. MCP specification documentation

**Every Yes must be backed by evidence** (quote or file path).

---

### Step 6 — Tools & Capabilities

Extract from source code:
- Find @mcp.tool() decorators
- Group by category
- List non-read-only operations
- Check capabilities (T/R/P/S)

---

### Step 7 — Basic Information

Extract from vendor docs + GitHub:
- Name, Description, Category, Git Repo Version

---

### Step 8 — Calculate Security Score (Optional)

```
Include CCI Security Score?
Option 1: Yes, calculate
Option 2: No, skip
Option 3: Cancel
```

**10 Attributes (0–53 total):**

| Attribute | Max | Scoring |
|-----------|-----|---------|
| Protocol Version | 5 | Current=5, 1 behind=3, older=1 |
| Distribution | 5 | Official=5, Community=3 |
| Pricing | 3 | Free=3, Paid=1 |
| Hosting | 5 | SaaS=5, GitHub=3, self-hosted=1 |
| Auth | 5 | OAuth 2.1=5, Bearer/PAT=3, Token=2, None=1 |
| TLS | 5 | 1.3=5, 1.2=3, N/A (STDIO)=3, None=1 |
| Transport | 5 | StreamableHttp=5, HTTP/SSE=3, STDIO=2 |
| Operations | 5 | R+U+D=5, R+U=3, Read-only=1 |
| Deployment | 3 | Remote vendor=3, Local/Container=2, Complex=1 |
| Capabilities | 3 | 4+ types=3, 2–3=2, 1=1 |

---

### Step 9 — Save Report

Location: User-configured path
Format: CSV + Markdown
Filename: `<servername>.csv` and `<servername>.md`

---

## Error Recovery: 7 Phases (Inline on Connection Test Fail)

If connection test fails at Step 3.6 or Step 2, automatically trigger error recovery:

### Phase 1 — Capture Error
Read error message from failed test, classify by category:
1. **Dependency/Install** — ModuleNotFoundError, ImportError, npm ERR!, pip failed, command not found
2. **Configuration** — Invalid config, JSON error, wrong path in settings.json
3. **Authentication** — 401, 403, token expired, missing env var
4. **Transport/Process** — Connection refused, ECONNREFUSED, EPIPE, spawn error
5. **Protocol Version** — Unsupported version, capability negotiation failed
6. **Runtime/Crash** — RuntimeError, TypeError, unhandled exception
7. **Environment/Path** — Python version mismatch, venv not active, binary not on PATH

### Phase 2 — Diagnose

Run in order, stop at first failure:
- Runtime version check (Python/Node/uv --version)
- Config file validity (syntax, command exists, paths resolve)
- Dependencies importable (python -c "import mcp")
- Server start directly (<command> <args>)
- Test initialize JSON-RPC (expect result + protocolVersion)
- Claude Code logs (tail ~/Library/Logs/Claude/mcp*.log)

### Phase 3 — Present Fix Plan
Show category + root cause + ordered steps with risk level. Ask: "Proceed with Step 1?"

### Phase 4 — Execute Fixes

| Category | Command |
|----------|---------|
| Dependency (Python) | python3 -m venv .venv && pip install -r requirements.txt |
| Dependency (uv) | uv sync |
| Dependency (Node) | npm install && npm run build |
| Config | Show diff, user edits directly |
| Auth | Direct to settings file, use <KEY> format |
| Transport | Restart Claude Code |

**Every step requires user approval** — "Proceed with Step 2? (yes/skip/cancel)"

### Phase 5 — Verify Connection
Test initialize request again. Success → continue research. Fail → re-diagnose (Phase 2).

### Phase 6 — Record Pattern (Skill 2.0)
If fix succeeds, append to `references/learned-fixes.md`:
```
## [Error title]
**Signals:** [symptoms]
**Cause:** [one line]
**Fix:** [steps]
**Verify:** [how to test]
**Date:** YYYY-MM-DD | **Category:** [1–7]
```

---

## Project Audit: Compliance Review

Use this when: `"Review the project"`, `"Is this ready to commit?"`, `"Check if my changes meet requirements"`

### Audit Checks (PASS/FAIL/WARN)

**Project Goal Alignment:**
- [ ] Project follows MCP research & deployment workflow
- [ ] Skills support research, setup, error handling, audit
- [ ] Output reports saved to correct locations

**Enterprise Security Requirements:**
- [ ] No credentials in chat (all via file editing)
- [ ] No hardcoded secrets in code
- [ ] All config examples use `<PLACEHOLDER>` syntax
- [ ] Security mandates enforced at every step

**Confidentiality Rules:**
- [ ] No API keys in committed files
- [ ] No credentials in README or reports
- [ ] Settings files added to .gitignore
- [ ] Credentials routed to filesystem only

**Cross-Skill Consistency:**
- [ ] All skills enforce same security rules
- [ ] Unified error recovery approach
- [ ] Consistent attribute documentation
- [ ] Same workflow terminology

**Completeness Verification:**
- [ ] All workflows have approval gates
- [ ] Error recovery covers all 7 categories
- [ ] Security scoring validated against CCI spec
- [ ] Reports include evidence backing

### Audit Output

```
PROJECT REVIEW — 2026-03-26
═══════════════════════════════════════════════════════════════

✅ PASS  Project goal alignment (all checks)
✅ PASS  Enterprise security requirements (8/8)
⚠️  WARN  Confidentiality rules: 3/4 (check .gitignore)
✅ PASS  Cross-skill consistency
✅ PASS  Completeness verification

Overall: PASS (3 items to review before commit)
Recommend: Address WARN items, then commit

═══════════════════════════════════════════════════════════════
```

---

## Common Mistakes to Avoid

✗ Not verifying SDK version before research
✗ Connecting to wrong endpoint/repo (always use user input)
✗ Mixing STDIO/Local or HTTP/Remote incorrectly
✗ Not storing user paths for future use
✗ Skipping local setup when user prefers local deployment
✗ Installing deps without user approval
✗ Asking user to paste credential during error diagnosis
✗ Marking multiple values in mutually exclusive fields
✗ Not saving new error patterns to learned-fixes.md
✗ Forcing local setup when remote endpoint exists and user prefers it

---

## Credential Placeholder Map

| Auth Type | Placeholder |
|-----------|-------------|
| API Token | `<API_KEY>` |
| Bearer Token | `<BEARER_TOKEN>` |
| PAT | `<PERSONAL_ACCESS_TOKEN>` |
| OAuth Client ID | `<OAUTH_CLIENT_ID>` |
| OAuth Client Secret | `<OAUTH_CLIENT_SECRET>` |
| Generic Secret | `<SECRET_KEY>` |

---

## Integration Rules

- **Before each session:** Read `references/learned-fixes.md` to check known error patterns
- **Step 3 decision:** Ask user preference (local setup vs. research only)
- **If local setup chosen:** Execute Steps 3.1–3.6 before research
- **If connection test fails:** Automatically trigger Phase 1 diagnosis (no manual invocation)
- **After error fix succeeds:** Continue research from interrupted point
- **Security check:** Every config change, every Phase 3 approval

---

## Workflow Decision Tree

```
USER INPUT
(endpoint URL, GitHub URL, server name, or "review" command)
        │
    ┌───┴───────────────────────┐
    │                           │
    ▼                           ▼
AUDIT COMMAND              RESEARCH/SETUP
("Review project",         (server info)
 "Is this ready?")              │
    │                           ├─ Type A: Remote
    │                           ├─ Type B: GitHub
    ▼                           └─ Type C: Name
PROJECT AUDIT
(PASS/FAIL/WARN)
└─ Output report
   Save decision     Type A Path:
                     ├─ TLS verify
                     ├─ Detect auth
                     └─ Test connection
                          │
                     Type B Path:
                     ├─ Ask: Local/Remote/Research?
                     │
                     ├─ If Local (Option 1):
                     │  ├─ Clone
                     │  ├─ Install
                     │  ├─ Extract config
                     │  └─ Test connection
                     │
                     └─ If Research (Option 3):
                        └─ Skip setup

                    (all paths converge)
                          │
                          ▼
                     FILL ATTRIBUTES
                     (logic enforcement)
                          │
                          ▼
                     SECURITY SCORE
                     (optional)
                          │
                          ▼
                     SAVE REPORT
                     (CSV + Markdown)
```

---

## Summary

**Unified MCP skill** — single optimized execution path for research, deployment, error recovery, and project auditing. Consolidates all four skills (mcp-researcher, attribute-researcher, error-handling, project-reviewer) into one unified, fast, token-efficient implementation. Every workflow embedded in one SKILL.md. All functionality, zero redundancy.

- ✅ 8-step research workflow
- ✅ 3 input type handling
- ✅ Conditional local setup
- ✅ 4 connection methods
- ✅ 7-phase inline error recovery
- ✅ Evidence-backed attributes
- ✅ 10-attribute security scoring
- ✅ Project compliance audit
- ✅ Skill 2.0 self-learning
- ✅ Security mandate enforced
- ✅ **FAST & EFFICIENT** — Single file, optimized paths, minimal token consumption
