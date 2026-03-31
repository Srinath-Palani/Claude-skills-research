# mcp-attribute-validator

## Purpose

Validates that all required MCP server attributes are present, correctly formatted, and evidence-backed in a research report or attribute document.

## Trigger

Use this skill when the user asks to:
- Validate MCP attributes
- Check if an MCP report is complete
- Verify attribute coverage for a documented MCP server
- Audit attribute quality before saving/publishing

## Workflow

### Step 1 — Load Attribute Schema

Load the required attribute list from the unified skill's attribute schema (defined in `skills/unified-mcp-skill-new/SKILL.md`).

### Step 2 — Check Each Attribute

For each required attribute, verify:
- [ ] Value is present (not empty, not `<PLACEHOLDER>`, not `Unknown`)
- [ ] Evidence source is cited (URL, doc page, or command output)
- [ ] Format matches the expected type (string, boolean, enum, etc.)

### Step 3 — Report Results

Output a validation summary:

```
ATTRIBUTE VALIDATION REPORT
============================
Server: <name>
Date: <date>

PASSED: <n> attributes
FAILED: <n> attributes
WARNINGS: <n> attributes

FAILURES:
- <attribute>: <reason>

WARNINGS:
- <attribute>: <reason>

VERDICT: PASS / FAIL
```

### Step 4 — Fix or Flag

- If `FAIL`: list the missing/invalid attributes and prompt the user to fill them in
- If `PASS`: confirm the report is ready to save

## Rules

- Never mark an attribute as valid without evidence
- Treat `Unknown` as a failure unless explicitly noted as intentionally unknown
- Treat `<PLACEHOLDER>` values as failures
- Security-sensitive attributes (auth, token, credentials) must reference a file path, not a literal value
