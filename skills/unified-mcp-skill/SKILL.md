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
✓ **Protocol version verification** before research (Priority: Framework > SDK)
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

## 🔐 Security Mandate — Always Enforced

**CRITICAL RULES (No exceptions):**

1. **Never ask for credentials in chat** — Absolute rule, all authentication via file editing only
2. **Always use `<PLACEHOLDER>` values** in all config examples (never pre-fill credentials)
3. **Direct user to edit files directly** — Credentials routed to filesystem only, never transmitted via chat
4. **Immutable pattern:** Show file path → User opens → User pastes credentials → User restarts

**If user accidentally pastes credential in chat:**
```
SECURITY ALERT: Do not share credentials here. Treat as compromised.
→ Revoke immediately at https://[vendor-docs]
→ Generate new credential
→ Add new value directly to: [file-path]
→ Do NOT paste here
```

**Never store credentials in:**
- README.md files
- CSV reports
- .env files committed to git
- Config files tracked by git
- Any committed or cached files

---

## Usage

**Invoke with:** `/start-mcp`

```
RESEARCH
/start-mcp "Research the GitHub MCP server"
/start-mcp "Get full report on the Slack MCP server"
/start-mcp "Document this MCP server: https://github.com/org/repo"
/start-mcp "Analyze and score this MCP: https://mcp.example.com"

LOCAL SETUP
/start-mcp "Set up the GitHub MCP server locally"
/start-mcp "Clone and get the Slack MCP running"

ERROR RECOVERY (automatic)
/start-mcp "The MCP server won't connect"
/start-mcp [paste error or stack trace]

PROJECT AUDIT
/start-mcp "Review the project"
/start-mcp "Is this ready to commit?"
```

---

## Unified Workflow: 8 Steps + Conditional Local Setup + Inline Error Recovery

### Step 0 — Configuration & Input Classification

**First-time path configuration:**
- Ask: "Where to save MCP reports?"
- Ask: "Clone location for local servers?" (default: `~/MCP_repos/`)
- Save to `.claude/settings.json` for future sessions

**Classify input as one of 3 types:**

| Type | Signal | Transport | Next Step |
|------|--------|-----------|-----------|
| **Remote Endpoint** | `https://`, path `/sse` or `/mcp` | HTTP/SSE or StreamableHttp | TLS verify, skip local setup |
| **GitHub URL** | Contains `github.com` | STDIO | Ask: Local setup or research only? |
| **Server Name** | Plain text (e.g., "GitHub MCP") | Detect | Search vendor + GitHub |

---

### Step 1 — Protocol Version Verification

**CRITICAL:** Run BEFORE filling attributes. Framework priority takes absolute precedence.

**Source Priority (use first match):**
1. Check dependencies for **FastMCP** (Python) → Use FastMCP mapping
2. Check dependencies for **@modelcontextprotocol/sdk** (TypeScript) → Use SDK mapping
3. Check base **mcp** SDK (Python) → Use mcp mapping

**Framework Mapping (FastMCP, Python):**
- `fastmcp ≥0.4.x` → Protocol `2025-06-18`
- `fastmcp 0.3.x` → Protocol `2025-03-26`
- `fastmcp <0.3` → Protocol `2024-11-05`

**SDK Mapping (Python mcp):**
- `mcp ≥1.24` → Protocol `2025-11-25`
- `mcp 1.15–1.23` → Protocol `2025-06-18`
- `mcp <1.15` → Protocol `2024-11-05`

**SDK Mapping (TypeScript @modelcontextprotocol/sdk):**
- `sdk ≥1.12` → Protocol `2025-11-25`
- `sdk 1.8–1.11` → Protocol `2025-06-18`
- `sdk <1.8` → Protocol `2024-11-05`

Record: Framework name, version, protocol version (with source).

---

### Step 2 — Remote Endpoint URL Path

