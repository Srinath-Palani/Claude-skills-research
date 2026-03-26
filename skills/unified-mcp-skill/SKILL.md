---
name: unified-mcp-skill
user-invocable: true
description: >
  All-in-one unified MCP skill: research MCP servers, document attributes,
  conditionally set up locally, diagnose errors inline, and audit projects. Single optimized
  execution path for research, deployment, error recovery, and compliance auditing. Faster,
  fewer tokens, full functionality.
---

🔒 **CRITICAL SYNC RULE** (LOCKED)
> Whenever this file (SKILL.md) is updated → MUST also update `references/multi-server.md`
> See `references/SYNC_GUIDE.md` for verification checklist before committing.
> Both files are tightly coupled and must stay synchronized.

# Unified MCP Skill — Research, Deploy, Fix, Audit

**All-in-one MCP server management:** Research and document attributes, conditional local setup (clone+install), inline error recovery, and project compliance auditing — all in one unified skill with optimized execution paths.

---

## Core Capabilities

| Capability | Description | Invocation Signal |
|-----------|-------------|------------------|
| **MCP Research** | Research and document MCP servers | "Research the GitHub MCP server", "Analyze this MCP: https://..." |
| **Attribute Docs** | Fill every MCP attribute with evidence | "Document attributes for...", "Catalogue this MCP server" |
| **Local Setup** | Clone, install, configure MCP servers | "Set up this server locally", "Clone and run..." |
| **Error Recovery** | 7-phase inline diagnosis and auto-fix | Automatic on connection fail |
| **Project Audit** | Audit project and skills for compliance | "Review the project", "Is this ready to commit?" |
| **Multi-Server Research** | Parallel batch research + comparison of N servers | "Research these MCPs: X, Y, Z", "Compare GitHub and Slack MCPs" |

---

## Feature Overview

✓ **3 input types:** Remote endpoint / GitHub URL / Server name (auto-detect)
✓ **Protocol version verification** before research (Priority: Framework > SDK)
✓ **Conditional local setup** — only clone+install if user wants local deployment
✓ **4 connection methods** — STDIO / Published Package / Docker / Remote
✓ **Configurable paths** (report save + clone paths, stored for future)
✓ **Logic enforcement** — STDIO↔Local, HTTP/SSE↔Remote auto-set
✓ **7-phase inline error recovery** — automatic diagnosis on connection fail
✓ **Multi-server parallel research** — layered sub-agent batch, ~80% fewer tokens vs. inline
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
- `.claude/settings.json` (project-level)
- `~/.claude/settings.json` (global)

**For MCP config files (`.claude/settings.json` / `~/.claude/settings.json`):**
Always write `<PLACEHOLDER>` for any API key, token, or secret field.
Direct the user to open the file and paste the real value themselves.
Never write, suggest, or echo an actual key into these files — not even partially.

---

## Usage

**Invoke with:** `/unified-mcp-skill`

> 🔒 **LOCKED — Do NOT change this invocation command without explicit user approval.**

```
RESEARCH
/unified-mcp-skill "Research the GitHub MCP server"
/unified-mcp-skill "Get full report on the Slack MCP server"
/unified-mcp-skill "Document this MCP server: https://github.com/org/repo"
/unified-mcp-skill "Analyze and score this MCP: https://mcp.example.com"

LOCAL SETUP
/unified-mcp-skill "Set up the GitHub MCP server locally"
/unified-mcp-skill "Clone and get the Slack MCP running"

ERROR RECOVERY (automatic)
/unified-mcp-skill "The MCP server won't connect"
/unified-mcp-skill [paste error or stack trace]

PROJECT AUDIT
/unified-mcp-skill "Review the project"
/unified-mcp-skill "Is this ready to commit?"

MULTI-SERVER / BATCH
/unified-mcp-skill "Research these MCPs: GitHub, Slack, Linear"
/unified-mcp-skill "Compare the GitHub and Stripe MCP servers"
/unified-mcp-skill "Batch report on: [server1], [server2], [server3]"
```

---

## Unified Workflow: 7 Steps + Conditional Local Setup + Inline Error Recovery

### Step 0 — Configuration & Input Classification

**Path configuration — always ask, never assume:**

Report save path — ask every first use:
```
Where should MCP reports be saved?
Enter full path (e.g. /Users/you/Desktop/MCP_reports):
```

Clone path — ask every time user chooses local setup:
```
Where should this server be cloned?
Enter full path:
```
If a path was used before, offer to reuse:
```
Last used clone path: /Users/you/projects/mcp
[ 1 ] Use this path
[ 2 ] Enter a different path
[ 3 ] Save this as my default path (skip asking next time)
```
Only save as default if user explicitly selects option 3.

