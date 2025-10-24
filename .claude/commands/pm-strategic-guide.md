# PM Strategic Guide Methodology

This methodology is loaded when you execute `/pm-strategic-guide`.

## When to Use This

**Main Thread Request Pattern**:
- "User wants [feature]. Provide strategic guide."
- "Analyze if [feature] aligns with vision."

This loads Type A consultation methodology for providing strategic implementation direction.

## Methodology

**When Main Thread consults**: "User wants to add [feature]. Provide strategic guide."

**Your process**:

### Step 1: Dynamic Discovery (see Discovery Protocol in base prompt)

Execute discovery protocol:
```bash
# Technology Stack
Glob: ideas*/**/eg-desk*ideas*/*tech*.md
Read: [discovered-path]/technology-stack.md

# Vision Documentation
Glob: ideas*/**/eg-desk*ideas*/**/*.md

# EG-DESK Custom Code
Glob: eg-desk*/**/*.ts
Glob: eg-desk*/CODEBASE_STRUCTURE.md

# Theia Framework
Glob: packages/*/package.json

# Implementation Status Check
Grep: [feature-name] in eg-desk*/**/*.ts
Grep: [feature-name] in packages/
```

### Step 2: Vision Analysis
1. Read relevant vision documents
2. Extract principles applicable to this feature
3. Check institutional memory (previous decisions on similar features)

### Step 3: Implementation Status Check
1. **Check EG-DESK custom code**: Grep feature name in eg-desk*/**/*.ts
2. **Check Theia framework**: Grep feature name in packages/
3. **Check CODEBASE_STRUCTURE.md**: Read registry for similar implementations
4. **Determine**: New implementation vs enhancement vs duplicate

### Step 4: Technology Stack Analysis
1. Match user requirements to discovered technology capabilities
2. Identify primary technology (main framework for feature)
3. Identify secondary technologies (supporting frameworks)
4. Check if new technology needed (not in current stack)

### Step 5: Project Structure Analysis
1. Discover which package this feature belongs to (eg-desk_taehwa/ vs packages/)
2. Find similar existing implementations for patterns
3. Identify integration points with existing systems

### Step 6: Strategic Decision Framework
Apply these questions:
1. **Vision Alignment**: Does this align with ambient AI workspace principles?
2. **UX Consistency**: Does this match spatial, ephemeral, proximity-based interaction?
3. **Technical Fit**: Does this leverage discovered technologies appropriately?
4. **Implementation Status**: Is this a new feature, enhancement, or duplicate?
5. **Competitive Advantage**: Does this strengthen EG-DESK's unique position?
6. **User Value**: Does this solve real knowledge worker pain points?

### Step 7: Provide Strategic Guide
- **Decision**: APPROVE / MODIFY / REJECT
- **Technology Stack**: Which technology/technologies from discovered stack (match capabilities to requirements)
- **Location**: Exact package and directory (eg-desk_taehwa/ or packages/)
- **Implementation approach**: High-level phasing and integration strategy
- **Considerations**: Important factors Main Thread must address
- **Create PRD** (if approved): Write feature PRD to `ideas/eg-desk ideas/features/`
- **Update tech stack** (if new tech proposed and approved): Add to technology-stack.md

## Code Location Guidance

### CRITICAL DISTINCTION: Two separate codebases

1. **EG-DESK custom code** (your custom implementations)
2. **Theia framework code** (base framework packages)

### EG-DESK Custom Codebase

**Purpose**: Custom features and extensions specific to EG-DESK application

**Discovery**:
```bash
# Find EG-DESK custom codebase root (NOT packages/)
Glob: eg-desk*/**/*.ts
Glob: eg-desk*/**/*.tsx
# This reveals the root directory, e.g.:
# - eg-desk_taehwa/
# - eg-desk-custom/
# - eg-desk/
```

**Structure document**:
```bash
# Find codebase structure tracking
Glob: eg-desk*/CODEBASE_STRUCTURE.md
```

**Code location decisions for EG-DESK features**:
- Custom services → `[eg-desk-root]/[feature]/[feature]-service.ts`
- Custom contributions → `[eg-desk-root]/[feature]/[feature]-contribution.ts`
- Feature-specific code → `[eg-desk-root]/[feature]/`
- **DO NOT put EG-DESK custom code in packages/** (that's Theia framework)

### Theia Framework Packages

**Purpose**: Base Theia framework (upstream code)

**Discovery**:
```bash
# Discover all Theia packages
Glob: packages/*/package.json

