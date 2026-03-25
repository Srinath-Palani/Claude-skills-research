# MCP Server Skill — Workflow Diagram

> Last updated: 2026-03-24 (GitHub pre-sets permanent + carry-forward on path switch; **TLS verification MANDATORY for remote endpoints** — run curl --tlsv1.2 and --tlsv1.3; Free/Paid rule update; **Description must include specific capabilities and features** — avoid generic descriptions; **Yes/No only** — never use N/A, mark No if unverified; **Dual-deployment servers** — mark both SaaS Vendor=Yes AND GitHub=Yes if both options available; also mark Transport and Deployment for both modes; TLS applies to Remote endpoint only)
> Update this file every time SKILL.md is modified.

---

## Full Workflow

```
USER PROVIDES URL / server name
              │
              ▼
┌─────────────────────────────────────────────────────────────┐
│  STEP 0 — Input Classification                              │
│                                                             │
│  Inspect the input before any research:                     │
│                                                             │
│  Remote Endpoint URL signals:                               │
│  • starts with https://                                     │
│  • path ends with /sse, /mcp, /mcp/, /sse/ or similar      │
│  • subdomain pattern: mcp.vendor.com or *.mcp.vendor.com    │
│  • NOT github.com                                           │
│  Examples: https://mcp.notion.com/sse                       │
│            https://mcp.linear.app/sse                       │
│            https://mcp.vercel.com                           │
│                                                             │
│  REMOTE ENDPOINT ─────────────────────────────────────────► │
│    Pre-set attributes (no research needed for these):       │
│    • Transport: HTTP/SSE (if /sse) or StreamableHttp (/mcp) │
│    • Deployment: Remote = Yes, Local = No, Container = No   │
│    • Hosting: SaaS Vendor = Yes (or 3rd Party SaaS)         │
│    • TLS: run curl --tlsv1.2 --tls-max 1.2 and             │
│            curl --tlsv1.3 → mark Yes only if connects       │
│    → Skip repo-clone; use REMOTE AUTH block at STEP 6       │
│                                                             │
│  GITHUB URL / SERVER NAME ────────────────────────────────► │
│    Pre-set (permanent, NOT overridden by path switch):      │
│    • STDIO = Yes, Local = Yes, GitHub = Yes                 │
│    Read README + vendor page first (STEP 1)                 │
│    → Check for remote endpoint after STEP 1                 │
└──────────────────────────────┬──────────────────────────────┘
                               │ (both paths continue)
                               ▼
┌─────────────────────────────────────┐
│  STEP 1 — Source Priority Fetch     │
│  1. Vendor docs site                │
│  2. GitHub repo (README, source)    │
│  3. PyPI / npm page                 │
│  4. MCP spec docs                   │
└──────────────┬──────────────────────┘
               │
               │ [GitHub/STDIO path only — skip if already Remote]
               ▼
    ┌──────────────────────────────────────┐
    │  Endpoint Discovery Check            │
    │                                      │
    │  Scan README + vendor docs for       │
    │  a remote endpoint URL:              │
    │  • /sse path or /mcp path            │
    │  • mcp.vendor.com subdomain          │
    │  • "remote MCP", "hosted endpoint",  │
    │    "connect via URL" mentions        │
    │                                      │
    │  FOUND ──────────────────────────►   │
    │  Show user:                          │
    │  "A remote endpoint is available:    │
    │   <url>                              │
    │   Would you like to connect via      │
    │   the remote endpoint, or use        │
    │   GitHub/STDIO instead?              │
    │   1. Remote endpoint                 │
    │   2. GitHub / STDIO"                 │
    │                                      │
    │  Answer 1 → switch to Remote path   │
    │    (pre-set Transport/Deploy/TLS     │
    │     same as STEP 0 Remote rules)     │
    │    ⚠ GitHub pre-sets STAY:           │
    │      STDIO=Yes, Local=Yes,           │
    │      GitHub=Yes NOT overridden       │
    │  Answer 2 → continue STDIO path      │
    │                                      │
    │  NOT FOUND ──────────────────────►   │
    │  Continue STDIO path                 │
    └──────────────────┬───────────────────┘
               │
               ▼
┌─────────────────────────────────────┐
│  STEP 2 — Fill All Attributes       │
│  • Distribution Type                │
│  • Pricing                          │
│  • Hosting Provider                 │
│  • Authentication                   │
│  • Data Protection / TLS            │
│  • Transport Protocol               │
│  • Tools Operations                 │
│  • Deployment Approach              │
│  • Capabilities                     │
└──────────────┬──────────────────────┘
               │
               ▼
┌─────────────────────────────────────┐
│  STEP 3 — MCP Protocol Version      │
│                                     │
│  A. Read uv.lock / package-lock     │
│     → get pinned SDK version        │
│                                     │
│  B. Fetch latest from PyPI / npm    │
│     → get current latest version   │
│                                     │
│  C. Pinned < Latest?                │
│     YES → upgrade SDK               │
│           uv add "mcp[cli]>=latest" │
│           verify server still starts│
│     NO  → use pinned version        │
│                                     │
│  D. Map version → protocol spec     │
│  Python mcp:                        │
│  ≥1.24.x → 2025-11-25              │
│  1.15–1.23 → 2025-06-18            │
│  1.5–1.14 → 2025-03-26             │
│  <1.5 → 2024-11-05                 │
└──────────────┬──────────────────────┘
               │
               ▼
┌─────────────────────────────────────┐
│  STEP 4 — Enumerate Tools           │
│                                     │
│  Capabilities - Tools               │
│  • Read source for @mcp.tool()      │
│  • Group by category                │
│  • Format:                          │
│    Category Name                    │
│        •  tool – description        │
│                                     │
│  Non-Read-Only Tools                │
│  • Detect: POST/PUT/DELETE calls,   │
│    file writes, state mutations     │
│  • Same grouped format              │
│  • Add Logic/Expert Note sub-bullets│
└──────────────┬──────────────────────┘
               │
               ▼
┌─────────────────────────────────────┐
│  STEP 5 — MCP Info                  │
│  • Git Repo Version                 │
│    - Fetch Releases page (/releases)│
│    - Fallback: Tags page (/tags)    │
│    - Use most recent version        │
│    - Format: v1.2.0 - Brief         │
│      description/focus (optional    │
│      emoji)                         │
│      Example: v1.2.0 - OpenAI SDK   │
│      Support 📎                     │
│    - If none found → Not published  │
│    - NO fallback to package.json /  │
│      pyproject.toml / setup.py      │
│  • Description (3–4 lines)          │
│  • Category (one of four types)     │
└──────────────┬──────────────────────┘
               │
               ▼
┌─────────────────────────────────────────────────────────────┐
│  STEP 6 — Auth + Connection  (branch on path chosen)         │
│  [Path set by STEP 0 OR by Endpoint Discovery user choice]  │
│                                                             │
│  ══════════════════════════════════════════════════════════ │
│  PATH A — REMOTE ENDPOINT                                   │
│  ══════════════════════════════════════════════════════════ │
│                                                             │
│  A1. Probe the endpoint (unauthenticated):                  │
│      curl -s -o /dev/null -w "%{http_code}" <endpoint-url> │
│      • 200 + SSE/JSON stream → no auth required            │
│      • 401 → read WWW-Authenticate header                   │
│      • 302/307 → OAuth redirect flow                        │
│      • 403/other → check vendor docs                        │
│                                                             │
│  A2. Detect auth method (check all):                        │
│      • WWW-Authenticate: Bearer → Bearer Token              │
│      • GET /.well-known/oauth-authorization-server          │
│        exists → OAuth 2.1 (Auth Code or Client Creds)       │
│      • Vendor docs / README: API key header name            │
│        (X-Api-Key, X-API-Key, Authorization: Bearer)        │
│      • No auth signal found → mark Authentication: No Auth  │
│                                                             │
│  A3. Ask config path (Global / Project)                     │
│                                                             │
│  A4. Build remote config with placeholder:                  │
│      Bearer/API key:                                        │
│        { "url": "<endpoint>",                               │
│          "headers": {                                       │
│            "Authorization": "Bearer <BEARER_TOKEN>" } }     │
│      OAuth 2.1 (browser flow):                              │
│        { "url": "<endpoint>" }                              │
│        (MCP client handles OAuth redirect — no key needed)  │
│                                                             │
│  A5. User fills credential in config file (never in chat)   │
│                                                             │
│  A6. Verify connection:                                     │
│      SSE endpoint:                                          │
│        curl -N -H "Accept: text/event-stream" \            │
│             -H "Authorization: Bearer <token>" \           │
│             <endpoint-url>                                  │
│        expect: event stream with endpoint/message event     │
│      StreamableHttp endpoint:                               │
│        curl -X POST -H "Content-Type: application/json" \  │
│             -H "Authorization: Bearer <token>" \           │
│             -d '{"jsonrpc":"2.0","id":1,                   │
│               "method":"initialize","params":{              │
│               "protocolVersion":"2024-11-05",               │
│               "capabilities":{},"clientInfo":               │
│               {"name":"test","version":"1.0"}}}' \         │
│             <endpoint-url>                                  │
│        expect: {"result":{"protocolVersion":...}}           │
│      SUCCESS → connection confirmed                         │
│      FAIL    → hand off to error-handling skill             │
│                                                             │
│  ══════════════════════════════════════════════════════════ │
│  PATH B — STDIO / LOCAL                                     │
│  (entered from STEP 0 GitHub URL  OR  user chose STDIO      │
│   when Endpoint Discovery offered a remote endpoint)        │
│  ══════════════════════════════════════════════════════════ │
│                                                             │
│  B1. Deployment: Local = Yes?                               │
│      NO  → skip to STEP 7                                   │
│      YES → trigger repo-clone skill                         │
│                                                             │
│  B2. After repo-clone returns:                              │
│      Step 1 — Extract command / args / env vars             │
│      Step 2 — Ask config path (Global / Project)            │
│      Step 3 — Credential check (Branch A / Branch B)        │
│        YES (Branch A) → write config + user fills creds     │
│        NO  (Branch B) → show how to get cred, then:        │
│          1. Get it now → wait → Branch A                    │
│          2. Proceed without → placeholder config            │
│          3. Stop → placeholder config + resume note         │
│      Step 4 — Write config with <KEY> placeholders          │
│      Step 5 — User fills credentials in file (never in chat)│
│      Step 6 — Verify connection (always attempt)            │
│               Send JSON-RPC initialize request:             │
│               echo '{"jsonrpc":"2.0","id":1,               │
│                 "method":"initialize","params":{             │
│                 "protocolVersion":"2024-11-05",              │
│                 "capabilities":{},"clientInfo":              │
│                 {"name":"test","version":"1.0"}}}' \        │
│               | <command> <args>                             │
│               expect: {"result":{"protocolVersion":...}}    │
│               SUCCESS → connection confirmed                │
│               FAIL    → hand off to error-handling skill    │
│                                                             │
│  ⚠ Connection verification is MANDATORY on both paths.      │
│    Never skip it — a verified connection is required        │
│    before proceeding to STEP 7.                             │
└──────────────────────────────┬──────────────────────────────┘
                               │ (connection done / auth setup done)
                               ▼
┌─────────────────────────────────────────────────────────────┐
│  STEP 7 — CCI Score Prompt                                  │
│                                                             │
│  "Include CCI MCP-server Score?"                            │
│                                                             │
│  1. Yes ──────────────────────────────────────┐            │
│                                               ▼            │
│                                   Calculate Score           │
│                                   10 attributes             │
│                                   → TOTAL / 53              │
│                                   → HIGH/MED/LOW %          │
│                                               │             │
│  2. No  ──────────────────────────────────────┤            │
│                                               │             │
└───────────────────────────────────────────────┼─────────────┘
                                                │
                                                ▼
                              ┌─────────────────────────────────┐
                              │  STEP 8 — Save CSV  (final)     │
                              │                                 │
                              │  Yes path → save with score rows│
                              │  No path  → save without score  │
                              │                                 │
                              │  → ~/Desktop/MCP_reports/       │
                              │    <servername>.csv             │
                              │                                 │
                              │  Print to user:                 │
                              │  "Report successfully generated │
                              │   and saved."                   │
                              │                                 │
                              │  DONE                           │
                              └─────────────────────────────────┘
```