**Classify input to determine starting point — then ALWAYS find the other source too:**

| Input Given | Starting Point | Also MUST Find |
|-------------|---------------|----------------|
| **Remote Endpoint** | Probe endpoint (TLS, auth, transport) | Search GitHub for source repo |
| **GitHub URL** | Read repo (README, deps, tools) | Search for + probe remote endpoint |
| **Server Name** | Search both simultaneously | GitHub URL + remote endpoint |

**MANDATORY DUAL-SOURCE RULE:**
Regardless of input type, research is NEVER complete with only one source.
- Given GitHub URL → always search for remote endpoint (README, docs, curl probe)
- Given remote endpoint → always find GitHub repo
- Given server name → find both before proceeding
Run both searches in parallel via Step 0.5. Combine results using Report Generation Rules 2–5.

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

**SDK Mapping (Go github.com/modelcontextprotocol/go-sdk):**
- `go-sdk ≥1.12` → Protocol `2025-11-25`
- `go-sdk 1.4–1.11` → Protocol `2025-06-18` ← **COMMON MISS: Don't assume latest**
- `go-sdk <1.4` → Protocol `2024-11-05`

**CRITICAL VERIFICATION CHECKLIST (to prevent mistakes):**
Before marking protocol version as final:
```
□ Step 1: Extract exact version from dependency file (go.mod, requirements.txt, package.json)
□ Step 2: Cross-reference version against the mapping table above (DO NOT guess/assume)
□ Step 3: If version is between ranges, use LOWER protocol (e.g., v1.4.1 is <1.12 → use 2025-06-18)
□ Step 4: Document the exact version + mapping source (file + line number)
□ Step 5: Check learned-fixes.md for "Protocol Version Mapping Error" to avoid repeat mistakes
```

**Common Mistake Patterns:**
- ❌ WRONG: "Latest protocol must be v2025-11-25" (assumes without checking)
- ✅ RIGHT: "SDK v1.4.1 maps to 2025-06-18" (checked version table)
- ❌ WRONG: "Go SDK in go.mod but no mapping" (missing Go SDK mapping)
- ✅ RIGHT: "Go SDK v1.4.1 → Protocol 2025-06-18 per Go SDK mapping" (complete mapping)

Record: Framework name, exact version, protocol version, mapping source, verification timestamp.

---

### Step 2 — Remote Endpoint URL Path

1. Extract protocol from path: `/sse` = HTTP/SSE, `/mcp` = StreamableHttp
2. Verify TLS: Run GET and POST tests (document results)
3. Test connection: Probing both HTTP methods
4. Detect auth method: Check WWW-Authenticate header, /.well-known/oauth, vendor docs
5. Ask config location:
   ```
   Where should the MCP config be added?
   [ 1 ] Global   → ~/.claude/settings.json  (all projects)
   [ 2 ] Project  → .claude/settings.json    (this project only)
   ```
6. Build config **with `<PLACEHOLDER>` values only** — direct user to edit file directly
7. Test initialize request — on fail → Error Recovery Phase 1

---

### Step 3 — GitHub Repository URL (Conditional Local Setup)

**Ask deployment preference:**