# For each package, read package.json to understand:
- Package name and purpose
- Dependencies
- Directory structure (src/browser, src/node, src/common, etc.)
```

**When to modify Theia packages**:
- Extending framework services (rare)
- Fixing upstream bugs
- Generally: **prefer EG-DESK custom code over modifying packages/**

**Code location decision process**:
1. Is this EG-DESK custom feature? → Use `eg-desk_taehwa/` (or discovered root)
2. Extending Theia service? → Depends:
   - Custom wrapper service → `eg-desk_taehwa/`
   - Direct Theia modification → `packages/[package]/`
3. When in doubt: Glob both locations to find similar features

## Implementation Status Discovery

**CRITICAL**: Check BOTH EG-DESK custom AND Theia framework code

### Check EG-DESK Custom Code First

```bash
# Find existing custom implementations
Glob: eg-desk*/**/*[feature-name]*.ts
Glob: eg-desk*/**/*theme*.ts (example)

# Check structure document for conflicts
Read: eg-desk*/CODEBASE_STRUCTURE.md
Grep: [feature-name] in CODEBASE_STRUCTURE.md
```

### Check Theia Framework Patterns

```bash
# Find Theia framework patterns
Glob: packages/*/src/**/*theme*.ts (example)
Glob: packages/*/src/**/*[feature-name]*.ts

# Read package.json files
Read: packages/[relevant-package]/package.json

# Check for similar features
Grep: "class.*ThemeSwitcher" (example pattern)
```

**Analysis process**:
1. **First**: Check EG-DESK custom codebase (avoid duplicate custom implementations)
2. **Second**: Check Theia framework (understand patterns to follow)
3. Identify gaps (what needs to be added)
4. Specify exact locations for new code:
   - Custom feature → `eg-desk_taehwa/[feature]/`
   - Framework extension → Consider carefully (prefer custom over modifying packages/)

## PRD & Ideas File Management

**File Types**:
- **PRD**: `features/[feature-name]-prd.md` (approved for "바로 구현")
- **Brainstorming**: `brainstorming/YYYY-MM-DD_HHMM_[feature-slug].md` (deferred "나중에 구현")

**PRD template structure**:
```markdown
# [Feature Name] - PRD

## Vision Alignment
[How this aligns with EG-DESK vision]

## User Value
[Problem being solved]

## Technical Approach
**Framework**: [Theia/Electron/Both]
**Location**: `packages/[package]/src/[path]/`
**Key Components**:
- [Component 1]
- [Component 2]

## Implementation Phases
1. Phase 1: [...]
2. Phase 2: [...]

## Decision Rationale
[Why these technical choices]

## References
- Vision doc: [path]
- Similar implementation: [path]
```

**Brainstorming Idea template structure**:
```markdown
# [Feature Name] - Brainstorming Idea

**Status**: 💭 Brainstorm (Not Yet Approved)
**Created**: YYYY-MM-DD HH:MM KST
**Last Updated**: YYYY-MM-DD HH:MM KST
**User Position at This Time**: [Tentative / Strong Interest / Needs Research]

## Context at This Moment

**Why This Idea Emerged**:
[What prompted this - user request, pain point discovered, competitive insight, etc.]

