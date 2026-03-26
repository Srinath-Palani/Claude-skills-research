# UPGRADE v2.0.2 — Comprehensive Learnings & Critical Rules

**Date:** 2026-03-27
**Status:** Complete & Locked
**Severity:** CRITICAL — All rules must be followed for all future research sessions

This document consolidates all learnings, error corrections, and critical rules discovered during the Runway API MCP server research session (2026-03-27). These rules are now **LOCKED** and apply to ALL future MCP research across unified-mcp-skill-new and multi-server.md.

---

## Overview: 8 Error Patterns Documented

During comprehensive MCP research, 8 distinct error patterns were identified and corrected. Each includes root cause analysis, verification checklist, and prevention rules.

| # | Error Pattern | Severity | Session | Status |
|---|---|---|---|---|
| 1 | Protocol Version Mapping Error | HIGH | Buildkite | ✅ Documented |
| 2 | Authentication Fields Marked No | HIGH | Buildkite | ✅ Documented |
| 3 | Tools Operations Violation | MEDIUM | Buildkite | ✅ Documented |
| 4 | Deployment Container Marked No | HIGH | Buildkite | ✅ Documented |
| 5 | TLS Encryption vs Bearer Token Confusion | **CRITICAL** | Runway | ✅ Documented |
| 6 | Git Repo Version — Wrong Source Priority | MEDIUM | Runway | ✅ Documented |
| 7 | Pricing Attribute Confusion | HIGH | Runway | ✅ Documented |
| 8 | Capabilities Tools Category Invention | HIGH | Runway | ✅ Documented |

---

## Critical Rule #1: TLS Encryption ≠ Bearer Token Authentication

**Severity:** CRITICAL
**Session:** Runway API MCP (2026-03-27)
**What Went Wrong:** Marked TLS 1.3 = Yes and TLS 1.2 = Yes for STDIO-only server with no remote endpoint.

### The Fundamental Mistake

```
❌ WRONG LOGIC: "Bearer Token authentication required → TLS must be Yes"
✅ CORRECT LOGIC: Bearer Token ≠ TLS (completely separate layers)
```

### Why They're Independent

| Layer | Concept | When Applicable | Example |
|-------|---------|-----------------|---------|
| **Authentication** | Bearer Token, API Token, PAT | All transports (STDIO, HTTP/SSE, etc.) | `Authorization: Bearer <token>` |
| **Transport Encryption** | TLS 1.3, TLS 1.2 | ONLY remote transports (HTTP/SSE, StreamableHttp) | HTTPS protocol for network |
| **Local Process** | STDIO | No encryption needed | stdin/stdout on local machine |

### Verification Checklist (MANDATORY)

```
BEFORE marking ANY TLS field:

□ Step 1: Identify Transport Protocol FIRST
   - STDIO? → All TLS = No (STOP HERE)
   - HTTP/SSE or StreamableHttp? → Continue to Step 2

□ Step 2: If remote transport detected
   - Verify endpoint exists with curl tests:
     curl -I https://endpoint/mcp        (GET test)
     curl -X POST https://endpoint/mcp   (POST test)
   - If endpoint responds → Check response format
   - If endpoint 404 → TLS = No

□ Step 3: NEVER check Authentication for TLS decision
   - Bearer Token presence ≠ TLS presence
   - API Token presence ≠ TLS presence
   - These are independent layers

□ Step 4: Mark TLS only if:
   - Transport = HTTP/SSE OR StreamableHttp AND
   - Remote endpoint verified AND
   - Response format indicates real MCP endpoint

□ Step 5: When in doubt
   - If no endpoint tested → TLS = No
   - If STDIO transport → TLS = No (ALWAYS)
```

### Real-World Examples

| Scenario | Transport | Endpoint | TLS 1.3 | TLS 1.2 | Why |
|----------|-----------|----------|---------|---------|-----|
| Runway | STDIO | N/A | **No** | **No** | Local process, no network |
| Slack MCP | HTTP/SSE | https://api.slack.com | Yes | Yes | Remote endpoint, TLS required |
| GitHub MCP | STDIO | N/A | No | No | Local process, no network |

