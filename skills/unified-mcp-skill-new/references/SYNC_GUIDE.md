# Unified MCP Skill — Synchronization Guide (Consolidated)

**Purpose:** Keep SKILL.md and multi-server.md synchronized
**When to use:** Before committing SKILL.md changes
**Last Updated:** 2026-03-26

---

## 🔒 CRITICAL SYNC RULE (LOCKED)

> **Whenever SKILL.md is updated → MUST also update multi-server.md**
>
> Both files are tightly coupled and must stay synchronized.
> This is NOT optional.

---

## ⚡ 30-SECOND CHECK (Before Committing)

Ask yourself these 4 questions:

1. **Did I change workflow steps?**
   - YES → Update Step M1-M3 descriptions in multi-server.md

2. **Did I change attribute fields?**
   - YES → Update Layer 1 JSON schema in multi-server.md

3. **Did I change authentication/framework/protocol rules?**
   - YES → Update multi-server.md agent instructions

4. **Did I change CSV report format?**
   - YES → Update Layer 0 output format in multi-server.md

**If ALL are NO:** You're done. Commit normally.
**If ANY are YES:** Follow the verification checklist below.

---

## 📋 FULL VERIFICATION CHECKLIST (5 Minutes)

### Step 1: Identify What Changed

Check these sections in SKILL.md:
- [ ] Workflow steps (Step 0, 1, 2, 3, etc.)
- [ ] Attribute definitions (Step 5)
- [ ] Report generation rules (Rules 1-5)
- [ ] Authentication detection rules
- [ ] Framework priority mapping
- [ ] Protocol version mappings
- [ ] Error recovery phases
- [ ] CSV formatting rules
- [ ] Security mandate language
- [ ] Tool operation categories

### Step 2: Update multi-server.md

Based on what changed, update:

| Change in SKILL.md | Update in multi-server.md |
|-------------------|---------------------------|
| Workflow steps | Step M1-M3 descriptions |
| Attributes (Step 5) | Layer 1 JSON output schema |
| Report rules | Layer 0 comparison output format |
| Auth detection rules | Agent rules section |
| Framework priority | Agent implementation logic |
| Protocol versions | Layer 1 agent documentation |
| Error recovery | Phase references (1-7) |
| CSV format | Layer 0 CSV formatting rules |

### Step 3: Verify Cross-References

- [ ] SKILL.md mentions multi-server.md correctly
- [ ] multi-server.md references SKILL.md Steps correctly
- [ ] Both files reference same attribute names
- [ ] Protocol version mappings match
- [ ] Framework priority matches between files

### Step 4: Update Other Files (If Major Changes)

- [ ] CLAUDE.md reflects any new features
- [ ] README.md reflects any version changes
- [ ] MEMORY.md reflects any new learnings
- [ ] learned-fixes.md contains relevant error patterns

### Step 5: Check for Red Flags

Stop if you notice ANY of these:

- ❌ Different attribute names (e.g., "Transport" vs "TransportMethod")
- ❌ Protocol versions don't match between files
- ❌ Step numbers out of sync (Step 0 vs Step M1)
- ❌ Authentication rules contradict
- ❌ Framework priority different
- ❌ CSV format rules differ
- ❌ Error recovery phases count mismatch (should be 7)
- ❌ Report generation rules missing
- ❌ Layer 1 JSON schema incomplete

**DO NOT COMMIT if red flags found. Fix them first.**

---

## 🎯 REAL-WORLD EXAMPLES

### Example 1: Adding New Attribute to Step 5

**Your SKILL.md Change:**
```markdown
## Step 5 — Attribute Filling

**NEW:** Add "Compliance Score" field (0-100 scale)
```

**What to update in multi-server.md:**
1. Find "Layer 1 — Research Agent" section
2. Update JSON schema:
   ```json
   "attributes": {
     // ... existing fields ...
     "compliance_score": "0-100",  // NEW
   }
   ```
3. Update Layer 0 comparison table if "Compliance Score" is a new column

