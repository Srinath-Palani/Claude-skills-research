---
name: error-handling
user-invocable: true
description: >
  Use when the user reports any MCP server or client error, connection failure, or
  installation problem. Use when the user says "it's not working", "MCP server failed",
  "connection refused", "can't find the server", "dependency error", "import error",
  "tool not found", "401", "403", "timeout", or "server crashed". Use when the user
  pastes any stack trace or error output from an MCP server. Use when the user wants
  help fixing an MCP server that won't start, authenticate, or connect. Also triggered
  automatically from mcp-server and repo-clone when a connection verification fails.
  Covers every error layer: dependencies, configuration, authentication, transport,
  protocol version mismatch, runtime panics, and Claude Code settings. Walks the user
  through each fix step-by-step, asks for approval before executing anything, and
  self-upgrades by saving every new pattern to references/learned-fixes.md.
---

# MCP Error Handler — Diagnose, Fix, Connect

> Full workflow diagram: see `workflow.md` in this directory.
> Update `workflow.md` whenever this skill is modified.

This skill owns all MCP error recovery. It works across the full stack:
`mcp-server` research → `repo-clone` installation → Claude Code config → live connection.

When an error appears, this skill takes over: classifies it, diagnoses root cause step by
step, presents a clear fix plan, gets the user's approval, executes the fix, verifies the
connection, and writes the new pattern into its own knowledge base so the next run starts
smarter.

---

## SECURITY MANDATE — Always enforced

- **Never ask the user to paste any credential, API key, or token into this chat**, even
  when diagnosing authentication errors.
- When fixing auth errors, always direct the user to edit the settings file directly —
  never ask them to share the credential value in chat.
- If the user accidentally pastes a credential during error diagnosis, immediately say:
  ```
  Security alert: Please do not share credentials here. Treat this value as potentially
  compromised — revoke it and generate a new one, then add it directly to the settings
  file at <path>. Do NOT paste it here.
  ```

---

## Usage

Diagnose and fix any MCP server error:

```
"The MCP server won't start"
"I'm getting a 401 error from my MCP server"
"Connection refused when Claude Code tries to connect"
"ModuleNotFoundError: No module named 'mcp'"
"My MCP server was working but now it's not"
[paste any stack trace or error output]
```

Walks through diagnosis phase by phase, presents a fix plan, gets approval before each
step, verifies the connection, and saves the solution for future sessions.

---

**Read `references/error-patterns.md` before starting every session** — it contains all
previously learned error patterns and their proven fixes. Check if the current error matches
a known pattern first; if it does, go straight to the fix.

**After every successful fix**, append the new pattern to `references/learned-fixes.md`
following the Skill 2.0 protocol at the bottom of this file.

---

## Phase 1 — Capture the error

Ask the user (or read from context if already provided):

```
To diagnose this I need to see the full error. Please share:
  1. The complete error message or stack trace
  2. The MCP server name or GitHub URL
  3. Where the error appears (Claude Code / terminal / during install / on first use)
  4. What you were doing when it happened
```

If the error is already in context (e.g. user pasted a stack trace), skip the question and
go straight to Phase 2.

---

## Phase 2 — Classify the error

Match the error to one of these seven categories. Read `references/error-patterns.md` first —
a known pattern match skips directly to Phase 4.

| # | Category | Key signals |
|---|----------|-------------|
| 1 | **Dependency / Install** | `ModuleNotFoundError`, `ImportError`, `Cannot find module`, `npm ERR!`, `pip install failed`, `command not found: uv/node/python` |
| 2 | **Configuration** | `Invalid config`, `command not found`, wrong path in `settings.json`, `args` mismatch, missing `mcpServers` block, JSON parse error in settings |
| 3 | **Authentication** | `401 Unauthorized`, `403 Forbidden`, `Invalid API key`, `token expired`, `OAuth error`, missing env var, `KeyError` on env var name |
| 4 | **Transport / Process** | `Connection refused`, `ECONNREFUSED`, `STDIO process exited`, `spawn error`, `broken pipe`, `EPIPE`, server process not starting |
| 5 | **Protocol Version** | `Unsupported protocol version`, `capability negotiation failed`, `unknown method`, JSON-RPC `method not found` (-32601) |
| 6 | **Runtime / Crash** | Unhandled exception in server, `RuntimeError`, `AttributeError`, `TypeError` in server code, OOM, segfault |
| 7 | **Environment / Path** | Wrong Python version, `venv` not activated, binary not on PATH, wrong working directory, permission denied |

If the error spans multiple categories, list all that apply — fix them in order (dependencies
before config before auth).

---

## Phase 3 — Step-by-step diagnosis

Run these checks in order. Stop at the first failure and report what you found.

### 3.1 — Check the runtime environment
```bash
python3 --version          # should match README requirement
node --version             # if Node.js server
uv --version               # if uv-managed project
which python3 / which node / which uv   # confirm binary is on PATH
```