1. Extract protocol from path: `/sse` = HTTP/SSE, `/mcp` = StreamableHttp
2. Verify TLS: Run GET and POST tests (document results)
3. Test connection: Probing both HTTP methods
4. Detect auth method: Check WWW-Authenticate header, /.well-known/oauth, vendor docs
5. Ask config location: Global (~/.claude/settings.json) or Project (.claude/settings.json)?
6. Build config **with `<PLACEHOLDER>` values only** — direct user to edit file directly
7. Test initialize request — on fail → Error Recovery Phase 1

---

### Step 3 — GitHub Repository URL (Conditional Local Setup)

**Ask deployment preference (1/2/3):**

```
How do you want to run this server?

1. Locally (clone + install + configure)
   → Full setup: clone, install deps, test connection

2. Use remote endpoint (if available)
   → Check README for remote endpoint

3. Just research attributes (no setup)
   → Skip local setup, proceed to research only

Select (1/2/3):
```

**If Option 1 (Local Setup) — Execute sub-steps:**

| Sub-step | Action | Verify |
|----------|--------|--------|
| 3.1 | Clone repo to configured path | Dir exists, expected files present |
| 3.2 | Determine connection method | Identify STDIO/Docker/Published from README |
| 3.3 | Get user approval for install | Show deps + env vars, wait for "yes" |
| 3.4 | Install dependencies | Language-specific (Python/Node/Docker) |
| 3.5 | Extract configuration | Read command, args, env vars from README + code |
| 3.6 | Test connection | Send initialize JSON-RPC, verify response |

**Connection Methods (Step 3.2):**

| Method | Indicator | Setup |
|--------|-----------|-------|
| **STDIO** | stdin/stdout mentions in README | Clone + install locally |
| **Published Package** | PyPI/npm package, pip/npx refs | No clone, use published package |
| **Docker** | Dockerfile in repo | Clone + docker build |
| **Remote** | Vendor URL endpoint | Skip all setup |

**Dependency Installation:**

| Language | Command | Verify |
|----------|---------|--------|
| Python | `python3 -m venv .venv && pip install -r requirements.txt` | `python -c "import mcp"` |
| Node.js | `npm install && npm run build` | Check `dist/` exists |
| Docker | `docker build -t <repo-name> .` | `docker images \| grep <repo-name>` |
| uv | `uv sync` | `uv run python -c "import mcp"` |

**If Option 3 (Research Only):**
Skip local setup, proceed to attribute research.

---

### Step 4 — Server Name Only

1. Search vendor documentation for remote endpoint
2. Search for GitHub repository
3. If both found: Ask user (Remote / GitHub / Help me decide)
4. Continue with chosen path (Type A or B)

---

### Step 5 — Attribute Filling with Logic Enforcement

**Transport ↔ Deployment Auto-Rules:**
- STDIO=Yes → Auto-set: Local=Yes
- HTTP/SSE OR StreamableHttp=Yes → Auto-set: Remote=Yes

**Mutually Exclusive Attributes (mark exactly ONE as Yes):**

| Attribute | Options |
|-----------|---------|
| Distribution | Official OR Community |
| Pricing | Free OR Paid |
| Hosting Provider | Choose: SaaS / GitHub / GitLab / Bitbucket / SourceHut |
| Transport Protocol | Choose: STDIO / HTTP/SSE / StreamableHttp |
| Tools Operations | Choose HIGHEST: Read-only / R+Update / R+Update+Delete |

**Evidence-Backed Attributes:**

Every "Yes" must have source + exact quote or file path.

| Attribute | Type | Source |
|-----------|------|--------|
| Name | Text | GitHub or vendor docs |
| Description | Text | README (3–4 lines, specific capabilities) |
| Category | Choice | mcp.md or vendor classification |
| Git Repo | URL | GitHub link (or N/A) |
| Git Version | Tag | Latest release (format: v1.2.3 + all versions) |
| Authentication | Choice | Vendor docs or source code (env vars, decorators) |
| Transport | Choice | Server startup code analysis |
| Deployment | Choice | Based on transport + availability |