### Rule (LOCKED — NEVER BREAK)

```
🔒 TLS encryption applies ONLY to remote transports
🔒 Bearer Token authentication is INDEPENDENT of TLS
🔒 STDIO = always TLS No (no exceptions)
🔒 Never mark TLS Yes without endpoint verification
🔒 Authentication layer ≠ Encryption layer
```

---

## Critical Rule #2: Git Repo Version — Source Priority & Format

**Severity:** MEDIUM
**Session:** Runway API MCP (2026-03-27)
**What Went Wrong:** Added documentation to version string: `v1.0.0 [ No GitHub Releases | No git tags | package.json source ]`

### The Fundamental Mistake

```
❌ WRONG: Version with documentation/notes
   "v1.0.0 (from package.json, no git tags available)"
   "v1.0.0 [ No GitHub Releases | No git tags | package.json source ]"

✅ CORRECT: Version EXACTLY as shown in source (NO transformation)
   "1.0.0" (from package.json)
   "v1.0.0" (from GitHub Releases)
   "release-1.0" (from git tags)
```

### Source Priority (Use First Match)

```
1st → GitHub Releases (official releases page)
   Extract: tag_name field EXACTLY
   Format: Use as shown (v1.0.0, 1.0.0, release-1.0, etc.)

2nd → GitHub Tags (git tags in repository)
   Extract: latest tag name EXACTLY
   Format: Use as shown

3rd → package.json (fallback only if no releases/tags)
   Extract: "version" field EXACTLY
   Format: Use as shown

NEVER → server initialize response
   Do NOT use protocol version responses
   Do NOT fabricate versions
```

### Verification Checklist (MANDATORY)

```
BEFORE marking Git Repo Version:

□ Step 1: Check GitHub Releases API
   curl https://api.github.com/repos/ORG/REPO/releases
   Extract: tag_name (latest release) — USE EXACTLY AS SHOWN
   If found → STOP, copy version exactly

□ Step 2: Check GitHub Tags
   git tag -l (or git describe --tags)
   Extract: latest tag — USE EXACTLY AS SHOWN
   If found → STOP, copy version exactly

□ Step 3: Check package.json
   Extract: "version" field — USE EXACTLY AS SHOWN
   If found → STOP, copy version exactly

□ Step 4: Copy to CSV (NO modifications, NO documentation added)
   Just the version string: "1.0.0" or "v2.5.1" or "release-1.0"
   NEVER add: brackets, pipes, documentation, notes, explanations
```

### Real Examples

| Server | Releases | Tags | package.json | **Correct Result** |
|--------|----------|------|--------------|---|
| Runway | ❌ None | ❌ None | 1.0.0 | **"1.0.0"** |
| Buildkite | ❌ None | ✅ v1.2.0 | v1.1.5 | **"v1.2.0"** |
| Slack | ✅ v2.5.1 | ✅ v2.5.1 | v2.5.1 | **"v2.5.1"** |
| Custom | ✅ release-3.0 | ✅ tag-3.0 | 2.5.0 | **"release-3.0"** |

### Rule (LOCKED — STRICT)

```
🔒 Always check Releases FIRST
🔒 Check Tags SECOND if no Releases
🔒 Use package.json LAST as fallback
🔒 Copy version EXACTLY as shown (NO transformation)
🔒 Never add documentation, brackets, pipes, or verification notes
🔒 Just the version string, period.
```

---

## Critical Rule #3: Pricing — Server Licensing, NOT Service Costs

**Severity:** HIGH
**Session:** Runway API MCP (2026-03-27)
**What Went Wrong:** Marked Pricing = Paid because Runway API service costs money (independent concern).

### The Fundamental Mistake

```
❌ WRONG LOGIC: "Runway API key required + Runway service is paid → Pricing = Paid"
✅ CORRECT LOGIC: Pricing = MCP server licensing (open-source = Free, proprietary = Paid)
```

