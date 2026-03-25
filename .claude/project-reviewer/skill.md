---
name: project-reviewer
user-invocable: true
description: >
  Review the mcp-research project — or any individual skill within it — for alignment with
  project goals, enterprise security requirements, and confidentiality rules. Use this skill
  whenever a user asks to "review the project", "audit the skills", "check alignment",
  "verify security compliance", or "review my changes". Also trigger automatically when
  uncommitted changes are detected and the user says "review my changes", "did I break
  anything?", or "is this ready to commit?". Produces a structured pass/fail review report
  with a gap list and a clear verdict — no file is saved, output is printed to chat only.
---

# MCP-Research Project Reviewer

> Full workflow diagram: see `workflow.md` in this directory.
> Update `workflow.md` whenever this skill is modified.

This skill audits every skill in the mcp-research project against four requirement pillars:
**project goals**, **enterprise security**, **confidentiality**, and **skill contribution
guidelines**. It also checks structural consistency across skills and verifies that
`workflow.md` files are up to date.

The **Skill Contribution Guidelines** pillar enforces the project's published rules for adding
and maintaining skills:
- Every skill must have `user_invocable: true` in its frontmatter
- Description must include explicit trigger phrases ("Use when the user...")
- Skills must include an example usage section
- Names must be action-oriented
- No credentials anywhere in skill files
- Skills must not duplicate each other
- Skills must not be overly specific to one workflow if they could be general

Triggered in three modes:

1. **Full project review** — reads every SKILL.md and workflow.md, applies all checks.
2. **Single skill review** — reviews one named skill; apply only the checks relevant to that skill.
3. **Git diff review** — reads `git diff HEAD` for uncommitted changes, applies targeted checks to
   changed files, and flags any modified SKILL.md whose workflow.md was NOT also updated.

Output is printed to chat only — no file is written.

---

## Usage

Review the project or a specific skill for compliance:

```
"Review the project"
"Audit all skills for alignment"
"Review the mcp-server skill"
"Check if my changes meet requirements"
"Is this ready to commit?"
"Did I break anything?"
```

Output is a structured PASS/FAIL/WARN report printed to chat — no file is written.

---

## Project Inventory

> **Note:** If this skill is adopted for a different project, update the inventory table
> below to list that project's skills. All check group references to specific skill names
> (mcp-server, attribute-researcher, etc.) in Groups 4 and 5 must also be updated.

The mcp-research project contains these skills. Every review starts by confirming this inventory
is complete (no skill has been added or removed without updating this list).

| Skill directory | SKILL.md | workflow.md | Purpose |
|----------------|----------|-------------|---------|
| `/` (root) | `SKILL.md` | `workflow.md` | mcp-server — main research + scorer |
| `attribute-researcher/` | `SKILL.md` | `workflow.md` | Attribute researcher |
| `repo-clone/` | `SKILL.md` | `workflow.md` | Clone + install local servers |
| `error-handling/` | `SKILL.md` | `workflow.md` | Error diagnosis + self-learning |
| `project-reviewer/` | `SKILL.md` | `workflow.md` | This review skill |

Additional reference files:
- `error-handling/references/error-patterns.md`
- `error-handling/references/learned-fixes.md`
- `project-reviewer/references/known-gap-patterns.md`

---

## Step 0 — Pre-load Known Gap Patterns

**Before any other step**, read `references/known-gap-patterns.md`.
If the file exists and contains entries, keep them in mind throughout the review —
a gap that matches a known pattern can be flagged immediately without re-deriving it.
If the file does not exist yet, create it with just the header line and continue.

---

## Step 1 — Determine Review Mode

Read the trigger context and select the mode:

- **User says "review the project" / "audit all skills" / "full review"** → Mode A (Full)
- **User names a specific skill** → Mode B (Single skill)
- **User says "review my changes" / "check uncommitted" / "is this ready?"** → Mode C (Git diff)
- **User just edited a skill** → Mode C (Git diff) — offer Mode A as well

For Mode C, run first:
```bash
git -C <project-root> diff HEAD --name-only
git -C <project-root> diff HEAD
```

If no uncommitted changes exist, report that and offer Mode A.

---

## Step 2 — Read All Target Files

**Mode A:** Read every SKILL.md and workflow.md in the project inventory.
**Mode B:** Read only the named skill's SKILL.md and workflow.md.
**Mode C:** Read only the files returned by `git diff HEAD --name-only`, plus their paired
  SKILL.md/workflow.md if only one of the pair was changed.

