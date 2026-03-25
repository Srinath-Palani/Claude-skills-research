---
name: project-reviewer
user-invocable: true
description: >
  Review the mcp-research project — or any individual skill within it — for alignment with
  project goals, enterprise security, and confidentiality. Use this skill when the user asks
  to "review the project", "audit all skills", "review <skill-name>", "review my changes",
  or "is this ready to commit?". Produces a structured pass/fail report with gap list and
  verdict — output to chat only, no file saved.
---

# MCP-Research Project Reviewer

> Full workflow diagram: see `workflow.md` in this directory.

Audits every skill in mcp-research against **project goals, enterprise security, confidentiality,
and skill contribution guidelines**. Checks structural consistency and workflow.md freshness.

## Feature Overview

✓ Three review modes: Full project / Single skill / Git diff
✓ Six check groups: 45+ auditable requirements
✓ Pass/Fail/Warn scoring with gap list
✓ Credential pattern detection in diffs
✓ Skill 2.0 self-learning protocol
✓ Cross-skill consistency validation

---

## Usage

```
"Review the project"              → Full audit of all skills
"Audit all skills for alignment"  → Same as above
"Review the repo-clone skill"     → Single skill audit
"Review my changes"               → Git diff + security scan
"Is this ready to commit?"        → Git diff + gap check
```

---

## Project Inventory

mcp-research contains: mcp-server (root), attribute-researcher/, repo-clone/, error-handling/, project-reviewer/ (this skill).

Reference files: error-handling/references/{error-patterns.md, learned-fixes.md}, project-reviewer/references/known-gap-patterns.md.

---

## Review Checks: 6 Groups × 45+ Checks

### GROUP 1 — Project Goal Alignment (12 checks)

| Check | Pass Condition |
|-------|--------|
| G1 | Frontmatter `name:` field exists |
| G2 | Frontmatter `description:` field exists |
| G3 | Description matches actual skill body |
| G4 | Output format specified (CSV path, report format, etc.) |
| G5 | Source priority order documented (if external data fetched) |
| G6 | Yes/No fields only (no "Partial", "Unknown", "N/A") |
| G7 | Evidence backing required (every Yes backed by quote or path) |
| G8 | Skill handoff map present (calls + called-by) |
| G9 | Common mistakes section present |
| G10 | workflow.md exists and is paired to SKILL.md |
| G11 | workflow.md reflects current SKILL.md steps |
| G12 | Frontmatter `user-invocable: true` is present |

### GROUP 2 — Enterprise Security (13 checks)

Apply to mcp-server and skills with credentials/config sections.

| Check | Pass Condition |
|-------|--------|
| S1 | Security Mandate at top level (before workflow steps) |
| S2 | Never-ask-for-keys rule explicit |
| S3 | Rule marked absolute / no exceptions |
| S4 | Credential alert response text defined |
| S5 | Placeholder standard defined (`<PLACEHOLDER>` style) |
| S6 | Auth type → placeholder mapping (API Token, PAT, OAuth, etc.) |
| S7 | User directed to edit file only (never paste in chat) |
| S8 | .gitignore reminder for `.claude/settings.json` |
| S9 | Verify only runs with real credentials filled |
| S10 | Security rules defer to top-level mandate |
| S11 | TLS = MCP transport layer only (not upstream API) |
| S12 | CSV output excludes credential value columns |
| S13 | Credential check asks yes/no, not for value |

### GROUP 3 — Confidentiality (7 checks)

| Check | Pass Condition |
|-------|--------|
| C1 | No hardcoded credentials in SKILL.md |
| C2 | No credentials in workflow.md |
| C3 | No credentials in reference files |
| C4 | CSV output excludes credential columns |
| C5 | Config examples use placeholder syntax |
| C6 | No git-committable credential paths |
| C7 | .env files not committed (if mentioned) |

### GROUP 4 — Cross-Skill Consistency (8 checks)

Apply in Full review or when handoff lines changed.

| Check | Pass Condition |
|-------|--------|
| X1 | mcp-server ↔ attribute-researcher handoff symmetric |
| X2 | mcp-server ↔ repo-clone handoff when Deployment:Local=Yes |
| X3 | repo-clone → mcp-server auth handoff explicit |
| X4 | error-handling referenced from mcp-server (verify fail) |
| X5 | error-handling referenced from repo-clone (start fail) |
| X6 | No orphaned skill references |
| X7 | Handoff maps symmetric (A calls B → B called-by A) |
| X8 | workflow.md handoff maps match SKILL.md |

### GROUP 5 — Completeness per Skill

| Skill | Required Sections |
|-------|--------|
| mcp-server | 15 verification steps, CCI prompt, 4 CSV categories, 3 auth triggers, 2 branches, 5 auth-type blocks |
| attribute-researcher | 5 steps, 9 attribute field rules |
| repo-clone | 8 steps, 4 connection types, Step 6 handoff to auth |
| error-handling | 7 phases, 7 error categories, Skill 2.0 protocol, 2 reference files |
| project-reviewer | 3 modes, 6 groups, PASS/FAIL/WARN format, Skill 2.0, known-gap-patterns.md |