### Example 2: Updating Authentication Detection Rules

**Your SKILL.md Change:**
```markdown
Bearer Token + OAuth = both Yes ALWAYS
```

**What to update in multi-server.md:**
1. Find "Agent rules section"
2. Add: "Detect Bearer Token + OAuth co-occurrence"
3. Add: "Mark both as Yes per SKILL.md auth rules"

### Example 3: Adding New Protocol Version

**Your SKILL.md Change:**
```markdown
Protocol `2026-02-10` added to SDK Mapping
```

**What to update in multi-server.md:**
1. Layer 1 JSON schema: Add new protocol option
2. Layer 0 comparison table: Include new protocol as column option

---

## 📍 KEY SYNC LOCATIONS

**In SKILL.md to check:**
- Line 0-8: Steps 0-8 (workflow)
- Step 5: Attribute definitions
- Report Generation Rules section
- Authentication Detection Rules section
- Framework Priority section
- Protocol Mapping section

**In multi-server.md to check:**
- Step M1-M3: Multi-server workflow (mirrors SKILL.md Steps 0-8)
- Layer 1: JSON output schema (mirrors Step 5 attributes)
- Layer 0: Comparison table (11 columns)
- Agent rules section (uses SKILL.md learnings)

---

## ✅ PRE-COMMIT WORKFLOW

```bash
# 1. Make your SKILL.md changes
vi SKILL.md

# 2. Run 30-second check (4 questions above)
# If all NO → proceed to step 6
# If any YES → proceed to step 3

# 3. Open multi-server.md
vi references/multi-server.md
# (Update based on your changes)

# 4. Verify no red flags
grep "protocol_version\|Bearer Token\|Step M" references/multi-server.md
grep "protocol_version\|Bearer Token\|Step " SKILL.md

# 5. Stage both files
git add SKILL.md references/multi-server.md

# 6. Commit together
git commit -m "Update skill: [description]"
```

---

## 🚨 QUICK RED FLAG CHECK

Before committing, run:

```bash
# Check protocol versions align
echo "=== SKILL.md ===" && grep -n "protocol_version\|2025-\|2026-" SKILL.md
echo "=== multi-server.md ===" && grep -n "protocol_version\|2025-\|2026-" references/multi-server.md

# Check attribute names match
echo "=== SKILL.md attributes ===" && grep -n "Bearer Token\|API Token\|OAuth" SKILL.md
echo "=== multi-server.md attributes ===" && grep -n "Bearer Token\|API Token\|OAuth" references/multi-server.md
```

If output differs → RED FLAG. Fix synchronization.

---

## 📝 MAINTENANCE LOG

Track sync updates here:

| Date | SKILL.md Change | multi-server.md Update | Status |
|------|-----------------|------------------------|--------|
| 2026-03-26 | Multi-server workflow finalized | Created & verified | ✅ |
| | | | |

---

## 🔗 FILES INVOLVED

**Must Stay Synchronized:**
- `/skills/unified-mcp-skill-new/SKILL.md` — Main skill (source of truth)
- `/skills/unified-mcp-skill-new/references/multi-server.md` — Multi-server workflow (dependent)

**Update if major changes:**
- `/CLAUDE.md` — Project documentation
- `/README.md` — Release notes
- `/.claude/projects/memory/MEMORY.md` — Persistent learnings
- `/skills/unified-mcp-skill-new/references/learned-fixes.md` — Error patterns

---

## 💡 REMEMBER

**The Golden Rule:**
```
UPDATE SKILL.md → CHECK multi-server.md → UPDATE if needed → COMMIT BOTH
```

**TL;DR:**
1. Change SKILL.md
2. Ask: "Does this affect multi-server.md?"
3. If YES → update multi-server.md
4. Commit both files together

---

**Status:** ✅ Active
**Enforcement:** 🔒 LOCKED (requires explicit approval to disable)
**Complexity:** Minimal (this one file)
**Purpose:** Prevent desynchronization between tightly coupled files
