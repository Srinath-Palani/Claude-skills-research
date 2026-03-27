# Unified MCP Skill 2.0 — Learned Fixes & Error Patterns

**Self-learning database for preventing recurring research mistakes.**

> These patterns were identified from real MCP research sessions and documented to prevent future errors in attribute research.
>
> **Error Index:** #1-#4 (Buildkite/Hyperbolic, 2026-03-26), #7-#8 (Runway, 2026-03-27), #9-#11 (CSV format/capability rows, 2026-03-27), #12-#13 (Stadia Maps — description newlines + Protocol Version/Pricing blank Status, 2026-03-27)
> Patterns #5-#6 are documented as SKILL.md Learnings 5-6 (no separate case study needed).
> Authoritative rules for all patterns live in SKILL.md Learnings 1-7.

---

## Error Pattern #1: Protocol Version Mapping Error

**Signals:**
- Marked latest protocol (e.g., 2025-11-25) without checking actual SDK version
- Assumed "newest version = latest protocol" without cross-reference
- Missing Go SDK mapping in version lookup table

**Root Cause:**
Did not cross-reference SDK version against protocol version mapping table. Workflow step exists but was skipped during execution.

**Scenario:** Buildkite MCP Server (Go SDK v1.4.1)
- ❌ Marked as: MCP Protocol Version 2025-11-25 = Yes
- ✅ Should be: MCP Protocol Version 2025-06-18 = Yes
- Why: Go SDK v1.4.1 is < 1.12, maps to 2025-06-18, not latest

**Fix (Verification Checklist):**
```
1. Extract exact SDK version from: go.mod, requirements.txt, package.json
2. Cross-reference against mapping table:
   - Go SDK v1.4–1.11 → 2025-06-18
   - Go SDK ≥1.12 → 2025-11-25
3. NEVER assume latest protocol
4. Document source file + line number where version found
5. Before finalizing, check learned-fixes.md for this pattern
```

**Prevention Rule:**
- Always look up the version mapping BEFORE marking protocol
- If version is in between ranges, use LOWER protocol version
- Go SDK mapping must be included in version lookup table

**Date Added:** 2026-03-26 | **Category:** Protocol Verification | **Confidence:** HIGH

---

## Error Pattern #2: Authentication Fields Marked No (When Required)

**Signals:**
- All authentication fields marked as "No" when credentials ARE actually required
- Did not check server.json or config files
- Only searched README (incomplete source check)

**Root Cause:**
Incomplete source investigation. Focused on README only, missed server.json which contains explicit API token requirements.

**Scenario:** Buildkite MCP Server
- ❌ Marked as: Bearer Token = No, PAT = No, API Token = No
- ✅ Should be: All three = Yes (BUILDKITE_API_TOKEN required)
- Why: server.json explicitly defines: `BUILDKITE_API_TOKEN (Required, Secret)`

**Evidence Pattern:**
- Evidence source: `server.json` line: `BUILDKITE_API_TOKEN (Required, Secret)`
- Documentation link: `https://buildkite.com/user/api-access-tokens`
- Pattern: Bearer Token delivery (HTTP header) for PAT + API Token

**Fix (Verification Checklist):**
```
1. Search server.json for "TOKEN", "API_KEY", "SECRET", "BEARER"
2. Search README for "authentication", "credentials", "token"
3. Search .env.example for credential field definitions
4. Search vendor documentation for API access / token page
5. If ANY credential requirement found → mark ALL applicable auth types = Yes
6. Document exact source file + quote where credential found
```

**Prevention Rule:**
- MUST check server.json BEFORE marking authentication fields
- MUST search .env.example for credential patterns
- MUST verify vendor documentation for API token requirements
- Authentication = No only if thoroughly searched all sources