### 3.2 — Check the config file
Read the target settings file:
```bash
cat ~/.claude/settings.json
# or: cat <project>/.claude/settings.json
```
Verify:
- `command` executable exists on PATH
- `args` paths resolve to real files/directories (`ls <path>`)
- All required `env` vars are present (not still showing `<KEY>` placeholder)
- JSON is valid (no trailing commas, mismatched brackets)

### 3.3 — Check dependencies
```bash
cd <clone-path>
# Python:
source .venv/bin/activate && python -c "import mcp; print(mcp.__version__)"
# Node.js:
node -e "require('@modelcontextprotocol/sdk')"
# uv:
uv run python -c "import mcp"
```

### 3.4 — Test the server start command directly
Run the exact command from `settings.json` in a terminal:
```bash
<command> <args...>
```
Capture stdout and stderr. A healthy STDIO server waits silently for input.
An error here is the ground truth — it tells you exactly what's wrong.

### 3.5 — Send a test initialize request
```bash
echo '{"jsonrpc":"2.0","id":1,"method":"initialize","params":{"protocolVersion":"2024-11-05","capabilities":{},"clientInfo":{"name":"test","version":"1.0"}}}' \
  | <command> <args...>
```
Expected: a JSON response containing `"result"` and `"protocolVersion"`.
Any other output (error, empty, crash) is the failure point.

### 3.6 — Check Claude Code logs
```bash
# macOS — Claude Code MCP logs:
tail -100 ~/Library/Logs/Claude/mcp*.log 2>/dev/null || \
tail -100 ~/Library/Application\ Support/Claude/logs/mcp*.log 2>/dev/null
```
Look for the exact error message Claude Code is emitting when it tries to connect.

---

## Phase 4 — Present fix plan and get approval

Before changing anything, show the user a clear plan:

```
Error Diagnosis
===============
Category  : <category from Phase 2>
Root cause: <one sentence — what is actually wrong>

Fix Plan (in order)
-------------------
Step 1: <what will be done — e.g. "Install missing mcp package in .venv">
  Command: <exact command>
  Risk   : <low / medium — and why>

Step 2: <next fix if needed>
  Command: <exact command>
  Risk   : <low / medium>

Step 3: Update ~/.claude/settings.json  [if config needs changing]
  Change : <exactly what will change — show before/after diff>

Shall I proceed with Step 1? (yes / skip / cancel)
```

**Never execute a fix without the user saying "yes" for that step.**
If the user says "skip", move to the next step. If "cancel", stop entirely.

For each subsequent step after the first, ask again:
```
Step 1 complete. Proceed with Step 2? (yes / skip / cancel)
```

---

## Phase 5 — Execute fixes (by category)

### Fix: Dependency / Install errors

```bash
cd <clone-path>

# Python — reinstall in venv:
python3 -m venv .venv
source .venv/bin/activate
pip install --upgrade pip
pip install -r requirements.txt     # or: pip install -e .

# uv project:
uv sync                             # re-resolves and installs from uv.lock

# Node.js:
rm -rf node_modules package-lock.json
npm install
npm run build                       # if TypeScript
```

If `command not found: uv`:
```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
source ~/.zshrc                     # or ~/.bashrc
```

If `command not found: node`:
— Direct user to install Node.js from nodejs.org (LTS version).

---

### Fix: Configuration errors

Read `~/.claude/settings.json`. Show the current entry for this server and the corrected version side by side:

```
Current:
  "command": "python",
  "args": ["/wrong/path/server.py"]

Corrected:
  "command": "<clone-path>/.venv/bin/python",
  "args": ["-m", "<module>"]
  or
  "command": "uv",
  "args": ["--directory", "<clone-path>", "run", "<script.py>"]
```

Common config mistakes to fix:
- `command: "python"` should be `"<clone-path>/.venv/bin/python"` (venv not activated by Claude Code)
- Path in `args` contains `~` — expand to full absolute path
- Missing `"env"` block — add it with required vars
- Placeholder `<KEY>` still present — prompt user to fill it (do not fill it yourself)

Read the existing settings.json, merge the corrected entry, write back. Never overwrite other servers.

---

### Fix: Authentication errors

If a `<KEY>` placeholder is still in settings.json:
```
The env var <VAR_NAME> still has its placeholder value.
Open: ~/.claude/settings.json
Find : "<VAR_NAME>": "<KEY>"
Replace <KEY> with your real credential.
Do NOT share the value here — fill it directly in the file.
```

If the key is filled but still getting 401/403:
- Check the key format matches what the README shows (some need `Bearer ` prefix, some do not)
- Check if the key has expired — link to the service's token management page
- Check the correct env var name — compare source code `os.environ.get(...)` with settings.json

---

### Fix: Transport / Process errors

If the process exits immediately after start:
1. Run the start command directly in terminal (Phase 3, Step 3.4) and capture the full output
2. Most common cause: missing env var (server crashes before first request)
3. Second most common: Python/Node binary not found — check `command` path

