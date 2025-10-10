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
     ├─ Development task? → Main Thread orchestrates agents (follows @.claude/prompts/agent-orchestration.md)
     └─ New agent? → Main Thread reads SDK docs and creates agent file directly
     │
     ▼
Main Thread implements (ONLY entity that writes code)
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
│  • Discovers agents (Read .claude/agents/)      │
│  • Creates mission prompts (orchestration)      │
│  • Invokes agents                               │
│  • Synthesizes results                          │
│  • Writes code (implementation)                 │
│                                                  │
│ Can read for orchestration:                     │
│  • .claude/agents/ (agent discovery)            │
│  • .claude/prompts/ (orchestration guidelines)  │
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

### Scenario 3: Strategic Decision (Vision-Focused)

**User Request:**
```
"Should we add a floating AI assistant that follows the mouse cursor?"
```

**Step-by-Step Flow:**

| Step | Entity | Action | Tools Used | Output |
|------|--------|--------|------------|--------|
| 1 | User | Proposes feature idea | None | Request sent |
| 2 | Main Thread | Analyzes: EG-DESK feature, needs vision validation | None | Follows orchestration guidelines |
| 3 | Main Thread | Discovers available agents | `Glob(".claude/agents/*.md")`<br>`Read(agent definitions)` | Identifies egdesk-pm-agent as relevant |
| 4 | Main Thread | Creates mission prompt | None (orchestration logic) | Detailed prompt for PM agent |
| 5 | Main Thread | Invokes PM agent | `Task(agent: "egdesk-pm-agent", prompt: "Analyze vision documents to validate: Does floating cursor-following AI assistant align with EG-DESK spatial canvas principles?")` | PM agent starts |
| 6 | egdesk-pm-agent | Discovers vision docs | `Glob("ideas&external_references/eg-desk ideas/*.md")` | Finds relevant docs |
| 7 | egdesk-pm-agent | Analyzes vision docs | `Read("EG-DESK_Whitepaper.md")`<br>`Read("EG-DESK_Spatial_Canvas_UX_Solutions.md")`<br>`Grep("spatial", "proximity")` | Extracts principles |
| 8 | egdesk-pm-agent | Evaluates alignment | None (analysis) | Decision: REJECT |
| 9 | egdesk-pm-agent | Returns decision | None | "REJECT: Conflicts with spatial canvas paradigm. Recommend proximity-based activation instead." |
| 10 | Main Thread | Presents to user | None | User sees rejection with alternative |

**Total Agents Invoked:** 1 (egdesk-pm-agent)

**Who Wrote Code:** Nobody (strategic decision only)

**Duration:** Single agent invocation

**Key Point:**
- **Main Thread** discovered agents and created mission prompt (orchestration)
- **PM Agent** did the actual vision document reading and analysis
- Feature rejected before any technical analysis - vision validation is the first gate

---

### Scenario 4: Cross-Framework Implementation (Full Orchestration)

**User Request:**
```
"Add a custom terminal theme that changes based on time of day"
```

**Step-by-Step Flow:**

