# Unified MCP Skill 2.0 — Learned Fixes & Error Patterns

**Self-learning database for preventing recurring research mistakes.**

> These patterns were identified from real MCP research sessions and documented to prevent future errors in attribute research.

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

1. **Step 1 - Protocol Version Verification** (SKILL.md line ~175)
   - Verification checklist added
   - Go SDK mapping documented
   - Common mistakes listed

2. **Step 5.1 - Authentication Verification** (SKILL.md line ~327)
   - Comprehensive checklist
   - Common mistake patterns
   - "Why All Three Can Be Yes" explanation

3. **Step 5.2 - Tools Operations Verification** (SKILL.md line ~360)
   - Mutually exclusive rule enforcement
   - Delete pattern recognition dictionary
   - Common mistake scenarios

4. **Step 5.3 - Deployment Approach Verification** (SKILL.md line ~398)
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
- ✅ Document source priority in CSV: "v1.0.0 [ No GitHub Releases | No git tags | package.json source ]"

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

## Version History

| Version | Date | Changes |
|---------|------|---------|
| 2.2 | 2026-03-27 | Added Error Pattern #8: Git Repo Version — Wrong Source Priority (Medium) — from Runway API MCP research |
| 2.1 | 2026-03-27 | Added Error Pattern #7: TLS Encryption vs Bearer Token Confusion (CRITICAL) — from Runway API MCP research |
| 2.0 | 2026-03-26 | Initial learned-fixes.md created with 4 error patterns from Buildkite research |

---

**Last Updated:** 2026-03-27
**Skill Version:** Unified MCP Skill 2.0.2+ (Self-Learning v2.1)
**Status:** Active — Auto-referenced in research workflows
**Critical Rules:** 7 error patterns documented with prevention checklists

