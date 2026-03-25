---
name: mcp-researcher
user-invocable: true
description: >
  Research, verify, document, and security-score every attribute of an MCP (Model Context Protocol)
  server. Use when user provides MCP server name, URL, or GitHub repo. Includes built-in local setup
  (clone + install when needed), error recovery, and security scoring. Outputs fully evidenced
  structured profile with security confidence score.
---

# MCP Server Researcher + Local Setup + Error Recovery

> Full workflow diagram: see `workflow.md` in this directory.

**All-in-one MCP server research skill:** research attributes, conditionally set up locally (clone+install if `Deployment: Local=Yes`), diagnose+fix errors inline, score security, and generate CSV reports. Never assumes — always verifies from priority sources. When in doubt, marks No.

## Feature Overview

✓ Three input types: Remote endpoint / GitHub URL / Server name (auto-detect)
✓ Protocol version verification before research
✓ Conditional local setup: Only clone+install if user wants local deployment
✓ Four connection methods: STDIO / Published Package / Docker / Remote
✓ Configurable paths (report save + clone paths, stored for future)
✓ Logic enforcement: STDIO↔Local, HTTP/SSE↔Remote auto-set
✓ Built-in error recovery: 7-phase diagnosis + auto-fix on connection fail
✓ 10-attribute security scoring (0–53 scale)
✓ CSV report output with evidence backing
✓ Skill 2.0 self-learning (learned-fixes.md)

---

## SECURITY MANDATE — Always Enforced

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

```
"Research the GitHub MCP server"
"Get full report on the Slack MCP server"
"Document this MCP server: https://github.com/org/repo"
"What does the Linear MCP server support?"
"Set up the GitHub MCP server locally"                    ← triggers local setup
"Clone and get the Slack MCP running"                     ← triggers local setup
```

---

## Workflow: 8 Steps (Research → Conditional Local Setup → Error Recovery)

### Step 0 — Configuration & Input Classification

**Path Configuration (Simple 2-Step Process):**

**Step 0a - First Time (or if no saved path):**
```
Where to save MCP reports?

Open folder picker to choose/create location:
  [Open Folder Picker Button]

User selects folder → Skill uses that path for this session
```

**Step 0b - Second Time (after first use):**
```
Saved path: /Users/yourname/Desktop/MCP_reports

Make this your default?
  [Yes - Save as default]
  [No - Choose different path]

If Yes → Saved to .claude/settings.json for future use
If No  → Open folder picker to choose new path
```

**Step 0c - Future Sessions (if default exists):**
```
Using saved default: /Users/yourname/Desktop/MCP_reports

Use this path?
  [Yes - Continue]
  [No - Choose different path]

If Yes → Use saved default
If No  → Open folder picker for one-time different path (not saved)
```

**Same Flow for Clone Location:**
- Same simple 2-step process
- Open folder picker → Choose location
- Second time: Ask to save as default

**Benefits:**
- No confusing options
- Simple: Open picker → Choose → Done
- Second use: Save as default option
- Clean & intuitive

**Classify input:**

| Input Type | Signal | Pre-Set | Next |
|-----------|--------|---------|------|
| **Remote Endpoint** | `https://`, `/sse` or `/mcp` path | Transport: HTTP/SSE or StreamableHttp; Deployment: Remote=Yes | TLS verify, skip local setup |
| **GitHub URL** | Contains `github.com` | Transport: STDIO=Yes; Deployment: Local=Yes | Offer local setup, test STDIO, check for remote |
| **Server Name Only** | Plain text (e.g., "GitHub MCP") | None | Search vendor docs, present options |

### Step 1 — Protocol Version Verification

Run before filling attributes:
1. Locate SDK version (uv.lock → package-lock.json → pyproject.toml → package.json → requirements.txt)
2. Extract pinned version, fetch latest from PyPI/npm
3. If outdated by 1+ minor: ask user (Option 1: Update / 2: Keep / 3: Cancel)
4. Map to protocol version (mcp ≥1.24=2025-11-25, 1.15–1.23=2025-06-18, etc.)
5. Record both SDK and protocol in report

### Step 2 — Type A: Remote Endpoint URL

1. Extract protocol from path (`/sse` → HTTP/SSE, `/mcp` → StreamableHttp)
2. Verify TLS (curl checks, document results)
3. Test connection: `curl -s -o /dev/null -w "%{http_code}" <url>`
4. Detect auth (WWW-Authenticate header, /.well-known/oauth, vendor docs)
5. Ask config location: Option 1: Global (~/.claude/settings.json) / 2: Project
6. Build config with placeholders, guide user to edit directly
7. **Test initialize request** — if fails: → Error Recovery Phase 1

### Step 3 — Type B: GitHub Repository URL

**Check Deployment preference first:**
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

**If Option 3 (Research Only):**
Skip local setup, proceed to Step 4 (attribute research).

**If Option 2 (Check Remote):**
Look for remote endpoint in README, offer as alternative to local setup.

