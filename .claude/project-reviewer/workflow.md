# project-reviewer Skill — Workflow Diagram

> Last updated: 2026-03-24 (added Skill 2.0 self-upgrade protocol + Step 0 pre-load)
> Update this file every time SKILL.md is modified.

---

## Full Workflow

```
USER TRIGGERS REVIEW
  │
  ├── "review the project" / "audit all skills"
  ├── "review <skill-name>"
  └── "review my changes" / "check uncommitted"
              │
              ▼
┌─────────────────────────────────────────────┐
│  STEP 0 — Pre-load Known Gap Patterns       │
│                                             │
│  Read references/known-gap-patterns.md      │
│  • If file exists: load known patterns      │
│  • If file missing: create empty file       │
│  Keep patterns in mind throughout review    │
└──────────────────┬──────────────────────────┘
                   │
                   ▼
┌─────────────────────────────────────────────┐
│  STEP 1 — Determine Review Mode             │
│                                             │
│  A: Full project review                     │
│     → read ALL SKILL.md + workflow.md       │
│                                             │
│  B: Single skill review                     │
│     → read named skill only                 │
│                                             │
│  C: Git diff review                         │
│     → git diff HEAD --name-only             │
│     → git diff HEAD (raw diff)              │
│     → read changed files + their pairs      │
└──────────────────┬──────────────────────────┘
                   │
                   ▼
┌─────────────────────────────────────────────┐
│  STEP 2 — Read All Target Files             │
│                                             │
│  Mode A: all SKILL.md + workflow.md         │
│  Mode B: named skill's files only           │
│  Mode C: git diff changed files             │
│          + paired SKILL.md/workflow.md      │
│                                             │
│  Do NOT apply any checks until              │
│  all target files are read.                 │
└──────────────────┬──────────────────────────┘
                   │
                   ▼
┌─────────────────────────────────────────────┐
│  STEP 3 — Apply Review Checks               │
│                                             │
│  GROUP 1 — Project Goal Alignment           │
│    G1  Frontmatter name present             │
│    G2  Frontmatter description present      │
│    G3  Description matches skill body       │
│    G4  Output format specified              │
│    G5  Source priority order documented     │
│    G6  Only Yes/No allowed in field rules   │
│    G7  Evidence backing required            │
│    G8  Skill handoff map present            │
│    G9  Common mistakes section present      │
│   G10  workflow.md paired to SKILL.md       │
│   G11  workflow.md matches SKILL.md steps   │
│                                             │
│  GROUP 2 — Enterprise Security              │
│    S1  Security Mandate at top level        │
│    S2  Never-ask-for-keys rule explicit     │
│    S3  Rule marked absolute / no exceptions │
│    S4  Credential alert response defined    │
│    S5  Placeholder standard defined         │
│    S6  Auth type → placeholder map present  │
│    S7  User directed to edit file only      │
│    S8  .gitignore reminder for project cfg  │
│    S9  Verify only runs with real creds     │
│   S10  Security rules defer to mandate      │
│   S11  TLS = transport layer only           │
│   S12  No credential columns in CSV output  │
│   S13  Credential check asks yes/no only    │
│                                             │
│  GROUP 3 — Confidentiality                  │
│    C1  No hardcoded creds in SKILL.md       │
│    C2  No creds in workflow.md              │
│    C3  No creds in reference files          │
│    C4  CSV output excludes cred columns     │
│    C5  Config examples use placeholders     │
│    C6  No git-committable cred paths        │
│    C7  .env files not committed             │
│                                             │
│  GROUP 4 — Cross-Skill Consistency          │
│    X1  mcp-server ↔ attribute-researcher          │
│    X2  mcp-server ↔ repo-clone              │
│    X3  repo-clone → mcp-server auth         │
│    X4  error-handling in mcp-server         │
│    X5  error-handling in repo-clone         │
│    X6  No orphaned skill references         │
│    X7  Handoff maps symmetric               │
│    X8  workflow.md handoff maps match       │
│                                             │
│  GROUP 5 — Completeness per Skill           │
│    Each skill's required sections present   │
│                                             │
│  GROUP 6 — Skill Contribution Guidelines    │
│    P1  user_invocable: true in frontmatter  │
│    P2  Trigger phrases in description       │
│    P3  Concrete trigger conditions          │
│    P4  Action-oriented name                 │
│    P5  Usage / example section in SKILL.md  │
│    P6  Edge cases addressed                 │
│    P7  No duplication of existing skill     │
│    P8  Not overly specific / hardcoded      │
│    P9  No credentials in skill file         │
│   P10  Instructions for knowledgeable eng.  │
└──────────────────┬──────────────────────────┘
                   │
                   ▼
┌─────────────────────────────────────────────┐
│  STEP 4 — Git Diff Checks (Mode C only)     │
│                                             │
│  For each changed file:                     │
│  • SKILL.md changed → workflow.md changed?  │
│    NO → WARN: workflow not updated          │
│  • workflow.md changed → SKILL.md changed?  │
│    NO → WARN: workflow updated w/o skill    │
│                                             │
│  Credential scan on raw diff:               │
│  • Scan + lines for: sk-, ghp_, glpat-,     │
│    Bearer , API_KEY=, TOKEN=                │
│  • Any long alphanumeric after = or :       │
│                                             │
│  FOUND → FAIL: potential credential in diff │
│  NOT FOUND → PASS: credential scan clean   │
└──────────────────┬──────────────────────────┘
                   │
                   ▼
┌─────────────────────────────────────────────┐
│  STEP 5 — Compile and Print Report          │
│                                             │
│  Print to chat (no file written):           │
│                                             │
│  MCP-Research Project Review                │
│  ============================               │
│  Review mode / Date / Files read            │
│                                             │
│  GROUP 1 results (per skill)                │
│    G1  PASS/FAIL/WARN + note                │
│    ...                                      │
│                                             │
│  GROUP 2 results                            │
│  GROUP 3 results                            │
│  GROUP 4 results                            │
│  GROUP 5 results                            │
│  GROUP 6 results                            │
│  GIT DIFF results (Mode C)                  │
│                                             │
│  SUMMARY                                    │
│  ─────────────────────────────              │
│  Total / PASS / WARN / FAIL count           │
│                                             │
│  VERDICT: ALL REQUIREMENTS MET              │
│       or: X ISSUE(S) FOUND                 │
│                                             │
│  GAP LIST (FAIL items only)                 │
│  RECOMMENDATION LIST (WARN items)           │
└──────────────────┬──────────────────────────┘
                   │
                   ▼
┌─────────────────────────────────────────────┐
│  SKILL 2.0 — Self-Upgrade (after session)   │
│                                             │
│  Did session surface new FAIL/WARN types?   │
│  YES → Is gap type new (not in ref file)?   │
│         YES → Append to                     │
│               references/known-gap-         │
│               patterns.md                  │
│               Consider adding new check     │
│               row to Groups 1–6            │
│         NO  → Skip (already captured)      │
│  NO  → Skip (no new patterns found)        │
└─────────────────────────────────────────────┘
```

