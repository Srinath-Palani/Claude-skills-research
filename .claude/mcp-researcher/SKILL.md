---
name: mcp-server
user-invocable: true
description: >
  Research, verify, document, and security-score every attribute of an MCP (Model Context Protocol)
  server. Use this skill whenever a user provides an MCP server name, URL, or GitHub repo and asks
  to research it, document its capabilities, fill in its attributes, catalog it, analyze it, score
  it, or produce a structured profile of it. Trigger even if the user just says "look up this MCP
  server", "what does this MCP server support?", or "get me a report on X MCP server" — the goal
  is a fully evidenced, structured document with every field filled in plus a security confidence
  score. The report is always saved to ~/Desktop/MCP_reports/. Use this skill for any MCP server
  investigation, full or partial.
---

# MCP Server Researcher + Security Scorer

> Full workflow diagram: see `workflow.md` in this directory.
> Update `workflow.md` whenever this skill is modified.

Research and document every attribute of an MCP server, then calculate its security confidence
score. Never assume — always verify from the priority sources. When in doubt, mark No.

---

## SECURITY MANDATE — Always enforced, no exceptions

These rules apply at every step of this skill, before anything else:

1. **Never ask the user to type, paste, or share any secret, API key, token, password, or credential in the chat.** This is an absolute rule — no exceptions for any reason.
2. **Always use `<PLACEHOLDER>` values in every config file written.** Never pre-fill real credentials.
3. **Always direct the user to open the config file in their own editor** and replace placeholders themselves — credentials must only ever transit through the user's local filesystem, never through chat.
4. **If the user accidentally pastes a credential into the chat**, immediately respond with:
   ```
   Security alert: Please do not share credentials here. Treat this value as potentially
   compromised — revoke it and generate a new one, then add the new one directly into
   the settings file at <path>. Do NOT paste it here.
   ```
5. **Never store credentials in any file that may be committed to git** (no keys in README, no keys in the CSV report, no plain-text `.env` committed). If writing a project-level config (`.claude/settings.json`), always remind the user to add it to `.gitignore`.
6. **Always follow the Security rules section** in the Authentication and Environment Setup section below — that section elaborates these rules but never overrides them.

---

## Usage

Research, document, and score any MCP server:

```
"Research the GitHub MCP server"
"Get a full report on the Slack MCP server"
"Document this MCP server: https://github.com/org/repo"
"Score this MCP server for enterprise use"
"Look up the Snowflake MCP server and give me a security score"
"What does the Linear MCP server support?"
```

The report is always saved to `~/Desktop/MCP_reports/<servername>.csv`.

---

Save the completed report as a **CSV only** to `~/Desktop/MCP_reports/`. Create the folder if it does not exist:
- CSV: `<servername>.csv`

Do **not** create a markdown `.md` file. The CSV is the single output file.
The CSV must use the three-column format: `Category, Attribute, Status` — matching the structure described in the Output Format section below.

---

## Source Priority Order

Use sources in this order (highest trust first):

1. **Official vendor documentation page** — the vendor's own docs site
2. **GitHub / source code repository** — for open-source servers
3. **PyPI / npm package page** — for packaged servers
4. **MCP spec documentation** — spec.modelcontextprotocol.io

Every "Yes" and every score must be backed by evidence from one of these sources. Record the
source and exact evidence (quote or file path) for every decision.

---

## Verification Steps

Follow these steps in order:

**Step 0 — Input Classification (always run first)**

Inspect the input before doing any research. Classify it as one of:

- **Remote Endpoint URL** — starts with `https://`, path ends with `/sse`, `/mcp`, `/mcp/`, or similar; subdomain pattern `mcp.vendor.com` or `*.mcp.vendor.com`; is NOT a `github.com` URL.
  Examples: `https://mcp.notion.com/sse`, `https://mcp.linear.app/sse`, `https://mcp.vercel.com`

  Pre-set these attributes immediately (no source research needed for them):
  - Transport: `HTTP/SSE = Yes` if path ends with `/sse`; `StreamableHttp = Yes` if path ends with `/mcp` or `/mcp/`
  - Deployment: `Remote = Yes`, `Local = No`, `Container = No`
  - Hosting Provider: `SaaS Vendor = Yes` (or `3rd Party SaaS vendor = Yes` if not the original vendor)
  - Data Protection: Run TLS verification (see TLS section below) and mark each version Yes/No based on actual connection result.

  Then continue Steps 1–13 for all remaining attributes (research vendor docs, enumerate tools, etc.).
  At Step 14: use the **Remote Endpoint Auth and Connection** section — skip `Authentication and Environment Setup`.

- **GitHub URL** (`github.com/...`) or **plain server name** — prioritize finding remote endpoint; if none exists, ask for GitHub repo.

  **Sub-step: Search for Remote Endpoint (ALWAYS DO THIS FIRST)**

  Before treating this as STDIO, proactively search for a vendor-hosted remote endpoint:
  1. Check if the input is a server name or GitHub URL
  2. Use GitHub API to search the repo for keywords: `remote endpoint`, `hosted endpoint`, `https://mcp`, `https://[vendor-name].mcp`, `.env.example`, `README`
  3. If the repo README links to vendor docs, fetch those docs and search for endpoint URLs
  4. Look for patterns: `https://mcp.vendor.com`, `https://mcp-*.vendor.com`, `https://[server-name].vendor.com/sse`, `https://[server-name].vendor.com/mcp`
  5. Check for `--endpoint` or `--url` command-line parameters in the README (indicates remote endpoint support)

  **If a remote endpoint URL is discovered**:

  Show the user exactly:
  ```
  Remote endpoint found for this server:
    <discovered-endpoint-url>

  You can research this server two ways:
    1. Remote endpoint  — research via vendor-hosted URL (TLS verified, SaaS Vendor)
    2. GitHub / STDIO   — research local deployment from GitHub

  Which would you prefer?
  Enter 1 or 2:
  ```

  - **Answer 1 (Remote endpoint):** Switch to Remote path. Pre-set Remote Endpoint transport/deployment/TLS/hosting attributes (Remote = Yes, HTTP/SSE or StreamableHttp, SaaS Vendor, TLS verification). **Do NOT override STDIO/Local/GitHub pre-sets** — keep them (dual-deployment). At Step 14, use **Remote Endpoint Auth and Connection**.
  - **Answer 2 (GitHub / STDIO):** Continue standard STDIO research flow, Steps 1–15.

  **If no remote endpoint is found**:

  If only a server name was provided (not a GitHub URL):
  ```
  I couldn't find a remote endpoint or GitHub repository for "<server-name>".

  To proceed with research, please provide one of:
    1. GitHub repository URL (e.g., https://github.com/org/repo)
    2. Remote MCP endpoint URL (e.g., https://mcp.vendor.com/sse)
    3. Vendor documentation URL

  Enter the URL:
  ```

  If a GitHub URL was provided but no remote endpoint exists in the repo:
  Continue standard STDIO research flow, Steps 1–15.

  **Pre-set these attributes** (permanent for GitHub input):
  - Transport: `STDIO = Yes`
  - Deployment: `Local = Yes`
  - Hosting Provider: `GitHub = Yes`

  ⚠ These three pre-sets are **permanent** — they are NOT overridden if the user later chooses the Remote endpoint path. A server that lives on GitHub is always STDIO-capable and locally deployable, regardless of whether a remote endpoint also exists.

---

**Step 0.5 — Remote Endpoint Discovery (for GitHub/server-name inputs)**

