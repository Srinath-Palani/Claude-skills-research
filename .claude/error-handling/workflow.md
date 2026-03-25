# error-handling Skill — Workflow Diagram

> Last updated: 2026-03-24
> Update this file every time SKILL.md is modified.

---

## Full Workflow

```
ERROR APPEARS
(any MCP failure: crash, 401, import error,
 connection refused, stack trace, "not working")
              │
              ▼
┌─────────────────────────────────────────┐
│  PRE-CHECK — Read learned-fixes.md      │
│                                         │
│  Does error match a known pattern?      │
│                                         │
│  YES ──────────────────────────────────►│
│       skip Phases 2–3, go to Phase 4   │
│  NO  ──────────────────────────────────►│
│       continue to Phase 1              │
└──────────────────┬──────────────────────┘
                   │
                   ▼
┌─────────────────────────────────────────┐
│  PHASE 1 — Capture the Error            │
│                                         │
│  Already in context? → skip, go Phase 2 │
│  Not in context? → ask:                 │
│    1. Full error / stack trace          │
│    2. MCP server name or GitHub URL     │
│    3. Where error appears               │
│    4. What triggered it                 │
└──────────────────┬──────────────────────┘
                   │
                   ▼
┌─────────────────────────────────────────┐
│  PHASE 2 — Classify the Error           │
│                                         │
│  ┌─────┬───────────────────────────┐    │
│  │  1  │ Dependency / Install      │    │
│  │     │ ModuleNotFoundError,      │    │
│  │     │ ImportError, npm ERR!     │    │
│  ├─────┼───────────────────────────┤    │
│  │  2  │ Configuration             │    │
│  │     │ wrong path, JSON error,   │    │
│  │     │ missing mcpServers block  │    │
│  ├─────┼───────────────────────────┤    │
│  │  3  │ Authentication            │    │
│  │     │ 401, 403, invalid key,    │    │
│  │     │ token expired, KeyError   │    │
│  ├─────┼───────────────────────────┤    │
│  │  4  │ Transport / Process       │    │
│  │     │ ECONNREFUSED, EPIPE,      │    │
│  │     │ STDIO exited, spawn error │    │
│  ├─────┼───────────────────────────┤    │
│  │  5  │ Protocol Version          │    │
│  │     │ Unsupported version,      │    │
│  │     │ method not found -32601   │    │
│  ├─────┼───────────────────────────┤    │
│  │  6  │ Runtime / Crash           │    │
│  │     │ RuntimeError, TypeError,  │    │
│  │     │ AttributeError, OOM       │    │
│  ├─────┼───────────────────────────┤    │
│  │  7  │ Environment / Path        │    │
│  │     │ wrong Python version,     │    │
│  │     │ venv not active, no PATH  │    │
│  └─────┴───────────────────────────┘    │
│                                         │
│  Multiple categories? List all.         │
│  Fix order: Dependency → Config → Auth  │
└──────────────────┬──────────────────────┘
                   │
                   ▼
┌─────────────────────────────────────────┐
│  PHASE 3 — Step-by-Step Diagnosis       │
│  (stop at first failure found)          │
│                                         │
│  3.1 Runtime environment                │
│      python3/node/uv --version          │
│      which python3 / node / uv          │
│             │                           │
│  3.2 Config file                        │
│      read settings.json                 │
│      check: command on PATH?            │
│             args paths exist?           │
│             env vars not <KEY>?         │
│             valid JSON?                 │
│             │                           │
│  3.3 Dependencies                       │
│      import mcp / require sdk           │
│      uv run python -c "import mcp"      │
│             │                           │
│  3.4 Direct server start                │
│      run exact command from settings    │
│      capture stdout + stderr            │
│             │                           │
│  3.5 Send initialize request            │
│      echo '{"jsonrpc":"2.0",...}'       │
│        | <command> <args>               │
│      expect JSON with "result"          │
│             │                           │
│  3.6 Claude Code logs                   │
│      tail ~/Library/Logs/Claude/mcp*.log│
└──────────────────┬──────────────────────┘
                   │
                   ▼
┌─────────────────────────────────────────┐
│  PHASE 4 — Present Fix Plan             │
│                                         │
│  Show:                                  │
│    Category  : <error category>         │
│    Root cause: <one sentence>           │
│                                         │
│    Step 1: <what will be done>          │
│      Command: <exact command>           │
│      Risk   : low / medium              │
│                                         │
│    Step 2: <next fix if needed>         │
│    ...                                  │
│                                         │
│  "Shall I proceed with Step 1?"         │
│  yes → execute                          │
│  skip → next step                       │
│  cancel → STOP                          │
└──────────────────┬──────────────────────┘
                   │
                   ▼
┌─────────────────────────────────────────┐
│  PHASE 5 — Execute Fix (by category)    │
│                                         │
│  1 Dependency  → pip install / uv sync  │
│                  npm install + build    │
│                  install uv/node if     │
│                  command not found      │
│                                         │
│  2 Config      → fix command path       │
│                  fix args paths         │
│                  add missing env block  │
│                  repair JSON syntax     │
│                                         │
│  3 Auth        → if <KEY> placeholder:  │
│                  tell user to fill file │
│                  if 401 with key filled:│
│                  check var name in code │
│                                         │
│  4 Transport   → run server directly    │
│                  capture crash reason   │
│                  → usually Auth or Dep  │
│                                         │
│  5 Protocol    → upgrade mcp SDK        │
│                  uv add "mcp>=1.15"     │
│                  npm install sdk@latest │
│                                         │
│  6 Runtime     → read stack trace line  │
│                  check for missing env  │
│                  check SDK compat       │
│                                         │
│  7 Environment → check python version   │
│                  uv python install X.Y  │
│                  chmod +x if denied     │
└──────────────────┬──────────────────────┘
                   │
        After each step: "Proceed with
        Step N+1? (yes / skip / cancel)"
                   │
                   ▼
┌─────────────────────────────────────────┐
│  PHASE 6 — Verify Connection            │
│                                         │
│  Send initialize request:               │
│  echo '{"jsonrpc":"2.0","id":1,         │
│   "method":"initialize","params":{...}}'│
│    | <command> <args>                   │
│                                         │
│  SUCCESS ──────────────────────────────►│
│  "Connection verified.                  │
│   Restart Claude Code."                 │
│                                         │
│  FAIL ─────────────────────────────────►│
│  Go back to Phase 3 with new error      │
└──────────────────┬──────────────────────┘
                   │
                   ▼
┌─────────────────────────────────────────┐
│  PHASE 7 — Report to User               │
│                                         │
│  Server     : <server name>             │
│  Error was  : <original error>          │
│  Root cause : <what was wrong>          │
│  Fix applied: <what was changed>        │
│  Status     : Connected / Still failing │
│  Config     : <settings.json path>      │
└──────────────────┬──────────────────────┘
                   │
                   ▼
┌─────────────────────────────────────────┐
│  SKILL 2.0 — Self-Upgrade               │
│  (only if fix was NEW — not in          │
│   error-patterns.md already)            │
│                                         │
│  Append to references/learned-fixes.md: │
│  • Error signals                        │
│  • Root cause                           │
│  • Fix commands                         │
│  • Verification output                  │
│  • First seen date + server name        │
│                                         │
│  Add signal to error-patterns.md        │
│  signal table for Phase 2 next time     │
└─────────────────────────────────────────┘
```