Do not fill any check until you have read the relevant file.

---

## Step 3 — Apply All Review Checks

For each skill in scope, apply the checks below. Record each as:
- **PASS** — requirement is clearly met with evidence
- **FAIL** — requirement is missing or violated
- **WARN** — requirement is present but weakly stated or ambiguous

---

### CHECK GROUP 1 — Project Goal Alignment

Apply to every SKILL.md.

| # | Check | Pass condition |
|---|-------|----------------|
| G1 | Frontmatter `name` is present | `name:` field exists in YAML front matter |
| G2 | Frontmatter `description` is present | `description:` field exists |
| G3 | Description accurately reflects skill behaviour | Description matches the actual steps in the skill body |
| G4 | Output format is specified | SKILL.md states what the skill produces (CSV path, report format, etc.) |
| G5 | Source priority order is documented | Skills that fetch external data list their source priority order |
| G6 | Every Yes/No field uses only Yes or No | No "Partial", "Unknown", "N/A" values are allowed in field rules |
| G7 | Evidence backing is required | Rules state that every Yes must be backed by a source quote or file path |
| G8 | Skill handoff map is present | SKILL.md documents which skills it calls and which call it |
| G9 | Common mistakes section is present | A "Common Mistakes to Avoid" or equivalent section exists |
| G10 | Workflow.md exists and is paired | A `workflow.md` exists in the same directory as every SKILL.md |
| G11 | Workflow.md reflects current SKILL.md | Workflow steps match the SKILL.md steps — no orphaned steps in either direction |
| G12 | Frontmatter `user-invocable: true` is present | `user-invocable: true` field exists in YAML front matter — required by skill contribution guidelines |

---

### CHECK GROUP 2 — Enterprise Security Requirements

Apply primarily to `mcp-server/SKILL.md` and any skill that handles credentials or config files.
Apply G2-group checks to all skills that have a credential or auth section.

| # | Check | Pass condition |
|---|-------|----------------|
| S1 | Security Mandate block exists at top level | A named security mandate or security rules section appears before any workflow steps |
| S2 | Rule: never ask for keys in chat | The mandate/rules explicitly state that credentials must never be typed, pasted, or shared in chat |
| S3 | Rule is absolute — no exceptions clause | The no-credentials-in-chat rule uses "absolute", "no exceptions", or equivalent language |
| S4 | Credential alert response is defined | The skill defines the exact text to use when a user accidentally pastes a credential |
| S5 | Placeholder standard is defined | SKILL.md specifies `<PLACEHOLDER>`-style values for all config examples |
| S6 | Auth type → placeholder mapping is present | A table or list maps each auth type (API Token, PAT, OAuth, etc.) to a placeholder label |
| S7 | User is directed to edit file themselves | Steps explicitly say to open the file and replace placeholders — never to paste values in chat |
| S8 | .gitignore reminder for project-level config | The skill reminds users to add `.claude/settings.json` to `.gitignore` when project-level config is chosen |
| S9 | Connection verify step requires real credentials | The JSON-RPC verify step only runs when credentials are confirmed filled (Branch A only, not B2/B3) |
| S10 | Security rules cross-reference the mandate | If a secondary security rules section exists, it defers to the top-level mandate |
| S11 | TLS rules are transport-layer-only | TLS marking rules explicitly say TLS applies to the MCP transport layer, not to upstream API calls |
| S12 | No credentials appear in CSV output columns | The CSV format spec has no column for actual credential values; only attribute classifications |
| S13 | Credential check step asks yes/no, not for the value | Step that checks credentials asks "do you have it?" not "paste it here" |

---

### CHECK GROUP 3 — Confidentiality Requirements

Apply to all skills.

| # | Check | Pass condition |
|---|-------|----------------|
| C1 | No hardcoded credentials in examples | No real API keys, tokens, passwords, or secrets appear anywhere in the SKILL.md |
| C2 | No credentials in workflow.md | workflow.md contains no real credential values |
| C3 | No credentials in reference files | error-patterns.md and learned-fixes.md contain no real credential values |
| C4 | Report output excludes credential columns | CSV output format has no "API Key", "Token value", or similar credential-value columns |
| C5 | All config examples use placeholder syntax | Any `settings.json` / `mcpServers` JSON blocks use `<KEY>` or `<PLACEHOLDER>` syntax |
| C6 | No git-committable credential paths | The skill does not instruct writing credentials to any path that would normally be committed |
| C7 | .env files are not committed | If `.env` creation is mentioned, the skill warns against committing it |