| Step | Entity | Action | Tools Used | Output |
|------|--------|--------|------------|--------|
| **PHASE 0: ROUTING & ORCHESTRATION** |
| 1 | User | Requests feature | None | Request sent |
| 2 | Main Thread | Analyzes: Development task, multi-domain | None | Follows orchestration guidelines |
| 3 | Main Thread | Discovers available agents | `Glob(".claude/agents/*.md")`<br>`Read(agent definitions)` | Understands agent capabilities |
| 4 | Main Thread | Identifies needed agents | None (orchestration logic) | List: egdesk-pm-agent, theia-analyzer-agent |
| 5 | Main Thread | Creates mission prompts | None (orchestration) | Detailed prompts for each agent |
| 6 | Main Thread | Plans phases | None (orchestration) | Phase 1: Parallel validation, Phase 2: Sequential architecture |
| **PHASE 1: VALIDATION (Parallel)** |
| 7 | Main Thread | Executes Phase 1 (parallel) | `Task(agent: "egdesk-pm-agent", prompt: "[mission]")`<br>`Task(agent: "theia-analyzer-agent", prompt: "[mission]")` | Both agents start simultaneously |
| 8a | egdesk-pm-agent | Validates vision alignment | `Read("EG-DESK_Whitepaper.md")`<br>`Grep("ambient AI")` | Finds: Aligns with principles |
| 8b | theia-analyzer-agent | Analyzes theme system | `Read("packages/terminal/src/browser/terminal-theme-service.ts")`<br>`Grep("@injectable.*Theme")` | Finds: Theme registration pattern |
| 9a | egdesk-pm-agent | Returns decision | None | "APPROVE: Aligns with ambient AI" |
| 9b | theia-analyzer-agent | Returns technical analysis | None | "Pattern: ThemeService.register() at line 45" |
| **PHASE 2: ARCHITECTURE (Sequential)** |
| 10 | Main Thread | Synthesizes Phase 1 results | None (internal) | Decision: Approved + Pattern identified |
| 11 | Main Thread | Executes Phase 2 | `Task(agent: "theia-analyzer-agent", prompt: "Using ThemeService pattern, analyze how to create TimeBasedSwitcher")` | Agent starts |
| 12 | theia-analyzer-agent | Designs architecture | `Read("packages/workspace/src/browser/workspace-service.ts")`<br>`Grep("@injectable.*Service")` | Finds: DI patterns |
| 13 | theia-analyzer-agent | Returns architecture | None | "Create TimeBasedThemeSwitcher service, register in DI, hook into lifecycle" |
| **PHASE 3: IMPLEMENTATION (Main Thread)** |
| 14 | Main Thread | Creates service file | `Write("packages/terminal/src/browser/time-based-theme-switcher.ts", "[code]")` | New file created |
| 15 | Main Thread | Registers in DI | `Read("terminal-frontend-module.ts")`<br>`Edit(old: "export default", new: "bind(TimeBasedSwitcher)...")` | DI binding added |
| 16 | Main Thread | Integrates with contribution | `Read("terminal-frontend-contribution.ts")`<br>`Edit(old: "export class", new: "@inject(TimeBasedSwitcher)...")` | Integration complete |
| 17 | Main Thread | Builds package | `Bash("npm run build")` | Build succeeds |
| 18 | Main Thread | Stages files | `Bash("git add packages/terminal/...")` | Files staged |
| 19 | Main Thread | Commits | `Bash("git commit -m 'feat(terminal): add time-based theme switcher'")` | Commit created |
| 20 | Main Thread | Reports to user | None | "Feature implemented and committed" |

**Total Agents Invoked:** 3 invocations (PM agent + Theia agent in parallel → Theia agent sequential)

**Who Wrote Code:** Main Thread ONLY (steps 14-19)

**Duration:** Multi-phase (orchestration → parallel validation → sequential architecture → implementation)

**Critical Observations:**
- **Main Thread** orchestrated everything: discovered agents, created mission prompts, planned phases
- **Agents** only analyzed and returned guidance (never touched code)
- **Main Thread** did ALL code writing, building, and committing
- **Parallel execution** in Phase 1 saved time

---

### Scenario 4b: Cross-Framework Implementation (With Coding Agent)

**User Request:**
```
"Add a custom terminal theme that changes based on time of day"
```

**Why use coding-agent:** Large implementation (3+ files), Main Thread context needs to stay clean for continued orchestration

**Step-by-Step Flow:**

| Step | Entity | Action | Tools Used | Output |
|------|--------|--------|------------|--------|
| **PHASE 0-2: SAME AS SCENARIO 4** |
| 1-13 | Various | (Same orchestration and validation phases) | (See Scenario 4) | All agent guidance collected |
| **PHASE 3: IMPLEMENTATION (Delegated to Coding Agent)** |
| 14 | Main Thread | Synthesizes all guidance | None (internal) | Complete implementation instructions |
| 15 | Main Thread | Delegates to coding agent | `Task(agent: "coding-agent", prompt: "[detailed implementation instructions with all agent guidance]")` | Coding agent starts |
| 16 | coding-agent | Reads pattern examples | `Read("packages/workspace/src/browser/workspace-service.ts")` | Understands DI pattern |
| 17 | coding-agent | Creates service file | `Write("packages/terminal/src/browser/time-based-theme-switcher.ts", "[code following pattern]")` | New file created |
| 18 | coding-agent | Reads module file | `Read("terminal-frontend-module.ts")` | Current structure |
| 19 | coding-agent | Registers in DI | `Edit(old: "export default", new: "bind(TimeBasedSwitcher)...")` | DI binding added |
| 20 | coding-agent | Reads contribution file | `Read("terminal-frontend-contribution.ts")` | Current structure |
| 21 | coding-agent | Integrates service | `Edit(old: "export class", new: "@inject(TimeBasedSwitcher)...")` | Integration complete |
| 22 | coding-agent | Returns report | None | "Implementation complete: 1 file created, 2 files modified" |
| **PHASE 4: VERIFICATION (Main Thread)** |
| 23 | Main Thread | Builds package | `Bash("npm run build")` | Build succeeds |
| 24 | Main Thread | Stages files | `Bash("git add packages/terminal/...")` | Files staged |
| 25 | Main Thread | Commits | `Bash("git commit -m 'feat(terminal): add time-based theme switcher'")` | Commit created |
| 26 | Main Thread | Reports to user | None | "Feature implemented and committed" |

