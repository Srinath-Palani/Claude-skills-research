---
name: unified-mcp-skill-new
user-invocable: true
description: >
  All-in-one unified MCP skill: research MCP servers, document attributes,
  conditionally set up locally, diagnose errors inline, and audit projects. Single optimized
  execution path for research, deployment, error recovery, and compliance auditing. Faster,
  fewer tokens, full functionality.
---

# Unified MCP Skill New

unified-mcp-skill-new is a single skill that handles the full MCP server lifecycle — research, attribute documentation, local setup, error recovery, and project audit — with 5-thread parallel search and 4 mandatory gates before generating any report. All rules, formats, and learnings live in this file as the single source of truth.

<!-- SKILL_VERSION: 3.0.5 | Updated: 2026-03-30 -->

---

## Table of Contents

| # | Section |
|---|---------|
| 1 | [Modification Policy & Zero-Assumption](#modification-policy) |
| 2 | [Enforcement Gates 1–4](#gates) |
| 3 | [Rules Summary 1–5](#rules-summary) |
| 4 | [Quick Reference Workflow](#quick-reference) |
| 5 | [Security Mandate — SSRF, Credentials, Network Safety](#security-mandate) |
| 6 | [Step 0 — Configuration & Input Classification](#step-0) |
| 7 | [Step 0.5 — Parallel Search (5 Concurrent Threads)](#step-05) |
| 8 | [Step 1 — Protocol Version Verification](#step-1) |
| 9 | [Steps 2–4 — Endpoint / GitHub Repo / Server Name](#steps-2-4) |
| 10 | [Step 5 — Attribute Filling (5.1–5.13)](#step-5) |
| 11 | [Step 6 — Save Report + Cost File](#step-6) |
| 12 | [Error Recovery — 7 Phases](#error-recovery) |
| 13 | [Project Audit — Compliance Review](#project-audit) |
| 14 | [Multi-Server Mode](#multi-server) |
| 15 | [Integration Rules](#integration-rules) |
| 17 | [Credential Placeholder Map](#credential-map) |
| 18 | [Report Format — CSV Structure](#report-format) |

---

🔒 **SYNC NOTE**
> SKILL.md is the single source of truth for all rules, learnings, and formats.
> `references/multi-server.md` contains ONLY multi-server orchestration logic (M1-M3, Layers 0/1).
> If you change workflow steps, attribute definitions, or report format here → verify multi-server.md references still point correctly.

---

🔒 **SKILL MODIFICATION POLICY — ABSOLUTE ENFORCEMENT**

When modifying ANY part of this skill (format, rule, instruction, workflow step, output template):

1. **Read the full context first** — understand how the existing instruction works before changing it
2. **Change only what was asked** — never silently remove working logic, calculation methods, enforcement rules, or checklists
3. **Preserve the working method** — if changing a format or presentation, keep the underlying rules and instructions intact
4. **No collateral removal** — dropping an instruction while reformatting its container is a violation

> Example: Changing a cost file output format → update the format block only. The calculation method, script reference, and timestamp instructions remain unchanged.

---

🚫 **ZERO-ASSUMPTION POLICY — ABSOLUTE ENFORCEMENT**

**Every Yes/No in the final CSV MUST have a verified source. No exceptions. No guessing. No skipping.**

If you cannot verify an attribute from source → mark it `No` and flag to user. NEVER guess Yes or No.

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

**Output: Display the evidence for ALL attributes in the chat window as part of the Final Report Output (see Evidence Table format in Final Report Output section). Do NOT embed raw evidence in the CSV Status column.**

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
MANDATORY LEARNING WALKTHROUGH — confirm each was applied (full rules in referenced step):

□ L1  Framework Priority      → Step 5.3  Was framework/SDK version extracted? Numeric comparison done?
□ L2  Endpoint Verification   → Step 2    GET + POST tests run? Response format recorded?
□ L3  TLS vs Bearer           → Step 5.7  Transport identified FIRST? STDIO → all TLS = No confirmed? Container = Yes → Lower/None = No confirmed? Mixed-deployment (STDIO + Container) → all 3 TLS rows = No confirmed (Lower/None must NOT be Yes)?
□ L4  Pricing                 → Step 5.4  LICENSE file checked? API key NOT confused with pricing?
□ L5  Tools Ops               → Step 5.9  All tool names parsed? HIGHEST level only marked?
□ L6  Taxonomy Categories     → Step 5.13 Only standard taxonomy titles used? Zero invented categories?
□ L7  Five Mandatory Rows     → Step 5.13 All five detailed_info rows present? "None" used if absent?
□ L8  Description Format      → Step 5.1  Description is 3–4 sentence single-line paragraph? No embedded newlines? No transport/TLS/auth/hosting details included?
□ L9  Capabilities Source     → Step 5.12 Entry point source file read? README alone NOT used?
□ L10 No Placeholders         → Step 5.13 Zero {servername}_ / {prefix}_ in any CSV cell?
```

**If ANY checkbox cannot be confirmed → STOP. Do not create CSV. Resolve first.**

---

---

### RULES 1-5 (Enforcement Summary)

🔒 **RULE 1: SOURCE-ONLY ATTRIBUTES** — Every value from actual documentation. Zero invented content.
🔒 **RULE 2: NUMERIC VERIFICATION** — Protocol version, TLS, pricing, auth all verified with explicit checks.
🔒 **RULE 3: FULL LEARNING WALKTHROUGH** — All Learnings L1–L10 checked every session. No skipping.
🔒 **RULE 4: NO CSV WITHOUT EVIDENCE** — Evidence Ledger must be complete. Empty evidence = blocked.
🔒 **RULE 5: DEFAULT MODEL REQUIRED** — This skill MUST run on the Default model (currently Sonnet 4.6). If a non-default model is active, STOP and instruct the user to switch before proceeding.



---

# Unified MCP Skill — Research, Deploy, Fix, Audit

**All-in-one MCP server management:** Research and document attributes, conditional local setup (clone+install), inline error recovery, and project compliance auditing — all in one unified skill with optimized execution paths.

---

## Quick Reference (Workflow at a Glance)
**⛔ LOCKED — Steps 0–6 and sub-steps 5.1–5.13 are fixed. Do NOT add, remove, rename, or reorder any step without explicit user permission.**

```
Step 0   Config & Input       → Classify input (endpoint/GitHub/name), set paths
Step 0.5 Parallel Search      → 5 threads: repo, files, endpoint probe, deps, compliance
Step 1   Protocol Version     → Map SDK/framework version to protocol date (NUMERIC comparison)
Step 2   Remote Endpoint      → Probe URL, detect transport, test TLS, build config
Step 3   GitHub Repo          → Ask: local setup / remote / research-only? Clone if local
Step 4   Server Name Only     → Search vendor docs + GitHub, resolve to URL
Step 5   Attribute Filling    → Fill CSV top-to-bottom, one pass per section (numbered order)
  5.1    MCP Info             → (1) Name  (2) Description  (3) Category  (4) Git Repo  (5) Endpoint URL
  5.2    Distribution Type    → (1) Official / (2) Community — mutually exclusive
  5.3    MCP Protocol Version → (1) 2025-11-25  (2) 2025-06-18  (3) 2025-03-26  (4) 2024-11-05 — ONE Yes only, from Step 1
  5.4    Pricing              → (1) Free / (2) Paid — mutually exclusive, L4 check
  5.5    Hosting Provider     → (1) SaaS Vendor  (2) 3rd Party SaaS  (3) GitHub  (4) GitLab  (5) Bitbucket  (6) SourceHut/Gitea/Gogs
  5.6    Authentication       → (1) OAuth 2.1 Auth Code  (2) OAuth 2.1 Client Creds  (3) Bearer Token  (4) PAT  (5) API Token
  5.7    Data Protection      → (1) TLS 1.3  (2) TLS 1.2  (3) Lower/None — STDIO → all No; Container = Yes → Lower/None = No; remote → probe result
  5.8    Transport Protocol   → (1) STDIO  (2) HTTP/SSE  (3) StreamableHttp  (4) FastAPI — mark all that apply
  5.9    Tools Operations     → (1) Read-only  (2) Read+Update  (3) Read+Update+Delete — HIGHEST level only
  5.10   Deployment Approach  → (1) Local  (2) Container  (3) Remote — mark all that apply
  5.11   Compliance           → (1) HIPAA  (2) GDPR  (3) SOC 2  (4) FedRAMP
  5.12   Capabilities         → (1) Tools  (2) Resources  (3) Prompts  (4) Sampling — from source code
  5.13   Capability Details   → (1) Capabilities-Tools  (2) Capabilities-Resources  (3) Capabilities-Prompts  (4) Capabilities-Sampling  (5) Non-Read-Only Tools
Step 6   Save Report          → CSV to user-configured path + cost file

On fail → 7-phase Error Recovery (auto-triggered)
2+ servers → Multi-Server mode (see references/multi-server.md)
```

**Enforcement Gates (MANDATORY — block CSV until passed):**
Gate 1=Evidence Ledger (every Yes/No has source proof)
Gate 2=Connection Verified (all 5 threads complete)
Gate 3=Learning Walkthrough (L1-L10 checked)

---

## 🔐 Security Mandate — Always Enforced

### Enterprise Security Quick-Reference (Audit Checklist)

```
NEVER:
□ Ask for credentials in chat
□ Write actual keys/tokens in any config, report, or CSV
□ Probe private IP ranges (127.x, 10.x, 172.16.x, 192.168.x, 169.254.x)
□ Follow redirects to blocked IP ranges
□ Store credentials in README, .env (committed), or settings.json

ALWAYS:
□ Use <PLACEHOLDER> in all config examples
□ Direct user to edit files directly for credentials
□ Follow immutable pattern: show file path → user opens → user pastes → user restarts
□ Validate paths before file write (no .., symlinks, system dirs)
□ Apply curl timeouts: --connect-timeout 5 --max-time 10 --max-redirs 3
□ Revoke + regenerate if credential accidentally appears in chat
```

→ Full rules, alert templates, blocklists, and path validation: detailed below.

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

## Workflow

1. **Classify Input** — endpoint / GitHub URL / server name / package name
2. **Detect Mode** — single server → Steps 0–6; 2+ servers → Multi-Server mode (M1–M3)
3. **Parallel Search** — 5 concurrent threads collect all source data (Step 0.5)
4. **Verify Protocol** — map SDK version to protocol date (Step 1)
5. **Fill Attributes** — one pass top-to-bottom through Steps 5.1–5.13
6. **Pass Gates** — Gate 1 Evidence · Gate 2 Connection · Gate 3 Learnings
7. **Save Report** — CSV + cost file to configured path (Step 6)

---

## Unified Workflow (Steps 0-8 + Error Recovery)

> Steps 0-8 run sequentially. Steps 5.1-5.4 are verification sub-steps within Step 5.
> Error Recovery (7 phases) triggers automatically on connection failure.
> Multi-Server mode (M1-M3) activates when 2+ servers detected — see `references/multi-server.md`.

### Step 0 — Configuration & Input Classification
**⛔ LOCKED — Do not modify this step without explicit user permission.**

#### 🔒 Model Enforcement (MANDATORY — check before anything else)

Before any research, setup, or audit begins, verify the active model:

```
REQUIRED MODEL: Default (recommended) ✔ — currently Sonnet 4.6

If the active model is NOT the Default model:
  → STOP immediately
  → Display:

  ⚠️  MODEL ENFORCEMENT: This skill requires the Default model (Sonnet 4.6).
      You are currently using a non-default model.

      To switch:
        /model  → select "Default (recommended) ✔"

      Re-invoke the skill after switching.

  → Do NOT proceed until the user confirms they have switched to Default.
```

If the active model IS the Default model → proceed to path configuration below.

**Path configuration — always ask, never assume:**

#### Path Validation (MANDATORY before use)
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

Report save path — ask first use; offer to reuse if path was used before:
```
Where should MCP reports be saved?
Enter full path (e.g. /Users/you/Documents/mcp-reports):
```
If a path was used before, offer to reuse:
```
Last used report path: /Users/you/Documents/mcp-reports
[ 1 ] Use this path
[ 2 ] Enter a different path
[ 3 ] Save this as my default path (skip asking next time)
```
Only save as default if user explicitly selects option 3. If a default is saved, use it silently — do NOT ask again.

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

#### Input Classification — Find BOTH sources always

| Input Given | Starting Point | Also MUST Find |
|-------------|---------------|----------------|
| **Remote Endpoint** | Probe endpoint (TLS, auth, transport) | Search GitHub for source repo |
| **GitHub URL** | Read repo (README, deps, tools) | Search for + probe remote endpoint |
| **Server Name** | Search both simultaneously | GitHub URL + remote endpoint |
| **Package Name (npm/PyPI/Cargo)** | Locate source repo and registry listing | Search GitHub for source repo |

#### Mandatory Dual-Source Rule
Regardless of input type, research is NEVER complete with only one source.
- Given GitHub URL → always search for remote endpoint (README, docs, curl probe)
- Given remote endpoint → always find GitHub repo
- Given server name → find both before proceeding
Run both searches in parallel via Step 0.5. Combine results using Report Generation Rules 2–5.

---

## Parallel Search & Data Collection (Critical Learning)

**CRITICAL:** Execute Step 0.5 with 5 CONCURRENT threads (not sequentially).

### Why Parallel Matters

Sequential research misses interdependencies. Endpoint found in Thread 3 reveals framework for Thread 4, which determines protocol for Thread 1. Parallel execution catches everything simultaneously.

### Step 0.5 — 5 Concurrent Threads
**⛔ LOCKED — Do not modify this step without explicit user permission.**

#### Thread 1: GitHub Repository Research *(always run — find repo if not given)*
→ Feeds: **Step 5.1** (Name, Description, Category, GitHub Repo) · **Step 5.2** (Official/Community) · **Step 5.6** (Authentication — README Detection Patterns) · **Step 5.12** (Capabilities source) · **Step 5.13** (tool names)
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
- **Auth scan (README — applies to ALL input types including GitHub repo / STDIO):**
  - Apply Detection Patterns against full README text:
    - API Token: `"api key"`, `"api_token"`, `"api_key"` in README text
    - PAT: `"personal access"`, `"pat"`, `"personal token"`, `"personal access token"`, `"personal_access_token"` in README text
    - Bearer Token: `"Authorization: Bearer"` in README text
  - Also scan README sections for: `"authentication"`, `"credentials"`, `"token required"`, `"requires auth"`
  - If ANY pattern matched → feed Step 5.6; record README section/line in Evidence Ledger; trigger Step 5.6.A **HARD BLOCK** prompt (MUST NOT continue without user response)

#### Thread 2: Repository File Search *(always run)*
→ Feeds: **Step 5.6** (Authentication — server.json, .env.example, config files) · **Step 5.6.B** (Implicit Auth Detection — env / args / headers) · **Built-in Security Controls** (see below)
- Files: .env.example, config.json, setup.md, mcp.json, server.json, docs/
- Grep: "https://", "endpoint", "url", "host" patterns
- API paths: /api/v2, /v1, /mcp
- Comments with example URLs
- **Auth scan (Step 5.6.B — applies to GitHub repo URL and server name input types):**
  - Scan `env` field names for: `TOKEN`, `API_KEY`, `SECRET`, `BEARER`, `AUTH`, `CREDENTIAL`, `ACCESS_KEY`, `PASSWORD`
  - Scan `args` flags for: `--token`, `--api-key`, `--secret`, `--auth`, `--bearer`, `--pat`, `--access-token`
  - Scan `headers` block in any JSON config for: `Authorization: Bearer`, `Authorization: token`, `X-API-Key`, `X-Auth-Token`, `Authorization: Basic`
  - Apply Detection Patterns: API Token (`"api key"`, `"api_token"`, `"api_key"` in combined source text); PAT (`"personal access"`, `"pat"`, `"personal token"` in doc_methods or `"personal access token"`, `"personal_access_token"` in combined text) — mark all matched types Yes
  - If ANY env/args/headers/pattern match found → mark applicable auth attributes Yes in Step 5.6; record field + file in Evidence Ledger; trigger Step 5.6.A **HARD BLOCK** prompt (MUST NOT continue without user response)

#### Built-in Security Controls (Thread 2 — Secondary Scan)
These do not change any CSV attribute values, but must be documented as risk context in the research notes.
```
□ Write gates     — env var flags that disable destructive ops by default
                    (e.g., ALLOW_WRITE=true, READ_ONLY_MODE, --dangerouslyAllowBrowser)
□ Output scrubbing — patterns that strip credentials, PII, or secrets from
                    tool responses before they reach the client
□ Input guards     — regex validation on tool parameters, path traversal
                    prevention, template injection checks
□ Auth wrappers    — permission decorators or functions wrapping tool execution
                    (e.g., secure_tool(), @requires_auth, permission check before call)
```
If any controls found → note them in research summary as: "Security mitigations present: [list]"
If none found → note: "No built-in write gates or response sanitization detected"

#### Thread 3: Remote Endpoint Probing *(run unless short-circuited)*
→ Feeds: **Step 5.1** (Endpoint URL) · **Step 5.6.B** (Implicit Auth Detection — headers) · **Step 5.7** (TLS — L3) · **Step 5.8** (Transport) · **Step 5.10** (Deployment Remote)
- **SHORT-CIRCUIT (with mandatory managed version check):**
  If Thread 1 confirmed STDIO-only AND zero remote/endpoint URLs found in README:
  → **BEFORE marking N/A, run Managed/Hosted Version Discovery (Steps A-E below)**
  → Only after Steps A-E return no results → mark Endpoint URL = N/A, TLS = No.
  → Document: "STDIO-only server, no remote endpoint references in README. Managed version check completed: [results]."

  <!-- Added 2026-03-30: Prevents missing managed/hosted endpoints for STDIO-only servers (e.g., ECS managed MCP server discovery) -->
  **Managed/Hosted Version Discovery (MANDATORY before marking Endpoint URL = N/A):**

  Even if the source code is STDIO-only, the vendor may offer a separate managed/hosted
  version with its own endpoint URL. This is common for cloud providers offering both
  open-source and managed editions, enterprise vendors with cloud-hosted MCP offerings,
  and servers with cloud marketplace listings.

  **Step A — Vendor Documentation Search**
  Search queries to run (replace [server-name] and [vendor-name] with actuals):
  1. `"[server-name]" managed MCP server` (web search)
  2. `"[server-name]" hosted endpoint URL` (web search)
  3. `"[vendor-name]" MCP server managed OR hosted OR cloud` (web search)
  4. Check the vendor's official documentation site (not just the GitHub README)
  5. Search for `[vendor] MCP server preview` or `[vendor] MCP server GA`
  6. Check vendor's cloud marketplace (if applicable) for hosted deployments
  Examples:
  - AWS: `site:docs.aws.amazon.com "[service-name]" MCP` → discovered managed ECS MCP server
  - AWS unified endpoint: `https://aws-mcp.us-east-1.api.aws/mcp` (preview)
  - AWS Knowledge: `https://knowledge-mcp.global.api.aws` (fully managed, no local install)
  - Stripe: check `site:docs.stripe.com MCP` for hosted vs local options
  - GitHub: check `site:docs.github.com MCP server` for Copilot-hosted endpoints

  **Step B — Monorepo / Multi-Package Cross-Reference**
  For servers in a monorepo or multi-package ecosystem:
  1. Check the monorepo root README for a list of managed vs local servers
  2. Look for proxy, gateway, or handler modules that suggest remote deployment
     paths for otherwise STDIO-only servers
  3. Check if sibling servers in the same repo have managed versions — if some
     do, verify whether this server also has one
  Examples:
  - awslabs/mcp monorepo: contains mcp-proxy-for-aws (SigV4 proxy for managed
    endpoints), mcp-lambda-handler (serverless deployment), and individual servers
    where some have managed versions (ecs-mcp-server) and others don't (aws-location)
  - Monorepo root README often lists which servers have managed alternatives

  **Step C — README "Legacy" / "Deprecated" Signals**
  Scan the server README for phrases indicating the local version is superseded:
  - "Legacy", "Option 2", "no longer receive updates", "deprecated"
  - "Recommended: managed version", "fully managed", "cloud-hosted"
  - "Enterprise", "production deployment", "hosted service"
  If found → the managed version's endpoint URL is the PRIMARY endpoint.
  Examples:
  - Amazon ECS MCP Server README labels local install as "Option 2: Local MCP
    Server (Legacy)" and recommends the AWS-managed version as Option 1
  - A vendor README stating "For production use, connect to https://api.vendor.com/mcp"

  **Step D — Self-Hosted HTTP Endpoint Detection**
  If the server supports HTTP transport (StreamableHttp, HTTP/SSE) but has no
  official vendor-hosted endpoint:
  - Document the default self-hosted endpoint pattern (e.g., http://localhost:PORT/mcp)
  - Check if configurable host/port environment variables exist
  - Note any reverse proxy or deployment documentation
  Examples:
  - aws-api-mcp-server: supports StreamableHttp with configurable HOST and PORT
    (default http://127.0.0.1:8000/mcp)
  - Can also be deployed to Amazon Bedrock AgentCore Runtime via AWS Marketplace,
    which provides a remote endpoint with SigV4/JWT auth

  **Step E — Cloud Deployment Platform Check**
  Check if the server can be deployed to managed cloud platforms that provide an endpoint:
  - Serverless: AWS Lambda, Google Cloud Functions, Azure Functions
  - Container platforms: AWS ECS/Fargate, Google Cloud Run, Azure Container Apps
  - Agent platforms: Amazon Bedrock AgentCore, Google Vertex AI, Azure AI Foundry
  - Look for deployment guides, Terraform/CDK templates, or marketplace listings
  Examples:
  - aws-api-mcp-server has an AWS Marketplace listing for AgentCore deployment
  - mcp-lambda-handler module enables any STDIO server to run on AWS Lambda
    behind API Gateway with DynamoDB session management

  **Decision Logic:**
  IF managed/hosted version found:
    Endpoint URL = managed endpoint URL
    Evidence: "Managed version: [URL]; Local version also available via STDIO"
    Update related attributes: Remote=Yes, SaaS Vendor=Yes, Transport, TLS, Auth
  IF managed version NOT found after Steps A-E:
    Endpoint URL = N/A
    Evidence: "No managed/hosted version found. Verified: [list search queries run]"
  IF server supports HTTP but no official hosted endpoint:
    Endpoint URL = N/A
    Evidence: "Self-hosted HTTP mode available ([host:port] configurable) but no
    vendor-hosted endpoint found"
- If only GitHub given → extract candidate endpoint URLs from README/docs first
- **OFFICIAL ENDPOINT VERIFICATION (MANDATORY):** A URL is only a valid official endpoint if it is explicitly documented in the server's own README or official docs. URLs found in third-party blogs, test deployments, or random cloud resources (e.g., AWS API Gateway IDs like `xmfe3hc3pk.execute-api.amazonaws.com`) are NOT official endpoints. If the URL is not in official docs → mark Endpoint URL = N/A. NEVER assume a cloud resource URL is official without README/doc confirmation.
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

- **Auth scan (Step 5.6.B — applies to remote endpoint URL input type):**
  - Check response headers for: `WWW-Authenticate`, `X-API-Key`, `Authorization`
  - Check endpoint JSON config `headers` block for: `Authorization: Bearer`, `Authorization: token`, `X-API-Key`, `X-Auth-Token`, `Authorization: Basic`
  - 401 / 403 response → confirms auth required; inspect `WWW-Authenticate` to identify auth type
  - Apply Detection Patterns: API Token (`"api key"`, `"api_token"`, `"api_key"` in combined source text); PAT (`"personal access"`, `"pat"`, `"personal token"` in doc_methods or `"personal access token"`, `"personal_access_token"` in combined text) — mark all matched types Yes
  - If ANY header/pattern signal found → mark applicable auth attributes Yes in Step 5.6; record header field + endpoint in Evidence Ledger; trigger Step 5.6.A **HARD BLOCK** prompt (MUST NOT continue without user response)

#### Thread 4: Dependency Extraction
→ Feeds: **Step 5.3** (Protocol Version — apply L1 framework priority mapping from Step 1)
- Priority 1: FastMCP version (if present)
- Priority 2: Framework version (Langchain, etc.)
- Priority 3: Base SDK version (mcp, @modelcontextprotocol/sdk)
- Check for HTTP frameworks: FastAPI, Express, Flask
- Dependency files to check: `package.json` (Node), `pyproject.toml` / `requirements.txt` (Python), `Cargo.toml` (Rust), `go.mod` (Go)

#### Thread 5: Vendor Compliance & Security
→ Feeds: **Step 5.11** (Compliance — HIPAA / GDPR / SOC 2 / FedRAMP)
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

### Step 1 — Protocol Version Verification
**⛔ LOCKED — Do not modify this step without explicit user permission.**

**CRITICAL:** Run BEFORE filling attributes. Framework priority takes absolute precedence.

#### Source Priority (use first match)
1. Check dependencies for **FastMCP** (Python) → Use FastMCP mapping
2. Check dependencies for **@modelcontextprotocol/sdk** (TypeScript) → Use SDK mapping
3. Check base **mcp** SDK (Python) → Use mcp mapping

#### Framework Mapping — FastMCP (Python)
- `fastmcp ≥3.0.0` → Protocol `2025-11-25` (FastMCP 3.x pulls in `mcp ≥1.24.0,<2.0` as a hard transitive requirement — the protocol version is governed by the underlying mcp SDK range, not the fastmcp version number itself)
- `fastmcp ≥0.4.x` → Protocol `2025-06-18`
- `fastmcp 0.3.x` → Protocol `2025-03-26`
- `fastmcp <0.3` → Protocol `2024-11-05`

#### SDK Mapping — Python mcp
- `mcp ≥1.24` → Protocol `2025-11-25`
- `mcp 1.15–1.23` → Protocol `2025-06-18`
- `mcp 1.5–1.14` → Protocol `2025-03-26`
- `mcp <1.5` → Protocol `2024-11-05`

#### SDK Mapping — TypeScript (@modelcontextprotocol/sdk)
- `sdk ≥1.24` → Protocol `2025-11-25`  <!-- Verified: v1.24.0 first introduced 2025-11-25 (Dec 2025); confirmed against SDK 1.20.2 (Clarity) and 1.27.1 (M365 Agents Toolkit) -->
- `sdk 1.8–1.23` → Protocol `2025-06-18`
- `sdk 1.5–1.7` → Protocol `2025-03-26`
- `sdk <1.5` → Protocol `2024-11-05`

#### SDK Mapping — Go (github.com/modelcontextprotocol/go-sdk)
- `go-sdk ≥1.12` → Protocol `2025-11-25`
- `go-sdk 1.4–1.11` → Protocol `2025-06-18` ← **COMMON MISS: Don't assume latest**
- `go-sdk <1.4` → Protocol `2024-11-05`

**NOTE:** Protocol `2025-03-26` applies to: FastMCP 0.3.x, TypeScript SDK 1.5–1.7, Python mcp SDK 1.5–1.14. Go SDK has no equivalent band for this version — if only Go SDK is found, `2025-03-26` does NOT apply.

#### Verification Checklist (before marking protocol version final)
```
□ Step 1: Extract exact version from dependency file (go.mod, requirements.txt, package.json)
□ Step 2: Cross-reference version against the mapping table above (DO NOT guess/assume)
□ Step 3: If version is between ranges, use LOWER protocol (e.g., TypeScript sdk 1.20 is <1.24 → use 2025-06-18; Go sdk 1.4.1 is <1.12 → use 2025-06-18)
□ Step 4: Document the exact version + mapping source (file + line number)
□ Step 5: Document the exact version + mapping source (file + line number) in Evidence Ledger
```

#### Common Mistake Patterns
- ❌ WRONG: "Latest protocol must be v2025-11-25" (assumes without checking)
- ✅ RIGHT: "SDK v1.4.1 maps to 2025-06-18" (checked version table)
- ❌ WRONG: "Go SDK in go.mod but no mapping" (missing Go SDK mapping)
- ✅ RIGHT: "Go SDK v1.4.1 → Protocol 2025-06-18 per Go SDK mapping" (complete mapping)

Record: Framework name, exact version, protocol version, mapping source, verification timestamp.

---

### Step 2 — Remote Endpoint URL Path
**⛔ LOCKED — Do not modify this step without explicit user permission.**

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
**⛔ LOCKED — Do not modify this step without explicit user permission.**

#### Deployment Preference

```
How do you want to run this server?

[ 1 ] Locally (clone + install + configure)
      → Full setup: clone, install deps, test connection

[ 2 ] Use remote endpoint (if available)
      → Check README for remote endpoint

[ 3 ] Just research attributes (no setup)
      → Skip local setup, proceed to research only
```

#### Local Setup Sub-steps (Option 1)

| Sub-step | Action | Verify |
|----------|--------|--------|
| 3.1 | Clone repo to configured path (`git clone --depth 1` for research; full clone only if user needs history) | Dir exists, expected files present, path validated |
| 3.2 | Determine connection method | Identify STDIO/Docker/Published from README |
| 3.3 | Get user approval for install | Show deps + env vars, wait for "yes" |
| 3.4 | Install dependencies | Language-specific (Python/Node/Docker) |
| 3.5 | Extract configuration | Read command, args, env vars from README + code |
| 3.6 | Test connection | Send initialize JSON-RPC, verify response |

#### Connection Methods (Step 3.2)

| Method | Indicator | Setup |
|--------|-----------|-------|
| **STDIO** | stdin/stdout mentions in README | Clone + install locally |
| **Published Package** | PyPI/npm package, pip/npx refs | No clone, use published package |
| **Docker** | Dockerfile in repo | Clone + docker build |
| **Remote** | Vendor URL endpoint | Skip all setup |

#### Dependency Installation

| Language | Command | Verify |
|----------|---------|--------|
| Python | `python3 -m venv .venv && pip install -r requirements.txt` | `python -c "import mcp"` |
| Node.js | `npm install && npm run build` | Check `dist/` exists |
| Docker | `docker build -t <repo-name> .` | `docker images \| grep <repo-name>` |
| uv | `uv sync` | `uv run python -c "import mcp"` |

#### Docker Security Check (before building)
- Verify Dockerfile uses pinned base image (e.g., `python:3.12-slim`, NOT `python:latest`)
- If `:latest` tag found → WARN user: "Unpinned base image detected — may introduce supply chain risk"
- Check for `USER` directive (non-root execution preferred)
- Review any `curl | sh` or piped install patterns → WARN user before proceeding

**If Option 3 (Research Only):**
Skip local setup, proceed to attribute research.

---

### Step 4 — Server Name Only
**⛔ LOCKED — Do not modify this step without explicit user permission.**

#### Resolution Order (always top-down — stop at first confirmed match)

0. **STEP 0 — The "Fast Lane" (Known Official Vendor Sources)**
   Check the Known Official Vendor Sources table below **before** running any GitHub search.
   - If the vendor/server name matches a row → go directly to that GitHub org/repo (skip STEP 1–2).
   - Example: `aws` → `awslabs/mcp` → proceed straight to STEP 3 (monorepo extraction).
   - Example: `github` → `github/github-mcp-server` → fetch README directly.
   - **Why:** Avoids topic-filter misses for official repos that don't self-tag (e.g., `awslabs/mcp`).

1. **STEP 1 — The "Source of Truth" (Reference Repo)**
   Target: `https://github.com/modelcontextprotocol/servers`
   - Search the `src/` directory of the official MCP reference repository.
   - **Why:** This is the curated registry of vetted MCP servers that conform to official protocol standards.
   - Logic: Look for subdirectories matching `{name}` or `{vendor}`.

2. **STEP 2 — The "Wide Net" (GitHub Topic + Text Search)**
   Target: GitHub API Repository Search.
   - Primary query: `https://api.github.com/search/repositories?q=topic:mcp-server+{name}&sort=stars`
   - Secondary query (if no results): broaden to `topic:mcp-server+{vendor}`
   - Tertiary query (if still no results): broader text search: `https://api.github.com/search/repositories?q={name}+mcp+server&sort=stars`
   - Also try name patterns: `{vendor}-mcp-server`, `mcp-{vendor}`, `{vendor}-mcp`
   - **Why:** The `topic:mcp-server` filter excludes "Minecraft" but some repos (e.g., `awslabs/mcp`) do not use that topic tag — the text fallback catches these.

3. **STEP 3 — The "Deep Dive" (Monorepo Extraction)**
   Target: Git Trees API for subdirectory discovery.
   - If a repo found in Step 2 is a large vendor monorepo (e.g., `awslabs/mcp`), do NOT assume the root is the server.
   - Request: `https://api.github.com/repos/{owner}/{repo}/git/trees/{branch}?recursive=1`
   - Logic: Scan the file list for paths containing `package.json` or `pyproject.toml` AND the string `mcp` (e.g., `src/google-maps-mcp-server/`).
   - Validation: Fetch `README.md` from that subdirectory to confirm it matches the target server.

4. **STEP 4 — The "Last Resort" (Keyword Pattern Matching)**
   Target: Standard GitHub Search with naming conventions.
   - Query: `https://api.github.com/search/repositories?q={vendor}-mcp-server+OR+mcp-{vendor}`
   - **Why:** Use only if Step 2 fails — some developers omit the `topic` tag but follow standard naming conventions.

5. If both GitHub repo and remote endpoint found, ask:
   ```
   [ 1 ] Use remote endpoint
   [ 2 ] Use GitHub repository
   [ 3 ] Help me decide
   ```

#### Known Official Vendor Sources

**Universal — Always check (every server, every research run):**

| Source | Org / URL |
|--------|-----------|
| MCP Reference | `modelcontextprotocol/servers` |
| GitHub API Search | `api.github.com/search/repositories?q={name}+mcp+server&sort=stars` |

**Vendor-Specific — Only check when server name matches the vendor:**

| Vendor | GitHub Org | Official Repo / Remote URL |
|--------|-----------|---------------------------|
| Salesforce | `advancedcommunities` | `advancedcommunities/salesforce-mcp-server` (or `mcp.salesforce.com`) |
| Google AI (Gemini) | `googleapis` | `googleapis/genai-toolbox` |
| Hugging Face | `mcp` (Organization) | `huggingface.co/mcp` (or `mcp/huggingface/hf-mcp-server`) |
| Zapier | `zapier` | `mcp.zapier.com` (6,000+ app integrations via one remote endpoint) |
| Notion | `makenotion` | `makenotion/notion-mcp-server` (or `mcp.notion.com`) |
| Box | `box` | `box/mcp-server-box-remote` (or `mcp.box.com`) |
| Vercel | `vercel-labs` | `vercel-labs/mcp-for-next.js` (or `mcp.vercel.com`) |
| Microsoft / Playwright | `microsoft` | `microsoft/playwright-mcp` |
| Supabase | `supabase` | `supabase/mcp-server-supabase` |
| Atlassian (Jira/Confluence) | `atlassian` | `atlassian/mcp-atlassian` |
| Cloudflare | `cloudflare` | `cloudflare/mcp-server-cloudflare` |
| GitHub | `github` | `github/github-mcp-server` |
| AWS | `awslabs` | `awslabs/mcp` (monorepo: `src/{service}-mcp-server/`) |
| Shopify | `Shopify` | `Shopify/shopify-mcp` |
| Sentry | `getsentry` | `getsentry/sentry-mcp` |
| Linear | `linear` | `linear/linear-mcp` |
| Stripe | `stripe` | `stripe/agent-toolkit` |
| Datadog | `DataDog` | `DataDog/datadog-mcp-server` |
| JetBrains | `JetBrains` | `JetBrains/mcp-jetbrains` |
| Brave | `brave` | Referenced in `modelcontextprotocol/servers` |

#### If Server Not Found
```
Could not find an MCP server matching "[server name]".

Searched:
- STEP 0: Known Official Vendor Sources table (vendor name match)
- STEP 1: MCP reference repo (github.com/modelcontextprotocol/servers)
- STEP 2: GitHub topic search (topic:mcp-server+[name], topic:mcp-server+[vendor]) + text fallback (q=[name]+mcp+server)
- STEP 3: Monorepo deep dive via Git Trees API (subdirectory discovery)
- STEP 4: Keyword pattern search ([vendor]-mcp-server, mcp-[vendor], [vendor]-mcp)
- Vendor docs: [name].com, api.[name].com

[ 1 ] Try a different name or URL
[ 2 ] Provide GitHub URL directly
[ 3 ] Cancel research
```
Do NOT fabricate a server or guess URLs. Always confirm with user before proceeding.

---

### Step 5 — Attribute Filling (Top-to-Bottom, One Pass per CSV Section)
**⛔ LOCKED — Do not modify this step or any sub-step (5.1–5.13) without explicit user permission.**

**HARD PREREQUISITE:** Step 0.5 (Parallel Search) MUST be complete before entering Step 5.
All 5 threads must have returned results or confirmed-empty. If any thread is incomplete → go back and finish it.

**Execution order mirrors CSV output order — fill sections 5.1 through 5.13 in sequence.**

**Transport ↔ Deployment Auto-Rules:**
- STDIO=Yes → Auto-set: (1) Local=Yes
- HTTP/SSE OR StreamableHttp=Yes → Auto-set: (3) Remote=Yes

**ZERO-ASSUMPTION RULE:** Every Yes/No must have source + exact quote or file path. Every No must have evidence of absence. If evidence is missing → mark `No` and flag to user.

---

**MCP INFO:**

### Step 5.1 — MCP Info

| # | Attribute | Source | Rule |
|---|-----------|--------|------|
| 1 | Name | GitHub repo name or vendor docs | Exact name only |
| 2 | Description | README first paragraph | 3–4 sentences as a single continuous line (NO embedded newlines), MUST start with "[Server Name] MCP Server…", functional purpose and capabilities ONLY — NEVER include transport, TLS, auth, hosting, or endpoint details (those have dedicated rows) (L8) |
| 3 | Category | mcp.md or vendor classification | Choose: File and Document Management / Developer and Coding Tools / Data and Information Retrieval / Productivity and Communication |
| 4 | GitHub Repository | GitHub URL | Full URL or N/A |
| 5 | Endpoint URL | README or probe result | Full URL or N/A |

**L8 — Description Format Rule:**
- Description is written as a single continuous line — 3 to 4 sentences, NO embedded newlines
- MUST start with "[Server Name] MCP Server…"
- **NEVER include in Description:** transport protocol, TLS version, authentication method, hosting provider, endpoint URL, or deployment details — these have dedicated attribute rows and MUST NOT be duplicated in Description
- Description = functional purpose + capabilities ONLY (what the server does, what tools it exposes, what platform it integrates with)

---

**DISTRIBUTION TYPE:**

### Step 5.2 — Distribution Type

**MUTUALLY EXCLUSIVE — mark exactly ONE as Yes:**

| # | Attribute | Mark Yes if… |
|---|-----------|-------------|
| 1 | Official | Published or actively maintained by the service's own vendor/org |
| 2 | Community | Published by a third-party contributor, unverified publisher, or independent author |

**Confirming Official status — check for multiple of these signals:**
```
□ Repo is under the vendor's own GitHub org
  (e.g., github org = "getsentry" for Sentry MCP, "DataDog" for Datadog MCP)
□ Manifest author/email matches vendor identity
  (e.g., pyproject.toml author = "Datadog, Inc. <packages@datadoghq.com>")
□ Package name uses vendor namespace
  (e.g., awslabs.bedrock-kb-retrieval-mcp-server, @stripe/agent-toolkit)
□ Source file copyright headers name the vendor
□ Server is part of a vendor-owned monorepo alongside other confirmed official servers
□ Documentation is hosted under the vendor's own domain
  (e.g., cloudflare.github.io/mcp-server-cloudflare/)
```
More signals present = higher confidence. A single signal (e.g., org name alone) may be sufficient for well-known vendors; less-known publishers need multiple signals.

**Monorepo trust note:** When a server belongs to a confirmed official monorepo (e.g., `awslabs/mcp`), the trust signals (stars, contributor count, commit activity) apply across the whole repo — not just the individual server subdirectory. Do not mark Community simply because the subdirectory folder has no dedicated stargazers.

---

**MCP PROTOCOL VERSION:**

### Step 5.3 — MCP Protocol Version

**Fill from Step 1 result. EXACTLY ONE row = Yes. ALL others = No. NO blanks.**

| # | Attribute | Rule |
|---|-----------|------|
| 1 | 2025-11-25 | Yes only if Step 1 mapped here |
| 2 | 2025-06-18 | Yes only if Step 1 mapped here |
| 3 | 2025-03-26 | Yes only if Step 1 mapped here (FastMCP 0.3.x only) |
| 4 | 2024-11-05 | Yes only if Step 1 mapped here |

🔒 Never leave a version row blank. Status column = `Yes` or `No` only — no extra text or notes.

#### 🔒 L1 — Framework Priority Rule

Framework version determines protocol — NOT base SDK. Check in this order:
1. FastMCP present? → Use FastMCP mapping
2. Framework present? → Use framework mapping
3. Otherwise → Use base SDK mapping

**Example:** FastMCP 0.4.1 + mcp 1.3.0 → use 2025-06-18 (FastMCP wins), not mcp mapping.

**Verification steps (MANDATORY before marking):**
```
□ 1: Extract exact version from dependency file (pyproject.toml / package.json / go.mod)
□ 2: Cross-reference against Step 1 mapping table — DO NOT guess
□ 3: If version is between ranges → use LOWER protocol (e.g., sdk 1.4.1 < 1.12 → 2025-06-18)
□ 4: Document exact version + mapping source (file + line number)
□ 5: Confirm evidence for version is logged in Evidence Ledger (file + line number)
```

**Common Mistakes:**
- ❌ WRONG: "Latest protocol must be 2025-11-25" (assumes without checking)
- ✅ RIGHT: "SDK v1.4.1 maps to 2025-06-18" (checked version table)
- ❌ WRONG: Go SDK found but no mapping applied
- ✅ RIGHT: "Go SDK v1.4.1 → 2025-06-18 per Go SDK mapping in Step 1"

---

**PRICING:**

### Step 5.4 — Pricing

**MUTUALLY EXCLUSIVE — mark exactly ONE as Yes. Status column = `Yes` or `No` only.**

| # | Attribute | Mark Yes if… |
|---|-----------|-------------|
| 1 | Free | Server code is open-source (check LICENSE file) |
| 2 | Paid | Server itself requires a license/subscription to use |

🔒 API key requirements and downstream service costs do NOT affect Pricing — server license only.

#### 🔒 L4 — Pricing vs Service Rule

Pricing reflects the **MCP server itself**, not the service it accesses or authentication required.

| Scenario | Free | Paid |
|----------|------|------|
| Open-source server (MIT), paid downstream service | Yes | No |
| Open-source server, free downstream service | Yes | No |
| Open-source server, API key required | Yes | No |
| Server requires paid license to use | No | Yes |
| Closed-source proprietary server | No | Yes |

**NEVER confuse with:**
- API key requirement → that's authentication, not pricing
- Downstream service costs (GPU, API calls, subscriptions) → separate concern
- Service fees paid by end users → not the server cost

**Examples:**
- ✅ Runway MCP: MIT License → Free = Yes (even though Runway AI service is paid)
- ✅ Telnyx MCP: Open-source → Free = Yes (even though Telnyx service charges per SMS)
- ❌ Proprietary MCP: Requires license → Paid = Yes

---

**HOSTING PROVIDER:**

### Step 5.5 — Hosting Provider

**NOT mutually exclusive — mark ALL that apply:**

```
HOSTING PROVIDER DETECTION CHECKLIST:
□ 1: Is it offered by the official vendor as SaaS? → SaaS Vendor = Yes
□ 2: Is it hosted by a third-party SaaS provider? (e.g., Vercel, Heroku, AWS) → 3rd Party SaaS = Yes
□ 3: Is the source code on GitHub? → GitHub = Yes
□ 4: Is the source code on GitLab? → GitLab = Yes
□ 5: Is the source code on Bitbucket? → Bitbucket = Yes
□ 6: Is the source code on SourceHut/Gitea/Gogs? → SourceHut/Gitea/Gogs = Yes
```

**Detection Patterns:**
- **SaaS Vendor:** Vendor's own domain (https://api.vendor.com) OR vendor docs state "managed service"
- **3rd Party SaaS:** Deployed URL on Vercel, Netlify, Heroku, AWS, Azure, etc. (visible in README/docs)
- **GitHub:** Repository URL path contains `github.com`
- **GitLab:** Repository URL path contains `gitlab.com`
- **Bitbucket:** Repository URL path contains `bitbucket.org`
- **SourceHut/Gitea/Gogs:** Repository URL contains `sr.ht`, custom Gitea/Gogs instance

---

**AUTHENTICATION:**

### Step 5.6 — Authentication

**NOT mutually exclusive — mark ALL that apply. Authentication is often MISSED.**

```
AUTHENTICATION VERIFICATION CHECKLIST:
□ 1: Search server.json for "TOKEN", "API_KEY", "SECRET", "BEARER"
□ 2: Search README for "authentication", "auth", "credentials", "token"
□ 3: Search .env.example for credential fields
□ 4: Search vendor docs for API access / token setup page
□ 5: If ANY credential found → mark ALL applicable auth types as Yes
□ 5a: Apply Detection Patterns — API Token: `"api key"`, `"api_token"`, `"api_key"` in combined text; PAT: `"personal access"`, `"pat"`, `"personal token"` in doc_methods or `"personal access token"`, `"personal_access_token"` in combined text — mark every matched type Yes
□ 6: Confirm all auth evidence is logged in Evidence Ledger (source file + exact quote)
```

| # | Attribute | Mark Yes if… |
|---|-----------|-------------|
| 1 | OAuth 2.1 - Authorization Code Flow | OAuth flow with user redirect found — **No for STDIO transport** |
| 2 | OAuth 2.1 - Client Credentials Flow | Machine-to-machine OAuth found — **No for STDIO transport** |
| 3 | Bearer Token | Token passed in Authorization header |
| 4 | Personal Access Token | User-generated PAT found |
| 5 | API Token | App-level API key found |

**STDIO exception:** OAuth 2.1 (Auth Code and Client Creds) = No always for STDIO transport. OAuth 2.1 requires a network-accessible endpoint; local STDIO connections do not use OAuth 2.1.

**Common Mistake Pattern (LEARNED):**
- ❌ WRONG: "No auth in README, so Authentication = No" (incomplete search)
- ✅ RIGHT: "BUILDKITE_API_TOKEN found in server.json → (3) Bearer Token, (4) PAT, (5) API Token all = Yes"

#### 🔒 Authentication Detection Rules

**Core Principle:** Bearer Token is a DELIVERY METHOD, not a credential type. Multiple auth fields can be Yes simultaneously.

**API Token — Detection Signal Patterns:**
```
API Token = Yes if ANY of the following match in combined source text:
  "api key" in combined_text or
  "api_token" in combined_text or
  "api_key" in combined_text
```

**Personal Access Token (PAT) — Detection Signal Patterns:**
```
PAT = Yes if ANY of the following match:
  "personal access" in doc_methods_lower or
  "pat" in doc_methods_lower or
  "personal token" in doc_methods_lower or
  "personal access token" in combined_text or
  "personal_access_token" in combined_text
```

```
Visual Relationship:
Bearer Token (delivery method — how it's sent)
    ├── Personal Access Token  (user-specific, acts on behalf of user)
    ├── OAuth Access Token     (from OAuth 2.1 flow)
    └── JWT Token              (signed token)

API Key (credential type — app identification, NOT user-specific)
    └── Can be delivered via Bearer header OR X-API-Key header OR query param
```

| Auth Type | Mark Yes When | Co-occurrence |
|-----------|--------------|---------------|
| Bearer Token | `Authorization: Bearer` found anywhere in source/docs | Always Yes when PAT, OAuth token, or API key uses Bearer delivery |
| Personal Access Token | "personal access token", "PAT" explicitly in docs/README/code | → Also set Bearer Token = Yes |
| API Token | `"api key"`, `"api_token"`, `"api_key"` in source or docs | → Also set Bearer Token = Yes if sent via Bearer |
| OAuth 2.1 - Authorization Code Flow | OAuth 2.1 auth code flow documented; user redirected to auth page | → Also set Bearer Token = Yes |
| OAuth 2.1 - Client Credentials Flow | OAuth 2.1 client credentials documented; machine-to-machine auth | → Also set Bearer Token = Yes |

**Key Rules:**
1. Bearer Token ≠ credential type — it's the transport mechanism. Never mark ONLY Bearer Token without also identifying the underlying credential type (PAT / API Token / OAuth)
2. PAT + Bearer = both Yes — PAT is delivered via Bearer header
3. API Token + Bearer = both Yes — if API key sent via `Authorization: Bearer`
4. API Token ≠ PAT — API keys are app-level static keys; PATs are user-level tokens

#### Detection Patterns (from source code)
```
Bearer Token = Yes if:
  "Authorization: Bearer" in source/docs

API Token = Yes if:
  "api key" in combined_text or
  "api_token" in combined_text or
  "api_key" in combined_text

Personal Access Token (PAT) = Yes if:
  "personal access" in doc_methods_lower or
  "pat" in doc_methods_lower or
  "personal token" in doc_methods_lower or
  "personal access token" in combined_text or
  "personal_access_token" in combined_text
```

**This rule applies during both single-server (Step 5.6) and multi-server (Layer 1 agents) research.**

---

**AUTHENTICATION KEY PROMPT:**

### Step 5.6.A — Authentication Key Prompt Workflow

**Trigger:** Execute whenever authentication is detected (any auth attribute = Yes from Step 5.6 checklist above, OR auth required detected during Step 2 endpoint probe / Step 3 local setup config extraction).

**IMMEDIATE — no exceptions:** As soon as ANY auth signal is found (API key, Bearer token, PAT, secret key, API token, OAuth access token, JWT token) during Steps 0.5 Thread 2/3, Step 5.6, or Step 5.6.B — PAUSE research and show the 3-option prompt. Do NOT continue to the next research step without user response.

**🔴 HARD BLOCK:** When a detection pattern match triggers (API Token pattern, PAT pattern, Bearer Token signal, or any env/args/headers signal from Step 5.6.B), the skill MUST present the 3-option prompt and MUST NOT proceed with any further research steps until the user selects an option. This applies in both single-server and multi-server (parallel) research modes.

**When auth is required, present this 3-option prompt to the user:**

```
Authentication required for this MCP server.
Detected: [list ALL detected types — e.g. API Token / Bearer Token / PAT / OAuth Token / JWT]
         (List every type that matched a detection pattern — do NOT omit any)

How do you want to proceed?

[ 1 ] I have the API key / token — proceed with authentication
[ 2 ] I don't have one — show me how to create it
[ 3 ] Proceed without authentication (manual verification only)
```

**Multiple auth types detected — marking rules:**
- If BOTH API Token and PAT (and/or other types) are detected → list ALL of them in the Detected line above.
- If user selects **Option 1** and provides/confirms credentials: mark only the auth types the user has confirmed as `Yes`. Mark unconfirmed types based on source evidence only.
- If user selects **Option 3** (proceed without): apply the Detection Patterns fallback — mark ALL types that matched a signal pattern as `Yes` (evidence = detection pattern match). Do NOT mark only a subset just because the user has no key.
- **Never invent auth types** — only list and mark what detection patterns confirmed from source.

#### Option 1 — User Has Key/Token
- **NEVER ask for the actual key, secret, or token value in chat** (Security Mandate — absolute rule, no exceptions)
- Show the config file path and the correct `<PLACEHOLDER>` field name
- Direct user to open and edit the file directly to paste their credential
- After user confirms the edit is done → proceed with connection test / research

#### Option 2 — User Needs to Create Key
- Retrieve the vendor authentication / API key creation URL from README or vendor source docs
- Provide step-by-step creation instructions:
  1. Direct URL to the vendor's token / API key creation page
  2. Required permissions or scopes needed for this MCP server to function
  3. Config field name and `<PLACEHOLDER>` format to use when storing the key
- After user creates the key → redirect to Option 1 flow above

#### Option 3 — Proceed Without Authentication
- Show this warning before continuing:
  ```
  ⚠️  API auth will fail — expect authentication errors (401 / 403).
      Proceeding with manual verification only. Connection tests may not succeed.
  ```
- Continue research and attribute filling with manual verification; note auth status in Evidence Ledger
- **Fallback Rule:** Apply the implicit auth detection rule — if ANY field matches a signal pattern (Detection Patterns above + Step 5.6.B signal patterns) → mark ALL applicable authentication attributes as `Yes`. Evidence = detection pattern match from source. Do NOT leave auth attributes as `No` just because the user has no key.

---

**IMPLICIT AUTH DETECTION:**

### Step 5.6.B — Implicit Auth Detection (env / args / headers)

**Applies to ALL 3 input types:** Remote Endpoint URL · GitHub Repository URL · Server Name

Scan ALL available sources for auth signals — README, JSON config files, env vars, args, and response headers. **README is a PRIMARY scan target:** auth requirements are frequently documented in README even for STDIO/GitHub repo servers. Scan across all three field locations:

**Scan targets per input type:**

| Input Type | Where to Scan |
|------------|--------------|
| Remote Endpoint URL | JSON config `headers` block (e.g. `mcp.json`, `settings.json`, `server.json`) |
| GitHub Repository URL | README (Detection Patterns — API Token, PAT, Bearer); `env` vars in `server.json` / `.env.example` / `config.json`; `args` fields in `mcp.json` |
| Server Name | All of the above — resolve to GitHub URL first, then apply same scans |

#### Signal Patterns by Field Type

```
headers (remote endpoint — JSON config):
  "headers": {
    "Authorization": "Bearer <...>"    → Bearer Token = Yes
    "Authorization": "token <...>"     → PAT = Yes + Bearer Token = Yes
    "X-API-Key": "<...>"               → API Token = Yes
    "X-Auth-Token": "<...>"            → API Token = Yes
    "Authorization": "Basic <...>"     → API Token = Yes
  }

env (STDIO / local / GitHub repo):
  Field name contains: TOKEN, API_KEY, SECRET, BEARER, AUTH, CREDENTIAL, ACCESS_KEY, PASSWORD
  Example: "GITHUB_TOKEN", "BUILDKITE_API_TOKEN", "OPENAI_API_KEY"

args (STDIO / local / GitHub repo):
  Arg flag contains: --token, --api-key, --secret, --auth, --bearer, --pat, --access-token
  Example: ["--token", "${GITHUB_TOKEN}"], ["--api-key", "<KEY>"]
```

**Rule:** If ANY field matches a signal pattern above → mark ALL applicable authentication attributes as Yes.
Do NOT require explicit README mention when official source JSON config provides evidence.
Record evidence in Evidence Ledger: `server.json headers.Authorization: "Bearer"` or `env field: "GITHUB_TOKEN"`.

This rule applies during both single-server (Step 5.6) and multi-server (Layer 1 agents) research.

---

**DATA PROTECTION:**

### Step 5.7 — Data Protection

**Determine transport (Step 5.8) FIRST, then apply TLS rule.**

| # | Attribute | Rule |
|---|-----------|------|
| 1 | TLS 1.3 | Yes only if remote endpoint confirmed TLS 1.3 via probe **OR** official vendor documentation explicitly states TLS 1.3 is supported (cite doc URL) |
| 2 | TLS 1.2 | Yes only if remote endpoint confirmed TLS 1.2 via probe **OR** official vendor documentation explicitly states TLS 1.2 is supported (cite doc URL) |
| 3 | Lower versions or no encryption | No if Container deployment = Yes (containers use proper TLS termination); No if STDIO (no network layer); Yes only if probe confirms no TLS on a remote endpoint |

**Org-scoped HTTPS endpoints (e.g. `https://<org>.service.com/api/mcp`):**
A URL pattern containing a placeholder like `<org>`, `<orgname>`, `<tenant>`, or `<subdomain>` is a **remote managed endpoint** — it is vendor-hosted, just with an org-specific subdomain. For such endpoints:
- Deployment Remote = Yes
- TLS: check vendor's official TLS documentation (probe is not possible without a real org credential). Official docs stating "TLS 1.2/1.3 required" count as valid evidence.
- Transport: determine StreamableHttp vs HTTP/SSE from the protocol version (2025-03-26+ → StreamableHttp; pre-2025-03-26 → HTTP/SSE). Confirm against README/docs if available.
- Do NOT set Endpoint URL = N/A — use the placeholder pattern as the documented value (e.g. `https://<orgname>.crm.dynamics.com/api/mcp`).

**Probe tools:**

Method 1 — curl (Linux/macOS):
```bash
curl -s -v https://<server-url> 2>&1 | grep "SSL connection using"
```
Method 1 — curl (Windows):
```bash
curl -s -v https://<server-url> 2>&1 | findstr "SSL connection using"
```
Method 2 — OpenSSL, check TLS 1.3:
```bash
openssl s_client -connect <hostname>:443 -tls1_3
```
Method 2 — OpenSSL, check TLS 1.2:
```bash
curl -v --tlsv1.2 --tls-max 1.2 https://<hostname>
```
**Rule:** Use Method 1 first. If it fails or returns no result → run Method 2. If one TLS version check fails → run the other version check before marking as unverifiable.
- For STDIO-only servers: mark all TLS rows as No, note "local transport — no network layer"
- For mixed-deployment servers (STDIO + Container, e.g. STDIO default + Docker/AgentCore): STDIO → TLS 1.3 = No, TLS 1.2 = No; Container = Yes → "Lower versions or no encryption" = No. All three TLS rows = No. Do NOT mark "Lower versions or no encryption" = Yes just because STDIO is the primary transport — the Container rule overrides.

#### 🔒 L3 — TLS vs Bearer Independence Rule

Bearer Token = authentication delivery method. TLS = transport encryption. These are completely independent layers.

| Transport / Deployment | TLS Apply? | Reason |
|------------------------|-----------|--------|
| STDIO (local) | Always No | stdin/stdout on local machine, no network |
| HTTP/SSE (remote) | Yes | Network transmission requires TLS |
| StreamableHttp (remote) | Yes | Network transmission requires TLS |
| Container deployment = Yes | Lower/None = No always | Containers are deployed with proper TLS termination (TLS 1.2 or 1.3); lower/no encryption is not applicable |
| Mixed: STDIO + Container | TLS 1.3 = No, TLS 1.2 = No, Lower/None = No | STDIO → rows 1 & 2 = No; Container = Yes → row 3 = No; all three TLS rows = No is valid |

**Rules:**
- STDIO transport → ALL TLS fields = No (no exceptions)
- Container deployment = Yes → "Lower versions or no encryption" = No (no exceptions)
- **Mixed-deployment (STDIO + Container):** Fill Step 5.10 (Deployment) BEFORE Step 5.7 (TLS). Container = Yes overrides STDIO-based logic for the Lower/None field. All three TLS rows = No is a valid and correct state.
- Bearer Token present does NOT imply TLS present
- TLS decision is based ONLY on transport protocol / deployment approach, never on authentication
- Check transport AND deployment FIRST, then determine TLS independently

**Examples:**
- STDIO server with Bearer Token → TLS = No (auth is local process, no network)
- HTTP endpoint with no auth → TLS = Yes (network needs encryption regardless)
- HTTP endpoint with Bearer Token → TLS = Yes AND Bearer = Yes (both, independently)
- Container deployment → "Lower versions or no encryption" = No (proper TLS termination assumed)
- STDIO (default) + Container (Dockerfile/Marketplace) → TLS 1.3 = No, TLS 1.2 = No, Lower/None = No (all three No — do NOT set Lower/None = Yes because STDIO is primary)

---

**TRANSPORT PROTOCOL:**

### Step 5.8 — Transport Protocol

**NOT mutually exclusive — mark ALL that apply:**

```
TRANSPORT PROTOCOL VERIFICATION CHECKLIST:
□ 1: Does README mention stdin/stdout or STDIO? → STDIO = Yes
□ 2: Is there an HTTP endpoint with GET (Content-Type: text/event-stream)? → HTTP/SSE = Yes
     ⚠️  NOTE: HTTP/SSE is deprecated as of the 2025-06-18 spec revision. If found, mark Yes but
     note it as legacy. Newer servers use StreamableHTTP instead.
□ 3: Is there an HTTP endpoint with POST (JSON-RPC streaming)? → StreamableHttp = Yes
□ 4: Is the server built with FastAPI or similar async web framework? → FastAPI = Yes
□   Check for @app.post(), @app.get(), async def patterns
```

---

**TOOLS OPERATIONS:**

### Step 5.9 — Tools Operations

**MUTUALLY EXCLUSIVE — mark HIGHEST level only:**

```
TOOLS OPERATIONS VERIFICATION CHECKLIST:
□ 1: Parse ALL tool names from documentation
□ 2: Look for patterns: create_*, update_*, delete_*, cancel_*, remove_*, destroy_*
□ 3: If ONLY read operations found → mark (1) Read-only = Yes
□ 4: If read + create/update found → mark (2) Read+Update = Yes
□ 5: If read + create/update + delete/cancel found → mark (3) Read+Update+Delete = Yes
□ 6: Mark ONLY the HIGHEST level — all others = No
□ 7: Confirm only ONE level is marked Yes and evidence is in Evidence Ledger
```

| # | Attribute | Pattern |
|---|-----------|---------|
| 1 | Read-only operations | get_*, list_*, search_*, fetch_* only |
| 2 | Read-only and/or update operations | create_*, update_* present, no delete |
| 3 | Read-only update and/or delete operations | delete_*, cancel_*, remove_*, destroy_* present |

**Common Mistake Patterns (LEARNED):**
- ❌ WRONG: Mark both (2) and (3) as Yes (violates exclusivity)
- ✅ RIGHT: Mark ONLY (3) = Yes when delete patterns found
- ❌ WRONG: "Tool names: create_build, cancel_build" → mark (2) (missed delete pattern)
- ✅ RIGHT: "cancel_* = delete operation" → mark (3)

#### 🔒 L5 — Tools Operations Decision Tree
```
Does server have delete/terminate operations? (delete_*, cancel_*, remove_*, destroy_*)
    → Yes: Mark (3) Read-only update and/or delete = Yes. Mark (1) and (2) = No.

Does server have only create/update (no delete)?
    → Yes: Mark (2) Read-only and/or update = Yes. Mark (1) and (3) = No.

Does server have ONLY read operations?
    → Yes: Mark (1) Read-only = Yes. Mark (2) and (3) = No.
```
**Example:** list (read) + create (write) + terminate (delete) → Highest = Delete → Mark (3) = Yes, others = No.

---

**DEPLOYMENT APPROACH:**

### Step 5.10 — Deployment Approach

**NOT mutually exclusive — mark ALL that apply:**

```
DEPLOYMENT APPROACH VERIFICATION CHECKLIST:
□ 1: Check for STDIO support (stdin/stdout) → Local = Yes
□ 2: Search for Dockerfile, docker.json, OCI registry refs (ghcr.io) → Container = Yes
□ 3: Check for vendor endpoint or remote URL in README → Remote = Yes
□   Also check server.json for "docker", "container", "run"
□   Confirm all deployment evidence is in Evidence Ledger (file + line)
```

| # | Attribute | Mark Yes if… |
|---|-----------|-------------|
| 1 | Local | STDIO transport or published package (pip/npx) found |
| 2 | Container | Dockerfile, docker-compose, or OCI image reference found |
| 3 | Remote | Vendor endpoint or hosted URL available — including org-scoped URL patterns (e.g. `https://<org>.service.com/api/mcp`) which are vendor-managed remote endpoints |

**Remote = No when:**
- Server only runs via localhost (e.g. `http://127.0.0.1:<port>`) — localhost addresses are runtime artefacts, NOT remote endpoints
- README roadmap lists remote deployment as a future/planned item but it is not yet available
- Docker/Container deployment is local only (no hosted cloud endpoint documented)

**TLS implication — Container = Yes:**
When Container = Yes, set "Lower versions or no encryption" = **No** in Step 5.7 (no probing needed — containers use proper TLS termination at the network edge).

**Common Mistake Pattern (LEARNED):**
- ❌ WRONG: "Go binary exists, so (2) Container = No" (missed Docker config)
- ✅ RIGHT: "Found Dockerfile + STDIO" → (1) Local = Yes AND (2) Container = Yes
- ❌ WRONG: "Container found, so Lower versions or no encryption = Yes" (containers use proper TLS)
- ✅ RIGHT: "Container = Yes" → "Lower versions or no encryption" = No (always)

---

**COMPLIANCE & CERTIFICATIONS:**

### Step 5.11 — Compliance & Certifications

Fill from Thread 5 (compliance docs search) of Step 0.5:

| # | Attribute | Mark Yes if… |
|---|-----------|-------------|
| 1 | HIPAA | Explicit HIPAA compliance statement in docs |
| 2 | GDPR | Explicit GDPR compliance statement in docs |
| 3 | SOC 2 | SOC 2 Type I or II certification stated |
| 4 | FedRAMP | FedRAMP authorization stated |

---

**CAPABILITIES:**

### Step 5.12 — Capabilities

**Extract from source code — README alone is NOT sufficient (L9).**

```
CAPABILITIES VERIFICATION CHECKLIST:
□ 1: Read the server's entry point source file first
   → Python: main.py / server.py
   → TypeScript: index.ts / server.ts
   → Java: Program.java / Application.java (look for McpSchema.ServerCapabilities.builder())
   → Go: main.go / server.go
   → NEVER mark Resources/Prompts/Sampling from README alone

□ 2: Look for tool registrations → (1) Tools = Yes if found
   - Python:     @mcp.tool() decorators or server.add_tool()
   - TypeScript: server.tool() calls
   - Java:       ITool implementations + registerTools(), .tools(true) in capabilities builder
   - Go:         mcp.NewTool() + server.AddTool()

□ 3: Look for resource registrations → (2) Resources = Yes if found
   - Python:     @mcp.resource() decorators
   - TypeScript: server.resource() calls
   - Java:       IResource implementations + registerResources(), .resources(...) in capabilities builder
   - Go:         server.AddResource()

□ 4: Look for prompt registrations → (3) Prompts = Yes if found
   - Python:     @mcp.prompt() decorators
   - TypeScript: server.prompt() calls
   - Java:       registerPrompts() + .prompts() in capabilities builder

□ 5: Look for sampling handlers → (4) Sampling = Yes if found

□   For Java — cross-check McpSchema.ServerCapabilities.builder() call
    → Only capabilities explicitly in builder are enabled. Absence = No.
```

#### 🔒 L9 — Capabilities Source Verification Rule

README does NOT reliably document all capabilities. Always read the server's entry point source file.

**Why this matters:** SAP BusinessObjects BI MCP by CData — README listed 3 tools only. Source had a `resources/` directory with `TableMetadataResource.java` — a fully registered MCP resource — never mentioned in README. `Capabilities,Resources` was initially marked `No` (wrong). Corrected only after reading `Program.java`.

```
🔒 NEVER mark Resources/Prompts/Sampling = No based on README alone
🔒 ALWAYS read entry point source to confirm absence
🔒 For Java: check both the capabilities builder AND the register methods
🔒 Check subdirectories: tools/, resources/, prompts/ may exist even if README omits them
```

---

**CAPABILITY DETAILS & NON-READ-ONLY TOOLS:**

### Step 5.13 — Capability Details & Non-Read-Only Tools

**ALL FIVE rows ALWAYS present. Use `"None"` if capability is absent. Attribute column = `detailed_info` for all five.**

| # | CSV Row | Content |
|---|---------|---------|
| 1 | Capabilities - Tools,detailed_info | Tool list grouped by standard taxonomy (L6) |
| 2 | Capabilities - Resources,detailed_info | Resource list — bullet points only, no category headers |
| 3 | Capabilities - Prompts,detailed_info | Prompt list or "None" |
| 4 | Capabilities - Sampling,detailed_info | Sampling details or "None" |
| 5 | Non-Read-Only Tools,detailed_info | Write/delete ops grouped by standard taxonomy (L6) or "None" |

**🔒 L7 — Five Mandatory Rows Rule:**
- ALL five rows are MANDATORY in every CSV report — NEVER omit any
- If content not found → row value = `"None"` (attribute is still `detailed_info`)
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

**Format for (1) Capabilities - Tools (CSV cell) — standard taxonomy only (L6):**
```
Category Name
  • tool_name – Description

Another Category
  • tool_name – Description
```

**Format for (2) Capabilities - Resources (CSV cell):**
```
  • resource_uri – Description (resource type: text/image/document)
```
🔒 **NO category header** — bullet points only. Never add a title line above the bullets.

**Format for (3) Capabilities - Prompts (CSV cell):**
```
  • prompt_name – Description (inputs: param1, param2)
```

**Format for (4) Capabilities - Sampling (CSV cell):**
```
Model Capabilities
  • model_name – Sampling support (temperature, top_p, max_tokens)
  • completion_method – Inference optimization details
```

**Format for (5) Non-Read-Only Tools (CSV cell) — standard taxonomy only (L6):**
```
Category Name
  • tool_name – Description of the write/delete operation

Another Category
  • tool_name – Description
```

#### 🔒 L6 — Standard Taxonomy Rule (applies to (1) and (5) above)

NEVER invent category titles. ALWAYS use these fixed titles only:

**Capabilities - Tools — 6 fixed titles:**
| Title | Use for |
|-------|---------|
| `Search & Query Utilities` | get_, list_, search_, find_, query_, fetch_ operations |
| `Project Management` | project-scoped CRUD operations |
| `Issue Management` | issue/ticket/log-related operations |
| `Team & Workspace Metadata` | user, org, team, workspace info retrieval |
| `Admin & Miscellaneous` | admin ops, token info, access control |
| `Other` | tools that don't fit any category above |

**Non-Read-Only Tools — 4 fixed titles:**
| Title | Use for |
|-------|---------|
| `Import/Export Tools` | upload, download, transfer, sync operations |
| `Content Management` | create/update/delete stored content or files |
| `Configuration Tools` | install, setup, configure, deploy operations |
| `Other Write Operations` | write/delete ops that don't fit above categories |

**❌ NEVER use invented titles:** "Device Management", "Asset Management", "Media Generation", "Task Management", "GPU Management", "Repository Management", or ANY domain-specific custom title.

**Classification rules:**
- Classify by WHAT THE TOOL DOES (get_, list_, create_) — NOT what it manages (devices, assets, tables)
- Each tool appears in ONE category only
- If a tool fits multiple → use the more specific one

#### 🔒 L10 — No Placeholders Rule
```
🔒 NEVER write {servername}_, {prefix}_, {name}_ in any CSV cell
🔒 Check Capabilities - Tools, Capabilities - Resources, Non-Read-Only Tools cells
🔒 If no default prefix documented → use base name only
🔒 Placeholder format is invisible/confusing in reports — always resolve to actual names
```

**❌ WRONG (placeholder format):**
```
  • {servername}_get_tables – List tables
  • {prefix}_run_query – Execute query
```
**✅ CORRECT — use base name only:**
```
  • get_tables – List all available tables
  • run_query – Execute a SQL SELECT query
```

---

### Step 6 — Save Report
**⛔ LOCKED — Do not modify this step without explicit user permission.**

**Location:** User-configured path
**Format:** CSV (3 columns: Category, Attribute, Status)
**Filename:** `<servername>.csv`
**CSV Rules:** Use `csv.QUOTE_ALL` for proper multiline handling

#### Step 6.1 — Save Cost File (MANDATORY — always runs after CSV is saved)

After saving the CSV, immediately generate and save `<servername>-cost.txt` at the same path.

**Required format for `<servername>-cost.txt`:**
```
MCP Server     : <GitHub repository URL or Remote MCP endpoint URL or Server Name>
Date           : <YYYY-MM-DD HH:MM:SS>
Model          : <model-id>

Token Usage
  Input        : <n>
  Output       : <n>
  Cache Read   : <n>

Cost
  Input        : $<amount>  (<n> x $3.00/1M)
  Output       : $<amount>  (<n> x $15.00/1M)
  Cache Read   : $<amount>  (<n> x $0.30/1M)
  Total        : $<amount>
```

**How to populate:**

1. **Token counts** — run `references/cost-script.py` to extract from the session JSONL:
   ```bash
   python3 references/cost-script.py \
     --start "<YYYY-MM-DDThh:mm:ss>" \
     --end   "<YYYY-MM-DDThh:mm:ss>" \
     --server "<servername-kebab-case>"
   ```
   - `--start` = timestamp when Step 0.5 Thread 1 began
   - `--end`   = timestamp when Step 6 (report save) completed
   - Script sums `input_tokens`, `output_tokens`, `cache_read_input_tokens` from the session JSONL within that window

2. **Cost calculation** — apply these rates to the token counts:
   - Input       : tokens × $3.00 / 1,000,000
   - Output      : tokens × $15.00 / 1,000,000
   - Cache Read  : tokens × $0.30 / 1,000,000
   - Total       : sum of all three

3. **Date** — UTC timestamp when the report was saved
4. **Model** — model ID used for this session (e.g. `claude-sonnet-4-6`)


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

## Multi-Server Parallel Research

**Single-server mode** uses Step 0.5's 5-thread parallel search directly (no sub-agents).
**Multi-server mode** dispatches one Layer 1 agent per server, each running the same Step 0.5 dual-source workflow. Gates 1–4, all Learnings, and all attribute rules are identical in both modes — only the orchestration layer differs.

**If 1 server detected** → skip this section, continue with standard workflow from Step 0.
**If 2+ servers detected** → load and follow `references/multi-server.md`.

**🔴 MULTI-SERVER AUTH PRE-SCAN (MANDATORY before dispatching any Layer 1 agents):**

Before starting parallel research for batch/multi-server mode, run a quick pre-scan across ALL servers to detect auth requirements:

```
For each server in the batch:
  1. Quick-scan README / GitHub repo / server.json for auth signal patterns
     (API Token patterns, PAT patterns, Bearer Token, env TOKEN/KEY/SECRET fields)
  2. Classify as:
     - AUTH REQUIRED   → at least one signal pattern matched
     - NO AUTH NEEDED  → no signal patterns found
```

Display this summary to the user BEFORE research starts:

```
Multi-Server Auth Pre-Scan Results:
┌─────────────────────────┬──────────────────────────────────────┐
│ Server                  │ Auth Status                          │
├─────────────────────────┼──────────────────────────────────────┤
│ <server-name>           │ 🔑 AUTH REQUIRED (API Token / PAT)  │
│ <server-name>           │ ✅ No auth needed                    │
│ ...                     │ ...                                  │
└─────────────────────────┴──────────────────────────────────────┘

Servers requiring auth — how do you want to proceed for each?
  [ 1 ] I have the key/token   [ 2 ] Help me create one   [ 3 ] Proceed without auth
```

**Batch execution rules:**
- **Do NOT wait** for API key input before starting research on servers marked "No auth needed" — begin those immediately in parallel.
- For servers marked "AUTH REQUIRED": show the 3-option prompt PER SERVER and collect responses.
- If user selects Option 3 for an auth-required server → apply the Detection Patterns fallback (mark all detected auth attributes Yes, proceed with manual verification).
- After collecting all auth decisions → dispatch all Layer 1 agents in parallel (auth-handled and no-auth servers together).

---

## Integration Rules

**Session Start (MANDATORY — never skip):**
- All learnings (L1–L10) are embedded in Step 5.1–5.13 — follow each section's rules directly
- These are gates — skipping them causes the same mistakes to recur

**During Research (enforce at every attribute):**
- Verify Transport Protocol FIRST, then determine TLS (never reverse) → Step 5.7 (L3)
- Verify Pricing = server license, NOT service cost (ask: "Is the MCP server itself paid?") → Step 5.4 (L4)
- Verify Categories using standard taxonomy only — NEVER invent → Step 5.13 (L6)
- Verify Tools Operations = HIGHEST level only (never mark multiple) → Step 5.9 (L5)
- All five capability rows ALWAYS present with attribute "detailed_info" — use "None" if absent → Step 5.13 (L7)
- Every Yes/No MUST have source evidence — if missing, mark `No` and flag to user

**Workflow Gates:**
- Step 3 decision: Ask user preference (local vs. research)
- If local setup: Execute Steps 3.1-3.6 before research
- If connection fails: Auto-trigger Phase 1 diagnosis
- After error fix: Continue research from interrupted point
- Security check: Every config change requires approval

**Session End (MANDATORY — never skip):**
- Run Gate 3 Learning Walkthrough (all Learnings checked)
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
**⛔ LOCKED — DO NOT add, remove, rename, or reorder any category or row without explicit user permission.**

```
MCP Info
├── Description (2–3 sentences, MUST start with "[Server Name] MCP Server…")
├── Git Repo Version (fetch EXACTLY as shown in source — never reformat)
│   Source priority: 1st → GitHub Releases  2nd → GitHub Tags  3rd → package.json (or pyproject.toml / Cargo.toml / setup.py)
│   Use version EXACTLY as shown in source (NO notes, NO transformation)
│   If no version found after checking all 3 sources → use "No"
│   Monorepo rule: If releases/tags are monorepo-wide (not server-specific) → fall through to package.json; if package.json is also monorepo-wide → use "No"
├── Category (pick ONE based on server functionality: File and Document Management / Developer and Coding Tools / Data and Information Retrieval / Productivity and Communication)
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
**⛔ LOCKED — The 17 categories above and the 3-column format (Category, Attribute, Status) are fixed. No modifications without explicit user permission.**

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
3b. **Connector (Non-Read-Only Tools)** — Use `–` (en-dash, NOT hyphen, NOT colon) between tool name and description
4. **Blank lines** — Always separate category blocks with blank line
5. **Tool names** — Use actual registered function names (not generic labels)
6. **Descriptions** — Concise, action-oriented (start with verb: "Create", "Get", "List", "Send")
7. **Multiline cells** — CSV escaping handles newlines, no manual escaping needed
8. **Attributes** — Final CSV must be Yes/No only. During research, use `UNVERIFIED` for unconfirmed attributes — resolve ALL to Yes/No before generating CSV (per Gate 1)
9. **Description field** — The `MCP Info,Description` cell MUST:
   - **Always start with `[Server Name] MCP Server`** (e.g., "Todoist MCP Server bridges AI assistants with…")
   - Sentence 1: What the server connects/bridges (AI agents ↔ what platform/service)
   - Sentence 2–3: Key features and capabilities (specific tools, workflows, or integrations)
   - Written as a single continuous line — NO embedded newlines (3–4 sentences joined with spaces)
   - Escape any double quotes inside the description as `""`
   - **NEVER include:** transport protocol, TLS version, authentication method, hosting provider, endpoint URL, or deployment details — these belong in dedicated attribute rows, NOT in Description. Including them overwrites the meaning of those Status column values.

   ❌ WRONG (technical attribute details in Description — overwrites Status column values):
   ```
   MCP Info,Description,"… supporting StreamableHttp transport with TLS 1.3 and TLS 1.2 encryption."
   MCP Info,Description,"… deployed on AWS API Gateway with no authentication required."
   ```
   ✅ CORRECT (3–4 sentences, single continuous line, functional purpose and capabilities only):
   ```
   MCP Info,Description,"Todoist MCP Server bridges AI assistants with the Todoist task management platform. It exposes tools for creating, updating, and completing tasks across projects and labels. Supports full task lifecycle management including due dates, priorities, and comments."
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
  • create_assistant – Create a new AI assistant
  • delete_assistant – Delete an existing AI assistant"
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

## Learned Fixes — Embedded in L1–L10

> All error patterns from past research sessions have been extracted and permanently embedded into SKILL.md as Learnings L1–L10 (Steps 5.1–5.13).
> These learnings are the authoritative rules — they run at every session via Gate 3.

---

## Final Report Output

**Display order (MANDATORY):**
1. Research Summary table (all attributes — shown first)
2. Evidence Table (every attribute with its source and evidence quote — shown second)
3. Attributes Information or Detected Information (notable findings — shown third)
4. Success box (shown last, below the summary):

**Evidence Table format (MANDATORY — display in chat window for every report):**
```
EVIDENCE TABLE
────────────────────────────────────────────────────────────────────────────────
Attribute                  | Value | Source                     | Evidence
──────────────────────────────────────────────────────────────────────────────
Protocol Version           | 2025-11-25 | package.json ln 5     | "@modelcontextprotocol/sdk": "^1.27.1"
TLS 1.2                    | Yes   | Microsoft Learn docs       | "Dataverse is using TLS 1.3 & 1.2 cipher suites"
TLS 1.3                    | Yes   | Microsoft Learn docs       | "Dataverse is using TLS 1.3 & 1.2 cipher suites"
Transport / StreamableHttp | Yes   | Protocol 2025-11-25        | StreamableHttp is the primary transport for ≥2025-03-26
Git Repo Version           | 0.2.1 | packages/mcp-server/package.json ln 3 | "version": "0.2.1"
...
────────────────────────────────────────────────────────────────────────────────
```
One row per attribute that has a Yes value (or notable No with evidence). Source = file/URL. Evidence = exact quote or probe result.

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

## Key Learnings from Research & Verification

### Learning 1: Framework Priority

> Core rule: FastMCP > framework > base SDK — always DO NUMERIC COMPARISON, never assume.
> Full mapping tables, verification checklist, and ✅/❌ examples: **Step 1** and **Step 5.3**.

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

> Canonical rules, co-occurrence table, and detection patterns are defined in **Step 5.6**.
> (Authentication Verification Checklist, visual relationship diagram, detection table, Key Rules 1–4, Detection Patterns — API Token + PAT signal patterns, and 3-option prompt workflow with HARD BLOCK enforcement.)
> Step 5.6 is the single source of truth for authentication detection — rules are not duplicated here.

---
