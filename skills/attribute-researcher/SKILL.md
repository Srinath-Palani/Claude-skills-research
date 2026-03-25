---
name: attribute-researcher
user-invocable: true
description: >
  Fill in every structured attribute of an MCP server profile with evidence-backed Yes/No values.
  Use when mcp-researcher needs attribute population, or when user asks to "fill attributes",
  "catalogue server", or "document fields". Returns fully evidenced attribute rows for CSV report.
---

# MCP Server Attribute Researcher

> Full workflow diagram: see `workflow.md` in this directory.
> Update `workflow.md` whenever this skill is modified.

Fill all MCP server attributes with evidence-backed Yes/No values. Never assume.
If unconfirmed from priority sources, mark **No**.

**Triggered by:**
1. mcp-researcher skill (during attribute-filling phase)
2. User directly (standalone attribute documentation)

---

## Feature Overview

✓ Evidence-backed research (every Yes has source quote or file path)
✓ Source priority order (vendor → GitHub → PyPI → MCP spec)
✓ Structured attributes (all attribute fields with Yes/No discipline)
✓ Tool enumeration (categorized tool listing)
✓ Non-read-only detection (write operation identification)
✓ Dual-mode operation (called from mcp-researcher or standalone)

---

## Usage

```
"Fill in the attributes for the Snowflake MCP server"
"Populate the MCP profile for: https://github.com/org/repo"
"Catalogue this MCP server"
"Document all fields for the GitHub MCP server"
```

---

## Source Priority (Highest to Lowest)

1. Official vendor documentation page
2. GitHub / source code repository
3. PyPI / npm package page
4. MCP specification documentation

**Every Yes must be backed by evidence from these sources.**

---

## Workflow: 5 Steps

### Step 1 — Fetch All Sources

Before filling any attribute, read:
- Vendor docs (if exists)
- GitHub README + setup/auth docs
- Source code (tool registration, auth modules, entry point)
- `requirements.txt` / `pyproject.toml` / `package.json` (SDK versions)
- `.env.example` (environment variable names)

**Do not proceed until all sources are read.**

### Step 2 — Fill Attribute Fields

**Values:** Yes or No only (no Partial, Unknown, N/A)

Record for each:
- Attribute value (Yes/No)
- Source (vendor docs, GitHub, etc.)
- Evidence (exact quote or file path)

### Step 3 — Enumerate Tools

Format:
```
Category Name
    •    tool_name – brief description

Another Category
    •    tool_name – brief description
```

Rules:
- Group by logical category
- Use human-readable labels
- 4 spaces before bullet, 4 after
- Use en-dash (–) between name and description

### Step 4 — Detect Non-Read-Only Tools

Signals (detect ANY of):
- `readOnlyHint=False` or `read_only=False` decorator
- HTTP POST/PUT/PATCH/DELETE calls
- File write/upload/delete operations
- Any create/update/delete/submit functions

Format: Same as Step 3 with logic notes

If none found: "None — all tools are read-only"

### Step 5 — Return to mcp-researcher

When called from mcp-researcher, return:
- All filled attributes with evidence
- Capabilities - Tools categorized list
- Non-Read-Only Tools categorized list

mcp-researcher then proceeds to scoring and CSV generation.

---

## Attribute Rules

| Attribute | Rule | Mutually Exclusive |
|-----------|------|-------------------|
| **Distribution** | Official (vendor) or Community (3rd party) | Yes - one only |
| **Pricing** | Free (any free access) or Paid (all paid) | Yes - one only |
| **Hosting** | SaaS/GitHub/GitLab/Bitbucket/SourceHut/3rd Party | Yes - one only |
| **Authentication** | Check all types (OAuth/Bearer/PAT/API Key) | No - multiple allowed |
| **TLS** | Verify with curl (remote only); STDIO=No | No - multiple allowed |
| **Transport** | STDIO/HTTP-SSE/StreamableHttp | Yes - one only |
| **Tools Operations** | Mark highest: Read-only, Read+Update, Read+Update+Delete | Yes - highest only |
| **Deployment** | Local/Container/Remote combinations | Partial (see rules) |
| **Capabilities** | Tools/Resources/Prompts/Sampling | No - multiple allowed |