If `EPIPE` or `broken pipe`:
- Server received a malformed first message — confirm the client is sending valid JSON-RPC
- Upgrade mcp SDK: `uv add mcp --upgrade` or `npm update @modelcontextprotocol/sdk`

---

### Fix: Protocol Version errors

Check the mcp SDK version in the clone:
```bash
cd <clone-path>
# Python:
uv run python -c "import mcp; print(mcp.__version__)"
# Node.js:
node -e "const p=require('./node_modules/@modelcontextprotocol/sdk/package.json'); console.log(p.version)"
```

If outdated:
```bash
# Python/uv:
uv add "mcp>=1.15"
# Node.js:
npm install @modelcontextprotocol/sdk@latest
npm run build
```

---

### Fix: Runtime / Crash errors

1. Read the full stack trace — identify the file and line number
2. If it's in the server's own code: check for a missing env var the server reads at startup
3. If it's in the mcp library: likely a version incompatibility — upgrade the SDK
4. If `AttributeError` on a tool function: check the function signature matches what `@mcp.tool()` expects

---

### Fix: Environment / Path errors

```bash
# Wrong Python version — check pyproject.toml for requires-python:
python3 --version          # compare to README requirement

# If version mismatch — use pyenv or uv's Python management:
uv python install 3.13     # install required version
uv sync                    # re-sync with correct Python

# Permission denied:
chmod +x <script>
# or: check if the clone directory is on a read-only volume
```

---

## Phase 6 — Verify the connection

After fixes are applied, run the full verification sequence:

```bash
# 1. Confirm server starts without errors:
<command> <args> &
sleep 1 && kill %1       # start briefly, check no immediate crash

# 2. Send initialize:
echo '{"jsonrpc":"2.0","id":1,"method":"initialize","params":{"protocolVersion":"2024-11-05","capabilities":{},"clientInfo":{"name":"test","version":"1.0"}}}' \
  | <command> <args>
```

Expected healthy response:
```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "result": {
    "protocolVersion": "2024-11-05",
    "capabilities": { "tools": {} },
    "serverInfo": { "name": "...", "version": "..." }
  }
}
```

If response is valid:
```
Connection verified.
Restart Claude Code — the MCP server will load automatically.
```

If still failing — go back to Phase 3 with the new error output.

---

## Phase 7 — Report to user

```
Error Resolution Summary
========================
Server     : <server name>
Error was  : <original error — one line>
Root cause : <what was actually wrong>
Fix applied: <what was changed — e.g. "Updated command path in settings.json">
Status     : Connected / Still failing

Config location: <path to settings.json>
```

---

## Skill 2.0 — Continuous Self-Upgrade Protocol

After every successful fix, append the new pattern to `references/learned-fixes.md`.
This is what makes the skill smarter over time — every error fixed becomes a known pattern
that future runs can match instantly instead of re-diagnosing from scratch.

**When to write a new entry:**
- A fix worked and the connection was verified
- The error was not already in `references/error-patterns.md`
- The pattern is specific enough to be useful (not just "reinstall everything")

**How to write the entry** — open `references/learned-fixes.md` and append:

```markdown
## [Short error title — e.g. "uv not found on macOS after install"]

**Error signals:**
- Exact error message or key substring
- Where it appears (terminal / Claude Code / install step)

**Root cause:**
One sentence explanation of why this happens.

**Fix:**
```bash
exact commands that fixed it
```

**Verification:**
What output confirmed the fix worked.

**First seen:** YYYY-MM-DD  |  **Server:** <server name>
```

Then update `references/error-patterns.md` signal table to include the new pattern so Phase 2
classification picks it up next time.

Read `references/learned-fixes.md` at the start of every session — a match there means
you can skip Phases 2–3 and go straight to Phase 4 with the proven fix.

---

## Common Mistakes to Avoid

- Skipping the `references/learned-fixes.md` pre-check — always read it first; a known
  fix skips Phases 2–3 entirely and saves re-diagnosing from scratch
- Classifying an error in only one category when multiple apply — list all and fix in
  order (Dependency → Config → Auth)
- Running a fix step without the user's explicit "yes" — every individual step requires
  approval before execution
- Treating a transport error as the root cause — ECONNREFUSED and STDIO exit errors are
  almost always symptoms of a deeper dependency, config, or auth failure; always run
  Phase 3 diagnosis first
- Forgetting to run the JSON-RPC initialize test after applying fixes — Phase 6
  verification is mandatory even if the server appears to start without errors
- Writing a new entry to `learned-fixes.md` for a pattern already in `error-patterns.md`
  — only append genuinely new patterns not yet captured
- Asking the user to paste a credential into the chat during auth error diagnosis —
  always direct them to edit the settings file directly (see SECURITY MANDATE above)
- Stopping at Phase 4 without approval — never execute any command without explicit
  user confirmation for that specific step