If a GitHub URL or server name was provided AND no remote endpoint was found in Step 0, run this proactive search:

1. **Search the GitHub repository for endpoint references**:
   - Keywords: `remote endpoint`, `hosted endpoint`, `cloud-hosted`, `https://mcp`, `endpoint url`, `host:` with `https://`
   - Files to check: `README.md`, `.env.example`, `config.json`, `setup.md`, any linked vendor docs

2. **Extract endpoint pattern from README**:
   - If README shows `--endpoint` or `--url` parameters → indicates remote endpoint support exists
   - If README links to vendor docs → fetch those docs and search for endpoint URLs

3. **Construct likely endpoint URL from server name**:
   - Pattern 1: `https://mcp.[vendor].com/sse` or `/mcp`
   - Pattern 2: `https://mcp-[servername].[vendor].com/sse`
   - Pattern 3: `https://[servername].mcp.[vendor].com/sse`
   - Try curl to verify if endpoint exists (even if returns 401, endpoint exists)

4. **If endpoint found**: Return to Step 0 user choice prompt with discovered URL
5. **If no endpoint found**: Proceed with STDIO research (Steps 1–15)

**Do not skip this step for GitHub inputs** — many servers have remote endpoints not prominently featured in README.

---

1. Find the official vendor documentation page for the MCP server.
2. Find the GitHub (or other) source code repository.
3. Read the README and any deployment/authentication docs.
4. Read the source code tool registration files to enumerate all tools.
5. Trigger the **`attribute-researcher` skill** to fill in all attribute fields using the
   sources read in Steps 1–4. Pass the vendor docs URL, GitHub URL, and any source content
   already fetched so attribute-researcher does not re-fetch what is already in context.
   Wait for attribute-researcher to return all attribute values with evidence before continuing.
6. For each Yes/No decision, record: the value, the source, and exact evidence (quote or file path).
7. Cross-check every field: does the vendor doc confirm what you marked? If not — correct it.
8. Apply all field rules below. Watch for the common mistakes listed at the end.
9. Write the Capabilities Tools and Non-Read-Only Tools detailed_info sections from source code.
10. Fetch the current release version from Tags/Releases only:
    1. Navigate to the GitHub repository's **Releases** page (`/releases`). If no releases exist, try the **Tags** page (`/tags`).
    2. List ALL tags and releases found — record every version label exactly as written, most recent first.
    3. Use the most recent entry as the referred version.
    4. If neither page has any entries → write `Not published`

    Format the Status field as:
    `v<latest_version> [ Referred from Tags/Releases ] | All versions: <v1>, <v2>, <v3>, ...`

    Always prefix with `v` regardless of how the version label is stored in the source.
    List ALL versions found, most recent first, comma-separated after the `|` separator.
    Examples:
    - `v1.2.0 [ Referred from Tags/Releases ] | All versions: v1.2.0, v1.1.0, v1.0.0, v0.9.0`
    - `v0.1.2 [ Referred from Tags/Releases ] | All versions: v0.1.2`
    - `Not published`

    Do NOT fall back to `package.json`, `pyproject.toml`, or `setup.py` — Tags/Releases is the only source.
11. Write a 3–4 line description of the MCP server. **Must include specific capabilities and features** — avoid generic descriptions. Required elements:
    - What it does (primary function)
    - What service/system/networks it connects to (be specific: name services, list supported chains, etc.)
    - Primary use case (e.g. querying, transactions, analysis)
    - Key capabilities or tool categories (e.g. 55+ tools, balance lookups, contract reads, etc.)

    Example (good): "Pocket Network MCP server for querying blockchain data across 63+ networks (EVM, Solana, Cosmos, Sui) via natural language. Provides 55+ tools for balance lookups, smart contract reads, transaction history, token metadata, multi-chain analysis, and domain resolution (ENS/Unstoppable Domains) without authentication."

    Example (bad): "Accesses blockchain data via Pocket Network." ← Too generic, user will request expansion.

    **Source**: README and vendor docs. Never invent capabilities.
12. Determine the server's category by analysing its tools, README, and primary use case. Assign exactly one of these four categories:
    - **File and Document Management** — primary purpose is creating, reading, editing, or organising files, documents, or storage (e.g., Google Drive, Dropbox, filesystem, PDF tools)
    - **Developer and Coding Tools** — primary purpose is supporting software development workflows (e.g., GitHub, GitLab, CI/CD, code search, IDEs, package managers)
    - **Data and Information Retrieval** — primary purpose is querying, searching, or extracting structured or unstructured data (e.g., databases, search engines, APIs, web scraping, analytics)
    - **Productivity and Communication** — primary purpose is task management, messaging, scheduling, or team collaboration (e.g., Slack, Notion, Jira, email, calendar)
13. **Fill in GitHub Repository and Endpoint URL**:
    - **GitHub Repository**: Populate with the source repo URL if discovered during Step 0.5 or from initial GitHub input. Write `N/A` if not applicable (pure remote vendor-hosted, no GitHub).
    - **Endpoint URL**: Populate with the remote endpoint URL if researching a remote server. Write `N/A` for STDIO-only local servers.
    - These fields are mandatory and help users quickly identify where to find source code and how to connect.

14. Final review: every Yes has evidence; every No is justified; version, description, category, GitHub repo, and endpoint URL are all filled in.

15. Proceed to the connection setup section that matches the input type (from Step 0):
    - **Remote Endpoint URL** → use the **Remote Endpoint Auth and Connection** section below.
    - **GitHub URL / server name (STDIO)** → use the **Authentication and Environment Setup** section below.
    Apply the SECURITY MANDATE throughout — never ask for credentials in chat, always use placeholders, always direct the user to edit the file themselves.

16. After connection setup is complete, trigger the **CCI MCP-server Scoring** section. The CSV is saved there as the final step — with or without score rows depending on the user's answer.

---

## Field Rules

### Yes/No Discipline

Only two values are allowed: **Yes** or **No**.
**NEVER use**: Partial, Unknown, N/A, "Not confirmed", or any other variant.
If you cannot confirm from the priority sources, mark **No**.

**Common error — TLS for STDIO servers**: Local STDIO servers have no network transport, so TLS 1.3/1.2/Lower = **No** (not N/A).
This applies to ALL local-only servers regardless of whether they internally call HTTPS endpoints.

---

### MCP Info Fields

**GitHub Repository** — Always populate if the server has a GitHub source repository:
- **Remote endpoints only**: Populate with the GitHub URL if source code exists on GitHub (e.g., `https://github.com/awslabs/mcp`)
- **Local STDIO servers**: Populate with the GitHub repository URL where the source is hosted
- **No GitHub**: Write `N/A` if the server is not distributed via GitHub
- **Example**: `https://github.com/awslabs/mcp/tree/main/src/aurora-dsql-mcp-server`
- **Source**: GitHub search results, vendor docs, or repository URL discovery during Step 0.5

**Endpoint URL** — Always populate for remote servers; mark N/A for STDIO-only servers:
- **Remote endpoints**: Write the full HTTPS endpoint URL (e.g., `https://xmfe3hc3pk.execute-api.us-east-2.amazonaws.com`)
- **STDIO-only servers (no remote)**: Write `N/A` (local deployment, no URL)
- **Dual-deployment servers**: Write the primary remote endpoint URL (the one used for this research)
- **Important**: This is the URL users connect to via Claude Code's `"url"` config key, not the `--cluster_endpoint` or other user-provided parameters
- **Source**: Discovered in Step 0.5 endpoint discovery or provided by user as input in Step 0

---