**Why All Three Can Be "Yes":**
- Bearer Token = delivery method (how it's sent via HTTP header)
- Personal Access Token = credential type (user-specific from vendor)
- API Token = alternative naming (same credential, different name)
These are not contradictory — all three describe the same authentication flow.

**Date Added:** 2026-03-26 | **Category:** Authentication Detection | **Confidence:** HIGH

---

## Error Pattern #3: Tools Operations Violation (Marked Multiple Exclusive Levels)

**Signals:**
- Marked BOTH "Read+Update" and "Read+Update+Delete" as Yes
- Violated mutually-exclusive attribute rule
- Did not recognize delete operation patterns (cancel_*, delete_*)

**Root Cause:**
Marked both read+update and read+update+delete as Yes, violating the mutual exclusion rule. Also failed to recognize that tool names like `cancel_build` and `cancel_build_job` represent delete operations.

**Scenario:** Buildkite MCP Server
- ❌ Marked as: Read+Update = Yes, Read+Update+Delete = Yes (BOTH marked)
- ✅ Should be: Read+Update = No, Read+Update+Delete = Yes (HIGHEST level only)
- Why: Tool names show `cancel_build`, `cancel_build_job` (delete patterns)

**Evidence Pattern:**
- Tool names containing `cancel_*` indicate delete operations
- Tool names containing `delete_*` indicate delete operations
- Tool names containing `remove_*` indicate delete operations
- Must choose HIGHEST operation level when multiple types exist

**Fix (Verification Checklist):**
```
1. Parse ALL tool names from documentation
2. Search for patterns: create_*, update_*, delete_*, cancel_*, remove_*
3. Count operation types found
4. Mark ONLY the HIGHEST level:
   - If only read: mark "Read-only" = Yes
   - If read + create/update: mark "Read+Update" = Yes
   - If read + delete/cancel: mark "Read+Update+Delete" = Yes
5. Verify only ONE level is marked Yes
6. Check learned-fixes.md for this pattern
```

**Prevention Rule:**
- Tools Operations is MUTUALLY EXCLUSIVE — mark exactly ONE level
- Always look for delete/cancel patterns in tool names
- When doubt exists, choose the HIGHEST level found
- Document which tool names indicate delete capability

**Cancel/Delete Pattern Dictionary:**
- `cancel_*` → Delete operation
- `delete_*` → Delete operation
- `remove_*` → Delete operation
- `destroy_*` → Delete operation
- `unblock_*` → Write operation (state change)
- `archive_*` → Delete operation
- `purge_*` → Delete operation

**Date Added:** 2026-03-26 | **Category:** Tools Operations | **Confidence:** HIGH

---

## Error Pattern #4: Deployment Container Marked No (When Docker Support Exists)

**Signals:**
- Container deployment marked No when Docker image is available
- Did not check server.json for transport configuration
- Did not search for Dockerfile or OCI registry references
- Only checked README (incomplete source check)

**Root Cause:**
Incomplete source investigation. Focused on README deployment section, missed server.json which explicitly specifies docker runtime and Dockerfile in repository.

**Scenario:** Buildkite MCP Server
- ❌ Marked as: Local = Yes, Container = No
- ✅ Should be: Local = Yes, Container = Yes
- Why: Found Dockerfile.local, OCI registry ghcr.io/buildkite/buildkite-mcp-server, docker image v0.13.0

**Evidence Pattern:**
- Evidence sources:
  - `server.json` line: `"transport": "docker run"`
  - File exists: `Dockerfile.local`
  - Docker image: `buildkite/buildkite-mcp-server:0.13.0`
  - OCI registry: `ghcr.io/buildkite/buildkite-mcp-server`

**Fix (Verification Checklist):**
```
1. Search repository for Dockerfile, Dockerfile.local, docker.json
2. Search server.json for "docker", "container", "image", "run"
3. Search for OCI registry references (ghcr.io, docker.io, etc.)
4. Search for published images (Docker Hub, ECR, etc.)
5. If ANY Docker evidence found → mark Container = Yes
6. Deployment Approach allows MULTIPLE selections (not exclusive)
7. Mark both Local AND Container if both supported
8. Document all Docker evidence sources
```

**Prevention Rule:**
- MUST check server.json transport section
- MUST search for Dockerfile in repository root + all subdirectories
- MUST look for OCI registry and Docker image references
- Deployment Approach is NOT mutually exclusive — multiple approaches possible
- Container = No only if thoroughly searched and found no Docker evidence

**Deployment Approach Rules:**
- Local + Container can BOTH be Yes (not exclusive)
- Local + Remote can BOTH be Yes (not exclusive)
- Container + Remote typically not both Yes
- Always mark ALL supported approaches

**Date Added:** 2026-03-26 | **Category:** Deployment Detection | **Confidence:** HIGH

---

## Verification Workflow (Before Submitting Reports)

**Apply these checks to catch all 4 error patterns:**

### Pre-Submission Checklist
```
BEFORE finalizing any MCP research report:

Protocol Version:
□ Extract exact SDK version from dependency file
□ Cross-reference against version mapping table
□ Look up Go SDK mapping if applicable
□ Compare against learned-fixes.md Error #1

Authentication:
□ Search server.json for TOKEN/API_KEY/SECRET
□ Search README for "authentication" and "credentials"
□ Search .env.example for credential fields
□ Check vendor docs for API access token page
□ Compare against learned-fixes.md Error #2

Tools Operations:
□ Parse all tool names from documentation
□ Search for cancel_*, delete_*, remove_* patterns
□ Choose HIGHEST operation level found
□ Verify only ONE level marked Yes
□ Compare against learned-fixes.md Error #3

Deployment:
□ Search for Dockerfile, Dockerfile.local, docker.json
□ Check server.json for docker/container references
□ Search for OCI registry (ghcr.io, docker.io, etc.)
□ Mark ALL applicable deployment approaches
□ Compare against learned-fixes.md Error #4
```

---

## Self-Learning Integration Points

**These patterns are checked automatically at:**

1. **Step 1 — Protocol Version Verification** (SKILL.md section "Step 1")
   - Verification checklist added
   - Go SDK mapping documented
   - Common mistakes listed

2. **Step 5.1 — Authentication Verification** (SKILL.md section "Step 5.1")
   - Comprehensive checklist
   - Common mistake patterns
   - "Why All Three Can Be Yes" explanation

3. **Step 5.2 — Tools Operations Verification** (SKILL.md section "Step 5.2")
   - Mutually exclusive rule enforcement
   - Delete pattern recognition dictionary
   - Common mistake scenarios

4. **Step 5.3 — Deployment Approach Verification** (SKILL.md section "Step 5.3")
   - Docker detection checklist
   - Multiple deployment support explanation
   - Common oversight patterns

---

## How to Use This File

**When starting MCP research:**
1. Read the error patterns in this file
2. Use the verification checklists BEFORE marking attribute values
3. Reference the "Prevention Rules" for each attribute type
4. Check the "Common Mistake Patterns" section before finalizing

**When discovering a new pattern:**
1. Document the pattern with exact scenario
2. Add root cause analysis
3. Create a verification checklist
4. Link to SKILL.md steps that need updating
5. Update the Pre-Submission Checklist above

**When fixing related code:**
- Update corresponding SKILL.md section
- Add prevention rules
- Test against real MCP servers
- Document test results

---

## Error Pattern #7: TLS Encryption vs Bearer Token Confusion (CRITICAL)

**Date:** 2026-03-27 | **Server:** Runway API MCP | **Severity:** CRITICAL

**Signals (What Went Wrong)**
- Marked TLS 1.3 = Yes and TLS 1.2 = Yes for STDIO server
- No remote endpoint exists (Endpoint URL = N/A)
- Transport Protocol = STDIO (local process only)
- Mistakenly assumed Bearer Token authentication implies TLS encryption

**Root Cause:** Conflated two INDEPENDENT concepts:
- Bearer Token = **Authentication delivery method** (how credentials are sent)
- TLS Encryption = **Transport layer encryption** (how data is protected in transit)

**Critical Misunderstanding:**
```
❌ WRONG LOGIC: "Bearer Token present → TLS must be Yes"
✅ CORRECT LOGIC: Bearer Token ≠ TLS (completely separate layers)
```

**The Correct Framework:**

| Layer | Concept | When Applicable |
|-------|---------|-----------------|
| **Authentication** | Bearer Token, API Token, PAT | All transports (STDIO, HTTP/SSE, etc.) |
| **Transport Encryption** | TLS 1.3, TLS 1.2 | ONLY remote transports (HTTP/SSE, StreamableHttp) |
| **Local Process** | STDIO | NO TLS needed (stdin/stdout on local machine) |

**Why Runway was wrong:**
1. Transport = STDIO (local process, no network)
2. Endpoint URL = N/A (no remote endpoint)
3. STDIO → automatic TLS = No (ALWAYS)
4. Bearer Token is auth layer (irrelevant to TLS decision)

**Fix (Verification Checklist - MANDATORY):**
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

**Real-World Examples:**

| Scenario | Transport | Endpoint | TLS 1.3 | TLS 1.2 | Why |
|----------|-----------|----------|---------|---------|-----|
| Runway (this case) | STDIO | N/A | No | No | Local process, no network |
| Slack MCP | HTTP/SSE | https://api.slack.com | Yes | Yes | Remote endpoint, TLS required |
| GitHub MCP | STDIO | N/A | No | No | Local process, no network |
| Telnyx (SaaS) | HTTP/SSE | https://api.telnyx.com | Yes | Yes | Remote endpoint, TLS required |

**Prevention Rule (LOCKED - NEVER BREAK):**
```
🔒 CRITICAL: TLS encryption applies ONLY to remote transports
🔒 Bearer Token authentication is INDEPENDENT of TLS
🔒 STDIO = always TLS No (no exceptions)
🔒 Never mark TLS Yes without endpoint verification
🔒 Authentication layer ≠ Encryption layer
```

**Result:**
- TLS 1.3 = No ✅
- TLS 1.2 = No ✅
- Bearer Token = Yes ✅ (separate concern)

**Session:** Runway API MCP Server (2026-03-27)

---

## Test Cases (Verification)

### Test Case #1: Buildkite MCP (Go SDK v1.4.1)
**Verifying:** Error patterns #1, #2, #3, #4

Expected Results:
- ✅ Protocol: 2025-06-18 (not 2025-11-25)
- ✅ Authentication: Bearer Token = Yes, PAT = Yes, API Token = Yes
- ✅ Tools Operations: Read+Update+Delete = Yes (not multiple)
- ✅ Deployment: Local = Yes, Container = Yes

### Test Case #2: Runway API MCP (TypeScript SDK v1.12.1)
**Verifying:** Error patterns #7 and #8 (TLS vs Bearer Token + Git Repo Version Priority)

Expected Results:
- ✅ Transport: STDIO = Yes
- ✅ TLS 1.3 = No (STDIO = no network)
- ✅ TLS 1.2 = No (STDIO = no network)
- ✅ Bearer Token = Yes (independent from TLS)
- ✅ Endpoint URL = N/A
- ✅ Authentication does NOT determine TLS encryption
- ✅ Git Repo Version: Check Releases first, Tags second, package.json last
- ✅ Version string copied EXACTLY as shown in source (NO notes, NO brackets, NO transformation)

---

## Error Pattern #8: Git Repo Version — Wrong Source Priority

**Date:** 2026-03-27 | **Server:** Runway API MCP | **Severity:** Medium

**Signals (What Went Wrong)**
- Marked Git Repo Version without checking GitHub Releases first
- Used package.json but didn't document what was checked (implicit assumption)
- Did not show verification order (Releases → Tags → package.json)

**Root Cause:** Did not follow **Rule: Git Repo Version Priority**
```
Source Priority:
1st → GitHub Releases (official releases page)
2nd → GitHub Tags (git tags in repository)
3rd → package.json (fallback only if no releases/tags)
NEVER → server initialize response
```

**Rule from SKILL.md (lines 1000-1001):**
```
Git Repo Version (v1.2.3 format — latest only)
Source priority: 1st → GitHub Releases  2nd → GitHub Tags  (never use server initialize response)
```

**What was wrong:**
```
❌ WRONG: Added documentation/verification notes to version string
   "v1.0.0 (from package.json, no git tags available)"
   "v1.0.0 [ No GitHub Releases | No git tags | package.json source ]"

✅ RIGHT: Use version EXACTLY as it appears in source (NO transformation)
   From package.json: 1.0.0 → Use "1.0.0"
   From Releases: v1.0.0 → Use "v1.0.0"
   From Tags: release-1.0 → Use "release-1.0"
```

**Fix (Verification Checklist - MANDATORY):**
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
```

**Real Examples:**

| Server | Releases | Tags | package.json | Correct Result |
|--------|----------|------|--------------|----------------|
| Runway | ❌ None | ❌ None | 1.0.0 | "1.0.0" |
| Buildkite | ❌ None | ✅ v1.2.0 | v1.1.5 | "v1.2.0" |
| Slack | ✅ v2.5.1 | ✅ v2.5.1 | v2.5.1 | "v2.5.1" |
| Custom | ✅ release-3.0 | ✅ tag-3.0 | 2.5.0 | "release-3.0" |

**Prevention Rule (STRICT - NO EXCEPTIONS):**
```
🔒 Always check Releases FIRST
🔒 Check Tags SECOND if no Releases
🔒 Use package.json LAST as fallback
🔒 Copy version EXACTLY as shown (NO transformation)
🔒 Never add documentation, brackets, pipes, or verification notes
🔒 Just the version string, period.
```

**Result (Runway):**
```
MCP Info,Git Repo Version,"1.0.0"
```

**Session:** Runway API MCP Server (2026-03-27)

---

## Error Pattern #9: Description Field Corrupts CSV Status Column

**Date:** 2026-03-27 | **Servers:** keeper-mcp-golang-docker, axiomhq-mcp, mcp-croit-ceph, mercadolibre-mcp-server | **Severity:** HIGH

**Signals (What Went Wrong)**
- Description text contained embedded newlines (written as multi-line cell)
- Subsequent rows (Git Repo Version, Category, etc.) appeared corrupted or overwritten in the Status column
- CSV file opened incorrectly in spreadsheet tools — rows were misaligned

**Root Cause:**
The `MCP Info,Description` cell was written with embedded newlines. CSV parsers interpret unescaped newlines inside quoted cells as row breaks, causing all subsequent content to shift columns and overwrite Status values in those rows.

**What was wrong:**
```
❌ WRONG (newline inside cell breaks row structure):
MCP Info,Description,"First sentence.
Second sentence covering capabilities."
MCP Info,Git Repo Version,"v1.0.0"   ← this gets absorbed into description cell
```

**Fix:**
```
✅ CORRECT (single continuous line, spaces instead of newlines):
MCP Info,Description,"First sentence. Second sentence covering capabilities."
MCP Info,Git Repo Version,"v1.0.0"   ← now correctly its own row
```

**Prevention Rule (STRICT):**
```
🔒 Description MUST be a single line — NO embedded newlines
🔒 Join sentences with spaces, not line breaks
🔒 Any double quotes inside description text must be escaped as ""
🔒 Verify the Description row is ONE CSV row before saving
```

**Reference:** SKILL.md Formatting Rule #9

---

## Error Pattern #11: Missing Capability Rows (Capabilities-Tools / Resources / Prompts / Non-Read-Only)

**Date:** 2026-03-27 | **Servers:** trackmage-mcp-server, odos-mcp, mercadolibre-mcp-server | **Severity:** HIGH

**Signals (What Went Wrong)**
- `Capabilities - Resources,detailed_info` row missing when server had no resources
- `Capabilities - Prompts,detailed_info` row missing when server had no prompts
- `Non-Read-Only Tools,detailed_info` row missing when all tools were read-only

**Root Cause:**
Learning 7 originally only protected Non-Read-Only Tools. The three Capabilities rows lacked the same "always present" mandate, so when content was absent they were silently omitted.

**Fix:**
All five rows are MANDATORY in every CSV report regardless of content. Attribute is always `detailed_info` — never `No` or blank:

```
✅ ALWAYS include all five (attribute = detailed_info):
Capabilities - Tools,detailed_info,"<tools OR None>"
Capabilities - Resources,detailed_info,"<resources OR None>"
Capabilities - Prompts,detailed_info,"<prompts OR None>"
Capabilities - Sampling,detailed_info,"<sampling OR None>"
Non-Read-Only Tools,detailed_info,"<write tools OR None>"

❌ WRONG:
Capabilities - Sampling,No
Capabilities - Sampling,No,
```

**Prevention Rule:**
```
🔒 Capabilities - Tools     → always present, "None" if Capabilities,Tools = No
🔒 Capabilities - Resources  → always present, "None" if Capabilities,Resources = No
🔒 Capabilities - Prompts    → always present, "None" if Capabilities,Prompts = No
🔒 Capabilities - Sampling   → always present, "None" if Capabilities,Sampling = No
🔒 Non-Read-Only Tools       → always present, "None" if all read-only
🔒 Attribute for ALL five    → always "detailed_info", never "No"
```

**Reference:** SKILL.md Learning 7, Gate 3 L7

---

## Error Pattern #10: Git Repo Version — "UNVERIFIED" Instead of "No"

**Date:** 2026-03-27 | **Servers:** keeper-mcp-golang-docker, axiomhq-mcp, mcp-croit-ceph, mercadolibre-mcp-server | **Severity:** Low

**Signals (What Went Wrong)**
- After checking all 3 sources (Releases → Tags → package.json), no version was found
- Field was left as `UNVERIFIED` in the final CSV instead of being resolved

**Root Cause:**
The general ZERO-ASSUMPTION policy says to use `UNVERIFIED` when a value cannot be confirmed. But Git Repo Version has a defined fallback — when no version exists in any source, the correct value is `No`, not `UNVERIFIED`.

**Fix:**
```
□ Step 1: Check GitHub Releases → not found
□ Step 2: Check GitHub Tags → not found
□ Step 3: Check package.json → not found
□ Step 4: Mark as "No" (do NOT leave as UNVERIFIED)
```

**CSV output:**
```
✅ CORRECT:
MCP Info,Git Repo Version,"No"

❌ WRONG:
MCP Info,Git Repo Version,"UNVERIFIED"
```

**Prevention Rule:**
```
🔒 Git Repo Version fallback when nothing found = "No"
🔒 Never leave Git Repo Version as UNVERIFIED in final CSV
```

**Reference:** SKILL.md Row Order — Git Repo Version

---

## Error Pattern #12: Description Newlines Cause MCP Protocol Version and Pricing Rows to Disappear

**Date:** 2026-03-27 | **Server:** Stadia Maps MCP Server | **Severity:** HIGH

**Signals (What Went Wrong)**
- `MCP Protocol Version` rows missing from report (no Yes/No visible in any version row)
- `Pricing` rows missing from report (Free = ?, Paid = ?)
- Description appears truncated or continues into rows below it
- Rows that should follow Description (Git Repo Version, Category, Distribution Type, MCP Protocol Version, Pricing…) are displaced or show description text as their Status value

**Root Cause:**
This is a **cascade failure from Error Pattern #9**. The `MCP Info,Description` cell was written with embedded newlines. The CSV parser splits the description into multiple rows at each newline. Every row that should come after Description gets displaced — their Status column is either blank or filled with the continuation of the description text. The rows appear "missing" even though the row labels exist, because their Status values were overwritten.

**What Happened (Stadia Maps MCP):**
```
❌ WRITTEN (with newline inside description):
MCP Info,Description,"Stadia Maps MCP Server (TypeScript) provides AI assistants with access to Stadia Maps API.
It supports map tiles, routing, geocoding, and place search."

← Parser sees TWO rows:
   Row A: MCP Info | Description | "Stadia Maps MCP Server... Stadia Maps API."
   Row B: (empty) | (empty)      | "It supports map tiles..."   ← spurious row

← Rows that follow (Distribution Type, MCP Protocol Version, Pricing) are still
   in the file but their Status column shows "" or description overflow — not Yes/No
```

**Correct Fix:**
```
✅ CORRECT (single continuous line):
MCP Info,Description,"Stadia Maps MCP Server (TypeScript) provides AI assistants with access to Stadia Maps API. It supports map tiles, routing, geocoding, and place search."

← Parser sees ONE row — all subsequent rows land in correct columns
```

**Prevention Rule (STRICT):**
```
🔒 Description = single line ALWAYS — NO embedded newlines
🔒 Join all sentences with a space, not a line break
🔒 VERIFY: after writing Description row, check that MCP Protocol Version
   rows appear immediately after Distribution Type with Yes/No values
🔒 If Protocol Version or Pricing rows are missing → description has newlines (fix first)
```

**Gate 3 Check (L8 — added):**
```
□ L8 Description Single-Line: Is MCP Info,Description a single unbroken line?
  → Open raw CSV in text editor — description must end on same line it starts
  → If MCP Protocol Version or Pricing rows appear empty → fix description first
  → NO embedded newlines — sentences joined with spaces only
```

**Reference:** Error Pattern #9 (root cause), SKILL.md Formatting Rule #9, Gate 3 L8

---

## Error Pattern #13: MCP Protocol Version and Pricing — Blank or Overwritten Status Values

**Date:** 2026-03-27 | **Server:** Stadia Maps MCP Server | **Severity:** HIGH

**Signals (What Went Wrong)**
- One MCP Protocol Version row marked Yes, but remaining version rows left blank or omitted
- Pricing row `Free = Yes` written, but `Paid` row missing or left blank
- Status column of Protocol Version or Pricing rows contains description text (overwritten by description newline cascade)

**Rules (MANDATORY — no exceptions):**

**MCP Protocol Version:**
```
✅ CORRECT — exactly one Yes, all others explicitly No:
MCP Protocol Version,2025-11-25,No
MCP Protocol Version,2025-06-18,Yes
MCP Protocol Version,2025-03-26,No
MCP Protocol Version,2024-11-05,No

❌ WRONG — blank rows:
MCP Protocol Version,2025-06-18,Yes
(other version rows missing)

❌ WRONG — commentary in Status:
MCP Protocol Version,2025-06-18,"Yes (from SDK v1.4.1)"
```

**Pricing:**
```
✅ CORRECT — both rows always present, status only Yes or No:
Pricing,Free,Yes
Pricing,Paid,No

❌ WRONG — only one row written:
Pricing,Free,Yes
(Paid row missing)

❌ WRONG — commentary in Status:
Pricing,Free,"Yes (MIT License)"
Pricing,Paid,"No (open source)"
```

**Prevention Rule (STRICT):**
```
🔒 Protocol Version: ALL four version rows ALWAYS present — exactly one Yes, rest No
🔒 Pricing: BOTH Free and Paid rows ALWAYS present — exactly one Yes, one No
🔒 Status column for both = ONLY "Yes" or "No" — no text, no notes, no commentary
🔒 Never leave any of these rows blank
🔒 If Status looks wrong → check description row for embedded newlines (Error Pattern #12)
```

**Reference:** SKILL.md Step 5 Mutually Exclusive Attributes, Error Pattern #12

---

## Version History

| Version | Date | Changes |
|---------|------|---------|
| 2.7 | 2026-03-27 | Added Error Pattern #13: Protocol Version and Pricing blank/overwritten Status values |
| 2.6 | 2026-03-27 | Added Error Pattern #12: Description newlines cause Protocol Version + Pricing to disappear — from Stadia Maps MCP research |
| 2.5 | 2026-03-27 | Added Error Pattern #11: Missing Capability rows — all four always mandatory with "None" fallback |
| 2.4 | 2026-03-27 | Added Error Pattern #10: Git Repo Version "UNVERIFIED" → use "No" — from keeper/axiomhq/mcp-croit/mercadolibre research |
| 2.3 | 2026-03-27 | Added Error Pattern #9: Description field corrupts Status column — from keeper/axiomhq/mcp-croit/mercadolibre research |
| 2.2 | 2026-03-27 | Added Error Pattern #8: Git Repo Version — Wrong Source Priority (Medium) — from Runway API MCP research |
| 2.1 | 2026-03-27 | Added Error Pattern #7: TLS Encryption vs Bearer Token Confusion (CRITICAL) — from Runway API MCP research |
| 2.0 | 2026-03-26 | Initial learned-fixes.md created with 4 error patterns from Buildkite research |

---

**Last Updated:** 2026-03-27
**Skill Version:** Unified MCP Skill 3.0.0 (Self-Learning v2.7)
**Status:** Active — Auto-referenced in research workflows
**Critical Rules:** 11 error patterns documented in this file (Patterns #1-#4, #7-#13); authoritative rules in SKILL.md Learnings 1-7