---

## Common Mistakes (added rules)

- FastAPI/uvicorn in deps ≠ MCP HTTP transport → check `mcp.run()` call
- TLS from upstream API calls (https://api.vendor.com) ≠ MCP transport TLS
- ngrok for inbound callbacks ≠ vendor-hosted MCP endpoint
- README says "see vendor docs for remote MCP" → ALWAYS fetch vendor docs first before finalising Transport / TLS / Hosting / Deployment

---

## CSV Structure

```
MCP Info          → Description, Git Repo Version, Category
Distribution Type → Official, Community
MCP Protocol Ver  → 2025-11-25 / 2025-06-18 / 2025-03-26 / 2024-11-05
Pricing           → Free, Paid
Hosting Provider  → SaaS Vendor, 3rd Party, GitHub, GitLab, Bitbucket, SourceHut
Authentication    → OAuth 2.1 Auth Code, OAuth 2.1 Client Creds, Bearer, PAT, API Token
Data Protection   → TLS 1.3, TLS 1.2, Lower/none
Transport         → STDIO, HTTP/SSE, StreamableHttp, FastAPI
Tools Operations  → Read-only / Read+Update / Read+Update+Delete
Deployment        → Local, Container, Remote
Capabilities      → Tools, Resources, Prompts, Sampling
Capabilities-Tools→ [grouped bullet format]
Non-Read-Only     → [grouped bullet format with sub-notes]
CCI Score         → (optional — only if user said Yes)
```

---

## Credential Placeholder Map

| Auth type                    | Placeholder               |
|------------------------------|---------------------------|
| API Token / API Key          | `<API_KEY>`               |
| Bearer Token                 | `<BEARER_TOKEN>`          |
| Personal Access Token        | `<PERSONAL_ACCESS_TOKEN>` |
| OAuth Client ID              | `<OAUTH_CLIENT_ID>`       |
| OAuth Client Secret          | `<OAUTH_CLIENT_SECRET>`   |
| Generic secret               | `<SECRET_KEY>`            |

---

## Skill Handoff Map

```
mcp-server
  │
  ├─► attribute-researcher  (attribute research + documentation phase)
  │
  ├─► repo-clone        (when Deployment: Local = Yes)
  │       │
  │       └─► mcp-server Auth section  (after clone + install)
  │
  └─► error-handling    (when connection verify fails)
```
