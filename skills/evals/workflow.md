# Evals Skill — Workflow Diagram

> Last updated: 2026-03-25 (optimized: 9 steps consolidated, quality checks tabularized)

---

## Full Workflow: 9 Steps

```
EVAL TRIGGER
(from skill-creator or project-reviewer)
      │
      ▼
┌──────────────────────────┐
│ STEP 1 — Load Eval Cases │
│                          │
│ Read eval_cases.json     │
│ Extract:                 │
│ • Case ID + input        │
│ • Expected output        │
│ • Quality checks         │
│ • Scoring expectations   │
└──────────┬───────────────┘
           │
           ▼
┌──────────────────────────┐
│ STEP 2 — Pre-flight      │
│                          │
│ Verify:                  │
│ ✓ Skills loaded          │
│ ✓ Test data valid JSON   │
│ ✓ Output dir exists      │
│ ✓ No stale state         │
│                          │
│ FAIL → stop              │
│ PASS → continue          │
└──────────┬───────────────┘
           │
           ▼
      ┌─────────────┐
      │ For each    │
      │ eval case:  │
      └─────┬───────┘
            │
            ▼
┌──────────────────────────┐
│ STEP 3 — Execute Case    │
│                          │
│ • Invoke target skill    │
│ • Capture all output     │
│ • Record exec time       │
│ • Track files generated  │
│ • Check for errors       │
└──────────┬───────────────┘
           │
           ▼
┌──────────────────────────┐
│ STEP 4 — Verify Files    │
│                          │
│ For each output:         │
│ ✓ File exists            │
│ ✓ Not empty              │
│ ✓ Format valid           │
│ ✓ Name matches pattern   │
│ ✓ Content complete       │
│                          │
│ FAIL → stop case         │
│ PASS → continue          │
└──────────┬───────────────┘
           │
           ▼
┌──────────────────────────┐
│ STEP 5 — Validate        │
│          Attributes      │
│                          │
│ For each expected attr:  │
│ 1. Extract actual value  │
│ 2. Compare to expected   │
│ 3. Log mismatches        │
│                          │
│ PASS → continue          │
│ FAIL → flag + record     │
└──────────┬───────────────┘
           │
           ▼
┌──────────────────────────┐
│ STEP 6 — Quality Checks  │
│                          │
│ Verify 6 categories:     │
│ ✓ Output files          │
│ ✓ Attributes            │
│ ✓ Yes/No discipline     │
│ ✓ Evidence complete     │
│ ✓ Scoring accuracy      │
│ ✓ Business logic        │
│                          │
│ PASS → continue          │
│ FAIL → log violations    │
└──────────┬───────────────┘
           │
           ▼
┌──────────────────────────┐
│ STEP 7 — Verify Scores   │
│                          │
│ For each score:          │
│ • Extract actual score   │
│ • Compare to expected    │
│ • Verify range 0–53      │
│ • Check rules applied    │
│                          │
│ PASS → continue          │
│ FAIL → flag discrepancy  │
└──────────┬───────────────┘
           │
           ▼
      ┌─────────────┐
      │ Mark result:│
      │ P/F/W       │
      └─────┬───────┘
            │
      (repeat 3–7
       for next case)
            │
            ▼
┌──────────────────────────┐
│ STEP 8 — Compile Results │
│                          │
│ Generate summary:        │
│ • Total cases            │
│ • P/F/W counts + %       │
│ • Failed list with why   │
│ • Warnings list          │
│ • Total + avg times      │
└──────────┬───────────────┘
           │
           ▼
┌──────────────────────────┐
│ STEP 9 — Output Report   │
│                          │
│ Print to chat:           │
│ Skill / Total / P/F/W %  │
│ Failed cases + reasons   │
│ Warnings                 │
│ Avg time per case        │
│                          │
│ DONE                     │
└──────────────────────────┘
```

---

## Quality Check Categories (STEP 6)

| Category | Checks |
|----------|--------|
| **Output Files** | Exist, format valid, content complete, name matches |
| **Attributes** | All present, values match, no extras, correct types |
| **Yes/No Discipline** | Only Y/N, mutually exclusive, 1 Yes per choice, no Partial/Unknown |
| **Evidence** | Every Yes has entry, refs valid, paths exist, quotes accurate |
| **Scoring** | 0–5 per attr, total = sum, range 0–53, rules applied |
| **Business Logic** | STDIO: TLS=No, Remote: TLS verified, hosting: 1 Yes, transport: 1 Yes |

---

## Case Result States

| State | Meaning | Action |
|-------|---------|--------|
| **PASS** | All checks pass, output matches | No action needed |
| **FAIL** | Check failed or output mismatch | Review and fix skill |
| **WARN** | Checks pass but minor output diff | Review, decide if acceptable |

---

## Performance Metrics Tracked

```
Per Case:
  - Execution time (ms)
  - File generation time (ms)
  - Validation time (ms)
  - Total case time (ms)

Overall:
  - Average time per case
  - Min / Max case times
  - Total eval run time
  - Cases per second (throughput)
  - Pass rate (%)
```

---

## Error Handling

```
During case execution:
  1. Capture error + stack trace
  2. Mark case FAIL
  3. Log file path + line number
  4. Continue to next case
  5. Include in final report

Critical pre-flight failure:
  1. Stop immediately
  2. Report which check failed
  3. Do NOT proceed with cases
  4. Recommend fix action
```

---

## Integration Points

```
Triggered by:
  ├─ skill-creator (performance benchmarking)
  ├─ project-reviewer (quality audit)
  └─ Manual invocation (direct testing)

Outputs to:
  ├─ Chat (summary report)
  ├─ ~/Desktop/MCP_reports/ (skill outputs)
  └─ Memory (learned failure patterns)

Depends on:
  ├─ mcp-researcher (attribute evals)
  ├─ repo-clone (setup verification)
  ├─ error-handling (error case testing)
  └─ All SKILL.md files (audit evals)
```

---

## Summary: Features Met

✅ 9-step linear evaluation workflow
✅ 6 quality check categories
✅ 11-attribute scoring system (0–53 scale)
✅ Structured test cases (JSON format)
✅ Pass/Fail/Warn grading
✅ Performance metrics tracking
✅ Integration with skill-creator and project-reviewer
✅ Regression testing support