### GROUP 6 — Skill Contribution Guidelines (10 checks)

| Check | Pass Condition | Scope |
|-------|--------|--------|
| P1 | `user-invocable: true` (not false) | All skills |
| P2 | Trigger phrases: "Use when user..." in description | All skills |
| P3 | Concrete trigger conditions (specific intents, not vague) | All skills |
| P4 | Action-oriented name (verb-noun, not noun-only) | All skills |
| P5 | Usage section with example in SKILL.md | All skills |
| P6 | Edge cases / Common Mistakes section | All skills |
| P7 | No duplication of existing skill (FAIL if same intent; WARN if overlap) | All skills |
| P8 | Not overly specific (generic if reusable across workflows) | All skills |
| P9 | No credentials/secrets in skill file | All skills |
| P10 | Instructions for knowledgeable engineer (no excessive hand-holding) | All skills |

---

## Workflow: 4 Steps

### Step 1 — Pre-load Known Gap Patterns

Read `references/known-gap-patterns.md` (create empty if missing). Keep in mind throughout review.

### Step 2 — Determine Mode and Read Files

- **Mode A (Full):** All SKILL.md + workflow.md
- **Mode B (Single):** Named skill's files only
- **Mode C (Git diff):** `git diff HEAD --name-only` + raw diff. Read changed files + pairs.

Do not apply checks until all files read.

### Step 3 — Apply Checks

For each file, apply Groups 1–6. Score: PASS / FAIL / WARN.

**Mode C only:**
- Scan diff for credential patterns (sk-, ghp_, glpat-, Bearer , API_KEY=, TOKEN=, long alphanumeric)
- If SKILL.md changed but workflow.md did not → WARN: workflow not updated

### Step 4 — Print Report + Self-Upgrade

Print to chat (never write file):

```
Review mode: [Full / Single: <name> / Git Diff]
Date: YYYY-MM-DD
Files: [list]

GROUP 1 — Project Goal Alignment
[Skill name]
  G1 PASS …
…
GROUP 2 — Enterprise Security
  S1 PASS …
[Groups 3–6 similarly]

SUMMARY
Total: [N] | PASS: [N] | WARN: [N] | FAIL: [N]
VERDICT: [ALL MET / X ISSUE(S)]

GAP LIST (FAIL only)
RECOMMENDATION LIST (WARN only)
```

After review: If new FAIL/WARN type found → append to `references/known-gap-patterns.md` + consider adding check.

---

## Behaviour Rules

- Never ask user to paste credentials (even when reviewing auth sections)
- Never write to any file — output is chat-only
- Do not fix issues during review — report only
- Credential scan runs in every Mode C, even if all structural checks pass
- WARN ≠ FAIL — warn items are improvements, not blockers
- Always update workflow.md after any SKILL.md change

---

## Skill 2.0 — Continuous Self-Upgrade

After every review session with new FAIL/WARN items:
1. Read `references/known-gap-patterns.md` to check if gap already captured
2. If gap is new, structural, and recurring → append with context:
   ```
   ## [Gap title]
   **Check group:** [G/S/C/X/P + number]
   **Gap signals:** [bullets]
   **Root cause:** [one sentence]
   **Recommended check:** [new check or existing one to strengthen]
   **First seen:** YYYY-MM-DD | **Skill:** [name]
   ```
3. Consider adding new check row to Groups 1–6 if gap is significant

---

## Common Mistakes to Avoid

✗ Mark check PASS without reading file — always read first
✗ Mark S2 PASS because security section exists somewhere — must appear before workflow steps
✗ Skip credential scan in Mode C if structural checks pass — credential scan always runs
✗ Report WARN as FAIL — WARN items are improvements only
✗ Miss workflow.md freshness check (G11) — most commonly overlooked after SKILL.md update
✗ Save review report to file — chat output only
✗ Mark G12/P1 PASS if `user-invocable` present but set to `false` — must be `true`
✗ Mark P2 PASS without "Use when the user..." phrase format — phrase format required
✗ Mark P5 PASS because workflow.md exists — P5 requires `## Usage` in SKILL.md itself
✗ Mark P7 WARN when two skills have identical trigger conditions — that is FAIL, not WARN

---

## Skill Handoff Map

```
project-reviewer
  ├─ triggered by: user directly OR after editing any skill
  ├─ reads: known-gap-patterns.md (Step 0)
  │         all SKILL.md + workflow.md in project
  │         error-handling/references/*.md
  ├─ writes: references/known-gap-patterns.md (Skill 2.0 only)
  └─ does NOT call: any other skill (review only)
```

---

## Summary

All 45+ checks consolidated into 6 groups with dynamic passing conditions. Three review modes (Full/Single/Diff) supported. Self-learning via known-gap-patterns.md. No file writes.
