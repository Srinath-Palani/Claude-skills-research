# MCP Researcher Skill References

This skill does not currently have external reference documentation files.

For detailed information, see:
- **workflow.md** — Complete workflow diagram with all steps
- **SKILL.md** — Full skill definition, security mandates, and usage patterns

Output Configuration (Simple & Clean):

**First Time:**
- Open folder picker → User chooses/creates location
- Path used for this session

**Second Time:**
- Show selected path
- Ask: "Make this your default?"
- Yes → Saved to .claude/settings.json
- No  → Open picker to choose different path

**Future Sessions:**
- Show saved default
- Ask: "Use this path?"
- Yes → Use saved default (fast)
- No  → Open picker for one-time different path

**For Both:**
- Reports location (where CSV/Markdown reports saved)
- Clone location (where GitHub repos cloned)

**No Confusion:**
- Clean folder picker UI
- Simple yes/no prompts
- One-click saved defaults
- Can always change by selecting "No"