**Total Agents Invoked:** 4 invocations (PM agent + Theia agent in parallel → Theia agent sequential → coding-agent for implementation)

**Who Wrote Code:** coding-agent (steps 17-21)

**Duration:** Multi-phase with coding delegation

**Critical Observations:**
- **Main Thread** orchestrated and synthesized but delegated actual coding
- **Main Thread's context** stayed clean - no file edit history
- **coding-agent** executed in separate context with fresh token budget
- **Main Thread** handled build, test, commit (Bash operations)
- **Benefit**: Main Thread can continue orchestrating other features while coding agent works

**When to use this pattern:**
- Large implementations (3+ files)
- Many file edits expected
- Main Thread context filling up
- Need to preserve Main Thread for continued orchestration

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
| 2 | Main Thread | Analyzes: Agent creation (meta-level task) | None | Handles directly without agents |
| 3 | Main Thread | Reads SDK best practices | `Read("ideas&external_references/claude-agent-sdk/subagent-best-practices.md")` | Learns agent design patterns |
| 4 | Main Thread | Examines existing agents | `Glob(".claude/agents/*.md")`<br>`Read(".claude/agents/theia-analyzer-agent.md")` | Finds proven patterns |
| 5 | Main Thread | Designs agent structure | None (planning) | YAML frontmatter + instructions |
| 6 | Main Thread | Creates agent file | `Write(".claude/agents/konva-analyzer-agent.md", "[complete agent spec]")` | Agent file created |
| 7 | Main Thread | Informs user | None | "Agent created. Restart session to use." |

**Total Agents Invoked:** 0

**Who Wrote Code:** Main Thread (designed and created agent file)

**Duration:** Direct execution (reads → plans → writes)

**Key Point:**
- **Main Thread** handles agent creation directly - this is a meta-level orchestration task
- **No agents needed** - Main Thread reads SDK best practices and designs the agent itself
- **claude-agent-sdk-analyzer-agent's actual purpose**: Implementing Claude Code SDK features into forked applications, NOT for creating new agents

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
| **Small Implementation** | Main Thread orchestrates → analyzers | 1-2 | Main Thread | "Add simple Theia widget" |
| **Large Implementation** | Main Thread orchestrates → analyzers + coding-agent | 3-4+ | coding-agent | "Implement multi-file feature" |
| **Agent Creation** | Main Thread directly | 0 | Main Thread | "Create Konva analyzer agent" |

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
         ├─ Complex development task? ────> Main Thread Orchestrates:
         │                                   1. Discover agents (.claude/agents/)
         │                                   2. Create mission prompts
         │                                   3. Plan phases (parallel/sequential)
         │                                   4. Invoke agents (Task tool)
         │                                   5. Synthesize results
         │                                   6. IMPLEMENT (write code)
         │                                        │
         │                                        ├─> Phase 1 (parallel agents)
         │                                        ├─> Phase 2 (sequential agents)
         │                                        └─> [DECISION POINT] → User
         │
         └─ New agent needed? ────────────> Main Thread reads SDK docs
                                             and creates agent file directly
                                             (0 agents)