---

## Key Field Details

### Distribution Type
- **Official** = developed by original service provider
- **Community** = third-party developer
- Check GitHub organization ownership

### Pricing
- **Free** = Yes if ANY free access path (OSS, trial, free tier)
- **Paid** = Yes ONLY if ALL paths require payment, zero free options

### Hosting Provider
- **SaaS Vendor** = vendor hosts remote endpoint at own domain
- **GitHub** = primary distribution via GitHub repo
- **GitLab/Bitbucket/SourceHut** = repository on these platforms
- **3rd Party SaaS** = third party hosts the server
- ⚠️ Mutually exclusive - mark primary distribution only

### Authentication
All types allowed. Check vendor docs + source code:
- OAuth 2.1 Authorization Code (browser login)
- OAuth 2.1 Client Credentials (M2M)
- Bearer Token (auth header)
- Personal Access Token (PAT)
- API Token (static key)

### Data Protection - TLS
- **Remote endpoints**: Run curl verification:
  ```bash
  curl -vI --tlsv1.2 --tls-max 1.2 <url>  # TLS 1.2
  curl -vI --tlsv1.3 <url>                # TLS 1.3
  ```
- **STDIO (local)**: TLS 1.3=No, TLS 1.2=No
- **Container**: TLS=No unless vendor docs say otherwise
- ⚠️ TLS applies to MCP transport layer only (not upstream APIs)

### Transport Protocol
- **STDIO** = local subprocess via stdin/stdout
- **HTTP/SSE** = Server-Sent Events endpoint
- **StreamableHttp** = modern HTTP transport
- ⚠️ Verify with `mcp.run()` call in code (don't infer from dependencies)

### Tools Operations
Mark **only the highest tier** as Yes:
- **Read-only** = ALL tools read-only
- **Read+Update** = read AND write/create/update (no delete)
- **Read+Update+Delete** = read AND write/update AND delete

### Deployment Approach
- **Local** = runs as local process (STDIO)
- **Container** = Dockerfile/docker-compose provided
- **Remote** = vendor-hosted endpoint
- ⚠️ Local + Container can both be Yes (self-hosted)
- ⚠️ If GitHub URL input: Local=Yes + STDIO=Yes permanent even if Remote=Yes

### Capabilities
- **Tools** = `@server.tool()` decorators or vendor lists tools
- **Resources** = `@server.resource()` decorators exist AND vendor confirms
- **Prompts** = `@server.prompt()` decorators or vendor mentions templates
- **Sampling** = server requests LLM completions (rare)

---

## Common Mistakes to Avoid

✗ Marking GitHub=No when repo IS on GitHub
✗ Both SaaS Vendor=Yes AND GitHub=Yes (only one primary)
✗ HTTP/SSE=Yes for STDIO-only server
✗ TLS=Yes based on upstream API HTTPS (not MCP transport)
✗ Remote=Yes without confirmed public endpoint URL
✗ Multiple Tools Operations rows =Yes (highest tier only)
✗ Resources=Yes from code alone (need vendor confirmation)
✗ Using "Partial" instead of Yes/No
✗ Inferring auth without checking docs/code

---

## Skill Handoff

```
attribute-researcher
  │
  ├─ Called by: mcp-researcher (Step 2: attribute filling)
  │
  └─ Returns: All attributes + tools + non-read-only list
     → mcp-researcher continues to scoring & CSV
```

---

## Output Format

Return to mcp-researcher:

```
{
  "attributes": {
    "Distribution": {"Official": "Yes", "Community": "No"},
    "Pricing": {"Free": "Yes", "Paid": "No"},
    ... (all attributes)
  },
  "evidence": {
    "each_yes_value": "quote or file_path"
  },
  "tools": "Category Name\n    • tool – description\n...",
  "non_read_only": "Category Name\n    • tool – description\n..."
}
```

Or to user (if standalone):

All attributes + evidence in Markdown table format
Tools grouped by category
Non-read-only operations listed