### What Pricing Attribute Measures

| Scenario | Free | Paid | Why |
|----------|------|------|-----|
| **Open-source server, free service** | **Yes** | No | Server is open-source (you can run it locally) |
| **Open-source server, paid service** | **Yes** | No | Server is open-source; service costs are separate |
| **MIT licensed MCP with Runway API** | **Yes** | No | Server is free; downstream API calls cost money |
| **Paid server (requires license)** | No | **Yes** | Can only use by paying vendor |
| **Closed-source vendor server** | No | **Yes** | Vendor controls access, charges fees |

### What Pricing Does NOT Measure

```
❌ API key authentication requirement (use Authentication fields instead)
❌ Downstream service costs (Runway rentals, GPU fees, API charges)
❌ Vendor subscription levels or premium features
❌ Installation requirements or resource costs
```

### Verification Checklist (MANDATORY)

```
BEFORE marking Pricing:

□ Step 1: Check server licensing
   - Is the MCP server itself open-source? (GitHub, MIT, Apache, etc.)
   - OR is it proprietary/closed-source?

□ Step 2: Ignore downstream service costs
   - Runway AI generation costs? IRRELEVANT
   - GPU rental fees? IRRELEVANT
   - API call charges? IRRELEVANT
   - Only measure: Can you run the MCP server for free?

□ Step 3: Verify vendor documentation
   - Check for licensing statement (MIT, Apache, Commercial, etc.)
   - Check for terms of service
   - Check for paid/subscription indicators for the SERVER itself

□ Step 4: Mark correctly
   - If server is open-source → Free = Yes, Paid = No
   - If server requires payment → Free = No, Paid = Yes
   - If unsure → Check LICENSE file in repository
```

### Real-World Example

**Runway API MCP Server:**
```
Repository: https://github.com/runwayml/runway-api-mcp-server
License: MIT (open-source)
Server Cost: $0 (you can run it locally)
Runway AI Service: Paid (separate concern)

CORRECT: Free = Yes, Paid = No
WRONG: Free = No, Paid = Yes
```

### Rule (LOCKED)

```
🔒 Pricing = MCP server licensing ONLY
🔒 Open-source/MIT = Free (yes)
🔒 Proprietary/Commercial = Paid (yes)
🔒 Never confuse with API key requirement (auth layer)
🔒 Never include downstream service costs (separate concern)
🔒 When doubt exists, check LICENSE file in repository
```

---

## Critical Rule #4: Capabilities - Tools Category Titles — NEVER Assume

**Severity:** HIGH
**Session:** Runway API MCP (2026-03-27)
**What Went Wrong:** Invented category titles based on assumptions instead of source documentation.

### The Fundamental Mistake

```
❌ WRONG (INVENTED):
   "Video Generation & Management"
   "Image Generation"
   "Task Management"

✅ CORRECT (FROM SOURCE):
   "Team & Workspace Metadata"
   "Search & Query Utilities"
   "Issue Management"
```

### Category Title Source Priority

**MANDATORY ORDER (Do not skip):**