---

### CHECK GROUP 4 — Cross-Skill Consistency

Apply in Mode A (Full project review) only, or in Mode C when handoff-related lines were changed.

| # | Check | Pass condition |
|---|-------|----------------|
| X1 | mcp-server → attribute-researcher handoff | mcp-server SKILL.md Step 5 explicitly triggers attribute-researcher; attribute-researcher SKILL.md confirms it can be called from mcp-server |
| X2 | mcp-server → repo-clone handoff | mcp-server SKILL.md triggers repo-clone when Deployment:Local=Yes; repo-clone SKILL.md confirms trigger from mcp-server |
| X3 | repo-clone → mcp-server auth handoff | repo-clone SKILL.md hands back to mcp-server Auth section after clone+install; mcp-server auth section accepts this trigger |
| X4 | error-handling is referenced from mcp-server | mcp-server SKILL.md hands off to error-handling when connection verify fails |
| X5 | error-handling is referenced from repo-clone | repo-clone SKILL.md hands off to error-handling when server start fails |
| X6 | No orphaned skill references | No SKILL.md references a skill name that does not exist in the project inventory |
| X7 | Skill handoff maps are symmetric | If skill A says it calls skill B, skill B's handoff map acknowledges skill A as a caller |
| X8 | Workflow.md handoff maps match SKILL.md | The handoff map in each workflow.md matches the handoff map in the corresponding SKILL.md |

---

### CHECK GROUP 5 — Completeness per Skill

Apply to each skill individually.

**mcp-server (root SKILL.md):**
- [ ] All 15 verification steps are present
- [ ] CCI score prompt asks before calculating (not auto-calculated)
- [ ] All four CSV categories exist: MCP Info, attributes, CCI score (optional), non-read-only tools
- [ ] Auth setup covers all three trigger paths (after report / from repo-clone / standalone)
- [ ] Branch A and Branch B are both defined
- [ ] All five auth-type credential instruction blocks exist (API Token, Bearer, PAT, OAuth Auth Code, OAuth Client Creds)

**attribute-researcher:**
- [ ] All five steps are present (Fetch Sources, Fill Attributes, Capabilities Tools, Non-Read-Only Tools, Return to mcp-server)
- [ ] Field rules cover all attributes: Distribution Type, Pricing, Hosting Provider, Authentication, TLS, Transport, Tools Operations, Deployment, Capabilities

**repo-clone:**
- [ ] All eight steps are present
- [ ] Connection method decision covers all four types: STDIO, Published Package, Docker, Remote
- [ ] Handoff to mcp-server auth section is explicit (Step 6)

**error-handling:**
- [ ] All seven phases are present
- [ ] All seven error categories are covered in classification table
- [ ] Self-upgrade protocol (Skill 2.0) is defined
- [ ] References to `error-patterns.md` and `learned-fixes.md` exist

**project-reviewer (this skill):**
- [ ] All three review modes are defined (Full, Single, Git diff)
- [ ] All six check groups are present
- [ ] Output format specifies PASS/FAIL/WARN per check
- [ ] Summary verdict format is specified
- [ ] Skill 2.0 self-upgrade protocol is defined
- [ ] `references/known-gap-patterns.md` is read at the start of every review session

---

### CHECK GROUP 6 — Skill Contribution Guidelines

Apply to every SKILL.md. These rules are derived from the project's published "Adding a New
Skill" guidelines and govern how every skill must be authored and maintained.

