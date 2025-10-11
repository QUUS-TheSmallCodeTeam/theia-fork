# Agent Swarm Flow: From Ideation to Implementation

This document maps the complete agent orchestration system for the EG-DESK project through concrete workflow scenarios, showing exactly who does what at each step.

## System Overview

```
User Request
     │
     ▼
Main Thread (analyzes & routes)
     │
     ├─ Simple? → Execute directly
     ├─ Framework question? → Direct to framework analyzer agent
     ├─ New agent? → Invoke claude-agent-sdk-analyzer-agent
     │
     └─ Development task? → PM-driven workflow:
            │
            ▼
         PM Agent (Strategic Guide)
            ├─ Vision alignment
            ├─ Framework selection (Theia/Electron/Both)
            ├─ Code location (packages/X/src/Y/)
            ├─ Implementation phasing
            ├─ Creates PRD (if approved)
            └─ Returns strategic guide to Main Thread
                 │
                 ▼
         Main Thread (Executor)
            ├─ Creates plan based on PM's guide
            ├─ Queries framework agents (per PM's direction)
            ├─ (Optional) Returns to PM for plan review
            ├─ Synthesizes PM guide + framework patterns
            └─ Spawns coding-agent(s):
                 • Direction (what to implement)
                 • File list (CREATE/MODIFY/DELETE/REFERENCE)
                 • coding-agent reads files for details
            │
            ▼
         Main Thread builds, tests, commits

Roles: PM guides strategy → Main Thread executes → coding-agent implements
```

## Critical Architecture Principle: Metaphysical Separation

**Main Thread** operates at a **meta-level** when orchestrating - it coordinates agents but delegates domain-specific file reading to specialized agents.

### File Reading Hierarchy:

```
┌─────────────────────────────────────────────────┐
│ Meta Level (Orchestration)                      │
│                                                  │
│ Main Thread:                                    │
│  • Routes requests                              │
│  • Identifies agents (from system prompt)       │
│  • Creates mission prompts (orchestration)      │
│  • Invokes agents                               │
│  • Synthesizes results                          │
│  • Writes code (implementation)                 │
│  • Preserves context (delegates heavy reading)  │
│                                                  │
│ Can read for orchestration:                     │
│  • .claude/prompts/ (orchestration guidelines)  │
│  • .claude/agents/ (OPTIONAL - only for details)│
│                                                  │
│ Cannot read when orchestrating:                 │
│  • vision docs, codebase (delegate to agents)   │
└─────────────────────────────────────────────────┘
         │
         │ (delegates to)
         ▼
┌─────────────────────────────────────────────────┐
│ Domain Level (Analysis & Expertise)             │
│                                                  │
│ PM Agent: reads vision docs                     │
│ Framework Agents: read codebase/docs            │
│ SDK Analyzer: reads SDK docs                    │
│                                                  │
│ Each agent reads ONLY its domain files          │
└─────────────────────────────────────────────────┘
```

## Workflow Scenarios

### Scenario 1: Simple Question (No Agents)

**User Request:**
```
"What files are in the packages/ai-chat directory?"
```

**Step-by-Step Flow:**

| Step | Entity | Action | Tools Used | Output |
|------|--------|--------|------------|--------|
| 1 | User | Asks question | None | Request sent |
| 2 | Main Thread | Analyzes: Simple file listing | None | Routes to direct execution |
| 3 | Main Thread | Lists files | `Glob("packages/ai-chat/**/*")` | File list |
| 4 | Main Thread | Returns answer to user | None | User sees file list |

**Total Agents Invoked:** 0

**Who Wrote Code:** Nobody (read-only operation)

**Duration:** Immediate (single step)

---

### Scenario 2: Framework Question (Direct Agent)

**User Request:**
```
"How does Theia's dependency injection work?"
```

**Step-by-Step Flow:**

| Step | Entity | Action | Tools Used | Output |
|------|--------|--------|------------|--------|
| 1 | User | Asks framework question | None | Request sent |
| 2 | Main Thread | Analyzes: Theia framework question | None | Routes to theia-analyzer-agent |
| 3 | Main Thread | Invokes agent | `Task(agent: "theia-analyzer-agent", prompt: "Analyze Theia's DI system in packages/core/")` | Agent starts |
| 4 | theia-analyzer-agent | Reads DI implementation | `Read("packages/core/src/common/di.ts")`<br>`Grep("@injectable")` | Finds patterns |
| 5 | theia-analyzer-agent | Analyzes examples | `Glob("**/*-frontend-module.ts")`<br>`Read(example files)` | Understands usage |
| 6 | theia-analyzer-agent | Returns report | None | Detailed explanation with file refs |
| 7 | Main Thread | Presents to user | None | User sees explanation |

**Total Agents Invoked:** 1 (theia-analyzer-agent)

**Who Wrote Code:** Nobody (analysis only)

**Duration:** Single agent invocation

**Key Point:** Main Thread directly invoked the framework agent without going through swarm manager (efficiency optimization for single-domain questions).

---

### Scenario 3: Strategic Decision with Insight Provision (Vision Conflict)

**User Request:**
```
"Should we add a floating AI assistant that follows the mouse cursor?"
```

**Step-by-Step Flow:**

| Step | Entity | Action | Tools Used | Output |
|------|--------|--------|------------|--------|
| 1 | User | Proposes feature idea | None | Request sent |
| 2 | Main Thread | Analyzes: EG-DESK feature, needs PM strategic guide | None | Follows orchestration guidelines |
| 3 | Main Thread | Invokes PM agent | `Task(agent: "egdesk-pm-agent", prompt: "User wants floating AI assistant that follows mouse cursor. Provide strategic guide.")` | PM agent starts |
| 4 | egdesk-pm-agent | Discovers vision docs | `Glob("ideas&external_references/eg-desk ideas/**/*.md")` | Finds UX and whitepaper docs |
| 5 | egdesk-pm-agent | Analyzes vision docs | `Read("EG-DESK_Whitepaper.md")`<br>`Read("EG-DESK_Spatial_Canvas_UX_Solutions.md")`<br>`Grep("spatial", "proximity", "floating")` | Finds: Previous decision against floating UI |
| 6 | egdesk-pm-agent | Evaluates alignment | None (analysis) | Decision: REJECT - conflicts with spatial canvas principles |
| 7 | egdesk-pm-agent | **Provides insight to user** (not just reject) | None | Explains conflict, suggests alternative |
| 8 | egdesk-pm-agent | Returns strategic guide | None | **Summary**: Floating cursor-following AI conflicts with spatial canvas principles.<br>**Decision**: REJECT<br>**Insight**: "In EG-DESK_Spatial_Canvas_UX_Solutions.md, we decided against floating/following UI because it breaks spatial affordances. Users lose sense of place when elements follow cursor."<br>**Alternative**: Proximity-based AI activation - AI appears NEAR relevant canvas objects, not following cursor<br>**Why alternative is better**: Maintains spatial relationships while providing contextual AI |
| 9 | Main Thread | Presents insight to user | None | User understands WHY it conflicts and sees vision-aligned alternative |

