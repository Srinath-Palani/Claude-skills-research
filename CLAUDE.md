# Claude-skills-research — Skill Configuration

This project contains a suite of specialized skills for researching, deploying, and maintaining MCP (Model Context Protocol) servers. All skills are automatically available when working in this directory.

## Quick Start

Clone this repo, navigate to the directory, and all skills will load automatically:

```bash
git clone <repo-url> Claude-skills-research
cd Claude-skills-research
```

Then invoke any skill with `/skill-name` in Claude Code.

---

## Available Skills

### 1. **mcp-researcher** — Main MCP Server Research & Security Scoring
**Invocation:** `/mcp-researcher`

Research and fully document an MCP server with security scoring. Use this skill whenever you:
- Want to research an MCP server by name, GitHub URL, or repo
- Need a structured profile of an MCP server with all attributes filled in
- Want security scoring and compliance assessment
- Need a report saved to `~/Desktop/MCP_reports/` in both Markdown and CSV formats

**Example usage:**
```
"Research the GitHub MCP server"
"Get a full report on: https://github.com/org/mcp-server"
"What does the Cloudflare MCP server support?"
"Analyze and score this MCP server"
```

**Output:** Structured reports with security confidence score in `~/Desktop/MCP_reports/`

---

### 2. **attribute-researcher** — MCP Server Attribute Documentation
**Invocation:** `/attribute-researcher`

Fill in every structured attribute of an MCP server profile with evidence-backed values. Use this skill when:
- The `mcp-researcher` skill needs to populate attributes for a server
- You need to document specific fields (pricing, authentication, hosting, etc.)
- You want evidence-backed Yes/No answers for every attribute
- You're cataloguing an MCP server with detailed specifications

**Example usage:**
```
"Fill in the attributes for the Snowflake MCP server"
"Populate the MCP profile for this repo: https://github.com/org/repo"
"Catalogue this MCP server"
"Document all the fields for the GitHub MCP server"
```

**Output:** Evidence-backed attribute rows with source quotes or file paths

---

### 3. **repo-clone** — Clone & Install MCP Servers Locally
**Invocation:** `/repo-clone`

Clone a GitHub-hosted MCP server repository, install dependencies, configure environment variables, and establish a working connection. Use this skill when:
- You want to set up an MCP server locally from a GitHub URL
- You need to install dependencies and configure the server
- You want step-by-step guidance with approval before each action
- Default clone location: `/Users/srinathp/MCP_repos/<repo-name>`

**Example usage:**
```
"Set up this MCP server: https://github.com/org/repo"
"Clone and connect this MCP repo"
"Install this MCP server from GitHub locally"
"Clone https://github.com/org/mcp-server to /Users/me/mcp"
```

**Outputs:**
- Cloned repository at specified path
- Installed dependencies
- Ready-to-connect server configuration

---

### 4. **error-handling** — MCP Server Error Diagnosis & Self-Learning
**Invocation:** `/error-handling`

Diagnose and fix MCP server or client errors with step-by-step guidance. Use this skill when:
- An MCP server won't start or connect
- You're getting connection errors (401, 403, timeout, etc.)
- There are dependency or installation problems
- You see any error message or stack trace from an MCP server
- A connection verification fails

**Example usage:**
```
"The MCP server won't start"
"I'm getting a 401 error from my MCP server"
"Connection refused when Claude Code tries to connect"
"ModuleNotFoundError: No module named 'mcp'"
[paste any stack trace or error output]
```

**Features:**
- Classifies errors across 7 categories (dependency, config, auth, transport, protocol, runtime, environment)
- Self-learning: saves new error patterns to `references/learned-fixes.md` for future sessions
- Walks through diagnosis phases with explicit user approval before each fix step
- Never asks for credentials in chat — always directs to file editing

---

### 5. **project-reviewer** — Project & Skill Audit
**Invocation:** `/project-reviewer`

Review the project or individual skills for alignment with project goals, enterprise security requirements, and contribution guidelines. Use this skill when:
- You want a full audit of the project
- You're reviewing a specific skill for compliance
- You've made changes and want to verify they meet requirements
- You're checking if uncommitted changes are ready to commit

**Example usage:**
```
"Review the project"
"Audit all skills for alignment"
"Review the mcp-researcher skill"
"Check if my changes meet requirements"
"Is this ready to commit?"
```