| # | Check | Pass condition |
|---|-------|----------------|
| P1 | `user-invocable: true` in frontmatter | Same as G12 — confirmed present and set to `true` (not `false`) |
| P2 | Description contains trigger phrases | `description` field includes at least one "Use when the user..." / "Use when the user asks..." / "Use when the user mentions..." phrase |
| P3 | Description lists concrete trigger conditions | Description names specific user intents, not just vague domains (e.g. "asks to research an MCP server" not just "MCP tasks") |
| P4 | Skill name is action-oriented | Name follows verb-noun or action pattern (e.g. `mcp-server`, `attribute-researcher`, `repo-clone`, `error-handling`, `project-reviewer`) — not a noun-only label |
| P5 | Example usage section exists | SKILL.md contains a `## Usage` section (or equivalent) with at least one concrete usage example |
| P6 | Edge cases are addressed | SKILL.md has a "Common Mistakes", "Edge Cases", or equivalent section that covers non-obvious failure paths |
| P7 | Skill does not duplicate another skill | The skill's core function is not already fully covered by an existing skill in the project inventory — check descriptions for overlap |
| P8 | Skill is not overly specific to one workflow | If the skill's logic could be reused across multiple workflows, it is written generically (not hardcoded to a single server, token name, or file path) |
| P9 | No credentials or secrets anywhere in the file | No real API keys, tokens, passwords, or hardcoded secret values appear in the skill — same as C1 but applied as a contribution gate |
| P10 | Instructions written for a knowledgeable engineer | Instructions assume competence — no excessive hand-holding, no redundant explanations of basic tools (git, pip, npm) beyond what is operationally required |

**Scope note:** P7 and P8 are WARN-level by default unless the duplication or hardcoding is
severe enough to block the skill from being reusable. Flag as FAIL only if two skills have
overlapping descriptions and both would trigger on the same user intent.

---

## Step 4 — Check Uncommitted Changes (Mode C only)

For each file in `git diff HEAD --name-only`:

1. **If it is a `SKILL.md`:** check whether the paired `workflow.md` in the same directory also appears in the diff. If not — flag as **WARN: workflow.md not updated**.
2. **If it is a `workflow.md`:** check whether the paired `SKILL.md` also changed. If it did not — WARN: workflow updated without corresponding SKILL.md change.
3. **Re-apply all relevant checks** from Groups 1–5 to the changed content only.
4. **Diff-specific security scan:** scan the raw diff for any line additions (`+` lines) that contain patterns matching credentials:
   - Strings matching: `sk-`, `ghp_`, `glpat-`, `Bearer `, `API_KEY=`, `TOKEN=` followed by a non-placeholder value
   - Any value that looks like a real key (long alphanumeric string after `=` or `:` in a diff line)
   - If found: immediately report as **FAIL: potential credential in diff** and show the line.

---

## Step 5 — Compile and Print the Review Report

Print the full report to chat in this exact format. Do not save it to any file.

```
MCP-Research Project Review
============================
Review mode : [Full / Single: <skill-name> / Git Diff]
Date        : <YYYY-MM-DD>
Files read  : <list of files reviewed>

─────────────────────────────────────────────
CHECK GROUP 1 — Project Goal Alignment
─────────────────────────────────────────────
[Skill name]
  G1  PASS  Frontmatter name present
  G2  PASS  Frontmatter description present
  G3  WARN  Description says "..." but skill body also does "..." — consider updating description
  G4  PASS  Output format specified (CSV to ~/Desktop/MCP_reports/)
  ...

[Next skill name]
  ...

─────────────────────────────────────────────
CHECK GROUP 2 — Enterprise Security
─────────────────────────────────────────────
  S1  PASS  Security Mandate block present at top
  S2  PASS  "Never ask for keys in chat" rule is explicit
  S3  PASS  Rule uses "absolute rule — no exceptions"
  S4  PASS  Credential alert response text defined
  ...

─────────────────────────────────────────────
CHECK GROUP 3 — Confidentiality
─────────────────────────────────────────────
  C1  PASS  No hardcoded credentials in examples
  ...

─────────────────────────────────────────────
CHECK GROUP 4 — Cross-Skill Consistency
─────────────────────────────────────────────
  X1  PASS  mcp-server ↔ attribute-researcher handoff symmetric
  ...

─────────────────────────────────────────────
CHECK GROUP 5 — Completeness
─────────────────────────────────────────────
  mcp-server          : all required sections present
  attribute-researcher: ...
  ...

─────────────────────────────────────────────
CHECK GROUP 6 — Skill Contribution Guidelines
─────────────────────────────────────────────
[Skill name]
  P1  PASS  user_invocable: true in frontmatter
  P2  PASS  Trigger phrases present in description
  P3  WARN  Trigger conditions vague — consider naming specific user intents
  P4  PASS  Name is action-oriented
  P5  FAIL  No Usage / example section found
  P6  PASS  Common Mistakes section present
  P7  PASS  No duplication with existing skills
  P8  PASS  Not hardcoded to a single workflow
  P9  PASS  No credentials in skill file
  P10 PASS  Instructions written for knowledgeable engineer
  ...

─────────────────────────────────────────────
GIT DIFF REVIEW  (Mode C only)
─────────────────────────────────────────────
  Changed files:
    + SKILL.md (root)  → workflow.md also changed: YES / NO
    + ...
  Credential scan: PASS — no credential patterns found in diff
  (or: FAIL — potential credential on line X: show the line)

─────────────────────────────────────────────
SUMMARY
─────────────────────────────────────────────
Total checks : [N]
  PASS       : [N]
  WARN       : [N]  ← review recommended
  FAIL       : [N]  ← must fix before use

VERDICT: ALL REQUIREMENTS MET
  (or: X ISSUE(S) FOUND — see FAIL and WARN items above)

GAP LIST (FAIL items only)
  1. [Skill name] — [Check ID] — [one-line description of what is missing]
  2. ...

RECOMMENDATION LIST (WARN items)
  1. [Skill name] — [Check ID] — [one-line suggestion]
  2. ...
```

