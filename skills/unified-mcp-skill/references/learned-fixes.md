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

## Test Cases (Verification)

### Test Case #1: Buildkite MCP (Go SDK v1.4.1)
**Verifying:** Error patterns #1, #2, #3, #4

Expected Results:
- ✅ Protocol: 2025-06-18 (not 2025-11-25)
- ✅ Authentication: Bearer Token = Yes, PAT = Yes, API Token = Yes
- ✅ Tools Operations: Read+Update+Delete = Yes (not multiple)
- ✅ Deployment: Local = Yes, Container = Yes

---

## Version History

| Version | Date | Changes |
|---------|------|---------|
| 2.0 | 2026-03-26 | Initial learned-fixes.md created with 4 error patterns from Buildkite research |

---

**Last Updated:** 2026-03-26
**Skill Version:** Unified MCP Skill 2.0 (Self-Learning)
**Status:** Active — Auto-referenced in research workflows