---

### Step 6 — Tools & Capabilities

Extract from source code:
- Find `@mcp.tool()` decorators
- Group by logical category
- List non-read-only operations (write/delete)
- Check capabilities: Tools/Resources/Prompts/Sampling

**Format (CSV cell):**
```
Category Name
    •    tool_name – Description

Another Category
    •    tool_name – Description
```

---

### Step 7 — Basic Information

Extract from vendor docs + GitHub:
- Name, Description, Category, Git Repo Version

---

### Step 8 — Calculate Security Score (Optional)

```
Include CCI Security Score?
1. Yes, calculate
2. No, skip
3. Cancel
```

**10 Attributes (max 53 total):**

| Attribute | Max | Scoring |
|-----------|-----|---------|
| Protocol Version | 5 | Latest=5, 1 behind=3, older=1 |
| Distribution | 5 | Official=5, Community=3 |
| Pricing | 3 | Free=3, Paid=1 |
| Authentication | 5 | OAuth 2.1=5, Bearer/PAT=3, Token=2, None=1 |
| Hosting | 5 | SaaS=5, GitHub=3, Self-hosted=1 |
| TLS | 5 | 1.3=5, 1.2=3, STDIO N/A=3, None=1 |
| Transport | 5 | StreamableHttp=5, HTTP/SSE=3, STDIO=2 |
| Tools Operations | 5 | R+U+D=5, R+U=3, Read-only=1 |
| Deployment | 3 | Remote=3, Local/Container=2, Hybrid=1 |
| Capabilities | 3 | 4+ types=3, 2-3=2, 1=1 |

---

### Step 9 — Save Report

**Location:** User-configured path
**Format:** CSV (3 columns: Category, Attribute, Status)
**Filename:** `<servername>.csv`
**CSV Rules:** Use `csv.QUOTE_ALL` for proper multiline handling

---

## Error Recovery: 7 Phases (Inline on Connection Test Fail)

If connection test fails at Step 3.6 or 2.7, automatically trigger:

### Phase 1 — Capture Error
Classify by category:
1. **Dependency** — ModuleNotFoundError, ImportError, pip/npm errors
2. **Configuration** — Invalid config, JSON errors, wrong path
3. **Authentication** — 401, 403, token expired, missing env var
4. **Transport** — Connection refused, ECONNREFUSED, spawn error
5. **Protocol** — Unsupported version, negotiation failed
6. **Runtime** — RuntimeError, TypeError, exception
7. **Environment** — Python version mismatch, venv not active, PATH issue

### Phase 2 — Diagnose
Run in order (stop at first failure):
1. Runtime version check (`python --version`, `node --version`)
2. Config syntax validation (parse JSON, test paths)
3. Dependencies importable (`python -c "import mcp"`)
4. Server start directly (test command + args)
5. Initialize JSON-RPC test (expect result + protocolVersion)
6. Claude Code logs (check MCP*.log files)

### Phase 3 — Present Fix Plan
Show category + root cause + ordered steps + risk level. Ask: "Proceed?"

### Phase 4 — Execute Fixes
| Category | Command |
|----------|---------|
| Dependency (Python) | `python3 -m venv .venv && pip install -r requirements.txt` |
| Dependency (uv) | `uv sync` |
| Dependency (Node) | `npm install && npm run build` |
| Config | Show diff, user edits directly |
| Auth | Direct user to settings file, use `<KEY>` format |
| Transport | Restart Claude Code |

**Every step requires approval:** "Proceed with Step N? (yes/skip/cancel)"

### Phase 5 — Verify Connection
Test initialize again. Success → continue research. Fail → re-diagnose.

### Phase 6 — Record Pattern (Skill 2.0)
If fix succeeds, append to `references/learned-fixes.md`:
```
## [Error Title]
**Signals:** [symptoms]
**Cause:** [root cause]
**Fix:** [step-by-step]
**Verify:** [how to test]
**Date:** YYYY-MM-DD | **Category:** [1–7]
```

