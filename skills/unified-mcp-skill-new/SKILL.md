---
name: unified-mcp-skill-new
user-invocable: true
description: >
  All-in-one unified MCP skill: research MCP servers, document attributes,
  conditionally set up locally, diagnose errors inline, and audit projects. Single optimized
  execution path for research, deployment, error recovery, and compliance auditing. Faster,
  fewer tokens, full functionality.
---

<!-- SKILL_VERSION: 3.0.0 | Updated: 2026-03-27 -->

🔒 **SYNC NOTE**
> SKILL.md is the single source of truth for all rules, learnings, and formats.
> `references/multi-server.md` contains ONLY multi-server orchestration logic (M1-M3, Layers 0/1).
> `references/learned-fixes.md` contains ONLY error case studies (no rule duplication).
> If you change workflow steps, attribute definitions, or report format here → verify multi-server.md references still point correctly.

---

🚫 **ZERO-ASSUMPTION POLICY — ABSOLUTE ENFORCEMENT**

**Every Yes/No in the final CSV MUST have a verified source. No exceptions. No guessing. No skipping.**

If you cannot verify an attribute from source → mark it `UNVERIFIED` and flag to user. NEVER guess Yes or No.

---

### GATE 1: EVIDENCE LEDGER (Every Attribute Needs Proof)

**Before writing ANY Yes/No value, you MUST have:**
```
For EVERY attribute in the CSV:
  ✅ REQUIRED: Source file or URL where the value was found
  ✅ REQUIRED: Exact quote, line number, or API response proving the value
  ❌ BLOCKED: "Probably Yes" / "Likely No" / "Seems like" / "Should be"
  ❌ BLOCKED: Copying from similar servers / assuming from server name
  ❌ BLOCKED: Marking Yes/No without reading actual source
```

**Evidence Ledger format (build internally before CSV):**
```
ATTRIBUTE              | VALUE | SOURCE                        | EVIDENCE
Protocol Version       | Yes   | package.json line 5           | "@modelcontextprotocol/sdk": "1.15.0"
TLS 1.3                | No    | Transport = STDIO (verified)  | No remote endpoint exists
Bearer Token           | Yes   | server.json line 12           | "BUILDKITE_API_TOKEN (Required, Secret)"
Pricing Free           | Yes   | LICENSE file                  | "MIT License"
Tools Ops Delete       | Yes   | README tools section          | "cancel_build" = delete pattern
```

**If evidence column is empty → DO NOT write that row to CSV. Ask user instead.**

---

### GATE 2: MANDATORY CONNECTION & SOURCE VERIFICATION

**You MUST establish contact with actual sources before reporting. Never report from memory or assumption.**

```
BEFORE filling ANY attribute:
□ GitHub repo accessed? (README read, files searched, deps extracted)
  → If repo 404 or private: flag to user, do NOT assume attributes
□ Remote endpoint probed? (GET + POST with curl, responses recorded)
  → If no endpoint: mark Endpoint URL = N/A with evidence "GET 404, POST 404"
□ Dependency file found and version extracted?
  → If no deps file: flag to user, do NOT guess protocol version
□ server.json / .env.example / config files checked?
  → If none exist: document "no config files found" — do NOT mark auth as No without this check
```

**HARD BLOCK: Do NOT proceed to Step 5 (Attribute Filling) until Step 0.5 (Parallel Search) is COMPLETE with all 5 threads returning results or confirmed-empty.**

---

### GATE 3: LEARNING GATE (All Learnings Checked Every Time)

**Before creating final CSV, walk through EVERY learning. This is NOT optional. Do NOT skip any.**

```
MANDATORY LEARNING WALKTHROUGH (check each one, document result):

□ L1 Framework Priority: Which framework/SDK found? Version? Mapping result?
  → Numeric comparison done? (e.g., "1.7.0 < 1.8 = YES → 2024-11-05")

□ L2 Endpoint Verification: GET test result? POST test result? Response format?
  → Both tests run? If skipped, why?

□ L3 TLS vs Bearer: Transport protocol identified FIRST?
  → If STDIO: All TLS = No (confirmed, not assumed)
  → Bearer Token presence did NOT influence TLS decision?

□ L4 Pricing: Is MCP server code open-source? License file checked?
  → API key requirement NOT confused with pricing?
  → Downstream service cost NOT confused with server cost?

□ L5 Tools Ops: All tool names parsed? Delete/cancel patterns searched?
  → HIGHEST level only marked? Only ONE level = Yes?

□ L6 Categories: Category titles from standard taxonomy (Learning 6)?
  → Each tool classified into best-fit standard category?
  → ZERO invented categories (only taxonomy titles used)?

□ L7 Five Mandatory Rows: Are ALL five rows present in the CSV with correct format?
  → Capabilities - Tools,detailed_info     present? (use "None" if no tools)
  → Capabilities - Resources,detailed_info present? (use "None" if no resources)
  → Capabilities - Prompts,detailed_info   present? (use "None" if no prompts)
  → Capabilities - Sampling,detailed_info  present? (use "None" if no sampling)
  → Non-Read-Only Tools,detailed_info      present? (use "None" if all read-only)
  → Attribute column is "detailed_info" for ALL five — never "No" or blank
  → ALL FIVE required — no exceptions

□ L3 recheck: TLS/Bearer independence confirmed? (STDIO → TLS always No)

□ L8 Description Single-Line: Is MCP Info,Description a single unbroken line?
  → NO embedded newlines — all sentences joined with spaces only
  → If Protocol Version or Pricing rows appear empty/missing → description has newlines (fix first)
  → Verify in raw CSV: description must start and end on the same line
```