1. **User-provided research documentation** (most accurate, explicit categorization)
2. **README official tool groupings** (vendor's own organization)
3. **Source code file structure** (folder names, decorator comments)
4. **Vendor API documentation** (official reference)
5. **Last resort: Infer from tool names** (only if NO other source available)

### Verification Checklist (MANDATORY)

```
BEFORE creating Capabilities - Tools category titles:

□ Step 1: Search user research documentation
   - Is there a provided list of tool categories?
   - Use EXACTLY as provided (no reorganization)

□ Step 2: Check README for "Tools", "Capabilities", "Features"
   - Are tools grouped in sections?
   - Use section headers EXACTLY as shown

□ Step 3: Check source code structure
   - Are tools organized in separate files or folders?
   - Do decorators have category comments?
   - Extract folder/file names or comments

□ Step 4: Check vendor API documentation
   - Is there an official API reference?
   - Are endpoints grouped by category?
   - Use official categories EXACTLY

□ Step 5: Never invent categories
   - Don't create logical groupings
   - Don't reorganize provided categories
   - Don't assume hierarchical structure
   - Use ONLY what's documented

□ Step 6: Document source
   - Note where each category came from
   - Verify against learned-fixes.md patterns
```

### Real-World Examples

**Runway API MCP (WRONG vs CORRECT):**

```
❌ WRONG (Invented):
   Video Generation & Management
   Image Generation
   Task Management

✅ CORRECT (From README):
   Video Generation & Management
   Image Generation
   Task Management
   ...when explicitly organized in source

❌ What Actually Happened:
   - Runway README DOES organize tools
   - I invented "Task Management" when README said "Task"
   - I formatted headers instead of copying exactly
```

**Slack MCP (Correct Example):**

```
✅ CORRECT (From source):
   Team & Workspace Metadata
   Search & Query Utilities
   Conversation Management
   File Management
   User Profile Management
   Other

NEVER:
   ❌ "Team Information" (slightly different from source)
   ❌ "Search Tools" (renamed from source)
   ❌ Reorganize into: "Reading", "Writing", "Deleting"
```

### Distinction: Capabilities - Tools vs Non-Read-Only Tools

**Capabilities - Tools:**
- Shows ALL tools available on the server
- Organized exactly as source documents them
- Format: `• tool_name – Description`
- Uses en-dash (`–`) separator

**Non-Read-Only Tools:**
- Shows ONLY tools that modify state (write/delete operations)
- Subset of all tools, organized by operation type
- Format: `• tool_name: Description (operation type)`
- Uses colon (`:`) separator
- Notes write/delete/state-change operations

### Rule (LOCKED — STRICT)

```
🔒 NEVER assume category titles
🔒 NEVER invent logical groupings
🔒 NEVER reorganize documented categories
🔒 Extract from source documentation ONLY
🔒 Use category titles EXACTLY as shown in source
🔒 Document source for each category (README line #, file name, etc.)
🔒 When in doubt, ask user for source documentation
```

---

## Critical Rule #5: Hosting Provider Priority — SaaS Vendor Takes Precedence

**Severity:** HIGH
**Session:** Runway API MCP (2026-03-27)
**What Went Wrong:** Marked both SaaS Vendor = Yes and GitHub = Yes when both were available.

### The Priority Rule

```
IF multiple hosting providers found:
  SaaS Vendor (official managed service) = HIGHEST PRIORITY
  ↓
  3rd Party SaaS (Vercel, Netlify, etc.)
  ↓
  GitHub / GitLab / Bitbucket / SourceHut
```

**When SaaS Vendor is available:**
- Mark SaaS Vendor = Yes
- Mark GitHub/GitLab = No (even if source is hosted there)
- Rationale: Primary deployment is vendor-managed, not community-hosted

### Verification Checklist (MANDATORY)

```
BEFORE marking Hosting Provider:

□ Step 1: Is there a vendor-managed SaaS offering?
   - https://vendor.com/mcp endpoint?
   - Vendor documentation mentioning "managed service"?
   - Official vendor infrastructure (not 3rd party)?
   → If YES: Mark SaaS Vendor = Yes

□ Step 2: Is source code on GitHub?
   - github.com/org/repo path exists?
   → Only mark if SaaS Vendor = No

□ Step 3: Is source on GitLab/Bitbucket/SourceHut?
   - gitlab.com or bitbucket.org or sr.ht?
   → Only mark if SaaS Vendor = No

□ Step 4: Mark only the PRIMARY hosting provider
   - If both SaaS and GitHub available → mark SaaS only
   - If only GitHub available → mark GitHub
   - Multiple selections allowed ONLY within same tier
```

### Real-World Example

**Runway API MCP:**
```
Status: SaaS vendor provides managed endpoint
Repository: https://github.com/runwayml/runway-api-mcp-server

CORRECT:
  SaaS Vendor = Yes
  GitHub = No (reason: vendor manages primary deployment)

WRONG:
  SaaS Vendor = Yes
  GitHub = Yes (both marked — violates priority rule)
```

### Rule (LOCKED)

```
🔒 SaaS Vendor = highest priority (marks as YES)
🔒 If SaaS available, mark GitHub = NO (unless explicitly vendor code)
🔒 Only mark GitHub if NO SaaS vendor offering
🔒 Priority order: SaaS → 3rd Party SaaS → GitHub/GitLab/etc
```

---

## Critical Rule #6: Protocol Version — Numeric Comparison, Never Assumption

**Severity:** CRITICAL
**Session:** Buildkite MCP (2026-03-26)
**What Went Wrong:** Marked Protocol Version = 2025-06-18 without verifying Go SDK version 1.4.1 against mapping table.

### The Fundamental Mistake

```
❌ WRONG LOGIC: "Latest protocol must be 2025-11-25" (guessed)
✅ CORRECT LOGIC: "SDK 1.4.1 < 1.12, so maps to 2025-06-18" (verified)
```

### Protocol Version Mapping Tables

**TypeScript @modelcontextprotocol/sdk:**
```
sdk ≥1.12 → 2025-11-25
sdk 1.8–1.11 → 2025-06-18
sdk <1.8 → 2024-11-05
```

**Go github.com/modelcontextprotocol/go-sdk:**
```
go-sdk ≥1.12 → 2025-11-25
go-sdk 1.4–1.11 → 2025-06-18  ← COMMON MISS
go-sdk <1.4 → 2024-11-05
```

**Python mcp:**
```
mcp ≥1.24 → 2025-11-25
mcp 1.15–1.23 → 2025-06-18
mcp <1.15 → 2024-11-05
```

**FastMCP (Python):**
```
fastmcp ≥0.4.x → 2025-06-18
fastmcp 0.3.x → 2025-03-26
fastmcp <0.3 → 2024-11-05
```

### Verification Checklist (MANDATORY)

```
BEFORE marking Protocol Version:

□ Step 1: Extract exact SDK version from dependency file
   - TypeScript: package.json "@modelcontextprotocol/sdk"
   - Go: go.mod "github.com/modelcontextprotocol/go-sdk"
   - Python: requirements.txt or pyproject.toml "mcp" or "fastmcp"
   Record: Exact version string (e.g., "1.4.1", "0.4.2")

□ Step 2: Identify framework/SDK type
   - FastMCP present? → Use FastMCP mapping
   - Other framework present? → Use framework mapping
   - Otherwise → Use base SDK mapping

□ Step 3: DO NUMERIC COMPARISON against mapping table
   - Is 1.4.1 < 1.12? YES → maps to 2025-06-18
   - Is 1.4.1 ≥ 1.12? NO
   - Is 1.4.1 between 1.4 and 1.11? YES → maps to 2025-06-18
   NEVER guess or assume

□ Step 4: Document the source
   - File: go.mod, package.json, requirements.txt, pyproject.toml
   - Line number or exact reference
   - Version found: "1.4.1"
   - Mapping rule applied: "1.4–1.11 → 2025-06-18"

□ Step 5: Verify against learned-fixes.md
   - Check Error Pattern #1: Protocol Version Mapping
   - Confirm you did numeric comparison
   - Not a guess or assumption
```

### Real Examples

**Buildkite (WRONG vs CORRECT):**
```
Found: Go SDK v1.4.1 in go.mod

❌ WRONG (Guessed):
   "Latest protocol = 2025-11-25"
   Marked: MCP Protocol Version 2025-11-25 = Yes

✅ CORRECT (Verified):
   Step 1: Extract v1.4.1
   Step 2: Go SDK type
   Step 3: 1.4.1 < 1.12? YES
   Step 4: 1.4–1.11 → 2025-06-18
   Marked: MCP Protocol Version 2025-06-18 = Yes
```

### Rule (LOCKED — CRITICAL)

```
🔒 STRICTLY DO NOT MAP without checking numeric comparison
🔒 DO NUMERIC COMPARISON, NOT ASSUMPTION
🔒 Extract exact version from dependency file
🔒 Cross-reference against mapping table
🔒 If version is in between ranges, use LOWER protocol
🔒 Document source file + line number where version found
🔒 Check learned-fixes.md Error #1 before finalizing
```

---

## Critical Rule #7: Formatting — Capabilities Tools vs Non-Read-Only Tools

**Severity:** HIGH
**Session:** Runway API MCP (2026-03-27)
**What Went Wrong:** Used same format for both, when they should use different separators.

### Format Distinction

**Capabilities - Tools (ALL tools):**
```
Category Name
    •    tool_name – Description (action-oriented verb)
    •    tool_name – Description

Another Category
    •    tool_name – Description
```
- Format: `– ` (en-dash + space)
- Shows: ALL tools regardless of operation type
- Organized: Exactly as source documents them

**Non-Read-Only Tools (WRITE operations only):**
```
Category Name
    •    tool_name: Description (operation type)
    •    tool_name: Description

Another Category
    •    tool_name: Description
```
- Format: `: ` (colon + space)
- Shows: ONLY write/delete operations
- Organized: By operation type (create, update, delete, modify)

### Real Example (Supabase - Correct vs Wrong)

**❌ WRONG (Same format for both):**
```
Capabilities - Tools:
    •    create_database – Create new database
    •    list_databases – List all databases

Non-Read-Only Tools:
    •    create_database – Creates database
    •    delete_database – Deletes database
```

**✅ CORRECT (Different formats):**
```
Capabilities - Tools:
    •    create_database – Create new database
    •    list_databases – List all databases
    •    delete_database – Delete a database

Non-Read-Only Tools:
    •    create_database: Creates database (write operation)
    •    delete_database: Deletes database (delete operation)
```

### Verification Checklist

```
BEFORE finalizing CSV:

□ Capabilities - Tools:
   - Uses en-dash separator: " – "
   - Lists ALL tools (read + write)
   - Descriptions are action-oriented

□ Non-Read-Only Tools:
   - Uses colon separator: ": "
   - Lists ONLY write/delete tools
   - Includes operation type in description
   - Only present if write operations exist

□ Format consistency:
   - No mixing of separators
   - Same indentation for both (4 spaces)
   - Same bullet style (•)
```

### Rule (LOCKED)

```
🔒 Capabilities - Tools: Format with " – " (en-dash)
🔒 Non-Read-Only Tools: Format with ": " (colon)
🔒 Never mix formats
🔒 Never use same format for both
🔒 Indentation: 4 spaces before •, 4 spaces after •
```

---

## Critical Rule #8: Authentication — Bearer Token is Delivery Method, Not Credential Type

**Severity:** HIGH
**Session:** Multiple sessions
**What Went Wrong:** Marked Bearer Token = No when API Token = Yes, treating them as exclusive.

### The Independence Principle

```
Bearer Token = DELIVERY METHOD (how credentials are sent via HTTP)
  Authorization: Bearer <token>

PAT / API Token / OAuth = CREDENTIAL TYPE (what credentials are used)
  Personal Access Token
  API Key/Token
  OAuth 2.1 tokens
```

**They are INDEPENDENT — multiple can be Yes simultaneously.**

### Authentication Decision Table

| Auth Type | Mark Yes When | Delivery | Co-occurrence |
|-----------|---------------|----------|---------------|
| **Bearer Token** | `Authorization: Bearer` in source/docs (HTTP header) | HTTP Bearer header | Always Yes when PAT/OAuth/API key uses Bearer |
| **Personal Access Token** | "personal access token" / "PAT" in docs; user-specific | Via Bearer header | → Also Bearer Token = Yes |
| **API Token** | "api key" / "api_token" in source/docs; app-level | Via Bearer header OR env var | → Bearer Token = Yes ONLY if Bearer header |
| **OAuth 2.1 AuthCode** | OAuth 2.1 auth code flow documented | Via Bearer header | → Bearer Token = Yes |
| **OAuth 2.1 Client Creds** | Machine-to-machine OAuth 2.1 | Via Bearer header | → Bearer Token = Yes |

### Real Examples

**Postmark MCP:**
```
API Token in: environment variable (POSTMARK_API_TOKEN)
HTTP delivery: X-Postmark-Server-Token header (NOT Bearer)

Correct:
  Bearer Token = No
  API Token = Yes
  (Token is delivered via custom header, not Bearer)
```

**Telnyx MCP:**
```
API Token in: environment variable (TELNYX_API_KEY)
HTTP delivery: Authorization: Bearer <token>

Correct:
  Bearer Token = Yes
  API Token = Yes
  (Token is delivered via Bearer header)
```

**Slack MCP:**
```
Personal Access Token documented
HTTP delivery: Authorization: Bearer <token>

Correct:
  Bearer Token = Yes
  Personal Access Token = Yes
  (Both describe same authentication flow)
```

### Verification Checklist

```
BEFORE marking Authentication:

□ Step 1: Search server.json for TOKEN, API_KEY, SECRET
□ Step 2: Search README for "authentication", "credentials", "token"
□ Step 3: Search .env.example for credential fields
□ Step 4: Check vendor docs for API access token page
□ Step 5: Identify delivery method
   - Bearer header? Mark Bearer Token = Yes
   - Custom header (X-API-Key)? Mark Bearer Token = No
   - Environment variable only? Mark Bearer Token = No
□ Step 6: Identify credential type
   - PAT explicitly mentioned? Mark PAT = Yes
   - "api_key" mentioned? Mark API Token = Yes
   - OAuth flow documented? Mark OAuth = Yes
□ Step 7: If multiple auth types → mark all that apply (NOT exclusive)
```

### Rule (LOCKED)

```
🔒 Bearer Token is a delivery method, not a credential type
🔒 PAT, API Token, OAuth are credential types
🔒 Multiple auth fields can be Yes simultaneously
🔒 Never confuse these layers
🔒 Mark ALL types that apply (union, not intersection)
🔒 Check learned-fixes.md Error #2 before finalizing
```

---

## Pre-Submission Verification Checklist

**MANDATORY — Use before creating ANY final CSV report:**

```
BEFORE FINALIZING REPORT:

Protocol Version:
□ Extract exact SDK/framework version from dependency file
□ Cross-reference against UPGRADE_v2.0.2.md mapping table
□ Do numeric comparison (NOT assumption)
□ Verify learned-fixes.md Error #1 checklist

Authentication:
□ Search server.json for TOKEN/API_KEY/SECRET
□ Search README for "authentication" and "credentials"
□ Search .env.example for credential fields
□ Mark ALL applicable auth types (not exclusive)
□ Verify learned-fixes.md Error #2 checklist

TLS Encryption:
□ Identify transport protocol (STDIO/HTTP/SSE/StreamableHttp)
□ STDIO? → All TLS = No (STOP)
□ HTTP/SSE? → Verify remote endpoint exists
□ Test with curl: GET and POST to verify endpoint
□ Only mark TLS Yes if remote endpoint verified
□ Verify learned-fixes.md Error #7 checklist

Pricing:
□ Check server licensing (open-source vs proprietary)
□ Check LICENSE file in repository
□ DO NOT confuse with API key requirement
□ DO NOT include downstream service costs
□ Verify UPGRADE_v2.0.2.md Critical Rule #3

Git Repo Version:
□ Check GitHub Releases first
□ Check GitHub Tags second
□ Use package.json last (fallback)
□ Copy version EXACTLY (no transformation)
□ NO documentation, brackets, pipes added
□ Verify UPGRADE_v2.0.2.md Critical Rule #2

Capabilities - Tools:
□ Extract category titles from source only
□ Never invent, assume, or reorganize
□ Use en-dash separator: " – "
□ Verify learned-fixes.md Error #8 checklist
□ Verify UPGRADE_v2.0.2.md Critical Rule #4

Tools Operations:
□ Look for cancel_*, delete_*, remove_* patterns
□ Mark ONLY the highest operation level
□ Mark exactly ONE level as Yes (mutually exclusive)
□ Verify learned-fixes.md Error #3 checklist

Hosting Provider:
□ If SaaS Vendor available → mark only SaaS
□ Mark GitHub only if no SaaS option
□ Priority: SaaS > 3rd Party SaaS > GitHub/etc
□ Verify UPGRADE_v2.0.2.md Critical Rule #5

Deployment Approach:
□ Search for Dockerfile and docker images
□ Check server.json for transport/container refs
□ Check for OCI registry references
□ Mark ALL applicable deployment approaches
□ Verify learned-fixes.md Error #4 checklist

Non-Read-Only Tools:
□ Only include if write operations exist
□ Use colon separator: ": "
□ Format: tool_name: Description (operation type)
□ Do NOT use en-dash like Capabilities - Tools
□ Verify format distinction rules

Final Review:
□ Cross-reference all attributes against SKILL.md Learning sections
□ Verify no assumptions made
□ Verify all sources documented
□ Check for consistency with previous MCP research
□ Run SYNC_GUIDE.md verification before commit
```

---

## How to Use This Document

### For Every MCP Research Session

1. **Start:** Read UPGRADE_v2.0.2.md (this file) — all 8 Critical Rules
2. **Before Research:** Review applicable sections from learned-fixes.md
3. **During Research:** Apply verification checklists from each rule section
4. **Before CSV Creation:** Run Pre-Submission Verification Checklist (above)
5. **After Research:** If new pattern discovered → document to learned-fixes.md

### If You Make a Mistake

1. Identify which Critical Rule was violated
2. Read the rule section in UPGRADE_v2.0.2.md
3. Review the verification checklist
4. Re-research with correct approach
5. Document the mistake pattern if it's new

### If You Discover a New Pattern

1. Document root cause in learned-fixes.md
2. Create verification checklist
3. Add to Prevention Checklists section
4. Update UPGRADE_v2.0.2.md if applicable
5. Update multi-server.md if it affects batch research

---

## Summary: Critical Rules Matrix

| Rule # | Title | Severity | Key Principle | Prevention |
|--------|-------|----------|---------------|------------|
| 1 | TLS ≠ Bearer Token | **CRITICAL** | Transport encryption ≠ auth delivery | Endpoint verification checklist |
| 2 | Git Version Format | MEDIUM | Copy exactly, no transformation | Source priority table |
| 3 | Pricing Attribute | HIGH | Server licensing only, not service costs | License file check |
| 4 | Category Titles | HIGH | Extract from source, never invent | Source priority checklist |
| 5 | Hosting Priority | HIGH | SaaS takes precedence over GitHub | Tier priority list |
| 6 | Protocol Version | **CRITICAL** | Numeric comparison, not assumption | Mapping table + calc |
| 7 | Format Distinction | HIGH | en-dash for tools, colon for non-read-only | Format checklist |
| 8 | Authentication | HIGH | Bearer is delivery; multiple types OK | Layer independence check |

---

## Status & Enforcement

**Status:** ✅ COMPLETE & LOCKED (2026-03-27)

**Enforcement:**
- All 8 Critical Rules are **LOCKED** — do not change without explicit user approval
- All rules apply to **ALL future MCP research** across unified-mcp-skill-new and multi-server.md
- Pre-Submission Verification Checklist is **MANDATORY** before CSV creation
- This document is the source of truth for attribute documentation standards

**Last Updated:** 2026-03-27
**Version:** 2.0.2 (Self-Learning Complete)
**Files Updated:** SKILL.md, multi-server.md, learned-fixes.md, MEMORY.md
