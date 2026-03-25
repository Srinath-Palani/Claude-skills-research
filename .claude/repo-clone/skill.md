---
name: repo-clone
user-invocable: true
description: >
  Clone a GitHub-hosted MCP server repository locally, install all dependencies, configure
  environment variables, and establish the MCP server connection. Use this skill whenever a
  user provides a GitHub URL for an MCP server and wants to run it locally — even if they
  just say "set up this MCP server", "clone and connect this MCP repo", "get this MCP server
  running", "install this MCP server from GitHub", or "connect to this MCP server locally".
  Always trigger when a GitHub URL is combined with any intent to run, install, connect, clone,
  or set up an MCP server. The default clone location is /Users/srinathp/MCP_repos/<repo-name>
  unless the user specifies otherwise.
---

# GitHub MCP Server Clone + Connect

> Full workflow diagram: see `workflow.md` in this directory.
> Update `workflow.md` whenever this skill is modified.

This is the single authoritative skill for cloning and installing local MCP servers. It is
triggered in two ways:
1. **Directly** — when the user provides a GitHub URL and wants to clone and connect an MCP server
2. **From the `mcp-server` skill** — when research confirms `Deployment: Local = Yes`; `mcp-server`
   hands off here after reading the README

When triggered from `mcp-server`, use the README content already fetched — skip the fetch in
Step 1 and proceed directly to Step 2 with the information already in context.

**After cloning and installing** (Steps 1–5), this skill hands back to the **`mcp-server` skill**
to analyse the local clone, extract `command / args / env`, and run the full authentication and
Claude Code config setup. `repo-clone` does not collect credentials or write config files itself.

Clone a GitHub MCP server repo, install its dependencies, configure it, and establish
a working connection — all with explicit user approval before taking any action.

The goal is zero surprises: show the user exactly what will happen, get their OK, then
execute each step cleanly.

---

## Usage

Clone and run any GitHub-hosted MCP server locally:

```
"Set up this MCP server: https://github.com/org/repo"
"Clone and connect this MCP repo"
"Install this MCP server from GitHub"
"Get this MCP server running locally: https://github.com/org/repo"
"Clone https://github.com/org/mcp-server to /Users/me/mcp"
```

Clones to `/Users/srinathp/MCP_repos/<repo-name>` by default unless the user specifies
a different path.

---

## Step 1 — Fetch and read the repository docs

Given the GitHub URL, fetch and read the following files (if they exist):
- `README.md` (or `README.rst`) — primary source for setup instructions
- Any linked setup/installation docs referenced from the README (e.g., `INSTALL.md`, `docs/setup.md`)
- `requirements.txt`, `pyproject.toml`, `setup.py`, `package.json`, or `Cargo.toml` — to enumerate exact dependencies
- `.env.example` or any config template files — to identify required environment variables

Do NOT skip or summarise. Read every section: prerequisites, installation, environment
variables, configuration, and running instructions. You need this to give the user a
complete picture before they approve anything.

---

## Step 2 — Determine the connection method

MCP servers connect in different ways. Read the README carefully and identify which applies:

- **STDIO (local subprocess)**: Server runs as a local process via stdin/stdout. Most common for
  open-source MCP servers. Requires cloning + installing deps locally.
- **Published package (no clone needed)**: Server is on PyPI/npm and runs via `pip install` +
  `python -m <module>`, or `npx <package>`. No clone required.
- **Docker/Container**: Server ships a `Dockerfile` and runs via `docker run`.
- **Remote endpoint**: Server is hosted by a vendor and accessed via a URL (no local setup needed).

Pick the method described in the README. If multiple are listed, pick the one marked as
"recommended" or the simplest one that does not require extra infrastructure. Explain your
choice briefly to the user.

---

## Step 3 — Present the plan and get explicit approval

Before cloning or installing anything, show the user this summary and wait for their "yes":

```
MCP Server Setup Plan
=====================
Repository:     <GitHub URL>
Server type:    <STDIO / Docker / Published package / Remote>
Connection:     <how it connects — e.g., "STDIO subprocess via python -m <module>">

Clone path:     /Users/srinathp/MCP_repos/<repo-name>
                (type a different path if you want to change this)

Dependencies to install:
  - <list each package/tool with version if pinned>
  - <e.g., "Python 3.11+", "mcp>=1.5", "httpx", "nodejs 18+">

Environment variables required:
  - <VAR_NAME>: <what it's for — e.g., "API_KEY: your service API key">
  - (none) if no env vars are needed

Steps I will take (in order):
  1. Clone the repo to <clone-path>
  2. <create venv / npm install / docker build — whichever applies>
  3. Install dependencies
  4. Create .env file with your provided values
  5. Verify the server starts correctly

Proceed? (yes / no, or type a different clone path)
```

Wait for the user's response. Do not proceed until you receive explicit approval.
If the user provides a different path, use that path throughout.

---

## Step 4 — Clone the repository

Use HTTPS cloning (not SSH) unless the user explicitly requests SSH:

```bash
git clone https://github.com/<org>/<repo>.git <clone-path>
```

