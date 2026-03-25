# repo-clone Skill — Workflow Diagram

> Last updated: 2026-03-24
> Update this file every time SKILL.md is modified.

---

## Full Workflow

```
TRIGGER
  │
  ├── Direct: user provides GitHub URL + wants to run locally
  └── From mcp-server: Deployment=Local detected, README already read
              │
              ▼
┌─────────────────────────────────────────┐
│  STEP 1 — Fetch Repository Docs         │
│                                         │
│  From mcp-server? → skip (use context) │
│  Direct trigger?  → fetch:             │
│    • README.md                          │
│    • requirements.txt / pyproject.toml  │
│    • package.json / Cargo.toml          │
│    • .env.example                       │
│    • Any linked setup docs              │
└──────────────────┬──────────────────────┘
                   │
                   ▼
┌─────────────────────────────────────────┐
│  STEP 2 — Determine Connection Method   │
│                                         │
│  Read README → identify type:           │
│                                         │
│  ┌──────────────────────────────────┐   │
│  │ STDIO      │ local subprocess    │   │
│  │ (most common for open-source)    │   │
│  ├──────────────────────────────────┤   │
│  │ Package    │ PyPI/npm, no clone  │   │
│  │            │ pip install / npx  │   │
│  ├──────────────────────────────────┤   │
│  │ Docker     │ Dockerfile present  │   │
│  ├──────────────────────────────────┤   │
│  │ Remote     │ vendor URL endpoint │   │
│  └──────────────────────────────────┘   │
│                                         │
│  Pick recommended/simplest method       │
│  Explain choice briefly to user         │
└──────────────────┬──────────────────────┘
                   │
                   ▼
┌─────────────────────────────────────────┐
│  STEP 3 — Present Plan + Get Approval   │
│                                         │
│  Show:                                  │
│  • Repository URL                       │
│  • Server type                          │
│  • Clone path (default or user-chosen)  │
│  • Dependencies list                    │
│  • Env vars required                    │
│  • Ordered steps to be taken            │
│                                         │
│  "Proceed? (yes / no / different path)" │
│                                         │
│  NO ──────────────────► STOP            │
│  Different path ───────► update path    │
│  YES ──────────────────► continue       │
└──────────────────┬──────────────────────┘
                   │
                   ▼
┌─────────────────────────────────────────┐
│  STEP 4 — Clone Repository              │
│                                         │
│  git clone https://github.com/<org>/    │
│    <repo>.git <clone-path>              │
│                                         │
│  Verify: directory exists,              │
│          expected files present         │
│  Report: "Cloned to <path>" or error    │
└──────────────────┬──────────────────────┘
                   │
                   ▼
┌─────────────────────────────────────────┐
│  STEP 5 — Install Dependencies          │
│                                         │
│  STDIO / Python:                        │
│    python3 -m venv .venv                │
│    pip install -r requirements.txt      │
│    (or: uv sync)                        │
│                                         │
│  Node.js / TypeScript:                  │
│    npm install                          │
│    npm run build  (if build step)       │
│                                         │
│  Docker:                                │
│    docker build -t <repo-name> .        │
│                                         │
│  Published package:                     │
│    pip install <package>                │
│    (npx needs no install)               │
│                                         │
│  Confirm: packages importable /         │
│           build artifacts exist         │
└──────────────────┬──────────────────────┘
                   │
                   ▼
┌─────────────────────────────────────────┐
│  STEP 6 — Extract & Hand off to         │
│           mcp-server Auth Section       │
│                                         │
│  Read from local disk:                  │
│  • README.md  → command, args, env vars │
│  • pyproject.toml / package.json        │
│  • .env.example                         │
│  • uv.lock / package-lock.json          │
│  • main script → @mcp.tool(), env reads │
│                                         │
│  Determine:                             │
│  • command  (uv / node / python3)       │
│  • args     (["--directory", ...])      │
│  • env vars (names + auth types)        │
│                                         │
│  Trigger mcp-server Auth section with:  │
│  → clone path                           │
│  → command / args / env var names       │
│  → README content (already read)        │
│  → "read from disk, not GitHub"         │
└──────────────────┬──────────────────────┘
                   │
                   ▼
        mcp-server Auth section runs:
        • Credential check (have key?)
        • Config path selection
        • Write settings.json
        • User fills credentials
                   │
                   ▼
┌─────────────────────────────────────────┐
│  STEP 7 — Verify Connection             │
│                                         │
│  STDIO:                                 │
│  echo '{"jsonrpc":"2.0","method":       │
│  "initialize",...}' | <command> <args>  │
│                                         │
│  Docker:                                │
│  docker run --rm -i --env-file .env ... │
│                                         │
│  Package:                               │
│  npx <pkg> --version                    │
│  python -m <module> --help              │
│                                         │
│  SUCCESS ──────────────► Step 8         │
│  FAIL    ──────────────► error-handling │
│                          skill          │
└──────────────────┬──────────────────────┘
                   │
                   ▼
┌─────────────────────────────────────────┐
│  STEP 8 — Final Summary                 │
│                                         │
│  Clone path  : <path>                   │
│  Command     : <command>                │
│  Args        : <args>                   │
│  Config file : <settings.json path>     │
│  Status      : Running / Failed         │
│                                         │
│  Next steps:                            │
│  1. Fill remaining <KEY> placeholders   │
│  2. Restart Claude Code                 │
└─────────────────────────────────────────┘
```

---

## Connection Method Decision

```
README says...
      │
      ├── runs via stdin/stdout ──────────────► STDIO
      │   (most open-source servers)
      │
      ├── pip install / npx available ────────► Published Package
      │   (no clone needed)
      │
      ├── Dockerfile present ────────────────► Docker
      │
      └── vendor URL endpoint ────────────────► Remote
          (skip all local setup)
```

---

## Skill Handoff Map

```
repo-clone
  │
  ├─── triggered by ──────────────────── mcp-server (Deployment=Local)
  │                                  OR  user directly
  │
  ├─── hands off to (Step 6) ─────────── mcp-server Auth section
  │
  └─── hands off to (Step 7 fail) ────── error-handling skill
```
