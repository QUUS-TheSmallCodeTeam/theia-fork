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
            ├─ Discovers tech stack (Glob technology-stack.md)
            ├─ Vision alignment check
            ├─ Technology stack selection (from discovered stack)
            ├─ Implementation status check (already implemented?)
            ├─ Code location (eg-desk_taehwa/ or packages/)
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
│  • Builds/tests/commits (Bash only)             │
│  • NEVER writes application code                │
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
│ coding-agent: WRITES code (only entity)         │
│                                                  │
│ Each agent reads/writes ONLY its domain         │
└─────────────────────────────────────────────────┘
```

## Workflow Scenarios

### Scenario 1: Simple Question (No Agents)

**User Request:** "What files are in the packages/ai-chat directory?"

**Flow:**
```
User → Main Thread (analyzes) → Glob("packages/ai-chat/**/*") → User
```

**Total Agents:** 0 | **Who Wrote Code:** Nobody | **Duration:** Immediate

---

### Scenario 2: Framework Question (Direct Agent)

**User Request:** "How does Theia's dependency injection work?"

**Flow:**
```
User → Main Thread (analyzes: Theia question)
     ↓
Task(agent: "theia-analyzer-agent", prompt: "Analyze Theia's DI system...")
     ↓
theia-analyzer-agent: Read(di.ts) + Grep("@injectable") + Glob(*-module.ts)
     ↓ (returns detailed explanation with file refs)
Main Thread → User
```

**Total Agents:** 1 (theia-analyzer-agent) | **Who Wrote Code:** Nobody | **Duration:** Single agent

**Key Point:** Main Thread directly invoked the framework agent without going through swarm manager (efficiency optimization for single-domain questions).

---

### Scenario 3: Vision Conflict with Decision Branches

**User Request:** "Should we add a floating AI assistant that follows the mouse cursor?"

**Why this scenario:** User proposes feature that conflicts with vision - PM provides insight + alternative. Shows user decision flow.

**Flow:**
```
PHASE 0: PM ANALYSIS
User → MT → PM (discovers vision docs, finds AGAINST floating UI, reason: breaks spatial affordances)
         ↓
PM Returns: REJECT + Alternative (proximity-based AI, vision-aligned)

★ USER DECISION 1: A) Accept alternative | B) Insist on original | C) Cancel
         │
   ┌─────┼─────┐
   A     B     C → [STOP]
   │     │
   │     └──→ PM Re-evaluates → ★ USER DECISION 2: Maintain vision | Evolve vision?
   │                                                │                  │
   │                                          [STOP - maintain]        ▼
   │                                                         PM: Update vision docs
   │                                                              + Document evolution
   └─────────────────────────────────────────────────────────────────┴──→ Implementation
```

**Decision Gates:**
1. **User Decision 1**: Accept alternative? Insist? Cancel?
2. **User Decision 2** (if insisted): Maintain vision or evolve vision?

**Total Agents:** 2-3 (PM rejection → optional PM re-evaluation → optional PM vision update)

**Critical Observations:**
- **PM provides insight, not just rejection**: Explains why conflict exists with evidence
- **Constructive alternative**: Vision-aligned approach that solves same user need
- **User can challenge vision**: PM reassesses, but user has final authority
- **Vision can evolve**: PM documents evolution with user's rationale
- **Institutional memory preserved**: Updates vision docs with reasoning

---

### Scenario 4: Development with PM Strategic Guide

**User Request:**
```
"Add a custom terminal theme that changes based on time of day"
```

**Variations covered:**
- **Base flow**: PM guide → Framework investigation → Implementation
- **Optional delegation**: Main Thread delegates to coding-agent if implementation complex (3+ files)
- **Optional conflict check**: PM checks via Grep if feature already exists in eg-desk_taehwa/ (EG-DESK custom code only, not Theia framework)
- **Optional UX validation**: ux-flow-simulator-agent validates complex flows

**Step-by-Step Flow:**

```
          ┌─────────────────────────────┐
          │ PHASE 0: PM STRATEGIC GUIDE │
          └──────────┬──────────────────┘
                     ↓
