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

**User Request:**
```
"What files are in the packages/ai-chat directory?"
```

**Step-by-Step Flow:**

```
┌──────────────────────────────────────────────┐
│ Step 1: User                                 │
│ • Action: Asks question                      │
│ • Tools: None                                │
│ • Output: Request sent                       │
└──────────────────┬───────────────────────────┘
                   │
                   ↓
┌──────────────────────────────────────────────┐
│ Step 2: Main Thread                          │
│ • Action: Analyzes - Simple file listing     │
│ • Tools: None                                │
│ • Output: Routes to direct execution         │
└──────────────────┬───────────────────────────┘
                   │
                   ↓
┌──────────────────────────────────────────────┐
│ Step 3: Main Thread                          │
│ • Action: Lists files                        │
│ • Tools: Glob("packages/ai-chat/**/*")       │
│ • Output: File list                          │
└──────────────────┬───────────────────────────┘
                   │
                   ↓
┌──────────────────────────────────────────────┐
│ Step 4: Main Thread                          │
│ • Action: Returns answer to user             │
│ • Tools: None                                │
│ • Output: User sees file list                │
└──────────────────────────────────────────────┘
```

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

```
┌──────────────────────────────────────────────────────────────┐
│ Step 1: User                                                 │
│ • Action: Asks framework question                            │
│ • Tools: None                                                │
│ • Output: Request sent                                       │
└────────────────────────┬─────────────────────────────────────┘
                         │
                         ↓
┌──────────────────────────────────────────────────────────────┐
│ Step 2: Main Thread                                          │
│ • Action: Analyzes - Theia framework question                │
│ • Tools: None                                                │
│ • Output: Routes to theia-analyzer-agent                     │
└────────────────────────┬─────────────────────────────────────┘
                         │
                         ↓
┌──────────────────────────────────────────────────────────────┐
│ Step 3: Main Thread                                          │
│ • Action: Invokes agent                                      │
│ • Tools: Task(agent: "theia-analyzer-agent",                 │
│          prompt: "Analyze Theia's DI system...")             │
│ • Output: Agent starts                                       │
└────────────────────────┬─────────────────────────────────────┘
                         │
                         ↓
┌──────────────────────────────────────────────────────────────┐
│ Step 4: theia-analyzer-agent                                 │
│ • Action: Reads DI implementation                            │
│ • Tools: Read("packages/core/src/common/di.ts")              │
│          Grep("@injectable")                                 │
│ • Output: Finds patterns                                     │
└────────────────────────┬─────────────────────────────────────┘
                         │
                         ↓
┌──────────────────────────────────────────────────────────────┐
│ Step 5: theia-analyzer-agent                                 │
│ • Action: Analyzes examples                                  │
│ • Tools: Glob("**/*-frontend-module.ts")                     │
│          Read(example files)                                 │
│ • Output: Understands usage                                  │
└────────────────────────┬─────────────────────────────────────┘
                         │
                         ↓
┌──────────────────────────────────────────────────────────────┐
│ Step 6: theia-analyzer-agent                                 │
│ • Action: Returns report                                     │
│ • Tools: None                                                │
│ • Output: Detailed explanation with file refs                │
└────────────────────────┬─────────────────────────────────────┘
                         │
                         ↓
┌──────────────────────────────────────────────────────────────┐
│ Step 7: Main Thread                                          │
│ • Action: Presents to user                                   │
│ • Tools: None                                                │
│ • Output: User sees explanation                              │
└──────────────────────────────────────────────────────────────┘
```

**Total Agents Invoked:** 1 (theia-analyzer-agent)

**Who Wrote Code:** Nobody (analysis only)

**Duration:** Single agent invocation

**Key Point:** Main Thread directly invoked the framework agent without going through swarm manager (efficiency optimization for single-domain questions).

---

### Scenario 3a: Strategic Decision with Vision Conflict (User Accepts Alternative)

**User Request:**
```
"Should we add a floating AI assistant that follows the mouse cursor?"
```

**Why this scenario:** User proposes feature that conflicts with vision - PM provides insight + alternative, user accepts alternative

**Step-by-Step Flow:**

```
┌────────────────────────────────────────────────────────────────────┐
│ Step 1: User                                                       │
│ • Action: Proposes feature idea                                    │
│ • Tools: None                                                      │
│ • Output: Request sent                                             │
└────────────────────┬───────────────────────────────────────────────┘
                     │
                     ↓
┌────────────────────────────────────────────────────────────────────┐
│ Step 2: Main Thread                                                │
│ • Action: Analyzes - EG-DESK feature, needs PM strategic guide     │
│ • Tools: None                                                      │
│ • Output: Follows orchestration guidelines                         │
└────────────────────┬───────────────────────────────────────────────┘
                     │
                     ↓
┌────────────────────────────────────────────────────────────────────┐
│ Step 3: Main Thread                                                │
│ • Action: Invokes PM agent                                         │
│ • Tools: Task(agent: "egdesk-pm-agent",                            │
│          prompt: "User wants floating AI assistant...")            │
│ • Output: PM agent starts                                          │
└────────────────────┬───────────────────────────────────────────────┘
                     │
                     ↓
┌────────────────────────────────────────────────────────────────────┐
│ Step 4: egdesk-pm-agent                                            │
│ • Action: Discovers vision docs                                    │
│ • Tools: Glob("ideas&external_references/eg-desk ideas/**/*.md")   │
│ • Output: Finds UX and whitepaper docs                             │
└────────────────────┬───────────────────────────────────────────────┘
                     │
                     ↓
┌────────────────────────────────────────────────────────────────────┐
│ Step 5: egdesk-pm-agent                                            │
│ • Action: Analyzes vision docs                                     │
│ • Tools: Read("EG-DESK_Whitepaper.md")                             │
│          Read("EG-DESK_Spatial_Canvas_UX_Solutions.md")            │
│          Grep("spatial", "proximity", "floating")                  │
│ • Output: Finds - Previous decision against floating UI            │
└────────────────────┬───────────────────────────────────────────────┘
                     │
                     ↓
┌────────────────────────────────────────────────────────────────────┐
│ Step 6: egdesk-pm-agent                                            │
│ • Action: Evaluates alignment                                      │
│ • Tools: None (analysis)                                           │
│ • Output: Decision - REJECT original, SUGGEST alternative          │
└────────────────────┬───────────────────────────────────────────────┘
                     │
                     ↓
┌────────────────────────────────────────────────────────────────────┐
│ Step 7: egdesk-pm-agent                                            │
│ • Action: Returns strategic guide with insight                     │
│ • Tools: None                                                      │
│ • Output:                                                          │
│   - Summary: Floating cursor-following AI conflicts with spatial   │
│     canvas principles                                              │
│   - Decision: REJECT original idea                                 │
│   - Insight: Vision doc decided against floating/following UI      │
│     (breaks spatial affordances, users lose sense of place)        │
│   - Alternative: Proximity-based AI activation                     │
│   - Why better: Maintains spatial relationships                    │
│   - Vision-Aligned: ✅ Proximity-based aligns with principles      │
└────────────────────┬───────────────────────────────────────────────┘
                     │
          ┌──────────┴──────────┐
          │ PHASE 1: USER DECISION │
          └──────────┬──────────┘
                     ↓
┌────────────────────────────────────────────────────────────────────┐
│ Step 8: Main Thread                                                │
│ • Action: **Presents rejection + alternative to user**             │
│ • Tools: None                                                      │
│ • Output:                                                          │
│   "PM rejected floating AI (conflicts with spatial canvas).        │
│    Suggests proximity-based AI instead."                           │
│   **Options:**                                                     │
│   A) Accept alternative (proximity-based AI)                       │
│   B) Insist on floating AI (requires vision change)                │
│   C) Cancel feature                                                │
└────────────────────┬───────────────────────────────────────────────┘
                     │
                     ↓
┌────────────────────────────────────────────────────────────────────┐
│ Step 9: User                                                       │
│ • Action: **Accepts alternative**                                  │
│ • Tools: None (human decision)                                     │
│ • Output: "Makes sense. Let's do proximity-based AI instead."      │
└────────────────────┬───────────────────────────────────────────────┘
                     │
          ┌──────────┴───────────────┐
          │ PHASE 2: PROCEED WITH ALTERNATIVE │
          └──────────┬───────────────┘
                     ↓
┌────────────────────────────────────────────────────────────────────┐
│ Step 10: Main Thread                                               │
│ • Action: Invokes PM for alternative feature guide                 │
│ • Tools: Task(agent: "egdesk-pm-agent",                            │
│          prompt: "User accepted proximity-based AI alternative...")│
│ • Output: PM agent starts                                          │
└────────────────────┬───────────────────────────────────────────────┘
                     │
                     ↓
┌────────────────────────────────────────────────────────────────────┐
│ Step 11: egdesk-pm-agent                                           │
│ • Action: Creates PRD for alternative                              │
│ • Tools: Write("ideas&external_references/eg-desk ideas/           │
│          features/proximity-based-ai-prd.md", "[PRD content]")     │
│ • Output: PRD created for vision-aligned feature                   │
└────────────────────┬───────────────────────────────────────────────┘
                     │
                     ↓
┌────────────────────────────────────────────────────────────────────┐
│ Step 12: egdesk-pm-agent                                           │
│ • Action: Returns implementation guide                             │
│ • Tools: None                                                      │
│ • Output:                                                          │
│   - Decision: APPROVE                                              │
│   - Feature: Proximity-based AI activation                         │
│   - Framework: Theia + Infinite Canvas                             │
│   - Location: eg-desk_taehwa/ai/                                   │
│   - Next Steps: Follow Pattern 2 for implementation                │
└────────────────────┬───────────────────────────────────────────────┘
                     │
                     ↓
┌────────────────────────────────────────────────────────────────────┐
│ Step 13: Main Thread                                               │
│ • Action: Proceeds to implementation                               │
│ • Tools: (Follow Pattern 2: PM-Driven Development)                 │
│ • Output: Implementation begins                                    │
└────────────────────────────────────────────────────────────────────┘
```

**Total Agents Invoked:** 2 (PM rejection + PM guide for alternative)

**Who Wrote Code:** coding-agent (via Pattern 2)

**Duration:** Two PM turns + implementation

**Critical Observations:**
- **PM provides insight, not just rejection**: Explains why conflict exists with evidence from vision docs
- **Institutional memory**: "We previously decided X in document Y because Z"
- **Alternative suggestion**: Vision-aligned approach that solves same user need
- **USER DECISION POINT** (Step 8-9): User chooses between alternative, insisting on original, or canceling
- **User understands vision**: Not just "no", but "no because... and here's better way"
- **Smooth transition**: User accepts alternative → PM provides implementation guide → Proceeds normally
- **Context Preservation**: Main Thread didn't read vision docs - PM synthesized everything

**Key Point:** PM rejection includes **constructive alternative** - not blocking user, redirecting to vision-aligned solution.

---

### Scenario 3b: User Insists on Original Despite Vision Conflict

**User Request:**
```
(Continuation from Scenario 3a, Step 8)
Main Thread: "PM rejected floating AI (conflicts with spatial canvas). Suggests proximity-based AI instead. Options?"
User: "I understand the vision, but I still want floating AI. I think the vision should evolve - floating UI is more intuitive."
```

**Why this scenario:** User challenges vision with strong rationale - shows vision evolution process

**Step-by-Step Flow:**

```
┌────────────────────────────────────────────────────────────┐
│ PHASE 0: SAME AS SCENARIO 3a (Steps 1-7)                  │
│ User proposes → PM analyzes → PM rejects with alternative  │
│ Output: PM rejected floating AI, suggested proximity-based │
└────────────────────┬───────────────────────────────────────┘
                     │
          ┌──────────┴────────────────┐
          │ PHASE 1: USER INSISTS ON ORIGINAL │
          └──────────┬────────────────┘
                     ↓
┌────────────────────────────────────────────────────────────┐
│ Step 8: Main Thread                                        │
│ • Action: Presents rejection + alternative to user         │
│ • Tools: None                                              │
│ • Output: "PM rejected floating AI. Suggests proximity-    │
│           based. **Options?**"                             │
└────────────────────┬───────────────────────────────────────┘
                     │
                     ↓
┌────────────────────────────────────────────────────────────┐
│ Step 9: User                                               │
│ • Action: **Insists on original with rationale**           │
│ • Tools: None (human decision)                             │
│ • Output: "I understand vision, but floating AI is more    │
│           intuitive. Vision should evolve. Here's why..."  │
└────────────────────┬───────────────────────────────────────┘
                     │
          ┌──────────┴─────────────────────┐
          │ PHASE 2: PM RE-EVALUATES WITH USER RATIONALE │
          └──────────┬─────────────────────┘
                     ↓
┌────────────────────────────────────────────────────────────┐
│ Step 10: Main Thread                                       │
│ • Action: Returns to PM with user's argument               │
│ • Tools: Task(agent: "egdesk-pm-agent",                    │
│          prompt: "User reviewed vision conflict.           │
│          Insists on floating AI. User rationale:           │
│          'Floating UI more intuitive...'")                 │
│ • Output: PM agent starts (re-evaluation)                  │
└────────────────────┬───────────────────────────────────────┘
                     │
                     ↓
┌────────────────────────────────────────────────────────────┐
│ Step 11: egdesk-pm-agent                                   │
│ • Action: Re-reads vision docs                             │
│ • Tools: Read("EG-DESK_Spatial_Canvas_UX_Solutions.md")    │
│ • Output: Spatial affordance principles                    │
└────────────────────┬───────────────────────────────────────┘
                     │
                     ↓
┌────────────────────────────────────────────────────────────┐
│ Step 12: egdesk-pm-agent                                   │
│ • Action: Analyzes user's rationale                        │
│ • Tools: None (analysis)                                   │
│ • Output: User prioritizes "intuitiveness" over "spatial   │
│           affordances". Valid but different priority.      │
└────────────────────┬───────────────────────────────────────┘
                     │
                     ↓
┌────────────────────────────────────────────────────────────┐
│ Step 13: egdesk-pm-agent                                   │
│ • Action: **Makes final assessment**                       │
│ • Tools: None (strategic decision)                         │
│ • Output: **Options:**                                     │
│   A) Maintain vision (spatial > intuitiveness) - proximity │
│   B) Evolve vision (user's rationale strong) - flag change │
│   C) Hybrid approach (floating in certain contexts)        │
└────────────────────┬───────────────────────────────────────┘
                     │
                     ↓
┌────────────────────────────────────────────────────────────┐
│ Step 14: egdesk-pm-agent                                   │
│ • Action: Returns re-evaluation                            │
│ • Tools: None                                              │
│ • Output:                                                  │
│   - Assessment: User's rationale valid but vision          │
│     prioritizes spatial affordances                        │
│   - PM Stance: Recommend maintaining current vision        │
│   - User Decision Required: Strategic vision choice        │
│     * Maintain vision (use proximity-based)                │
│     * OR evolve vision (update docs, proceed with floating)│
└────────────────────┬───────────────────────────────────────┘
                     │
          ┌──────────┴────────────────┐
          │ PHASE 3: USER MAKES STRATEGIC DECISION │
          └──────────┬────────────────┘
                     ↓
┌────────────────────────────────────────────────────────────┐
│ Step 15: Main Thread                                       │
│ • Action: Presents vision choice to user                   │
│ • Tools: None                                              │
│ • Output: "PM reassessed. User's rationale valid, but      │
│           vision prioritizes spatial affordances.          │
│           **Your decision:**                               │
│           A) Maintain vision - use proximity-based AI      │
│           B) Evolve vision - proceed with floating AI"     │
└────────────────────┬───────────────────────────────────────┘
                     │
                     ↓
┌────────────────────────────────────────────────────────────┐
│ Step 16: User                                              │
│ • Action: **Makes strategic decision**                     │
│ • Tools: None (human authority)                            │
│ • Output: OPTIONS:                                         │
│   A) "OK, let's stick with vision - proximity-based is     │
│       fine"                                                │
│   B) "I want to evolve vision - floating AI is worth it"   │
└────────────────┬───────────────────────────────────────────┘
                 │
      ┌──────────┴──────────┐
      │                     │
   Branch A             Branch B
      │                     │
  Maintain              Evolve
   Vision               Vision
      │                     │
      └─────────────────────┼─────────────────►
                            ↓
      ┌─────────────────────────────────────────────────┐
      │ Step 17: Main Thread                            │
      │ • Action: Invokes PM to update vision           │
      │ • Tools: Task(agent: "egdesk-pm-agent",         │
      │          prompt: "User decided to evolve        │
      │          vision. Proceed with floating AI...")  │
      │ • Output: PM agent starts (vision update)       │
      └──────────────────┬──────────────────────────────┘
                         │
                         ↓
      ┌─────────────────────────────────────────────────┐
      │ Step 18: egdesk-pm-agent                        │
      │ • Action: **Updates vision document**           │
      │ • Tools: Edit("EG-DESK_Spatial_Canvas_UX_       │
      │          Solutions.md", add: "### Vision        │
      │          Evolution: Floating AI...")            │
      │ • Output: Vision doc updated                    │
      └──────────────────┬──────────────────────────────┘
                         │
                         ↓
      ┌─────────────────────────────────────────────────┐
      │ Step 19: egdesk-pm-agent                        │
      │ • Action: Creates PRD for floating AI           │
      │ • Tools: Write("ideas&external_references/      │
      │          eg-desk ideas/features/floating-ai-    │
      │          assistant-prd.md", "[PRD]")            │
      │ • Output: PRD created                           │
      └──────────────────┬──────────────────────────────┘
                         │
                         ↓
      ┌─────────────────────────────────────────────────┐
      │ Step 20: egdesk-pm-agent                        │
      │ • Action: Returns implementation guide          │
      │ • Tools: None                                   │
      │ • Output:                                       │
      │   - Decision: APPROVE (vision evolved per user) │
      │   - Vision Updated: Docs document evolution     │
      │   - Framework: Theia + Infinite Canvas          │
      │   - Location: eg-desk_taehwa/ai/                │
      │   - Next Steps: Follow Pattern 2                │
      └──────────────────┬──────────────────────────────┘
                         │
                         ↓
      ┌─────────────────────────────────────────────────┐
      │ Step 21: Main Thread                            │
      │ • Action: Proceeds to implementation            │
      │ • Tools: (Follow Pattern 2)                     │
      │ • Output: Implementation of floating AI         │
      └─────────────────────────────────────────────────┘
```