---

## Error Category Quick Reference

```
Signal in error text               → Category
──────────────────────────────── ───────────────────
ModuleNotFoundError / ImportError → Dependency
Cannot find module / npm ERR!     → Dependency
command not found: uv/node/python → Environment
No such file or directory (path)  → Configuration
JSON parse error in settings      → Configuration
spawn <cmd> ENOENT                → Configuration
401 Unauthorized / 403 Forbidden  → Authentication
Invalid API key / token expired   → Authentication
<KEY> still in settings.json      → Authentication
Connection refused / ECONNREFUSED → Transport
STDIO process exited / EPIPE      → Transport
Unsupported protocol version      → Protocol Version
method not found (-32601)         → Protocol Version
RuntimeError / AttributeError     → Runtime/Crash
TypeError in tool function        → Runtime/Crash
python3 version mismatch          → Environment/Path
permission denied                 → Environment/Path
```

---

## Multi-Layer Error Fix Order

```
"Server won't start + missing module"
  → Fix: Dependency first, then Configuration

"401 after fresh install"
  → Fix: Authentication (<KEY> placeholder still set)

"Connection refused on port"
  → Fix: Run server directly, then Auth or Dependency

"Unknown method / method not found"
  → Fix: Protocol Version (upgrade mcp SDK)
```

---

## Self-Upgrade Loop

```
Fix succeeds + new pattern
        │
        ▼
Append to learned-fixes.md
        │
        ▼
Add signal to error-patterns.md
        │
        ▼
Next session: Pre-check matches instantly
→ skip Phases 2–3 → go straight to Phase 4
```

---

## Skill Handoff Map

```
error-handling
  │
  ├─── triggered by ──── mcp-server (connection verify fails)
  │                   OR repo-clone (server start fails)
  │                   OR user directly (any MCP error)
  │
  └─── references ────── references/error-patterns.md
                          references/learned-fixes.md
```