### Distribution Type — Official vs Community

- **Official** = developed and maintained by the original service provider (e.g. Snowflake-Labs for Snowflake, vercel for Vercel).
- **Community** = third-party developer or open-source contributor who is not the original service provider.
- Mutually exclusive — only one can be Yes.

Check the GitHub organisation or company that owns the repository.

---

### MCP Protocol Version

Mark **only the single latest** version the server supports. Do not mark multiple as Yes.

**Step 1 — Fetch the pinned SDK version AND the latest from the live registry:**

**Always do both sub-steps — do not stop after reading the lock file.**

**Sub-step A — Read the pinned version from the repo:**
- Python/uv projects: check `uv.lock` for the `mcp` package entry (`name = "mcp"` → `version = "x.y.z"`)
- Python/pip projects: check `requirements.txt` for `mcp==x.y.z`
- Node.js projects: check `package-lock.json` or `yarn.lock` for `@modelcontextprotocol/sdk`
- If no lock file exists, read the minimum constraint from `pyproject.toml`/`package.json`

**Sub-step B — Fetch the latest version from the live registry:**
- Python: `curl -s https://pypi.org/pypi/mcp/json` → read `info.version`
- Node.js: `curl -s https://registry.npmjs.org/@modelcontextprotocol/sdk/latest` → read `version`

**Sub-step C — Upgrade if the pinned version is behind the latest:**
- Compare the pinned version (Sub-step A) with the latest (Sub-step B).
- If the pinned version is **older than the latest**, upgrade it:
  ```bash
  # Python/uv:
  uv add "mcp[cli]>={latest_version}" --directory <clone-path>
  # Node.js:
  npm install @modelcontextprotocol/sdk@latest
  ```
- After upgrading, verify the server still starts correctly (run the JSON-RPC initialize test).
- Use the **upgraded version** for all protocol version mapping below.
- If the repo is not locally cloned, use the latest registry version for mapping.

**Step 2 — Map the SDK version to a protocol revision:**

- Check vendor docs or FastMCP/SDK changelog for the exact spec revision string.
- If using FastMCP, check the fastmcp release notes for which spec revision it implements.
- Valid versions: `2025-11-25` / `2025-06-18` / `2025-03-26` / `2024-11-05`. All others = No.

**TypeScript/Node.js SDK version mapping** (`@modelcontextprotocol/sdk`):
- SDK >= 1.12.x → Protocol `2025-11-25`
- SDK ~1.8.x–1.11.x → Protocol `2025-06-18`
- SDK ~1.5.x–1.7.x → Protocol `2025-03-26`
- SDK < 1.5.x → Protocol `2024-11-05`

**Python SDK version mapping** (`mcp`):
- SDK >= 1.24.x → Protocol `2025-11-25`
- SDK ~1.15.x–1.23.x → Protocol `2025-06-18`
- SDK ~1.5.x–1.14.x → Protocol `2025-03-26`
- SDK < 1.5.x → Protocol `2024-11-05`

**Step 3 — Record in the Evidence Log:**
- Log the exact declared SDK version, the live registry version fetched, the source file (e.g. `package.json` line), and the resulting protocol revision mapped.

---

### Pricing — Free vs Paid

- **Free** = Yes if the MCP server can be used without payment via ANY connection method (remote endpoint, STDIO, container): open-source repos, free tier, or any free trial (even time-limited, e.g. 14-day trial). A free trial counts as Free = Yes even if payment details are collected at signup.
- **Paid** = Yes ONLY if ALL connection methods require entering payment details to sign up AND there is absolutely no free trial, no free tier, and no free-access path whatsoever. If any free or trial access exists for any method — even behind a credit-card wall — Paid = No.
- Note: underlying platform costs (e.g. Snowflake credits, AWS usage) are separate from MCP server pricing.

---

### Hosting Provider

Mark the primary platform where the server's source code or endpoint is hosted. **Mutually exclusive — only ONE should be marked Yes.**

- **SaaS Vendor** = Yes if the original vendor hosts and manages a remote MCP endpoint at their own domain (e.g. a cloud URL like `https://mcp.vendor.com`) and primary distribution is vendor-hosted (not GitHub).
- **GitHub** = Yes if the primary distribution is via a GitHub repository (including STDIO servers cloned from GitHub for local deployment). GitHub presence alone (docs/reference only) does NOT count.
- **GitLab** = Yes if the source repository is on GitLab.
- **Bitbucket** = Yes if the source repository is on Bitbucket.
- **SourceHut/Gitea/Gogs** = Yes if the source repository is on one of these platforms.
- **3rd Party SaaS vendor** = Yes if a third-party (not the original service vendor) hosts the MCP server as a cloud service.

**Important Rules:**
- Mark only ONE hosting provider as Yes (primary deployment/distribution method).
- If a server has BOTH a vendor remote endpoint AND a GitHub repo for local proxy: Mark SaaS Vendor = Yes, GitHub = No (vendor infrastructure is primary).
- Example: Vendor endpoint `https://mcp.vendor.com` with GitHub local proxy → SaaS Vendor = Yes, GitHub = No
- If distributed only on GitHub for local STDIO deployment → GitHub = Yes, SaaS Vendor = No

Do **not** add a "Self-hosted" row — instead mark the actual source platform (GitHub, GitLab, etc.).

---

### Authentication

Check vendor docs and source code for each method. Report all detected methods; score using the
highest-security method found.

- **OAuth 2.1 Authorization Code Flow** = Yes if vendor docs mention OAuth or browser-based login flow; look for `/authorize` + `/callback` routes, PKCE (`code_challenge`, `code_verifier`).
- **OAuth 2.1 Client Credentials Flow** = Yes if M2M/service account OAuth is supported; look for `client_credentials` grant type.
- **Bearer Token** = Yes if `Authorization: Bearer <token>` headers are used with token validation.
- **Personal Access Token** = Yes if vendor docs or source code accept PAT (`GITHUB_TOKEN`, `glpat_`, etc.).
- **API Token** = Yes if static API key or password-based auth is supported (`API_KEY`, `X-API-Key`, etc.).

Sources: vendor auth docs, source code connection/auth modules, environment variable names.

**Dual-deployment servers:** Research both connection paths and mark every auth method found from either:
- Remote endpoint: probe the endpoint (Step 1 of Remote Endpoint Auth) for Bearer, OAuth, API key
- Local proxy source code: scan env var names and connection modules

---

### Data Protection — TLS

**⚠️ Critical: TLS applies ONLY to the MCP transport layer, not to underlying database/API connections**

Apply this rule strictly based on deployment type:

- **Remote** (vendor-hosted HTTPS endpoint): **ALWAYS verify** TLS support by running both commands against the endpoint. Mark each version Yes ONLY if the connection succeeds.

  **Quick Method (curl):**
  ```bash
  curl -v --tlsv1.2 https://your-endpoint
  curl -v --tlsv1.3 https://your-endpoint
  ```

  **Advanced Method (openssl - more reliable):**
  ```bash
  openssl s_client -connect host:443 -tls1_2
  openssl s_client -connect host:443 -tls1_3
  ```

  If connection succeeds (shows certificate chain) → mark that version **Yes**. If connection fails → mark **No**.
  Examples: If both succeed → `TLS 1.3 = Yes`, `TLS 1.2 = Yes`, `Lower versions = No`.

  **Important**: For **dual-deployment servers** (Remote + Local STDIO):
  - **Remote connection**: Verify TLS versions (use curl as above)
  - **Local STDIO connection**: Mark TLS as No (local transport has no TLS)
  - Report the TLS values that apply to the Remote endpoint only