All code writing: Main Thread ONLY
All agent invocations: Main Thread executes Task() calls
All mission prompts: Created by Main Thread (following orchestration guidelines)
```

---

## Key Execution Principles

### 1. Tool Ownership

| Tool | Only Used By |
|------|--------------|
| `Write` | Main Thread, coding-agent |
| `Edit` | Main Thread, coding-agent |
| `Bash` (with modifications) | Main Thread ONLY |
| `Task` (to invoke agents) | Main Thread ONLY |
| `Read`, `Glob`, `Grep` | Main Thread & All Agents |

### 2. When to Use coding-agent vs Direct Implementation

**Use coding-agent when:**
- ✅ Implementing 3+ files
- ✅ Large file edits (500+ lines total)
- ✅ Main Thread context > 200k tokens used
- ✅ Need to preserve Main Thread for continued orchestration
- ✅ Multiple features being implemented in parallel

**Code directly in Main Thread when:**
- ✅ Single file edit
- ✅ Small changes (< 100 lines)
- ✅ Simple implementations
- ✅ Main Thread context < 100k tokens
- ✅ Last task before commit (no more orchestration needed)

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

---

## Orchestration Guidelines Deep Dive

### How Main Thread Orchestrates

When handling complex development tasks, Main Thread follows orchestration guidelines from `@.claude/prompts/agent-orchestration.md`:

1. ✅ **Discovers agents** by reading `.claude/agents/` directory
2. ✅ **Understands agent capabilities** by reading agent definition files
3. ✅ **Analyzes task requirements** from user request
4. ✅ **Selects** appropriate agents based on capabilities
5. ✅ **Creates** detailed mission prompts for each agent
6. ✅ **Plans** execution phases (parallel/sequential)
7. ✅ **Identifies** decision points (분기점)
8. ✅ **Invokes** agents using Task tool
9. ✅ **Synthesizes** agent results
10. ✅ **Implements** solution (writes code)

### File Reading Scope (Critical)

**Main Thread CAN Read for Orchestration:**
- `.claude/agents/*.md` - Agent definitions (for agent discovery)
- `.claude/prompts/agent-orchestration.md` - Orchestration guidelines
- Meta-level orchestration files only

**Main Thread CANNOT Read When Orchestrating:**
- `ideas&external_references/eg-desk ideas/` - That's PM agent's domain
- `packages/` - That's framework analyzer agents' domain
- Any application or framework code
- Any vision or strategy documents

**Principle**: Main Thread operates at the metaphysical level when orchestrating - it coordinates but delegates actual domain analysis to specialized agents.

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
- **egdesk-pm-agent**: Vision document analysis, strategic alignment validation

### Framework Analysis
- **theia-analyzer-agent**: Theia codebase analysis
- **electron-analyzer-agent**: Electron documentation analysis
- **infinite-canvas-analyzer-agent**: Infinite Canvas codebase analysis
- **gemini-cli-analyzer-agent**: Gemini CLI codebase analysis

### Development Tools
- **claude-agent-sdk-analyzer-agent**: For implementing Claude Code SDK features into forked applications (NOT for creating new agents)
- **coding-agent**: Code execution specialist - implements code based on Main Thread instructions to keep Main Thread's context clean

### Orchestration
- **Main Thread**: Orchestrates agents following guidelines in `.claude/prompts/agent-orchestration.md`
  - Discovers available agents
  - Creates mission prompts
  - Plans execution phases
  - Invokes agents
  - Synthesizes results
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
         ├─ Discovers agents (.claude/agents/)
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
User → Main Thread reads SDK docs → Main Thread designs + creates agent file
(No agents - meta-level task handled directly)
```

### Pattern 5: Large Implementation (Context Preservation)
```
User → Main Thread orchestrates:
         ├─ Discovers agents (.claude/agents/)
         ├─ Gets guidance from PM + Framework analyzers
         ├─ Synthesizes into detailed implementation plan
         └─ Delegates to coding-agent:
               ├─ coding-agent writes all code (in separate context)
               ├─ Returns status report
               └─ Main Thread builds, tests, commits

Benefit: Main Thread's context stays clean for continued orchestration
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
| **Main Conversation Thread** | • Analyze and route requests<br>• **Orchestrate agents** (following `@.claude/prompts/agent-orchestration.md`):<br>  - Discover agents (Read `.claude/agents/`)<br>  - Create mission prompts<br>  - Plan execution phases<br>  - Identify decision points (분기점)<br>  - Design parallel/sequential workflows<br>• Invoke agents (Task tool)<br>• Synthesize agent outputs<br>• Write/Edit/Read files<br>• Execute bash commands<br>• Run git operations<br>• Commit code<br>• Create PRs<br>• Install packages<br>• Run builds and tests<br>• Make implementation decisions | • **When orchestrating**: Read domain files (vision docs, codebase) - delegate to agents<br>• Delegate simple tasks unnecessarily<br>• Guess framework patterns without agent consultation<br>• Implement EG-DESK features without vision validation | Write, Edit, Read, Glob, Grep, Bash, Task (all tools) |
| **Specialized Analyzer Agents**<br>(theia, electron, infinite-canvas, etc.) | • Analyze codebase/documentation<br>• Provide evidence-based guidance<br>• Explain framework patterns<br>• Troubleshoot issues<br>• Reference specific files/APIs<br>• Find proven patterns<br>• Return detailed reports | • Write ANY code<br>• Execute bash commands<br>• Create files<br>• Edit files<br>• Commit changes<br>• Invoke other agents<br>• Implement features | Read, Glob, Grep (analysis only) |
| **egdesk-pm-agent** | • Analyze vision documents<br>• Validate strategic alignment<br>• Provide APPROVE/MODIFY/REJECT decisions<br>• Recommend architecture<br>• Ensure vision consistency<br>• Extract project principles | • Write ANY code<br>• Make implementation decisions without vision analysis<br>• Approve features without document evidence<br>• Execute commands<br>• Commit changes | Read, Glob, Grep (for vision docs) |
| **coding-agent** | • **Execute code writing/editing** based on Main Thread instructions<br>• Create new files following provided patterns<br>• Edit existing files precisely<br>• Follow framework patterns from analyzer guidance<br>• Handle multi-file implementations<br>• Return implementation status reports | • Make architectural decisions<br>• Choose implementation approaches<br>• Execute bash commands<br>• Run builds/tests<br>• Commit changes<br>• Invoke other agents<br>• Analyze frameworks<br>• Validate vision | Write, Edit, Read, Glob, Grep |
| **claude-agent-sdk-analyzer-agent** | • **Analyze how to implement Claude Code SDK into forked apps**<br>• Explain SDK integration patterns<br>• Guide SDK feature implementation<br>• Troubleshoot SDK usage issues | • **Create new agents** (Main Thread does this)<br>• Write ANY code<br>• Execute commands<br>• Commit changes<br>• Invoke other agents | Read, Glob, Grep (for SDK docs) |
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
│   • Discovers agents (.claude/agents/)                   │
│   • Creates mission prompts                              │
│   • Plans execution phases                               │
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
         ├─ Discovers agents (Read .claude/agents/)
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

**Main Thread (Orchestrator & Decision Maker):**
- Routes requests intelligently
- **Orchestrates complex tasks** (following `@.claude/prompts/agent-orchestration.md`):
  - Discovers available agents
  - Creates mission prompts
  - Plans execution phases
  - Invokes agents
  - Synthesizes guidance
- **Controls all code writing**:
  - Directly for small changes
  - Via coding-agent for large implementations (context preservation)
- Builds, tests, commits (exclusive Bash access)

**Specialized Analyzer Agents (Domain Experts):**
- Analyze codebases and documentation in their domains
- Provide evidence-based guidance
- Return patterns and file references
- Never write code
- Never orchestrate other agents

**coding-agent (Code Executor):**
- Executes code writing/editing in separate context
- Follows detailed instructions from Main Thread
- Implements based on analyzer guidance
- Returns status reports
- **Preserves Main Thread's context** for continued orchestration

**Architectural Principles:**
1. **Meta-level** (Main Thread): Orchestrates and decides, delegates domain reading and (optionally) coding
2. **Domain-level** (Analyzer Agents): Read and analyze domain files, return guidance
3. **Execution-level** (coding-agent): Executes code in separate context to preserve Main Thread's token budget

**Result:** Streamlined flow from ideation → strategic validation → framework research → architectural design → implementation, with:
- Main Thread as the single point of orchestration and decision-making
- Context preservation through coding delegation
- Parallel execution for efficiency
- Evidence-based implementation through specialized agents
