# Evals Skill References

---

## eval_cases.json

Structured test cases for evaluating skill performance and correctness.

### Case: github-mcp-official

**Purpose:** Test official STDIO server with PAT authentication

**Input:** Research the GitHub MCP server at https://github.com/github/github-mcp-server

**Expected Attributes:**
- Distribution: Official = Yes, Community = No
- Transport: STDIO = Yes, HTTP/SSE = No, StreamableHttp = No
- Authentication: Personal Access Token = Yes
- Deployment: Local = Yes, Remote = No
- Data Protection: TLS 1.3 = No, TLS 1.2 = No

**Expected Output Files:**
- `~/Desktop/MCP_reports/github-mcp-server-report.md`
- `~/Desktop/MCP_reports/mcp_inspection_report_github-mcp-server_*.csv`

**Scoring Expectations:**
- Transport score: 3 (STDIO = 2)
- Distribution score: 5 (Official)
- TLS score: 3 (N/A for STDIO)

---

### Case: cloudflare-mcp-remote

**Purpose:** Test remote endpoint with OAuth and StreamableHttp

**Input:** Research the Cloudflare MCP server

**Expected Attributes:**
- Distribution: Official = Yes, Community = No
- Transport: STDIO = No, StreamableHttp = Yes
- Authentication: OAuth 2.1 Auth Code Flow = Yes
- Deployment: Remote = Yes, Local = No
- Data Protection: TLS 1.3 = Yes

**Expected Output Files:**
- `~/Desktop/MCP_reports/cloudflare-mcp-server-report.md`
- `~/Desktop/MCP_reports/mcp_inspection_report_cloudflare_*.csv`

**Scoring Expectations:**
- Transport score: 5 (StreamableHttp)
- Auth score: 3 (OAuth 2.1)
- TLS score: 5 (TLS 1.3)
- Distribution score: 5 (Official)

---

### Case: postgres-mcp-community

**Purpose:** Test official server with read-only tools via STDIO

**Input:** Research the MCP server for PostgreSQL at https://github.com/modelcontextprotocol/servers/tree/main/src/postgres

**Expected Attributes:**
- Distribution: Official = Yes, Community = No
- Transport: STDIO = Yes, HTTP/SSE = No, StreamableHttp = No
- Deployment: Local = Yes, Remote = No
- Data Protection: TLS 1.3 = No, TLS 1.2 = No
- Tools Operations: Read-only = Yes, others = No

**Expected Output Files:**
- `~/Desktop/MCP_reports/postgres-report.md`
- `~/Desktop/MCP_reports/mcp_inspection_report_postgres_*.csv`

**Scoring Expectations:**
- Transport score: 3 (STDIO)
- Tools Operations score: 5 (Read-only)

---

## Quality Checks Reference

All evals verify these quality requirements:

### Output File Checks
```
✓ Report saved to ~/Desktop/MCP_reports/ as both .md and .csv
✓ CSV uses three-column format: Category, Attribute, Status
✓ Markdown report includes all sections
✓ File naming follows pattern: mcp_inspection_report_*.csv
```

### Data Discipline
```
✓ Only one Tools Operations row is Yes
✓ Only one MCP Protocol Version row is Yes
✓ Only one Hosting Provider is Yes
✓ Official and Community are mutually exclusive
✓ No use of Partial, Unknown, or N/A in Yes/No fields
```

### Evidence & Accuracy
```
✓ Every Yes has evidence in the Evidence Log
✓ Evidence correctly cites sources
✓ No inference without source backing
✓ Quote accuracy verified
```

### Scoring Validation
```
✓ Security score total matches sum of individual scores
✓ STDIO servers have TLS marked No (scored as N/A = 3)
✓ Remote HTTPS servers have TLS 1.3 = Yes
✓ GitHub not marked as hosting provider when vendor hosts runtime
✓ Score total is valid (0–53 range)
```

### Business Logic
```
✓ CSV written using Python csv.writer with QUOTE_ALL
✓ Transport, Protocol, and Hosting Provider are mutually exclusive
✓ Distribution type (Official/Community) is mutually exclusive
✓ All field combinations are valid per MCP spec
```

---

## Scoring Rules Reference

### Security Confidence Score Calculation

**Total points available: 53**

| Attribute | Max Score | Rules |
|-----------|-----------|-------|
| **Protocol Version** | 5 | Current = 5, 1 behind = 3, older = 1 |
| **Distribution** | 5 | Official = 5, Community = 3 |
| **Pricing** | 3 | Free = 3, Paid = 1 |
| **Hosting** | 5 | SaaS = 5, GitHub = 3, Self = 1 |
| **Authentication** | 5 | OAuth 2.1 = 5, Bearer/PAT = 3, API Key = 2, None = 1 |
| **Data Protection (TLS)** | 5 | TLS 1.3 = 5, TLS 1.2 = 3, N/A (STDIO) = 3, None = 1 |
| **Transport** | 5 | StreamableHttp = 5, HTTP/SSE = 3, STDIO = 2 |
| **Tools Operations** | 5 | Read+Update+Delete = 5, Read+Update = 3, Read-only = 1 |
| **Deployment** | 3 | Remote vendor = 3, Local/Container = 2, Complex = 1 |
| **Capabilities** | 3 | 4+ types = 3, 2-3 types = 2, 1 type = 1 |

---

## Common Eval Mistakes

### Scoring Errors
- ❌ Scoring STDIO servers as TLS 1.3 = No, TLS 1.2 = No (0 points)
- ✓ Correct: Both No = N/A = 3 points (acknowledged not applicable)

- ❌ Marking both SaaS Vendor = Yes AND GitHub = Yes
- ✓ Correct: Only one hosting provider Yes (mutually exclusive)

- ❌ Total score = 100 or higher
- ✓ Correct: Total must be ≤ 53 (sum of attribute max scores)

### Attribute Errors
- ❌ Using "Partial", "Unknown", "N/A" in attribute fields
- ✓ Correct: Only "Yes" or "No" allowed

- ❌ Not backing every Yes with evidence
- ✓ Correct: Each Yes must have source quote or file path

### Mutual Exclusivity Violations
- ❌ Multiple rows Yes in single-choice categories
- ✓ Correct: Only one of {Transport STDIO, HTTP/SSE, StreamableHttp} = Yes

- ❌ Both Official = Yes AND Community = Yes
- ✓ Correct: Only one distribution type = Yes

- ❌ Both Protocol versions marked Yes
- ✓ Correct: Only one MCP protocol version = Yes

### Output Format Errors
- ❌ Not saving both .md and .csv reports
- ✓ Correct: Always save both formats to ~/Desktop/MCP_reports/

- ❌ CSV not using three-column format
- ✓ Correct: Category | Attribute | Status (or Yes/No)

- ❌ CSV not using QUOTE_ALL
- ✓ Correct: Use Python csv.writer with quoting=csv.QUOTE_ALL

---

## When to Add New Eval Cases

Add a new eval case when:
1. Adding support for a new server type (e.g., container-only)
2. Testing a new authentication method
3. Testing edge cases that existing cases don't cover
4. Adding a new quality check that needs verification
5. Testing integration between skills

**Do not add** redundant cases that only repeat existing validation.

---

## Integration with Skill Tests

```
evals (this skill)
├─ attribute-researcher evals
│  └─ Tests Yes/No discipline + scoring
│
├─ project-reviewer evals
│  └─ Tests audit completeness
│
├─ error-handling evals
│  └─ Tests error classification + fixes
│
└─ mcp-researcher evals
   └─ Tests full workflow end-to-end
```
