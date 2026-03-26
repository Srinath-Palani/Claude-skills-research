# Multi-Server Parallel Research

🔒 **CRITICAL SYNC RULE** (LOCKED)
> This file must stay synchronized with `SKILL.md`
> Whenever SKILL.md is updated → verify this file needs updates too
> See `SYNC_GUIDE.md` for verification checklist before committing.

🎓 **SELF-LEARNING INTEGRATION (Unified MCP Skill 2.0)**
> Before each batch research:
> 1. Read `learned-fixes.md` — Error Patterns #1-4 (Protocol, Auth, Tools Ops, Deployment)
> 2. Apply verification checklists during research
> 3. Use pre-submission checklist before finalizing reports
> This prevents recurring mistakes across multi-server batches.

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
    "version": "v1.2.3",
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
- If data missing → use `"unknown"` not null
- If repo inaccessible → set `errors: ["repo not found"]`, return partial data
- Do not call other agents or tools outside README + file search + endpoint probe

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
Reports saved: ~/Desktop/MCP_reports/
```

**All 11 comparison columns:**
1. Version (git tag)
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
- New error patterns found → append to "Learned Fixes" section in SKILL.md

---

## Key Learnings Applied (Updated 2026-03-26)

**🔒 LOCKED RULES** — Use these for every multi-server research:

1. **TLS ≠ Bearer Token**
   - TLS only for HTTP/SSE/StreamableHttp endpoints
   - STDIO servers → TLS = STDIO N/A
   - Bearer Token in code ≠ automatic TLS presence

2. **Pricing = Server Licensing (Not Services)**
   - Free/Paid reflects MCP server itself
   - Don't confuse with downstream service costs
   - Ask: "Is the MCP server itself paid?"

3. **Tools Operations — Mark Highest Level Only**
   - Identify all operations present
   - Mark ONLY: Delete (highest) > Update > Read-only (lowest)
   - Other levels = No

4. **Capabilities Categories from User Research**
   - Use EXACT categories from research documentation
   - Never reorganize or rename
   - Priority: User research > README > Source code

5. **Non-Read-Only Tools — Include Only If Researched**
   - Don't fabricate this row
   - Include only if documented
   - Text: "None — all tools are read-only" if applicable

6. **Protocol Version — Numeric Comparison (Not Assumption)**
   - Extract exact version from source (package.json, pyproject.toml)
   - **DO NUMERIC COMPARISON** against SKILL.md mapping ranges
   - Example: SDK 1.7.0 < 1.8? YES → 2024-11-05 (NOT 2025-06-18)
   - **NEVER proceed without explicit verification**
   - Verify before writing to CSV

See "Learned Fixes" section in SKILL.md for detailed error examples and prevention checklist.

---

## Final Report Format (for Multi-Server Batch Research & Sessions)

When generating final reports for multi-server research projects, use this format:

```
╔════════════════════════════════════════════════════════════════════════════════════════╗
║                                                                                        ║
║                    ✅ ✅ ✅  REPORT SUCCESSFULLY GENERATED  ✅ ✅ ✅                 ║
║                                                                                        ║
║                                                                                        ║
║                                FILE LOCATION:                                         ║
║                                                                                        ║
║              /path/to/report/REPORT_NAME_DATE.txt                                     ║
║                                                                                        ║
║                                                                                        ║
╚════════════════════════════════════════════════════════════════════════════════════════╝
```

**Format Features:**
- Large, bold text (visible in all IDEs and terminals)
- Clear file location path with full file path
- Works across all platforms and editors
- Supports terminal and IDE display

**Usage:**
- Use for multi-server batch research final reports
- Use for project audit completions
- Use for session summaries
- Customize file location path as needed
