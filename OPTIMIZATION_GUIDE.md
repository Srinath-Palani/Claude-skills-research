# Skills Optimization Guide

This guide shows how to apply the same optimization logic used for `mcp-researcher` to all remaining skills.

---

## Already Optimized ✅

- **mcp-researcher** (1,265 → 650 lines: 50% reduction)
  - Removed repetition
  - Added feature overview
  - Consolidated rules into tables
  - Streamlined workflow diagrams

- **attribute-researcher** (SKILL.md partially optimized)
  - SKILL.md: 257 → 165 lines (36% reduction)
  - Still needs: workflow.md optimization

---

## Remaining Skills to Optimize

### 1. attribute-researcher (workflow.md only)

**Current:** 138 lines
**Target:** 100 lines (28% reduction)

**Optimization steps:**
1. Consolidate 5-step workflow into single flow diagram
2. Remove repeated explanations
3. Add tables for attribute types
4. Add key enforcement rules section

**Key changes:**
- Combine "Fetch Sources" + "Fill Attributes" steps
- Single evidence requirement statement (not repeated)
- Table: Attribute | Rule | Mutually Exclusive
- Table: Mistake | Solution

---

### 2. repo-clone (SKILL.md + workflow.md)

**Current:** 286 + 205 = 491 lines
**Target:** 320 lines (35% reduction)

**SKILL.md optimization:**
- Remove repeated "repo-clone prerequisites" mentions
- Consolidate connection methods into single table
- Step count: 8 → 5 (merge related steps)
- Add feature overview (clone, install, config, verify, return)

**workflow.md optimization:**
- Remove duplicate "extract command/args/env" boxes
- Single flow instead of branching paths
- Add key checkpoints table

**Key sections to consolidate:**
- Clone + verify into one section
- Dependencies install rules (Python/Node/Docker) into table
- Common mistakes into table format

---

### 3. error-handling (SKILL.md + workflow.md)

**Current:** 461 + 292 = 753 lines
**Target:** 450 lines (40% reduction)

**SKILL.md optimization:**
- Phase count: 7 phases but many repeated diagnostics
- Consolidate "Phase 3" diagnosis steps into checklist table
- Combine Phase 5 fixes by category (create one section per error type)
- Error classification table instead of text
- Remove repeated "ask user approval" instructions

**workflow.md optimization:**
- Multi-path workflow into sequential flow
- Phase boxes are repetitive - single template with input/output
- Error signals table instead of nested definitions

**Key consolidations:**
- Table: Error Signal | Category | Go to Fix
- Table: Error Type | Diagnosis | Solution
- Single "Get User Approval" section (referenced, not repeated)
- Phases → Checklist format

---

### 4. project-reviewer (SKILL.md + workflow.md)

**Current:** 451 + 252 = 703 lines
**Target:** 420 lines (40% reduction)

**SKILL.md optimization:**
- Check Groups: 6 groups with 45+ checks
- Create master check table instead of individual sections
- Consolidate common patterns
- Use check ID references instead of full text

**workflow.md optimization:**
- Current: 7-phase detailed flow
- Target: 4-step flow with decision tree
- Mode selection (Full/Single/Diff) simplified

**Key consolidations:**
- Table: Check ID | Requirement | Pass Condition | Fail Message
- Single "Apply Check" procedure (referenced 45+ times)
- Three review modes → single template with parameters
- Remove repeated "output format" instructions

---

### 5. evals (SKILL.md + workflow.md)

**Current:** 170 + 313 = 483 lines
**Target:** 300 lines (38% reduction)

**SKILL.md optimization:**
- Eval cases: 3 cases with repetitive structure
- Create template: Single case format, then list parameters
- Quality checks: 12+ checks - consolidate into table

**workflow.md optimization:**
- 10 steps in current workflow
- Merge into 6 key phases
- Add case parameter table

**Key consolidations:**
- Table: Test Case | Purpose | Input | Expected Output
- Table: Quality Check | Verification | Pass/Fail
- Single eval case template (reference 3 times with different params)
- Scoring rules → reference table

---

