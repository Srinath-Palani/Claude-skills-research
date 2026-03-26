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
- **Output location:** `~/Desktop/MCP_reports/` (configurable)
- **Cloned repos location:** `~/MCP_repos/<repo-name>` (configurable)
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

## Last Updated: 2026-03-27
**Status:** Production Ready | **Version:** 3.0.0 | **SSOT Architecture**