```
How do you want to run this server?

[ 1 ] Locally (clone + install + configure)
      → Full setup: clone, install deps, test connection

[ 2 ] Use remote endpoint (if available)
      → Check README for remote endpoint

[ 3 ] Just research attributes (no setup)
      → Skip local setup, proceed to research only
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
3. If both found, ask:
   ```
   [ 1 ] Use remote endpoint
   [ 2 ] Use GitHub repository
   [ 3 ] Help me decide
   ```
4. Continue with chosen path

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
| Hosting Provider | Choose: SaaS / GitHub |
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
| Git Version | Tag | Latest only — Releases first, Tags second (format: v1.2.3) |
| Authentication | Choice | Vendor docs or source code (env vars, decorators) — see Authentication Detection Rules below |
| Transport | Choice | Server startup code analysis |
| Deployment | Choice | Based on transport + availability |

---

### Step 5.1 — Authentication Verification (Self-Learning Fix)

**CRITICAL:** Authentication is often MISSED. Use this checklist:

```
AUTHENTICATION VERIFICATION CHECKLIST:
□ Step 1: Search server.json for "TOKEN", "API_KEY", "SECRET", "BEARER"
□ Step 2: Search README for "authentication", "auth", "credentials", "token"
□ Step 3: Search .env.example for credential fields
□ Step 4: Search vendor docs for API access / token setup page
□ Step 5: If ANY credential found → mark ALL applicable auth types as Yes
□ Step 6: Check learned-fixes.md for "Authentication Fields Marked No" pattern
```

**Common Mistake Pattern (LEARNED):**
- ❌ WRONG: "No authentication fields found" (didn't search server.json)
- ✅ RIGHT: "BUILDKITE_API_TOKEN found in server.json → Bearer Token, PAT, API Token all = Yes"
- ❌ WRONG: "No auth in README, so Authentication = No" (incomplete search)
- ✅ RIGHT: "Check server.json, .env.example, config files, vendor docs"

**Why All Three Can Be "Yes":**
- Bearer Token = delivery method (HTTP header)
- Personal Access Token = credential type (user-specific)
- API Token = alternative naming (same credential)
All can coexist for same server.

---

### Step 5.2 — Tools Operations Verification (Self-Learning Fix)

**CRITICAL:** Tools Operations is MUTUALLY EXCLUSIVE. Choose ONE level only.

```
TOOLS OPERATIONS VERIFICATION CHECKLIST:
□ Step 1: Parse ALL tool names from documentation
□ Step 2: Look for patterns: create_*, update_*, delete_*, cancel_*, remove_*
□ Step 3: If ONLY read operations found → mark "Read-only" = Yes
□ Step 4: If read + create/update found → mark "Read+Update" = Yes
□ Step 5: If read + create/update + delete/cancel found → mark "Read+Update+Delete" = Yes
□ Step 6: Mark ONLY the HIGHEST level (not multiple levels)
□ Step 7: Check learned-fixes.md for "Tools Operations Violation" pattern
```

**Common Mistake Patterns (LEARNED):**
- ❌ WRONG: Mark both "Read+Update = Yes" AND "Read+Update+Delete = Yes" (violates exclusivity)
- ✓ RIGHT: Mark ONLY "Read+Update+Delete = Yes" (highest level)
- ❌ WRONG: "Tool names: create_build, cancel_build" → mark "Read+Update" (missed delete pattern)
- ✅ RIGHT: "Tool names show cancel_* pattern" → mark "Read+Update+Delete" (delete operations present)

**Cancel/Delete Pattern Recognition:**
- `cancel_*` → Delete operation
- `delete_*` → Delete operation
- `remove_*` → Delete operation
- `destroy_*` → Delete operation

---

### Step 5.3 — Deployment Approach Verification (Self-Learning Fix)

**CRITICAL:** Deployment Approach allows MULTIPLE selections (NOT mutually exclusive).

```
DEPLOYMENT APPROACH VERIFICATION CHECKLIST:
□ Step 1: Search for Dockerfile, docker.json, Dockerfile.local in repo
□ Step 2: Search server.json for "docker", "container", "run"
□ Step 3: Search for OCI registry references (ghcr.io, etc.)
□ Step 4: Check for PyPI/npm package (published package = Local possible)
□ Step 5: Check for STDIO support (stdin/stdout = Local)
□ Step 6: Check for vendor endpoint (= Remote)
□ Step 7: If multiple found → mark ALL applicable approaches
□ Step 8: Check learned-fixes.md for "Deployment Container Marked No" pattern
```

**Common Mistake Patterns (LEARNED):**
- ❌ WRONG: "Go binary exists, so Container = No" (missed Docker config)
- ✅ RIGHT: "Found Dockerfile + docker image + STDIO" → mark Local = Yes AND Container = Yes
- ❌ WRONG: "Check README only for deployment" (incomplete search)
- ✅ RIGHT: "Check README + server.json + Dockerfile + OCI registry"

---

### Step 6 — Tools & Capabilities

Extract from source code:
- Find `@mcp.tool()` decorators
- Group by logical category
- List non-read-only operations (write/delete)
- Check capabilities: Tools only

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

### Step 8 — Save Report

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
If fix succeeds, append to "Learned Fixes" section in SKILL.md:
```
### Error #[N]: [Error Title]

**Date:** YYYY-MM-DD | **Server:** [server-name] | **Severity:** [High/Medium/Low/CRITICAL]

**Signals:** [symptoms]

**Root Cause:** [root cause]

**Fix (Step-by-Step):**
[step-by-step instructions]

**Result:** [outcome]
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

**Thread 1: GitHub Repository Research** *(always run — find repo if not given)*
- If only endpoint given → search for GitHub repo first, then read it
- Read ENTIRE README (all sections)
- Search: endpoint, https://, remote, hosted, cloud
- Extract ALL external URLs
- Look for deprecation notices (may point to new versions)