**If ANY checkbox cannot be confirmed → STOP. Do not create CSV. Resolve first.**

---

### GATE 4: POST-RESEARCH SELF-IMPROVEMENT

**After EVERY research session (before presenting final report to user):**

```
□ Did any attribute require correction during this session?
  → YES: Document the error pattern in learned-fixes.md (new entry)
  → NO: Confirm all attributes matched first evidence found

□ Did any new SDK/framework version mapping appear?
  → YES: Verify it's in SKILL.md Step 1 mapping table, add if missing

□ Did any new authentication pattern appear?
  → YES: Verify it's in Authentication Detection Rules, add if missing

□ Was any Learning insufficient or ambiguous for this server?
  → YES: Refine the Learning with new example in SKILL.md
```

**The skill gets BETTER with every session. Never discard a learning opportunity.**

---

### RULES 1-5 (Enforcement Summary)

🔒 **RULE 1: SOURCE-ONLY ATTRIBUTES** — Every value from actual documentation. Zero invented content.
🔒 **RULE 2: NUMERIC VERIFICATION** — Protocol version, TLS, pricing, auth all verified with explicit checks.
🔒 **RULE 3: FULL LEARNING WALKTHROUGH** — All Learnings checked. All learned-fixes.md patterns reviewed.
🔒 **RULE 4: NO CSV WITHOUT EVIDENCE** — Evidence Ledger must be complete. Empty evidence = blocked.
🔒 **RULE 5: SELF-IMPROVEMENT** — New patterns documented. Learnings refined. Skill improves each session.



---

# Unified MCP Skill — Research, Deploy, Fix, Audit

**All-in-one MCP server management:** Research and document attributes, conditional local setup (clone+install), inline error recovery, and project compliance auditing — all in one unified skill with optimized execution paths.

---

## Quick Reference (Workflow at a Glance)

```
Step 0   Config & Input       → Classify input (endpoint/GitHub/name), set paths
Step 0.5 Parallel Search      → 5 threads: repo, files, endpoint probe, deps, compliance
Step 1   Protocol Version     → Map SDK/framework version to protocol date (NUMERIC comparison)
Step 2   Remote Endpoint      → Probe URL, detect transport, test TLS, build config
Step 3   GitHub Repo          → Ask: local setup / remote / research-only? Clone if local
Step 4   Server Name Only     → Search vendor docs + GitHub, resolve to URL
Step 5   Attribute Filling    → Fill all fields with evidence + logic enforcement
  5.1    Auth Verification    → Check server.json, README, .env, vendor docs
  5.2    Tools Ops Check      → Highest level only (mutually exclusive)
  5.3    Deployment Check     → Mark ALL applicable (not exclusive)
  5.4    Hosting Check        → SaaS priority over GitHub
Step 6   Transport & Caps     → Detect transports, extract capabilities
Step 7   Basic Info           → Name, description, category, version
Step 8   Save Report          → CSV to user-configured path

On fail → 7-phase Error Recovery (auto-triggered)
2+ servers → Multi-Server mode (see references/multi-server.md)
```

**Enforcement Gates (MANDATORY — block CSV until passed):**
Gate 1=Evidence Ledger (every Yes/No has source proof)
Gate 2=Connection Verified (all 5 threads complete)
Gate 3=Learning Walkthrough (L1-L7 checked)
Gate 4=Self-Improvement (new patterns documented)

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

**SSRF Protection — Endpoint Probing (No exceptions):**
Before probing ANY user-provided URL, validate the target:
```
BLOCKED targets (NEVER probe):
- 127.0.0.0/8       (localhost / loopback)
- 10.0.0.0/8        (private class A)
- 172.16.0.0/12     (private class B)
- 192.168.0.0/16    (private class C)
- 169.254.0.0/16    (link-local / cloud metadata)
- 0.0.0.0           (all interfaces)
- ::1, fd00::/8     (IPv6 loopback / private)
- localhost, *.local, *.internal
```
If URL resolves to a blocked range → REJECT immediately, do NOT probe.

**Network Safety — All curl/HTTP probes:**
- Always set timeouts: `--connect-timeout 5 --max-time 10`
- Max 3 probe attempts per endpoint, 2-second delay between retries
- Never follow redirects to blocked IP ranges (`--max-redirs 3`)

---

## Usage

**Invoke with:** `/unified-mcp-skill-new`

> ✅ **UNLOCKED — Invocation command updated to `/unified-mcp-skill-new` with explicit user approval (2026-03-27).**