---

## Project Audit: Compliance Review

Triggered by: `"Review the project"`, `"Is this ready to commit?"`, `"Check requirements"`

### Audit Checks (PASS/FAIL/WARN)

**Project Goal Alignment:**
- [ ] Project follows MCP research & deployment workflow
- [ ] Skills support research, setup, error handling, audit
- [ ] Output reports saved to correct locations

**Enterprise Security:**
- [ ] No credentials in chat (all via file editing)
- [ ] No hardcoded secrets in code
- [ ] All config examples use `<PLACEHOLDER>` syntax
- [ ] Security mandates enforced at every step

**Confidentiality:**
- [ ] No API keys in committed files
- [ ] No credentials in README or reports
- [ ] Settings files added to .gitignore
- [ ] Credentials routed to filesystem only

**Consistency:**
- [ ] All skills enforce same security rules
- [ ] Unified error recovery approach
- [ ] Consistent attribute documentation

**Completeness:**
- [ ] All workflows have approval gates
- [ ] Error recovery covers all 7 categories
- [ ] Security scoring validated against CCI spec

### Audit Output

```
PROJECT REVIEW — 2026-03-26
═══════════════════════════════════════════════════════════════

✅ PASS  Project goal alignment
✅ PASS  Enterprise security requirements (8/8)
⚠️  WARN  Confidentiality: 3/4 (check .gitignore)
✅ PASS  Cross-skill consistency
✅ PASS  Completeness verification

Overall: PASS (review 1 WARN item before commit)
═══════════════════════════════════════════════════════════════
```

---

## Parallel Search & Data Collection (Critical Learning)

**CRITICAL:** Execute Step 0.5 with 5 CONCURRENT threads (not sequentially).

### Why Parallel Matters

Sequential research misses interdependencies. Endpoint found in Thread 3 reveals framework for Thread 4, which determines protocol for Thread 1. Parallel execution catches everything simultaneously.

### Step 0.5 — 5 Concurrent Threads

**Thread 1: GitHub README Full Read**
- Read ENTIRE README (all sections)
- Search: endpoint, https://, remote, hosted, cloud
- Extract ALL external URLs
- Look for deprecation notices (may point to new versions)

**Thread 2: Repository File Search**
- Files: .env.example, config.json, setup.md, docs/
- Grep: "https://", "endpoint", "url", "host" patterns
- API paths: /api/v2, /v1, /mcp
- Comments with example URLs

**Thread 3: Endpoint Pattern Probing (Multi-Method)**
- Test 1: GET request (check HTTP/SSE)
  - `curl -I https://api.{vendor}.com/v2/mcp`
  - 200/401/403 → exists; 404 → not GET-based

- Test 2: POST with JSON-RPC (check StreamableHttp)
  - `curl -X POST https://.../ -d '{"jsonrpc":"2.0",...}'`
  - 401/403 → exists; 404 → doesn't exist

- Test 3: Response format
  - JSON-RPC response → Real MCP endpoint
  - Vendor API error → Placeholder endpoint
  - 404 both → Doesn't exist

**Thread 4: Dependency Extraction**
- Priority 1: FastMCP version (if present)
- Priority 2: Framework version (Langchain, etc.)
- Priority 3: Base SDK version (mcp, @modelcontextprotocol/sdk)
- Check for HTTP frameworks: FastAPI, Express, Flask

**Thread 5: Vendor Compliance & Security**
- HIPAA, GDPR, SOC 2, FedRAMP badges
- GitHub issues for compliance
- Vendor blog/documentation
- BAA (Business Associate Agreement) availability

### After All Threads Complete

**Checkpoint:** Verify all complete before proceeding:
- ✅ Full README read?
- ✅ All repo files searched?
- ✅ Endpoint patterns probed?
- ✅ Framework version identified?
- ✅ Compliance documented?

If ANY incomplete → Continue until done.

---

## Key Learnings from Research & Verification

