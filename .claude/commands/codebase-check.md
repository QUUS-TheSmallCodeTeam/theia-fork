---
description: Check codebase file lengths, update structure docs, and suggest refactoring if needed (PM-driven)
allowed-tools: Read, Glob, Grep, Write, Edit, Task, Bash
model: inherit
---

# Codebase Health Check & Structure Update (PM-Driven)

You are running the **Codebase Health Check** command. This command performs post-implementation validation and maintenance tasks.

---

## CODEBASE PRINCIPLES (Reference)

### File Length Management

**Golden Rule: 750 Lines Maximum** (with exceptions)

**Why 750 Lines?**
- Human readability limit
- Faster navigation and search
- Easier code review
- Reduces merge conflicts
- Forces modular design

**Exception Categories:**

1. **Data Files** (No logic, pure data)
   - ✅ Variable lists, constant definitions, configuration objects
   - ✅ Example: `keyboard-layouts.ts` (1200 lines of key mappings)
   - ⚠️ Limit: 2000 lines (split if larger)

2. **Generated Code**
   - ✅ Auto-generated files (e.g., protocol definitions, API bindings)
   - ✅ Must have `// AUTO-GENERATED - DO NOT EDIT` header
   - ⚠️ No line limit (but should be readable)

3. **Comprehensive Test Suites**
   - ✅ Test files covering complex feature end-to-end
   - ✅ Example: `spatial-canvas.spec.ts` (1500 lines testing full lifecycle)
   - ⚠️ Limit: 1500 lines (split into multiple test files if larger)

### Code Structure Principles

**Feature-Based Organization** (not Type-Based):
```
✅ GOOD: eg-desk_taehwa/theme/theme-service.ts
❌ BAD:  eg-desk_taehwa/services/theme-service.ts
```

**Separation of Concerns**:
- Service Layer: `*-service.ts` (business logic)
- UI Layer: `*-widget.tsx` (presentation)
- Contribution Layer: `*-contribution.ts` (Theia integration)
- Utils Layer: `*-utils.ts` (helpers, optional)

**Complexity Metrics**:
- Cyclomatic Complexity: Max 10 per function
- Function Length: Max 50 lines
- Class Member Count: Max 15 methods
- Import Count: Max 20 imports per file

---

## Your Mission

After coding-agent completes implementation, perform these tasks in order:

### Phase 1: Gather Context

**Step 1: Find modified files**
Ask user which files were modified, OR use:
```bash
Bash: git diff --name-only HEAD
```

**Step 2: Check each modified file's line count**
For each modified file:
```bash
Read: [file-path]
# Count lines
```

### Phase 2: File Length Analysis

For each file:

**If < 700 lines:**
✅ OK - No action needed

**If 700-750 lines:**
⚠️ WARNING - Approaching limit
- Note it for monitoring
- Continue to Phase 3

**If > 750 lines:**
❌ EXCEEDS LIMIT
- Check exception categories:
  - Data file? (constants, configs)
  - Generated code? (AUTO-GENERATED header)
  - Test suite? (*.spec.ts)
- If NOT exception:
  - **STOP** - Refactoring required
  - Proceed to Phase 2.5 (Consult PM)

### Phase 2.5: Consult PM for Refactoring (If file > 750 lines)

**Invoke PM agent:**
```
Task(agent: "egdesk-pm-agent",
     prompt: "File [path] exceeds 750 lines ([N] lines total).

             Current structure (summary):
             - [Class/Module 1]: Lines [X-Y] ([Z] lines)
             - [Class/Module 2]: Lines [A-B] ([C] lines)

             Exception check:
             - Data file: [Yes/No]
             - Generated: [Yes/No]
             - Test suite: [Yes/No]

             Suggest refactoring strategy following principles:
             - Feature-based organization (not type-based)
             - Separation of Concerns (service/UI/contribution layers)
             - Single responsibility per module")
```

**PM will return:**
- Refactoring plan (split into N files)
- Module boundaries
- Rationale

**Present to user:**
```markdown
⚠️ FILE LENGTH VIOLATION

**File**: `[path]`
**Lines**: [N] (exceeds 750 limit)
**Exception**: No

**PM Refactoring Plan**:
[Paste PM's plan]

**User Decision Required**:
A) Approve refactoring (I'll coordinate with coding-agent)
B) Override limit (provide rationale - will document)
C) Cancel / Review later
```