```
RESEARCH
/unified-mcp-skill-new "Research the GitHub MCP server"
/unified-mcp-skill-new "Get full report on the Slack MCP server"
/unified-mcp-skill-new "Document this MCP server: https://github.com/org/repo"
/unified-mcp-skill-new "Analyze and score this MCP: https://mcp.example.com"

LOCAL SETUP
/unified-mcp-skill-new "Set up the GitHub MCP server locally"
/unified-mcp-skill-new "Clone and get the Slack MCP running"

ERROR RECOVERY (automatic)
/unified-mcp-skill-new "The MCP server won't connect"
/unified-mcp-skill-new [paste error or stack trace]

PROJECT AUDIT
/unified-mcp-skill-new "Review the project"
/unified-mcp-skill-new "Is this ready to commit?"

MULTI-SERVER / BATCH
/unified-mcp-skill-new "Research these MCPs: GitHub, Slack, Linear"
/unified-mcp-skill-new "Compare the GitHub and Stripe MCP servers"
/unified-mcp-skill-new "Batch report on: [server1], [server2], [server3]"
```

---

## Unified Workflow (Steps 0-8 + Error Recovery)

> Steps 0-8 run sequentially. Steps 5.1-5.4 are verification sub-steps within Step 5.
> Error Recovery (7 phases) triggers automatically on connection failure.
> Multi-Server mode (M1-M3) activates when 2+ servers detected — see `references/multi-server.md`.

### Step 0 — Configuration & Input Classification

**Path configuration — always ask, never assume:**

**Path Validation (MANDATORY before use):**
```
REJECT any path that contains:
- ".." segments           (path traversal)
- Symlinks to outside     (resolve with realpath first)
- System directories      (/etc, /var, /usr, /bin, /sbin, /System, /Library)
- Other users' homes      (/Users/other, /home/other)
- Root home               (/root, ~root)

ALLOW only:
- User's home directory subtree  (~/ or /Users/<current-user>/)
- Explicitly approved directories (e.g., /tmp for testing)
```
Validate BEFORE any file write, clone, or report save operation.

Report save path — ask every first use:
```
Where should MCP reports be saved?
Enter full path (e.g. /Users/you/Documents/mcp-reports):
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

**NOTE:** Protocol `2025-03-26` ONLY applies to FastMCP 0.3.x. No Python/TypeScript/Go SDK maps to this version. If you don't find FastMCP 0.3.x, this protocol version does NOT apply.

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
| 3.1 | Clone repo to configured path (`git clone --depth 1` for research; full clone only if user needs history) | Dir exists, expected files present, path validated |
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

**Docker Security Check (before building):**
- Verify Dockerfile uses pinned base image (e.g., `python:3.12-slim`, NOT `python:latest`)
- If `:latest` tag found → WARN user: "Unpinned base image detected — may introduce supply chain risk"
- Check for `USER` directive (non-root execution preferred)
- Review any `curl | sh` or piped install patterns → WARN user before proceeding

**If Option 3 (Research Only):**
Skip local setup, proceed to attribute research.

---

### Step 4 — Server Name Only

1. Search vendor documentation for remote endpoint
2. Search for GitHub repository (try: `{vendor}-mcp-server`, `mcp-{vendor}`, `{vendor}-mcp`)
3. If both found, ask:
   ```
   [ 1 ] Use remote endpoint
   [ 2 ] Use GitHub repository
   [ 3 ] Help me decide
   ```
4. Continue with chosen path

**If server not found (neither endpoint nor repo):**
```
Could not find an MCP server matching "[server name]".