**Thread 2: Repository File Search** *(always run)*
- Files: .env.example, config.json, setup.md, docs/
- Grep: "https://", "endpoint", "url", "host" patterns
- API paths: /api/v2, /v1, /mcp
- Comments with example URLs

**Thread 3: Remote Endpoint Probing** *(always run — find endpoint if not given)*
- If only GitHub given → extract candidate endpoint URLs from README/docs first
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
- ✅ GitHub repo found AND full README read?
- ✅ All repo files searched?
- ✅ Remote endpoint found AND probed (GET + POST)?
- ✅ Framework/SDK version identified?
- ✅ Compliance documented?

**Both GitHub + remote endpoint MUST be researched before moving to Step 1.**
If either source is missing → continue searching until found or confirmed non-existent.

---

## Multi-Server Parallel Research

**If 1 server detected** → skip this section, continue with standard workflow from Step 0.
**If 2+ servers detected** → load and follow `references/multi-server.md`.

---

## Key Learnings from Research & Verification

### Learning 1: Framework Priority

Framework version determines protocol, NOT base SDK.

**Check order:**
1. FastMCP present? → Use FastMCP mapping
2. Framework present? → Use framework mapping
3. Otherwise → Use base SDK mapping

**Example:** Telnyx with FastMCP 0.4.1 + mcp 1.3.0 → 2025-06-18 (FastMCP), not 2025-03-26 (mcp).

**🔒 CRITICAL:** Before marking protocol version in CSV:
1. Extract exact version from source (package.json, pyproject.toml, etc.)
2. **DO NUMERIC COMPARISON** against SKILL.md mapping ranges (NOT assumptions)
3. Example: SDK 1.7.0 < 1.8? YES → 2024-11-05 (NOT 2025-06-18)
4. **NEVER proceed without explicit verification**

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

## Authentication Detection Rules

**Core Principle:** Bearer Token is a DELIVERY METHOD, not a credential type. Multiple auth fields can be Yes simultaneously.

### Visual Relationship
```
Bearer Token (delivery method — how it's sent)
    ├── Personal Access Token  (user-specific, acts on behalf of user)
    ├── OAuth Access Token     (from OAuth 2.1 flow)
    └── JWT Token              (signed token)

API Key (credential type — app identification, NOT user-specific)
    └── Can be delivered via Bearer header OR X-API-Key header OR query param
```

### Detection Rules per Auth Type

> 🔒 **LOCKED — Do NOT modify this table without explicit user approval.**

| Auth Type | Mark Yes When | Example Header | Co-occurrence |
|-----------|--------------|----------------|---------------|
| **Bearer Token** | `Authorization: Bearer` found anywhere in source/docs | `Authorization: Bearer <token>` | Always Yes when PAT, OAuth token, or API key uses Bearer delivery |
| **Personal Access Token** | "personal access token", "PAT" explicitly in docs/README/code; user-specific token acting on behalf of user | `Authorization: Bearer <pat>` | → Also set Bearer Token = Yes |
| **API Token** | `"api key"`, `"api_token"`, `"api_key"` in source or docs; static key for app identification | `X-API-Key: abc123` OR `Authorization: Bearer <api_key>` | → Also set Bearer Token = Yes if sent via Bearer |
| **OAuth 2.1 - Authorization Code Flow** | OAuth 2.1 auth code flow documented; user redirected to auth page | `Authorization: Bearer <oauth_token>` | → Also set Bearer Token = Yes |
| **OAuth 2.1 - Client Credentials Flow** | OAuth 2.1 client credentials documented; machine-to-machine auth | `Authorization: Bearer <token>` | → Also set Bearer Token = Yes |

### Key Rules

> 🔒 **LOCKED — Do NOT modify these rules without explicit user approval.**

1. **Bearer Token ≠ credential type** — it's the transport mechanism. Never mark ONLY Bearer Token without also identifying the underlying credential type (PAT / API Token / OAuth).
2. **PAT + Bearer = both Yes** — PAT is delivered via Bearer header, so both are always Yes together.
3. **API Token + Bearer = both Yes** — if API key is sent via `Authorization: Bearer`, both are Yes.
4. **API Token ≠ PAT** — API keys are app-level static keys; PATs are user-level tokens. Do not conflate.
5. **Multiple Yes allowed** — Auth fields are NOT mutually exclusive. Mark all that apply.

### Detection Patterns (from source code)