### Step 4 — Type C: Server Name Only

1. Search vendor documentation for remote endpoint
2. Search for GitHub repository
3. If both found: ask (Option 1: Remote / 2: GitHub local / 3: Help me decide)
4. Continue with chosen Type A or B workflow

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

**Source Priority (highest trust first):**
1. Official vendor documentation
2. GitHub repository & source code
3. PyPI / npm package page
4. MCP specification documentation

**Every Yes must be backed by evidence** (quote or file path).

### Step 6 — Basic Information

Extract from vendor docs + GitHub:
- Name, Description, Category, Git Repo Version

### Step 7 — Calculate Security Score (Optional)

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

### Step 8 — Save Report

Location: User-configured path
Format: CSV only
Filename: `<servername>.csv`
Columns: Category | Attribute | Status/Yes/No

---

## Local Setup Details (Conditional: Only if Option 1 Selected)

### Connection Methods

| Method | Indicator | Setup | Deps Install |
|--------|-----------|-------|--------------|
| **STDIO** | stdin/stdout, subprocess mention | Clone + install locally | Python: venv+pip / Node: npm+build |
| **Published Package** | PyPI/npm registry, pip/npx | No clone needed | `pip install <pkg>` or `npx <pkg>` |
| **Docker** | Dockerfile present | Clone + docker build | `docker build -t <name> .` |
| **Remote** | vendor URL, no local setup | Skip all setup | N/A |

### Dependency Installation

| Language | Command | Verification |
|----------|---------|--------|
| Python | `python3 -m venv .venv && source .venv/bin/activate && pip install -r requirements.txt` | `python -c "import mcp"` |
| Node/TS | `npm install && npm run build` (if build in scripts) | Check dist/ exists |
| Docker | `docker build -t <repo-name> .` | `docker images \| grep <repo-name>` |
| uv | `uv sync` | `uv run python -c "import mcp"` |
| Published | `pip install <package-name>` or `npx <package>` | Version check or --help |

### Configuration Extraction

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

---

## Error Recovery: 7 Phases (Inline on Connection Test Fail)

If connection test fails at Step 3.6 or Step 2, automatically trigger error recovery:

### Phase 1 — Capture Error
Read error message from failed test, classify by category (Dependency/Config/Auth/Transport/Protocol/Runtime/Environment)

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

## Error Categories (Phase 1)

| # | Category | Signals |
|---|----------|---------|
| 1 | **Dependency/Install** | ModuleNotFoundError, ImportError, npm ERR!, pip failed, command not found |
| 2 | **Configuration** | Invalid config, JSON error, wrong path in settings.json |
| 3 | **Authentication** | 401, 403, token expired, missing env var |
| 4 | **Transport/Process** | Connection refused, ECONNREFUSED, EPIPE, spawn error |
| 5 | **Protocol Version** | Unsupported version, capability negotiation failed |
| 6 | **Runtime/Crash** | RuntimeError, TypeError, unhandled exception |
| 7 | **Environment/Path** | Python version mismatch, venv not active, binary not on PATH |

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

## Skill Handoff Map

```
mcp-researcher (all-in-one entry point)
  ├─ Input: Server name / URL / GitHub repo (user provides)
  ├─ Step 3 (Option 1): Conditional local setup (clone + install)
  │  ├─ Clone repo to configured path
  │  ├─ Install dependencies (Python/Node/Docker/uv/Published)
  │  ├─ Extract command/args/env vars
  │  └─ Test connection
  ├─ Built-in: error-recovery phases 1–6 (inline on connection fail)
  ├─ Calls: attribute-researcher (if needed for attribute filling)
  ├─ Writes: learned-fixes.md (Skill 2.0 on successful error fix)
  └─ Output: CSV report saved to user-configured path
```

---

## Integration Rules

- **Before each session:** Read `references/learned-fixes.md` to check known error patterns
- **Step 3 decision:** Ask user preference (local setup vs. research only)
- **If local setup chosen:** Execute Steps 3.1–3.6 before research
- **If connection test fails:** Automatically trigger Phase 1 diagnosis (no manual invocation)
- **After error fix succeeds:** Continue research from interrupted point
- **Security check:** Every config change, every Phase 3 approval

---

## When to Skip Local Setup

✅ **Always skip** when:
- User selects "Research only" (Option 3 at Step 3)
- Input is Remote Endpoint (Type A)
- Server is remote-hosted (Deployment: Remote = Yes)
- User already has server running elsewhere

✅ **Offer as alternative** when:
- GitHub URL provided but user wants remote endpoint
- Both local and remote options available

---

## Summary

Unified all-in-one MCP server research skill: research attributes, conditionally set up locally
(only if user chooses Option 1), diagnose+fix errors inline, calculate security score, and generate
CSV reports. Skips repo-clone logic entirely for remote/research-only modes. All functionality
consolidated into single skill for efficiency and speed.