┌────────────────────────────────────────────────────────────┐
│ Steps 1-7: User → Main Thread → PM Agent                  │
│ • PM: Dynamic discovery (Glob vision docs + tech stack)    │
│ • PM: Analyzes vision + existing structure                 │
│ • PM: Creates PRD                                          │
│ • PM Returns: APPROVE                                      │
│   - Framework: Theia (terminal theming is Theia domain)    │
│   - Location: packages/terminal/src/browser/               │
│   - Approach: Phase 1-3 (analyze → design → implement)     │
│   - Considerations: Manual override, preference persistence│
└────────────────────┬───────────────────────────────────────┘
                     │
          ┌──────────┴────────────────────────────────────┐
          │ PHASE 1: FRAMEWORK INVESTIGATION PLANNING     │
          └──────────┬────────────────────────────────────┘
                     ↓
┌────────────────────────────────────────────────────────────┐
│ Step 8: Main Thread                                        │
│ • Plans investigation based on PM's guide                  │
│ • Output: Query theia-analyzer for theme patterns, DI      │
└────────────────────┬───────────────────────────────────────┘
                     │
          ┌──────────┴────────────────────────────────────┐
          │ PHASE 2: FRAMEWORK INVESTIGATION (EXECUTION)  │
          └──────────┬────────────────────────────────────┘
                     ↓
┌────────────────────────────────────────────────────────────┐
│ Steps 9-11: Main Thread → theia-analyzer-agent             │
│ • Analyzer reads terminal-theme-service.ts                 │
│ • Analyzer reads terminal-frontend-module.ts               │
│ • Analyzer returns: Pattern found + File List              │
│   - CREATE time-based-theme-switcher.ts                    │
│   - MODIFY terminal-frontend-module.ts:36                  │
│   - REFERENCE workspace-service.ts:89 (@injectable pattern)│
└────────────────────┬───────────────────────────────────────┘
                     │
          ┌──────────┴────────────────────────┐
          │ PHASE 3: IMPLEMENTATION PLANNING  │
          └──────────┬────────────────────────┘
                     ↓
┌────────────────────────────────────────────────────────────┐
│ Step 12: Main Thread                                       │
│ • Synthesizes PM guide + framework patterns                │
│ • Creates implementation plan:                             │
│   - Direction: Create TimeBasedThemeSwitcher (Theia DI)    │
│   - File List: CREATE/MODIFY/REFERENCE with line numbers   │
└────────────────────┬───────────────────────────────────────┘
                     │
          ┌──────────┴──────────────────────────────────┐
          │ PHASE 4: IMPLEMENTATION                     │
          │ If complex: Delegate to coding-agent        │
          │ If simple: Main Thread implements directly  │
          └──────────┬──────────────────────────────────┘
                     ↓
