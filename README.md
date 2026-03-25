# Claude Skills - MCP Research Suite

All-in-one MCP server research suite with built-in deployment, error recovery, and security scoring. This consolidated skill collection helps you research, document, and securely deploy Model Context Protocol (MCP) servers with comprehensive attribute analysis and automated error handling.

**Key Features:** Research MCP server attributes • Conditional local setup (clone+install) • 7-phase inline error recovery • 10-attribute security scoring • Skill 2.0 self-learning • 45+ quality checks

---

## Quick Start

### Clone the Repository

```bash
git clone https://github.com/Srinath-Palani/Claude-skills-research.git
cd Claude-skills-research
```

### What's Included

- **4 Consolidated Skills** (optimized from 6 → 26% reduction)
  - `mcp-researcher` (TIER 1) - All-in-one research + setup + error recovery
  - `attribute-researcher` (TIER 2) - Deep attribute research with rule enforcement
  - `project-reviewer` (TIER 3) - Quality audit and compliance checks
  - `evals` (TIER 3) - Test framework and evaluation cases

- **Documentation**
  - `skills/README.md` - Complete navigation guide with use cases
  - `marketplace.json` - Skills metadata and discovery
  - `claude-plugin.json` - Plugin descriptor

---

## Usage

### Research an MCP Server

```bash
mcp-researcher "Research the GitHub MCP server"
mcp-researcher "Clone and setup the Slack MCP locally"
mcp-researcher "Generate security report for Linear MCP"
```

### Review Project Quality

```bash
project-reviewer "Review the project"
project-reviewer "Audit all skills"
```

### Run Tests

```bash
evals "Run test cases"
```

---

## Project Structure

```
Claude-skills-research/
├── skills/                          # Production skills (source of truth)
│   ├── mcp-researcher/              # Primary entry point
│   ├── attribute-researcher/        # Support skill
│   ├── project-reviewer/            # Validation skill
│   ├── evals/                       # Testing framework
│   └── README.md                    # Detailed navigation
├── marketplace.json                 # Skills metadata
├── claude-plugin.json               # Plugin descriptor
├── CLAUDE.md                        # Project documentation
└── README.md                        # This file
```

---

## Key Metrics

| Metric | Value |
|--------|-------|
| **Total Skills** | 4 (consolidated from 6) |
| **Total Lines** | ~2,196 (26% reduction) |
| **Consolidation Savings** | 1,186 lines |
| **Avg Optimization** | 40% per skill |
| **Security Checks** | 45+ quality requirements |

---

## Consolidation Details

- **error-handling (753 lines)** → Merged into mcp-researcher as Phases 1-6 error recovery
- **repo-clone (433 lines)** → Merged into mcp-researcher as Step 3 conditional local setup

---

## Workflow Hierarchy

```
User Input (MCP Server)
    ↓
1️⃣ mcp-researcher (TIER 1 - Primary)
    • Research attributes
    • Conditional local setup
    • Error recovery
    • Generate CSV report
    ↓
2️⃣ attribute-researcher (TIER 2 - Support, called as needed)
    ↓
3️⃣ project-reviewer (TIER 3 - Optional audit)
4️⃣ evals (TIER 3 - Optional tests)
```

---

## Security & Features

✅ **Security Mandate**
- Never asks for credentials in chat
- All credentials handled via filesystem only
- Placeholder syntax for all config examples
- No real secrets stored or transmitted

✅ **Built-in Capabilities**
- 8-step research workflow
- 7-phase inline error recovery (no context-switching)
- 3 deployment options (local/remote/research-only)
- 10-attribute security scoring (0–53 scale)
- Skill 2.0 self-learning via learned-fixes.md

---

## Documentation

- **For Users:** Start with `skills/README.md` for complete navigation and use cases
- **For Discovery:** Check `marketplace.json` and `claude-plugin.json` for metadata
- **For Project Details:** See `CLAUDE.md` for complete project documentation

---

## Upgrade Instructions

If you already have this repository and want the latest:

```bash
git pull origin main
```

**No manual cleanup needed!** Deleted skills are automatically removed, new structure in place.

---

Made with 🤖 Claude | Last Updated: 2026-03-25 | Status: Production Ready