---

## Behaviour Rules

- **Never ask the user to paste credentials** — this applies even when reviewing auth sections.
- **Never write to any file** — output is chat-only. If the user wants to save the report, tell them to copy it from chat.
- **Do not fix issues during the review** — report only. If the user wants fixes applied, they should trigger the relevant skill or ask you to edit the skill files separately.
- **For Mode C:** always run the credential scan on the raw diff, even if all structural checks pass.
- **WARN does not block** — a WARN means the requirement is met but could be stronger. A FAIL means a requirement is genuinely absent or violated.
- **Always update workflow.md** after any change to this SKILL.md.

---

## Skill 2.0 — Continuous Self-Upgrade Protocol

**Read `references/known-gap-patterns.md` before starting every review session.** It contains
gap patterns discovered in previous reviews — checking it first lets you flag known issues
instantly instead of re-deriving them from scratch during the review.

After every review session that produces at least one new FAIL or WARN item, evaluate whether
the gap type is genuinely new (not already captured in `references/known-gap-patterns.md`).
If it is, append it. This is what makes the skill smarter over time.

**When to write a new entry:**
- A FAIL or WARN was found that is not covered by any existing check in Groups 1–6
- The gap is structural or recurring (not a one-off mistake in a single file)
- The pattern is specific enough to guide a future check (not just "keep the skill updated")

**How to write the entry** — open `references/known-gap-patterns.md` and append:

```markdown
## [Short gap title — e.g. "workflow.md not updated after Step 11 change"]

**Check group:** [G / S / C / X / P + number, or "New — not yet in a check group"]

**Gap signals:**
- What was missing or violated (one bullet per signal)
- Which skill file and section contained the gap

**Root cause:**
One sentence explaining why this gap tends to occur.

**Recommended check addition:**
Describe the new check rule that would catch this gap in future reviews,
or reference the existing check it should strengthen.

**First seen:** YYYY-MM-DD  |  **Skill:** <skill name>
```

After appending a new gap pattern, consider whether a new check row should be added to one
of the existing check groups (Groups 1–6) in this SKILL.md. If the gap is significant and
recurring, add the check so it is enforced in every future review.

---

## Common Mistakes to Avoid

- Marking a check PASS without reading the file — always read before scoring
- Marking S2 PASS because a security section exists somewhere, even if it is buried after all the workflow steps — the security mandate must appear before the steps to count as PASS
- Skipping the credential scan in Mode C because all structural checks passed
- Reporting WARN as FAIL — WARN items are improvements, not blockers
- Missing the workflow.md freshness check (G11) — the most commonly overlooked check after a SKILL.md update
- Saving the review report to a file — this skill prints to chat only
- Skipping cross-skill consistency checks (Group 4) in Mode C just because only one skill changed — a change to one skill's handoff map may break another's consistency
- Marking G12 / P1 PASS if `user-invocable` is present but set to `false` — it must be `true`
- Marking P2 PASS if the description has trigger examples but not the "Use when the user..." phrase format — the phrase format is required, not just any description of triggers
- Marking P5 PASS because a workflow.md exists — P5 requires a `## Usage` section inside the SKILL.md itself, not in the workflow diagram
- Marking P7 WARN when two skills have identical trigger conditions and would both fire on the same user input — that is a FAIL, not a WARN
- Skipping Group 6 checks when reviewing an existing skill just because it predates the contribution guidelines — the guidelines apply retroactively; flag missing items so they can be remediated
