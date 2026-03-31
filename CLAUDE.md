# Claude-skills-research -- Unified MCP Skill v3.0

This project contains one **unified, optimized MCP skill** that consolidates research, deployment, error recovery, and project auditing into a single executable skill. Security hardened, token-optimized, 4-gate enforcement.

## Quick Start

Clone this repo, navigate to the directory, and the unified skill will load automatically:

```bash
git clone <repo-url> Claude-skills-research
cd Claude-skills-research
```

Then invoke the unified MCP skill with `/unified-mcp-skill-new` in Claude Code.

---

## Unified Skill: `/unified-mcp-skill-new`

**The all-in-one MCP skill** -- research and document attributes, conditionally set up locally, diagnose errors inline, run multi-server batch research, and audit projects -- all in one optimized execution path.

**Invocation:** `/unified-mcp-skill-new`

Use this skill for any MCP-related task:

| Task | Example |
|------|---------|
| **Research** | "Research the GitHub MCP server" |
| **Attribute Docs** | "Document attributes for this MCP: https://github.com/org/repo" |
| **Local Setup** | "Set up this server locally", "Clone and run the Slack MCP" |
| **Error Recovery** | "The MCP server won't connect" [paste error] |
| **Project Audit** | "Review the project", "Is this ready to commit?" |
| **Batch Research** | "Research these MCPs: GitHub, Slack, Linear", "Compare GitHub and Stripe MCPs" |

**Features:**
- Evidence-backed attribute documentation (Gate 1: Evidence Ledger)
- 4-gate enforcement: Evidence Ledger, Connection Verification, Learning Walkthrough, Self-Improvement
- Multi-server parallel research + comparison
- Conditional local setup (clone + install if user wants)
- 7-phase inline error recovery (auto-diagnosis on connection fail)
- SSRF protection, curl timeouts, path validation
- 3 input types: Remote endpoint / GitHub URL / Server name
- 4 connection methods: STDIO / Published Package / Docker / Remote
- Protocol version verification before research
- Security mandate enforced (no credentials in chat, always via file editing)
- CSV report output
- Token-optimized SSOT architecture (~37% fewer tokens vs v2.0)

---

## Project Structure (v3.0)

```
Claude-skills-research/
  CLAUDE.md                          <-- This file
  README.md                          <-- Public-facing readme
  marketplace.json                   <-- Marketplace config (v3.0.0)
  optimization_guide.md              <-- Full optimization reference
  .claude-plugin/
    plugin.json                      <-- Plugin config (v3.0.0)
  skills/
    README.md                        <-- Skills overview
    unified-mcp-skill-new/           <-- Main unified skill
      SKILL.md                       <-- All workflows, rules, gates (source of truth)
      references/
        learned-fixes.md             <-- Error case studies (#1-#4, #7-#8)
        multi-server.md              <-- Batch research orchestration
        csv-example.md               <-- CSV formatting reference
```

---

## Security & Confidentiality

All skills enforce strict security rules:

- **Never ask for credentials in chat** -- credentials are always entered via file editing
- **No hardcoded secrets** -- all config examples use `<PLACEHOLDER>` syntax
- **Credentials routed to local filesystem only** -- never transited through chat
- **Security mandates enforced at every step** -- no exceptions
- **SSRF protection** -- IP blocklist for endpoint probing (private ranges, localhost, metadata)
- **Curl safety** -- `--connect-timeout 5 --max-time 10 --max-redirs 3` on all probes
- **Path validation** -- reject `..`, symlinks, system dirs, other users' homes

---

## SSOT Architecture (v3.0)

```
SKILL.md (source of truth -- authoritative for ALL rules)
  |
  +-- references/multi-server.md (batch orchestration ONLY, references SKILL.md)
  |
  +-- references/learned-fixes.md (error case studies ONLY, references SKILL.md)
  |
  +-- references/csv-example.md (formatting reference ONLY)
```

**Rules are NEVER duplicated across files.** Each reference file points to SKILL.md for authoritative rules.

---

## Workflow: Unified MCP Skill

```
"Research the GitHub MCP server and set it up locally"

  Skill handles:
  1. Research: Fills all attributes with evidence (Gate 1: Evidence Ledger)
  2. Verification: All 5 threads complete (Gate 2: Connection Verification)
  3. Learning check: L1-L8 applied (Gate 3: Learning Walkthrough)
  4. Setup: Asks deployment preference (local/remote/research-only)
  5. Local Setup (if chosen): Clones, installs, configures
  6. Error Recovery (if needed): Diagnoses and fixes connection errors inline
  7. Report: Saves CSV to configured path
  8. Self-Improvement: Records new patterns (Gate 4)
```

**No context-switching.** One skill. One command. Complete workflow.

---

## Team Setup

When team members clone this repo:

1. They get the unified skill automatically (in `skills/unified-mcp-skill-new/`)
2. CLAUDE.md loads and shows available skills
3. They can invoke `/unified-mcp-skill-new` immediately
4. All functionality consolidated -- no version conflicts

**No additional setup needed.**

---

## References