- Use the full clone (no `--depth 1`) unless the README explicitly instructs shallow cloning.
- After cloning, verify the directory exists and contains the expected files (README, dependency
  files, source code).
- Report: "Cloned to <clone-path>" or the error if it failed.

---

## Step 5 — Install dependencies

Follow the README exactly. The most common patterns:

**Python (venv + pip):**
```bash
cd <clone-path>
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
# or if pyproject.toml: pip install -e .
# or if setup.py: pip install .
```

**Node.js / TypeScript:**
```bash
cd <clone-path>
npm install          # or: pnpm install / yarn install — per README
npm run build        # only if README specifies a build step
```

**Docker / Container:**
```bash
cd <clone-path>
docker build -t <repo-name> .
```

**Published package (no clone needed):**
```bash
pip install <package-name>
# or: npx <package-name> (no install step needed)
```

After installation, confirm success — check that key packages are importable or the
build artifacts exist as expected.

---

## Step 6 — Hand off to mcp-server for analysis and authentication

Once the repo is cloned and dependencies are installed, hand off to the **`mcp-server` skill**
to analyse the local clone and handle all authentication and config setup. Do not collect
credentials or write config files yourself — `mcp-server` owns that workflow.

### What to extract before handing off

Read these files from the **local cloned directory** (not from GitHub — use disk):

| File | What to extract |
|------|----------------|
| `README.md` | Start command, args, env var names, auth instructions |
| `package.json` / `pyproject.toml` | Exact start script, entry point, binary name |
| `.env.example` | Every required and optional env var with its description |
| `uv.lock` / `package-lock.json` | Pinned dependency versions (especially `mcp` package) |
| Main script (`index.js`, `server.py`, `opentip.py`, etc.) | `@mcp.tool()` decorators, env reads (`os.environ`, `process.env`), HTTP method calls |

From these, determine:
- **`command`** — the executable to run (e.g. `uv`, `node`, `python3`, `docker`)
- **`args`** — the argument list (e.g. `["--directory", "<clone-path>", "run", "opentip.py"]`)
- **`env`** — every environment variable name the server reads, and its auth type

### Pass to mcp-server

Trigger the `mcp-server` skill's **Authentication and Environment Setup** section with:
- The clone path
- The extracted `command`, `args`, and `env` var names
- The README content already read (so `mcp-server` does not need to re-fetch from GitHub)

Tell `mcp-server` explicitly: "This server has been cloned locally — read files from disk at
`<clone-path>`, not from GitHub."

`mcp-server` will then:
1. Detect the auth type from the env var names and source code
2. Ask the user which Claude Code config path to use (`~/.claude/settings.json` vs project-level)
3. Write the config with `<KEY>` placeholders
4. Guide the user to open the file and fill in real credentials directly — never in chat
5. Apply all security rules (no secrets in chat, `.gitignore` reminder for project configs)

---

## Step 7 — Verify connection after auth setup

Once `mcp-server` has written the config and the user has filled in credentials, verify the
server starts correctly using the command and args that were determined.

**STDIO servers** — send a test `initialize` request:
```bash
echo '{"jsonrpc":"2.0","method":"initialize","params":{"protocolVersion":"2024-11-05","capabilities":{}},"id":1}' | <command> <args>
```
Look for a valid JSON response. If the server errors out, hand off to the
**`error-handling` skill** with the full error output — do not attempt to diagnose
the error yourself.

**Docker:**
```bash
docker run --rm -i --env-file .env <repo-name> --help
```

**Published package:**
```bash
npx <package-name> --version
# or: python -m <module> --help
```

---

## Step 8 — Final summary

Report to the user:

```
Setup Complete
==============
Clone path   :  <clone-path>
Command      :  <command>
Args         :  <args>
Config file  :  <path written by mcp-server, e.g. ~/.claude/settings.json>
Status       :  Running / Failed

Next steps:
  1. Open <config-file-path> and replace any remaining <KEY> placeholders with real values
  2. Restart Claude Code — the MCP server will load automatically
```

If the server **failed to start**, report:
- The exact error output
- The most likely cause (missing credential, wrong Python version, missing dependency)
- The specific fix the user should try

---

## Common setup mistakes to watch for

- **Wrong Python version**: Check `python3 --version` against the README's requirement.
  If mismatched, tell the user before installing deps.
- **Missing system dependencies**: Some servers need `brew install <tool>` or `apt install <lib>`
  before pip/npm install. Look for these in the README prerequisites section.
- **Build tools missing**: TypeScript servers need `npm run build` after `npm install`.
  If you see `.ts` source files but no `dist/` folder, a build step is needed.
- **Wrong working directory**: Always `cd` into the cloned repo before running install commands.
- **Virtual env not activated**: For Python servers, the venv must be active when running the server
  (not just during install). The start command should reference the venv's Python explicitly:
  `<clone-path>/.venv/bin/python -m <module>`.
- **STDIO vs persistent process**: STDIO MCP servers are NOT run as background services.
  They are launched on-demand by the MCP client. The "start command" goes in the client config,
  not into a `systemctl` or `pm2` setup.