```
Bearer Token = Yes if:
  "Authorization: Bearer" in source/docs

PAT = Yes if:
  "personal access token" in combined_text OR
  "PAT" explicitly mentioned in docs

API Token = Yes if:
  "api key" in combined_text OR
  "api_token" in combined_text OR
  "api_key" in combined_text
```

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

## Key Learnings — Attribute Documentation (Session 2026-03-26)

### Learning 3: TLS vs Bearer Token Confusion

**❌ WRONG:** Marking TLS 1.3 = Yes just because Bearer Token = Yes

**✅ CORRECT:** TLS encryption only applies to REMOTE transport (HTTP/SSE or StreamableHttp)

| Transport | TLS Apply? | Reason |
|-----------|-----------|--------|
| **STDIO** (local) | No | stdin/stdout on local machine, no encryption needed |
| **HTTP/SSE** (remote) | Yes | Network transmission requires TLS |
| **StreamableHttp** (remote) | Yes | Network transmission requires TLS |

**Rule:** If Transport = STDIO AND no remote endpoint exists → **All TLS fields = No**

Bearer Token authentication (for API calls inside tools) ≠ TLS encryption (for transport layer). They are independent.

---

### Learning 4: Pricing Attribute — Server vs Service

**❌ WRONG:** Marking Paid = Yes because GPU rentals charge money

**✅ CORRECT:** Pricing reflects the **MCP SERVER itself**, not the service it accesses

| Scenario | Free | Paid |
|----------|------|------|
| **Open-source server, free service** | Yes | No |
| **Open-source server, paid service** | Yes | No |
| **Paid server (requires license)** | No | Yes |
| **Closed-source vendor server** | No | Yes |

**Rule:**
- Mark **Free = Yes** if the MCP server itself is open-source/no-cost
- Mark **Paid = Yes** ONLY if you must pay to access/use the MCP server itself
- Don't confuse with downstream service costs (GPU rentals, API calls, subscriptions)