**Branch A: User maintains vision** → Proceed with proximity-based AI (follow Scenario 3a from Step 10)

**Branch B: User evolves vision** → Continue with Steps 17-21 above

**Total Agents Invoked:** 3 (PM rejection → PM re-evaluation → PM vision update + guide)

**Who Wrote Code:** coding-agent (via Pattern 2, Branch B only)

**Duration:** Three PM turns + implementation (if vision evolved)

**Critical Observations:**
- **User can challenge vision** with strong rationale
- **PM doesn't dictate** - PM provides assessment, user decides
- **Vision can evolve** - user has authority to change strategic direction
- **PM documents evolution** - updates vision docs with user's rationale
- **Tradeoffs documented** - "Why we evolved vision, what we're accepting"
- **Institutional memory preserved** - future developers know why vision changed
- **USER DECISION POINT** (Step 15-16): User chooses between maintaining vision or evolving it

**When user might insist:**
- Strong user rationale based on UX research
- Market feedback contradicts vision
- Strategic pivot needed
- User expertise in domain

**Key Point:** Vision is **not immutable** - PM enforces it, but user can evolve it with documented rationale.

---

### Scenario 3 Flow Diagram (Horizontal ASCII)

```
┌────────────────────────────────────────────────────────────────────────────┐
│ USER REQUEST: "Add floating AI assistant?"                                │
└──────────────────────┬─────────────────────────────────────────────────────┘
                       ▼
    ┌───────────────┐      ┌─────────────────┐      ┌──────────────────────┐
    │ MT: Route     │ ───→ │ PM: Analyze     │ ───→ │ PM: Glob vision docs │
    │ to PM         │      │ Vision Align    │      │ Read Whitepaper+UX   │
    └───────────────┘      └─────────────────┘      │ Grep 'floating'      │
                                                     └──────┬───────────────┘
                                                            ▼
                       ┌──────────────────────────────────────────────────────┐
                       │ PM Found: Previous decision AGAINST floating UI      │
                       │ Reason: Breaks spatial affordances                   │
                       └────────────────────┬─────────────────────────────────┘
                                            ▼
                       ┌──────────────────────────────────────────────────────┐
                       │ PM: Generate Alternative - Proximity-based AI        │
                       │ (vision-aligned solution)                            │
                       └────────────────────┬─────────────────────────────────┘
                                            ▼
                       ┌──────────────────────────────────────────────────────┐
                       │ PM Returns: REJECT original + Insight + Alternative  │
                       │ + Rationale                                          │
                       └────────────────────┬─────────────────────────────────┘
                                            ▼
                       ┌──────────────────────────────────────────────────────┐
                       │ Main Thread: Present to User                         │
                       └────────────────────┬─────────────────────────────────┘
                                            ▼
                       ┌──────────────────────────────────────────────────────┐
                       │ ★ USER DECISION 1: Accept alternative?               │
                       └──┬────────────────┬────────────────┬─────────────────┘
                          │                │                │
                A: Accept Alt    B: Insist Orig    C: Cancel
                          │                │                │
                          ▼                ▼                ▼
┌────────────────────────┐  ┌──────────────────────────────┐  ┌──────────────┐
│ PM: Guide for Alt      │  │ PM: Re-evaluate w/ User      │  │ STOP: Feature│
│  ↓                     │  │ Rationale                    │  │ canceled     │
│ PM: Create PRD         │  │  ↓                           │  └──────────────┘
│ proximity-ai-prd.md    │  │ PM: Assess Rationale         │
│  ↓                     │  │ Valid priority shift?        │
│ PM: Impl Guide         │  │ Vision evolution OK?         │
│  ↓                     │  │  ↓                           │
│ MT: Proceed            │  │ PM Returns: Rationale valid  │
│  ↓                     │  │ Options: A) Maintain vision  │
│ Follow Pattern 2:      │  │ B) Evolve vision (user pick) │
│ Implementation         │  │  ↓                           │
└────────────────────────┘  │ MT: Present Choice           │
                            │  ↓                           │
                            │ ★ USER DECISION 2: Maintain  │
                            │ or Evolve vision?            │
                            └──┬─────────────────┬─────────┘
                               │                 │
                     A: Maintain             B: Evolve
                               │                 │
                               │                 ▼
                               │   ┌──────────────────────────────┐
                               │   │ PM: Update Vision Docs       │
                               │   │  ↓                           │
                               │   │ PM: Edit UX Solutions doc    │
                               │   │ Document vision evolution    │
                               │   │ User rationale + tradeoffs   │
                               │   │  ↓                           │
                               │   │ PM: Create PRD               │
                               │   │ floating-ai-assistant-prd.md │
                               │   │  ↓                           │
                               │   │ PM: Impl Guide for float AI  │
                               │   │  ↓                           │
                               │   │ MT: Proceed to Impl          │
                               │   │  ↓                           │
                               └───→ Follow Pattern 2: Impl       │
                                   └──────────────────────────────┘
```

**Flow Paths:**
- **Path A (Accept Alternative)**: User accepts PM's vision-aligned alternative → Implementation
- **Path B (Insist Original)**: User insists → PM re-evaluates → User chooses maintain/evolve
  - **Path B1 (Maintain Vision)**: Same as Path A
  - **Path B2 (Evolve Vision)**: PM updates vision docs → Implementation of original idea

**Decision Gates:**
1. **User Decision 1**: Accept alternative? Insist on original? Cancel?
2. **User Decision 2** (if insisted): Maintain vision? Or evolve vision?

**Key Insight:**
- Vision conflicts don't block users
- PM provides constructive alternative
- User has authority to evolve vision (with documented rationale)

---

### Scenario 4: Development with PM Strategic Guide

**User Request:**
```
"Add a custom terminal theme that changes based on time of day"
```

**Step-by-Step Flow:**

```
          ┌─────────────────────────────┐
          │ PHASE 0: PM STRATEGIC GUIDE │
          └──────────┬──────────────────┘
                     ↓
┌────────────────────────────────────────────────────────────┐
│ Step 1: User                                               │
│ • Action: Requests feature                                 │
│ • Tools: None                                              │
│ • Output: Request sent                                     │
└────────────────────┬───────────────────────────────────────┘
                     │
                     ↓
┌────────────────────────────────────────────────────────────┐
│ Step 2: Main Thread                                        │
│ • Action: Analyzes - Development task, needs strategic     │
│           direction                                        │
│ • Tools: None                                              │
│ • Output: Routes to PM for initial guide                   │
└────────────────────┬───────────────────────────────────────┘
                     │
                     ↓
┌────────────────────────────────────────────────────────────┐
│ Step 3: Main Thread                                        │
│ • Action: Invokes PM agent                                 │
│ • Tools: Task(agent: "egdesk-pm-agent",                    │
│          prompt: "User wants time-based terminal theme...") │
│ • Output: PM agent starts                                  │
└────────────────────┬───────────────────────────────────────┘
                     │
                     ↓
┌────────────────────────────────────────────────────────────┐
│ Step 4: egdesk-pm-agent                                    │
│ • Action: Dynamic discovery                                │
│ • Tools: Glob("ideas&external_references/eg-desk ideas/    │
│          **/*.md")                                         │
│          Glob("packages/*/package.json")                   │
│          Grep("theme", "terminal")                         │
│ • Output: Finds vision docs + existing theme code          │
└────────────────────┬───────────────────────────────────────┘
                     │
                     ↓
┌────────────────────────────────────────────────────────────┐
│ Step 5: egdesk-pm-agent                                    │
│ • Action: Analyzes vision + structure                      │
│ • Tools: Read("EG-DESK_Whitepaper.md")                     │
│          Read("packages/terminal/package.json")            │
│          Read("packages/terminal/src/browser/terminal-     │
│          theme-service.ts")                                │
│ • Output: Extracts principles + discovers structure        │
└────────────────────┬───────────────────────────────────────┘
                     │
                     ↓
┌────────────────────────────────────────────────────────────┐
│ Step 6: egdesk-pm-agent                                    │
│ • Action: Creates PRD                                      │
│ • Tools: Write("ideas&external_references/eg-desk ideas/   │
│          features/time-based-terminal-theme-prd.md",       │
│          "[PRD content]")                                  │
│ • Output: PRD file created                                 │
└────────────────────┬───────────────────────────────────────┘
                     │
                     ↓
┌────────────────────────────────────────────────────────────┐
│ Step 7: egdesk-pm-agent                                    │
│ • Action: Returns strategic guide                          │
│ • Tools: None                                              │
│ • Output:                                                  │
│   - Decision: APPROVE                                      │
│   - Framework: Theia (terminal theming is Theia domain)    │
│   - Location: packages/terminal/src/browser/               │
│   - Approach: Phase 1 - analyze theme system,              │
│     Phase 2 - design service, Phase 3 - implement          │
│   - Considerations: Manual override needed, preference     │
│     persistence                                            │
│   - PRD Created: time-based-terminal-theme-prd.md          │
└────────────────────┬───────────────────────────────────────┘
                     │
          ┌──────────┴──────────────────────────────────┐
          │ PHASE 1: FRAMEWORK INVESTIGATION PLANNING │
          └──────────┬──────────────────────────────────┘
                     ↓
┌────────────────────────────────────────────────────────────┐
│ Step 8: Main Thread                                        │
│ • Action: Plans framework investigation based on PM guide  │
│ • Tools: None (planning)                                   │
│ • Output: Investigation plan - Query theia-analyzer for    │
│           theme patterns, DI registration patterns         │
└────────────────────┬───────────────────────────────────────┘
                     │
          ┌──────────┴────────────────────────────────────┐
          │ PHASE 2: FRAMEWORK INVESTIGATION (EXECUTION) │
          └──────────┬────────────────────────────────────┘
                     ↓
┌────────────────────────────────────────────────────────────┐
│ Step 9: Main Thread                                        │
│ • Action: Queries framework agent (per PM's guide)         │
│ • Tools: Task(agent: "theia-analyzer-agent",               │
│          prompt: "Analyze terminal theme system at         │
│          packages/terminal/src/browser/terminal-theme-     │
│          service.ts for registration and DI patterns")     │
│ • Output: Agent starts                                     │
└────────────────────┬───────────────────────────────────────┘
                     │
                     ↓
┌────────────────────────────────────────────────────────────┐
│ Step 10: theia-analyzer-agent                              │
│ • Action: Analyzes theme system                            │
│ • Tools: Read("packages/terminal/src/browser/terminal-     │
│          theme-service.ts")                                │
│          Read("packages/terminal/src/browser/terminal-     │
│          frontend-module.ts")                              │
│          Grep("@injectable")                               │
│ • Output: Finds patterns                                   │
└────────────────────┬───────────────────────────────────────┘
                     │
                     ↓
┌────────────────────────────────────────────────────────────┐
│ Step 11: theia-analyzer-agent                              │
│ • Action: Returns analysis                                 │
│ • Tools: None                                              │
│ • Output:                                                  │
│   - Files Analyzed: terminal-theme-service.ts:45,          │
│     terminal-frontend-module.ts:32                         │
│   - Pattern: ThemeService.register() with DI binding       │
│   - File List: CREATE time-based-theme-switcher.ts,        │
│     MODIFY terminal-frontend-module.ts:36,                 │
│     REFERENCE workspace-service.ts:89 for @injectable()    │
└────────────────────┬───────────────────────────────────────┘
                     │
          ┌──────────┴────────────────────────┐
          │ PHASE 3: IMPLEMENTATION PLANNING │
          └──────────┬────────────────────────┘
                     ↓
┌────────────────────────────────────────────────────────────┐
│ Step 12: Main Thread                                       │
│ • Action: Creates implementation plan (synthesizes PM      │
│           guide + framework patterns)                      │
│ • Tools: None (internal)                                   │
│ • Output: Implementation plan:                             │
│   - Direction: Create TimeBasedThemeSwitcher following     │
│     Theia DI pattern                                       │
│   - File List: CREATE time-based-theme-switcher.ts,        │
│     MODIFY terminal-frontend-module.ts:36,                 │
│     terminal-contribution.ts:89,                           │
│     REFERENCE workspace-service.ts:89 for @injectable()    │
│     pattern                                                │
└────────────────────┬───────────────────────────────────────┘
                     │
          ┌──────────┴──────────────────────────────────┐
          │ PHASE 4: IMPLEMENTATION EXECUTION (via      │
          │          coding-agent)                      │
          └──────────┬──────────────────────────────────┘
                     ↓
┌────────────────────────────────────────────────────────────┐
│ Step 13: Main Thread                                       │
│ • Action: Delegates to coding-agent                        │
│ • Tools: Task(agent: "coding-agent",                       │
│          prompt: "Implement time-based terminal theme      │
│          switcher. Direction: [what to implement].         │
│          Files: [file list]. [You will read files for      │
│          details]")                                        │
│ • Output: Coding agent starts                              │
└────────────────────┬───────────────────────────────────────┘
                     │
                     ↓
┌────────────────────────────────────────────────────────────┐
│ Step 14: coding-agent                                      │
│ • Action: Reads files and implements                       │
│ • Tools: Read(...) + Write(...) + Edit(...)                │
│ • Output: Implementation complete                          │
└────────────────────┬───────────────────────────────────────┘
                     │
                     ↓
┌────────────────────────────────────────────────────────────┐
│ Step 15: coding-agent                                      │
│ • Action: Returns report                                   │
│ • Tools: None                                              │
│ • Output: "Implementation complete: 1 CREATE, 2 MODIFY"    │
└────────────────────┬───────────────────────────────────────┘
                     │
          ┌──────────┴──────────────────────────┐
          │ PHASE 4.5: UX FLOW VALIDATION       │
          │            (Optional)               │
          └──────────┬──────────────────────────┘
                     ↓
┌────────────────────────────────────────────────────────────┐
│ Step 16a: Main Thread                                      │
│ • Action: (Optional) Decides to validate UX flow           │
│ • Tools: None                                              │
│ • Output: Complex feature, worth validating                │
└────────────────────┬───────────────────────────────────────┘
                     │
                     ↓
┌────────────────────────────────────────────────────────────┐
│ Step 16b: Main Thread                                      │
│ • Action: Invokes ux-flow-simulator                        │
│ • Tools: Task(agent: "ux-flow-simulator-agent",            │
│          prompt: "Trace execution flow: User opens         │
│          terminal → theme should switch based on time.     │
│          Files: [list]. Predict runtime behavior.")        │
│ • Output: Agent starts                                     │
└────────────────────┬───────────────────────────────────────┘
                     │
                     ↓
┌────────────────────────────────────────────────────────────┐
│ Step 16c: ux-flow-simulator-agent                          │
│ • Action: Traces code execution paths                      │
│ • Tools: Read(files) + traces logic                        │
│ • Output: Finds - No race conditions, expected behavior    │
│           correct                                          │
└────────────────────┬───────────────────────────────────────┘
                     │
                     ↓
┌────────────────────────────────────────────────────────────┐
│ Step 16d: ux-flow-simulator-agent                          │
│ • Action: Returns validation report                        │
│ • Tools: None                                              │
│ • Output: "✅ Flow validated: No issues predicted"         │
└────────────────────┬───────────────────────────────────────┘
                     │
          ┌──────────┴──────────────┐
          │ PHASE 5: BUILD & COMMIT │
          └──────────┬──────────────┘
                     ↓
┌────────────────────────────────────────────────────────────┐
│ Step 17: Main Thread                                       │
│ • Action: Builds, tests, commits                           │
│ • Tools: Bash("npm run build && git add . && git commit")  │
│ • Output: Committed                                        │
└────────────────────────────────────────────────────────────┘
```

**Total Agents Invoked:** 3-4 (PM strategic guide → framework analyzer → coding-agent → optional: ux-flow-simulator)

**Who Wrote Code:** coding-agent (step 14)

**Duration:** Multi-phase (PM guide → investigation planning → framework investigation → implementation planning → implementation execution)

