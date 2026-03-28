<!-- SKILL_VERSION: 3.0.0 — must match SKILL.md version -->
# Multi-Server Parallel Research

🔒 **SKILL MODIFICATION POLICY — ABSOLUTE ENFORCEMENT**

When modifying ANY part of this file (format, rule, instruction, workflow step, output template):

1. **Read the full context first** — understand how the existing instruction works before changing it
2. **Change only what was asked** — never silently remove working logic, calculation methods, enforcement rules, or checklists
3. **Preserve the working method** — if changing a format or presentation, keep the underlying rules and instructions intact
4. **No collateral removal** — dropping an instruction while reformatting its container is a violation

---

🚫 **ZERO-ASSUMPTION POLICY applies to EVERY server in the batch.**

🔒 **MANDATORY before any batch research:**
> 1. Follow SKILL.md Gates 1-4 (Evidence Ledger, Connection Verification, Learning Gate, Self-Improvement)
> 2. All learnings (L1–L10) are embedded in SKILL.md Step 5.1–5.13 — follow each section's rules directly
> 3. Read `learned-fixes.md` all error patterns (#1-#4, #7-#16) for real case studies
> 4. Each Layer 1 agent MUST build its own Evidence Ledger — no attribute without source proof
> 5. If an agent cannot verify an attribute → return `"No"` and flag, never `"UNVERIFIED"` or `"unknown"`
> SKILL.md is the single source of truth — rules are not duplicated here.

**Trigger:** Auto-detected via Step 0-M (see below). No explicit command needed.

**Architecture:** 2-layer sub-agent system. Token-bounded. Parallel per server. No scoring.

---

## Step 0-M — Multi-Server Detection

Before any research begins, check if input contains multiple servers.

**Detection patterns (any match → enter Multi-Server mode):**

| Pattern | Example |
|---------|---------|
| Comma-separated | "GitHub MCP, Slack MCP, Notion MCP" |
| "and"-separated | "GitHub MCP and Slack MCP and Notion MCP" |
| Numbered list | "1. GitHub MCP\n2. Slack MCP" |
| JSON array | `["github-mcp", "slack-mcp"]` |
| Explicit keyword | "research X, Y, Z in parallel" |

**If 1 server detected** → continue with standard single-server workflow (Step 0 onward).
**If 2+ servers detected** → enter Multi-Server mode (Steps M1–M3 below).

---

## Layer 0 — Coordinator

**Step M1: Parse & Classify**
Extract all server identifiers. Classify each (name / GitHub URL / endpoint).
Resolve names to GitHub URLs before dispatching.

Confirm list before proceeding:
```
Found [N] servers to research:
1. GitHub MCP → github.com/github/github-mcp-server
2. Slack MCP  → github.com/slack/slack-mcp-server

[ 1 ] Proceed
[ 2 ] Adjust list
```

Limit: 10 servers max per batch. If more → ask user to split.

**Step M2: Dispatch Layer 1 Agents (parallel)**
Launch one Research Agent per server simultaneously.
Pass each agent: server URL + attribute schema + JSON output format.
Do NOT wait for one to finish before starting others.

**Step M3: Finalize**
Once ALL Layer 1 results received → render individual CSVs + comparison table.

---

## Layer 1 — Research Agent (per server, token budget: ~1,500)

Each agent runs the **existing Step 0.5 dual-source workflow** (5 concurrent threads) for its assigned server, then returns ONLY this JSON structure (no prose):

```json
{
  "server": "GitHub MCP",
  "repo": "github.com/github/github-mcp-server",
  "attributes": {
    "description": "...",
    "version": "v1.2.3",   // "NA" if no releases/tags/package.json found, or if only SNAPSHOT/pre-release exists
    "category": "Developer Tools",
    "distribution": "Official",
    "protocol_version": "2025-06-18",
    "framework": "mcp 1.15",
    "pricing": "Free",
    "hosting": "GitHub",
    "auth": "Personal Access Token",
    "tls": "STDIO N/A",
    "transport": "STDIO",
    "tools_ops": "R+Update+Delete",
    "deployment": "Local",
    "remote": "No",
    "remote_endpoint": "N/A",
    "capabilities": ["Tools"]
  },
  "evidence": ["README line 12", "pyproject.toml line 5"],
  "errors": []
}
```

**Agent rules:**
- Output JSON only — no explanations, no markdown
- **ZERO-ASSUMPTION:** If data missing → return `"No"` and flag — never `"UNVERIFIED"`, never `"unknown"`, never guess
- If repo inaccessible → set `errors: ["repo not found"]`, return partial data with UNVERIFIED fields
- Do not call other agents or tools outside README + file search + endpoint probe
- **Evidence required:** Every attribute value MUST include source in `"evidence"` array (file + line or URL)
- **Learning gate:** Apply all Learnings from SKILL.md Step 5.1–5.13 (L1→Step 5.3, L3→Step 5.7, L4→Step 5.4, L5→Step 5.9, L6→Step 5.13, L7→Step 5.13, L8→Step 5.1, L9→Step 5.12, L10→Step 5.13)
- **SSRF protection:** Validate all probe URLs against SKILL.md Security Mandate blocklist before probing
- **Curl safety:** All probes use `--connect-timeout 5 --max-time 10 --max-redirs 3`
- **Rate limit:** Max 3 attempts per endpoint, 2-second delay between retries

---

## Layer 0 — Comparison Summary Output

Show ALL attributes for each server in a full comparison table:

```
MULTI-SERVER RESEARCH SUMMARY — [date]
════════════════════════════════════════════════════════════════════════════════════════════════════════
Server          | Version | Category         | Protocol    | Hosting  | Auth         | TLS     | Transport    | Tools Ops    | Deployment | Remote | Dist
----------------|---------|------------------|-------------|----------|--------------|---------|--------------|--------------|------------|--------|------
GitHub MCP      | v1.2.3  | Developer Tools  | 2025-06-18  | GitHub   | PAT          | STDIO/NA| STDIO        | R+U+D        | Local      | No     | Official
Slack MCP       | v2.0.1  | Productivity     | 2025-03-26  | SaaS     | OAuth 2.1    | TLS 1.3 | HTTP/SSE     | R+U          | Remote     | Yes    | Official
Linear MCP      | v0.9.0  | Productivity     | 2024-11-05  | GitHub   | API Token    | STDIO/NA| STDIO        | Read-only    | Local      | No     | Community
════════════════════════════════════════════════════════════════════════════════════════════════════════
Default sort: Protocol Version (newest first)
Sort options: protocol / hosting / auth / transport / deployment
Reports saved: ~/Documents/mcp-reports/
```

**All 11 comparison columns:**
1. Version (git tag — "NA" if no releases/tags/package.json, or SNAPSHOT/pre-release only)
2. Category (Developer Tools / Productivity / Data Retrieval / File Management)
3. Protocol (MCP protocol version)
4. Hosting (SaaS / GitHub)
5. Auth (OAuth 2.1 / Bearer / PAT / API Token)
6. TLS (TLS 1.3 / TLS 1.2 / STDIO N/A)
7. Transport (STDIO / HTTP/SSE / StreamableHttp)
8. Tools Ops (Read-only / R+U / R+U+D)
9. Deployment (Local / Container / Remote)
10. Remote (Yes / No)
11. Dist (Official / Community)

After summary, prompt:
```
What next?
[ 1 ] Set up one of these locally
[ 2 ] View full report for a specific server
[ 3 ] Re-sort by different criteria
[ 4 ] Done
```

---

## Error Handling
- Server unreachable → log error, continue batch, mark as "ERROR" in summary
- Duplicate servers in list → deduplicate silently, note once
- New error patterns found → append to `learned-fixes.md`

---

## Key Learnings Reference

> **All Learnings (L1–L10) are embedded in SKILL.md Step 5.1–5.13.** They are the single source of truth.
> **All error case studies are in `learned-fixes.md` (Errors #1-#4, #7-#16).**
> Read both before starting any multi-server batch research. Do not duplicate rules here.

---

## Final Report Format

> Use the Final Report Format box defined in SKILL.md for all batch research reports.
