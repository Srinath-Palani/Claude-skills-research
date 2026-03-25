# project-reviewer Skill — Workflow Diagram

> Last updated: 2026-03-25 (optimized: 4-step simplified flow, check tables)

---

## Full Workflow

```
USER TRIGGERS REVIEW
  │
  ├── "review the project" / "audit all skills"     ──► Mode A (Full)
  ├── "review <skill-name>"                         ──► Mode B (Single)
  └── "review my changes" / "check uncommitted"    ──► Mode C (Git Diff)
              │
              ▼
┌─────────────────────────────────────────────┐
│  STEP 1 — Pre-load Known Gap Patterns       │
│                                             │
│  Read references/known-gap-patterns.md      │
│  Keep patterns in mind throughout review    │
└──────────────────┬──────────────────────────┘
                   │
                   ▼
┌─────────────────────────────────────────────┐
│  STEP 2 — Determine Mode & Read Files       │
│                                             │
│  Mode A: All SKILL.md + workflow.md         │
│  Mode B: Named skill's files only           │
│  Mode C: git diff HEAD changes + pairs      │
│                                             │
│  Do NOT apply checks until all read         │
└──────────────────┬──────────────────────────┘
                   │
                   ▼
┌─────────────────────────────────────────────┐
│  STEP 3 — Apply Review Checks               │
│                                             │
│  GROUP 1: Project Goal Alignment (G1–G12)  │
│  GROUP 2: Enterprise Security (S1–S13)     │
│  GROUP 3: Confidentiality (C1–C7)          │
│  GROUP 4: Cross-Skill Consistency (X1–X8)  │
│  GROUP 5: Completeness per Skill           │
│  GROUP 6: Skill Contribution (P1–P10)      │
│                                             │
│  Score each: PASS / FAIL / WARN            │
│                                             │
│  Mode C only:                               │
│  • Scan diff for credential patterns        │
│  • Check workflow.md freshness              │
└──────────────────┬──────────────────────────┘
                   │
                   ▼
┌─────────────────────────────────────────────┐
│  STEP 4 — Print Report & Self-Upgrade      │
│                                             │
│  Print to chat (NO file written):           │
│  • Review mode, date, files                 │
│  • Group results with PASS/FAIL/WARN       │
│  • Summary: total, counts, verdict         │
│  • Gap list (FAIL only)                    │
│  • Recommendation list (WARN only)         │
│                                             │
│  Skill 2.0: If new gap found               │
│  → append known-gap-patterns.md             │
│  → consider adding new check row            │
└─────────────────────────────────────────────┘
```

---

## Review Mode Decision

```
Trigger text
    │
    ├─ "review the project" / "audit all"  ────► Mode A (Full)
    │
    ├─ "review mcp-server" / named skill   ────► Mode B (Single)
    │
    ├─ "review my changes" / uncommitted   ────► Mode C (Git Diff)
    │  → git diff HEAD --name-only
    │  → git diff HEAD (raw)
    │
    └─ just edited a skill ────────────────────► Mode C + offer A
```

---

## Check Groups Summary

| Group | Name | Checks | Scope |
|-------|------|--------|-------|
| 1 | Project Goal Alignment | G1–G12 | All skills |
| 2 | Enterprise Security | S1–S13 | mcp-server + auth skills |
| 3 | Confidentiality | C1–C7 | All skills |
| 4 | Cross-Skill Consistency | X1–X8 | Full review + handoff changes |
| 5 | Completeness | Per-skill checklist | Each skill |
| 6 | Skill Contribution | P1–P10 | All skills |

---

## Score Meanings

```
PASS  — Requirement clearly met with evidence found
WARN  — Requirement present but weakly stated; improvement recommended
FAIL  — Requirement absent or violated; must fix before production
```

---

## Output Rules

```
✓ Print to chat ONLY — never write a file
✓ Do not fix issues — report only
✓ Credential scan runs in EVERY Mode C review
✓ WARN ≠ FAIL — warn items are improvements only
✓ Workflow freshness check (G11) after every SKILL.md change
✓ Skill 2.0: Update known-gap-patterns.md with new findings
```

---

## Skill Handoff Map

```
project-reviewer
  ├─ triggered by: user OR after editing any skill
  ├─ reads: all SKILL.md + workflow.md
  │         known-gap-patterns.md (pre-load)
  │         error-handling/references/*.md
  ├─ writes: known-gap-patterns.md (Skill 2.0 only)
  └─ calls: NO other skills (audit only)
```

---

## Key Enforcement Rules

| Situation | Action |
|-----------|--------|
| User says "review project" | Mode A: read all files |
| User says "review repo-clone" | Mode B: read only repo-clone files |
| User says "review my changes" | Mode C: git diff + credential scan |
| SKILL.md updated, workflow.md not | WARN: G11 (workflow freshness) |
| Credential pattern in diff | FAIL: potential credential leak |
| New FAIL/WARN type discovered | Append known-gap-patterns.md + consider new check |

---

## Summary: Features Met

✅ Four-step linear flow (pre-load, mode, apply, report)
✅ 45+ checks across 6 groups with standardized scoring
✅ Three review modes with proper file selection logic
✅ Credential detection and workflow freshness validation
✅ Self-learning via known-gap-patterns.md (Skill 2.0)
✅ Chat-only output, no file modifications
✅ Comprehensive handoff validation across skills