**Critical Observations:**
- **PM provides complete strategic direction** (Phase 0): Framework, location, approach, considerations
- **PM creates PRD**: Documents approved feature
- **Main Thread plans investigation** (Phase 1): Identifies what framework patterns to research
- **Framework agent investigates** (Phase 2): Provides technical patterns (not strategic direction)
- **Main Thread creates implementation plan** (Phase 3): Synthesizes PM guide + framework patterns
- **coding-agent executes implementation** (Phase 4): Follows synthesized plan
- **(Optional) ux-flow-simulator validates logic** (Phase 4.5): Before build (Main Thread's discretion)
- **Main Thread builds/tests/commits** (Phase 5): Retains control of deployment

**Note**: ALL file changes go through coding-agent for consistency and context preservation.

**When to use ux-flow-simulator (optional):**
- ✅ Complex user interaction flows
- ✅ State management with potential race conditions
- ✅ Async operations or event handling
- ✅ Critical features (security, data integrity)
- ✅ Multi-step workflows
- ❌ Simple CRUD operations
- ❌ Pure UI styling changes
- ❌ Documentation updates

---

### Scenario 4 Flow Diagram (Horizontal ASCII)

```
┌────────────────────────────────────────────────────────────────────────────┐
│ USER REQUEST: "Add time-based terminal theme"                             │
└──────────────────────┬─────────────────────────────────────────────────────┘
                       ▼
[MT: Route to PM] ──→ [PM: Strategic Guide] ──→ [PM: Dynamic Discovery -
                                                  Glob vision docs + tech
                                                  stack, Grep features]
                                                       ▼
                       ┌──────────────────────────────────────────────────┐
                       │ PM Analysis: ✅ Vision aligned                   │
                       │              ✅ Theia framework                  │
                       │              ✅ No duplicates                    │
                       └────────────────────┬─────────────────────────────┘
                                            ▼
                       ┌──────────────────────────────────────────────────┐
                       │ PM: Create PRD - time-based-terminal-theme-      │
                       │ prd.md                                           │
                       └────────────────────┬─────────────────────────────┘
                                            ▼
                       ┌──────────────────────────────────────────────────┐
                       │ PM Returns: APPROVE                              │
                       │  + Framework: Theia                              │
                       │  + Location: packages/terminal/                  │
                       │  + Phasing: 3 phases                             │
                       │  + Considerations                                │
                       └────────────────────┬─────────────────────────────┘
                                            ▼
                       ┌──────────────────────────────────────────────────┐
                       │ MT: Plan Investigation                           │
                       │ Query theia-analyzer for patterns                │
                       └────────────────────┬─────────────────────────────┘
                                            ▼
┌────────────────────────────────────────────────────────────────────────────┐
│ theia-analyzer: Analyze theme system ──→ Read terminal-theme-service.ts   │
│                 ──→ Grep DI patterns                                       │
│                 ──→ Return: Patterns found + File list (CREATE/MODIFY)    │
└────────────────────────────────────────┬───────────────────────────────────┘
                                         ▼
                       ┌──────────────────────────────────────────────────┐
                       │ MT: Create Implementation Plan                   │
                       │ Synthesize PM guide + patterns                   │
                       └────────────────────┬─────────────────────────────┘
                                            ▼
                       ┌──────────────────────────────────────────────────┐
                       │ MT: Spawn coding-agent                           │
                       │ Direction + File list                            │
                       └────────────────────┬─────────────────────────────┘
                                            ▼
┌────────────────────────────────────────────────────────────────────────────┐
│ coding-agent: Read files ──→ Implement ──→ Return Report                  │
│               • CREATE time-based-switcher.ts                              │
│               • MODIFY frontend-module.ts                                  │
│               • MODIFY contribution.ts                                     │
└────────────────────────────────────────┬───────────────────────────────────┘
                                         ▼
                       ┌──────────────────────────────────────────────────┐
                       │ ★ MT: Validate UX flow?                          │
                       └──────┬────────────────────┬──────────────────────┘
                              │                    │
                       Complex flow            Simple
                              │                    │
                              ▼                    │
        ┌──────────────────────────────┐          │
        │ ux-flow-simulator:           │          │
        │ Trace execution              │          │
        └──────────┬───────────────────┘          │
                   ▼                               │
        ┌──────────────────────┐                  │
        │ Issues found?        │                  │
        └──┬───────────────┬───┘                  │
           │Yes            │No                    │
           ▼               │                      │
  [coding-agent: Fix] ────┘                      │
           │                                      │
           └──────────────────────────────────────┴──→ [MT: Build & Test]
                                                              ▼
                                                        [MT: Commit]
                                                              ▼
                                                  [Complete: Feature committed]
```

**Phases:**
- **Phase 0** (Yellow): PM Strategic Guide
- **Phase 1** (Green): Main Thread plans investigation
- **Phase 2** (Purple): Framework agent investigates
- **Phase 3** (Green): Main Thread creates implementation plan
- **Phase 4** (Pink): coding-agent executes
- **Phase 4.5** (Optional, Purple): UX flow validation
- **Phase 5** (Green): Build & commit

**Optional Validation Loop:**
- Main Thread decides: Complex flow? → Validate
- If issues found → coding-agent fixes → Re-validate
- If no issues → Proceed to build

---

### Scenario 4b: Development with PM Guide + Coding Agent Delegation

**User Request:**
```
"Add a custom terminal theme that changes based on time of day"
```

**Why use coding-agent:** Large implementation (3+ files), Main Thread context needs to stay clean for continued orchestration

**Step-by-Step Flow:**

```
┌──────────────────────────────────────────────────────────┐
│ PHASE 0-2: SAME AS SCENARIO 4 (Steps 1-11)              │
│ PM provides strategic guide → Main Thread creates plan  │
│ → Framework agent investigates                           │
│ Output: Strategic guide + Framework patterns collected   │
└────────────────────┬─────────────────────────────────────┘
                     │
          ┌──────────┴────────────────────────────────┐
          │ PHASE 3: IMPLEMENTATION                  │
          │          (Delegated to Coding Agent)     │
          └──────────┬────────────────────────────────┘
                     ↓
┌──────────────────────────────────────────────────────────┐
│ Step 12: Main Thread                                     │
│ • Action: Synthesizes PM guide + framework findings      │
│ • Tools: None (internal)                                 │
│ • Output: Direction - Create TimeBasedThemeSwitcher      │
│           File List: CREATE time-based-theme-switcher.ts,│
│           MODIFY terminal-frontend-module.ts:36,         │
│           terminal-contribution.ts:89,                   │
│           REFERENCE workspace-service.ts:89              │
└────────────────────┬─────────────────────────────────────┘
                     │
                     ↓
┌──────────────────────────────────────────────────────────┐
│ Step 13: Main Thread                                     │
│ • Action: Delegates to coding agent                      │
│ • Tools: Task(agent: "coding-agent",                     │
│          prompt: "Implement time-based terminal theme    │
│          switcher. Direction: Create                     │
│          TimeBasedThemeSwitcher service following Theia  │
│          DI pattern with automatic time-based switching  │
│          + manual override + preference persistence.     │
│          Files: CREATE:                                  │
│          packages/terminal/src/browser/time-based-theme- │
│          switcher.ts MODIFY: packages/terminal/src/      │
│          browser/terminal-frontend-module.ts:36 (add DI  │
│          binding) MODIFY: packages/terminal/src/browser/ │
│          terminal-contribution.ts:89 (inject service)    │
│          REFERENCE: packages/workspace/src/browser/      │
│          workspace-service.ts:89 (follow @injectable()   │
│          pattern) [You will read files for               │
│          implementation details]")                       │
│ • Output: Coding agent starts                            │
└────────────────────┬─────────────────────────────────────┘
                     │
                     ↓
┌──────────────────────────────────────────────────────────┐
│ Step 14: coding-agent                                    │
│ • Action: Reads pattern reference                        │
│ • Tools: Read("packages/workspace/src/browser/workspace- │
│          service.ts")                                    │
│ • Output: Understands DI pattern                         │
└────────────────────┬─────────────────────────────────────┘
                     │
                     ↓
┌──────────────────────────────────────────────────────────┐
│ Step 15: coding-agent                                    │
│ • Action: Creates service file                           │
│ • Tools: Write("packages/terminal/src/browser/time-based-│
│          theme-switcher.ts", "[code following pattern]") │
│ • Output: New file created                               │
└────────────────────┬─────────────────────────────────────┘
                     │
                     ↓
┌──────────────────────────────────────────────────────────┐
│ Step 16: coding-agent                                    │
│ • Action: Reads module file                              │
│ • Tools: Read("packages/terminal/src/browser/terminal-   │
│          frontend-module.ts")                            │
│ • Output: Current structure                              │
└────────────────────┬─────────────────────────────────────┘
                     │
                     ↓
┌──────────────────────────────────────────────────────────┐
│ Step 17: coding-agent                                    │
│ • Action: Registers in DI                                │
│ • Tools: Edit(old: "export default", new:                │
│          "bind(TimeBasedThemeSwitcher)...")              │
│ • Output: DI binding added                               │
└────────────────────┬─────────────────────────────────────┘
                     │
                     ↓
┌──────────────────────────────────────────────────────────┐
│ Step 18: coding-agent                                    │
│ • Action: Reads contribution file                        │
│ • Tools: Read("packages/terminal/src/browser/terminal-   │
│          contribution.ts")                               │
│ • Output: Current structure                              │
└────────────────────┬─────────────────────────────────────┘
                     │
                     ↓
┌──────────────────────────────────────────────────────────┐
│ Step 19: coding-agent                                    │
│ • Action: Integrates service                             │
│ • Tools: Edit(old: "export class", new:                  │
│          "@inject(TimeBasedThemeSwitcher)...")           │
│ • Output: Integration complete                           │
└────────────────────┬─────────────────────────────────────┘
                     │
                     ↓
┌──────────────────────────────────────────────────────────┐
│ Step 20: coding-agent                                    │
│ • Action: Returns report                                 │
│ • Tools: None                                            │
│ • Output: **Implementation Complete**: 1 CREATE, 2 MODIFY│
│           following workspace-service.ts pattern         │
└────────────────────┬─────────────────────────────────────┘
                     │
          ┌──────────┴────────────────────────────┐
          │ PHASE 4: VERIFICATION (Main Thread)  │
          └──────────┬────────────────────────────┘
                     ↓
┌──────────────────────────────────────────────────────────┐
│ Step 21: Main Thread                                     │
│ • Action: Builds package                                 │
│ • Tools: Bash("npm run build")                           │
│ • Output: Build succeeds                                 │
└────────────────────┬─────────────────────────────────────┘
                     │
                     ↓
┌──────────────────────────────────────────────────────────┐
│ Step 22: Main Thread                                     │
│ • Action: Stages and commits                             │
│ • Tools: Bash("git add packages/terminal && git commit   │
│          -m 'feat(terminal): add time-based theme        │
│          switcher'")                                     │
│ • Output: Committed                                      │
└────────────────────┬─────────────────────────────────────┘
                     │
                     ↓
┌──────────────────────────────────────────────────────────┐
│ Step 23: Main Thread                                     │
│ • Action: Reports to user                                │
│ • Tools: None                                            │
│ • Output: "Feature implemented and committed"            │
└──────────────────────────────────────────────────────────┘
```

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

### Scenario 4c: Development with Conflict Detection (EG-DESK Custom Code)

**User Request:**
```
"Bind Ctrl+K to the new QuickSearch feature"
```

**Why this scenario:** EG-DESK custom feature that needs conflict checking against existing keybindings

**Step-by-Step Flow:**

```
          ┌─────────────────────────────────────────────────┐
          │ PHASE 0-2: SAME AS SCENARIO 4b (Steps 1-11)    │
          │ PM strategic guide → Main Thread plan →         │
          │ Framework agent investigates                    │
          │ Output: Strategic guide + Framework patterns    │
          └──────────────────┬──────────────────────────────┘
                             │
          ┌──────────────────┴───────────────────────────────┐
          │ PHASE 3: CONFLICT DETECTION & IMPLEMENTATION     │
          └──────────────────┬───────────────────────────────┘
                             ↓
┌──────────────────────────────────────────────────────────────┐
│ Step 12: Main Thread                                         │
│ • Action: Synthesizes + adds conflict check instruction      │
│ • Tools: None (internal)                                     │
│ • Output: Direction: Bind Ctrl+K to QuickSearch              │
│           File List: CREATE search-contribution.ts           │
│           in eg-desk_taehwa/search/                          │
│           "Check CODEBASE_STRUCTURE.md for conflicts first"  │
└────────────────────────┬─────────────────────────────────────┘
                         │
                         ↓
┌──────────────────────────────────────────────────────────────┐
│ Step 13: Main Thread                                         │
│ • Action: Delegates to coding-agent                          │
│ • Tools: Task(agent: "coding-agent",                         │
│          prompt: "Bind Ctrl+K to QuickSearch.                │
│                   Direction: Create keybinding contribution  │
│                   Files: CREATE eg-desk_taehwa/search/       │
│                          search-contribution.ts              │
│                   BEFORE: Check CODEBASE_STRUCTURE.md")      │
│ • Output: Coding agent starts                                │
└────────────────────────┬─────────────────────────────────────┘
                         │
                         ↓
┌──────────────────────────────────────────────────────────────┐
│ Step 14: coding-agent                                        │
│ • Action: Discovers EG-DESK codebase                         │
│ • Tools: Glob("eg-desk*/**/*.ts")                            │
│ • Output: Finds: eg-desk_taehwa/                             │
└────────────────────────┬─────────────────────────────────────┘
                         │
                         ↓
┌──────────────────────────────────────────────────────────────┐
│ Step 15: coding-agent                                        │
│ • Action: Discovers structure document                       │
│ • Tools: Glob("eg-desk*/CODEBASE_STRUCTURE.md")              │
│ • Output: Finds: eg-desk_taehwa/CODEBASE_STRUCTURE.md        │
└────────────────────────┬─────────────────────────────────────┘
                         │
                         ↓
┌──────────────────────────────────────────────────────────────┐
│ Step 16: coding-agent                                        │
│ • Action: Reads structure document                           │
│ • Tools: Read("eg-desk_taehwa/CODEBASE_STRUCTURE.md")        │
│ • Output: Current keybindings registry                       │
└────────────────────────┬─────────────────────────────────────┘
                         │
                         ↓
┌──────────────────────────────────────────────────────────────┐
│ Step 17: coding-agent                                        │
│ • Action: **Checks for conflicts**                           │
│ • Tools: Grep("Ctrl+K") in structure doc                     │
│ • Output: **CONFLICT FOUND**:                                │
│           Ctrl+K already bound to DifferentFeature           │
│           at search-old.ts:45                                │
└────────────────────────┬─────────────────────────────────────┘
                         │
                         ↓
┌──────────────────────────────────────────────────────────────┐
│ Step 18: coding-agent                                        │
│ • Action: **STOPS implementation**                           │
│ • Tools: None                                                │
│ • Output: **Does NOT create any files**                      │
└────────────────────────┬─────────────────────────────────────┘
                         │
                         ↓
┌──────────────────────────────────────────────────────────────┐
│ Step 19: coding-agent                                        │
│ • Action: Returns conflict report                            │
│ • Tools: None                                                │
│ • Output: ❌ CONFLICT DETECTED                               │
│           Type: Keybinding conflict                          │
│           Requested: Ctrl+K for QuickSearch                  │
│           Existing: Ctrl+K for DifferentFeature              │
│                     (eg-desk_taehwa/search/search-old.ts:45) │
│           Severity: BLOCKER                                  │
│           Alternatives: Ctrl+Shift+K, Ctrl+Alt+K, Ctrl+J     │
│           **User decision required**                         │
└────────────────────────┬─────────────────────────────────────┘
                         │
          ┌──────────────┴────────────────┐
          │ PHASE 4: USER DECISION        │
          └──────────────┬────────────────┘
                         ↓
┌──────────────────────────────────────────────────────────────┐
│ Step 20: Main Thread                                         │
│ • Action: Presents conflict to user                          │
│ • Tools: None                                                │
│ • Output: "Conflict: Ctrl+K already used for                 │
│            DifferentFeature.                                 │
│            Options:                                          │
│            A) Use Ctrl+Shift+K                               │
│            B) Use Ctrl+Alt+K                                 │
│            C) Override DifferentFeature                      │
│            D) Choose different key"                          │
└────────────────────────┬─────────────────────────────────────┘
                         │
                         ↓
┌──────────────────────────────────────────────────────────────┐
│ Step 21: User                                                │
│ • Action: Makes decision                                     │
│ • Tools: None (human input)                                  │
│ • Output: "Use Ctrl+Shift+K"                                 │
└────────────────────────┬─────────────────────────────────────┘
                         │
          ┌──────────────┴─────────────────────┐
          │ PHASE 5: RETRY WITH RESOLUTION     │
          └──────────────┬─────────────────────┘
                         ↓
┌──────────────────────────────────────────────────────────────┐
│ Step 22: Main Thread                                         │
│ • Action: Retries with resolved key                          │
│ • Tools: Task(agent: "coding-agent",                         │
│          prompt: "Bind Ctrl+Shift+K to QuickSearch           │
│                   (Ctrl+K was conflicted).                   │
│                   Files: CREATE eg-desk_taehwa/search/       │
│                          search-contribution.ts")            │
│ • Output: Coding agent starts (retry)                        │
└────────────────────────┬─────────────────────────────────────┘
                         │
                         ↓
┌──────────────────────────────────────────────────────────────┐
│ Step 23: coding-agent                                        │
│ • Action: Checks for conflicts (Ctrl+Shift+K)                │
│ • Tools: Read + Grep CODEBASE_STRUCTURE.md                   │
│ • Output: ✅ No conflicts - Ctrl+Shift+K available           │
└────────────────────────┬─────────────────────────────────────┘
                         │
                         ↓
┌──────────────────────────────────────────────────────────────┐
│ Step 24: coding-agent                                        │
│ • Action: Implements code                                    │
│ • Tools: Write("eg-desk_taehwa/search/                       │
│          search-contribution.ts")                            │
│ • Output: File created with Ctrl+Shift+K binding             │
└────────────────────────┬─────────────────────────────────────┘
                         │
                         ↓
┌──────────────────────────────────────────────────────────────┐
│ Step 25: coding-agent                                        │
│ • Action: **Updates structure document**                     │
│ • Tools: Edit("eg-desk_taehwa/CODEBASE_STRUCTURE.md")        │
│ • Output: Added:                                             │
│           "Ctrl+Shift+K: QuickSearch                         │
│            (search-contribution.ts:67)"                      │
│           Added timeline entry:                              │
│           "2025-10-15: QuickSearch keybinding"               │
└────────────────────────┬─────────────────────────────────────┘
                         │
                         ↓
┌──────────────────────────────────────────────────────────────┐
│ Step 26: coding-agent                                        │
│ • Action: Returns success report                             │
│ • Tools: None                                                │
│ • Output: ✅ Implementation Complete                         │
│           Conflict Check: Passed (Ctrl+Shift+K available)    │
│           Files Created: search-contribution.ts              │
│           Structure Updated: Added keybinding + timeline     │
└────────────────────────┬─────────────────────────────────────┘
                         │
          ┌──────────────┴────────────────┐
          │ PHASE 6: VERIFICATION         │
          └──────────────┬────────────────┘
                         ↓
┌──────────────────────────────────────────────────────────────┐
│ Step 27: Main Thread                                         │
│ • Action: Builds and commits                                 │
│ • Tools: Bash("npm run build && git add eg-desk_taehwa       │
│          && git commit")                                     │
│ • Output: Committed                                          │
└────────────────────────┬─────────────────────────────────────┘
                         │
                         ↓
┌──────────────────────────────────────────────────────────────┐
│ Step 28: Main Thread                                         │
│ • Action: Reports to user                                    │
│ • Tools: None                                                │
│ • Output: "QuickSearch bound to Ctrl+Shift+K,                │
│            implemented and committed"                        │
└──────────────────────────────────────────────────────────────┘
```

**Total Agents Invoked:** 3 (PM strategic guide → framework analyzer → coding-agent twice)

**Who Wrote Code:** coding-agent (after conflict resolution)

**Duration:** Multi-phase with conflict detection and user decision

**Critical Observations:**
- **coding-agent discovered EG-DESK codebase dynamically** (Glob eg-desk*/**/*.ts)
- **Conflict detected BEFORE implementation** (prevented duplicate keybinding)
- **coding-agent STOPPED immediately** when conflict found (no auto-resolve)
- **User made final decision** on which alternative to use
- **Structure document automatically updated** after successful implementation
- **Always up-to-date registry** prevents future conflicts

**When to use this pattern:**
- Implementing EG-DESK custom features (not Theia framework modifications)
- Adding keybindings, services, commands, or any named entities
- Need to prevent naming/binding conflicts
- Want automatic structure tracking

**Key Difference from 4b:**
- 4b: Theia framework code (packages/*) - no conflict checking needed
- 4c: EG-DESK custom code (eg-desk_taehwa/*) - conflict checking required via CODEBASE_STRUCTURE.md

---

### Scenario 5: Agent Creation Request

**User Request:**
```
"We need an agent that analyzes Konva.js integration patterns"
```

**Step-by-Step Flow:**

```
┌──────────────────────────────────────────────────────────────┐
│ Step 1: User                                                 │
│ • Action: Requests new agent                                 │
│ • Tools: None                                                │
│ • Output: Request sent                                       │
└────────────────────────┬─────────────────────────────────────┘
                         │
                         ↓
┌──────────────────────────────────────────────────────────────┐
│ Step 2: Main Thread                                          │
│ • Action: Analyzes - Agent creation task                     │
│ • Tools: None                                                │
│ • Output: Routes to claude-agent-sdk-analyzer-agent          │
└────────────────────────┬─────────────────────────────────────┘
                         │
                         ↓
┌──────────────────────────────────────────────────────────────┐
│ Step 3: Main Thread                                          │
│ • Action: Invokes claude-agent                               │
│ • Tools: Task(agent: "claude-agent-sdk-analyzer-agent",      │
│          prompt: "Create an agent that analyzes Konva.js     │
│          integration patterns in the codebase")              │
│ • Output: Agent starts                                       │
└────────────────────────┬─────────────────────────────────────┘
                         │
                         ↓
┌──────────────────────────────────────────────────────────────┐
│ Step 4: claude-agent-sdk-analyzer-agent                      │
│ • Action: Reads best practices                               │
│ • Tools: Read("ideas&external_references/claude-agent-sdk/   │
│          subagent-best-practices.md")                        │
│ • Output: Learns agent design patterns                       │
└────────────────────────┬─────────────────────────────────────┘
                         │
                         ↓
┌──────────────────────────────────────────────────────────────┐
│ Step 5: claude-agent-sdk-analyzer-agent                      │
│ • Action: Examines existing analyzer agents                  │
│ • Tools: Glob(".claude/agents/*-analyzer-agent.md")          │
│          Read(".claude/agents/theia-analyzer-agent.md")      │
│          Read(".claude/agents/                               │
│               infinite-canvas-analyzer-agent.md")            │
│ • Output: Extracts proven patterns                           │
└────────────────────────┬─────────────────────────────────────┘
                         │
                         ↓
┌──────────────────────────────────────────────────────────────┐
│ Step 6: claude-agent-sdk-analyzer-agent                      │
│ • Action: Designs agent architecture                         │
│ • Tools: None (analysis)                                     │
│ • Output: YAML frontmatter + instruction structure planned   │
└────────────────────────┬─────────────────────────────────────┘
                         │
                         ↓
┌──────────────────────────────────────────────────────────────┐
│ Step 7: claude-agent-sdk-analyzer-agent                      │
│ • Action: Creates agent file                                 │
│ • Tools: Write(".claude/agents/konva-analyzer-agent.md",     │
│          "[complete agent spec following best practices]")   │
│ • Output: Agent file created                                 │
└────────────────────────┬─────────────────────────────────────┘
                         │
                         ↓
┌──────────────────────────────────────────────────────────────┐
│ Step 8: claude-agent-sdk-analyzer-agent                      │
│ • Action: Returns report (standard format)                   │
│ • Tools: None                                                │
│ • Output: **Summary**: Created konva-analyzer-agent          │
│           following framework analyzer patterns.             │
│           **Specifications**: Tools: Bash, Read, Glob,       │
│           Grep, WebSearch; Model: inherit                    │
│           **Best Practices Applied**: Evidence-based         │
│           analysis, contextual tool restriction,             │
│           standard reporting format                          │
│           **File Created**:                                  │
│           .claude/agents/konva-analyzer-agent.md             │
│           **Next Steps**: Session restart may be needed      │
│           to use new agent                                   │
└────────────────────────┬─────────────────────────────────────┘
                         │
                         ↓
┌──────────────────────────────────────────────────────────────┐
│ Step 9: Main Thread                                          │
│ • Action: Presents to user                                   │
│ • Tools: None                                                │
│ • Output: "Agent created. Restart session to use             │
│            konva-analyzer-agent."                            │
└──────────────────────────────────────────────────────────────┘
```

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

### Scenario 6: Simple File Edit (coding-agent)

**User Request:**
```
"Fix the typo in the README - change 'intsall' to 'install'"
```

**Step-by-Step Flow:**

```
┌──────────────────────────────────────────────────────────────┐
│ Step 1: User                                                 │
│ • Action: Reports typo                                       │
│ • Tools: None                                                │
│ • Output: Request sent                                       │
└────────────────────────┬─────────────────────────────────────┘
                         │
                         ↓
┌──────────────────────────────────────────────────────────────┐
│ Step 2: Main Thread                                          │
│ • Action: Analyzes - Simple edit                             │
│ • Tools: None                                                │
│ • Output: Routes to coding-agent                             │
└────────────────────────┬─────────────────────────────────────┘
                         │
                         ↓
┌──────────────────────────────────────────────────────────────┐
│ Step 3: Main Thread                                          │
│ • Action: Delegates to coding-agent                          │
│ • Tools: Task(agent: "coding-agent",                         │
│          prompt: "Fix typo in README.md:                     │
│          change 'intsall' to 'install'.                      │
│          Files: MODIFY README.md")                           │
│ • Output: Coding agent starts                                │
└────────────────────────┬─────────────────────────────────────┘
                         │
                         ↓
┌──────────────────────────────────────────────────────────────┐
│ Step 4: coding-agent                                         │
│ • Action: Reads README                                       │
│ • Tools: Read("README.md")                                   │
│ • Output: Finds typo                                         │
└────────────────────────┬─────────────────────────────────────┘
                         │
                         ↓
┌──────────────────────────────────────────────────────────────┐
│ Step 5: coding-agent                                         │
│ • Action: Fixes typo                                         │
│ • Tools: Edit("README.md",                                   │
│          old: "intsall", new: "install")                     │
│ • Output: File corrected                                     │
└────────────────────────┬─────────────────────────────────────┘
                         │
                         ↓
┌──────────────────────────────────────────────────────────────┐
│ Step 6: coding-agent                                         │
│ • Action: Returns report                                     │
│ • Tools: None                                                │
│ • Output: "Fixed typo in README.md"                          │
└────────────────────────┬─────────────────────────────────────┘
                         │
                         ↓
┌──────────────────────────────────────────────────────────────┐
│ Step 7: Main Thread                                          │
│ • Action: Commits                                            │
│ • Tools: Bash("git add README.md &&                          │
│          git commit -m 'docs: fix typo'")                    │
│ • Output: Committed                                          │
└────────────────────────┬─────────────────────────────────────┘
                         │
                         ↓
┌──────────────────────────────────────────────────────────────┐
│ Step 8: Main Thread                                          │
│ • Action: Reports to user                                    │
│ • Tools: None                                                │
│ • Output: "Fixed and committed"                              │
└──────────────────────────────────────────────────────────────┘
```

**Total Agents Invoked:** 1 (coding-agent)

**Who Wrote Code:** coding-agent (step 5)

**Duration:** Single agent invocation + commit

**Key Point:** ALL file changes go through coding-agent for consistency. Main Thread NEVER edits application files directly.

---

### Scenario 7a: New Technology Research & Evaluation (Happy Path)

**User Request:**
```
"캔버스에 3D visualization을 추가하고 싶어. 어떤 framework가 좋을까?"
```

**Why this scenario:** User needs capability not in current stack - requires PM to diagnose limitations, plan research, Main Thread to investigate options, PM to evaluate and recommend. Shows complete flow WITH user decision points.

**Step-by-Step Flow:**

```
          ┌───────────────────────────────────────────────────────┐
          │ PHASE 0: LIMITATION DIAGNOSIS & RESEARCH PLANNING     │
          └──────────────────────┬────────────────────────────────┘
                                 ↓
┌────────────────────────────────────────────────────────────────────┐
│ Step 1: User                                                       │
│ • Action: Requests new capability                                  │
│ • Tools: None                                                      │
│ • Output: "Add 3D visualization to canvas"                         │
└──────────────────────────┬─────────────────────────────────────────┘
                           │
                           ↓
┌────────────────────────────────────────────────────────────────────┐
│ Step 2: Main Thread                                                │
│ • Action: Analyzes - New capability request, may need new tech     │
│ • Tools: None                                                      │
│ • Output: Routes to PM for gap assessment                          │
└──────────────────────────┬─────────────────────────────────────────┘
                           │
                           ↓
┌────────────────────────────────────────────────────────────────────┐
│ Step 3: Main Thread                                                │
│ • Action: Invokes PM agent                                         │
│ • Tools: Task(agent: "egdesk-pm-agent",                            │
│          prompt: "User wants 3D visualization on canvas.           │
│          Assess if current stack supports this or if               │
│          new technology research needed.")                         │
│ • Output: PM agent starts                                          │
└──────────────────────────┬─────────────────────────────────────────┘
                           │
                           ↓
┌────────────────────────────────────────────────────────────────────┐
│ Step 4: egdesk-pm-agent                                            │
│ • Action: **Discovers tech stack**                                 │
│ • Tools: Glob("ideas*/**/eg-desk*ideas*/*tech*.md")                │
│          Read("technology-stack.md")                               │
│ • Output: Current stack: Theia, Electron, Infinite Canvas          │
│           (research), Konva.js (research), Claude API              │
└──────────────────────────┬─────────────────────────────────────────┘
                           │
                           ↓
┌────────────────────────────────────────────────────────────────────┐
│ Step 5: egdesk-pm-agent                                            │
│ • Action: **Analyzes current capabilities**                        │
│ • Tools: None (analysis of tech stack doc)                         │
│ • Output: Konva.js = 2D canvas library,                            │
│           Infinite Canvas = viewport transform system              │
└──────────────────────────┬─────────────────────────────────────────┘
                           │
                           ↓
┌────────────────────────────────────────────────────────────────────┐
│ Step 6: egdesk-pm-agent                                            │
│ • Action: **Diagnoses limitation**                                 │
│ • Tools: None (analysis)                                           │
│ • Output: **Gap Found**: User needs true 3D rendering              │
│           (lighting, shadows, depth). Konva.js limited to 2D       │
│           shapes. Pseudo-3D (perspective tricks) insufficient      │
│           for complex 3D scenes.                                   │
└──────────────────────────┬─────────────────────────────────────────┘
                           │
                           ↓
┌────────────────────────────────────────────────────────────────────┐
│ Step 7: egdesk-pm-agent                                            │
│ • Action: Discovers vision docs                                    │
│ • Tools: Glob("ideas*/**/eg-desk*ideas*/**/*.md")                  │
│          Read("EG-DESK_Whitepaper.md")                             │
│ • Output: Vision: Spatial canvas principles support 3D             │
│           visualization                                            │
└──────────────────────────┬─────────────────────────────────────────┘
                           │
                           ↓
┌────────────────────────────────────────────────────────────────────┐
│ Step 8: egdesk-pm-agent                                            │
│ • Action: **Creates research plan**                                │
│ • Tools: None (planning)                                           │
│ • Output: **Research Plan**: Investigate 3 options: Three.js,      │
│           Babylon.js, Custom WebGL. Criteria: bundle size          │
│           (<500KB), integration complexity, performance            │
│           (1000+ objects @60fps), Infinite Canvas compatibility    │
└──────────────────────────┬─────────────────────────────────────────┘
                           │
                           ↓
┌────────────────────────────────────────────────────────────────────┐
│ Step 9: egdesk-pm-agent                                            │
│ • Action: Returns research planning report                         │
│ • Tools: None                                                      │
│ • Output: **Decision**: RESEARCH_NEEDED                            │
│           **Current Stack Limitation**: Konva.js is 2D-only,       │
│           cannot render true 3D with lighting/shadows              │
│           **Why Research Needed**: User needs 3D object            │
│           manipulation with realistic rendering                   │
│           **Evaluation Criteria**: Bundle size, integration        │
│           complexity, performance, compatibility                   │
│           **Investigation Scope**: 3 parallel investigations       │
│           (Three.js, Babylon.js, Custom WebGL)                     │
│           **Expected Outcome**: Main Thread will return with       │
│           research findings for PM evaluation                      │
└──────────────────────────┬─────────────────────────────────────────┘
                           │
          ┌────────────────┴──────────────────────────────┐
          │ PHASE 1: USER DECISION (PROCEED WITH RESEARCH?)│
          └────────────────┬──────────────────────────────┘
                           ↓
┌────────────────────────────────────────────────────────────────────┐
│ Step 10: Main Thread                                               │
│ • Action: **Presents research plan to user**                       │
│ • Tools: None                                                      │
│ • Output: "PM diagnosed: Konva.js cannot do true 3D.               │
│            Recommends investigating Three.js, Babylon.js,          │
│            Custom WebGL. Criteria: bundle <500KB, integration      │
│            complexity, performance. **Proceed with research?**"    │
└──────────────────────────┬─────────────────────────────────────────┘
                           │
                           ↓
┌────────────────────────────────────────────────────────────────────┐
│ Step 11: User                                                      │
│ • Action: **Approves research**                                    │
│ • Tools: None (human decision)                                     │
│ • Output: "Yes, proceed with investigation"                        │
└──────────────────────────┬─────────────────────────────────────────┘
                           │
          ┌────────────────┴────────────────────────┐
          │ PHASE 2: PARALLEL RESEARCH EXECUTION    │
          └────────────────┬────────────────────────┘
                           ↓
┌────────────────────────────────────────────────────────────────────┐
│ Step 12: Main Thread                                               │
│ • Action: Plans parallel investigations                            │
│ • Tools: None (planning)                                           │
│ • Output: 3 investigations, all independent, run simultaneously    │
└──────────────────────────┬─────────────────────────────────────────┘
                           │
                           ↓
┌────────────────────────────────────────────────────────────────────┐
│ Step 13: Main Thread                                               │
│ • Action: Executes 3 parallel investigations                       │
│ • Tools: **Single message with 3 Tasks**:                          │
│          Task(agent: "general-purpose",                            │
│               prompt: "Research Three.js: bundle size,             │
│               3D capabilities, Infinite Canvas integration         │
│               approach, performance. Organize findings.")          │
│          Task(agent: "general-purpose",                            │
│               prompt: "Research Babylon.js: bundle size,           │
│               3D capabilities, Infinite Canvas integration         │
│               approach, performance. Organize findings.")          │
│          Task(agent: "general-purpose",                            │
│               prompt: "Research Custom WebGL: feasibility,         │
│               dev time, maintenance burden. Organize findings.")   │
│ • Output: 3 agents start simultaneously                            │
└──────────────────────────┬─────────────────────────────────────────┘
                           │
                           ├─────────────────────────┐
                           │                         │
                           ↓                         ↓
┌──────────────────────────────────┐  ┌──────────────────────────────┐
│ Step 14a: general-purpose (1)    │  │ Step 14b: general-purpose (2)│
│ • Action: Researches Three.js    │  │ • Action: Researches         │
│ • Tools: WebSearch("Three.js     │  │           Babylon.js         │
│          bundle size")           │  │ • Tools: WebSearch("Babylon  │
│          WebSearch("Three.js     │  │          .js bundle size")   │
│          Infinite Canvas         │  │          WebSearch("Babylon  │
│          integration")           │  │          .js canvas          │
│          WebFetch(threejs.org)   │  │          integration")       │
│ • Output: Three.js findings      │  │          WebFetch(babylon    │
│                                  │  │          js.com docs)        │
│                                  │  │ • Output: Babylon.js findings│
└──────────────┬───────────────────┘  └──────────┬───────────────────┘
               │                                  │
               │         ┌────────────────────────┘
               │         │
               │         │   ┌────────────────────────────────────┐
               │         │   │ Step 14c: general-purpose (3)      │
               │         │   │ • Action: Researches Custom WebGL  │
               │         │   │ • Tools: WebSearch("WebGL canvas   │
               │         │   │          integration")             │
               │         │   │          WebSearch("WebGL          │
               │         │   │          performance benchmarks")  │
               │         │   │ • Output: Custom WebGL feasibility │
               │         │   └──────────┬─────────────────────────┘
               │         │              │
               ↓         ↓              ↓
┌──────────────────────────────────────────────────────────────────┐
│ Steps 15a-c: general-purpose agents (1-3)                        │
│ • Action: Return findings                                        │
│ • Tools: None                                                    │
│ • Output: 15a: Three.js: ~600KB bundle, mature ecosystem, good   │
│                docs                                              │
│           15b: Babylon.js: ~1.2MB bundle, game-focused, complex  │
│                API                                               │
│           15c: Custom WebGL: Feasible but 2-3 months dev time +  │
│                maintenance                                       │
└──────────────────────────┬───────────────────────────────────────┘
                           │
          ┌────────────────┴──────────────────────┐
          │ PHASE 3: RESEARCH ORGANIZATION        │
          └────────────────┬──────────────────────┘
                           ↓
┌────────────────────────────────────────────────────────────────────┐
│ Step 16: Main Thread                                               │
│ • Action: Organizes findings into documents                        │
│ • Tools: Write("ideas&external_references/threejs-research.md",    │
│                "[Three.js findings]")                              │
│          Write("ideas&external_references/babylonjs-research.md",  │
│                "[Babylon.js findings]")                            │
│          Write("ideas&external_references/custom-webgl-            │
│                research.md", "[Custom WebGL findings]")            │
│ • Output: 3 research docs created                                  │
└──────────────────────────┬─────────────────────────────────────────┘
                           │
          ┌────────────────┴────────────────────┐
          │ PHASE 4: TECHNICAL ANALYSIS         │
          └────────────────┬────────────────────┘
                           ↓
┌────────────────────────────────────────────────────────────────────┐
│ Step 17: Main Thread                                               │
│ • Action: Queries analyzer for integration assessment              │
│ • Tools: Task(agent: "infinite-canvas-analyzer-agent",             │
│          prompt: "Read threejs-research.md and                     │
│          babylonjs-research.md. Assess integration complexity      │
│          with Infinite Canvas viewport transforms.")               │
│ • Output: Agent starts                                             │
└──────────────────────────┬─────────────────────────────────────────┘
                           │
                           ↓
┌────────────────────────────────────────────────────────────────────┐
│ Step 18: infinite-canvas-analyzer-agent                            │
│ • Action: Analyzes integration                                     │
│ • Tools: Read("ideas&external_references/threejs-research.md")     │
│          Read("ideas&external_references/babylonjs-research.md")   │
│          Read("ideas&external_references/infinite-canvas/          │
│               [integration code]")                                 │
│ • Output: Integration analysis                                     │
└──────────────────────────┬─────────────────────────────────────────┘
                           │
                           ↓
┌────────────────────────────────────────────────────────────────────┐
│ Step 19: infinite-canvas-analyzer-agent                            │
│ • Action: Returns technical assessment                             │
│ • Tools: None                                                      │
│ • Output: **Three.js**: Medium complexity - camera sync with       │
│           IC viewport                                              │
│           **Babylon.js**: High complexity - scene graph conflicts  │
│           with IC transform system                                 │
└──────────────────────────┬─────────────────────────────────────────┘
                           │
          ┌────────────────┴────────────────┐
          │ PHASE 5: PM EVALUATION          │
          └────────────────┬────────────────┘
                           ↓
┌────────────────────────────────────────────────────────────────────┐
│ Step 20: Main Thread                                               │
│ • Action: Returns to PM with research                              │
│ • Tools: Task(agent: "egdesk-pm-agent",                            │
│          prompt: "Previously you requested research on 3D          │
│          rendering with criteria [bundle, integration,             │
│          performance]. I've organized findings in                  │
│          ideas&external_references/:                               │
│          - threejs-research.md: 600KB, mature                      │
│          - babylonjs-research.md: 1.2MB, complex                   │
│          - custom-webgl-research.md: feasible but 3 months         │
│          Analyzer reported:                                        │
│          - Three.js: medium integration complexity                 │
│          - Babylon.js: high complexity                             │
│          Evaluate and recommend.")                                 │
│ • Output: PM agent starts (evaluation turn)                        │
└──────────────────────────┬─────────────────────────────────────────┘
                           │
                           ↓
┌────────────────────────────────────────────────────────────────────┐
│ Step 21: egdesk-pm-agent                                           │
│ • Action: Reads research documents                                 │
│ • Tools: Read("ideas&external_references/threejs-research.md")     │
│          Read("ideas&external_references/babylonjs-research.md")   │
│          Read("ideas&external_references/custom-webgl-             │
│               research.md")                                        │
│ • Output: All findings collected                                   │
└──────────────────────────┬─────────────────────────────────────────┘
                           │
                           ↓
┌────────────────────────────────────────────────────────────────────┐
│ Step 22: egdesk-pm-agent                                           │
│ • Action: **Evaluates against criteria**                           │
│ • Tools: None (analysis)                                           │
│ • Output: **Three.js**: Bundle size acceptable (600KB vs 500KB     │
│           target = minor overage), integration medium, best        │
│           balance                                                  │
│           **Babylon.js**: Bundle too large (1.2MB), integration    │
│           complex                                                  │
│           **Custom WebGL**: Too much dev time                      │
└──────────────────────────┬─────────────────────────────────────────┘
                           │
                           ↓
┌────────────────────────────────────────────────────────────────────┐
│ Step 23: egdesk-pm-agent                                           │
│ • Action: **Assesses vision alignment**                            │
│ • Tools: None (analysis)                                           │
│ • Output: Three.js best fits: lightweight enough, proven for       │
│           spatial UIs, good TypeScript support                     │
└──────────────────────────┬─────────────────────────────────────────┘
                           │
                           ↓
┌────────────────────────────────────────────────────────────────────┐
│ Step 24: egdesk-pm-agent                                           │
│ • Action: Returns evaluation report (WITHOUT updating stack yet)   │
│ • Tools: None                                                      │
│ • Output: **Recommended**: Three.js                                │
│           **Rationale**: Best balance of bundle size, integration  │
│           complexity, and capability. 600KB acceptable for 3D      │
│           rendering value.                                         │
│           **Scoring**: Three.js (4.2/5), Babylon.js (3.1/5),       │
│           Custom (2.5/5)                                           │
│           **Why not Babylon.js**: Bundle 2x too large              │
│           **Why not Custom**: 3 months dev time not justified      │
│           **Vision Alignment**: Supports spatial canvas with 3D    │
│           depth                                                    │
│           **Integration Strategy**: Phase 1 - POC, Phase 2 - IC    │
│           integration, Phase 3 - production                        │
└──────────────────────────┬─────────────────────────────────────────┘
                           │
          ┌────────────────┴─────────────────────────────┐
          │ PHASE 6: USER DECISION (ACCEPT RECOMMENDATION?)│
          └────────────────┬─────────────────────────────┘
                           ↓
┌────────────────────────────────────────────────────────────────────┐
│ Step 25: Main Thread                                               │
│ • Action: **Presents PM recommendation to user**                   │
│ • Tools: None                                                      │
│ • Output: "Research complete. PM evaluated 3 options:              │
│            - Three.js: 4.2/5 (recommended)                         │
│            - Babylon.js: 3.1/5 (bundle too large)                  │
│            - Custom WebGL: 2.5/5 (too much dev time)               │
│            PM recommends Three.js. **Proceed with Three.js?**"     │
└──────────────────────────┬─────────────────────────────────────────┘
                           │
                           ↓
┌────────────────────────────────────────────────────────────────────┐
│ Step 26: User                                                      │
│ • Action: **Approves recommendation**                              │
│ • Tools: None (human decision)                                     │
│ • Output: "Yes, proceed with Three.js"                             │
└──────────────────────────┬─────────────────────────────────────────┘
                           │
          ┌────────────────┴────────────────────┐
          │ PHASE 7: TECHNOLOGY ADOPTION        │
          └────────────────┬────────────────────┘
                           ↓
┌────────────────────────────────────────────────────────────────────┐
│ Step 27: Main Thread                                               │
│ • Action: Requests PM to finalize adoption                         │
│ • Tools: Task(agent: "egdesk-pm-agent",                            │
│          prompt: "User approved Three.js. Update                   │
│          technology-stack.md and create integration PRD.")         │
│ • Output: PM agent starts (finalization)                           │
└──────────────────────────┬─────────────────────────────────────────┘
                           │
                           ↓
┌────────────────────────────────────────────────────────────────────┐
│ Step 28: egdesk-pm-agent                                           │
│ • Action: **Updates technology-stack.md**                          │
│ • Tools: Edit("ideas&external_references/eg-desk ideas/            │
│          technology-stack.md",                                     │
│          add: "### Three.js\n- Category: 3D Rendering\n            │
│          - Status: Approved for Integration\n                      │
│          - Capabilities: 3D object rendering, lighting, shadows\n  │
│          - Bundle: ~600KB\n                                        │
│          - Integration: Layers with Infinite Canvas")              │
│ • Output: Stack updated                                            │
└──────────────────────────┬─────────────────────────────────────────┘
                           │
                           ↓
┌────────────────────────────────────────────────────────────────────┐
│ Step 29: egdesk-pm-agent                                           │
│ • Action: **Creates integration PRD**                              │
│ • Tools: Write("ideas&external_references/eg-desk ideas/features/  │
│          3d-visualization-with-threejs-prd.md",                    │
│          "[PRD with research summary + decision rationale +        │
│          integration phases]")                                     │
│ • Output: PRD created                                              │
└──────────────────────────┬─────────────────────────────────────────┘
                           │
                           ↓
┌────────────────────────────────────────────────────────────────────┐
│ Step 30: egdesk-pm-agent                                           │
│ • Action: Returns finalization report                              │
│ • Tools: None                                                      │
│ • Output: **Technology Stack Updated**: Three.js added             │
│           (Approved for Integration)                               │
│           **PRD Created**: 3d-visualization-with-threejs-prd.md    │
│           **Next Steps**: Query framework agents for integration   │
│           patterns                                                 │
└──────────────────────────┬─────────────────────────────────────────┘
                           │
          ┌────────────────┴────────────────────────┐
          │ PHASE 8: IMPLEMENTATION PLANNING        │
          └────────────────┬────────────────────────┘
                           ↓
┌────────────────────────────────────────────────────────────────────┐
│ Step 31: Main Thread                                               │
│ • Action: Reports to user                                          │
│ • Tools: None                                                      │
│ • Output: "Three.js added to stack. Ready to begin integration     │
│            planning."                                              │
└──────────────────────────┬─────────────────────────────────────────┘
                           │
                           ↓
┌────────────────────────────────────────────────────────────────────┐
│ Step 32: Main Thread                                               │
│ • Action: Queries framework agents for integration                 │
│ • Tools: Task(agent: "theia-analyzer-agent",                       │
│          prompt: "Analyze webview integration for Three.js canvas  │
│          in Theia")                                                │
│          Task(agent: "electron-analyzer-agent",                    │
│          prompt: "Electron security for Three.js in renderer       │
│          process")                                                 │
│ • Output: 2 agents start (parallel)                                │
└──────────────────────────┬─────────────────────────────────────────┘
                           │
                           ↓
┌────────────────────────────────────────────────────────────────────┐
│ Step 33: Framework agents                                          │
│ • Action: Analyze integration patterns                             │
│ • Tools: Read(...) + Grep(...)                                     │
│ • Output: Integration guidance                                     │
└──────────────────────────┬─────────────────────────────────────────┘
                           │
                           ↓
┌────────────────────────────────────────────────────────────────────┐
│ Step 34: Main Thread                                               │
│ • Action: Synthesizes + spawns coding-agent                        │
│ • Tools: Task(agent: "coding-agent",                               │
│          prompt: "Implement Three.js integration.                  │
│          Direction: [PM guide].                                    │
│          Files: [from analyzer agents]")                           │
│ • Output: Implementation begins                                    │
└────────────────────────────────────────────────────────────────────┘
```

**Total Agents Invoked:** 8 (PM planning → 3 research agents parallel → 1 analyzer → PM evaluation → PM finalization → 2 framework agents parallel → coding-agent)

**Who Wrote Code:** coding-agent (step 34)

**Duration:** Multi-phase with 2 user decision points (diagnosis → **user approves research** → parallel research → organization → analysis → evaluation → **user approves recommendation** → adoption → implementation)

**Critical Observations:**
- **PM diagnosed current stack limitation** (Konva.js cannot do true 3D)
- **PM articulated WHY research needed** (not just "we need X", but "current tech Y has limitation Z")
- **PM created detailed research plan** with evaluation criteria (bundle size, integration, performance)
- **PM designed for parallel execution** (3 independent investigations)
- **USER DECISION POINT 1** (Step 10-11): User approves proceeding with research
- **Main Thread executed 3 parallel investigations** (faster than sequential)
- **Main Thread organized findings** in ideas&external_references/ (institutional memory)
- **Main Thread returned to PM** with research results
- **PM evaluated based on actual research** (not assumptions)
- **PM recommended Three.js** with detailed rationale (scoring against criteria)
- **USER DECISION POINT 2** (Step 25-26): User approves PM's recommendation
- **PM updated technology-stack.md AFTER user approval** (stack evolution)
- **Research documents preserved** (why Three.js chosen over alternatives documented)

**When to use this pattern:**
- User needs capability not in current stack
- Current stack has limitations for requirement
- Multiple technology options exist
- Need comparative analysis before decision
- Technology stack evolution

**Key Points:**
1. **PM doesn't choose framework** - PM diagnoses gap + plans research
2. **User approves research plan** before investigation starts
3. **Main Thread does actual investigation** - WebSearch, organize findings (parallel)
4. **PM evaluates results** - vision-aligned recommendation with scoring
5. **User approves recommendation** before technology adoption
6. **Institutional memory** - research docs preserved in ideas&external_references/
7. **Technology stack evolves** - PM updates stack doc after user approval

---

### Scenario 7b: User Rejects PM Recommendation (Chooses Different Option)

**User Request:**
```
(Continuation from Scenario 7a, Step 25)
Main Thread: "PM recommends Three.js. Proceed with Three.js?"
User: "No, I prefer Babylon.js - I need the advanced features even if bundle is larger"
```

**Why this scenario:** User overrides PM recommendation based on different priorities - shows iterative decision-making

**Step-by-Step Flow:**

```
          ┌──────────────────────────────────────────────────┐
          │ PHASE 0-5: SAME AS SCENARIO 7a (Steps 1-24)     │
          │ PM diagnoses → User approves research →          │
          │ Parallel investigation → PM evaluates            │
          │ Output: PM recommended: Three.js (4.2/5)         │
          └──────────────────┬───────────────────────────────┘
                             │
          ┌──────────────────┴──────────────────────────┐
          │ PHASE 6: USER REJECTS RECOMMENDATION        │
          └──────────────────┬──────────────────────────┘
                             ↓
┌────────────────────────────────────────────────────────────────────┐
│ Step 25: Main Thread                                               │
│ • Action: Presents PM recommendation to user                       │
│ • Tools: None                                                      │
│ • Output: "PM recommends Three.js. **Proceed with Three.js?**"    │
└──────────────────────────┬─────────────────────────────────────────┘
                           │
                           ↓
┌────────────────────────────────────────────────────────────────────┐
│ Step 26: User                                                      │
│ • Action: **Rejects recommendation, chooses alternative**          │
│ • Tools: None (human decision)                                     │
│ • Output: "No, use Babylon.js - advanced features worth the        │
│            bundle size tradeoff"                                   │
└──────────────────────────┬─────────────────────────────────────────┘
                           │
          ┌────────────────┴───────────────────────────────┐
          │ PHASE 7: PM RE-EVALUATION WITH USER OVERRIDE   │
          └────────────────┬───────────────────────────────┘
                           ↓
┌────────────────────────────────────────────────────────────────────┐
│ Step 27: Main Thread                                               │
│ • Action: Returns to PM with user decision                         │
│ • Tools: Task(agent: "egdesk-pm-agent",                            │
│          prompt: "User reviewed PM recommendation of Three.js      │
│          but chooses Babylon.js instead. User rationale:           │
│          'Advanced features worth bundle size tradeoff'.           │
│          Reassess: Does Babylon.js align with vision given         │
│          user's priority shift? Update integration strategy        │
│          accordingly.")                                            │
│ • Output: PM agent starts (re-evaluation)                          │
└──────────────────────────┬─────────────────────────────────────────┘
                           │
                           ↓
┌────────────────────────────────────────────────────────────────────┐
│ Step 28: egdesk-pm-agent                                           │
│ • Action: Reads Babylon.js research                                │
│ • Tools: Read("ideas&external_references/babylonjs-research.md")   │
│ • Output: Babylon.js details                                       │
└──────────────────────────┬─────────────────────────────────────────┘
                           │
                           ↓
┌────────────────────────────────────────────────────────────────────┐
│ Step 29: egdesk-pm-agent                                           │
│ • Action: Re-evaluates with user priority                          │
│ • Tools: None (analysis)                                           │
│ • Output: User prioritizes features over bundle. Babylon.js has    │
│           advanced lighting, physics, VR support. Vision           │
│           alignment: ✅ if user accepts bundle impact              │
└──────────────────────────┬─────────────────────────────────────────┘
                           │
                           ↓
┌────────────────────────────────────────────────────────────────────┐
│ Step 30: egdesk-pm-agent                                           │
│ • Action: **Updates technology-stack.md with Babylon.js**          │
│ • Tools: Edit("ideas&external_references/eg-desk ideas/            │
│          technology-stack.md",                                     │
│          add: "### Babylon.js\n- Category: 3D Rendering\n          │
│          - Status: Approved (user priority: features > bundle)\n   │
│          - Capabilities: Advanced 3D, physics, VR\n                │
│          - Bundle: ~1.2MB (accepted tradeoff)")                    │
│ • Output: Stack updated with Babylon.js                            │
└──────────────────────────┬─────────────────────────────────────────┘
                           │
                           ↓
┌────────────────────────────────────────────────────────────────────┐
│ Step 31: egdesk-pm-agent                                           │
│ • Action: **Creates integration PRD**                              │
│ • Tools: Write("ideas&external_references/eg-desk ideas/features/  │
│          3d-visualization-with-babylonjs-prd.md",                  │
│          "[PRD: User chose Babylon.js, rationale documented,       │
│          integration strategy adjusted for larger bundle]")        │
│ • Output: PRD created                                              │
└──────────────────────────┬─────────────────────────────────────────┘
                           │
                           ↓
┌────────────────────────────────────────────────────────────────────┐
│ Step 32: egdesk-pm-agent                                           │
│ • Action: Returns updated guide                                    │
│ • Tools: None                                                      │
│ • Output: **Technology Adopted**: Babylon.js (per user decision)   │
│           **User Rationale Documented**: Advanced features         │
│           prioritized over bundle size                             │
│           **PM Assessment**: Vision-aligned given user's priority  │
│           **Adjusted Considerations**: Lazy-loading for bundle     │
│           mitigation, bundle splitting strategy                    │
│           **Integration Strategy**: Phase 1 - POC with bundle      │
│           analysis, Phase 2 - bundle optimization, Phase 3 - IC    │
│           integration                                              │
└──────────────────────────┬─────────────────────────────────────────┘
                           │
          ┌────────────────┴────────────────────────┐
          │ PHASE 8: IMPLEMENTATION PLANNING        │
          └────────────────┬────────────────────────┘
                           ↓
┌────────────────────────────────────────────────────────────────────┐
│ Step 33: Main Thread                                               │
│ • Action: Reports to user                                          │
│ • Tools: None                                                      │
│ • Output: "Babylon.js added to stack per your decision.            │
│            Proceeding with integration planning."                  │
└──────────────────────────┬─────────────────────────────────────────┘
                           │
                           ↓
┌────────────────────────────────────────────────────────────────────┐
│ Step 34: Main Thread                                               │
│ • Action: Queries framework agents for Babylon.js integration      │
│ • Tools: Task(agent: "theia-analyzer-agent",                       │
│          prompt: "Analyze webview integration for Babylon.js       │
│          in Theia")                                                │
│          Task(agent: "electron-analyzer-agent",                    │
│          prompt: "Electron bundle optimization for Babylon.js      │
│          (1.2MB)")                                                 │
│ • Output: 2 agents start                                           │
└──────────────────────────┬─────────────────────────────────────────┘
                           │
                           ↓
┌────────────────────────────────────────────────────────────────────┐
│ Step 35: Framework agents                                          │
│ • Action: Analyze integration + bundle optimization                │
│ • Tools: Read(...) + Grep(...)                                     │
│ • Output: Integration guidance + bundle mitigation strategies      │
└──────────────────────────┬─────────────────────────────────────────┘
                           │
                           ↓
┌────────────────────────────────────────────────────────────────────┐
│ Step 36: Main Thread                                               │
│ • Action: Synthesizes + spawns coding-agent                        │
│ • Tools: Task(agent: "coding-agent",                               │
│          prompt: "Implement Babylon.js integration with bundle     │
│          optimization. Direction: [PM guide].                      │
│          Files: [from analyzer agents]")                           │
│ • Output: Implementation begins                                    │
└────────────────────────────────────────────────────────────────────┘
```

**Total Agents Invoked:** 8 (Same as 7a, but PM re-evaluates with user override)

**Who Wrote Code:** coding-agent (step 36)

**Duration:** Multi-phase with user override (same as 7a + re-evaluation turn)

**Critical Observations:**
- **User has final decision authority** - can override PM recommendation
- **PM adapts to user priorities** - re-evaluates with different weighting
- **User rationale documented** - "Why Babylon.js chosen despite PM recommending Three.js"
- **Integration strategy adjusted** - PM accounts for bundle size mitigation needs
- **Institutional memory preserved** - PRD documents user's decision rationale
- **Vision alignment re-assessed** - PM confirms alignment given user's priority shift

**When user might override PM:**
- Different priorities (features > bundle size)
- Team expertise (familiar with Babylon.js)
- Strategic reasons (future VR support needed)
- Risk tolerance (willing to accept tradeoffs)

**Key Point:** PM guides, but **user decides**. PM adapts strategy to user's choice.

---

### Scenario 7c: User Modifies Research Criteria (Iterative Investigation)

**User Request:**
```
(After Scenario 7a Step 10)
Main Thread: "PM recommends investigating Three.js, Babylon.js, Custom WebGL. Criteria: bundle <500KB. Proceed?"
User: "Add Pixi.js to investigation. And change bundle criterion - I don't care about bundle size, performance is critical."
```

**Why this scenario:** User modifies research scope before investigation starts - shows iterative planning

**Step-by-Step Flow:**

```
          ┌──────────────────────────────────────────────────┐
          │ PHASE 0: SAME AS SCENARIO 7a (Steps 1-9)        │
          │ PM diagnoses limitation and creates research plan│
          │ Output: Investigate Three.js, Babylon.js,        │
          │ Custom WebGL with bundle <500KB criteria         │
          └──────────────────┬───────────────────────────────┘
                             │
          ┌──────────────────┴────────────────────────┐
          │ PHASE 1: USER MODIFIES RESEARCH PLAN      │
          └──────────────────┬────────────────────────┘
                             ↓
┌────────────────────────────────────────────────────────────────────┐
│ Step 10: Main Thread                                               │
│ • Action: Presents research plan to user                           │
│ • Tools: None                                                      │
│ • Output: "PM recommends investigating 3 options with bundle       │
│            <500KB criterion. **Proceed?**"                         │
└──────────────────────────┬─────────────────────────────────────────┘
                           │
                           ↓
┌────────────────────────────────────────────────────────────────────┐
│ Step 11: User                                                      │
│ • Action: **Requests modifications**                               │
│ • Tools: None (human input)                                        │
│ • Output: "Add Pixi.js to investigation. Change criterion:         │
│            ignore bundle size, prioritize performance (render      │
│            speed)"                                                 │
└──────────────────────────┬─────────────────────────────────────────┘
                           │
          ┌────────────────┴──────────────────────────┐
          │ PHASE 2: PM ADJUSTS RESEARCH PLAN         │
          └────────────────┬──────────────────────────┘
                           ↓
┌────────────────────────────────────────────────────────────────────┐
│ Step 12: Main Thread                                               │
│ • Action: Returns to PM with user modifications                    │
│ • Tools: Task(agent: "egdesk-pm-agent",                            │
│          prompt: "User reviewed research plan. Requests            │
│          modifications:                                            │
│          1. Add Pixi.js as Option D                                │
│          2. Change evaluation criteria: deprioritize bundle size,  │
│             prioritize rendering performance                       │
│          Adjust research plan accordingly.")                       │
│ • Output: PM agent starts (plan adjustment)                        │
└──────────────────────────┬─────────────────────────────────────────┘
                           │
                           ↓
┌────────────────────────────────────────────────────────────────────┐
│ Step 13: egdesk-pm-agent                                           │
│ • Action: Analyzes Pixi.js relevance                               │
│ • Tools: WebSearch("Pixi.js 3D capabilities")                      │
│ • Output: Pixi.js = primarily 2D WebGL renderer, limited 3D        │
│           support                                                  │
└──────────────────────────┬─────────────────────────────────────────┘
                           │
                           ↓
┌────────────────────────────────────────────────────────────────────┐
│ Step 14: egdesk-pm-agent                                           │
│ • Action: Adjusts research plan                                    │
│ • Tools: None (planning)                                           │
│ • Output: **Research Plan Updated**: 4 options (added Pixi.js),    │
│           **Criteria Adjusted**: Performance (1000 obj @60fps) =   │
│           CRITICAL, Bundle size = DEPRIORITIZED, Integration       │
│           complexity, Infinite Canvas compatibility                │
└──────────────────────────┬─────────────────────────────────────────┘
                           │
                           ↓
┌────────────────────────────────────────────────────────────────────┐
│ Step 15: egdesk-pm-agent                                           │
│ • Action: Returns adjusted research plan                           │
│ • Tools: None                                                      │
│ • Output: **Investigation Updated**: Added Pixi.js (Option D)      │
│           **Evaluation Criteria Adjusted**: Performance now        │
│           critical weight, bundle deprioritized                    │
│           **New Investigation Questions**: Focus on render speed   │
│           benchmarks, less on bundle size                          │
│           **Parallel Execution**: 4 investigations (Three.js,      │
│           Babylon.js, Custom WebGL, Pixi.js)                       │
└──────────────────────────┬─────────────────────────────────────────┘
                           │
          ┌────────────────┴─────────────────────────────┐
          │ PHASE 3: USER CONFIRMS ADJUSTED PLAN         │
          └────────────────┬─────────────────────────────┘
                           ↓
┌────────────────────────────────────────────────────────────────────┐
│ Step 16: Main Thread                                               │
│ • Action: Presents adjusted plan                                   │
│ • Tools: None                                                      │
│ • Output: "PM adjusted plan: 4 options (added Pixi.js),            │
│            performance prioritized. **Proceed?**"                  │
└──────────────────────────┬─────────────────────────────────────────┘
                           │
                           ↓
┌────────────────────────────────────────────────────────────────────┐
│ Step 17: User                                                      │
│ • Action: Approves adjusted plan                                   │
│ • Tools: None (human decision)                                     │
│ • Output: "Yes, proceed"                                           │
└──────────────────────────┬─────────────────────────────────────────┘
                           │
          ┌────────────────┴───────────────────────────────┐
          │ PHASE 4: PARALLEL RESEARCH EXECUTION (4 OPTIONS)│
          └────────────────┬───────────────────────────────┘
                           ↓
┌────────────────────────────────────────────────────────────────────┐
│ Step 18: Main Thread                                               │
│ • Action: Executes 4 parallel investigations                       │
│ • Tools: **Single message with 4 Tasks**:                          │
│          Task(agent: "general-purpose",                            │
│               "Research Three.js performance...")                  │
│          Task(agent: "general-purpose",                            │
│               "Research Babylon.js performance...")                │
│          Task(agent: "general-purpose",                            │
│               "Research Custom WebGL performance...")              │
│          Task(agent: "general-purpose",                            │
│               "Research Pixi.js 3D capabilities...")               │
│ • Output: 4 agents start simultaneously                            │
└──────────────────────────┬─────────────────────────────────────────┘
                           │
                           ↓
┌────────────────────────────────────────────────────────────────────┐
│ Steps 19a-d: general-purpose (1-4)                                 │
│ • Action: Research 4 options                                       │
│ • Tools: WebSearch(...) + WebFetch(...)                            │
│ • Output: Findings for each option                                 │
└──────────────────────────┬─────────────────────────────────────────┘
                           │
                           ↓
┌────────────────────────────────────────────────────────────────────┐
│ Steps 20a-d: general-purpose (1-4)                                 │
│ • Action: Return findings                                          │
│ • Tools: None                                                      │
│ • Output: Research results with performance focus                  │
└──────────────────────────┬─────────────────────────────────────────┘
                           │
          ┌────────────────┴───────────────────────────────────────┐
          │ PHASE 5-8: SAME AS SCENARIO 7a                         │
          │ (Research organization → Analysis → PM evaluation →    │
          │  User approval → Adoption → Implementation)            │
          │ Output: Complete with user's modified criteria         │
          └────────────────────────────────────────────────────────┘
```

**Total Agents Invoked:** 9 (PM planning → PM adjust plan → 4 research agents → 1 analyzer → PM evaluation → PM finalization → 2 framework agents → coding-agent)

**Who Wrote Code:** coding-agent (final step)

**Duration:** Multi-phase with iterative planning (user modifies criteria before research)

**Critical Observations:**
- **User can modify research plan** before investigation starts
- **PM adapts to user priorities** - adjusts evaluation criteria, adds/removes options
- **Pixi.js added to investigation** per user request
- **Evaluation criteria re-weighted** - performance critical, bundle deprioritized
- **4 parallel investigations** instead of 3 (user-requested scope expansion)
- **PM re-evaluates with new criteria** - different scoring, potentially different recommendation
- **Iterative planning** - PM → User feedback → PM adjusts → User approves → Execute

**When user might modify plan:**
- Add more options to investigate
- Change evaluation criteria priorities
- Remove options from investigation
- Specify different technical constraints

**Key Point:** Research plan is **collaborative** - PM proposes, user refines, PM adjusts, user approves.

---

### Scenario 7 Flow Diagram (Horizontal ASCII)

```
┌──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────┐
│ USER REQUEST: "Add 3D visualization"                                                                                        │
└────────────────────────┬─────────────────────────────────────────────────────────────────────────────────────────────────────┘
                         ▼
[Main Thread: Route to PM] ──→ [PM: Diagnose Stack Limitation] ──→ [PM: Glob tech stack, Read technology-stack.md]
                                                                                    ▼
                                          ┌─────────────────────────────────────────────────────────┐
                                          │ PM Analysis: Konva.js = 2D only                         │
                                          │              User needs true 3D                         │
                                          │              GAP FOUND                                  │
                                          └───────────────────────┬─────────────────────────────────┘
                                                                  ▼
                                          ┌─────────────────────────────────────────────────────────┐
                                          │ PM: Create Research Plan                                │
                                          │  - Options: Three.js, Babylon.js, WebGL                 │
                                          │  - Criteria: bundle, perf, integration                  │
                                          │  - Design: Parallel execution                           │
                                          └───────────────────────┬─────────────────────────────────┘
                                                                  ▼
                                          ┌─────────────────────────────────────────────────────────┐
                                          │ PM Returns: RESEARCH_NEEDED                             │
                                          │  + Limitation diagnosis                                 │
                                          │  + Evaluation criteria                                  │
                                          │  + Investigation scope                                  │
                                          └───────────────────────┬─────────────────────────────────┘
                                                                  ▼
                                          ┌─────────────────────────────────────────────────────────┐
                                          │ Main Thread: Present to User                            │
                                          └───────────────────────┬─────────────────────────────────┘
                                                                  ▼
                                          ┌─────────────────────────────────────────────────────────┐
                                          │ ★ USER DECISION 1: Proceed with research?              │
                                          └───┬─────────────────────────┬──────────────────┬────────┘
                                              │                         │                  │
                                         A: Approve                B: Modify          C: Reject
                                              │                         │                  │
                                              │                         ▼                  ▼
                                              │         ┌────────────────────────┐   [STOP: Use
                                              │         │ PM: Adjust Plan        │    current stack]
                                              │         └──────┬─────────────────┘
                                              │                ▼
                                              │         [Main Thread: Present Adjusted Plan]
                                              │                ▼
                                              │         [User: Approve adjusted plan?] ──Yes──┐
                                              │                │No                            │
                                              │                └────→ [STOP: Use current]     │
                                              │                                               │
                                              └───────────────────────────────────────────────┘
                                                                  ▼
                                     ┌────────────────────────────────────────────────────────┐
                                     │ Main Thread: Execute Parallel Research                 │
                                     │ Single Message, 3 Tasks (Parallel)                     │
                                     └─────┬──────────────────┬──────────────────┬────────────┘
                                           ▼                  ▼                  ▼
                      ┌─────────────────────────┐  ┌──────────────────────┐  ┌────────────────────────┐
                      │ general-purpose #1:     │  │ general-purpose #2:  │  │ general-purpose #3:    │
                      │ Research Three.js       │  │ Research Babylon.js  │  │ Research Custom WebGL  │
                      │   ↓                     │  │   ↓                  │  │   ↓                    │
                      │ WebSearch + WebFetch    │  │ WebSearch + WebFetch │  │ WebSearch + WebFetch   │
                      │ Three.js docs           │  │ Babylon.js docs      │  │ WebGL resources        │
                      └─────────┬───────────────┘  └──────────┬───────────┘  └────────┬───────────────┘
                                └──────────────────────────────┴──────────────────────┘
                                                               ▼
                                             ┌─────────────────────────────────────────────┐
                                             │ Main Thread: Collect Results                │
                                             └───────────────────┬─────────────────────────┘
                                                                 ▼
                                             ┌─────────────────────────────────────────────┐
                                             │ Main Thread: Organize Findings              │
                                             │ Write to ideas&external_references/         │
                                             │  - threejs-research.md                      │
                                             │  - babylonjs-research.md                    │
                                             │  - custom-webgl-research.md                 │
                                             └───────────────────┬─────────────────────────┘
                                                                 ▼
[Main Thread: Query Analyzer] ──→ [infinite-canvas-analyzer: Assess integration] ──→ [Return: Three.js=Medium, Babylon.js=High]
                                                                 ▼
                                             ┌─────────────────────────────────────────────┐
                                             │ Main Thread: Return to PM                   │
                                             └───────────────────┬─────────────────────────┘
                                                                 ▼
                    ┌────────────────────────────────────────────────────────────────────────────────────────┐
                    │ PM: Evaluate Research Results                                                          │
                    │  ↓                                                                                     │
                    │ PM: Read all research docs (threejs-research.md, babylonjs-research.md, webgl-...)    │
                    │  ↓                                                                                     │
                    │ PM: Score Against Criteria                                                            │
                    │  - Three.js: 4.2/5                                                                    │
                    │  - Babylon.js: 3.1/5                                                                  │
                    │  - Custom WebGL: 2.5/5                                                                │
                    │  ↓                                                                                     │
                    │ PM Returns: Recommend Three.js (WITHOUT updating stack yet)                           │
                    │  + Detailed rationale + Scoring breakdown                                             │
                    └────────────────────────────────────────┬───────────────────────────────────────────────┘
                                                             ▼
                                     ┌───────────────────────────────────────────────────────┐
                                     │ Main Thread: Present Recommendation                   │
                                     └─────────────────────┬─────────────────────────────────┘
                                                           ▼
                                     ┌───────────────────────────────────────────────────────┐
                                     │ ★ USER DECISION 2: Accept Three.js?                   │
                                     └──┬──────────────┬──────────────┬──────────────┬───────┘
                                        │              │              │              │
                                   A: Approve    B: Babylon.js  C: More Research D: Reject All
                                        │              │              │              │
                                        │              │              │              ▼
                                        │              │              │         [STOP: No new tech]
                                        │              │              │
                                        │              ▼              └──────→ [Return to Parallel Research]
                                        │    ┌────────────────────────┐
                                        │    │ PM: Re-evaluate with   │
                                        │    │ Babylon.js             │
                                        │    │ Document user rationale│
                                        │    │ Adjust strategy        │
                                        │    └──────────┬─────────────┘
                                        │               │
                                        └───────────────┘
                                                        ▼
                           ┌────────────────────────────────────────────────────────────────┐
                           │ PM: Finalize Adoption                                          │
                           │  ↓                                                             │
                           │ PM: Update technology-stack.md                                 │
                           │ Create Integration PRD                                         │
                           └────────────────────────┬───────────────────────────────────────┘
                                                    ▼
                           ┌────────────────────────────────────────────────────────────────┐
                           │ Main Thread: Report to User                                    │
                           │  ↓                                                             │
                           │ Main Thread: Query Framework Agents                            │
                           │  ├──→ theia-analyzer: Webview integration                      │
                           │  └──→ electron-analyzer: Security patterns                     │
                           │  ↓                                                             │
                           │ Main Thread: Synthesize                                        │
                           │  ↓                                                             │
                           │ coding-agent: Implement Integration                            │
                           │  ↓                                                             │
                           │ Complete: 3D Viz Integrated                                    │
                           └────────────────────────────────────────────────────────────────┘
```

**Legend:**
- 🔵 Blue: User actions / Start
- 🔴 Red border: **User Decision Points** (critical gates)
- 🟡 Yellow: PM agent actions
- 🟢 Green: Main Thread orchestration
- 🟣 Purple: Research/Analyzer agents
- 🔴 Pink: coding-agent (implementation)
- ⚪ Gray: End states

**Key Decision Gates:**
1. **User Decision 1** (After PM diagnosis): Proceed with research? Modify? Stop?
2. **User Decision 2** (After PM evaluation): Accept recommendation? Choose different? More research?

**Parallel Execution Shown:**
- Single message spawns 3 research agents simultaneously
- Faster runtime (3x speedup vs sequential)

---

## Comparison Matrix: When to Use What

```
┌──────────────────────┬───────────────────────────────────┬───────┬───────────────┬──────────┐
│ Request Type         │ Route                             │ Agents│ Who Codes     │ Conflict │
├──────────────────────┼───────────────────────────────────┼───────┼───────────────┼──────────┤
│ Simple Question      │ Direct execution                  │ 0     │ Nobody        │ N/A      │
│                      │                                   │       │               │          │
│ Example: "List files", "Show git status"                                                   │
├──────────────────────┼───────────────────────────────────┼───────┼───────────────┼──────────┤
│ File Edit            │ MT → coding-agent                 │ 1     │ coding-agent  │ N/A      │
│                      │                                   │       │               │          │
│ Example: "Fix typo", "Update version"                                                      │
├──────────────────────┼───────────────────────────────────┼───────┼───────────────┼──────────┤
│ Framework Question   │ Direct to framework agent         │ 1     │ Nobody        │ N/A      │
│                      │                                   │       │               │          │
│ Example: "How does Theia DI work?"                                                         │
├──────────────────────┼───────────────────────────────────┼───────┼───────────────┼──────────┤
│ Strategic Decision   │ MT orchestrates → PM agent        │ 1     │ Nobody        │ N/A      │
│                      │                                   │       │               │          │
│ Example: "Should we add feature X?"                                                        │
├──────────────────────┼───────────────────────────────────┼───────┼───────────────┼──────────┤
│ Theia Framework Impl │ MT → analyzers → coding-agent     │ 2-3   │ coding-agent  │ ❌ No    │
│                      │                                   │       │               │ (Theia)  │
│ Example: "Modify Theia terminal service"                                                   │
├──────────────────────┼───────────────────────────────────┼───────┼───────────────┼──────────┤
│ EG-DESK Custom       │ MT → analyzers → coding-agent     │ 2-3   │ coding-agent  │ ✅ Yes   │
│ Feature              │                                   │       │               │ (STRUCT) │
│ Example: "Add custom QuickSearch with Ctrl+K"                                              │
├──────────────────────┼───────────────────────────────────┼───────┼───────────────┼──────────┤
│ Large Multi-Feature  │ MT → analyzers + coding-agent(s)  │ 3-4+  │ coding-agents │ ✅ If    │
│                      │                                   │       │               │ EG-DESK  │
│ Example: "Implement custom dashboard system"                                               │
├──────────────────────┼───────────────────────────────────┼───────┼───────────────┼──────────┤
│ New Tech Research    │ PM diagnoses → User approves →    │ 8+    │ coding-agent  │ N/A      │
│ (happy path)         │ MT (parallel) → PM evaluates →    │       │ (after appr)  │          │
│                      │ User approves → PM finalizes      │       │               │          │
│ Example: "Add 3D viz - which framework?" (7a)                                              │
├──────────────────────┼───────────────────────────────────┼───────┼───────────────┼──────────┤
│ New Tech Research    │ Same as above but user rejects PM │ 8+    │ coding-agent  │ N/A      │
│ (user override)      │ recom → PM re-evaluates with user │       │ (after choice)│          │
│                      │ choice                            │       │               │          │
│ Example: User: "No, use Babylon.js" (7b)                                                   │
├──────────────────────┼───────────────────────────────────┼───────┼───────────────┼──────────┤
│ New Tech Research    │ PM proposes → User modifies       │ 9+    │ coding-agent  │ N/A      │
│ (iterative)          │ criteria → PM adjusts → User      │       │ (after appr)  │          │
│                      │ approves → MT investigates        │       │               │          │
│ Example: User: "Add Pixi.js, change criteria" (7c)                                         │
├──────────────────────┼───────────────────────────────────┼───────┼───────────────┼──────────┤
│ Agent Creation       │ MT → claude-agent                 │ 1     │ claude-agent  │ N/A      │
│                      │                                   │       │ (agent file)  │          │
│ Example: "Create Konva analyzer agent"                                                     │
└──────────────────────┴───────────────────────────────────┴───────┴───────────────┴──────────┘
```

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

```
┌─────────────────┬───────────────────────────────┬──────────────────────────────────┐
│ Entity          │ Tool Access                   │ Usage Restriction (Contextual)   │
├─────────────────┼───────────────────────────────┼──────────────────────────────────┤
│ Analyzer Agents │ Bash, Glob, Grep, Read,       │ Bash for READ-ONLY analysis      │
│                 │ WebFetch, WebSearch           │ (inspect outputs, run tests to   │
│                 │                               │ understand behavior). NEVER for  │
│                 │                               │ implementation (commits, builds, │
│                 │                               │ installations). Role enforced    │
│                 │                               │ via agent prompt.                │
├─────────────────┼───────────────────────────────┼──────────────────────────────────┤
│ coding-agent    │ Write, Edit, Read, Glob, Grep │ Code execution only. NO Bash     │
│                 │                               │ (no builds/tests/commits).       │
├─────────────────┼───────────────────────────────┼──────────────────────────────────┤
│ Main Thread     │ ALL tools                     │ Full access: Orchestration,      │
│                 │                               │ implementation, builds, commits, │
│                 │                               │ everything                       │
├─────────────────┼───────────────────────────────┼──────────────────────────────────┤
│ Task tool       │ Main Thread ONLY              │ Agent invocation exclusive to    │
│                 │                               │ Main Thread.                     │
└─────────────────┴───────────────────────────────┴──────────────────────────────────┘
```

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

### 6. Multi-turn Communication with PM Agent

**PRINCIPLE**: Main Thread may return to PM Agent for additional guidance after initial strategic guide, especially for complex implementations requiring plan review, clarification, or progressive phasing.

**When to Return to PM**:

✅ **Return for**:
- **Plan Review**: Created execution plan, want PM validation before implementing
- **Clarification**: PM's guide ambiguous (desktop vs web? custom vs framework modification?)
- **Progressive Phases**: Phase N complete, need guidance for Phase N+1 based on findings
- **Decision Support**: Framework agents suggest multiple approaches, need vision-aligned choice
- **Conflict Resolution**: Found existing feature, unclear whether to enhance/replace/separate

❌ **Don't return for**:
- Simple implementation details (ask framework agents)
- Following PM's guide exactly as provided (just execute)
- Minor code adjustments (handle directly)
- Pure technical patterns (query framework agents)

**Multi-turn Pattern**:

```
Turn 1: Initial Strategic Guide
Main Thread → PM: "User wants [feature]. Provide strategic guide."
PM → Main Thread: Strategic guide (framework, location, phasing)

Turn 2: Plan Review (optional, if complex)
Main Thread → PM: "I created this plan based on your guide:
[PLAN]

Framework agents reported:
[FINDINGS]

Review against vision?"
PM → Main Thread: Plan assessment (PROCEED / REVISE / CONSULT USER)

Turn 3: Implementation
Main Thread executes (or revises plan based on PM feedback)
```

**Key Principle: Include Full Context**

Since PM Agent is stateless, Main Thread must include previous guidance:

```
Task(agent: "egdesk-pm-agent",
     prompt: "Previously you provided this strategic guide:

[QUOTE ENTIRE PREVIOUS GUIDE]

I've now completed Phase 1 and found:
- [Finding 1]
- [Finding 2]

Framework agents reported:
- [Key insight]

My plan for Phase 2: [PLAN]

Questions:
1. Does this align with vision given findings?
2. Should I adjust based on agent reports?

Provide guidance for Phase 2.")
```

**Multi-turn Patterns Supported**:

1. **Plan Review**: Validate execution plan against vision
2. **Clarification**: Resolve ambiguity in strategic guide
3. **Progressive Phases**: Get next-phase guidance after completing previous phase
4. **Decision Support**: Choose between multiple valid approaches
5. **Conflict Resolution**: Resolve tension between found implementations and new requirements
6. **Research Planning**: PM diagnoses stack limitation, provides research plan for Main Thread to execute
7. **Research Results Evaluation**: Main Thread returns with findings, PM evaluates and recommends

**Benefits**:
- Complex implementations get vision alignment at multiple checkpoints
- Reduces risk of implementing wrong approach
- Adapts strategy based on discoveries during execution
- User makes informed decisions at critical junctures

**Smart Judgment**:
- Trust Main Thread to decide when PM consultation adds value
- Don't over-consult (slows execution)
- Do consult strategically (prevents costly rework)

**Example: Plan Review Turn**

```
┌──────┬─────────────────┬──────────────────────────────────────────────────────────────┐
│ Step │ Entity          │ Action → Output                                              │
├──────┼─────────────────┼──────────────────────────────────────────────────────────────┤
│ 8a   │ Main Thread     │ Created plan, wants PM review                                │
│      │                 │ → Plan: [3-phase implementation]                             │
├──────┼─────────────────┼──────────────────────────────────────────────────────────────┤
│ 8b   │ Main Thread     │ Returns to PM                                                │
│      │                 │ → Task(agent: "egdesk-pm-agent",                             │
│      │                 │   prompt: "Previously you said [GUIDE].                      │
│      │                 │   I created this plan: [PLAN]. Framework                     │
│      │                 │   agents found [FINDINGS]. Review?")                         │
├──────┼─────────────────┼──────────────────────────────────────────────────────────────┤
│ 8c   │ egdesk-pm-agent │ Reviews plan                                                 │
│      │                 │ → Assessment: "Add Phase 2.5 for preference persistence"     │
├──────┼─────────────────┼──────────────────────────────────────────────────────────────┤
│ 8d   │ Main Thread     │ Revises plan                                                 │
│      │                 │ → Updated plan with PM suggestions                           │
├──────┼─────────────────┼──────────────────────────────────────────────────────────────┤
│ 8e   │ Main Thread     │ Proceeds to framework investigation                          │
│      │                 │ → Executes revised plan                                      │
└──────┴─────────────────┴──────────────────────────────────────────────────────────────┘
```

This pattern allows Main Thread to "course-correct" after gathering technical findings but before committing to implementation.

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
               └─ Phase 3: coding-agent implements (Main Thread commits)
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

### Pattern 6: Conflict Detection & Resolution (EG-DESK Custom Features)
```
User → Main Thread orchestrates:
         ├─ PM provides strategic guide (EG-DESK custom feature)
         ├─ Framework agents provide patterns
         │
         └─ Spawns coding-agent with conflict check instruction:
               │
               ├─ Step 1: Discover EG-DESK codebase (Glob eg-desk*/**/*.ts)
               ├─ Step 2: Find structure document (Glob eg-desk*/CODEBASE_STRUCTURE.md)
               ├─ Step 3: Read structure document
               ├─ Step 4: Check for conflicts (service names, keybindings, etc.)
               │
               ├─ IF CONFLICT DETECTED:
               │   ├─ STOP immediately (don't implement)
               │   ├─ Report to Main Thread:
               │   │   - What conflicts
               │   │   - Where existing
               │   │   - Alternatives suggested
               │   └─ Main Thread → User decision
               │       └─ User chooses alternative
               │           └─ Main Thread retries with resolution
               │
               └─ IF NO CONFLICT:
                   ├─ Implement code
                   ├─ Update CODEBASE_STRUCTURE.md
                   └─ Return success + structure updated

Benefit: Prevents duplicate implementations and naming conflicts
When to use: EG-DESK custom features (keybindings, services, commands)
When to skip: Theia framework modifications (packages/*), bug fixes
```

### Pattern 7: Optional UX Flow Validation (Pre-Build)
```
User → Main Thread orchestrates:
         ├─ PM provides strategic guide
         ├─ Framework agents provide patterns
         ├─ coding-agent implements
         │
         └─ (Optional) Main Thread validates flow:
               │
               ├─ Decision: Is this complex/critical?
               │   - Complex user flows? → Validate
               │   - Race conditions possible? → Validate
               │   - Critical feature? → Validate
               │   - Simple CRUD? → Skip
               │
               └─ IF VALIDATE:
                   ├─ Invoke ux-flow-simulator-agent:
                   │   - "Trace execution: [user action] → [expected result]"
                   │   - "Files: [implemented files]"
                   │   - "Check for: race conditions, runtime errors, UX issues"
                   │
                   ├─ ux-flow-simulator traces code execution
                   ├─ Predicts runtime behavior
                   ├─ Identifies potential issues BEFORE build
                   │
                   └─ Returns validation:
                       - ✅ No issues → Proceed to build
                       - ⚠️ Issues found → Fix via coding-agent → Re-validate

Benefit: Catch runtime errors, race conditions, UX issues before build/test
When to use: Complex flows, async operations, state management, critical features
When to skip: Simple changes, pure styling, documentation, obvious correctness
```

### Pattern 8: New Technology Research & Evaluation (Multi-Option Investigation with User Decisions)
```
User → Main Thread orchestrates:
         │
         ├─ Step 1: PM diagnoses limitation
         │   Main Thread → PM: "User wants [capability]. Current stack support?"
         │   PM analyzes: Current tech X has limitation Y for requirement Z
         │   PM returns: RESEARCH_NEEDED
         │     - Limitation diagnosis (WHY research needed)
         │     - Evaluation criteria (bundle, integration, performance)
         │     - Investigation scope (Option A, B, C)
         │     - Parallel execution design (all independent)
         │
         ├─ Step 2: USER DECISION POINT 1 (Proceed with research?)
         │   Main Thread → User: "PM diagnosed limitation. Research plan: [...]"
         │   User → OPTIONS:
         │     A) Approve plan → Continue to Step 3
         │     B) Modify plan → PM adjusts (add options, change criteria) → User approves → Step 3
         │     C) Reject research → Stop (use current stack workaround)
         │
         ├─ Step 3: Main Thread investigates (PARALLEL - if user approved)
         │   Single message, 3 Tasks:
         │   ├─ Task(agent: "general-purpose", "Research Three.js...")
         │   ├─ Task(agent: "general-purpose", "Research Babylon.js...")
         │   └─ Task(agent: "general-purpose", "Research Custom WebGL...")
         │   → All 3 run simultaneously (faster runtime)
         │
         ├─ Step 4: Main Thread organizes findings
         │   ├─ Write("ideas&external_references/threejs-research.md")
         │   ├─ Write("ideas&external_references/babylonjs-research.md")
         │   └─ Write("ideas&external_references/custom-webgl-research.md")
         │   → Institutional memory preserved
         │
         ├─ Step 5: Main Thread queries analyzers
         │   Task(agent: "infinite-canvas-analyzer-agent",
         │        "Read research docs, assess integration complexity")
         │   → Technical analysis of integration
         │
         ├─ Step 6: PM evaluates results
         │   Main Thread → PM: "Research complete. Files: [docs]. Analyzer: [findings]. Evaluate."
         │   PM reads research docs
         │   PM scores against criteria
         │   PM recommends: Option X (with detailed rationale)
         │   PM returns evaluation (WITHOUT updating stack yet - waits for user)
         │   → Vision-aligned recommendation
         │
         ├─ Step 7: USER DECISION POINT 2 (Accept recommendation?)
         │   Main Thread → User: "PM recommends [Option X]. Scoring: [X:4.2, Y:3.1, Z:2.5]"
         │   User → OPTIONS:
         │     A) Approve recommendation → PM finalizes (updates stack + PRD) → Implementation
         │     B) Choose different option → PM re-evaluates with user choice → PM finalizes → Implementation
         │     C) Request more research → Add options → Return to Step 3
         │     D) Reject all → Stop
         │
         └─ Step 8: PM finalizes adoption (if user approved)
             PM updates: technology-stack.md (with user-approved choice)
             PM creates: Integration PRD (documents decision rationale)
             → Ready for implementation (follow Pattern 2)

Benefit:
- Parallel research (3x faster than sequential)
- Evidence-based decision (not assumptions)
- **User control at 2 decision points** (research approval + recommendation approval)
- **Iterative refinement** (user can modify criteria, add options, choose alternatives)
- Institutional memory (research preserved with user decision rationale)
- Technology stack evolution (documented with WHY user chose X over Y)

When to use:
- User needs capability not in current stack
- Current stack has limitations
- Multiple technology options exist
- Need comparative analysis

User Decision Points:
1. **After PM diagnosis**: Proceed with research? Modify criteria? Stop?
2. **After PM evaluation**: Accept recommendation? Choose different? More research?

Variants:
- **Scenario 7a**: User approves both decision points (happy path)
- **Scenario 7b**: User rejects PM recommendation, chooses alternative
- **Scenario 7c**: User modifies research plan before investigation

Flow: PM diagnoses → **User approves** → MT investigates (parallel) → MT organizes → MT analyzes → PM evaluates → **User approves** → PM finalizes
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

### ❌ Anti-Pattern 5: Main Thread Writing Code Directly
```
BAD:
User asks: "Fix typo in README"
Main Thread → Edit("README.md", ...) directly
(Breaks separation of concerns - Main Thread should NEVER edit application files)

GOOD:
User asks: "Fix typo in README"
Main Thread → coding-agent → Edit README
(Consistent delegation pattern)

Why always use coding-agent:
✅ Consistent separation of concerns
✅ Main Thread stays clean (orchestration only)
✅ Single responsibility: Main Thread orchestrates, coding-agent executes
✅ Context preservation for Main Thread

Main Thread responsibilities:
✅ Orchestrate (create plans, invoke agents)
✅ Build/test/commit (Bash commands)
✅ Synthesize agent reports
❌ NEVER Write/Edit application files
```

### ❌ Anti-Pattern 6: Hardcoding Paths
```
BAD:
PM agent: "Implement in packages/terminal/src/browser/"
coding-agent: Read("packages/terminal/src/browser/file.ts")
(Assumes fixed structure)

GOOD:
PM agent: Glob("eg-desk*/**/*.ts") → discovers eg-desk_taehwa/
PM agent: "Implement in eg-desk_taehwa/terminal/"
coding-agent: Glob("eg-desk*/CODEBASE_STRUCTURE.md") → discovers dynamically
(Structure-agnostic, flexible)

Why bad:
- Project structure may change
- EG-DESK custom code might move
- Hard to refactor
- Breaks when directory renamed
```

### ❌ Anti-Pattern 7: Not Checking Conflicts for EG-DESK Features
```
BAD:
User: "Bind Ctrl+K to QuickSearch"
coding-agent: Implements immediately without checking
→ Conflict: Ctrl+K already used elsewhere
→ Discovered only after deployment (user confusion)

GOOD:
User: "Bind Ctrl+K to QuickSearch"
coding-agent: Reads CODEBASE_STRUCTURE.md first
→ Finds Ctrl+K conflict
→ Reports to user with alternatives
→ User chooses Ctrl+Shift+K
→ Implements with correct key

Why bad:
- Duplicate keybindings confuse users
- Hard to debug "which feature does Ctrl+K trigger?"
- Wastes time fixing after implementation
- No registry of what's used
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
- ✅ **Conflict Prevention**: EG-DESK custom features checked for naming conflicts before implementation
- ✅ **Dynamic Discovery**: No hardcoded paths, all locations discovered via Glob
- ✅ **Structure Tracking**: CODEBASE_STRUCTURE.md automatically maintained, always up-to-date

---

## Appendix: Roles and Responsibilities Reference

### Comprehensive Capability Matrix

```
┌──────────────────────────┬──────────────────────────────────────────────────────────────────────────────────┬──────────────────────────────────────────────────────┬────────────────────────────────────────┐
│ Entity                   │ CAN Do                                                                           │ CANNOT Do                                            │ Tools Available                        │
├──────────────────────────┼──────────────────────────────────────────────────────────────────────────────────┼──────────────────────────────────────────────────────┼────────────────────────────────────────┤
│ Main Conversation Thread │ • Analyze/route requests                                                         │ • Write/Edit application code (delegate to coding)   │ Bash, Task, Read, Glob, Grep           │
│                          │ • Orchestrate agents: Identify, create prompts, plan phases, design workflows   │ • Read domain files when orchestrating (delegate)   │ (NO Write/Edit for app code)           │
│                          │ • Invoke agents (Task tool), synthesize outputs                                 │ • Delegate simple tasks unnecessarily                │                                        │
│                          │ • Execute bash (build/test/commit), git ops, PRs, install packages              │ • Guess framework patterns without consultation      │                                        │
│                          │ • Make implementation decisions, preserve context by delegating heavy reads      │ • Implement EG-DESK features without vision check    │                                        │
├──────────────────────────┼──────────────────────────────────────────────────────────────────────────────────┼──────────────────────────────────────────────────────┼────────────────────────────────────────┤
│ Specialized Analyzer     │ • Analyze codebase/docs, provide evidence-based guidance                        │ • Write ANY code                                     │ Bash (READ-ONLY), Read, Glob, Grep,    │
│ Agents (theia, electron, │ • Explain framework patterns, troubleshoot, reference files/APIs                │ • Use Bash for implementation (commits, builds)      │ WebFetch, WebSearch                    │
│ infinite-canvas, etc.)   │ • Find proven patterns, return detailed reports                                 │ • Create/edit files, commit, invoke other agents     │ (Bash contextually enforced)           │
│                          │ • Use Bash for READ-ONLY analysis (inspect outputs, run tests to understand)    │ • Implement features                                 │                                        │
├──────────────────────────┼──────────────────────────────────────────────────────────────────────────────────┼──────────────────────────────────────────────────────┼────────────────────────────────────────┤
│ egdesk-pm-agent          │ • Strategic PM: implementation guide (tech stack, location, phasing)            │ • Write app code (only writes docs: PRDs, vision,   │ Bash (discovery only), Read, Write,    │
│                          │ • Tech Stack Discovery: Glob + Read tech-stack.md (NEVER hardcode)              │   tech-stack.md, research in ideas&ext_refs/)        │ Edit (docs only), Glob, Grep,          │
│                          │ • Tech Stack Selection: match requirements to capabilities                       │ • Execute implementation commands                    │ WebFetch, WebSearch                    │
│                          │ • Tech Gap Assessment: diagnose stack limitations                                │ • Commit changes, invoke other agents                │                                        │
│                          │ • Research Planning: create parallel investigation plans                         │ • Provide technical patterns (framework agents do)   │                                        │
│                          │ • Evaluation Criteria: specify criteria for MT to evaluate tech options          │ • Hardcode technology stack                          │                                        │
│                          │ • Research Results Evaluation: assess findings, recommend vision-aligned choice  │ • Do tech research directly (PM plans, MT executes)  │                                        │
│                          │ • Tech Stack Management: update tech-stack.md when new tech added                │                                                      │                                        │
│                          │ • Implementation Status Check: Grep/Glob verify no duplicates                    │                                                      │                                        │
│                          │ • Plan Reviewer: validate MT's plans, suggest improvements                       │                                                      │                                        │
│                          │ • Documentation Manager: create PRDs, update vision/ideas/tech-stack docs        │                                                      │                                        │
│                          │ • Institutional Memory: recall previous decisions, record new ones               │                                                      │                                        │
│                          │ • Insight Provider: explain vision conflicts, suggest alternatives               │                                                      │                                        │
│                          │ • Dynamic Discovery: Glob all docs dynamically (vision, tech, structure)         │                                                      │                                        │
│                          │ • Code Location: specify exact package/dir (eg-desk_taehwa/ vs packages/)       │                                                      │                                        │
│                          │ • Preserve MT's context: read vision/tech docs, synthesize strategic direction   │                                                      │                                        │
├──────────────────────────┼──────────────────────────────────────────────────────────────────────────────────┼──────────────────────────────────────────────────────┼────────────────────────────────────────┤
│ coding-agent             │ • Execute code writing/editing based on MT instructions                          │ • Make architectural decisions                       │ Write, Edit, Read, Glob, Grep          │
│                          │ • Create new files following provided patterns                                  │ • Choose implementation approaches                   │ (NO Bash)                              │
│                          │ • Edit existing files precisely, follow framework patterns from analyzer         │ • Auto-resolve conflicts (must report, user decides) │                                        │
│                          │ • Handle multi-file implementations                                              │ • Hardcode paths (always discover dynamically)       │                                        │
│                          │ • Discover EG-DESK codebase dynamically (Glob eg-desk*/**/*.ts)                  │ • Execute bash, run builds/tests, commit changes     │                                        │
│                          │ • Check for naming conflicts before implementing (CODEBASE_STRUCTURE.md)        │ • Invoke other agents, analyze frameworks            │                                        │
│                          │ • STOP immediately when conflict detected (report to MT, user decides)           │ • Validate vision                                    │                                        │
│                          │ • Update structure document after successful implementation                      │                                                      │                                        │
│                          │ • Prevent duplicate implementations (conflict prevention system)                 │                                                      │                                        │
│                          │ • Return implementation status reports                                           │                                                      │                                        │
├──────────────────────────┼──────────────────────────────────────────────────────────────────────────────────┼──────────────────────────────────────────────────────┼────────────────────────────────────────┤
│ claude-agent-sdk-        │ • Subagent Architect: design and create new specialized agents                  │ • Write application code (only writes agent files)   │ Bash, Read, Write (agent files only),  │
│ analyzer-agent           │ • Read best practices from subagent-best-practices.md                            │ • Execute commands, commit changes                   │ Glob, Grep, WebFetch, WebSearch        │
│                          │ • Examine existing agents to extract proven patterns                             │ • Invoke other agents                                │                                        │
│                          │ • Write agent definition files (.claude/agents/*.md)                             │                                                      │                                        │
│                          │ • SDK Integration Guidance: explain Claude Code SDK patterns                     │                                                      │                                        │
│                          │ • Guide SDK feature implementation into forked apps                              │                                                      │                                        │
│                          │ • Troubleshoot SDK usage issues                                                  │                                                      │                                        │
├──────────────────────────┼──────────────────────────────────────────────────────────────────────────────────┼──────────────────────────────────────────────────────┼────────────────────────────────────────┤
│ User                     │ • Make final decisions at decision points                                        │ (User has full authority)                            │ None (human input)                     │
│                          │ • Provide preferences, validate experimental results                             │                                                      │                                        │
│                          │ • Approve/reject architectural plans, request clarifications                     │                                                      │                                        │
│                          │ • Override any recommendation                                                    │                                                      │                                        │
└──────────────────────────┴──────────────────────────────────────────────────────────────────────────────────┴──────────────────────────────────────────────────────┴────────────────────────────────────────┘
```

### Code Ownership Hierarchy

**Main Thread NEVER writes code** - ALWAYS delegates ALL file changes to coding-agent.

```
┌──────────────────────────────────────────────────────────┐
│   Main Thread (Orchestrator ONLY)                        │
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
│   Code delegation (ALWAYS):                              │
│   └─ ALL file changes → coding-agent                     │
│                                                           │
│   NEVER:                                                 │
│   • Write/Edit application files directly                │
│                                                           │
│   Receives guidance from:                                │
│   ├─ Framework analyzers (patterns)                      │
│   └─ egdesk-pm-agent (vision alignment)                 │
└───────────┬───────────────────────┬──────────────────────┘
            │                       │
    (guidance only)          (implementation ONLY)
            │                       │
┌───────────▼─────────┐    ┌────────▼───────────────┐
│ Framework Analyzers │    │ coding-agent           │
│ (Explain patterns)  │    │ (ONLY entity that      │
│ Returns guidance    │    │  writes code)          │
│ with file refs      │    │                        │
└─────────────────────┘    │ Receives:              │
                           │ • Detailed impl plan   │
┌─────────────────────┐    │ • Agent guidance       │
│ PM Agent            │    │ • Patterns to follow   │
│ (Validates vision)  │    │                        │
│ Returns             │    │ Executes:              │
│ APPROVE/REJECT      │    │ • Write new files      │
└─────────────────────┘    │ • Edit existing files  │
                           │ • Follow patterns      │
                           │                        │
                           │ Returns:               │
                           │ • Status report        │
                           └────────────────────────┘
```

**Key Points:**
1. **Main Thread**: NEVER writes code, ALWAYS delegates to coding-agent
2. **coding-agent**: ONLY entity that writes application code
3. **Analyzer agents**: Never touch code
4. **Only Main Thread**: Has Bash (build, test, commit)
5. **Clear separation**: Main Thread orchestrates, coding-agent executes

### Orchestration Hierarchy

**Main Thread orchestrates everything:**
- Creates mission prompts
- Invokes agents
- Synthesizes results
- Delegates implementation to coding-agent
- Builds, tests, commits

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
         │  - Phase 3: Implementation via coding-agent
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
         ├─ Synthesizes all guidance (PM guide + framework patterns)
         │
         ├─ Executes Phase 3: Delegates to coding-agent
         │  └─ Task(agent: "coding-agent",
         │          prompt: "Direction: [what to implement]
         │                   Files: [CREATE/MODIFY/DELETE/REFERENCE]
         │                   [You will read files for details]")
         │
         └─ After coding-agent completes:
            ├─ Bash(build, test, commit)
            └─ Reports to user
```

**Critical Points:**
1. **Main Thread**: Orchestrates, creates prompts, invokes agents, NEVER writes code
2. **Agents**: Analyze and return guidance only
3. **coding-agent**: ONLY entity that writes code
4. **User**: Makes decisions at decision points (분기점)

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
- **Controls all implementation**:
  - Delegates ALL file changes to coding-agent (direction + file list)
  - NEVER writes application code directly
  - Retains control of build/test/commit (exclusive Bash for implementation)

**egdesk-pm-agent (Strategic PM & Administrative Orchestrator):**
- **Discovers technology stack dynamically**: Reads technology-stack.md (NEVER hardcodes tech list)
- **Provides strategic guide**: Technology stack selection, code location, implementation phasing
- **Diagnoses stack limitations**: Identifies when current stack cannot meet requirements
- **Plans technology research**: Creates detailed, parallel investigation plans with evaluation criteria
- **Evaluates research results**: Assesses Main Thread's findings, recommends vision-aligned choice
- **Checks implementation status**: Prevents duplicate implementations (Grep EG-DESK + Theia code)
- **Reviews execution plans**: Validates completeness, suggests improvements
- **Manages documentation**: Creates PRDs, updates vision docs, updates technology-stack.md, preserves research docs
- **Maintains institutional memory**: Recalls decisions, records new ones
- **Provides insights**: Explains vision conflicts, suggests alternatives to user
- **Dynamically discovers structure**: Globs vision docs + tech stack + codebase to understand project state
- **Never writes application code** (only documentation)
- **Never does research directly** (plans research, Main Thread executes, PM evaluates)

**Specialized Analyzer Agents (Technical Experts):**
- Analyze codebases and documentation in their domains
- Provide technical patterns and file references
- Return evidence-based guidance with file lists (CREATE/MODIFY/DELETE/REFERENCE)
- Never write code or make strategic decisions

**coding-agent (Code Executor & Structure Manager):**
- Executes code writing/editing based on Main Thread's direction + file list
- Reads files for implementation details
- Follows patterns from framework analyzers
- **Discovers EG-DESK codebase dynamically** (never hardcodes paths)
- **Checks for conflicts before implementing** EG-DESK custom features
- **Stops immediately when conflict detected** (reports to Main Thread, user decides)
- **Updates CODEBASE_STRUCTURE.md automatically** after implementation
- **Prevents duplicate implementations** via conflict detection system
- Returns status reports
- **Preserves Main Thread's context** for continued communication

**Architectural Principles:**
1. **Strategic Level** (PM Agent): Guides what to build, where, and how - manages vision alignment
2. **Communication Level** (Main Thread): Facilitates between user, PM, and technical agents - executes plans
3. **Technical Level** (Analyzer Agents): Provides framework-specific patterns and guidance
4. **Execution Level** (coding-agent): Implements code following all guidance

**Result:** PM-driven strategic flow with Main Thread as facilitator and built-in conflict prevention:
- **PM discovers technology stack dynamically** (reads technology-stack.md, NEVER hardcoded)
- **PM diagnoses stack limitations** (identifies when new technology research needed)
- **PM plans technology research** (detailed criteria + parallel investigation design)
- **Main Thread executes parallel research** (3+ agents simultaneously for faster runtime)
- **Main Thread organizes findings** (preserves in ideas&external_references/ for institutional memory)
- **PM evaluates research results** (vision-aligned recommendation with scoring)
- **PM checks implementation status** (prevents duplicate implementations via Grep)
- PM provides strategic direction (technology stack selection, location, approach)
- PM updates technology-stack.md when new technologies added
- Main Thread creates plans and queries technical agents
- Framework agents provide implementation patterns
- **coding-agent checks for conflicts** (EG-DESK custom features only)
- **User makes decisions** when conflicts detected or new tech proposed
- coding-agent executes code (after conflict resolution)
- **coding-agent updates structure document** (automatic maintenance)
- Main Thread builds, tests, commits
- Clear separation: strategy (PM) → planning (Main Thread) → patterns (analyzers) → conflict check (coding-agent) → execution (coding-agent) → structure update (coding-agent)
- **Dynamic discovery**: All paths AND technology stack discovered via Glob, no hardcoding
- **Conflict prevention**: CODEBASE_STRUCTURE.md prevents duplicate implementations, PM prevents duplicate tech stack
- **Evolving tech stack**: Technology stack grows through research workflow (PM plans → Main Thread investigates → PM evaluates)