## Optimization Template

Use this template for each skill:

### SKILL.md Structure

```markdown
---
name: skill-name
user-invocable: [true/false]
description: > [one-liner]
---

# Skill Title

> Full workflow diagram: see `workflow.md`

Brief description (2-3 sentences).

---

## Feature Overview

✓ Feature 1
✓ Feature 2
✓ Feature 3 (max 6-8 features)

---

## Usage

Quick usage examples (2-3 examples).

---

## Workflow: N Steps

### Step 1 — Action
Description...

### Step 2 — Action
Description...

---

## Key Tables

| Field | Rule | Notes |
|-------|------|-------|
| Item | Rule | Info |

---

## Common Mistakes to Avoid

✗ Mistake 1
✗ Mistake 2

---

## Skill Handoff

```
skill-name
  ├─ Called by: X, Y
  └─ Calls: A, B
```

---

## Summary

All requirements met. Features: 1, 2, 3.
```

### workflow.md Structure

```markdown
# Skill Name — Workflow Diagram

> Last updated: [DATE]

---

## Full Workflow

```
INPUT
  │
  ▼
┌─────────────────┐
│ STEP 1 — Action │
└─────────────────┘
  │
  ▼
[Repeat for each step]
```

---

## Key Decision Points

```
Decision?
├─ Option 1 → Path A
├─ Option 2 → Path B
└─ Option 3 → Path C
```

---

## Summary: Features Met

✅ Feature 1
✅ Feature 2
```

---

## Line Reduction Targets

| Skill | Before | Target | Reduction |
|-------|--------|--------|-----------|
| attribute-researcher | 386 | 270 | 30% |
| repo-clone | 491 | 330 | 33% |
| error-handling | 753 | 480 | 36% |
| project-reviewer | 703 | 440 | 37% |
| evals | 483 | 300 | 38% |
| **TOTAL** | **2,816** | **1,820** | **35%** |

---

## Implementation Order

1. ✅ **mcp-researcher** (DONE: 50% reduction)
2. **attribute-researcher** (workflow.md only)
3. **repo-clone** (high priority: 491 lines)
4. **error-handling** (high priority: 753 lines)
5. **project-reviewer** (medium: 703 lines)
6. **evals** (medium: 483 lines)

---

## Quality Checklist for Each Skill

After optimization, verify:

- [ ] No repeated explanations (one source of truth)
- [ ] Tables for rules/attributes/checks
- [ ] Feature overview present
- [ ] Workflow diagram simplified
- [ ] Common mistakes section present
- [ ] Skill handoff map present
- [ ] 30%+ line reduction achieved
- [ ] All original functionality preserved
- [ ] Numbering is sequential (Step 1 → N, not 0-14)
- [ ] Examples are clear and consistent

---

## Tips for Consolidation

1. **Repetition identification:**
   - Search for repeated phrases ("do not", "always", "check")
   - If explained 3+ times, consolidate to 1 location + references

2. **Table creation:**
   - Lists of items → table rows
   - Multiple similar sections → table columns
   - Rules with exceptions → table with "Notes" column

3. **Step reduction:**
   - If Step N is "prepare for Step N+1" → merge into Step N+1
   - If all steps reference common rule → section reference instead

4. **Diagram simplification:**
   - Remove nested boxes (flatten into sequence)
   - Use decision trees for branching (not multiple flowcharts)
   - Inline notes instead of separate boxes

5. **Workflow merging:**
   - If 10 steps, try to get to 5-8 key phases
   - Each phase has clear input/output
   - Decisions branch but reconverge

---

## Validation

After optimizing all skills:

1. Run `/project-reviewer` for full audit
2. Verify no functionality lost
3. Test each skill works as expected
4. Check all cross-skill handoffs still valid
5. Confirm total reduction ~35-40%

---

## Expected Results

- ✅ All skills optimized (50% reduction for mcp-researcher template)
- ✅ ~35% average reduction across all skills
- ✅ Maximum clarity with tables and hierarchies
- ✅ No repetition
- ✅ Easier to maintain and train on
- ✅ All functionality preserved
