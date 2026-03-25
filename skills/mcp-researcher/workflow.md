# MCP Researcher + Local Setup + Error Recovery — Workflow Diagram

> Last updated: 2026-03-25 (repo-clone fully integrated, conditional local setup)

---

## Full Workflow: 8 Steps + Conditional Local Setup + Inline Error Recovery

```
USER INPUT
(endpoint URL, GitHub URL, or server name)
        │
        ▼
┌─────────────────────────────────────┐
│ STEP 0 — Config & Input Type        │
│                                     │
│ First-time:                         │
│  ? Save path? [~/Desktop/MCP_...]   │
│  ? Clone path? [~/MCP_repos/]       │
│  → Store in .claude/settings.json   │
│                                     │
│ Classify input:                     │
│  ├─ https://... → Type A (endpoint) │
│  ├─ github.com → Type B (GitHub)    │
│  └─ Plain name → Type C (search)    │
└────────┬────────────────────────────┘
         │
    ┌────┼────┐
    ▼    ▼    ▼
Type A  B    C
    │
    ▼
┌─────────────────────────────────────┐
│ STEP 1 — Protocol Version Verify    │
│                                     │
│ • Find SDK (uv.lock, package.json)  │
│ • Check if outdated                 │
│ • Ask to upgrade if needed          │
│ • Map to protocol version           │
└────────┬────────────────────────────┘
         │
    ┌────┴────────────┐
    ▼                 ▼
 Type A            Type B
Endpoint           GitHub

Type A Path:
TLS verify
Detect auth
Ask config location
Test connection
(curl endpoint)

Type B Path:
┌─────────────────────────┐
│ ASK: How to run?        │
├─────────────────────────┤
│ Option 1: Local setup   │
│ (clone+install)         │
│ Option 2: Remote        │
│ (check endpoint)        │
│ Option 3: Research only │
│ (skip setup)            │
└────────┬────────────────┘
         │
    ┌────┼────┐
    │    │    │
    O1   O2   O3

Option 1 Path (Local Setup):
    │
    ▼
┌─────────────────────┐
│ STEP 3 — Local      │
│ Setup (Conditional) │
├─────────────────────┤
│ 3.1 Clone repo      │
│     to path         │
│ 3.2 Determine       │
│     method          │
│ 3.3 Get approval    │
│     for install     │
│ 3.4 Install deps    │
│     (Python/Node/   │
│      Docker/uv)     │
│ 3.5 Extract config  │
│     (cmd/args/env)  │
│ 3.6 Test connection │
│     (init JSON-RPC) │
│                     │
│ SUCCESS → Step 4    │
│ FAIL → Error Rec.   │
└────┬────────────────┘
     │
Option 2 Path:
    │
    └─► Check for remote endpoint in README
        Use as remote type (like Type A)

Option 3 Path:
    │
    └─► SKIP local setup, go to Step 4

     │
    ┌┴───────────────────────────┐
    │                             │
(all paths converge)
    │
    ▼
┌─────────────────────────────────────┐
│ STEP 4 — Fill Attributes            │
│                                     │
│ Enforce rules:                      │
│ • STDIO=Yes → Local=Yes (auto)      │
│ • HTTP/SSE → Remote=Yes (auto)      │
│ • Only ONE in mutually exclusive    │
│ • EVERY YES backed by evidence      │
│                                     │
│ Source priority:                    │
│ 1. Vendor docs                      │
│ 2. GitHub / source                  │
│ 3. PyPI / npm                       │
│ 4. MCP spec                         │
└────────┬────────────────────────────┘
         │
         ▼
┌─────────────────────────────────────┐
│ STEP 5 — Tools & Capabilities       │
│                                     │
│ • Find @mcp.tool() decorators       │
│ • Group by category                 │
│ • List non-read-only operations     │
│ • Check capabilities (T/R/P/S)      │
└────────┬────────────────────────────┘
         │
         ▼
┌─────────────────────────────────────┐
│ STEP 6 — MCP Info & Scoring         │
│                                     │
│ • Git repo version                  │
│ • Description, Category             │
│ • CCI Score (Option 1/2/3)          │
│                                     │
│ Scoring (10 attributes, 0–53):      │
│ • Protocol, Distribution, Pricing   │
│ • Hosting, Auth, TLS                │
│ • Transport, Operations             │
│ • Deployment, Capabilities          │
└────────┬────────────────────────────┘
         │
         ▼
┌─────────────────────────────────────┐
│ STEP 7 — Save CSV Report            │
│                                     │
│ Location: User-configured path      │
│ Filename: <servername>.csv          │
│ Columns: Category | Attr | Status   │
│                                     │
│ Done: "Report saved successfully"   │
└─────────────────────────────────────┘

ERROR RECOVERY (Inline on Connection Test Fail at 3.6 or 2)
└───────────────────────────────────────────────────────
    │
    ▼
┌──────────────────────────┐
│ PHASE 1 — Capture Error  │
│ Classify by category:    │
│ 1. Dependency/Install    │
│ 2. Configuration         │
│ 3. Authentication        │
│ 4. Transport/Process     │
│ 5. Protocol Version      │
│ 6. Runtime/Crash         │
│ 7. Environment/Path      │
└────────┬─────────────────┘
         │
         ▼
┌──────────────────────────┐
│ PHASE 2 — Diagnose       │
│                          │
│ Run checks in order:     │
│ 1. Runtime version       │
│ 2. Config validity       │
│ 3. Dependencies          │
│ 4. Server start direct   │
│ 5. JSON-RPC test         │
│ 6. Claude Code logs      │
│                          │
│ Stop at first failure    │
└────────┬─────────────────┘
         │
         ▼
┌──────────────────────────┐
│ PHASE 3 — Fix Plan       │
│                          │
│ Show:                    │
│ • Root cause             │
│ • Step 1: command+risk   │
│ • Step 2: command+risk   │
│ • (etc.)                 │
│                          │
│ Ask approval for each    │
│ step before execution    │
└────────┬─────────────────┘
         │
    ┌────┴────┐
    │         │
 CANCEL      YES
    │         │
    │         ▼
    │ ┌──────────────────────┐
    │ │ PHASE 4 — Execute    │
    │ │                      │
    │ │ By category:         │
    │ │ • Deps: venv/npm/uv  │
    │ │ • Config: file edit  │
    │ │ • Auth: file edit    │
    │ │ • Transport: restart │
    │ │                      │
    │ │ Approval for each    │
    │ └────────┬─────────────┘
    │          │
    │          ▼
    │ ┌──────────────────────┐
    │ │ PHASE 5 — Verify     │
    │ │                      │
    │ │ Test init again      │
    │ │                      │
    │ │ SUCCESS → PHASE 6    │
    │ │ FAIL → PHASE 2       │
    │ └────────┬─────────────┘
    │          │
    │          ▼
    │ ┌──────────────────────┐
    │ │ PHASE 6 — Record     │
    │ │ (Skill 2.0)          │
    │ │                      │
    │ │ Append to            │
    │ │ learned-fixes.md:    │
    │ │ • Error title        │
    │ │ • Signals            │
    │ │ • Root cause         │
    │ │ • Fix steps          │
    │ │ • Date + category    │
    │ └────────┬─────────────┘
    │          │
    └──────────┴─ Continue to Step 4 (research)
```