- **Local only** (STDIO subprocess, no network): TLS 1.3 = No, TLS 1.2 = No (no network encryption).
  - Note: A STDIO server may establish TLS-encrypted connections to upstream services (databases, APIs) internally, but this is an **implementation detail**, not part of the MCP transport layer. Do NOT mark TLS = Yes based on upstream connections.
  - Example: An MCP server that runs locally and connects to a remote PostgreSQL database with `sslmode=require` still has TLS 1.3=No, TLS 1.2=No because the MCP transport itself (STDIO) has no encryption.

- **Container** (self-hosted local Docker): TLS = No unless vendor docs explicitly mention TLS for the MCP transport layer itself. If docs mention TLS, run the verification commands above.
- Lower versions = Yes only if vendor docs confirm unencrypted or legacy TLS access is allowed.

---

### Transport Protocol

- **STDIO** = Yes if the server runs as a local subprocess via stdin/stdout. No hosted URL exists.
- **HTTP/SSE** = Yes if the server exposes a legacy Server-Sent Events endpoint (spec 2024-11-05).
- **StreamableHttp** = Yes if the server exposes an HTTP endpoint using StreamableHttp transport (spec 2025-03-26 or later).
- For **dual-deployment servers** (GitHub-input with discovered remote endpoint): mark ALL applicable transports — STDIO=Yes AND (HTTP/SSE=Yes or StreamableHttp=Yes). These are NOT mutually exclusive when both deployment modes exist.
- For STDIO-only servers (no remote endpoint): HTTP/SSE=No, StreamableHttp=No.

---

### Tools Operations

Evaluate all tools and mark **only the highest** applicable operation tier as Yes:

- **Read-only** = Yes (others No) — ALL tools are read-only (list, get, search, describe).
- **Read-only and/or update** = Yes (others No) — tools include read AND write/create/update but NO delete.
- **Read-only, update and/or delete** = Yes (others No) — tools include read AND write/update AND delete.

Only one row = Yes. The other two must be No.

---

### Deployment Approach

- **Local** = Yes if the server runs as a local process on the user machine (STDIO, no network endpoint).
- **Container** = Yes if a Dockerfile or docker-compose is provided AND the user runs it themselves.
- **Remote** = Yes if the server is hosted remotely by the vendor and accessed via a URL endpoint.
- Not mutually exclusive — Local + Container can both be Yes for self-hosted servers.
- If Remote = Yes (vendor managed) AND input was NOT a GitHub URL: Local = No, Container = No.
- If Remote = Yes AND input was a GitHub URL (dual-deployment): Local = Yes and STDIO = Yes remain. Container = Yes if Dockerfile or docker-compose exists in the repo.

---

### Capabilities

Check vendor docs and source code:

- **Tools** = Yes if `@server.tool()` decorators exist or vendor docs list tool capabilities.
- **Resources** = Yes ONLY if `@server.resource()` decorators exist AND vendor docs confirm Resources capability is exposed.
- **Prompts** = Yes only if `@server.prompt()` decorators exist or vendor docs mention Prompt templates.
- **Sampling** = Yes only if the server requests LLM completions from the client (rare).
- If vendor docs say "only supports tool capabilities" → Resources = No, Prompts = No, Sampling = No.

---

### Capabilities - Tools detailed_info

