# Subagent Reports Directory

This directory contains **file-based reports** from subagents to Main Thread.

## Purpose

**Problem**: Subagents are stateless and don't retain conversation history.

**Solution**: File-based reporting system where:
1. Subagent creates report file when task assigned
2. Main Thread reads report and adds comments/requests
3. Subagent reads previous report + comments, updates report
4. Cycle continues until task complete
5. Main Thread archives/deletes report when resolved

## Report Lifecycle

```
┌─────────────────────────────────────────────────────────────┐
│ Phase 1: Report Creation (Subagent)                        │
│ - Subagent receives task from Main Thread                  │
│ - Creates: subagent_reports/[topic]-[agent-name].md        │
│ - Initial findings documented                              │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│ Phase 2: Main Thread Review                                │
│ - Read: subagent_reports/[topic]-[agent-name].md          │
│ - Adds <!-- MT: ... --> comments                           │
│ - Requests clarification or additional investigation       │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│ Phase 3: Subagent Update (Multi-turn)                     │
│ - Re-invoke subagent with: "Update report at [path]"      │
│ - Subagent reads entire report (including MT comments)     │
│ - Addresses MT requests, updates findings                  │
│ - Appends update to report                                 │
└─────────────────────────────────────────────────────────────┘
                            ↓
                   (Repeat Phase 2-3 as needed)
                            ↓
┌─────────────────────────────────────────────────────────────┐
│ Phase 4: Resolution & Cleanup (Main Thread)               │
│ - Task complete (research done, PRD created, error fixed)  │
│ - Main Thread deletes report: rm subagent_reports/[file]  │
│ - OR archives if institutionally valuable                  │
└─────────────────────────────────────────────────────────────┘
```

## File Naming Convention

**Format**: `YYYYMMDD_HHMM_[topic]-[agent-name].md`

**Examples**:
- `20251021_1630_threejs-research-pm.md` (PM agent researching Three.js)
- `20251021_1445_terminal-theme-analysis-theia-analyzer.md` (Theia analyzer)
- `20251021_0920_menu-implementation-coding-agent.md` (coding-agent)
- `20251022_1115_canvas-error-recovery-error-recovery.md` (error-recovery-agent)

**Why datetime prefix?**
- Allows multiple reports on same topic over time
- Easy chronological sorting (ls shows oldest→newest)
- Clear timestamp of when investigation started
- Easy to identify stale reports (>7 days old = YYYYMMDD far in past)

## Report Template (Streamlined - Fast to Write)

**Minimal format** - subagent fills in ONLY what's needed, no extra ceremony:

```markdown
# [Topic]

**Agent**: [name] | **Created**: YYYY-MM-DD HH:MM | **Updated**: YYYY-MM-DD HH:MM

## Task
[What Main Thread requested - 1-2 sentences]

## Findings
[Investigation results - bullet points preferred, no need for prose]

<!-- MT: [Main Thread comments go here as HTML comments] -->

---
## Updates

**YYYY-MM-DD HH:MM**:
[What changed this update - brief]

**YYYY-MM-DD HH:MM**:
[Next update - only if multi-turn needed]
```

**Key Principles**:
- ✅ **Minimal headers**: Just Task, Findings, Updates
- ✅ **Bullet points over paragraphs**: Faster to read/write
- ✅ **MT comments inline**: `<!-- MT: ... -->` wherever relevant
- ✅ **No redundant sections**: Only add Updates if multi-turn happens
- ✅ **Datetime in one line**: Created + Updated in header
- ❌ **No "status" field**: File existence = in progress, deletion = complete
- ❌ **No "resolution status"**: MT just deletes when done

**Example (Quick Write)**:

```markdown
# Three.js Bundle Size Analysis

**Agent**: egdesk-pm-agent | **Created**: 2025-10-21 16:30 | **Updated**: 2025-10-21 16:30

## Task
Analyze Three.js bundle size impact for spatial canvas 3D rendering.

## Findings
- Core: 600KB (gzipped)
- With GLTFLoader + OrbitControls: 720KB
- Performance budget: <500KB (per tech requirements)
- **Exceeds budget by 220KB**

Options:
1. Use tree-shaking → Reduce to ~450KB (meets budget)
2. Lazy-load 3D features → Core bundle unaffected
3. Alternative: Babylon.js (~1.2MB - worse)

**Recommendation**: Option 1 (tree-shaking) + Option 2 (lazy-load)

<!-- MT: Good. Also check if we can defer 3D until user explicitly requests it. -->
```

**Example (Multi-turn with Update)**:

```markdown
# Terminal Theme DI Pattern

**Agent**: theia-analyzer-agent | **Created**: 2025-10-21 14:45 | **Updated**: 2025-10-21 17:20

## Task
Find Theia's DI pattern for terminal theming.

## Findings
- Pattern at `packages/terminal/src/browser/terminal-theme-service.ts:45`
- Uses `@injectable()` decorator
- Lifecycle: onStart() hook in contribution
- Files to modify:
  - CREATE: `time-based-switcher.ts`
  - MODIFY: `terminal-frontend-module.ts:67` (DI binding)

<!-- MT: What about preference persistence? Need PreferenceService integration pattern. -->

---
## Updates

**2025-10-21 17:20**:
Found PreferenceService pattern at `packages/preferences/src/browser/preference-service.ts:89`
- Use `preferences.set('theme.current', themeId)`
- Integration: Inject PreferenceService into TimeBasedSwitcher
- Example at `packages/workspace/src/browser/workspace-service.ts:120`
```

## Main Thread Responsibilities

### Creating Reports
- Subagents automatically create reports when invoked
- Main Thread does NOT manually create reports

### Reviewing Reports
```bash
# After subagent completes initial investigation
Read: subagent_reports/[topic]-[agent-name]-YYYYMMDD.md
```

### Adding Comments
```bash
# Add MT comments inline
Edit: subagent_reports/[topic]-[agent-name]-YYYYMMDD.md
# Insert: <!-- MT (2025-10-21 17:00): Need more detail on integration approach -->
```

### Re-invoking Subagent
```bash
# For follow-up investigation
Task(agent: "theia-analyzer-agent",
     prompt: "Update your report at subagent_reports/terminal-theme-analysis-theia-analyzer-20251021.md

             Address these Main Thread requests:
             - [Request 1 from MT comment]
             - [Request 2 from MT comment]")
```

### Cleanup (When Complete)
```bash
# Task resolved - delete report
Bash: rm subagent_reports/threejs-research-pm-20251021.md

# OR archive if valuable for institutional memory
Bash: mv subagent_reports/threejs-research-pm-20251021.md ideas&external_references/threejs-research.md
```

## When to Delete Reports

**DELETE immediately when:**
- ✅ Research incorporated into PRD
- ✅ Error resolved and code fixed
- ✅ Investigation complete, no further action needed
- ✅ Task abandoned/cancelled

**ARCHIVE (move to ideas&external_references/) when:**
- ⚠️ Research findings valuable for future reference
- ⚠️ Contains institutional knowledge worth preserving
- ⚠️ May be revisited later

**Example Archive Workflow**:
```bash
# Research complete, findings valuable
Bash: mv subagent_reports/threejs-research-pm-20251021.md ideas&external_references/threejs-research.md

# Update research doc to reflect it was from subagent report
Edit: ideas&external_references/threejs-research.md
# Add header: "Originally from subagent_reports/ on 2025-10-21"
```

## Report Hygiene

**Keep this directory clean:**
- ❌ No stale reports (>7 days old without updates)
- ❌ No resolved tasks lingering
- ✅ Only active, in-progress work
- ✅ Regular cleanup by Main Thread

**Weekly cleanup check**:
```bash
# List all reports older than 7 days
Bash: find subagent_reports -name "*.md" -mtime +7

# Review each:
# - Still active? Keep it.
# - Resolved? Delete it.
# - Abandoned? Delete it.
```

## Examples

### Example 1: PM Research → PRD (Delete)

1. PM creates `20251021_1630_threejs-research-pm.md`
2. MT adds comment "Need bundle size analysis"
3. PM updates report with bundle size findings
4. MT: Research complete, incorporated into PRD
5. MT: `rm subagent_reports/20251021_1630_threejs-research-pm.md`

### Example 2: Error Analysis → Archive

1. error-recovery creates `20251021_0920_build-failure-error-recovery.md`
2. MT: "What's the root cause?"
3. Agent: Identifies root cause in DI configuration
4. MT: Error fixed, but diagnosis process valuable
5. MT: `mv subagent_reports/20251021_0920_build-failure-error-recovery.md ideas&external_references/di-error-diagnosis.md`

### Example 3: Multi-turn Investigation

1. **Day 1**: theia-analyzer creates `20251021_1445_terminal-analysis-theia-analyzer.md`
2. **Day 1**: MT adds 3 clarification requests via `<!-- MT: ... -->`
3. **Day 2**: Agent updates report (addresses 2/3)
4. **Day 2**: MT adds 1 more request
5. **Day 3**: Agent final update (all addressed)
6. **Day 3**: MT: `rm subagent_reports/20251021_1445_terminal-analysis-theia-analyzer.md`

---

## Technical Notes

**File Format**: Markdown (.md)
- Easy to read/edit in any text editor
- Git-trackable (version history)
- Supports inline comments

**Comment Syntax**: `<!-- MT: ... -->`
- HTML comments (invisible in rendered Markdown)
- Main Thread prefix clearly identifies who added it
- Include timestamp for multi-turn clarity

**Concurrent Reports**: Same agent can have multiple active reports
- Different topics get different files
- Date suffix prevents conflicts

---

This system ensures **clear, persistent communication** between stateless subagents and Main Thread, eliminating prompt repetition and maintaining institutional memory of investigation processes.