┌────────────────────────────────────────────────────────────┐
│ Steps 13-15: Coding Agent Delegation (if complex)          │
│                                                            │
│ ★ EXECUTION OPTIONS (Main Thread decides):                │
│                                                            │
│ ○ Option A - Sequential (Single Agent):                   │
│   Task(agent: "coding-agent",                             │
│        prompt: "Direction + File List")                   │
│                                                            │
│ ○ Option B - Simple Parallel (Independent Files):        │
│   Task(agent: "coding-agent", prompt: "[Agent 1: X,Y]")  │
│   Task(agent: "coding-agent", prompt: "[Agent 2: A,B]")  │
│   Use when: Agents modify different files                 │
│                                                            │
│ ○ Option C - Worktree Parallel (Shared Files):           │
│   # Create worktrees                                      │
│   Bash: git worktree add ../theia-fork-t1 -b task1       │
│   Bash: git worktree add ../theia-fork-t2 -b task2       │
│   # Spawn agents in different worktrees                   │
│   Task(agent: "coding-agent",                             │
│        prompt: "Working dir: C:/Projects/theia-fork-t1    │
│                 [Agent 1 instructions]")                  │
│   Task(agent: "coding-agent",                             │
│        prompt: "Working dir: C:/Projects/theia-fork-t2    │
│                 [Agent 2 instructions]")                  │
│   # After completion                                      │
│   Bash: git checkout master && git merge task1 task2     │
│   Bash: git worktree remove ../theia-fork-t1 ../t2       │
│   Use when: Multiple agents need same files (package.json)│
│                                                            │
│ • Decision tree: Single task → A                          │
│                  Multiple tasks, different files → B      │
│                  Multiple tasks, same files → C           │
│                                                            │
│ • All options: coding-agent checks CODEBASE_STRUCTURE.md  │
│   (EG-DESK custom code only) for conflicts first          │
│ • If conflict: STOP, report to Main Thread → User decides │
│ • If no conflict: Implement + Update structure doc        │
└────────────────────┬───────────────────────────────────────┘
                     │
          ┌──────────┴──────────────────────────┐
          │ PHASE 4.5: UX FLOW VALIDATION       │
          │            (Optional)               │
          └──────────┬──────────────────────────┘
                     ↓
┌────────────────────────────────────────────────────────────┐
│ Steps 16a-d: UX Flow Simulator (optional)                  │
│ • Main Thread decides: Complex flow? Worth validating?     │
│ • If yes: Invoke ux-flow-simulator-agent                   │
│ • Simulator traces execution paths                         │
│ • If issues found → coding-agent fixes → Re-validate       │
│ • If no issues → Proceed to build                          │
│                                                            │
│ Skip validation for:                                       │
│ ❌ Simple CRUD operations                                  │
│ ❌ Pure UI styling changes                                 │
│ ❌ Documentation updates                                   │
│                                                            │
│ Use validation for:                                        │
│ ✅ Complex user interaction flows                          │
│ ✅ State management with race conditions                   │
│ ✅ Async operations or event handling                      │
│ ✅ Critical features (security, data integrity)            │
└────────────────────┬───────────────────────────────────────┘
                     │
          ┌──────────┴──────────────┐
          │ PHASE 5: BUILD & COMMIT │
          └──────────┬──────────────┘
                     ↓
┌────────────────────────────────────────────────────────────┐
│ Step 17: Main Thread                                       │
│ • Builds, tests, commits                                   │
│ • Tools: Bash("npm run build && git add . && git commit")  │
└────────────────────────────────────────────────────────────┘
```

**Total Agents Invoked:** 2-4 (PM → framework analyzer → optional coding-agent → optional ux-flow-simulator)

**Who Wrote Code:** coding-agent (if delegated) or Main Thread (if simple)

**Critical Observations:**
- **PM provides complete strategic direction**: Framework, location, approach, considerations
- **PM creates PRD**: Documents approved feature
- **Main Thread plans investigation**: Identifies framework patterns to research
- **Framework agent investigates**: Provides technical patterns (not strategic direction)
- **Main Thread creates implementation plan**: Synthesizes PM guide + framework patterns
- **Conflict detection** (EG-DESK custom code only): coding-agent checks CODEBASE_STRUCTURE.md BEFORE implementing, STOPS if conflict found, reports to user for decision
- **Structure document updates** (EG-DESK custom code only): coding-agent updates after successful implementation
- **coding-agent executes** (optional): Follows synthesized plan, keeps Main Thread context clean
- **UX flow validation** (optional): Main Thread's discretion for complex flows
- **Main Thread builds/tests/commits**: Retains control of deployment

**When to use coding-agent delegation:**
- Large implementations (3+ files)
- Main Thread needs to stay available for orchestration
- Context preservation is critical

**When conflict checking applies:**
- **YES**: EG-DESK custom features (eg-desk_taehwa/*) - check CODEBASE_STRUCTURE.md
- **NO**: Theia framework modifications (packages/*) - no conflict checking needed

---

### Scenario 5: Agent Creation Request

**User Request:** "We need an agent that analyzes Konva.js integration patterns"

**Flow:**
```
User → MT (analyzes: agent creation)
     ↓
Task(agent: "claude-agent-sdk-analyzer-agent", prompt: "Create Konva analyzer agent")
     ↓
claude-agent: Read(subagent-best-practices.md) + Glob(existing agents) + Design + Create(.claude/agents/konva-analyzer-agent.md)
     ↓
MT → User ("Agent created. Restart session to use konva-analyzer.")
```

**Total Agents:** 1 (claude-agent-sdk-analyzer-agent) | **Who Wrote Code:** claude-agent (created agent file) | **Duration:** Single agent

**Key Points:**
- **Main Thread** delegates agent creation to specialized claude-agent
- **claude-agent** reads best practices, examines existing agents, designs and creates new agent file
- **Evidence-based design**: claude-agent always reads `subagent-best-practices.md` before creating agents

---

### Scenario 6: Simple File Edit (coding-agent)

**User Request:** "Fix the typo in the README - change 'intsall' to 'install'"

**Flow:**
```
User → MT (analyzes: simple edit) → coding-agent (Read README, Edit typo) → MT (commits) → User
```

**Total Agents:** 1 (coding-agent) | **Who Wrote Code:** coding-agent | **Duration:** Single agent + commit

**Key Point:** ALL file changes go through coding-agent for consistency. Main Thread NEVER edits application files directly.

---

### Scenario 7: New Technology Research & Evaluation

**User Request:** "캔버스에 3D visualization을 추가하고 싶어. 어떤 framework가 좋을까?"

**Why this scenario:** User needs capability not in current stack - requires technology research. Shows complete flow with PM diagnosis, research execution, and user decision points.

**Critical Flow (CORRECTED):**
1. PM diagnoses limitation (NOT full research plan)
2. Main Thread plans research (decides frameworks to investigate)
3. NO user approval for starting research (auto-proceed)
4. Research agents write to subagent_reports/ (temporary, NOT ideas/)
5. PM reviews subagent_reports/, may use WebSearch
6. User approval for technology adoption (decision gate)
7. PM moves confirmed findings: subagent_reports/ → ideas&external_references/
8. Delete subagent_reports/ after decision

**Flow:**
```
PHASE 0: LIMITATION DIAGNOSIS
User → MT → PM (discovers tech stack, diagnoses: Konva.js = 2D-only, cannot render 3D)
         ↓
PM Returns: RESEARCH_NEEDED + Gap + Criteria (bundle, integration, perf) + Options (Three.js, Babylon.js, WebGL)

PHASE 1: MT PLANS RESEARCH (NO USER APPROVAL - AUTO-PROCEED)
MT: Chooses 3 parallel investigations

PHASE 2: PARALLEL RESEARCH EXECUTION → subagent_reports/ (temporary)
MT spawns 3 Tasks simultaneously:
  Task(agent: "general-purpose", "Research Three.js...") → subagent_reports/threejs.md
  Task(agent: "general-purpose", "Research Babylon.js...") → subagent_reports/babylonjs.md
  Task(agent: "general-purpose", "Research WebGL...") → subagent_reports/webgl.md

PHASE 3: TECHNICAL ANALYSIS
MT → infinite-canvas-analyzer-agent (reads subagent_reports/*.md)
  ↓ Returns: Three.js (medium), Babylon.js (high complexity)

PHASE 4: PM EVALUATION
MT → PM (reads subagent_reports/, may WebSearch)
  ↓ Evaluates against criteria + vision alignment
  ↓ Returns: Recommended Three.js (4.2/5) | Rationale + Scoring + Tradeoffs

★ USER DECISION: A) Approve Three.js | B) Choose different | C) Modify criteria (re-research)
         │
   ┌─────┼──────┐
   A     B      C → Loop to Phase 2
   │     │
   └─────┴──→ PM: Adopt tech → Update tech-stack.md → Move [choice].md → ideas/
               → Delete other reports → Create PRD → Implementation
```

**Decision Gate:** User approves technology adoption? Choose different? Modify criteria?

**Total Agents:** 5+ (PM diagnosis → 3 parallel research → analyzer → PM evaluation → PM adoption)

**Critical Flow Corrections:**
- ✅ PM diagnoses gap (NOT full research plan)
- ✅ Main Thread creates research plan (decides frameworks)
- ✅ NO user approval for "proceed with research?" (auto-proceed)
- ✅ Research findings go to **subagent_reports/** first (NOT ideas/)
- ✅ PM reviews **subagent_reports/**, may use WebSearch
- ✅ User approves **technology adoption decision** (not research start)
- ✅ PM moves confirmed findings: subagent_reports/ → ideas&external_references/
- ✅ Delete subagent_reports/ after decision

**Why subagent_reports/ first?**
- Temporary workspace for research notes
- PM can validate findings before institutional memory
- User can reject entire research without polluting ideas/
- Clean separation: tentative vs confirmed knowledge

---

## Quick Reference

### Decision Matrix

| Request Type | Route To | When | Who Writes Code | Duration |
|--------------|----------|------|-----------------|----------|
| **Simple Question** | Direct execution | File listing, basic git, grep | Nobody | Immediate |
| **Framework Question** | Direct to framework agent | Theia/Electron/Canvas specific questions | Nobody | Single agent |
| **Vision Question** | egdesk-pm-agent | Feature alignment, strategic decisions | Nobody | Single PM turn |
| **Vision Conflict** | egdesk-pm-agent | User proposes conflicting feature | coding-agent (if user evolves vision) | 2-3 PM turns |
| **Development Task** | PM-driven workflow | Implementing EG-DESK features | coding-agent | Multi-phase |
| **Technology Research** | PM diagnosis → Research → PM evaluation | Need capability not in current stack | coding-agent (after adoption) | 5+ agents |
| **Agent Creation** | claude-agent-sdk-analyzer-agent | Need new specialized agent | claude-agent | Single agent |
| **Simple Edit** | coding-agent | Typo fixes, small changes | coding-agent | Single agent |

### Common Patterns (Brief)

**Pattern 1: Quick Framework Question** → See Scenario 2
- Main Thread → framework analyzer → User

**Pattern 2: Strategic Validation** → See Scenario 3
- Main Thread → PM → User decision → Optional PM evolution

**Pattern 3: Full Development Cycle** → See Scenario 4
- Main Thread → PM guide → Framework investigation → Implementation → Build/commit

**Pattern 4: Agent Creation** → See Scenario 5
- Main Thread → claude-agent-sdk-analyzer → User (restart session)

**Pattern 5: Large Implementation** → See Scenario 4 (coding-agent delegation)
- Same as Pattern 3, but Main Thread delegates to coding-agent (3+ files)

**Pattern 6: Conflict Detection** → See Scenario 4 (EG-DESK custom code)
- coding-agent checks CODEBASE_STRUCTURE.md BEFORE implementing
- If conflict: STOP → User decides → Retry with resolution

**Pattern 7: UX Flow Validation** → See Scenario 4 (optional)
- coding-agent implements → Main Thread validates via ux-flow-simulator
- If issues: coding-agent fixes → Re-validate → Build

**Pattern 8: Technology Research** → See Scenario 7
- PM diagnoses gap → MT plans research → Research to subagent_reports/ → PM reviews → User approves → PM adopts (moves to ideas/)

**Pattern 9: Worktree-based Parallel Implementation** → See Scenario 4 (Option C)
- Main Thread creates git worktrees (separate directories)
- Spawns multiple coding-agents in different worktrees
- Each agent works in isolated filesystem (no collision possible)
- Main Thread merges branches and cleans up worktrees
- Use when: Multiple agents need to modify same files (package.json, tsconfig.json, shared configs)

### Anti-Patterns to Avoid

❌ **Main Thread Reads Vision Docs When Orchestrating**
- **Why wrong**: Pollutes Main Thread's context with large documents
- **What instead**: PM agent reads vision docs, returns summary

❌ **Main Thread Reads Codebase When Orchestrating**
- **Why wrong**: Context overload, metaphysical separation violation
- **What instead**: Framework agents read codebase, return patterns

❌ **Main Thread Edits Application Files Directly**
- **Why wrong**: Inconsistent with coding-agent pattern, context pollution
- **What instead**: Always delegate to coding-agent (even simple edits)

❌ **Skipping PM for Development Tasks**
- **Why wrong**: No strategic alignment, no PRD, no vision check
- **What instead**: Always consult PM first for development work

❌ **User Approval Gate for "Proceed with Research?"**
- **Why wrong**: Unnecessary friction, research is exploration not commitment
- **What instead**: Auto-proceed with research, user approves adoption decision

❌ **Research Findings Directly to ideas&external_references/**
- **Why wrong**: Pollutes institutional memory with unvalidated research
- **What instead**: subagent_reports/ → PM review → User approval → PM moves to ideas/

❌ **PM Creates Full Research Plan**
- **Why wrong**: PM should diagnose gap, not design research execution
- **What instead**: PM diagnoses limitation + suggests options, Main Thread creates research plan

❌ **Auto-Resolving Conflicts**
- **Why wrong**: coding-agent cannot make strategic decisions about naming/keybindings
- **What instead**: STOP immediately, report to Main Thread → User decides

❌ **Forgetting to Update CODEBASE_STRUCTURE.md**
- **Why wrong**: Future conflicts undetectable, registry becomes stale
- **What instead**: coding-agent updates structure doc AFTER successful implementation

❌ **Hardcoding EG-DESK Paths**
- **Why wrong**: Project structure may change
- **What instead**: coding-agent uses Glob to discover eg-desk*/ and CODEBASE_STRUCTURE.md dynamically

## Success Metrics

**For Main Thread:**
- ✅ Routes requests to appropriate handlers (direct vs agent vs PM-driven)
- ✅ Delegates domain reading to specialized agents
- ✅ Synthesizes agent results clearly
- ✅ Builds, tests, commits successfully
- ✅ Context remains clean (doesn't read vision/codebase when orchestrating)

**For PM Agent:**
- ✅ Discovers vision docs and tech stack dynamically (Glob)
- ✅ Provides complete strategic direction (framework, location, phasing)
- ✅ Creates/updates PRDs
- ✅ Detects vision conflicts with evidence
- ✅ Suggests vision-aligned alternatives
- ✅ Documents vision evolution with rationale

**For Framework Agents:**
- ✅ Returns evidence-based patterns (file:line references)
- ✅ Provides file lists with actions (CREATE/MODIFY/DELETE/REFERENCE)
- ✅ Stays within domain expertise

**For coding-agent:**
- ✅ Discovers EG-DESK codebase dynamically (Glob eg-desk*/)
- ✅ Checks CODEBASE_STRUCTURE.md for conflicts (EG-DESK custom code only)
- ✅ STOPS immediately if conflict detected (reports to Main Thread)
- ✅ Updates structure document after successful implementation
- ✅ Implements exactly what Main Thread instructed
- ✅ Follows framework patterns precisely
- ✅ Keeps Main Thread's context clean

**For User:**
- ✅ Understands why features approved/rejected (vision alignment)
- ✅ Can challenge vision with rationale (PM reassesses)
- ✅ Makes informed decisions (PM provides scoring, tradeoffs)
- ✅ Sees institutional memory preserved (PRDs, updated vision docs)

## Appendix: Roles and Responsibilities Reference

| Agent | Domain | Can Read | Can Write | Output Format | Model |
|-------|--------|----------|-----------|---------------|-------|
| **Main Thread** | Orchestration | .claude/prompts/, .claude/agents/ (optional) | None (application code) | Synthesized plans, user communication | Default |
| **egdesk-pm-agent** | Vision & Strategy | ideas&external_references/eg-desk ideas/, technology-stack.md | PRDs, vision docs, tech stack updates | Strategic guide, PRD, evaluation report | Inherit |
| **theia-analyzer-agent** | Theia Framework | packages/*, Theia docs | None | Pattern analysis, file lists (CREATE/MODIFY/DELETE/REFERENCE) | Inherit |
| **electron-analyzer-agent** | Electron Framework | Electron docs (WebFetch) | None | Pattern analysis, API guidance | Inherit |
| **infinite-canvas-analyzer-agent** | Infinite Canvas | ideas&external_references/infinite-canvas/ | None | Integration analysis | Inherit |
| **coding-agent** | Code Execution | Target files (any worktree) | Application code (CREATE/MODIFY), CODEBASE_STRUCTURE.md | Implementation report, conflict reports | Inherit |
| **claude-agent-sdk-analyzer-agent** | Agent Architecture | .claude/agents/, subagent-best-practices.md | .claude/agents/*.md | Agent spec, best practices report | Inherit |
| **ux-flow-simulator-agent** | UX Validation | Implemented files | None | Flow validation report, issue list | Inherit |
| **error-recovery-agent** | Error Diagnosis | Build logs, error traces, relevant code | None | Root cause analysis, recovery strategy | Inherit |
| **general-purpose** | Research | Web (WebSearch, WebFetch) | subagent_reports/*.md (temporary) | Research findings | Inherit |

**Key Ownership:**
- **Vision**: egdesk-pm-agent owns vision docs, decides alignment
- **Code**: coding-agent is ONLY entity that writes application code
- **Build/Test/Commit**: Main Thread retains control (Bash tool)
- **Worktree Management**: Main Thread creates/merges/removes worktrees (git worktree commands), coding-agents work within assigned worktrees
- **Structure Registry**: coding-agent maintains CODEBASE_STRUCTURE.md (EG-DESK custom code only)
- **Research Notes**: general-purpose writes to subagent_reports/, PM moves to ideas/ after approval
- **PRDs**: egdesk-pm-agent creates/updates in ideas&external_references/eg-desk ideas/features/

## Conclusion

This agent swarm system balances **strategic alignment** (PM guides vision) with **efficient execution** (Main Thread orchestrates, specialized agents analyze, coding-agent implements). The **metaphysical separation** principle ensures Main Thread's context remains clean for orchestration by delegating domain reading to specialized agents.

**Key success factors:**
1. **PM provides strategic direction** - framework, location, phasing, vision alignment
2. **Main Thread orchestrates but doesn't read domains** - preserves context for coordination
3. **Specialized agents analyze their domains** - return synthesized findings, not raw data
4. **coding-agent implements exclusively** - all code changes go through one path for consistency
5. **Conflict detection prevents duplicates** - CODEBASE_STRUCTURE.md registry for EG-DESK custom code
6. **User retains authority** - can challenge vision, approve technology, resolve conflicts
7. **Research flow validated** - subagent_reports/ → PM review → user approval → ideas/ (institutional memory)

The result: **Fast, vision-aligned development with institutional memory preservation.**