List ALL tools (read-only and non-read-only alike) grouped by logical category with bullet
points. Infer sensible category names from the tool names and README (e.g. "Guardrails &
Policy Management", "Model & Agent Management", "Red-Teaming", "Deployment Management").

Use this exact format inside the CSV cell:

```
Category Name
    •    Action Name – Description of what this tool does.
    •    Action Name – Description of what this tool does.

Another Category Name
    •    Action Name – Description of what this tool does.
```

- Category headers are plain text — no brackets, no special characters around them.
- Each bullet uses 4 spaces before `•` and 4 spaces after `•`.
- Use `–` (en-dash) between the action name and description.
- Leave a blank line between category blocks.
- **Action Name** — use a short human-readable label that describes the operation type, not the raw function name. Examples: `Search Issues`, `Create Issue`, `Add Comment`, `Create Project`, `Get Users`. For utility or admin tools where no cleaner label exists, use the actual function name.
- Source: source code tool registration files and vendor docs.

---

### Non-Read-Only Tools detailed_info

**Detection — look for ALL of these signals in source code:**
- `readOnlyHint=False` or `read_only=False` in the `@mcp.tool()` decorator (Python)
- `annotations: { readOnlyHint: false }` in TypeScript tool registration
- HTTP POST / PUT / PATCH / DELETE calls inside the tool function body
- File write, file upload, file delete operations
- Any function that mutates state: creates, updates, deletes, submits, sends, publishes

List ONLY tools that match at least one signal above. Group them into logical operation
categories based on what they do (e.g. "Design Operations", "Organization Tools",
"Import/Export Tools", "Collaboration Tools", "Configuration Tools") — use whatever
category names best fit the server's domain.

**If the server is connected locally**: run `tools/list` via the MCP protocol to get the
live tool descriptions. Use those exact descriptions — do not invent or paraphrase. If the
live description is unreadable or overly technical, clean the wording while keeping every
detail and value unchanged.

Use this exact format inside the CSV cell:

```
Category Name
    •    tool_name – Description of what the tool does.
    •    Logic: Any important logic, constraint, or workflow note for the tool above.
    •    tool_name – Description of what the tool does.

Another Category Name
    •    tool_name – Description.
    •    Expert Note: Any critical usage note for the tool above.
```

- Category headers are plain text — no brackets, no special characters around them.
- Each bullet uses 4 spaces before `•` and 4 spaces after `•`.
- Use `–` (en-dash) between the tool name and description.
- Sub-notes (Logic / Constraint / Expert Note / Workflow / Critical) go on the line directly after the tool they describe, indented the same way (`    •    Note type: ...`).
- Leave a blank line between category blocks.
- Use the actual registered function name (`tool_name`) not a generic label.
- Only include categories that have at least one non-read-only tool.
- If no write signals are found for any tool, write exactly: `None — all tools are read-only`

**Never leave this section blank.** Either list the non-read-only tools or write the
"None" line — the CSV row must always be populated.

---

## Local Server Setup & Connection

Execute this section **only if** `Deployment: Local = Yes` (STDIO server). Skip entirely for remote servers.

### Step 1 — Read the full setup instructions

Fetch and read the complete GitHub README and any linked setup/installation docs for this server.
Do not skip or summarise — read every section: prerequisites, installation, configuration, environment
variables, and running instructions.

### Step 2 — Trigger the repo-clone skill for all local setup

Once you have read the README in Step 1, hand off entirely to the **`repo-clone` skill**.
Do not attempt to clone, install, configure, or run the server yourself — `repo-clone` owns
the complete local setup workflow from this point forward.

The `repo-clone` skill will handle:
- Deciding whether a local clone is needed or if a published package (PyPI/npm) is sufficient
- Presenting a full setup plan to the user and waiting for their approval
- Cloning the repository to the user's chosen path (default: `/Users/srinathp/MCP_repos/<repo-name>`)
- Installing all dependencies (Python venv/pip, npm/yarn, Docker — whatever the README requires)
- Asking the user for required environment variables and creating the `.env` file
- Verifying the server starts correctly
- Reporting back with the clone path, start command, and MCP client config JSON

Pass the GitHub URL and the README content you already fetched to `repo-clone` so it does
not need to re-fetch what you already read.

---

## CCI MCP-server Scoring

This section is triggered after authentication and connection setup is complete (Step 15).
Saving the CSV is the final action of this section — the file is written here, not before.

Ask the user this question exactly:

```
Would you like to include the CCI MCP-server Security Score in the report?

  1. Yes — calculate the score and save the full report
  2. No  — save the report now without the score section

Enter 1 or 2:
```

- If the user answers **No** (or 2): save the CSV now without any `CCI MCP-server Score` rows. After saving, print exactly: `✓ Report successfully generated and saved.`
- If the user answers **Yes** (or 1): calculate the score using the rules below, add the score rows to the CSV, then save. After saving, print exactly: `✓ Report successfully generated and saved.`

---

After the user confirms, calculate the security confidence score using this table.
Each attribute maps directly to the values already documented above — no extra research needed.

### Score Table

| # | Attribute | Score | Max |
|---|-----------|-------|-----|
| 1 | **MCP Protocol Version** | see below | 5 |
| 2 | **Authentication** | see below | 3 |
| 3 | **Transport Protocol** | see below | 5 |
| 4 | **Encryption (TLS)** | see below | 5 |
| 5 | **Tool Operations** | see below | 5 |
| 6 | **Distribution & Trust** | see below | 5 |
| 7 | **HIPAA** | see below | 5 |
| 8 | **GDPR** | see below | 5 |
| 9 | **SOC 2** | see below | 5 |
| 10 | **FedRAMP** | see below | 5 |
| | **TOTAL** | | **53** |

### Scoring Rules

**MCP Protocol Version** (max 5):
- `2025-11-25` → 5
- `2025-06-18` → 4
- `2025-03-26` → 2
- `2024-11-05` → 1

**Authentication** (max 3) — score the highest method found:
- OAuth 2.1 Authorization Code Flow → 3
- OAuth 2.1 Client Credentials Flow → 2
- Bearer Token (OAuth-issued, with validation) → 2
- Personal Access Token → 1
- API Token / API Key → 1
- No Auth → 1

**Transport Protocol** (max 5):
- StreamableHttp → 5
- STDIO → 3
- HTTP/SSE (deprecated) → 1

**Encryption — TLS** (max 5):
- TLS 1.3 → 5
- TLS 1.2 → 4
- Below TLS 1.2 or None → 1
- STDIO local transport → N/A (score 3, treated as neutral)

**Tool Operations** (max 5):
- Read-only → 5
- Includes Update/Create (no delete) → 2
- Includes Delete → 1

**Distribution & Trust** (max 5):
- Official (vendor-published) → 5
- Verified Community (high stars, active maintenance, reputable org) → 3
- Unverified Community (unknown publisher, low activity) → 1

**Compliance** — check the underlying SaaS service's trust/security page (max 5 each):
- HIPAA: BAA available → 5 | Partial coverage → 3 | None → 1
- GDPR: DPA + full compliance → 5 | Partial → 3 | None → 1
- SOC 2: Type II current → 5 | Type I → 3 | None → 1
- FedRAMP: High → 5 | Moderate → 3 | None → 1

### Confidence Level

- **High Confidence (≥80%, ≥43/53)**: Strong security posture, suitable for enterprise use.
- **Medium Confidence (50–79%, 27–42/53)**: Acceptable with mitigations, review recommended.
- **Low Confidence (<50%, <27/53)**: Significant risk, not recommended without remediation.

---

## Output Format

The **only output file** is a CSV saved to `~/Desktop/MCP_reports/<servername>.csv`.
Do not create any `.md` or other files. See the CSV Output Format section below for the
exact row structure.

---

## CSV Output Format

The CSV file is saved as `~/Desktop/MCP_reports/<servername>.csv`.
It uses **exactly three columns**: `Category, Attribute, Status` (always in this order).
Every cell is quoted with `csv.QUOTE_ALL` for proper multiline handling.

**Column Definitions:**
- **Category** — The attribute category (e.g., "MCP Info", "Distribution Type", "Capabilities")
- **Attribute** — The specific attribute name (e.g., "Description", "Official", "Tools")
- **Status** — The value/response (Yes/No/version/description text/detailed_info content)

⚠️ **Important**: When viewing the CSV in a table/spreadsheet application, long Status values (descriptions, detailed_info) will wrap or truncate in the display. This is normal. The complete content is stored in the CSV file.

Rows follow this order exactly:

```
Category,Attribute,Status
MCP Info,Description,"[3–4 line description of the server]"
MCP Info,Git Repo Version,[v1.2.0 - Brief description of version/release focus (optional emoji)]
MCP Info,Category,[one of the four categories]
MCP Info,GitHub Repository,[https://github.com/org/repo or N/A if not applicable]
MCP Info,Endpoint URL,[https://mcp.vendor.com/sse for remote servers, or N/A for STDIO-only servers]
Distribution Type,Official,Yes/No
Distribution Type,Community,Yes/No
MCP Protocol Version,2025-11-25,Yes/No
MCP Protocol Version,2025-06-18,Yes/No
MCP Protocol Version,2025-03-26,Yes/No
MCP Protocol Version,2024-11-05,Yes/No
Pricing,Free,Yes/No
Pricing,Paid,Yes/No
Hosting Provider,SaaS Vendor,Yes/No
Hosting Provider,3rd Party SaaS vendor,Yes/No
Hosting Provider,GitHub,Yes/No
Hosting Provider,GitLab,Yes/No
Hosting Provider,Bitbucket,Yes/No
Hosting Provider,SourceHut/Gitea/Gogs,Yes/No
Authentication,OAuth 2.1 - Authorization Code Flow,Yes/No
Authentication,OAuth 2.1 - Client Credentials Flow,Yes/No
Authentication,Bearer Token,Yes/No
Authentication,Personal Access Token,Yes/No
Authentication,API Token,Yes/No
Data Protection,TLS 1.3,Yes/No
Data Protection,TLS 1.2,Yes/No
Data Protection,Lower versions or no encryption,Yes/No
Transport Protocol,STDIO,Yes/No
Transport Protocol,HTTP/SSE,Yes/No
Transport Protocol,StreamableHttp,Yes/No
Transport Protocol,FastAPI,Yes/No
Tools Operations,Read-only operations,Yes/No
Tools Operations,Read-only and/or update operations,Yes/No
Tools Operations,Read-only update and/or delete operations,Yes/No
Deployment Approach,Local,Yes/No
Deployment Approach,Container,Yes/No
Deployment Approach,Remote,Yes/No
Capabilities,Tools,Yes/No
Capabilities,Resources,Yes/No
Capabilities,Prompts,Yes/No
Capabilities,Sampling,Yes/No
Capabilities - Tools,detailed_info,"Category Name↵  • tool_name – description.↵  • tool_name – description.↵↵Category Name↵  • tool_name – description."
Non-Read-Only Tools,detailed_info,"Category Name↵  • tool_name – description.↵↵Category Name↵  • tool_name – description. (or: None — all tools are read-only)"

--- ONLY include the rows below if the user answered Yes to the CCI score prompt ---

CCI MCP-server Score,MCP Protocol Version,[score] / [max]
CCI MCP-server Score,Authentication,[score] / [max]
CCI MCP-server Score,Transport Protocol,[score] / [max]
CCI MCP-server Score,Encryption (TLS),[score] / [max]
CCI MCP-server Score,Tool Operations,[score] / [max]
CCI MCP-server Score,Distribution & Trust,[score] / [max]
CCI MCP-server Score,HIPAA,[score] / [max]
CCI MCP-server Score,GDPR,[score] / [max]
CCI MCP-server Score,SOC 2,[score] / [max]
CCI MCP-server Score,FedRAMP,[score] / [max]
CCI MCP-server Score,TOTAL,[sum] / 53
CCI MCP-server Score,Confidence Level,[HIGH/MEDIUM/LOW] ([percentage]%)
CCI MCP-server Score,Pricing (informational),[Free/Paid description]
```

Write the CSV using Python's `csv.writer` with `quoting=csv.QUOTE_ALL` to correctly handle
multiline cells in the detailed_info rows.

---

## Remote Endpoint Auth and Connection

This section is triggered from Step 14 when the input is a **remote endpoint URL** (classified in Step 0). Skip `Authentication and Environment Setup` entirely for remote servers.

**Never ask the user to paste any secret, token, or key into the chat.** Apply the SECURITY MANDATE throughout.

---

### Step 1 — Probe the endpoint

Send an unauthenticated request to determine the server's auth requirement:

```bash
curl -s -o /dev/null -w "%{http_code} %{header_json}" <endpoint-url>
```

Interpret the response:

| Response code | Meaning | Next action |
|---|---|---|
| `200` + SSE/JSON body | No auth required | Skip to Step 4 |
| `401` | Auth required | Read `WWW-Authenticate` header → Step 2 |
| `302` / `307` | OAuth redirect | → Step 2 (OAuth flow) |
| `403` | Forbidden — likely wrong/missing key | Check vendor docs → Step 2 |

---

### Step 2 — Detect authentication method

Check all of the following signals — mark every auth method found:

**Bearer Token:**
- `WWW-Authenticate: Bearer realm="..."` in 401 response
- Vendor docs show `Authorization: Bearer <token>` header

**OAuth 2.1 — Authorization Code Flow:**
```bash
curl -s <scheme>://<host>/.well-known/oauth-authorization-server
```
If this returns a JSON document with `authorization_endpoint` and `token_endpoint` → OAuth 2.1 Auth Code Flow. The MCP client handles the browser redirect — no token needs to be pre-configured.

**OAuth 2.1 — Client Credentials Flow:**
- Vendor docs describe M2M or service-account access with `client_credentials` grant type.

**API Token / Key:**
- `WWW-Authenticate: ApiKey` or vendor docs specify a header like `X-Api-Key`, `X-API-Key`, or a query parameter `?api_key=`.

**No Auth:**
- All probes return 200 and no auth header is present → Authentication: No Auth.

Record every method found. Score using the highest-security method for the CCI score.

---

### Step 3 — Ask the user which Claude Code config path to use

Ask exactly:

```
Where would you like to add this MCP server config?

  1. Global (all projects)  →  ~/.claude/settings.json
  2. This project only      →  <current-working-directory>/.claude/settings.json

Enter 1 or 2:
```

If the user chooses option 2, add the `.gitignore` security reminder.

---

### Step 4 — Check whether the user has credentials

Present the credential requirement and ask exactly:

```
This MCP server requires the following credentials:

  <CREDENTIAL TYPE>  (<description>)
  Get one: <link from vendor docs>

Do you have these credentials ready?

  1. Yes — I have my <credential type> and I'm ready to add it
  2. No  — I don't have it yet

Enter 1 or 2:
```

Follow Branch A (has credentials) or Branch B (does not) as defined in the Authentication and Environment Setup section. Apply the same placeholder and credential-handling rules.

---

### Step 5 — Build the remote config block with placeholders

Use the `url` key (not `command`/`args`) for remote servers. Use `<KEY>` placeholders for every credential:

**Bearer Token / API Key:**
```json
{
  "mcpServers": {
    "<server-name>": {
      "url": "<endpoint-url>",
      "headers": {
        "Authorization": "Bearer <BEARER_TOKEN>"
      }
    }
  }
}
```

**API Key in custom header:**
```json
{
  "mcpServers": {
    "<server-name>": {
      "url": "<endpoint-url>",
      "headers": {
        "X-Api-Key": "<API_KEY>"
      }
    }
  }
}
```

**OAuth 2.1 (browser-managed — no pre-configured token):**
```json
{
  "mcpServers": {
    "<server-name>": {
      "url": "<endpoint-url>"
    }
  }
}
```
Note: Claude Code will open a browser window for OAuth login automatically.

Read the existing settings file and merge — do not overwrite other configured servers.

---

### Step 6 — Verify the remote connection

Run this only when credentials are ready (Branch A). Skip for Branch B options 2 and 3.

**SSE endpoint:**
```bash
curl -N \
  -H "Accept: text/event-stream" \
  -H "Authorization: Bearer <token>" \
  <endpoint-url>
```
Expected: an SSE stream with at least one `event:` line (e.g. `endpoint` or `message` event).

**StreamableHttp endpoint:**
```bash
curl -s -X POST \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer <token>" \
  -d '{"jsonrpc":"2.0","id":1,"method":"initialize","params":{"protocolVersion":"2024-11-05","capabilities":{},"clientInfo":{"name":"test","version":"1.0"}}}' \
  <endpoint-url>
```
Expected: `{"result":{"protocolVersion":"...","serverInfo":{...},...}}`

If verified, show:

```
Connection confirmed
====================
Server     : <serverInfo.name>
Protocol   : <protocolVersion>
Transport  : <HTTP/SSE or StreamableHttp>
Endpoint   : <endpoint-url>

Restart Claude Code — the MCP server will load automatically.
```

If failed: hand off to the **error-handling** skill with the full error output.

---

## Authentication and Environment Setup

This section is triggered in **three ways** — the source of data changes depending on trigger:

| Trigger | Data source | What's already known |
|---------|-------------|----------------------|
| After report is saved (normal flow) | GitHub README already fetched | Attributes, auth type, env var names |
| Called from `repo-clone` after local clone | Local disk files at `<clone-path>` | `command`, `args`, env var names from local files |
| User asks to set up auth independently | GitHub URL or local path | Nothing yet — start from Step 1 |

When called from **`repo-clone`**: skip fetching from GitHub. Use the `command`, `args`, and
env var names already extracted from the local clone. Proceed directly to Step 2.

**Never ask the user to paste any secret, token, or key into the chat.** Credentials must
only ever be entered directly into the config file by the user in their own editor.

---

### Step 1 — Extract command, args, and env vars

Read the following sources (GitHub or local disk depending on trigger) to determine the three
values needed for the MCP client config entry:

**`command`** — the executable that starts the server:
- Check the README's "Installation" or "Usage" section for the run command
- Common values: `uv`, `node`, `python3`, `python`, `docker`, `npx`

**`args`** — the argument list passed to the command:
- Read the README's config JSON example (usually shown as an `mcpServers` block)
- For Python/uv: often `["--directory", "<clone-path>", "run", "<script.py>"]`
- For Node.js: often `["<clone-path>/dist/index.js"]` or `["-y", "<package-name>"]`
- Replace any hardcoded paths with the actual clone path

**`env`** — every environment variable the server reads:
- README: look for `.env.example` references, config tables, export statements
- Source code: look for `os.environ.get(...)`, `process.env.VAR_NAME`, `getenv()`
- For each var note: exact name, required/optional, auth type, where to obtain it (link)

Auth type classification:
- `API_KEY`, `X-API-KEY`, `*_API_KEY` → API Token
- `*_TOKEN` (non-OAuth) → Bearer Token or PAT depending on README context
- `GITHUB_TOKEN`, `GITLAB_TOKEN`, `glpat_*` → Personal Access Token
- `*_CLIENT_ID` / `*_CLIENT_SECRET` → OAuth 2.1
- `BEARER_TOKEN`, `AUTH_TOKEN` → Bearer Token

---

### Step 2 — Ask the user which Claude Code config path to use

Ask this question exactly:

```
Where would you like to add this MCP server config?

  1. Global (all projects)  →  ~/.claude/settings.json
  2. This project only      →  <current-working-directory>/.claude/settings.json

Enter 1 or 2:
```

Wait for their answer. Use the chosen path in all subsequent steps.

If the user chooses option 2 (project-level), add a security reminder:
```
Security note: The project settings file (.claude/settings.json) may be committed to git.
Add it to your .gitignore if it will contain credentials.
```

---

### Step 3 — Credential check: ask the user if they have the required credentials

Before writing any config, present the credentials this server needs and ask whether the
user has them. Show the credential type, the env var name, and where to obtain one.

Ask exactly:

```
This MCP server requires the following credentials:

  <CREDENTIAL TYPE>  (<VAR_NAME>)
  Get one: <link from README>

  [repeat for each required credential]

Do you have these credentials ready?

  1. Yes — I have my <credential type> and I'm ready to add it
  2. No  — I don't have it yet

Enter 1 or 2:
```

Then follow the branch below based on their answer.

---

#### Branch A — User has credentials (answer: 1)

Proceed with the full keyed setup:

1. Write the config file with `<KEY>` placeholders (Step 4).
2. Tell the user to open the file and replace each placeholder directly — never in chat.
3. After they confirm the file is saved, verify the connection (Step 6).

---

#### Branch B — User does not have credentials (answer: 2)

Show them exactly how to obtain the credential type they need, then ask:

```
No problem. Here's how to get your <credential type>:

  <Auth-type-specific instructions — see Step 5>

Once you have it, you can either:
  1. Get it now and come back — I'll wait and continue setup when you're ready
  2. Proceed without it — the server config will be written with a placeholder.
     The server will load in Claude Code but all API calls will fail until you
     add a real credential later.
  3. Stop here — I'll save the config with a placeholder so you can fill it in
     whenever you're ready. Just restart Claude Code after you update the file.

Enter 1, 2, or 3:
```

- **Answer 1 (Get it now):** Wait. When the user says they're ready, continue as Branch A.
- **Answer 2 (Proceed without):** Write the config with placeholders. Note clearly that API calls will return 401/403 until credentials are added. Proceed to verify server startup only (not full API connection).
- **Answer 3 (Stop):** Write the config with placeholders. Show the path and placeholder names. End with:
  ```
  Config saved with placeholder values at:
    <path to settings.json>

  When you have your <credential type>:
    1. Open the file above
    2. Replace <PLACEHOLDER> with your real credential
    3. Restart Claude Code

  Come back any time to continue — just say "resume MCP setup for <server name>".
  ```

---

### Step 4 — Build the config block with placeholders

Build the `mcpServers` entry using the `command`, `args`, and `env` var names determined in
Step 1. Use `<KEY>` style placeholders for every credential — never pre-fill real values:

```json
{
  "mcpServers": {
    "<server-name>": {
      "command": "<command — e.g. uv, node, python>",
      "args": ["<arg1>", "<arg2>"],
      "env": {
        "<VAR_NAME>": "<KEY>"
      }
    }
  }
}
```

Placeholder naming convention — use the label that matches the credential type:

| Auth type | Placeholder to use |
|-----------|-------------------|
| API Token / API Key | `<API_KEY>` |
| Bearer Token | `<BEARER_TOKEN>` |
| Personal Access Token | `<PERSONAL_ACCESS_TOKEN>` |
| OAuth Client ID | `<OAUTH_CLIENT_ID>` |
| OAuth Client Secret | `<OAUTH_CLIENT_SECRET>` |
| Generic secret | `<SECRET_KEY>` |

Read the existing content of the target settings file (if it exists). Merge the new
`mcpServers` entry into the existing `mcpServers` block — do not overwrite other servers
already configured. If the file does not exist, create it with just this entry.

For **optional** vars, note in the message (not in the JSON) that they can be left blank
or removed if not needed.

---

### Step 5 — Auth-type-specific credential instructions

Use these instructions in Branch B when explaining how to obtain each credential type.

**API Token / API Key:**
```
To get your API key:
  1. Open: <sign-up or dashboard link from README>
  2. Go to Settings → API Keys (or equivalent)
  3. Generate a new key
  4. Open <path to settings.json> and replace <API_KEY> with your key
  Do NOT paste the key here.
```

**Bearer Token:**
```
Bearer tokens are issued after login or via a token endpoint.
  1. Follow the token issuance steps in the README: <link>
  2. Note: bearer tokens may be short-lived — check if refresh is needed
  3. Open <path to settings.json> and replace <BEARER_TOKEN> with your token
  Do NOT paste the token here.
```

**Personal Access Token (PAT):**
```
To create a Personal Access Token:
  1. Open: <token generation page from README>
  2. Select required scopes: <list scopes from README>
  3. Generate → open <path to settings.json> and replace <PERSONAL_ACCESS_TOKEN>
  Do NOT paste the token here.
```

**OAuth 2.1 — Authorization Code Flow:**
```
To set up OAuth:
  1. Open the app registration page: <link from README>
  2. Set the redirect URI to: <value from README>
  3. Select required scopes: <from README>
  4. Copy the Client ID → open <path to settings.json>, replace <OAUTH_CLIENT_ID>
  5. Copy the Client Secret → replace <OAUTH_CLIENT_SECRET>
  Do NOT share Client ID or Client Secret here.
```

**OAuth 2.1 — Client Credentials Flow:**
```
These are service-account credentials from the vendor's developer console.
  1. Open: <developer console link from README>
  2. Create a service account or application
  3. Copy Client ID → replace <OAUTH_CLIENT_ID> in settings.json
  4. Copy Client Secret → replace <OAUTH_CLIENT_SECRET> in settings.json
  Do NOT share credentials here.
```

---

### Step 6 — Verify the connection (keyed setup only)

Run this only when the user has provided real credentials (Branch A). Skip for Branch B options 2 and 3.

Send the JSON-RPC initialize request to confirm the server starts and responds:

```bash
echo '{"jsonrpc":"2.0","id":1,"method":"initialize","params":{"protocolVersion":"2024-11-05","capabilities":{},"clientInfo":{"name":"test","version":"1.0"}}}' \
  | <command> <args>
```

Expected healthy response contains `"result"` with `"protocolVersion"` and `"serverInfo"`.

If verified, show this message (fill in actual values):

```
Connection confirmed
====================
Server     : <serverInfo.name>
Version    : <serverInfo.version>
Protocol   : <protocolVersion>
Transport  : STDIO

Report saved : ~/Desktop/MCP_reports/<servername>.csv

Restart Claude Code — the MCP server will load automatically.
```

If failed: hand off to the **error-handling** skill with the full error output.

---

### Security rules — always follow these

> These rules are a subset of the **SECURITY MANDATE** at the top of this skill.
> The mandate takes precedence. If any step in this skill appears to conflict with the
> mandate, follow the mandate.

- **Never ask the user to type or paste any secret, token, or key into this chat.**
- **Never display, log, echo, or include any credential value in the report files (CSV).**
- **Never store credentials in plain text outside the settings file** — no `.env` files committed to git, no keys in README, no keys in the CSV report.
- If the user accidentally pastes a key into the chat, immediately tell them:
  ```
  Security alert: Please do not share credentials here. Treat this value as potentially
  compromised — revoke it and generate a new one, then add the new one directly into
  the settings file at <path>. Do NOT paste it here.
  ```
- If the config file is project-level (`.claude/settings.json`), always remind the user
  to add it to `.gitignore` before committing.

---

## Common Mistakes to Avoid

- **Incorrectly handling dual-deployment servers** — For servers with BOTH a vendor remote endpoint AND a GitHub local proxy: (1) Hosting Provider: SaaS Vendor=Yes, GitHub=No (vendor is primary). (2) Transport: mark BOTH STDIO=Yes AND the remote transport (StreamableHttp or HTTP/SSE)=Yes. (3) Deployment: Local=Yes, Remote=Yes, Container=Yes if Dockerfile exists. (4) Auth: combine methods found from both paths.
- **Missing TLS verification for remote connections** — When Remote = Yes, ALWAYS run curl --tlsv1.2 and curl --tlsv1.3 tests against the endpoint. Do NOT skip TLS verification. For dual-deployment servers, TLS values apply to the Remote endpoint only (Local STDIO has no network encryption).
- **Using N/A instead of No** — CSV allows ONLY "Yes" or "No". Never write "N/A", "Unknown", "Partial". For STDIO servers: TLS 1.3=No, TLS 1.2=No, Lower versions=No (not N/A). Users will catch this immediately.
- **Writing generic, vague descriptions** — Description field MUST include specific details about capabilities, tool counts, supported networks, and key features. Users WILL request expansion if description is too generic. Always use concrete feature names, not abstract statements.
- Creating a `.md` report file — the only output is `<servername>.csv`
- Using flat `tool_name: description` format in Capabilities or Non-Read-Only Tools — always use grouped `[Category Name]` + `• tool_name – description` format
- Omitting the blank line between category blocks in the detailed_info cell — always separate groups with a blank line for readability
- Leaving Non-Read-Only Tools blank instead of writing `None — all tools are read-only` when all tools are read-only
- Missing non-read-only tools by only checking the decorator name — always check the function body for POST/PUT/DELETE calls and file upload operations too
- Using `"Security Score"` as the CSV category name — the correct name is `"CCI MCP-server Score"`
- Marking GitHub = No for hosting when the repo IS on GitHub (even STDIO servers should have GitHub = Yes)
- Marking Self-hosted = Yes instead of marking the actual source platform (GitHub/GitLab/etc.)
- Marking SaaS Vendor = Yes for a local STDIO server just because the upstream API is vendor-hosted
- Marking HTTP/SSE or StreamableHttp = Yes when the server has no remote endpoint (STDIO only)
- Marking HTTP/SSE = Yes because `fastapi` or `uvicorn` appear in dependencies — always verify these are used for MCP transport, not for a separate webhook receiver or internal HTTP utility. Check the actual `mcp.run()` call and transport parameter in source code
- Marking TLS = Yes based on upstream API calls or database connections (e.g. the server calls `https://api.vendor.com` or connects to PostgreSQL with `sslmode=require`) — TLS only applies to the MCP transport layer itself, not to upstream services the server connects to. A STDIO server that connects to a TLS-encrypted database internally still has TLS 1.3=No, TLS 1.2=No for the MCP layer.
- Marking Remote = Yes because ngrok or a webhook URL appears in the code — ngrok for receiving inbound API callbacks is not the same as a vendor-hosted MCP endpoint. Remote = Yes only if the vendor hosts an MCP server at a public URL for clients to connect to
- **Skipping remote endpoint discovery for GitHub/server-name inputs** — ALWAYS run Step 0.5 to search for remote endpoints even if not mentioned in the README title. Many servers have vendor-hosted remote endpoints that aren't prominently featured. Check vendor docs, `.env.example` files, and likely URL patterns.
- Stopping at the GitHub repo when the README says "see vendor docs for our remote MCP" or "we offer a hosted/remote MCP implementation" — always follow that link and check the vendor docs for a remote endpoint URL before finalising Transport, TLS, Hosting Provider, and Deployment. The repo may only show the local STDIO mode while the vendor separately hosts a StreamableHttp endpoint over HTTPS
- Marking Remote = Yes without a confirmed public endpoint URL
- **Assuming STDIO-only without checking for remote endpoint** — just because a server is open-source on GitHub does NOT mean it's STDIO-only. Always run endpoint discovery before deciding deployment type is Local-only.
- Marking TLS = Yes for local STDIO servers that have no network transport — even if the server internally uses TLS for upstream connections
- Marking TLS = No for remote HTTPS vendor-hosted servers — for Remote deployments, ALWAYS verify with curl/openssl before marking TLS
- **Requiring a remote endpoint URL to mark TLS = Yes** — TLS assessment is only meaningful for remote MCP endpoints. For STDIO-only servers (no vendor-hosted endpoint), mark all TLS versions as No, regardless of how the server internally connects to databases/APIs
- Marking all three Tools Operations rows as Yes instead of only the highest applicable
- Marking Resources = Yes based only on source code when vendor docs say tools-only
- Marking multiple MCP protocol versions as Yes instead of only the latest
- Using "Partial" instead of Yes or No
- Assuming auth methods from general knowledge without checking vendor docs or source code
- Forgetting to save the report to `~/Desktop/MCP_reports/`
- Skipping connection verification for either path — connection verification is MANDATORY on both remote endpoint and STDIO paths. Never proceed to the CCI Score Prompt without a confirmed connection (or a documented failure handed to error-handling)
- Treating a remote endpoint URL (`https://mcp.vendor.com/sse`) as a STDIO server — if the input URL matches the remote endpoint pattern, pre-set Transport/Deployment/TLS immediately and use the Remote Endpoint Auth and Connection section
- Using `command`/`args` config for a remote endpoint — remote servers always use `url` (and optionally `headers`) in the mcpServers config, never a local process command
- Skipping the endpoint probe (Step 1 of Remote Endpoint Auth) and guessing the auth method — always probe first; the HTTP response headers are the ground truth
- Marking `OAuth 2.1 Authorization Code Flow = Yes` for a remote endpoint without checking `/.well-known/oauth-authorization-server` — always fetch the discovery document to confirm
- Skipping the README and vendor page endpoint scan for GitHub URLs — always scan both before assuming STDIO; if a remote endpoint is found, always ask the user which mode they prefer
- Marking `HTTP/SSE = Yes` for a `/mcp` endpoint or `StreamableHttp = Yes` for a `/sse` endpoint — path ending determines transport: `/sse` → HTTP/SSE, `/mcp` or `/mcp/` → StreamableHttp
- **Leaving GitHub Repository blank** — Always populate this field. Use the GitHub URL if source exists on GitHub, or `N/A` if not applicable. Never leave it empty.
- **Leaving Endpoint URL blank** — Always populate this field. For remote servers: use the endpoint URL. For STDIO-only: write `N/A`. Never leave it empty.
- **Confusing user-provided parameters with the MCP endpoint** — For servers like Aurora DSQL with `--cluster_endpoint` parameter: the user provides their own cluster endpoint, but the MCP Server itself connects at `https://xmfe3hc3pk.execute-api.us-east-2.amazonaws.com`. The latter goes in Endpoint URL.
- **Writing the wrong endpoint for dual-deployment servers** — For servers with BOTH remote endpoint AND GitHub STDIO: choose ONE endpoint URL (the one researched). If researching remote, use remote endpoint. If researching STDIO, use `N/A`. Do NOT list both.