**Hyperbolic Example:** Server is open-source (Free = Yes), but using it to rent GPUs costs money (that's a service fee, not server fee).

---

### Learning 5: Tools Operations — Presence vs Absence

**❌ WRONG:** Marking "Read-only and/or update operations = Yes" when server also has delete operations

**✅ CORRECT:** Mark the HIGHEST level of operation capability present

**Decision Tree:**
```
Does server have delete/terminate operations?
    → Yes: Mark "Read-only update and/or delete operations = Yes"
           Mark others = No

Does server have only create/update (no delete)?
    → Yes: Mark "Read-only and/or update operations = Yes"
           Mark others = No

Does server have ONLY read operations?
    → Yes: Mark "Read-only operations = Yes"
           Mark others = No
```

**Hyperbolic Example:**
- Has: list (read), get (read), rent (write), terminate (delete)
- Highest level = Delete
- **Result:** Read-only update and/or delete = Yes, others = No

---

### Learning 6: Capabilities - Tools Categorization

**❌ WRONG:** Creating custom categories (e.g., "GPU Management", "SSH Management")

**✅ CORRECT:** Use the EXACT categories from your research documentation

**Rule:** When user provides tool categories in research, use those verbatim. Don't reorganize or rename.

**Source Priority:**
1. User-provided research documentation (most accurate)
2. README official tool groupings
3. Source code file structure
4. Last resort: Infer from tool names

**Hyperbolic Example:**
User research showed:
```
Team & Workspace Metadata
Search & Query Utilities
Other
```
Use EXACTLY these categories, not custom ones.

---

### Learning 7: Non-Read-Only Tools Row — Conditional Presence

**❌ WRONG:** Always including "Non-Read-Only Tools" row in CSV

**✅ CORRECT:** Only include this row if it exists in your research documentation

**Rule:**
- If research has no "Non-Read-Only Tools" section → Don't add the row
- If research DOES have it → Include with exact details
- If all tools are read-only → Row says "None — all tools are read-only"

---

### Learning 8: Bearer Token ≠ TLS Encryption

**Core Principle:** These are INDEPENDENT concepts

| Concept | What It Is | Where It Appears |
|---------|-----------|------------------|
| **Bearer Token** | Authentication credential delivery method | Authorization header for API calls |
| **TLS Encryption** | Transport layer encryption | HTTPS protocol for network communication |

**Example:**
- STDIO server with Bearer Token = uses auth inside local process (no TLS needed)
- HTTP endpoint with no auth = needs TLS for network safety but no Bearer needed
- HTTP endpoint with Bearer Token = needs BOTH TLS (network) + Bearer (auth)

**Rule:** Don't assume TLS based on Bearer Token presence. Check transport protocol instead.

---

## Integration Rules (Updated 2026-03-26)

- **Before each session:** Read "Learned Fixes" section and Prevention Checklist before researching
- **When researching:** Verify Transport Protocol FIRST, then determine TLS
- **When checking Pricing:** Ask "Is the MCP server itself paid?" not "Does it access paid services?"
- **When categorizing Tools:** Use user's exact categories from research
- **When marking Tools Operations:** Find the HIGHEST capability level present
- **When adding rows:** Only include sections that exist in research documentation

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
├── Git Repo Version (v1.2.3 format — latest only)
│   Source priority: 1st → GitHub Releases  2nd → GitHub Tags  (never use server initialize response)
├── Category (one of 4: File Management / Developer Tools / Data Retrieval / Productivity)
├── GitHub Repository (URL or N/A)
└── Endpoint URL (URL or N/A)

Distribution Type (Official / Community)
MCP Protocol Version (2025-11-25 / 2025-06-18 / 2025-03-26 / 2024-11-05)
Pricing (Free / Paid)
Hosting Provider (SaaS / GitHub)
Authentication (OAuth 2.1 AuthCode / OAuth 2.1 Client Creds / Bearer / PAT / API Token)
Data Protection (TLS 1.3 / TLS 1.2 / Lower versions or no encryption)
Transport Protocol (STDIO / HTTP/SSE / StreamableHttp)
Tools Operations (Read-only / R+Update / R+Update+Delete)
Deployment Approach (Local / Container / Remote)
Compliance & Certifications (HIPAA / GDPR / SOC 2 / FedRAMP)
Capabilities (Tools)
Capabilities - Tools (detailed_info with categories and bullet points)
Non-Read-Only Tools (detailed_info with categories OR "None — all tools are read-only")
```

**Example Rows:**

Row 1 (Basic Info):
```csv
MCP Info,Description,"Telnyx Python MCP Server for managing telephony, messaging, and AI assistant workflows via Claude and other MCP clients. Provides 50+ tools across SMS/MMS messaging, voice call control, AI assistant creation, cloud storage (S3-compatible), embeddings, webhook management, and integration secrets handling."
```

Row 2 (Version):
```csv
MCP Info,Git Repo Version,"v0.1.2 [ Referred from Tags/Releases ]"
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
MCP Info,Git Repo Version,"v0.1.2 [ Referred from Tags/Releases ]"
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
Hosting Provider,GitHub,Yes
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
Tools Operations,Read-only operations,No
Tools Operations,Read-only and/or update operations,No
Tools Operations,Read-only update and/or delete operations,Yes
Deployment Approach,Local,Yes
Deployment Approach,Container,No
Deployment Approach,Remote,No
Compliance & Certifications,HIPAA,No
Compliance & Certifications,GDPR,No
Compliance & Certifications,SOC 2,No
Compliance & Certifications,FedRAMP,No
Capabilities,Tools,Yes
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
```

---

## Report Generation Rules (Single & Multi-Server)

### Rule 1 — Per-Server CSV Naming
Each server gets its own report: `~/Desktop/MCP_reports/<mcp-name>.csv`
Example: `github-mcp-server.csv`, `slack-mcp.csv`

---

### Rule 2 — Always Check Both Sources (GitHub + Remote Endpoint)
Regardless of what input the user provides, ALWAYS research both:

| User provides | Also check |
|---------------|------------|
| GitHub URL | Search for + probe remote endpoint (README, docs, curl test) |
| Remote endpoint | Search for + find GitHub repository |
| Server name | Find both GitHub URL and remote endpoint |

Run both source searches in parallel (existing Step 0.5 + endpoint probe).
Generate the report from the **combined findings of both sources**.

---

### Rule 3 — Multi-Value Category Merge (Auth / Transport / Deployment)
For these 3 categories, if **either source** yields a value → mark it `Yes`:

**Authentication** — union of both sources:
```
If GitHub shows: API Token = Yes
If endpoint shows: Bearer Token = Yes
→ Report both as Yes:
  Authentication,API Token,Yes
  Authentication,Bearer Token,Yes
```

Same logic applies to:
- **Transport Protocol** — union of all detected transports across both sources:
  - GitHub = STDIO + endpoint probe = HTTP/SSE → both Yes
  - GitHub = STDIO + endpoint probe = StreamableHttp → both Yes
  - Endpoint = HTTP/SSE AND StreamableHttp detected → both Yes
  - All three possible → STDIO Yes, HTTP/SSE Yes, StreamableHttp Yes
- **Deployment Approach** — if GitHub = Local and endpoint = Remote → both Yes

---

### Rule 4 — Capabilities - Tools: Combine & Mention Both
If GitHub source lists tools and remote endpoint lists additional tools:
- Merge all tool names into one `Capabilities - Tools,detailed_info` cell
- Note the source of each tool group if they differ

---

### Rule 5 — Hosting Provider Priority Rule
If research finds **both** SaaS Vendor and GitHub as hosting:
```
→ Mark SaaS Vendor = Yes (SaaS Vendor takes priority)
→ Mark GitHub = No
```
Rationale: If a vendor hosts on SaaS infrastructure AND has a GitHub repo, the primary hosting is SaaS.

---

## Learned Fixes — MCP Skill 2.0 Self-Learning (Consolidated)

**Purpose:** Track attribute documentation errors, their root causes, and corrections for future research sessions.

**Last Updated:** 2026-03-26

### Error #1: TLS Encryption Mismatch

**Date:** 2026-03-26 | **Server:** Hyperbolic MCP | **Severity:** High

**Signals (What Went Wrong)**
- Marked TLS 1.3 = Yes and TLS 1.2 = No
- Server uses STDIO transport (local process, no remote endpoint)
- Confused Bearer Token authentication with TLS encryption

**Root Cause:** Assumption that Bearer Token (API authentication) implies HTTPS/TLS (transport encryption). These are independent:
- Bearer Token = how credentials are sent (auth layer)
- TLS = how data is encrypted (transport layer)

**Fix (Step-by-Step):**
1. Check Transport Protocol first: Is it STDIO, HTTP/SSE, or StreamableHttp?
2. For STDIO: All TLS fields = No (local process, no network transmission)
3. For HTTP/SSE or StreamableHttp: Check actual endpoint protocol (curl -I to verify)
4. Don't assume: Bearer Token ≠ TLS automatically

**Result:** TLS 1.3 = No, TLS 1.2 = No, Transport = STDIO only

---

### Error #2: Pricing Attribute Confusion

**Date:** 2026-03-26 | **Server:** Hyperbolic MCP | **Severity:** High

**Signals:** Marked Paid = Yes because GPU rentals cost money; Confused service fees with server licensing fees

**Root Cause:** Misunderstood "Pricing" attribute scope. Should reflect the **MCP SERVER itself** not downstream services it accesses.

**Fix (Step-by-Step):**
1. Ask: "Is the MCP server itself paid or open-source?"
2. If open-source: Free = Yes, Paid = No (regardless of service costs)
3. If proprietary/licensed: Free = No, Paid = Yes
4. Ignore: GPU rental costs, subscription fees, service charges
5. Document: Only the server's own licensing/distribution model

**Result:** Free = Yes, Paid = No (open-source server, paid services independent)

---

### Error #3: Tools Operations Highest Level Misidentification

**Date:** 2026-03-26 | **Server:** Hyperbolic MCP | **Severity:** Medium

**Signals:** Marked both "Read-only and/or update operations = Yes" AND "Read-only update and/or delete operations = Yes"; These are mutually exclusive

**Root Cause:** Didn't identify highest capability level. Server has delete operations (terminate-gpu-instance) so should mark only the delete-level option.

**Fix (Step-by-Step):**
1. Scan all tools: Identify all operation types present (Read-only / Update / Delete)
2. Find highest level:
   - Delete present? → Mark "Read-only update and/or delete operations = Yes"
   - Update present (no delete)? → Mark "Read-only and/or update operations = Yes"
   - Read-only only? → Mark "Read-only operations = Yes"
3. Mark others = No (mutually exclusive)

**Result:**
- Read-only operations = Yes (list-available-gpus, get-cluster-details, list-user-instances, ssh-status)
- Read-only and/or update = No
- Read-only update and/or delete = Yes (rent-gpu-instance creates, terminate-gpu-instance deletes, remote-shell executes)

---

### Error #4: Capabilities - Tools Incorrect Categorization

**Date:** 2026-03-26 | **Server:** Hyperbolic MCP | **Severity:** Medium

**Signals:** Created custom categories: "GPU Management", "SSH Management"; User provided specific categories: "Team & Workspace Metadata", "Search & Query Utilities", "Other"

**Root Cause:** Made assumptions about logical groupings instead of using user-provided documentation.

**Fix (Step-by-Step):**
1. Get user's research first: Always ask for exact categories
2. Extract from documentation: Check README, docs, or user research
3. Use verbatim: Don't reorganize, rename, or regroup
4. Priority order: User research > README > Source code > Infer from tool names

**Result:** Used exact user-provided categories:
- Team & Workspace Metadata
- Search & Query Utilities
- Other

---

### Error #5: Non-Read-Only Tools Row Unconditional Inclusion

**Date:** 2026-03-26 | **Server:** Hyperbolic MCP | **Severity:** Low

**Signals:** Added "Non-Read-Only Tools" row to CSV when user research didn't include it; Fabricated detailed_info content

**Root Cause:** Assumed this row should always exist. It's optional and only appears if documented in research.

**Fix (Step-by-Step):**
1. Check research: Does user's documentation have "Non-Read-Only Tools" section?
2. If No: Don't add the row to CSV
3. If Yes: Include exactly as documented
4. If all read-only: Include row with text "None — all tools are read-only"
5. Never fabricate: Only include what exists in research

**Result:** Don't include Non-Read-Only Tools row if not in user's research

---

### Error #6: Protocol Version Range Mapping Without Numeric Verification

**Date:** 2026-03-26 | **Server:** Hyperbolic MCP | **Severity:** CRITICAL

**Signals:** Marked Protocol Version = 2025-06-18; User pointed out: SDK 1.7.0 should map to 2024-11-05; Never performed numeric comparison

**Root Cause:** **CRITICAL MISTAKE:** Noted the SDK version but did NOT verify it against SKILL.md mapping ranges.
- Found: `@modelcontextprotocol/sdk: 1.7.0`
- Mapping rule: `sdk <1.8` → `2024-11-05`
- Failed to compare: 1.7.0 < 1.8? YES
- Assumed instead: Guessed 2025-06-18 without checking

**Fix (Step-by-Step) — STRICT PROTOCOL**

**BEFORE marking ANY protocol version in CSV:**
1. Extract exact SDK/Framework version from source
2. **DO NUMERIC COMPARISON** against SKILL.md ranges
3. Verify: Is 1.7.0 < 1.8? Is 1.15 ≤ version < 1.23? Calculate explicitly
4. Only AFTER numeric verification → mark the CSV

**Example (Hyperbolic):**
```
Step 1: SDK = 1.7.0
Step 2: Check range: sdk <1.8 → 2024-11-05
Step 3: Compare: 1.7.0 < 1.8? YES ✅
Step 4: Mark: 2024-11-05 = Yes
```

**NEVER DO THIS:**
- "It looks like 1.7, probably maps to..." ❌
- Skipping the numeric comparison ❌
- Assuming version ranges without verification ❌

**Result:** Protocol Version = 2024-11-05 (not 2025-06-18)

**🔒 LOCKED RULE:** **STRICTLY DO NOT MAP without checking right MCP protocol version.** **DO NUMERIC COMPARISON, NOT ASSUMPTION.**

---

### Prevention Checklist (Before Final CSV)

Use this checklist EVERY TIME before generating final CSV:

- [ ] **Protocol Version:** Did numeric comparison (not assumption) against SKILL.md ranges?
- [ ] **Transport Protocol:** Verified (STDIO/HTTP/StreamableHttp) in source code?
- [ ] **TLS Encryption:** Only marked Yes if HTTP/SSE/StreamableHttp endpoint exists?
- [ ] **Pricing:** Reflects MCP server licensing, not service fees?
- [ ] **Tools Operations:** Only ONE "highest level" marked as Yes?
- [ ] **Capabilities Categories:** Exactly match user's research documentation?
- [ ] **Non-Read-Only Tools:** Only included if present in user research?
- [ ] **Bearer Token ≠ TLS:** Verified these are independent concepts?

---

### Session Log

| Date | Server | Errors | Status |
|------|--------|--------|--------|
| 2026-03-26 | Hyperbolic MCP | 6 errors, all corrected | ✅ Complete |

---

## Summary

**Unified MCP skill** — single optimized execution path for research, deployment, error recovery, and project auditing. Consolidates all functionality into one unified, fast, token-efficient implementation.

- ✅ 8-step research workflow
- ✅ 3 input type handling (Remote / GitHub / Name)
- ✅ Conditional local setup (user choice)
- ✅ 4 connection methods (STDIO / Package / Docker / Remote)
- ✅ 7-phase inline error recovery (auto-triggered)
- ✅ Evidence-backed attributes (source tracking)
- ✅ Project compliance audit (PASS/FAIL/WARN)
- ✅ Skill 2.0 self-learning (Learned Fixes section)
- ✅ Security mandate enforced (credentials via file only)
- ✅ **FAST & EFFICIENT** — Single file, optimized paths, minimal tokens

---

**All workflows, instructions, and processes preserved. Zero functionality lost. Pure optimization.**