### Learning 1: Framework Priority

Framework version determines protocol, NOT base SDK.

**Check order:**
1. FastMCP present? → Use FastMCP mapping
2. Framework present? → Use framework mapping
3. Otherwise → Use base SDK mapping

**Example:** Telnyx with FastMCP 0.4.1 + mcp 1.3.0 → 2025-06-18 (FastMCP), not 2025-03-26 (mcp).

### Learning 2: Endpoint Verification Strategy

README claims ≠ actual implementation. "Remote MCP Now Available" ≠ endpoint is functional.

**Three-Step Verification:**

Step 1: GET request (HTTP/SSE detection)
```
curl -I https://api.vendor.com/v2/mcp
Result: 404 → not SSE endpoint
```

Step 2: POST with JSON-RPC (StreamableHttp detection)
```
curl -X POST https://api.vendor.com/v2/mcp \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0",...}'
Result: 401 → endpoint recognized, requires auth
        404 → endpoint doesn't exist
```

Step 3: Response format analysis
- POST returns JSON-RPC → Real MCP endpoint
- POST returns vendor API error → Placeholder (not MCP yet)
- Both GET & POST return 404 → Endpoint doesn't exist

**Critical Distinction:**
- ❌ WRONG: 404 on GET → Assume endpoint doesn't exist
- ✅ RIGHT: GET=404 but POST=401 → Endpoint exists but not SSE (could be StreamableHttp)

**When README Claims Remote But Endpoint Doesn't Work:**
1. Check documentation links → 404? Not deployed yet
2. Check repository status → Archived? Outdated claim
3. Verify actual implementation → Can you call MCP methods?

If endpoint returns vendor API errors (not JSON-RPC) → Mark as: Endpoint URL = N/A (not functional yet).

---

## Common Mistakes to Avoid

✗ Not verifying SDK version before research
✗ Connecting to wrong endpoint/repo
✗ Mixing STDIO/Local or HTTP/Remote incorrectly
✗ Not storing user paths for future use
✗ Installing deps without user approval
✗ Asking user to paste credentials during diagnosis
✗ Marking multiple values in mutually exclusive fields
✗ **Assuming 404 means endpoint doesn't exist** → test POST too
✗ **Trusting README claims without verification** → may not be deployed
✗ **Skipping endpoint probing** → use GET + POST to distinguish types
✗ **Not checking response format** → distinguishes real MCP vs placeholder

---

## Credential Placeholder Map

| Auth Type | Placeholder |
|-----------|-------------|
| API Token/Key | `<API_KEY>` |
| Bearer Token | `<BEARER_TOKEN>` |
| Personal Access Token | `<PERSONAL_ACCESS_TOKEN>` |
| OAuth Client ID | `<OAUTH_CLIENT_ID>` |
| OAuth Client Secret | `<OAUTH_CLIENT_SECRET>` |
| Generic Secret | `<SECRET_KEY>` |

---

## Integration Rules

- **Before each session:** Read `references/learned-fixes.md`
- **Step 3 decision:** Ask user preference (local vs. research)
- **If local setup:** Execute Steps 3.1–3.6 before research
- **If connection fails:** Auto-trigger Phase 1 diagnosis
- **After error fix:** Continue research from interrupted point
- **Security check:** Every config change requires approval

---

## Report Format: CSV Structure

**Output Location:** `~/Desktop/MCP_reports/<servername>.csv` (user-configured)

**Format:** CSV with exactly 3 columns: `Category, Attribute, Status`

**CSV Requirements:**
- Use Python's `csv.writer` with `quoting=csv.QUOTE_ALL`
- All cells quoted, even Yes/No values
- Newlines in multiline cells preserved via proper CSV escaping
- **Output CSV ONLY — no markdown (.md) file**

**Row Order (Exact Sequence):**