- **MCP Spec:** https://spec.modelcontextprotocol.io
- **MCP Servers Directory:** https://github.com/modelcontextprotocol/servers
- **Output location:** `~/Documents/mcp-reports/` (configurable)
- **Cloned repos location:** Configurable (asked at clone time, see SKILL.md Step 3)
- **Self-learning:** `skills/unified-mcp-skill-new/references/learned-fixes.md`
- **Optimization guide:** `optimization_guide.md` (full v3.0 reference)

---

## Support & Feedback

For issues:
1. Use `/unified-mcp-skill-new` with error message to diagnose (built-in error recovery)
2. Check `skills/unified-mcp-skill-new/references/learned-fixes.md` for known solutions
3. Run project audit: `/unified-mcp-skill-new` -> "Review the project"

For feedback on Claude Code, visit: https://github.com/anthropics/claude-code/issues

---

## Session Learnings & Discovered Endpoints

### Known Remote MCP Endpoints (Confirmed via Probe)

| MCP Server | Endpoint URL | Auth | Notes |
|---|---|---|---|
| Amazon ECS MCP Server | `https://ecs-mcp.{region}.api.aws/mcp` | SigV4 (AWS IAM) | Multi-region. Requires `mcp-proxy-for-aws`. HTTP 403 on probe. |
| AWS Knowledge MCP Server | `https://knowledge-mcp.global.api.aws` | None | Global. No AWS account needed. POST → HTTP 200. |
| AWS MCP Server | `https://aws-mcp.us-east-1.api.aws/mcp` | SigV4 (AWS IAM) | Preview. `us-east-1` only — other regions return HTTP 000. Requires `mcp-proxy-for-aws`. |
| Microsoft Dataverse MCP Server | `https://<orgname>.crm.dynamics.com/api/mcp` | Entra app + `mcp.tools` | Org-specific URL. Also supports local proxy via `@microsoft/dataverse` npm. |

### STDIO-Only Servers (No Remote Endpoint — Confirmed)

All remaining AWS servers in the `awslabs/mcp` monorepo are STDIO-only (no managed endpoint):
Aurora DSQL, Aurora PostgreSQL, Bedrock AgentCore, Bedrock KB, CloudWatch App Signals,
Q Business (Anon), Q Index, Redshift, SNS/SQS, Timestream, Bedrock Data Automation,
Billing & Cost, CDK, Cloud Control API (deprecated), CloudFormation (deprecated).

**Bedrock AgentCore — Gateway false positive (verified 2026-03-31, Steps A–E):**
AgentCore **Gateway** is an AWS service that converts users' own APIs/Lambda functions into
MCP-compatible tools with managed endpoints. It does NOT provide a managed remote endpoint
for `awslabs.amazon-bedrock-agentcore-mcp-server` itself. The MCP server is local STDIO-only.
The `manage_agentcore_gateway` tool inside the server is documentation guidance only.
Rule: "Service X creates MCP endpoints" ≠ "MCP server for Service X has a managed endpoint".
See Error Pattern #19 in learned-fixes.md.

Microsoft STDIO-only servers (no remote endpoint):
- **Microsoft 365 Agents Toolkit**: `npx @microsoft/m365agentstoolkit-mcp@latest server start`
- **Microsoft Clarity**: `npx @microsoft/clarity-mcp-server`
- **Microsoft Dev Box**: `npx -y @microsoft/devbox-mcp@latest`

### Server Location Learnings

**Microsoft 365 Agents Toolkit** — NOT under `microsoft/TeamsFx` or `microsoft/teams-toolkit`.
Correct location: `OfficeDev/microsoft-365-agents-toolkit` monorepo → `packages/mcp-server/`.
npm package: `@microsoft/m365agentstoolkit-mcp`. Search by npm package when GitHub search returns 0.

**AWS managed MCP pattern** — Pattern for AWS-hosted endpoints: `https://{service}-mcp.{region}.api.aws/mcp`.
Not all AWS services have managed versions — check the ECS developer guide style docs per service.
AWS MCP Server unified endpoint: `https://aws-mcp.us-east-1.api.aws/mcp` (listed at `awslabs.github.io/mcp/`).

**Microsoft MCP servers span two orgs** — `microsoft/` (Clarity, Dev Box, Dataverse, Playwright) and
`OfficeDev/` (365 Agents Toolkit). Always check both orgs when searching for Microsoft MCP servers.

**Microsoft Dataverse MCP Server — false positive GitHub URL** — `https://github.com/microsoft/Dataverse-skills`
is NOT the Dataverse MCP server repo. It is a Power Platform / Copilot Studio skill templates repository
(contains `.yml` skill definition files, no MCP SDK). The Dataverse MCP Server has no separate public GitHub
source repo — it is accessed via `https://<orgname>.crm.dynamics.com/api/mcp` (remote endpoint) or
`@microsoft/dataverse` npm package (local proxy). Always verify a GitHub URL contains MCP SDK dependencies
before accepting it as the server repo. See Error Pattern #19 in learned-fixes.md.

---

## Last Updated: 2026-03-31
**Status:** Production Ready | **Version:** 3.0.0 | **SSOT Architecture**