**Outputs:** Structured PASS/FAIL/WARN report with compliance checks across:
- Project goal alignment
- Enterprise security requirements
- Confidentiality rules
- Cross-skill consistency
- Completeness verification
- Skill contribution guidelines

---

## Skill Handoff Map

Skills work together in a workflow:

```
┌──────────────────┐
│  mcp-researcher  │  (main entry point)
└────────┬─────────┘
         │ triggers
         ├─────────────────────────────────────┐
         │                                     │
    ┌────▼─────────────┐            ┌─────────▼────────┐
    │ attribute-        │            │ repo-clone       │
    │ researcher       │            │ (if Local=Yes)   │
    └──────────────────┘            └────────┬─────────┘
         │                                  │
         │ returns attributes               │ installs & extracts
         │                                  │ command/args/env
         └──────────────────┬───────────────┘
                            │
                     ┌──────▼──────────┐
                     │ mcp-researcher  │  (auth & config)
                     └────────┬────────┘
                              │ if error
                              ▼
                     ┌──────────────────┐
                     │ error-handling   │
                     └──────────────────┘
```

---

## Project Structure

```
Claude-skills-research/
├── CLAUDE.md                    ← This file
├── README.md
├── .claude/
│   ├── mcp-researcher/
│   │   ├── skill.md            (main research skill)
│   │   └── workflow.md         (detailed workflow diagram)
│   ├── attribute-researcher/
│   │   ├── skill.md
│   │   └── workflow.md
│   ├── repo-clone/
│   │   ├── skill.md
│   │   └── workflow.md
│   ├── error-handling/
│   │   ├── skill.md
│   │   ├── workflow.md
│   │   └── references/
│   │       ├── error-patterns.md
│   │       └── learned-fixes.md
│   ├── project-reviewer/
│   │   ├── skill.md
│   │   ├── workflow.md
│   │   └── references/
│   │       └── known-gap-patterns.md
│   └── evals/
│       └── README.md           (test cases for skills)
```

---

## Security & Confidentiality

All skills enforce strict security rules:

✓ **Never ask for credentials in chat** — credentials are always entered via file editing
✓ **No hardcoded secrets** — all config examples use `<PLACEHOLDER>` syntax
✓ **Credentials routed to local filesystem only** — never transited through chat
✓ **Security mandates enforced at every step** — no exceptions

---

## Workflow: From Research to Running Locally

### Scenario: You want to research and set up an MCP server

**Step 1:** Use `/mcp-researcher` to research the server
```
"Research the GitHub MCP server"
→ Returns full documentation with security score
```

**Step 2:** If deployment is local, use `/repo-clone` to set it up
```
"Set up https://github.com/modelcontextprotocol/servers/tree/main/src/github"
→ Clones, installs, configures, ready to connect
```

**Step 3:** If there's an error, use `/error-handling`
```
"The server won't connect"
→ Diagnoses root cause, presents fix plan, applies fixes with approval
```

**Step 4:** Use `/project-reviewer` to audit your changes
```
"Is this ready to commit?"
→ Audits for security, completeness, alignment
```

---

## Team Setup

When team members clone this repo:

1. They get all skill files automatically (in `.claude/` directory)
2. CLAUDE.md loads and displays available skills
3. They can invoke any skill with `/skill-name` immediately
4. All skills share the same version — no version conflicts

**No additional setup needed.**

---

## References

- **MCP Spec:** https://spec.modelcontextprotocol.io
- **MCP Servers Directory:** https://github.com/modelcontextprotocol/servers
- **Output location:** `~/Desktop/MCP_reports/` (for research reports)
- **Cloned repos location:** `/Users/srinathp/MCP_repos/<repo-name>` (default)

---

## Contributing / Modifying Skills

If you update a skill's SKILL.md:
- Always update the paired `workflow.md` in the same directory
- Use `/project-reviewer` to verify changes meet requirements
- Follow the "Adding a New Skill" guidelines in `project-reviewer/SKILL.md`

---

## Support & Feedback

For issues with skills:
1. Use `/error-handling` to diagnose
2. Check `error-handling/references/learned-fixes.md` for known solutions
3. Use `/project-reviewer` to audit the skill

For feedback on Claude Code, visit: https://github.com/anthropics/claude-code/issues
