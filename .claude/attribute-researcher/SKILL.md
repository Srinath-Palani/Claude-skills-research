---
name: attribute-researcher
description: >
  Research and fill in every structured attribute of an MCP server profile. Use when the
  mcp-server skill needs to populate the attribute fields for an MCP server. Use when the
  user asks to "fill in attributes", "document the fields", "catalogue this MCP server",
  or "populate the MCP profile" for a given server. Use when the user wants evidence-backed
  attribute rows covering Distribution Type, Pricing, Hosting Provider, Authentication,
  Data Protection, Transport Protocol, Tools Operations, Deployment Approach, and
  Capabilities. Produces the evidence-backed attribute rows that feed into the final CSV
  report.
user-invocable: true
---

# MCP Server Attribute Researcher

> Full workflow diagram: see `workflow.md` in this directory.
> Update `workflow.md` whenever this skill is modified.

This skill is triggered in two ways:
1. **From the `mcp-server` skill** — called during the attribute-filling phase (Verification Steps 1–9) after sources have been identified.
2. **Directly** — when a user wants to fill in or document the structured attribute fields for a given MCP server.

The goal is a fully evidenced set of attribute values — every Yes/No backed by a source quote or file path. Never assume. If you cannot confirm from the priority sources, mark **No**.

---

## Usage

Fill in structured attribute fields for any MCP server:

```
"Fill in the attributes for the Snowflake MCP server"
"Populate the MCP profile for this repo: https://github.com/org/repo"
"Catalogue this MCP server"
"Document all the fields for the GitHub MCP server"
"What are the authentication and transport attributes of this MCP server?"
```

Returns all attribute values with evidence back to `mcp-server`, or directly to the user
when triggered standalone.

---

## Source Priority Order

Use sources in this order (highest trust first):

1. **Official vendor documentation page** — the vendor's own docs site
2. **GitHub / source code repository** — for open-source servers
3. **PyPI / npm package page** — for packaged servers
4. **MCP spec documentation** — spec.modelcontextprotocol.io

Every "Yes" must be backed by evidence from one of these sources.

---

## Step 1 — Fetch Sources

Read all available sources before filling any attribute:

- Vendor documentation page (if one exists)
- GitHub README and any linked setup/auth docs
- Source code: tool registration files, connection/auth modules, main entry point
- `requirements.txt` / `pyproject.toml` / `package.json` — for dependency versions
- `.env.example` — for environment variable names and auth types

Do not fill any attribute until you have read the relevant source.

---

## Step 2 — Fill All Attribute Fields

For each attribute below, apply the field rule, record the value, the source, and the exact evidence (quote or file path).

Only two values are allowed: **Yes** or **No**. Do not use Partial, Unknown, or N/A.

---

### Distribution Type

- **Official** = developed and maintained by the original service provider.
- **Community** = third-party developer not affiliated with the original service.
- Mutually exclusive — only one can be Yes.
- Check the GitHub organisation or company that owns the repository.

---

### Pricing

- **Free** = Yes if the server can be used without payment via ANY connection method (remote endpoint, STDIO, container): open-source, free tier, or any free trial (even time-limited, even if payment details are collected at signup). A free trial counts as Free = Yes.
- **Paid** = Yes ONLY if ALL connection methods require entering payment details to sign up AND there is absolutely no free trial, no free tier, and no free-access path whatsoever. If any free or trial access exists for any method → Paid = No.

---

### Hosting Provider

**Mutually exclusive — only ONE should be marked Yes** (primary deployment/distribution method):

- **SaaS Vendor** = Yes if the original vendor hosts a remote MCP endpoint at their own domain and primary distribution is vendor-hosted (not GitHub).
- **GitHub** = Yes if the primary distribution is via a GitHub repository (including STDIO servers cloned from GitHub for local deployment). GitHub presence alone (docs/reference only) does NOT count.
- **GitLab** = Yes if the source repository is on GitLab.
- **Bitbucket** = Yes if on Bitbucket.
- **SourceHut/Gitea/Gogs** = Yes if on one of these platforms.
- **3rd Party SaaS vendor** = Yes if a third party (not the original service vendor) hosts the server.

Example:
- Vendor endpoint at `https://mcp.vendor.com` (even with local GitHub proxy available) → SaaS Vendor = Yes, GitHub = No
- Distributed on GitHub for local STDIO deployment only → GitHub = Yes, SaaS Vendor = No

Do **not** add a "Self-hosted" row. Mark only the actual hosting platform.

---

### Authentication

Check vendor docs and source code. Report all detected methods.

- **OAuth 2.1 Authorization Code Flow** = Yes if vendor docs mention OAuth or browser-based login; look for `/authorize` + `/callback` routes, PKCE.
- **OAuth 2.1 Client Credentials Flow** = Yes if M2M/service account OAuth is supported.
- **Bearer Token** = Yes if `Authorization: Bearer <token>` headers are used with token validation.
- **Personal Access Token** = Yes if PAT is accepted (`GITHUB_TOKEN`, `glpat_`, etc.).
- **API Token** = Yes if static API key is accepted (`API_KEY`, `X-API-Key`, etc.).

---

### Data Protection — TLS