---

## Connection Methods (Step 3.2 Decision)

| Method | Indicator | Setup | Deps Install |
|--------|-----------|-------|--------------|
| **STDIO** (local subprocess, most common) | stdin/stdout, subprocess | Clone + install | Python: venv+pip / Node: npm+build |
| **Published Package** | PyPI/npm registry, pip/npx | No clone needed | `pip install <pkg>` or `npx <pkg>` |
| **Docker** | Dockerfile present | Clone + docker build | `docker build -t <repo-name> .` |
| **Remote** | vendor URL, no local setup | Skip all setup | N/A |

---

## Local Setup Decision (Step 3)

```
GitHub URL detected. How to proceed?

Option 1: Local setup (clone + install + test)
          → Full setup: clone, install deps, test connection
          → Then research attributes

Option 2: Use remote endpoint (if available)
          → Check README for remote endpoint
          → Use as remote type (like endpoint URL)
          → Then research attributes

Option 3: Just research attributes (no setup)
          → Skip clone + install entirely
          → Only research and document attributes
          → No local connection testing

Select option (1/2/3):
```

**User Choice Determines Path:**
- **Option 1**: Execute Steps 3.1–3.6 (local setup) then Step 4 (research)
- **Option 2**: Look for remote endpoint, use as Type A, then Step 4
- **Option 3**: Skip to Step 4 (research only)