**Total Agents Invoked:** 1 (egdesk-pm-agent)

**Who Wrote Code:** Nobody (strategic decision only)

**Duration:** Single agent invocation

**Key Points:**
- **PM provides insight, not just rejection**: Explains why conflict exists
- **Institutional memory**: "We previously decided X in document Y because Z"
- **Alternative suggestion**: Vision-aligned approach that solves same user need
- **User understands vision**: Not just "no", but "no because... and here's better way"
- **Context Preservation**: Main Thread didn't read vision docs - PM synthesized everything

---

### Scenario 4: Development with PM Strategic Guide

**User Request:**
```
"Add a custom terminal theme that changes based on time of day"
```

**Step-by-Step Flow:**

| Step | Entity | Action | Tools Used | Output |
|------|--------|--------|------------|--------|
| **PHASE 0: PM STRATEGIC GUIDE** |
| 1 | User | Requests feature | None | Request sent |
| 2 | Main Thread | Analyzes: Development task, needs strategic direction | None | Routes to PM for initial guide |
| 3 | Main Thread | Invokes PM agent | `Task(agent: "egdesk-pm-agent", prompt: "User wants time-based terminal theme. Provide strategic guide.")` | PM agent starts |
| 4 | egdesk-pm-agent | Dynamic discovery | `Glob("ideas&external_references/eg-desk ideas/**/*.md")`<br>`Glob("packages/*/package.json")`<br>`Grep("theme", "terminal")` | Finds vision docs + existing theme code |
| 5 | egdesk-pm-agent | Analyzes vision + structure | `Read("EG-DESK_Whitepaper.md")`<br>`Read("packages/terminal/package.json")`<br>`Read("packages/terminal/src/browser/terminal-theme-service.ts")` | Extracts principles + discovers structure |
| 6 | egdesk-pm-agent | Creates PRD | `Write("ideas&external_references/eg-desk ideas/features/time-based-terminal-theme-prd.md", "[PRD content]")` | PRD file created |
| 7 | egdesk-pm-agent | Returns strategic guide | None | **Decision**: APPROVE<br>**Framework**: Theia (terminal theming is Theia domain)<br>**Location**: `packages/terminal/src/browser/`<br>**Approach**: Phase 1 - analyze theme system, Phase 2 - design service, Phase 3 - implement<br>**Considerations**: Manual override needed, preference persistence<br>**PRD Created**: time-based-terminal-theme-prd.md |
| **PHASE 1: MAIN THREAD CREATES PLAN** |
| 8 | Main Thread | Creates execution plan based on PM guide | None (planning) | Plan: Query theia-analyzer for theme patterns, design TimeBasedSwitcher, implement |
| **PHASE 2: FRAMEWORK INVESTIGATION** |
| 9 | Main Thread | Queries framework agent (per PM's guide) | `Task(agent: "theia-analyzer-agent", prompt: "Analyze terminal theme system at packages/terminal/src/browser/terminal-theme-service.ts for registration and DI patterns")` | Agent starts |
| 10 | theia-analyzer-agent | Analyzes theme system | `Read("packages/terminal/src/browser/terminal-theme-service.ts")`<br>`Read("packages/terminal/src/browser/terminal-frontend-module.ts")`<br>`Grep("@injectable")` | Finds patterns |
| 11 | theia-analyzer-agent | Returns analysis | None | **Files Analyzed**: terminal-theme-service.ts:45, terminal-frontend-module.ts:32<br>**Pattern**: ThemeService.register() with DI binding<br>**File List**: CREATE time-based-theme-switcher.ts, MODIFY terminal-frontend-module.ts:36, REFERENCE workspace-service.ts:89 for @injectable() |
| **PHASE 3: IMPLEMENTATION (Main Thread)** |
| 12 | Main Thread | Implements directly (small change) | `Write("time-based-theme-switcher.ts")`<br>`Edit("terminal-frontend-module.ts")`<br>`Edit("terminal-contribution.ts")` | Code written |
| 13 | Main Thread | Builds, tests, commits | `Bash("npm run build && git add . && git commit")` | Committed |

**Total Agents Invoked:** 2 (PM strategic guide → framework analyzer)

**Who Wrote Code:** Main Thread directly (small implementation)

**Duration:** Multi-phase (PM guide → plan → framework investigation → implementation)

**Critical Observations:**
- **PM provides complete strategic direction**: Framework, location, approach, considerations
- **PM creates PRD**: Documents approved feature
- **Main Thread creates plan** based on PM's guide
- **Framework agent** provides technical patterns (not strategic direction)
- **Main Thread implements** following both PM's strategy and framework patterns

**Note**: This shows direct implementation. For larger changes, Main Thread would delegate to coding-agent (see Scenario 4b).

---

### Scenario 4b: Development with PM Guide + Coding Agent Delegation

**User Request:**
```
"Add a custom terminal theme that changes based on time of day"
```

**Why use coding-agent:** Large implementation (3+ files), Main Thread context needs to stay clean for continued orchestration

**Step-by-Step Flow:**

| Step | Entity | Action | Tools Used | Output |
|------|--------|--------|------------|--------|
| **PHASE 0-2: SAME AS SCENARIO 4 (Steps 1-11)** |
| 1-11 | Various | PM provides strategic guide → Main Thread creates plan → Framework agent investigates | (See Scenario 4) | Strategic guide + Framework patterns collected |
| **PHASE 3: IMPLEMENTATION (Delegated to Coding Agent)** |
| 12 | Main Thread | Synthesizes PM guide + framework findings | None (internal) | Direction: Create TimeBasedThemeSwitcher<br>File List: CREATE time-based-theme-switcher.ts, MODIFY terminal-frontend-module.ts:36, terminal-contribution.ts:89, REFERENCE workspace-service.ts:89 |
| 13 | Main Thread | Delegates to coding agent | `Task(agent: "coding-agent", prompt: "Implement time-based terminal theme switcher.<br><br>Direction: Create TimeBasedThemeSwitcher service following Theia DI pattern with automatic time-based switching + manual override + preference persistence.<br><br>Files:<br>CREATE: packages/terminal/src/browser/time-based-theme-switcher.ts<br>MODIFY: packages/terminal/src/browser/terminal-frontend-module.ts:36 (add DI binding)<br>MODIFY: packages/terminal/src/browser/terminal-contribution.ts:89 (inject service)<br>REFERENCE: packages/workspace/src/browser/workspace-service.ts:89 (follow @injectable() pattern)<br><br>[You will read files for implementation details]")` | Coding agent starts |
| 14 | coding-agent | Reads pattern reference | `Read("packages/workspace/src/browser/workspace-service.ts")` | Understands DI pattern |
| 15 | coding-agent | Creates service file | `Write("packages/terminal/src/browser/time-based-theme-switcher.ts", "[code following pattern]")` | New file created |
| 16 | coding-agent | Reads module file | `Read("packages/terminal/src/browser/terminal-frontend-module.ts")` | Current structure |
| 17 | coding-agent | Registers in DI | `Edit(old: "export default", new: "bind(TimeBasedThemeSwitcher)...")` | DI binding added |
| 18 | coding-agent | Reads contribution file | `Read("packages/terminal/src/browser/terminal-contribution.ts")` | Current structure |
| 19 | coding-agent | Integrates service | `Edit(old: "export class", new: "@inject(TimeBasedThemeSwitcher)...")` | Integration complete |
| 20 | coding-agent | Returns report | None | **Implementation Complete**: 1 CREATE, 2 MODIFY following workspace-service.ts pattern |
| **PHASE 4: VERIFICATION (Main Thread)** |
| 21 | Main Thread | Builds package | `Bash("npm run build")` | Build succeeds |
| 22 | Main Thread | Stages and commits | `Bash("git add packages/terminal && git commit -m 'feat(terminal): add time-based theme switcher'")` | Committed |
| 23 | Main Thread | Reports to user | None | "Feature implemented and committed" |

**Total Agents Invoked:** 3 (PM strategic guide → framework analyzer → coding-agent)

**Who Wrote Code:** coding-agent (steps 15-19)

**Duration:** Multi-phase with coding delegation

**Critical Observations:**
- **PM's strategic guide** drove entire workflow (framework choice, location, approach)
- **Main Thread synthesized** PM guide + framework patterns into precise coding-agent instructions
- **coding-agent received**:
  - Clear direction (what to implement)
  - File list with actions (CREATE/MODIFY/REFERENCE)
  - Pattern references (which files to learn from)
  - Implementation details left to coding-agent (reads files as needed)
- **Main Thread's context** stayed clean - coding-agent handled all file reading/editing
- **Main Thread** retained control of build/test/commit

**When to use this pattern:**
- Large implementations (3+ files)
- Main Thread needs to stay available for orchestration
- Multiple features being developed in parallel
- Context preservation is critical

---

### Scenario 5: Agent Creation Request

**User Request:**
```
"We need an agent that analyzes Konva.js integration patterns"
```

**Step-by-Step Flow:**

| Step | Entity | Action | Tools Used | Output |
|------|--------|--------|------------|--------|
| 1 | User | Requests new agent | None | Request sent |
| 2 | Main Thread | Analyzes: Agent creation task | None | Routes to claude-agent-sdk-analyzer-agent |
| 3 | Main Thread | Invokes claude-agent | `Task(agent: "claude-agent-sdk-analyzer-agent", prompt: "Create an agent that analyzes Konva.js integration patterns in the codebase")` | Agent starts |
| 4 | claude-agent | Reads best practices | `Read("ideas&external_references/claude-agent-sdk/subagent-best-practices.md")` | Learns agent design patterns |
| 5 | claude-agent | Examines existing analyzer agents | `Glob(".claude/agents/*-analyzer-agent.md")`<br>`Read(".claude/agents/theia-analyzer-agent.md")`<br>`Read(".claude/agents/infinite-canvas-analyzer-agent.md")` | Extracts proven patterns |
| 6 | claude-agent | Designs agent architecture | None (analysis) | YAML frontmatter + instruction structure planned |
| 7 | claude-agent | Creates agent file | `Write(".claude/agents/konva-analyzer-agent.md", "[complete agent spec following best practices]")` | Agent file created |
| 8 | claude-agent | Returns report (standard format) | None | **Summary**: Created konva-analyzer-agent following framework analyzer patterns.<br>**Specifications**: Tools: Bash, Read, Glob, Grep, WebSearch; Model: inherit<br>**Best Practices Applied**: Evidence-based analysis, contextual tool restriction, standard reporting format<br>**File Created**: `.claude/agents/konva-analyzer-agent.md`<br>**Next Steps**: Session restart may be needed to use new agent |
| 9 | Main Thread | Presents to user | None | "Agent created. Restart session to use konva-analyzer-agent." |

**Total Agents Invoked:** 1 (claude-agent-sdk-analyzer-agent)

**Who Wrote Code:** claude-agent-sdk-analyzer-agent (created agent file)

**Duration:** Single agent invocation

**Key Points:**
- **Main Thread** delegates agent creation to specialized claude-agent
- **claude-agent** reads best practices, examines existing agents, designs and creates new agent file
- **claude-agent's dual purpose**:
  1. Create new subagents (primary)
  2. Provide SDK implementation guidance (secondary)
- **Evidence-based design**: claude-agent always reads `subagent-best-practices.md` before creating agents

---

### Scenario 6: Simple Implementation (Direct, No Orchestration)

**User Request:**
```
"Fix the typo in the README - change 'intsall' to 'install'"
```

**Step-by-Step Flow:**

| Step | Entity | Action | Tools Used | Output |
|------|--------|--------|------------|--------|
| 1 | User | Reports typo | None | Request sent |
| 2 | Main Thread | Analyzes: Simple edit, no domain knowledge needed | None | Direct execution |
| 3 | Main Thread | Reads README | `Read("README.md")` | Finds typo |
| 4 | Main Thread | Fixes typo | `Edit("README.md", old: "intsall", new: "install")` | File corrected |
| 5 | Main Thread | Commits | `Bash("git add README.md && git commit -m 'docs: fix typo'")` | Committed |
| 6 | Main Thread | Reports to user | None | "Fixed and committed" |

**Total Agents Invoked:** 0

**Who Wrote Code:** Main Thread

**Duration:** Immediate (direct execution)

**Key Point:** No agents needed for simple, straightforward tasks.

---

## Comparison Matrix: When to Use What

| Request Type | Route | Agents Used | Who Codes | Example |
|--------------|-------|-------------|-----------|---------|
| **Simple Question** | Direct execution | 0 | Nobody | "List files", "Show git status" |
| **File Edit** | Direct execution | 0 | Main Thread | "Fix typo", "Update version" |
| **Framework Question** | Direct to framework agent | 1 | Nobody | "How does Theia DI work?" |
| **Strategic Decision** | Main Thread orchestrates → PM agent | 1 | Nobody | "Should we add feature X?" |
| **Small Implementation** | Main Thread orchestrates → analyzers → coding-agent | 2-3 | coding-agent | "Add simple Theia widget" |
| **Large Implementation** | Main Thread orchestrates → analyzers + coding-agent(s) | 3-4+ | coding-agent(s) | "Implement multi-file feature" |
| **Agent Creation** | Main Thread → claude-agent | 1 | claude-agent | "Create Konva analyzer agent" |

---

## Decision Flow Diagram

```
User Request
     │
     ▼
┌────────────────────────────────────┐
│ Main Thread: Analyze Request       │
└────────┬───────────────────────────┘
         │
         ├─ Can I answer directly? ────────────────────> ANSWER
         │                                               (0 agents)
         │
         ├─ Single framework question? ───> Framework Agent ──> ANSWER
         │                                   (1 agent)
         │
         ├─ Development task? ────────────> PM-Driven Workflow:
         │                                        │
         │                                        ▼
         │                                  ┌──────────────────┐
         │                                  │ PM Agent         │
         │                                  │ Strategic Guide: │
         │                                  │ • Framework      │
         │                                  │ • Location       │
         │                                  │ • Phasing        │
         │                                  │ • Create PRD     │
         │                                  └────────┬─────────┘
         │                                           │
         │                                           ▼
         │                                  ┌──────────────────────┐
         │                                  │ Main Thread          │
         │                                  │ • Creates plan       │
         │                                  │ • Queries framework  │
         │                                  │   agents             │
         │                                  │ • (Optional) PM      │
         │                                  │   plan review        │
         │                                  │ • Synthesizes        │
         │                                  │ • Spawns coding-agent│
         │                                  └────────┬─────────────┘
         │                                           │
         │                                           ▼
         │                                  coding-agent implements
         │                                           │
         │                                           ▼
         │                                  Main Thread: build, test, commit
         │
         └─ New agent needed? ────────────> Invoke claude-agent-sdk-analyzer-agent
                                             (reads best practices, creates agent file)
                                             (1 agent)

Hierarchy: PM guides → Main Thread executes → coding-agent codes
All agent invocations: Main Thread executes Task() calls
Strategic direction: PM Agent provides
Technical patterns: Framework agents provide
```

---

## Key Execution Principles

### 1. Tool Ownership (Contextual Restriction)

**PRINCIPLE**: Tools are restricted **contextually through prompts**, not mechanically removed. Agents have tools but their role descriptions define HOW to use them.

| Entity | Tool Access | Usage Restriction (Contextual) |
|--------|-------------|--------------------------------|
| **Analyzer Agents** | Bash, Glob, Grep, Read, WebFetch, WebSearch | **Bash for READ-ONLY analysis** (inspect outputs, run tests to understand behavior). **NEVER for implementation** (commits, builds, installations). Role enforced via agent prompt. |
| **coding-agent** | Write, Edit, Read, Glob, Grep | **Code execution only**. NO Bash (no builds/tests/commits). |
| **Main Thread** | ALL tools | **Full access**: Orchestration, implementation, builds, commits, everything. |
| **Task tool** | Main Thread ONLY | Agent invocation exclusive to Main Thread. |

**Why Contextual Works**:
- Agents understand their role from prompts
- More flexible (agents can investigate runtime behavior)
- No artificial tool removal needed
- Trust agent instructions to enforce boundaries

### 2. When to Spawn Subagent (Context Preservation Strategy)

**PRINCIPLE**: Spawn subagent when task requires heavy domain-specific reading that would pollute Main Thread's context.

**Spawn subagent when:**
- ✅ Task requires extensive domain reading (vision docs, framework docs, large codebase sections)
- ✅ Need to synthesize knowledge from reference materials before action
- ✅ Domain analysis needed (Theia patterns, Electron security, EG-DESK vision alignment)
- ✅ Task is "learn this domain, then apply to implementation"
- ✅ Would burden Main Thread context with reference material not needed later

**Main Thread handles directly when:**
- ✅ Simple questions answerable from system knowledge
- ✅ File operations with clear instructions
- ✅ Orchestration tasks (creating mission prompts, invoking agents)
- ✅ Implementation with guidance already received from agents

**Effect of Context Preservation**:
```
WITHOUT subagent:
Main Thread reads 50 files → synthesizes patterns → implements
(Context polluted with all 50 files)

WITH subagent:
Main Thread → Task(agent: "analyzer") → Agent reads 50 files, synthesizes
→ Returns 3-paragraph summary
(Main Thread context: clean, only has summary)
```

**Specific: coding-agent for Large Implementations**
- ✅ Implementing 3+ files
- ✅ Large file edits (500+ lines total)
- ✅ Need to preserve Main Thread for continued orchestration
- ✅ Multiple features planned in single session

### 3. Prompt Creation Flow

```
Main Thread asks: "Help me implement X"
         ↓
agent-swarm-manager creates:
    ├─ "egdesk-pm-agent: Analyze if X aligns with vision..."
    ├─ "theia-analyzer-agent: Examine how Theia implements Y..."
    └─ "electron-analyzer-agent: Research Electron API for Z..."
         ↓
Main Thread executes:
    ├─ Task(agent: "egdesk-pm-agent", prompt: "[exact prompt above]")
    ├─ Task(agent: "theia-analyzer-agent", prompt: "[exact prompt above]")
    └─ Task(agent: "electron-analyzer-agent", prompt: "[exact prompt above]")
```

**Critical:** Main Thread writes the prompts (following orchestration guidelines) and invokes the agents.

### 4. Parallel vs Sequential

**Parallel (Single message, multiple Task calls):**
```typescript
// Main Thread sends ONE message:
Task(agent: "theia-analyzer-agent", prompt: "...")
Task(agent: "electron-analyzer-agent", prompt: "...")
Task(agent: "infinite-canvas-analyzer-agent", prompt: "...")

// All three execute simultaneously
```

**Sequential (Separate messages):**
```typescript
// Message 1:
Task(agent: "electron-analyzer-agent", prompt: "Find security requirements")

// Wait for response, then Message 2:
Task(agent: "theia-analyzer-agent", prompt: "Given the security reqs from previous agent, analyze Theia implementation")
```

**When to use which:**
- **Parallel**: Independent analyses (e.g., "How does each framework handle menus?")
- **Sequential**: Later agent needs earlier agent's findings (e.g., "Given security requirements, analyze implementation")

### 5. Conversational Re-query Pattern (Handling Incomplete Reports)

**PRINCIPLE**: Agents are stateless (single invocation, single response), but Main Thread maintains conversation state and can contextually re-query by including previous context.

**Scenario: Agent returns incomplete report**

```
Main Thread receives report:
✓ Summary: Present
✓ Findings: Present
✗ Files Analyzed: MISSING (REQUIRED!)

Main Thread re-queries contextually:
Task(agent: "theia-analyzer-agent",
     prompt: "You previously provided this analysis:

[ENTIRE PREVIOUS REPORT]

However, the 'Files Analyzed' section is missing, which is REQUIRED.
Please provide the complete list of files you analyzed with file:line references.
No need to re-analyze - just add the missing Files Analyzed section to your previous report.")
```

**Why this works:**
- Agent is stateless, BUT Main Thread provides full context
- Prompt is longer, but keeps conversation flexible
- Agent can extract file list from its previous analysis
- More natural than strict mechanical validation

**When to use:**
- Report missing required sections
- Need clarification on specific findings
- Want more detail on particular aspect
- Need file list broken down differently

**Example flow:**
```
Phase 1: Initial query
→ Agent returns: Good analysis, but Files Analyzed incomplete

Phase 2: Contextual re-query
→ Main Thread includes previous report + specific request
→ Agent completes the missing section

Phase 3: Continue conversation
→ Main Thread uses complete report for next phase
```

**File List Format Enforcement:**

When agents DO include file lists, ensure proper categorization:

```markdown
**File List for Implementation**:

1. **CREATE**:
   - `packages/terminal/src/browser/time-based-theme-switcher.ts` - New service

2. **MODIFY**:
   - `packages/terminal/src/browser/terminal-frontend-module.ts:36` - Add DI binding
   - `packages/terminal/src/browser/terminal-contribution.ts:89` - Inject service

3. **DELETE** (if applicable):
   - `packages/terminal/src/browser/deprecated-service.ts` - Remove old implementation

4. **REFERENCE** (for patterns, not to modify):
   - `packages/workspace/src/browser/workspace-service.ts:89` - Follow @injectable() pattern
```

This structure allows Main Thread to:
- Extract exact action list for coding-agent
- Know which files need creation vs modification
- Identify pattern references separately from implementation targets
- Provide precise instructions: "CREATE 1 file, MODIFY 2 files, following pattern from REFERENCE"

---

## Orchestration Guidelines Deep Dive

### How Main Thread Orchestrates

When handling complex development tasks, Main Thread follows orchestration guidelines from `@.claude/prompts/agent-orchestration.md`:

1. ✅ **Identifies agents** from system prompt knowledge (Task tool description lists all agents)
2. ✅ **Selects** appropriate agents based on capabilities from descriptions
3. ✅ **Analyzes task requirements** from user request
4. ✅ **(Optional) Reads agent details** if needs specific implementation examples from agent files
5. ✅ **Creates** detailed mission prompts for each agent
6. ✅ **Plans** execution phases (parallel/sequential)
7. ✅ **Identifies** decision points (분기점)
8. ✅ **Invokes** agents using Task tool
9. ✅ **Synthesizes** agent results
10. ✅ **Implements** solution (writes code)

### File Reading Scope (Critical)

**Main Thread CAN Read for Orchestration:**
- `.claude/prompts/agent-orchestration.md` - Orchestration guidelines
- `.claude/agents/*.md` - **OPTIONAL**: Only when needs detailed implementation examples from agent files (agent discovery already in system prompt)
- Meta-level orchestration files only

**Main Thread CANNOT Read When Orchestrating:**
- `ideas&external_references/eg-desk ideas/` - That's PM agent's domain (preserves Main Thread context)
- `packages/` - That's framework analyzer agents' domain (they synthesize and return conclusions)
- Any application or framework code (unless already guided by agents)
- Any vision or strategy documents (PM agent reads these)

**Principle**: Main Thread operates at the metaphysical level when orchestrating - it coordinates but delegates actual domain analysis to specialized agents to preserve context.

**Why Context Preservation Matters:**
- PM agent reads ALL vision docs → returns summary → Main Thread's context stays clean
- Framework agents read codebase → return patterns → Main Thread doesn't load 50 files
- Main Thread only gets synthesized conclusions, not raw reference materials

**Exception**: Main Thread CAN read domain files when directly implementing (after receiving agent guidance) or handling simple tasks that don't need orchestration.

### Example Plan Output

```markdown
## Task Analysis
User wants to add time-based terminal theming.

## Execution Plan

### Phase 1 (Parallel)
Agents: egdesk-pm-agent, theia-analyzer-agent

**Mission for egdesk-pm-agent:**
"Analyze EG-DESK_Whitepaper.md to validate: Does automatic
time-based theming align with ambient AI principles?"

**Mission for theia-analyzer-agent:**
"Analyze packages/terminal/src/browser/terminal-theme-service.ts
to understand theme registration and application patterns."

### Phase 2 (Sequential, after Phase 1)
Agent: theia-analyzer-agent

**Mission for theia-analyzer-agent:**
"Using the theme pattern found in Phase 1, analyze how to
create a TimeBasedThemeSwitcher service that integrates
with Theia's DI system."

### Execution Commands

**Phase 1** (Main Thread sends single message with both):
Task(agent: "egdesk-pm-agent", prompt: "[mission above]")
Task(agent: "theia-analyzer-agent", prompt: "[mission above]")

**Phase 2** (Main Thread sends after Phase 1 completes):
Task(agent: "theia-analyzer-agent", prompt: "[mission above with Phase 1 context]")
```

The Main Thread then executes this plan exactly as specified.

---

## Complete Agent Ecosystem

### Vision & Strategy
- **egdesk-pm-agent**: Strategic PM - provides implementation guidance (framework choice, code location, phasing), reviews Main Thread's plans, creates/updates PRDs and vision docs, maintains institutional memory, dynamically discovers project structure

### Framework Analysis
- **theia-analyzer-agent**: Theia codebase analysis
- **electron-analyzer-agent**: Electron documentation analysis
- **infinite-canvas-analyzer-agent**: Infinite Canvas codebase analysis
- **gemini-cli-analyzer-agent**: Gemini CLI codebase analysis

### Development Tools
- **claude-agent-sdk-analyzer-agent**: Subagent Architect - designs and creates new specialized agents by analyzing best practices, also provides Claude Code SDK implementation guidance
- **coding-agent**: Code execution specialist - implements code based on Main Thread's direction (what to fix/create + file list), reads files for details, keeps Main Thread's context clean
- **error-recovery-agent**: Analyzes build/test failures, traces root causes, provides specific recovery strategies with file:line references
- **ux-flow-simulator-agent**: Simulates user interaction flows, predicts runtime errors, identifies race conditions and UX issues before they reach production

### Orchestration
- **Main Thread**: Orchestrates agents following guidelines in `.claude/prompts/agent-orchestration.md`
  - Identifies available agents (from system prompt)
  - Creates mission prompts
  - Plans execution phases
  - Invokes agents
  - Synthesizes results
  - **Preserves context** by delegating heavy domain reading to specialized agents
  - **Can delegate coding to coding-agent** to preserve its context
  - Or implements directly for smaller changes

**All specialized agents:**
- ✅ Analyze codebases or documentation
- ✅ Return evidence-based guidance
- ✅ Provide file references and patterns
- ❌ NEVER write code
- ❌ NEVER execute commands
- ❌ NEVER commit changes

---

## Common Patterns

### Pattern 1: Quick Framework Question
```
User → Main Thread → theia-analyzer-agent → Answer
(Direct invocation - no orchestration needed)
```

### Pattern 2: Strategic Validation
```
User → Main Thread orchestrates → egdesk-pm-agent → Decision
(Main Thread discovers agent, creates prompt, invokes)
(No implementation if rejected)
```

### Pattern 3: Full Development Cycle
```
User → Main Thread orchestrates:
         ├─ Identifies agents (from system prompt)
         ├─ Creates mission prompts
         ├─ Plans phases
         └─ Executes:
               ├─ Phase 1: Parallel validation (PM + Framework agents)
               ├─ Phase 2: Sequential architecture (Framework agent)
               ├─ Synthesizes all guidance
               └─ Phase 3: Main Thread implements (writes code)
```

### Pattern 4: Agent Creation
```
User → Main Thread → claude-agent-sdk-analyzer-agent:
                      ├─ Reads best practices
                      ├─ Examines existing agents
                      ├─ Designs new agent
                      └─ Creates agent file
(1 agent - claude-agent handles complete agent creation)
```

### Pattern 5: Large Implementation (Context Preservation)
```
User → Main Thread orchestrates:
         ├─ Phase 1-N: Queries sub-agents, gathers insights
         │   (PM validation, framework patterns, architecture guidance)
         │
         ├─ Synthesizes insights:
         │   - What needs to be changed (direction)
         │   - Which files need modification (file list)
         │
         └─ Spawns coding-agent(s):
               ├─ Provides: direction + file list
               ├─ coding-agent reads files for implementation details
               ├─ coding-agent writes/edits code (in separate context)
               ├─ Returns status report
               └─ Main Thread builds, tests, commits

Benefit: Main Thread's context stays clean for continued orchestration
Details: coding-agent handles file reading and implementation specifics
```

---

## Anti-Patterns to Avoid

### ❌ Anti-Pattern 1: Over-Orchestration
```
BAD:
User asks: "What files are in packages/core?"
Main Thread → Orchestrates agents → Creates elaborate plan

GOOD:
User asks: "What files are in packages/core?"
Main Thread → Glob("packages/core/**/*") → Answer
(Direct execution - no agents needed)
```

### ❌ Anti-Pattern 2: Sequential When Parallel Possible
```
BAD:
Phase 1: theia-analyzer-agent analyzes menus
Phase 2: electron-analyzer-agent analyzes menus
(These are independent!)

GOOD:
Phase 1 (Parallel):
  - theia-analyzer-agent analyzes menus
  - electron-analyzer-agent analyzes menus
(Both run simultaneously)
```

### ❌ Anti-Pattern 3: Agent Writing Code
```
BAD:
Agent returns: "Here's the code: class Foo { ... }"

GOOD:
Agent returns: "Pattern at packages/core/src/common/foo.ts:45
shows @injectable() decorator with @inject() parameters.
Follow this pattern for your FooService."
```

### ❌ Anti-Pattern 4: Assumption-Based Guidance
```
BAD:
Agent: "Theia probably uses dependency injection like Angular"

GOOD:
Agent: "Analyzing packages/core/src/common/di.ts:
Theia uses InversifyJS for DI. Example at line 89
shows @injectable() decorator usage."
```

### ❌ Anti-Pattern 5: Using coding-agent for Small Edits
```
BAD:
User asks: "Fix typo in README"
Main Thread → coding-agent → Edit README
(Adds unnecessary agent invocation overhead)

GOOD:
User asks: "Fix typo in README"
Main Thread → Edit("README.md", ...) directly
(Immediate execution)

Use coding-agent when:
✅ Large multi-file implementations
✅ Main Thread context filling up
✅ Need to preserve Main Thread for orchestration

Don't use coding-agent when:
❌ Single small edit
❌ Simple typo fixes
❌ Quick updates
```

---

## Success Metrics

An effective agent swarm flow has:

- ✅ **Right Routing**: Simple questions answered directly, complex tasks orchestrated
- ✅ **Parallel Execution**: Independent analyses run simultaneously
- ✅ **Evidence-Based**: All recommendations backed by actual file analysis
- ✅ **Clear Boundaries**: Agents analyze, Main Thread implements
- ✅ **Strategic Alignment**: Vision validated before implementation
- ✅ **Decision Points**: User input requested when needed (분기점)
- ✅ **No Redundancy**: Each agent contributes unique value

---

## Appendix: Roles and Responsibilities Reference

### Comprehensive Capability Matrix

| Entity | CAN Do | CANNOT Do | Tools Available |
|--------|--------|-----------|-----------------|
| **Main Conversation Thread** | • Analyze and route requests<br>• **Orchestrate agents** (following `@.claude/prompts/agent-orchestration.md`):<br>  - Identify agents (from system prompt - Task tool lists all)<br>  - Create mission prompts<br>  - Plan execution phases<br>  - Identify decision points (분기점)<br>  - Design parallel/sequential workflows<br>• Invoke agents (Task tool)<br>• Synthesize agent outputs<br>• Write/Edit/Read files<br>• Execute bash commands<br>• Run git operations<br>• Commit code<br>• Create PRs<br>• Install packages<br>• Run builds and tests<br>• Make implementation decisions<br>• **Preserve own context** by delegating heavy reading to agents | • **When orchestrating**: Read domain files (vision docs, codebase) - delegate to agents to preserve context<br>• Delegate simple tasks unnecessarily<br>• Guess framework patterns without agent consultation<br>• Implement EG-DESK features without vision validation | Write, Edit, Read, Glob, Grep, Bash, Task (all tools) |
| **Specialized Analyzer Agents**<br>(theia, electron, infinite-canvas, etc.) | • Analyze codebase/documentation<br>• Provide evidence-based guidance<br>• Explain framework patterns<br>• Troubleshoot issues<br>• Reference specific files/APIs<br>• Find proven patterns<br>• Return detailed reports<br>• **Use Bash for READ-ONLY analysis** (inspect outputs, run tests to understand behavior) | • Write ANY code<br>• **Use Bash for implementation** (commits, builds, installations) - contextually restricted<br>• Create files<br>• Edit files<br>• Commit changes<br>• Invoke other agents<br>• Implement features | Bash, Read, Glob, Grep, WebFetch, WebSearch (Bash for analysis only, contextually enforced) |
| **egdesk-pm-agent** | • **Strategic PM**: Provide implementation guide (framework, location, phasing)<br>• **Plan Reviewer**: Validate Main Thread's plans, suggest improvements<br>• **Documentation Manager**: Create PRDs, update vision/ideas docs<br>• **Institutional Memory**: Recall previous decisions, record new ones<br>• **Insight Provider**: Explain vision conflicts, suggest alternatives<br>• **Dynamic Discovery**: Glob vision docs + packages to understand project structure<br>• **Framework Selection**: Decide which framework to use based on vision + requirements<br>• **Code Location**: Specify exact package/directory for implementation<br>• **Preserve Main Thread's context**: Read vision docs, synthesize strategic direction | • Write application code (only writes documentation: PRDs, vision docs, ideas files)<br>• Execute implementation commands (Bash for discovery only)<br>• Commit changes<br>• Invoke other agents<br>• Provide technical patterns (framework agents do this) | Bash, Read, Write, Edit, Glob, Grep, WebFetch, WebSearch (Bash for discovery only, Write/Edit for documentation) |
| **coding-agent** | • **Execute code writing/editing** based on Main Thread instructions<br>• Create new files following provided patterns<br>• Edit existing files precisely<br>• Follow framework patterns from analyzer guidance<br>• Handle multi-file implementations<br>• Return implementation status reports | • Make architectural decisions<br>• Choose implementation approaches<br>• Execute bash commands<br>• Run builds/tests<br>• Commit changes<br>• Invoke other agents<br>• Analyze frameworks<br>• Validate vision | Write, Edit, Read, Glob, Grep |
| **claude-agent-sdk-analyzer-agent** | • **Subagent Architect**: Design and create new specialized agents<br>• Read best practices from `subagent-best-practices.md`<br>• Examine existing agents to extract proven patterns<br>• Write agent definition files (`.claude/agents/*.md`)<br>• **SDK Integration Guidance**: Explain Claude Code SDK patterns<br>• Guide SDK feature implementation into forked apps<br>• Troubleshoot SDK usage issues | • Write application code (only writes agent files)<br>• Execute commands<br>• Commit changes<br>• Invoke other agents | Bash, Read, Write, Glob, Grep, WebFetch, WebSearch (Write for agent files only) |
| **User** | • Make final decisions at decision points<br>• Provide preferences<br>• Validate experimental results<br>• Approve/reject architectural plans<br>• Request clarifications<br>• Override any recommendation | (User has full authority) | None (human input) |

### Code Ownership Hierarchy

**Main Thread controls all code writing** - either directly or by delegating to coding-agent.

```
┌──────────────────────────────────────────────────────────┐
│   Main Thread (Orchestrator & Decision Maker)            │
│                                                           │
│   Exclusive capabilities:                                │
│   • Bash tool (execute commands, git, npm, build, test)  │
│   • Task tool (invoke agents)                            │
│   • Commit and PR creation                               │
│                                                           │
│   Orchestrates by following:                             │
│   • @.claude/prompts/agent-orchestration.md              │
│   • Identifies agents (from system prompt)               │
│   • Creates mission prompts                              │
│   • Plans execution phases                               │
│   • Preserves context (delegates heavy reading)          │
│                                                           │
│   Can write code:                                        │
│   ├─ Directly (for small changes)                        │
│   └─ OR delegate to coding-agent (for large impl)        │
│                                                           │
│   Receives guidance from:                                │
│   ├─ Framework analyzers (patterns)                      │
│   └─ egdesk-pm-agent (vision alignment)                 │
└───────────┬───────────────────────┬──────────────────────┘
            │                       │
    (guidance only)          (implementation only)
            │                       │
┌───────────▼─────────┐    ┌────────▼───────────────┐
│ Framework Analyzers │    │ coding-agent           │
│ (Explain patterns)  │    │ (Executes code)        │
│ Returns guidance    │    │                        │
│ with file refs      │    │ Receives:              │
└─────────────────────┘    │ • Detailed impl plan   │
                           │ • Agent guidance       │
┌─────────────────────┐    │ • Patterns to follow   │
│ PM Agent            │    │                        │
│ (Validates vision)  │    │ Executes:              │
│ Returns             │    │ • Write new files      │
│ APPROVE/REJECT      │    │ • Edit existing files  │
└─────────────────────┘    │ • Follow patterns      │
                           │                        │
                           │ Returns:               │
                           │ • Status report        │
                           └────────────────────────┘
```

**Key Points:**
1. **Main Thread**: Can code directly OR delegate to coding-agent
2. **coding-agent**: Only codes when instructed by Main Thread
3. **Analyzer agents**: Never touch code
4. **Only Main Thread**: Has Bash (build, test, commit)

### Orchestration Hierarchy

**Main Thread orchestrates everything:**
- Creates mission prompts
- Invokes agents
- Synthesizes results
- Implements solutions

```
User: "Implement feature X"
         │
         ▼
Main Thread:
         ├─ Analyzes request (complex development task)
         ├─ Follows @.claude/prompts/agent-orchestration.md
         ├─ Identifies agents (from system prompt - Task tool lists all)
         ├─ Selects agents needed
         ├─ Creates detailed mission prompts:
         │  ├─ "egdesk-pm-agent: Validate feature X against vision docs..."
         │  ├─ "theia-analyzer-agent: Analyze packages/Y to find pattern Z..."
         │  └─ "electron-analyzer-agent: Research API for W security..."
         │
         ├─ Plans phases:
         │  - Phase 1 (Parallel): PM + Framework analyzers
         │  - Phase 2 (Sequential): Integration analysis
         │
         ├─ Executes Phase 1 (single message, multiple Tasks):
         │  ├─ Task(agent: "egdesk-pm-agent", prompt: "[exact mission]")
         │  ├─ Task(agent: "theia-analyzer-agent", prompt: "[exact mission]")
         │  └─ Task(agent: "electron-analyzer-agent", prompt: "[exact mission]")
         │
         ├─ Waits for all Phase 1 agents to complete
         │
         ├─ Executes Phase 2 (after Phase 1):
         │  └─ Task(agent: "theia-analyzer-agent",
         │          prompt: "[mission with Phase 1 context]")
         │
         └─ Synthesizes all guidance and IMPLEMENTS:
            ├─ Write(files)
            ├─ Edit(files)
            ├─ Bash(build, test, commit)
            └─ Reports to user
```

**Critical Points:**
1. **Main Thread**: Orchestrates, creates prompts, invokes agents, implements
2. **Agents**: Analyze and return guidance only
3. **User**: Makes decisions at decision points (분기점)

---

## Conclusion

This agent swarm system provides a scalable, efficient framework for developing EG-DESK through clear role separation:

**Main Thread (Communicator & Executor):**
- Routes user requests to appropriate agents
- **Communicates with PM and sub-agents**:
  - Presents user requests to PM for strategic guide
  - Queries framework agents based on PM's guide
  - Reports agent findings back to PM for plan review (if needed)
  - Presents PM's insights/alternatives to user
- **Creates execution plans** based on PM's strategic guidance
- **Synthesizes** PM guide + framework patterns into coding instructions
- **Controls all code writing**:
  - Via coding-agent delegation (direction + file list)
  - Directly for trivial changes only
- **Executes implementation**: Builds, tests, commits (exclusive Bash for implementation)

**egdesk-pm-agent (Strategic PM & Administrative Orchestrator):**
- **Provides strategic guide**: Framework choice, code location, implementation phasing
- **Reviews execution plans**: Validates completeness, suggests improvements
- **Manages documentation**: Creates PRDs, updates vision docs
- **Maintains institutional memory**: Recalls decisions, records new ones
- **Provides insights**: Explains vision conflicts, suggests alternatives to user
- **Dynamically discovers structure**: Globs vision docs + packages to understand project state
- **Never writes application code** (only documentation)

**Specialized Analyzer Agents (Technical Experts):**
- Analyze codebases and documentation in their domains
- Provide technical patterns and file references
- Return evidence-based guidance with file lists (CREATE/MODIFY/DELETE/REFERENCE)
- Never write code or make strategic decisions

**coding-agent (Code Executor):**
- Executes code writing/editing based on Main Thread's direction + file list
- Reads files for implementation details
- Follows patterns from framework analyzers
- Returns status reports
- **Preserves Main Thread's context** for continued communication

**Architectural Principles:**
1. **Strategic Level** (PM Agent): Guides what to build, where, and how - manages vision alignment
2. **Communication Level** (Main Thread): Facilitates between user, PM, and technical agents - executes plans
3. **Technical Level** (Analyzer Agents): Provides framework-specific patterns and guidance
4. **Execution Level** (coding-agent): Implements code following all guidance

**Result:** PM-driven strategic flow with Main Thread as facilitator:
- PM provides strategic direction (framework, location, approach)
- Main Thread creates plans and queries technical agents
- Framework agents provide implementation patterns
- coding-agent executes code
- Main Thread builds, tests, commits
- Clear separation: strategy (PM) → planning (Main Thread) → patterns (analyzers) → execution (coding-agent)