**User's Current Thinking**:
[Direct quotes or paraphrasing of user's position RIGHT NOW]

## Idea Summary

[2-3 sentence description]

## User Rationale (Timestamped)

**User's Reasoning** (YYYY-MM-DD HH:MM):
- [Point 1 user made]
- [Point 2 user emphasized]
- [Concern user raised]

## Vision Alignment Assessment (PM's View at This Time)

**Alignment** (✅/⚠️/❌):
- ✅ **Aligns**: [How it fits vision]
- ⚠️ **Tensions**: [Where it conflicts or unclear]

**Alternative Suggested by PM** (if applicable):
[PM's alternative proposal at this time]

## Future Decision Points

**What Needs to Happen Before Implementation**:
- [ ] [Prerequisite 1 - e.g., "Complete Phase 2 of spatial canvas"]
- [ ] [Prerequisite 2 - e.g., "User research validates need"]
- [ ] [Decision needed - e.g., "Resolve tension with vision principle X"]

## Evolution History

### YYYY-MM-DD HH:MM - Initial Brainstorm
- User position: [Summary]
- PM assessment: [Summary]

[Future entries when idea is revisited:]
### YYYY-MM-DD HH:MM - Position Update
- User position changed: [How]
- New context: [What changed]
- PM re-assessment: [Updated view]
```

**When user revisits same brainstorming idea**:
1. **Find existing brainstorm**: `Glob: ideas*/**/brainstorming/*[keyword]*.md`
2. **Append to Evolution History section** with new timestamp
3. **Update header**: "Last Updated" timestamp and current user position
4. **Do NOT create duplicate file** - always update existing brainstorm

## Output Format

### Type A: Strategic Guide Report (Initial Consultation)

```markdown
## EG-DESK PM: Strategic Guide

### Summary (Concise)
[2-3 sentences: What was requested, decision (APPROVE/MODIFY/REJECT), recommended technology stack & location]

### Context Recall (Institutional Memory)
**Previous Decisions:**
- [Related previous decisions: "We decided X in document Y"]
- [Or: "No previous decisions on this topic"]

**Relevant Vision Principles:**
- [Quote key principles from vision docs]

### Technology Stack Available (Discovered)
**Technologies Found** (from technology-stack.md):
- [List categories and tech names discovered from document]
- Example: "IDE Framework: Theia; Canvas System: Infinite Canvas, Konva.js; AI: Claude API"
- **Important**: This list is discovered dynamically, NOT hardcoded

### Strategic Guide (Fully Detailed)

**Decision**: [APPROVE / MODIFY / REJECT]

**Technology Stack Selection**:
- **Primary Technology**: [Technology Name] ([Category from tech stack doc])
  - **Capabilities Used**: [List relevant capabilities from tech stack doc]
  - **Rationale**: [Why this technology matches user requirements]
  - **Documentation**: [Link/path from tech stack doc]

- **Secondary Technology** (if multi-tech feature):
  - [Technology Name] ([Category])
  - **Integration Point**: [How it works with primary technology]
  - **Rationale**: [Why secondary tech needed]

- **Custom Implementation** (if needed):
  - **Scope**: [What requires custom code beyond existing technologies]
  - **Rationale**: [Why existing technologies insufficient]

- **Technology Not Available** (if applicable):
  - ⚠️ User requirement needs [NewTechnology] not in current stack
  - **Options**:
    - Option A: Use alternative [ExistingTech] (tradeoffs: ...)
    - Option B: Add [NewTechnology] to stack (requires research phase)
  - **User decision required**: Which option to proceed with?

**Code Location**:
- **Package**: `eg-desk_taehwa/[feature]/` or `packages/[package-name]/`
- **Directory**: Full path based on technology choice
- **Rationale**: [Why this location based on technology stack + existing structure discovered via Glob]

**Implementation Approach**:
- **Phase 1**: [What to do first - usually vision validation + pattern discovery]
- **Phase 2**: [Architecture design based on framework agent findings]
- **Phase 3**: [Implementation]
- **Integration Points**: [How this connects with existing systems]

**Critical Considerations**:
- [Important factor 1 Main Thread must address]
- [Important factor 2 related to existing implementations]
- [Edge case or constraint from vision]

**Project Structure Discovered**:
- **Existing Similar Features**: [Found via Glob - reference implementations]
- **Related Packages**: [Dependencies discovered in package.json]
- **Patterns to Follow**: [Existing patterns from codebase]

### Documentation Actions Taken

**PRD Created** (if APPROVED):
- File: `ideas&external_references/eg-desk ideas/features/[feature-name]-prd.md`
- Content: [Brief summary of PRD contents]

**Vision Docs Updated** (if applicable):
- File: `ideas&external_references/eg-desk ideas/[doc].md`
- Changes: [What was updated]

**No Documentation Changes** (if REJECTED or waiting for user input)

### References for Main Thread

**Vision Documents Analyzed**:
- `ideas&external_references/eg-desk ideas/[doc1].md` - [Key principle extracted]

**Code Structure Discovered**:
- `packages/[package]/` - [Similar feature found here]

**Next Steps for Main Thread**:
1. [Create execution plan based on this guide]
2. [Query framework agents if needed: theia-analyzer-agent for X]
3. [Return to PM for plan review (optional) or proceed to implementation]
```

## Example

**Request**: "Add time-based terminal theme"

**Response** (using Type A format above):

## EG-DESK PM: Strategic Guide

**Summary**: Time-based terminal theming approved. Theia's theme system in packages/terminal. Automatic with manual override.

**Decision**: APPROVE

**Tech Stack**: Eclipse Theia (IDE Framework) - TerminalThemeService at packages/terminal/src/browser/terminal-theme-service.ts

**Location**: `packages/terminal/src/browser/`

**Phases**:
1. Query theia-analyzer-agent for theme patterns
2. Design TimeBasedThemeSwitcher service
3. Implement & integrate

**Considerations**: Manual override, preference persistence, smooth transitions

**PRD Created**: `ideas&external_references/eg-desk ideas/features/time-based-terminal-theme-prd.md`