---

## Dependency Installation Commands (Step 3.4)

| Language | Command |
|----------|---------|
| Python | `cd <clone-path> && python3 -m venv .venv && source .venv/bin/activate && pip install -r requirements.txt` |
| Node/TS | `cd <clone-path> && npm install && npm run build` (if build in README) |
| Docker | `cd <clone-path> && docker build -t <repo-name> .` |
| uv | `uv sync` |
| Published | `pip install <package-name>` or `npx <package>` |

---

## Diagnosis Checklist (Phase 2)

| # | Check | Command | Pass Condition |
|---|-------|---------|---|
| 1 | Runtime version | `python3 --version` / `node --version` | Matches README |
| 2 | Config validity | `cat ~/.claude/settings.json` | Valid JSON, command exists, paths resolve |
| 3 | Dependencies | `python -c "import mcp"` | No ImportError |
| 4 | Server start | `<command> <args>` | Exit 0 or wait for input |
| 5 | JSON-RPC test | `echo '{"jsonrpc":...}' \| <cmd>` | Response with result + protocolVersion |
| 6 | Claude Logs | `tail ~/Library/Logs/Claude/mcp*.log` | Error message visible |

---

## Approval Gate (Every Phase 3/4 Step)

```
For Step 1:
  "Proceed with Step 1? (yes / skip / cancel)"

After success:
  "Step 1 complete. Proceed with Step 2? (yes / skip / cancel)"

REQUIRED: explicit "yes" for each step
BEHAVIOR: "cancel" = STOP, "skip" = next step
```

---

## Skill 2.0: Learning Protocol (Phase 6)

When error fix succeeds, append to `references/learned-fixes.md`:

```markdown
## [Error title — e.g., "ImportError: No module named '_mcp_core'"]

**Error signals:**
- [what user saw]
- [where it appeared]

**Root cause:**
[one sentence]

**Fix steps (in order):**
1. [step]
2. [step]
3. [step]

**Verification:**
[how to verify fix worked]

**First seen:** YYYY-MM-DD | **Category:** [1–7]
```

---

## Key Enforcement Rules

```
TRANSPORT ↔ DEPLOYMENT
  STDIO = Yes → Local = Yes (auto-set)
  HTTP/SSE OR StreamableHttp = Yes → Remote = Yes (auto-set)

MUTUALLY EXCLUSIVE (only ONE = Yes):
  • Distribution: Official OR Community
  • Pricing: Free OR Paid
  • Hosting: SaaS OR GitHub OR GitLab
  • Transport: STDIO OR HTTP/SSE OR StreamableHttp
  • Tools Ops: Read-only OR R+U OR R+U+D

LOCAL SETUP SKIP CONDITIONS:
  ✓ User selects "Research only" (Option 3)
  ✓ Input is Remote Endpoint (Type A)
  ✓ Server is remote-hosted (Deployment: Remote = Yes)
  ✓ User already has server running
```

---

## Integration Points

| Event | Trigger | Action |
|-------|---------|--------|
| Type B (GitHub) received | Always | Ask deployment preference (Option 1/2/3) |
| Option 1 selected | Local setup | Execute Steps 3.1–3.6 before research |
| Option 3 selected | Research only | Skip to Step 4 |
| Connection test fails | Either Type | Auto-trigger Phase 1 (inline recovery) |
| Known pattern found | learned-fixes.md | Phase 2: skip to fix directly |
| Fix succeeds | After Phase 5 | Phase 6: record in learned-fixes.md |

---

## Summary: Features Met

✅ 8-step research workflow
✅ 3 input type handling (endpoint/GitHub/name)
✅ Conditional local setup (only if user chooses Option 1)
✅ 4 connection methods (STDIO/Package/Docker/Remote)
✅ Dependency auto-detection and installation
✅ Configuration extraction from local repo
✅ Protocol version verification
✅ Logic enforcement (Transport↔Deployment)
✅ 7-phase inline error recovery (no context-switching)
✅ 7-category auto-classification of errors
✅ 6-check diagnosis checklist
✅ Approval-gated fix execution
✅ 10-attribute security scoring (53-point scale)
✅ CSV report generation
✅ Skill 2.0 self-learning (learned-fixes.md)
✅ Security mandate enforced throughout
✅ **FAST & EFFICIENT:** Skips repo-clone entirely for remote/research-only modes
