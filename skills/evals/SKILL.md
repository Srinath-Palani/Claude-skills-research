---
name: evals
user-invocable: false
description: >
  Test cases and evaluation framework for all skills in the claude-skills project.
  Not user-invocable; provides eval cases for measuring skill performance, correctness,
  and completeness. Used by project-reviewer and skill-creator to run quality checks
  and benchmarks.
---

# Evals — Test Cases & Quality Framework

Provides structured test cases and quality checks for all claude-skills skills.
Used by **project-reviewer** (audits) and **skill-creator** (benchmarking).

## Feature Overview

✓ Structured test cases with expected inputs/outputs
✓ Quality checks for correctness and completeness
✓ Performance metrics (execution time, accuracy)
✓ Regression testing for behavior changes
✓ Scoring validation per MCP attribute rules
✓ Integration with project-reviewer and skill-creator

---

## Test Coverage

### attribute-researcher Skill Evals

| Case ID | Input | Tests |
|---------|-------|-------|
| `github-mcp-official` | GitHub MCP server | Official STDIO, PAT auth, TLS N/A scoring |
| `cloudflare-mcp-remote` | Cloudflare MCP | Remote OAuth, StreamableHttp, TLS 1.3 |
| `postgres-mcp-community` | PostgreSQL MCP | Read-only tools, STDIO, community scoring |

---

## Quality Checks: 6 Categories

| Category | Checks |
|----------|--------|
| **Output Files** | Exist, format valid, content complete, naming matches |
| **Attributes** | All present, values match expected, no extras, correct types |
| **Yes/No Discipline** | Only Yes/No used, mutually exclusive enforced, single-choice has 1 Yes, no Partial/Unknown/N/A |
| **Evidence** | Every Yes has entry, references valid, paths exist, quotes accurate |
| **Scoring** | 0–5 per attribute, total = sum, total in range 0–53, rules applied correctly |
| **Business Logic** | STDIO: TLS=No, Remote: TLS verified, hosting: 1 Yes, transport: 1 Yes, no invalid combos |

---

## Scoring System: 11 Attributes

| Attribute | Max | Scoring Rules |
|-----------|-----|--------|
| Protocol Version | 5 | Current=5, 1 behind=3, older=1 |
| Distribution | 5 | Official=5, Community=3 |
| Pricing | 3 | Free=3, Paid=1 |
| Hosting | 5 | SaaS Vendor=5, GitHub=3, self-hosted=1 |
| Auth | 5 | OAuth 2.1=5, Bearer/PAT=3, API Key=2, None=1 |
| TLS | 5 | 1.3=5, 1.2=3, N/A (STDIO)=3, None=1 |
| Transport | 5 | StreamableHttp=5, HTTP/SSE=3, STDIO=2 |
| Operations | 5 | R+U+D=5, R+U=3, Read-only=1 |
| Deployment | 3 | Remote vendor=3, Local/Container=2, Complex=1 |
| Capabilities | 3 | 4+ types=3, 2-3=2, 1=1 |
| **TOTAL** | **53** | Sum of all |

---

## Workflow: 9 Steps

### Step 1 — Load Eval Cases
Read `eval_cases.json`. Extract: case ID, input, expected output, quality checks, scoring expectations.

### Step 2 — Pre-flight Checks
Verify:
- All required skills loaded
- Test data files valid JSON
- Output dir `~/Desktop/MCP_reports/` exists
- No leftover state from previous runs

Stop if any pre-flight check fails.

### Step 3–7 — Execute Each Case

For each eval case:
1. **Execute** — Invoke target skill with test input, capture output, record time
2. **Verify Files** — Check existence, format, content completeness, naming
3. **Validate Attributes** — Extract values, compare to expected, log differences
4. **Quality Checks** — Run all 6 check categories, flag violations
5. **Verify Scores** — Compare actual vs expected scores, verify range 0–53

**Result per case:** PASS (all checks pass), FAIL (any check fails), WARN (mixed).

### Step 8 — Compile Results

Generate summary:
- Total cases run
- Passed / Failed / Warned counts + percentages
- List failed cases with reasons
- Total exec time + avg time per case

### Step 9 — Output Report

Print to chat:

```
Eval Results Summary
════════════════════
Skill:       <skill-name>
Total cases: <N>
PASS:        <N> (<percent>%)
FAIL:        <N> (<percent>%)
WARN:        <N> (<percent>%)
Avg time:    <Xms> per case

Failed cases:
  1. case-id: reason

Warnings:
  - warning desc
```

---

## Test Case Structure

```json
{
  "id": "case-identifier",
  "input": "User input / skill trigger",
  "expected": {
    "attribute_field": "expected_value",
    "output_files": {
      "markdown": "path/to/expected/report.md",
      "csv": "path/to/expected/report.csv"
    }
  },
  "scoring_checks": {
    "field_name": expected_score_value
  }
}
```

---

## Common Eval Mistakes

✗ Scoring STDIO as TLS 1.3=No, 1.2=No (correct: both No = N/A = 3 points)
✗ Marking both SaaS Vendor=Yes AND GitHub=Yes (only one hosting provider allowed)
✗ Using "Partial" or "Unknown" instead of Yes/No (only Yes/No allowed)
✗ Not verifying every Yes has evidence entry
✗ Allowing multiple Yes in mutually exclusive categories
✗ Not saving both .md and .csv reports
✗ Mismatching security score total vs individual sum

---

## Files

| File | Purpose |
|------|---------|
| `SKILL.md` | This file — eval metadata |
| `workflow.md` | Workflow diagram + detailed steps |
| `REFERENCE.md` | Reference docs + test case details |
| `eval_cases.json` | Structured test cases with expected outputs |
| `quality_checks.md` | Detailed quality check specs |

---

## Integration with Other Skills

- **project-reviewer** — Reads evals to verify skill quality during audits
- **skill-creator** — Runs evals to measure skill performance
- **mcp-researcher** — Tested against eval cases for accuracy
- **attribute-researcher** — Tested against attribute field rules
- **error-handling** — Tested for error classification and fix success rate

---

## Adding New Evals

1. Identify skill and scenario being tested
2. Define clear input and expected output
3. List all quality checks that should pass
4. Include scoring checks with expected point values
5. Add case to `eval_cases.json` with proper structure
6. Document case in this SKILL.md

---

## Skill Handoff Map

```
evals
  ├─ triggered by: project-reviewer (audit) OR skill-creator (benchmark)
  ├─ reads: eval_cases.json, quality_checks.md
  ├─ writes: ~/Desktop/MCP_reports/* (skill outputs being tested)
  └─ calls: All skills in claude-skills project (invokes each for testing)
```

---

## Summary

9-step evaluation framework with 6 quality check categories, 11-attribute scoring system,
and structured test cases. Supports regression testing, performance benchmarking, and
quality audits. 0–53 point scoring scale per MCP server attributes.
