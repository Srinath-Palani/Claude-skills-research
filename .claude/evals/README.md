# attribute-researcher Skill — Evals

## Purpose

These eval cases test the **MCP Server Researcher + Security Scorer** skill for correctness,
completeness, and scoring accuracy.

## Files

| File | Description |
|------|-------------|
| `eval_cases.json` | Structured test cases with inputs, expected attribute values, and scoring checks |

## Eval Cases

| ID | Input | What it tests |
|----|-------|---------------|
| `github-mcp-official` | GitHub MCP server | Official STDIO server with PAT auth, TLS N/A scoring |
| `cloudflare-mcp-remote` | Cloudflare MCP server | Remote server with OAuth + StreamableHttp + TLS 1.3 |
| `postgres-mcp-community` | PostgreSQL MCP server | Read-only tools, STDIO, community server scoring |

## Quality Checks

Each eval verifies:
- Report files saved to `~/Desktop/MCP_reports/` in both `.md` and `.csv` formats
- Mutual exclusivity rules respected (one transport, one protocol version, one hosting provider)
- Yes/No discipline (no Partial, Unknown, N/A in attribute fields)
- Every Yes has an entry in the Evidence Log
- Security score total matches sum of individual scores
- TLS scored correctly for STDIO vs remote servers

## Running Evals

Use the `skill-creator` skill to run these evals and measure skill performance:

> "Run evals for the attribute-researcher skill"