```
MCP Info
├── Description (3–4 lines with specific capabilities)
├── Git Repo Version (v1.2.3 format + all versions list)
├── Category (one of 4: File Management / Developer Tools / Data Retrieval / Productivity)
├── GitHub Repository (URL or N/A)
└── Endpoint URL (URL or N/A)

Distribution Type (Official / Community)
MCP Protocol Version (2025-11-25 / 2025-06-18 / 2025-03-26 / 2024-11-05)
Pricing (Free / Paid)
Hosting Provider (SaaS / GitHub / GitLab / Bitbucket / SourceHut)
Authentication (OAuth 2.1 AuthCode / OAuth 2.1 Client Creds / Bearer / PAT / API Token)
Data Protection (TLS 1.3 / TLS 1.2 / Lower versions or no encryption)
Transport Protocol (STDIO / HTTP/SSE / StreamableHttp / FastAPI)
Tools Operations (Read-only / R+Update / R+Update+Delete)
Deployment Approach (Local / Container / Remote)
Capabilities (Tools / Resources / Prompts / Sampling)
Capabilities - Tools (detailed_info with categories and bullet points)
Non-Read-Only Tools (detailed_info with categories OR "None — all tools are read-only")
Compliance & Certifications (HIPAA / GDPR / SOC 2 / FedRAMP)
```

**Example Rows:**

Row 1 (Basic Info):
```csv
MCP Info,Description,"Telnyx Python MCP Server for managing telephony, messaging, and AI assistant workflows via Claude and other MCP clients. Provides 50+ tools across SMS/MMS messaging, voice call control, AI assistant creation, cloud storage (S3-compatible), embeddings, webhook management, and integration secrets handling."
```

Row 2 (Version):
```csv
MCP Info,Git Repo Version,"v0.1.2 [ Referred from Tags/Releases ] | All versions: v0.1.2, v0.1.1, v0.1.0"
```

Row 3 (Tools detailed_info):
```csv
Capabilities - Tools,detailed_info,"AI Assistants
    •    create_assistant – Create new AI assistant with custom instructions
    •    list_assistants – List all existing AI assistants
    •    get_assistant – Get details for a specific AI assistant
    •    update_assistant – Update an existing assistant's properties
    •    delete_assistant – Delete an AI assistant

Call Control
    •    make_call – Make an outbound phone call to a destination number
    •    hangup – Hang up an active phone call
    •    transfer – Transfer an active call to a new destination
    •    playback_start – Play audio file or URL during an active call
    •    playback_stop – Stop audio playback on an active call
    •    send_dtmf – Send DTMF (touch-tone) signals during a call
    •    speak – Convert text to speech and play during a call

Messaging
    •    send_message – Send SMS or MMS messages to recipients
    •    get_message – Get details for a specific message
    •    sms_conversations – Access and view ongoing SMS conversations (resource)"
```

Row 4 (Non-Read-Only Tools):
```csv
Non-Read-Only Tools,detailed_info,"AI Assistants
    •    create_assistant – Creates new AI assistant (write operation)
    •    update_assistant – Modifies existing assistant (write operation)
    •    delete_assistant – Removes assistant from system (delete operation)
    •    start_assistant_call – Initiates outbound call (write operation)

Call Control
    •    make_call – Creates new call leg (write operation)
    •    hangup – Terminates call (delete operation)
    •    transfer – Modifies call routing (write operation)

Messaging
    •    send_message – Sends SMS/MMS (write operation)"
```

**Formatting Rules (Critical):**

1. **Category headers** — Plain text, no brackets, no special formatting
2. **Bullets** — Exactly 4 spaces before `•`, exactly 4 spaces after `•`
3. **Connector** — Use `–` (en-dash, NOT hyphen) between tool name and description
4. **Blank lines** — Always separate category blocks with blank line
5. **Tool names** — Use actual registered function names (not generic labels)
6. **Descriptions** — Concise, action-oriented (start with verb: "Create", "Get", "List", "Send")
7. **Multiline cells** — CSV escaping handles newlines, no manual escaping needed
8. **Attributes** — Every attribute must be Yes/No only (never "N/A", "Unknown", "Partial")