**If user approves refactoring:**
- Invoke coding-agent with PM's plan
- coding-agent performs refactoring
- Re-run this command after refactoring

**If user overrides:**
- Document in CODEBASE_STRUCTURE.md under "Exceptions"

### Phase 3: Update CODEBASE_STRUCTURE.md

**Step 1: Find structure document**
```bash
Glob: eg-desk*/CODEBASE_STRUCTURE.md
# OR
Glob: CODEBASE_STRUCTURE.md
```

**Step 2: Read current structure**
```bash
Read: [discovered-path]/CODEBASE_STRUCTURE.md
```

**Step 3: Determine what to update**

Based on modified files, update these sections:

**If new services/classes created:**
- Update "Services Registry" section
- Format: `- ServiceName (path/to/file.ts:line) - Purpose`

**If new keybindings added:**
- Update "Keybindings Registry"
- Format: `- Ctrl+K: Feature (path/to/contribution.ts:line)`

**If new commands added:**
- Update "Command Registry"
- Format: `- custom.commandId: Description (path/to/file.ts:line)`

**If new feature implemented:**
- Update "Custom Features Timeline"
- Format:
  ```markdown
  ### YYYY-MM-DD HH:MM: Feature Name
  - Files: [list]
  - Dependencies: [list]
  - Notes: [any important info]
  ```

**Step 4: Write updates**
```bash
Edit: [structure-doc-path]
# Add entries to appropriate sections
```

### Phase 4: Check Stale Reports

**Check for old reports in subagent_reports/:**

```bash
Bash: find subagent_reports -name "*.md" -type f 2>/dev/null | head -20
```

**For each report found:**
1. Extract date from filename: `YYYYMMDD_HHMM_[topic]-[agent].md`
2. Compare to today's date
3. If >7 days old:
   ```markdown
   ⚠️ STALE REPORT FOUND

   **File**: `subagent_reports/[filename]`
   **Age**: [N] days old
   **Topic**: [Extract from filename]

   **Action Required**:
   A) Delete (task resolved/abandoned)
   B) Archive to ideas&external_references/ (valuable insights)
   C) Keep (still actively working on it)
   ```

### Phase 5: Report to User

**Output format:**
```markdown
## Codebase Health Check Complete

### File Length Analysis
✅ **Passed**: [N] files under 750 lines
⚠️ **Warning**: [M] files approaching limit (700-750 lines):
  - [file-1]: [X] lines
❌ **Failed**: [K] files exceed limit:
  - [file-2]: [Y] lines (refactoring needed)

### Structure Document Updates
✅ **Updated**: CODEBASE_STRUCTURE.md
  - Added [N] new services to registry
  - Added [M] new keybindings
  - Added feature to timeline (YYYY-MM-DD HH:MM)

### Stale Reports Check
✅ **Clean**: No stale reports (or all resolved)
⚠️ **Found**: [N] reports >7 days old:
  - [report-1]: [age] days (action required)
  - [report-2]: [age] days (action required)

### Actions Taken
- [List of updates made]

### Actions Required (if any)
- [Refactoring needed for file X]
- [User decision needed for override]
- [Stale report cleanup needed]

### Next Steps
[What user or Main Thread should do next]
```

## Special Cases

### If CODEBASE_STRUCTURE.md doesn't exist:

Ask user:
```
CODEBASE_STRUCTURE.md not found.

Options:
A) Create it now (I'll initialize with template)
B) Skip structure update (not recommended)
C) Specify custom location

Which option?
```

If user chooses A, create file with template structure.

### If user wants to customize principles:

Inform user:
```
File length limit: 750 lines (from CODEBASE PRINCIPLES above)

To customize:
- Edit .claude/commands/codebase-check.md
- Modify "CODEBASE PRINCIPLES (Reference)" section
- Adjust limits, exceptions, or add new rules
```

## Error Handling

**If git diff fails** (not a git repo):
- Ask user to manually specify modified files

**If file too large to read**:
- Use `wc -l [file]` to count lines
- Report to user without full context

**If PM agent times out**:
- Report to user, suggest manual refactoring review

## Constraints

- **Never auto-refactor** without PM consultation + user approval
- **Always update structure doc** after validation
- **Always report findings** even if no action needed
- **Preserve user context** - don't pollute with unnecessary reads

## Success Criteria

✅ All modified files validated against 750-line limit
✅ CODEBASE_STRUCTURE.md updated with new entries
✅ User informed of any violations with PM-backed refactoring plan
✅ Clear next steps provided