- **Remote** (vendor-hosted HTTPS endpoint): Always verify with curl before marking Yes. Run both:
  ```bash
  # TLS 1.2 check — connects = Yes
  curl -vI --tlsv1.2 --tls-max 1.2 <endpoint-url>
  # TLS 1.3 check — connects = Yes
  curl -vI --tlsv1.3 <endpoint-url>
  ```
  If macOS LibreSSL cannot run `--tlsv1.3` (curl error 4), fall back to `openssl s_client -connect <host>:443` to check the negotiated protocol, or use Cloudflare infrastructure evidence (Cloudflare-proxied endpoints always support TLS 1.3). Only mark Yes if the connection actually succeeds or infrastructure evidence confirms support.
- **Local only** (STDIO subprocess, no network): TLS 1.3 = No, TLS 1.2 = No.
- **Container** (self-hosted Docker): TLS = No unless vendor docs explicitly mention TLS. If mentioned, run the verification commands above.
- TLS applies to the **MCP transport layer** only — not to upstream API calls the server makes.

---

### Transport Protocol

- **STDIO** = Yes if the server runs as a local subprocess via stdin/stdout.
- **HTTP/SSE** = Yes if the server exposes a legacy Server-Sent Events endpoint (spec 2024-11-05).
- **StreamableHttp** = Yes if the server exposes an HTTP endpoint using StreamableHttp transport (spec 2025-03-26 or later).
- For **dual-deployment servers** (GitHub-input with discovered remote endpoint): mark ALL applicable transports — STDIO=Yes AND (HTTP/SSE=Yes or StreamableHttp=Yes). Not mutually exclusive when both deployment modes exist.
- For STDIO-only servers (no confirmed remote endpoint): HTTP/SSE=No, StreamableHttp=No.
- Always verify by checking the actual `mcp.run()` call and transport parameter in source code — do not infer from `fastapi` or `uvicorn` appearing in dependencies.

---

### Tools Operations

Evaluate all tools and mark **only the highest** applicable tier as Yes:

- **Read-only** = Yes (others No) — ALL tools are read-only.
- **Read-only and/or update** = Yes (others No) — tools include read AND write/create/update but NO delete.
- **Read-only, update and/or delete** = Yes (others No) — tools include read AND write/update AND delete.

Only one row = Yes. The other two must be No.

---

### Deployment Approach

- **Local** = Yes if the server runs as a local process on the user machine (STDIO).
- **Container** = Yes if a Dockerfile or docker-compose is provided and the user runs it themselves.
- **Remote** = Yes if the server is hosted remotely by the vendor and accessed via a URL endpoint.
- Local + Container can both be Yes for self-hosted servers.
- If Remote = Yes (vendor managed): Local = No, Container = No — **unless the input was a GitHub URL**, in which case Local = Yes and STDIO = Yes must remain set even when Remote = Yes.

---

### Capabilities

Check vendor docs and source code:

- **Tools** = Yes if `@server.tool()` decorators exist or vendor docs list tool capabilities.
- **Resources** = Yes ONLY if `@server.resource()` decorators exist AND vendor docs confirm Resources are exposed.
- **Prompts** = Yes only if `@server.prompt()` decorators exist or vendor docs mention Prompt templates.
- **Sampling** = Yes only if the server requests LLM completions from the client (rare).

---

## Step 3 — Capabilities Tools detailed_info

List ALL tools grouped by logical category. Infer sensible category names from tool names and README.

Format inside the CSV cell:

```
Category Name
    •    Action Name – Description of what this tool does.
    •    Action Name – Description of what this tool does.

Another Category Name
    •    Action Name – Description of what this tool does.
```

- Category headers are plain text — no brackets, no special characters.
- Each bullet: 4 spaces before `•`, 4 spaces after `•`.
- Use `–` (en-dash) between action name and description.
- Blank line between category blocks.
- Use short human-readable action labels, not raw function names (e.g. `Search Issues`, not `search_issues`).

---

## Step 4 — Non-Read-Only Tools detailed_info

Detect non-read-only tools using ALL of these signals:

- `readOnlyHint=False` or `read_only=False` in the decorator
- `annotations: { readOnlyHint: false }` in TypeScript
- HTTP POST / PUT / PATCH / DELETE calls inside the function body
- File write, file upload, file delete operations
- Any function that creates, updates, deletes, submits, sends, or publishes

List only tools that match at least one signal. Use the same grouped bullet format as above, but use the actual registered function name (`tool_name`) and add sub-notes where needed:

```
Category Name
    •    tool_name – Description of what the tool does.
    •    Logic: Any important logic, constraint, or workflow note.
    •    tool_name – Description.
```

If no write signals exist for any tool, write exactly: `None — all tools are read-only`

**Never leave this section blank.**

---

## Step 5 — Return Results to mcp-server

When triggered from `mcp-server`, return:

- All filled attribute values with evidence
- The `Capabilities - Tools detailed_info` content
- The `Non-Read-Only Tools detailed_info` content

`mcp-server` will then proceed to MCP Info (version, description, category), the CCI score prompt, and CSV generation.

---

## Common Mistakes to Avoid

- Marking GitHub = No when the repo IS on GitHub
- Marking both SaaS Vendor = Yes AND GitHub = Yes (only one hosting provider should be primary)
- Marking HTTP/SSE or StreamableHttp = Yes for a STDIO-only server
- Marking TLS = Yes based on upstream API calls (e.g. `https://api.vendor.com`) — TLS only applies to the MCP transport layer
- Marking Remote = Yes without a confirmed public endpoint URL
- Marking multiple Tools Operations rows as Yes instead of only the highest
- Marking Resources = Yes from source code alone when vendor docs say tools-only
- Using "Partial" instead of Yes or No
- Inferring auth methods from general knowledge without checking vendor docs or source code