---

## Review Mode Decision

```
Trigger text
      │
      ├── "review the project" / "audit all" ──────────► Mode A (Full)
      │
      ├── "review mcp-server" / "review repo-clone" ───► Mode B (Single)
      │
      ├── "review my changes" / "check uncommitted" ───► Mode C (Git Diff)
      │     → also run: git diff HEAD --name-only
      │                 git diff HEAD
      │
      └── just edited a skill ──────────────────────────► Mode C + offer A
```

---

## Check Result Legend

```
PASS  — Requirement clearly met; evidence found in the file
WARN  — Requirement present but weakly stated; improvement recommended
FAIL  — Requirement absent or violated; must fix before skill is production-ready
```

---

## Output Rules

```
• Print to chat ONLY — never write a file
• Do not fix issues — report only
• Credential scan runs in EVERY Mode C review, even if all structural checks pass
• WARN ≠ FAIL — warn items are improvements, not blockers
```

---

## Skill Handoff Map

```
project-reviewer
  │
  ├─── triggered by ──────────────────── user directly
  │                                  OR  after editing any skill in the project
  │
  ├─── reads ─────────────────────────── references/known-gap-patterns.md (Step 0)
  │                                      mcp-server SKILL.md + workflow.md
  │                                      attribute-researcher SKILL.md + workflow.md
  │                                      repo-clone SKILL.md + workflow.md
  │                                      error-handling SKILL.md + workflow.md
  │                                      error-handling/references/*.md
  │                                      project-reviewer SKILL.md + workflow.md
  │
  ├─── writes ────────────────────────── references/known-gap-patterns.md
  │                                      (Skill 2.0: new gap patterns only)
  │
  └─── does NOT call ─────────────────── any other skill
                                         (review only — no modifications)
```
