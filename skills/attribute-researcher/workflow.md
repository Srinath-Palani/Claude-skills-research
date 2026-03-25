# attribute-researcher Skill — Workflow Diagram

> Last updated: 2026-03-24
> Update this file every time SKILL.md is modified.

---

## Full Workflow

```
TRIGGER
  │
  ├── From mcp-server: attribute-filling phase reached
  └── Direct: user asks to fill/document MCP server attributes
              │
              ▼
┌─────────────────────────────────────┐
│  STEP 1 — Fetch Sources             │
│                                     │
│  1. Vendor documentation page       │
│  2. GitHub README + linked docs     │
│  3. Source code:                    │
│     • tool registration files      │
│     • connection / auth modules     │
│     • main entry point              │
│  4. requirements.txt / pyproject    │
│     / package.json (dep versions)   │
│  5. .env.example (env var names)    │
└──────────────┬──────────────────────┘
               │
               ▼
┌─────────────────────────────────────┐
│  STEP 2 — Fill All Attributes       │
│                                     │
│  For each field:                    │
│  • Apply field rule                 │
│  • Record: Yes / No                 │
│  • Record: source + evidence        │
│                                     │
│  Fields:                            │
│  • Distribution Type (Official /    │
│    Community — mutually exclusive)  │
│  • Pricing (Free / Paid)            │
│  • Hosting Provider (mark all       │
│    that apply: SaaS Vendor,         │
│    GitHub, GitLab, Bitbucket,       │
│    SourceHut, 3rd Party SaaS)       │
│  • Authentication (OAuth 2.1 Auth   │
│    Code, OAuth 2.1 Client Creds,    │
│    Bearer Token, PAT, API Token)    │
│  • Data Protection / TLS            │
│    (Remote=TLS yes; STDIO=TLS no)   │
│  • Transport (STDIO / HTTP/SSE /    │
│    StreamableHttp — check mcp.run() │
│    not just framework imports)      │
│  • Tools Operations (only highest   │
│    tier = Yes; others = No)         │
│  • Deployment (Local / Container /  │
│    Remote — not mutually exclusive) │
│  • Capabilities (Tools / Resources  │
│    / Prompts / Sampling)            │
└──────────────┬──────────────────────┘
               │
               ▼
┌─────────────────────────────────────┐
│  STEP 3 — Capabilities Tools        │
│           detailed_info             │
│                                     │
│  • Read @mcp.tool() decorators      │
│  • Group by logical category        │
│  • Format:                          │
│    Category Name                    │
│        •  Action Name – description │
│  • Blank line between categories    │
└──────────────┬──────────────────────┘
               │
               ▼
┌─────────────────────────────────────┐
│  STEP 4 — Non-Read-Only Tools       │
│           detailed_info             │
│                                     │
│  Detect via:                        │
│  • readOnlyHint=False decorator     │
│  • POST/PUT/PATCH/DELETE calls      │
│  • File write / delete ops          │
│  • Any state-mutating function      │
│                                     │
│  Format same as Step 3 but use      │
│  actual tool_name + sub-notes       │
│                                     │
│  If none found:                     │
│  "None — all tools are read-only"   │
└──────────────┬──────────────────────┘
               │
               ▼
┌─────────────────────────────────────┐
│  STEP 5 — Return to mcp-server      │
│                                     │
│  Pass back:                         │
│  • All attribute values + evidence  │
│  • Capabilities Tools detail        │
│  • Non-Read-Only Tools detail       │
│                                     │
│  mcp-server continues with:         │
│  • MCP Info (version, desc, cat)    │
│  • CCI score prompt                 │
│  • CSV generation + save            │
└─────────────────────────────────────┘
```

---

## Field Rule Quick Reference

```
Distribution Type  → mutually exclusive (Official XOR Community)
Pricing            → Free and/or Paid
Hosting Provider   → mark ALL that apply
Authentication     → report ALL detected methods
TLS                → transport layer only (not upstream API calls)
Transport          → verify mcp.run() call, not framework imports
Tools Operations   → ONLY the highest tier = Yes
Deployment         → Local + Container can both be Yes
Capabilities       → vendor docs + source code both required for Resources
```

---

## Skill Handoff Map

```
attribute-researcher
  │
  ├─── triggered by ──────────────────── mcp-server (attribute phase)
  │                                  OR  user directly
  │
  └─── returns results to ────────────── mcp-server (MCP Info + CSV phase)
```