**Example Bad vs Good:**

❌ WRONG:
```
Capabilities - Tools,detailed_info,"- AI Assistants: create_assistant, list_assistants"
```

✅ RIGHT:
```
Capabilities - Tools,detailed_info,"AI Assistants
    •    create_assistant – Create new AI assistant
    •    list_assistants – List all existing AI assistants"
```

**Telnyx_MCP_Server.csv Example (Complete Structure):**

```csv
Category,Attribute,Status
MCP Info,Description,"Telnyx Python MCP Server for managing telephony, messaging, and AI assistant workflows via Claude and other MCP clients. Provides 50+ tools across SMS/MMS messaging, voice call control, AI assistant creation, cloud storage (S3-compatible), embeddings, webhook management, and integration secrets handling. Supports selective tool filtering and webhook receivers via ngrok integration."
MCP Info,Git Repo Version,"v0.1.2 [ Referred from Tags/Releases ] | All versions: v0.1.2, v0.1.1, v0.1.0"
MCP Info,Category,"Developer and Coding Tools"
MCP Info,GitHub Repository,"https://github.com/team-telnyx/telnyx-mcp-server"
MCP Info,Endpoint URL,"N/A"
Distribution Type,Official,Yes
Distribution Type,Community,No
MCP Protocol Version,2025-11-25,No
MCP Protocol Version,2025-06-18,Yes
MCP Protocol Version,2025-03-26,No
MCP Protocol Version,2024-11-05,No
Pricing,Free,Yes
Pricing,Paid,No
Hosting Provider,SaaS Vendor,No
Hosting Provider,3rd Party SaaS vendor,No
Hosting Provider,GitHub,Yes
Hosting Provider,GitLab,No
Hosting Provider,Bitbucket,No
Hosting Provider,SourceHut/Gitea/Gogs,No
Authentication,OAuth 2.1 - Authorization Code Flow,No
Authentication,OAuth 2.1 - Client Credentials Flow,No
Authentication,Bearer Token,No
Authentication,Personal Access Token,No
Authentication,API Token,Yes
Data Protection,TLS 1.3,No
Data Protection,TLS 1.2,No
Data Protection,Lower versions or no encryption,No
Transport Protocol,STDIO,Yes
Transport Protocol,HTTP/SSE,No
Transport Protocol,StreamableHttp,No
Transport Protocol,FastAPI,No
Tools Operations,Read-only operations,No
Tools Operations,Read-only and/or update operations,No
Tools Operations,Read-only update and/or delete operations,Yes
Deployment Approach,Local,Yes
Deployment Approach,Container,No
Deployment Approach,Remote,No
Capabilities,Tools,Yes
Capabilities,Resources,Yes
Capabilities,Prompts,No
Capabilities,Sampling,No
Capabilities - Tools,detailed_info,"AI Assistants
    •    create_assistant – Create a new AI assistant with custom instructions and configurations
    •    list_assistants – List all existing AI assistants
    •    get_assistant – Get details for a specific AI assistant
    •    update_assistant – Update an existing assistant's properties and configuration
    •    delete_assistant – Delete an AI assistant
    •    get_assistant_texml – Get TEXML configuration for an assistant
    •    start_assistant_call – Start an outbound call using an AI assistant

Call Control
    •    make_call – Make an outbound phone call to a destination number
    •    hangup – Hang up an active phone call
    •    transfer – Transfer an active call to a new destination
    •    playback_start – Play audio file or URL during an active call
    •    playback_stop – Stop audio playback on an active call
    •    send_dtmf – Send DTMF (touch-tone) signals during a call
    •    speak – Convert text to speech and play during a call
    •    list_call_control_applications – List call control applications
    •    get_call_control_application – Get details for a call control application
    •    create_call_control_application – Create a new call control application

Messaging
    •    send_message – Send SMS or MMS messages to recipients
    •    get_message – Get details for a specific message
    •    sms_conversations – Access and view ongoing SMS conversations (resource)

Phone Numbers
    •    list_phone_numbers – List all phone numbers on the account
    •    buy_phone_numbers – Purchase new phone numbers
    •    update_phone_numbers – Update phone number configurations
    •    list_available_numbers – Search for available phone numbers by area code or pattern

Connection Management
    •    list_connections – List voice connections configured on the account
    •    get_connection – Get details for a specific connection
    •    update_connection – Update connection configurations

Cloud Storage
    •    cloud_storage_create_bucket – Create a new cloud storage bucket
    •    cloud_storage_list_buckets – List all storage buckets across all regions
    •    cloud_storage_upload_file – Upload files to a bucket
    •    cloud_storage_download_file – Download files from a bucket
    •    cloud_storage_list_objects – List objects in a bucket
    •    cloud_storage_delete_object – Delete objects from a bucket
    •    cloud_storage_get_bucket_location – Get bucket location information

Embeddings & AI
    •    list_embedded_buckets – List existing embedded buckets
    •    scrape_and_embed_url – Scrape website URL and create embeddings for AI training
    •    create_file_embeddings – Create embeddings for custom files

Secrets Manager
    •    list_secrets – List integration secrets
    •    create_secret – Create new bearer or basic auth secrets
    •    delete_secret – Delete integration secrets

Webhooks
    •    webhook_configuration – Configure webhook receivers for Telnyx events
    •    webhook_management – Manage active webhooks via ngrok integration"
Non-Read-Only Tools,detailed_info,"AI Assistants
    •    create_assistant – Creates new AI assistant (write operation)
    •    update_assistant – Modifies existing assistant (write operation)
    •    delete_assistant – Removes assistant from system (delete operation)
    •    start_assistant_call – Initiates outbound call (write operation)

Call Control
    •    make_call – Creates new call leg (write operation)
    •    hangup – Terminates call (delete operation)
    •    transfer – Modifies call routing (write operation)
    •    playback_start – Initiates audio playback (write operation)
    •    playback_stop – Stops playback (write operation)
    •    send_dtmf – Sends signals to call (write operation)
    •    speak – Initiates TTS playback (write operation)

Messaging
    •    send_message – Sends SMS/MMS (write operation)

Phone Numbers
    •    buy_phone_numbers – Purchases numbers (write operation - financial)
    •    update_phone_numbers – Modifies number configuration (write operation)

Cloud Storage
    •    cloud_storage_create_bucket – Creates bucket (write operation)
    •    cloud_storage_upload_file – Uploads files (write operation)
    •    cloud_storage_delete_object – Removes objects (delete operation)

Secrets Manager
    •    create_secret – Creates secret entry (write operation)
    •    delete_secret – Removes secret (delete operation)

Webhooks
    •    webhook_configuration – Creates/modifies webhook (write operation)"
Compliance & Certifications,HIPAA,No
Compliance & Certifications,GDPR,No
Compliance & Certifications,SOC 2,No
Compliance & Certifications,FedRAMP,No
```

---

## Summary

**Unified MCP skill** — single optimized execution path for research, deployment, error recovery, and project auditing. Consolidates all functionality into one unified, fast, token-efficient implementation.

- ✅ 8-step research workflow
- ✅ 3 input type handling (Remote / GitHub / Name)
- ✅ Conditional local setup (user choice)
- ✅ 4 connection methods (STDIO / Package / Docker / Remote)
- ✅ 7-phase inline error recovery (auto-triggered)
- ✅ Evidence-backed attributes (source tracking)
- ✅ 10-attribute security scoring (CCI-based)
- ✅ Project compliance audit (PASS/FAIL/WARN)
- ✅ Skill 2.0 self-learning (learned-fixes.md)
- ✅ Security mandate enforced (credentials via file only)
- ✅ **FAST & EFFICIENT** — Single file, optimized paths, minimal tokens

---

**All workflows, instructions, and processes preserved. Zero functionality lost. Pure optimization.**