Searched:
- GitHub: github.com/*/[name]-mcp-server, mcp-[name], [name]-mcp
- Vendor: [name].com, api.[name].com
- MCP directory: https://github.com/modelcontextprotocol/servers

[ 1 ] Try a different name or URL
[ 2 ] Provide GitHub URL directly
[ 3 ] Cancel research
```
Do NOT fabricate a server or guess URLs. Always confirm with user before proceeding.

---

### Step 5 — Attribute Filling with Logic Enforcement

**HARD PREREQUISITE:** Step 0.5 (Parallel Search) MUST be complete before entering Step 5.
All 5 threads must have returned results or confirmed-empty. If any thread is incomplete → go back and finish it.

**Transport ↔ Deployment Auto-Rules:**
- STDIO=Yes → Auto-set: Local=Yes
- HTTP/SSE OR StreamableHttp=Yes → Auto-set: Remote=Yes

**Mutually Exclusive Attributes (mark exactly ONE as Yes):**

| Attribute | Options |
|-----------|---------|
| Distribution | Official OR Community |
| Pricing | Free OR Paid |
| Tools Operations | Choose HIGHEST: Read-only / R+Update / R+Update+Delete |

**🔒 MCP Protocol Version — Mutual Exclusion Rule (MANDATORY):**
- Exactly ONE version row = Yes. ALL other version rows = No. NO blanks. NO omissions.
- If `2025-06-18 = Yes` → `2025-11-25 = No`, `2025-03-26 = No`, `2024-11-05 = No`
- Never leave a version row blank — blank = broken CSV row (overwritten data risk)
- Never write extra text, notes, or commentary in a version row's Status column

**🔒 Pricing — Mutual Exclusion Rule (MANDATORY):**
- If `Free = Yes` → `Paid = No`. Write exactly that. Nothing else in those two rows.
- If `Paid = Yes` → `Free = No`. Write exactly that. Nothing else in those two rows.
- Never leave Pricing rows blank. Never add commentary in the Status column.
- The Status column for Pricing rows contains ONLY `Yes` or `No` — no other text.

**Non-Exclusive Attributes (mark ALL that apply):**

| Attribute | Options |
|-----------|---------|
| Hosting Provider | SaaS Vendor / 3rd Party SaaS / GitHub / GitLab / Bitbucket / SourceHut/Gitea/Gogs (multiple Yes allowed, SaaS Vendor takes display priority) |
| Transport Protocol | STDIO / HTTP/SSE / StreamableHttp / FastAPI (multiple Yes allowed if server supports both) |
| Deployment Approach | Local / Container / Remote (multiple Yes allowed) |
| Authentication | All types can be Yes simultaneously (see Auth Detection Rules) |

**ZERO-ASSUMPTION ATTRIBUTE RULE:**

For EVERY attribute below — you MUST cite the source. If you cannot find evidence:
- Do NOT mark Yes or No by guessing
- Mark as `UNVERIFIED` and flag to user: "Could not verify [attribute] — source not found"
- Ask user to provide source or confirm before proceeding

Every "Yes" must have source + exact quote or file path. Every "No" must have evidence of absence (searched X, Y, Z — not found).

| Attribute | Type | Source |
|-----------|------|--------|
| Name | Text | GitHub or vendor docs |
| Description | Text | README (2–3 sentences, MUST start with "[Server Name] MCP Server…") |
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

**Why All Three Can Be "Yes":** See Authentication Detection Rules below (Bearer = delivery, PAT = credential type, API Token = app key — not mutually exclusive).

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

### Step 5.4 — Hosting Provider Verification

**Detection Context — Determine where the MCP server is hosted:**

```
HOSTING PROVIDER DETECTION CHECKLIST:
□ Step 1: Is it offered by the official vendor as SaaS? → SaaS Vendor = Yes
□ Step 2: Is it hosted by a third-party SaaS provider? (e.g., Vercel, Heroku, AWS) → 3rd Party SaaS = Yes
□ Step 3: Is the source code on GitHub? → GitHub = Yes
□ Step 4: Is the source code on GitLab? → GitLab = Yes
□ Step 5: Is the source code on Bitbucket? → Bitbucket = Yes
□ Step 6: Is the source code on SourceHut/Gitea/Gogs? → SourceHut/Gitea/Gogs = Yes
□ Step 7: Multiple sources possible → mark ALL applicable platforms
```

**Detection Patterns:**
- **SaaS Vendor:** Vendor's own domain (https://api.vendor.com) OR vendor docs state "managed service"
- **3rd Party SaaS:** Deployed URL on Vercel, Netlify, Heroku, AWS, Azure, etc. (visible in README/docs)
- **GitHub:** Repository URL path contains `github.com`
- **GitLab:** Repository URL path contains `gitlab.com`
- **Bitbucket:** Repository URL path contains `bitbucket.org`
- **SourceHut/Gitea/Gogs:** Repository URL contains `sr.ht`, custom Gitea/Gogs instance

**Note:** Hosting Provider is NOT mutually exclusive — mark all platforms where the server is available.

---

### Step 6 — Transport Protocol & Capabilities Verification

**Transport Protocol Detection:**

Check for these transports in source code and documentation:
```
TRANSPORT PROTOCOL VERIFICATION CHECKLIST:
□ Step 1: Does README mention stdin/stdout or STDIO? → STDIO = Yes
□ Step 2: Is there an HTTP endpoint with GET (Content-Type: text/event-stream)? → HTTP/SSE = Yes
□ Step 3: Is there an HTTP endpoint with POST (JSON-RPC streaming)? → StreamableHttp = Yes
□ Step 4: Is the server built with FastAPI or similar async web framework? → FastAPI = Yes
□ Step 5: Check for @app.post(), @app.get(), async def patterns
□ Step 6: Mark ALL applicable transport protocols (not mutually exclusive)
```

**Capabilities Detection:**

Extract from source code and README (not mutually exclusive — mark all that apply):
```
CAPABILITIES VERIFICATION CHECKLIST:
□ Step 1: Does server expose @mcp.tool() decorators? → Tools = Yes
□ Step 2: Does server expose @mcp.resource() decorators? → Resources = Yes
□ Step 3: Does server expose @mcp.prompt() decorators? → Prompts = Yes
□ Step 4: Does server expose sampling/completion methods? → Sampling = Yes
```

**Format for Capabilities - Tools (CSV cell):**
```
Category Name
  • tool_name – Description

Another Category
  • tool_name – Description
```

**Format for Capabilities - Resources (CSV cell):**
```
  • resource_uri – Description (resource type: text/image/document)
```

**Format for Capabilities - Prompts (CSV cell):**
```
  • prompt_name – Description (inputs: param1, param2)
```

**Format for Capabilities - Sampling (CSV cell):**
```
Model Capabilities
  • model_name – Sampling support (temperature, top_p, max_tokens)
  • completion_method – Inference optimization details
```

**Format for Non-Read-Only Tools (CSV cell):**
```
Category Name
  • tool_name: Description of the write/delete operation

Another Category
  • tool_name: Description
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

**Step 8.1 — Save Cost File (MANDATORY — always runs after CSV is saved)**

After saving the CSV, immediately generate and save `<servername>-cost.txt` at the same path.

**Timestamp capture (MANDATORY):**
- **Before Thread 1 starts:** capture `RESEARCH_START = datetime.utcnow().isoformat()` (e.g. `"2026-03-27T19:08:03"`)
- **After Gate 4 completes:** capture `RESEARCH_END = datetime.utcnow().isoformat()`
- These timestamps define the window for token counting

**How to compute token delta for this research session:**

1. Find the current session JSONL file (auto-detect from working directory):
   ```bash
   CWD_SLUG=$(pwd | tr '/' '-' | sed 's/^-//')
   ls -t ~/.claude/projects/-${CWD_SLUG}/*.jsonl | head -1
   ```
2. Sum all `message.usage.input_tokens`, `output_tokens`, `cache_read_input_tokens` where timestamp is between `RESEARCH_START` and `RESEARCH_END`
3. That sum = tokens used for this MCP research

**Cost script:** `references/cost-script.py` — set `RESEARCH_START`, `RESEARCH_END`, `SERVER_NAME` then run.
`SERVER_NAME` = kebab-case (e.g. `"abacatepay-mcp"`). Timestamps = ISO 8601 (e.g. `"2026-03-27T19:08:03"`).

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
- [ ] `.gitignore` includes required entries: `.env`, `.env.*`, `*.key`, `*.pem`, `.claude/settings.json`, `**/secrets/`, `**/*.secret`
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
PROJECT REVIEW — [current date]
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
- **If GitHub API returns 403 (rate limited):** Wait 60s, retry once. If still 403 → flag to user, do NOT guess attributes.
- **If repo returns 404:** Confirm private/non-existent. Flag to user: "Repo not accessible. Provide URL or skip?"
- **If repo is archived:** WARN user immediately:
  ```
  This repository is ARCHIVED. Research may be outdated.
  [ 1 ] Continue anyway (results marked as potentially stale)
  [ 2 ] Cancel research
  ```

**Thread 2: Repository File Search** *(always run)*
- Files: .env.example, config.json, setup.md, docs/
- Grep: "https://", "endpoint", "url", "host" patterns
- API paths: /api/v2, /v1, /mcp
- Comments with example URLs

**Thread 3: Remote Endpoint Probing** *(run unless short-circuited)*
- **SHORT-CIRCUIT:** If Thread 1 confirmed STDIO-only AND zero remote/endpoint URLs found in README → skip probing, mark Endpoint URL = N/A, TLS = No. Document: "STDIO-only server, no remote endpoint references in README."
- If only GitHub given → extract candidate endpoint URLs from README/docs first
- **BEFORE probing:** Validate URL against SSRF blocklist (see Security Mandate)
- **All probes use:** `--connect-timeout 5 --max-time 10 --max-redirs 3`
- **Rate limit:** Max 3 attempts per endpoint, 2-second delay between retries

- Test 1: GET request (check HTTP/SSE)
  - `curl -I --connect-timeout 5 --max-time 10 https://api.{vendor}.com/v2/mcp`
  - 200/401/403 → exists; 404 → not GET-based

- Test 2: POST with JSON-RPC (check StreamableHttp)
  - `curl -X POST --connect-timeout 5 --max-time 10 https://.../ -d '{"jsonrpc":"2.0",...}'`
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

### After All Threads Complete — HARD CHECKPOINT

**BLOCKING GATE: Do NOT proceed to Step 1 until ALL threads report results.**

```
Thread Status Check (ALL must be ✅ or ❌-confirmed):

□ Thread 1 — GitHub repo: ✅ Found and README read / ❌ Confirmed not found (searched 3+ patterns)
□ Thread 2 — File search:  ✅ Config files checked / ❌ No config files exist (ls confirmed)
□ Thread 3 — Endpoint:     ✅ Probed (GET + POST results recorded) / ❌ No endpoint (404 both, documented)
□ Thread 4 — Dependencies: ✅ SDK/framework version extracted / ❌ No deps file (confirmed absence)
□ Thread 5 — Compliance:   ✅ Checked vendor docs / ❌ No compliance info found (documented)
```

**If ANY thread is still pending → DO NOT proceed. Complete it first.**
**If a thread returned empty → document WHY it's empty (e.g., "no releases page, no tags, version from package.json").**

**Both GitHub + remote endpoint MUST be researched before moving to Step 1.**
If either source is missing → continue searching until found or confirmed non-existent with evidence.

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

**🔒 LOCKED:** Before marking protocol version in CSV:
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

## Key Learnings — Attribute Documentation

### Learning 3: TLS vs Bearer Token (INDEPENDENT Concepts)

**Core Principle:** Bearer Token = authentication delivery method. TLS = transport encryption. These are completely independent layers.

| Transport | TLS Apply? | Reason |
|-----------|-----------|--------|
| **STDIO** (local) | **Always No** | stdin/stdout on local machine, no network |
| **HTTP/SSE** (remote) | Yes | Network transmission requires TLS |
| **StreamableHttp** (remote) | Yes | Network transmission requires TLS |

**Rules:**
- STDIO transport → All TLS fields = No (no exceptions)
- Bearer Token present does NOT imply TLS present
- TLS decision is based ONLY on transport protocol, never on authentication
- Check transport FIRST, then determine TLS independently

**Examples:**
- STDIO server with Bearer Token → TLS = No (auth is local process, no network)
- HTTP endpoint with no auth → TLS = Yes (network needs encryption regardless)
- HTTP endpoint with Bearer Token → TLS = Yes AND Bearer = Yes (both, independently)

> See also: `references/learned-fixes.md` Error #1 (Buildkite) and Error #7 (Runway) for real case studies.

---

### Learning 4: Pricing Attribute — Server vs Service (Updated 2026-03-27)

**❌ WRONG:** Marking Paid = Yes because:
- API key is required
- The downstream service charges money
- Authentication requirement exists

**✅ CORRECT:** Pricing reflects the **MCP SERVER itself**, not the service it accesses or authentication required

| Scenario | Free | Paid | Notes |
|----------|------|------|-------|
| **Open-source server (MIT), paid Runway service** | Yes | No | MCP server is free; Runway AI service is paid (separate concern) |
| **Open-source server, free service** | Yes | No | Both free |
| **Open-source server, paid service** | Yes | No | MCP is free regardless of service costs |
| **Paid server (requires license)** | No | Yes | The server itself costs money |
| **Closed-source vendor server** | No | Yes | Proprietary licensing |

**CRITICAL RULE:**
- Mark **Free = Yes** if the **MCP server code is open-source** (regardless of API key requirement)
- Mark **Paid = Yes** ONLY if **you must pay to access/use the MCP server itself** (proprietary license)
- **NEVER confuse with:**
  - API key requirement (that's authentication, not pricing)
  - Downstream service costs (GPU rentals, API calls, subscriptions)
  - Service fees (paid by users of the service, not the MCP server)

**Examples:**
- ✅ Runway MCP: MIT License → Free = Yes (even though Runway AI service is paid)
- ✅ Telnyx MCP: Open-source → Free = Yes (even though Telnyx service is paid)
- ❌ Proprietary MCP: Requires license purchase → Free = No, Paid = Yes

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

**Example:** list (read) + rent (write) + terminate (delete) → Highest = Delete → Mark "Read-only update and/or delete = Yes", others = No

---

### Learning 6: Capabilities - Tools Categorization (Updated 2026-03-27)

**CRITICAL RULE:** Apply the **standard taxonomy** below. These titles are reused consistently across ALL servers — they are not invented per server, not sourced from README, not derived from paths. Classify each tool into the best-fit standard category.

**Standard Taxonomy — Capabilities - Tools:**

| Title | Use for |
|-------|---------|
| `Search & Query Utilities` | get_, list_, search_, find_, query_, fetch_ operations |
| `Project Management` | project-scoped CRUD operations |
| `Issue Management` | issue/ticket/log-related operations |
| `Team & Workspace Metadata` | user, org, team, workspace info retrieval |
| `Admin & Miscellaneous` | admin ops, token info, access control |
| `Other` | tools that don't fit any category above |

**Standard Taxonomy — Non-Read-Only Tools:**

| Title | Use for |
|-------|---------|
| `Import/Export Tools` | upload, download, transfer, sync operations |
| `Content Management` | create/update/delete stored content or files |
| `Configuration Tools` | install, setup, configure, deploy operations |
| `Other Write Operations` | write/delete ops that don't fit above categories |

**Classification Rules:**
- Assign each tool to the BEST-FIT category from the standard taxonomy
- A tool can only appear in ONE category
- If a tool fits multiple → use the more specific one (e.g. "Import/Export" over "Other Write Operations")
- `Other` / `Other Write Operations` = catch-all for unclassifiable tools

**❌ WRONG:**
```
❌ "Media Generation" (custom title — use "Other" instead)
❌ "Video Generation & Management" (invented — not in taxonomy)
❌ "GPU Management" (invented — not in taxonomy)
❌ "Task Management" (invented — not in taxonomy, use "Search & Query Utilities" for get/list, "Other Write Operations" for cancel/delete)
```

**✅ CORRECT (Runway API MCP example):**
```
Capabilities - Tools:
  Search & Query Utilities → runway_getTask, runway_getOrg
  Other → runway_generateVideo, runway_generateImage, runway_upscaleVideo, runway_editVideo, runway_cancelTask

Non-Read-Only Tools:
  Other Write Operations → runway_generateVideo, runway_generateImage, runway_upscaleVideo, runway_editVideo, runway_cancelTask
```

---

### Learning 7: Capabilities & Non-Read-Only Tools — All Five Rows Always Present

**❌ WRONG:** Omitting any of these five rows when content is not found or not applicable

**✅ CORRECT:** All five rows are MANDATORY in every CSV report — use `"None"` as the value when content is unavailable

**Five mandatory rows (NEVER omit any):**

| Row | Attribute | When to use "None" |
|-----|-----------|-------------------|
| `Capabilities - Tools` | `detailed_info` | Server exposes no tools (Capabilities,Tools = No) |
| `Capabilities - Resources` | `detailed_info` | Server exposes no resources (Capabilities,Resources = No) |
| `Capabilities - Prompts` | `detailed_info` | Server exposes no prompts (Capabilities,Prompts = No) |
| `Capabilities - Sampling` | `detailed_info` | Server has no sampling capability (Capabilities,Sampling = No) |
| `Non-Read-Only Tools` | `detailed_info` | All tools are read-only OR no write/delete tools found |

**Rules:**
- If content found → Include with exact categorized details using `detailed_info` attribute
- If not found / not applicable → Row value = `"None"` (attribute is still `detailed_info`)
- NEVER omit any of these five rows — all are mandatory fields in every report
- NEVER use `No` as the Attribute value — always use `detailed_info`

**Example (server with tools only, no resources/prompts/sampling, read-only):**
```csv
Capabilities - Tools,detailed_info,"Search & Query Utilities
  • list_items – List all items"
Capabilities - Resources,detailed_info,"None"
Capabilities - Prompts,detailed_info,"None"
Capabilities - Sampling,detailed_info,"None"
Non-Read-Only Tools,detailed_info,"None"
```

---

---

## Integration Rules

**Session Start (MANDATORY — never skip):**
- Read `references/learned-fixes.md` all error patterns (#1-#4, #7-#11)
- Read Learnings 1-7 in this file
- These are not suggestions — they are gates. Skipping them causes the same mistakes to recur.

**During Research (enforce at every attribute):**
- Verify Transport Protocol FIRST, then determine TLS (never reverse)
- Verify Pricing = server license, NOT service cost (ask: "Is the MCP server itself paid?")
- Verify Categories from source documentation (never invent)
- Verify Tools Operations = HIGHEST level only (never mark multiple)
- All five capability rows ALWAYS present with attribute "detailed_info": Capabilities-Tools, Capabilities-Resources, Capabilities-Prompts, Capabilities-Sampling, Non-Read-Only Tools — use "None" for any that have no content
- Every Yes/No MUST have source evidence — if missing, flag to user

**Workflow Gates:**
- Step 3 decision: Ask user preference (local vs. research)
- If local setup: Execute Steps 3.1-3.6 before research
- If connection fails: Auto-trigger Phase 1 diagnosis
- After error fix: Continue research from interrupted point
- Security check: Every config change requires approval

**Session End (MANDATORY — never skip):**
- Run Gate 3 Learning Walkthrough (all Learnings checked)
- Run Gate 4 Self-Improvement (document new patterns if any)
- Only THEN create final CSV

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

## Report Format: CSV Structure

**Output Location:** `~/Documents/mcp-reports/<servername>.csv` (user-configured)

**Format:** CSV with exactly 3 columns: `Category, Attribute, Status`

**CSV Requirements:**
- Use Python's `csv.writer` with `quoting=csv.QUOTE_ALL`
- All cells quoted, even Yes/No values
- Newlines in multiline cells preserved via proper CSV escaping
- **Output CSV ONLY — no markdown (.md) file**

**Row Order (Exact Sequence):**

```
MCP Info
├── Description (2–3 sentences, MUST start with "[Server Name] MCP Server…")
├── Git Repo Version (v1.2.3 format — latest only)
│   Source priority: 1st → GitHub Releases  2nd → GitHub Tags  3rd → package.json
│   Use version EXACTLY as shown in source (NO notes, NO transformation)
│   If no version found after checking all 3 sources → use "No"
├── Category (File Management / Developer Tools / Data Retrieval / Productivity / Other)
├── GitHub Repository (URL or N/A)
└── Endpoint URL (URL or N/A)

Distribution Type (Official / Community)
MCP Protocol Version (2025-11-25 / 2025-06-18 / 2025-03-26 / 2024-11-05)
Pricing (Free / Paid)
Hosting Provider (SaaS Vendor / 3rd Party SaaS / GitHub / GitLab / Bitbucket / SourceHut/Gitea/Gogs)
Authentication (OAuth 2.1 AuthCode / OAuth 2.1 Client Creds / Bearer / PAT / API Token)
Data Protection (TLS 1.3 / TLS 1.2 / Lower versions or no encryption)
Transport Protocol (STDIO / HTTP/SSE / StreamableHttp / FastAPI)
Tools Operations (Read-only / R+Update / R+Update+Delete)
Deployment Approach (Local / Container / Remote)
Compliance & Certifications (HIPAA / GDPR / SOC 2 / FedRAMP)
Capabilities (Tools / Resources / Prompts / Sampling)
Capabilities - Tools (detailed_info — ALWAYS present; use "None" if server has no tools)
Capabilities - Resources (detailed_info — ALWAYS present; use "None" if server has no resources)
Capabilities - Prompts (detailed_info — ALWAYS present; use "None" if server has no prompts)
Capabilities - Sampling (detailed_info — ALWAYS present; use "None" if server has no sampling capability)
Non-Read-Only Tools (detailed_info — ALWAYS present; use "None" if all tools are read-only or no write/delete tools found)
```

**Example Rows (minimal — see `references/csv-example.md` for full Telnyx example):**

```csv
MCP Info,Description,"Telnyx MCP Server connects AI agents with Telnyx telephony, messaging, and AI assistant workflows. It provides 50+ tools across SMS/MMS messaging, voice call control, AI assistant creation, cloud storage, embeddings, webhook management, and integration secrets handling."
MCP Info,Git Repo Version,"v0.1.2"
Distribution Type,Official,Yes
Authentication,API Token,Yes
Capabilities - Tools,detailed_info,"Category Name
  • tool_name – Description of what it does

Another Category
  • another_tool – Another description"
Capabilities - Resources,detailed_info,"None"
Capabilities - Prompts,detailed_info,"None"
```

**Formatting Rules (Critical):**

1. **Category headers** — Plain text, no brackets, no special formatting
2. **Bullets** — Exactly 2 spaces before `•`, exactly 1 space after `•` (i.e. `  • tool_name`)
3. **Connector (Capabilities - Tools)** — Use `–` (en-dash, NOT hyphen, NOT colon) between tool name and description
3b. **Connector (Non-Read-Only Tools)** — Use `:` (colon, NOT en-dash) between tool name and description
4. **Blank lines** — Always separate category blocks with blank line
5. **Tool names** — Use actual registered function names (not generic labels)
6. **Descriptions** — Concise, action-oriented (start with verb: "Create", "Get", "List", "Send")
7. **Multiline cells** — CSV escaping handles newlines, no manual escaping needed
8. **Attributes** — Final CSV must be Yes/No only. During research, use `UNVERIFIED` for unconfirmed attributes — resolve ALL to Yes/No before generating CSV (per Gate 1)
9. **Description field (CRITICAL — format + corruption prevention)** — The `MCP Info,Description` cell MUST:
   - **Always start with `[Server Name] MCP Server`** (e.g., "Todoist MCP Server bridges AI assistants with…")
   - Sentence 1: What the server connects/bridges (AI agents ↔ what platform/service)
   - Sentence 2–3: Key features and capabilities (specific tools, workflows, or integrations)
   - Be a **single continuous line** with NO embedded newlines (multi-sentence joined with spaces, not line breaks)
   - Escape any double quotes inside the description as `""`
   - If the description contains newlines or unescaped quotes, it will break the CSV row structure and overwrite subsequent Status column values.

   ❌ WRONG (newline breaks row — corrupts subsequent Status values):
   ```
   MCP Info,Description,"First sentence.
   Second sentence."
   ```
   ✅ CORRECT (single line):
   ```
   MCP Info,Description,"First sentence. Second sentence. Third sentence."
   ```

**Example Bad vs Good:**

❌ WRONG:
```
Capabilities - Tools,detailed_info,"- AI Assistants: create_assistant, list_assistants"
```

✅ RIGHT (Capabilities - Tools):
```
Capabilities - Tools,detailed_info,"AI Assistants
  • create_assistant – Create new AI assistant
  • list_assistants – List all existing AI assistants"
```

✅ RIGHT (Non-Read-Only Tools):
```
Non-Read-Only Tools,detailed_info,"Write Operations
  • create_assistant: Create a new AI assistant
  • delete_assistant: Delete an existing AI assistant"
```

> **Full CSV example:** See `references/csv-example.md` for complete Telnyx MCP Server report with all rows.

---

## Report Generation Rules (Single & Multi-Server)

### Rule 1 — Per-Server CSV Naming
Each server gets its own report: `~/Documents/mcp-reports/<mcp-name>.csv`
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
→ Mark SaaS Vendor = Yes
→ Mark GitHub = Yes
→ Note: SaaS Vendor is the PRIMARY hosting (takes display priority in comparisons)
```
Both are marked Yes because both are real hosting locations. SaaS Vendor takes priority only for sorting/display in multi-server comparisons, not for exclusion.

---

## Learned Fixes — Error Case Studies

> **All error patterns with full details are in `references/learned-fixes.md`.**
> Learnings 1-7 above contain the authoritative rules extracted from those errors.
> Read learned-fixes.md before each research session for real-world case studies.

**Error patterns documented:** #1-#4 (Buildkite/Hyperbolic), #7-#8 (Runway), #9-#11 (CSV format/capability rows)
**Covers:** Protocol version, Authentication, Tools Operations, Deployment, TLS, Pricing, Git Version, CSV Description, Capability Rows

---

## Final Report Output

**On successful CSV generation, display:**
```
╔═══════════════════════════════════════════════════════════════════╗
║   REPORT SUCCESSFULLY GENERATED                                   ║
║   CSV  : /path/to/report/<servername>.csv                         ║
║   COST : /path/to/report/<servername>-cost.txt                    ║
╚═══════════════════════════════════════════════════════════════════╝
```

Both files are always saved together. The cost file contains:
- MCP server name
- Research date and model
- Token breakdown (input, output, cache read)
- Per-category cost and total cost
