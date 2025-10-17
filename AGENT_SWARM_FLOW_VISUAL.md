# Agent Swarm Flow: Visual Guide (Self-Documenting Mermaid Diagrams)

This document provides **self-contained visual representations** where each Mermaid node includes complete information (tools used, actions, outputs). The diagrams are self-documenting - you don't need external explanations.

For the original table-based version, see [AGENT_SWARM_FLOW.md](./AGENT_SWARM_FLOW.md).

## Color Legend

- 🔵 **Light Blue**: User actions / Start
- 🔴 **Red border (thick)**: **User Decision Points** (critical gates)
- 🟡 **Yellow**: PM agent
- 🟢 **Green**: Main Thread
- 🟣 **Purple**: Analyzer/Research agents
- 🔴 **Pink**: coding-agent
- ⚪ **Gray**: End states

---

## Scenario 1: Simple Question (No Agents)

**User:** "What files are in the packages/ai-chat directory?"

```mermaid
flowchart LR
    User(["User Request:
    'What files in packages/ai-chat?'"])

    --> MT["Main Thread
    Action: Analyze request
    Decision: Simple file listing
    Route: Direct execution"]

    --> Glob["Main Thread
    Tool: Glob
    Pattern: packages/ai-chat/**/*
    Output: File list"]

    --> End(["Complete
    User sees file list

    Agents: 0
    Duration: Immediate"])

    style User fill:#e1f5ff
    style MT fill:#e1ffe1
    style Glob fill:#e1ffe1
    style End fill:#e1ffe1
```

---

## Scenario 2: Framework Question (Direct Agent)

**User:** "How does Theia's dependency injection work?"

```mermaid
flowchart TD
    User(["User Request:
    'How does Theia DI work?'"])

    --> MT1["Main Thread
    Action: Analyze request
    Decision: Theia framework question
    Route: theia-analyzer-agent"]

    --> Invoke["Main Thread
    Tool: Task
    Agent: theia-analyzer-agent
    Prompt: 'Analyze Theia DI system
    in packages/core/'"]

    --> Agent["theia-analyzer-agent
    Tools Used:
    - Read: packages/core/src/common/di.ts
    - Grep: @injectable pattern
    - Glob: **/*-frontend-module.ts
    - Read: Example files

    Actions:
    - Analyzes DI implementation
    - Examines usage examples
    - Extracts patterns

    Output:
    - Detailed explanation
    - File references with line numbers
    - Pattern examples"]

    --> MT2["Main Thread
    Action: Receive report
    Output: Present to user"]

    --> End(["Complete
    User sees explanation

    Agents: 1 (theia-analyzer)
    Duration: Single invocation"])

    style User fill:#e1f5ff
    style MT1 fill:#e1ffe1
    style MT2 fill:#e1ffe1
    style Invoke fill:#e1ffe1
    style Agent fill:#f0e1ff
    style End fill:#e1ffe1
```

---

## Scenario 3a: Vision Conflict → User Accepts Alternative

**User:** "Should we add a floating AI assistant that follows the mouse cursor?"

```mermaid
flowchart TD
    User(["User Request:
    'Add floating AI assistant
    that follows cursor?'"])

    --> MT1["Main Thread
    Action: Analyze request
    Decision: EG-DESK feature
    Route: PM for strategic guide"]

    --> PM1["PM Agent: Analyze Vision
    Tools Used:
    - Glob: ideas&external_references/eg-desk ideas/**/*.md
    - Read: EG-DESK_Whitepaper.md
    - Read: EG-DESK_Spatial_Canvas_UX_Solutions.md
    - Grep: 'spatial', 'proximity', 'floating'

    Findings:
    - Previous decision AGAINST floating UI
    - Reason: Breaks spatial affordances
    - Users lose sense of place

    Decision: REJECT original
    Alternative: Proximity-based AI"]

    --> PMReturn["PM Returns to Main Thread:
    Summary: Floating conflicts with spatial canvas
    Decision: REJECT
    Insight: 'We decided against floating UI
    in UX_Solutions.md because it breaks
    spatial affordances'
    Alternative: Proximity-based AI
    (appears NEAR objects, not following cursor)
    Rationale: Maintains spatial relationships
    Vision-Aligned: ✅"]

    --> MT2["Main Thread
    Action: Present to user"]

    --> UserDec{"User Decision:

    Options:
    A) Accept alternative
       (proximity-based AI)
    B) Insist on original
       (requires vision change)
    C) Cancel feature"}

    UserDec -->|A: Accept| PM2["PM Agent: Guide for Alternative
    Tools:
    - Write: proximity-based-ai-prd.md

    Actions:
    - Creates PRD for alternative
    - Provides implementation guide

    Returns:
    - Decision: APPROVE
    - Feature: Proximity-based AI
    - Framework: Theia + Infinite Canvas
    - Location: eg-desk_taehwa/ai/
    - Next: Follow Pattern 2"]

    UserDec -->|B: Insist| Scenario3b["See Scenario 3b:
    PM re-evaluates
    → User decides maintain/evolve vision"]

    UserDec -->|C: Cancel| End1(["Stop
    Feature canceled

    Agents: 1 (PM only)"])

    PM2 --> MT3["Main Thread
    Action: Proceed to implementation
    Next: Follow Pattern 2 (PM-Driven Dev)"]

    --> End2(["Complete
    Proximity-based AI implemented

    Agents: 2 (PM rejection + PM guide)
    Duration: 2 PM turns + implementation"])

    style User fill:#e1f5ff
    style UserDec fill:#ffe1e1,stroke:#ff0000,stroke-width:3px
    style PM1 fill:#fff4e1
    style PM2 fill:#fff4e1
    style PMReturn fill:#fff4e1
    style MT1 fill:#e1ffe1
    style MT2 fill:#e1ffe1
    style MT3 fill:#e1ffe1
    style End1 fill:#f0f0f0
    style End2 fill:#e1ffe1
    style Scenario3b fill:#e1f0ff
```

**Key Insight:** PM rejection is constructive - provides alternative + rationale

---

## Scenario 3b: User Insists on Original Despite Vision Conflict

**User:** "I understand vision, but floating AI is more intuitive. Vision should evolve."

```mermaid
flowchart TD
    Start(["From Scenario 3a:
    User insists on floating AI
    despite PM rejection"])

    --> PM2["PM Agent: Re-evaluate
    Tools:
    - Read: EG-DESK_Spatial_Canvas_UX_Solutions.md

    Analysis:
    - User rationale: 'Floating more intuitive'
    - Vision rationale: 'Spatial affordances for coherence'
    - Valid priority shift (intuitiveness vs spatial)

    Assessment:
    - User rationale valid
    - Vision prioritizes spatial (different reason)
    - This is STRATEGIC decision (not technical)"]

    --> PMReturn["PM Returns:
    Assessment: User rationale valid BUT
    vision prioritizes spatial affordances

    User Decision Required:
    A) Maintain vision (use proximity-based)
    B) Evolve vision (document rationale,
       update vision doc, proceed with floating)"]

    --> MT1["Main Thread
    Action: Present choice to user"]

    --> UserDec{"User Strategic Decision:

    A) Maintain vision
       → Use proximity-based AI

    B) Evolve vision
       → Update vision docs
       → Proceed with floating AI"}

    UserDec -->|A: Maintain| PM3a["PM: Guide for Proximity-based
    (Same as Scenario 3a)"]

    UserDec -->|B: Evolve| PM3b["PM Agent: Update Vision
    Tools:
    - Edit: EG-DESK_Spatial_Canvas_UX_Solutions.md
      (Add 'Vision Evolution: Floating AI'
       section with date, user rationale,
       tradeoffs accepted)
    - Write: floating-ai-assistant-prd.md

    Actions:
    - Documents vision evolution
    - Records user rationale
    - Notes tradeoffs (spatial disorientation accepted)

    Returns:
    - Decision: APPROVE (vision evolved)
    - Vision Updated: UX_Solutions.md
    - Framework: Theia + Infinite Canvas
    - Location: eg-desk_taehwa/ai/
    - Next: Follow Pattern 2"]

    PM3a --> End1(["Complete
    Proximity-based AI

    Agents: 2 (PM re-eval + PM guide)"])

    PM3b --> MT2["Main Thread
    Action: Proceed to implementation
    Next: Follow Pattern 2"]

    --> End2(["Complete
    Floating AI implemented
    Vision evolution documented

    Agents: 3 (PM reject → re-eval → update)"])

    style Start fill:#e1f5ff
    style UserDec fill:#ffe1e1,stroke:#ff0000,stroke-width:3px
    style PM2 fill:#fff4e1
    style PM3a fill:#fff4e1
    style PM3b fill:#fff4e1
    style PMReturn fill:#fff4e1
    style MT1 fill:#e1ffe1
    style MT2 fill:#e1ffe1
    style End1 fill:#e1ffe1
    style End2 fill:#e1ffe1
```

**Key:** Vision NOT immutable - user can evolve with documented rationale

---

## Scenario 4: Development with PM Strategic Guide

**User:** "Add a custom terminal theme that changes based on time of day"

```mermaid
flowchart LR
    subgraph Phase0["PHASE 0: PM Guide"]
        User(["User: Add time-based<br/>terminal theme"])
        PM["PM: Approve<br/>Framework: Theia<br/>Location: packages/terminal/<br/>Creates PRD"]
        User --> PM
    end

    subgraph Phase1_2["PHASE 1-2: Investigation"]
        FW["theia-analyzer<br/>Theme patterns<br/>DI binding<br/>CREATE+MODIFY list"]
    end

    Phase0 --> Phase1_2

    subgraph Phase3_4["PHASE 3-4: Implementation"]
        Coding["coding-agent<br/>Read patterns<br/>Write+Edit files<br/>1 CREATE, 2 MODIFY"]
    end

    Phase1_2 --> Phase3_4

    subgraph Decision["PHASE 4.5: UX Validation?"]
        Decide{"Validate?<br/>Complex/Critical?"}
    end

    Phase3_4 --> Decision

    subgraph Validation["Optional UX Flow"]
        direction TB
        UXSim["ux-flow-simulator<br/>Trace flow<br/>Predict issues"]
        Check{Issues?}
        Fix["coding-agent<br/>Fix"]
        UXSim --> Check
        Check -->|Yes| Fix
        Fix --> UXSim
    end

    Decide -->|Yes| Validation
    Decide -->|No| Build

    Check -->|No| Build["PHASE 5: Build+Commit<br/>npm run build<br/>git commit"]

    Build --> E(["Complete<br/>Agents: 3-4<br/>Duration: Multi-phase"])

    style User fill:#e1f5ff
    style Decide fill:#ffe1e1,stroke:#ff0000,stroke-width:2px
    style Check fill:#ffe1e1,stroke:#ff0000,stroke-width:2px
    style PM fill:#fff4e1
    style FW fill:#f0e1ff
    style UXSim fill:#f0e1ff
    style Coding fill:#ffe1f0
    style Fix fill:#ffe1f0
    style Build fill:#e1ffe1
    style E fill:#e1ffe1
```

---

## Scenario 4c: Conflict Detection (EG-DESK Custom Code)

**User:** "Bind Ctrl+K to the new QuickSearch feature"

```mermaid
flowchart TD
    User(["User Request:
    'Bind Ctrl+K to QuickSearch'"])

    --> Phases["PHASES 0-2: Same as Scenario 4
    (PM guide → Investigation)

    Result:
    - PM approved
    - Framework patterns collected"]

    --> MT3["PHASE 3: Main Thread
    Implementation Planning

    Plan:
    Direction: Bind Ctrl+K to QuickSearch
    File List: CREATE search-contribution.ts
    in eg-desk_taehwa/search/

    ⚠️ INSTRUCTION: Check CODEBASE_STRUCTURE.md
    for conflicts BEFORE implementing"]

    --> Coding1["coding-agent: Conflict Check

    Step 1 - Discover codebase:
    - Tool: Glob eg-desk*/**/*.ts
    - Found: eg-desk_taehwa/ (custom codebase root)

    Step 2 - Find structure doc:
    - Tool: Glob eg-desk*/CODEBASE_STRUCTURE.md
    - Found: eg-desk_taehwa/CODEBASE_STRUCTURE.md

    Step 3 - Read registry:
    - Tool: Read CODEBASE_STRUCTURE.md

    Step 4 - Check conflict:
    - Tool: Grep 'Ctrl+K' in structure doc"]

    --> ConflictCheck{"coding-agent Check Result:

    Ctrl+K conflict?"}

    ConflictCheck -->|"CONFLICT<br/>FOUND"| Stop["coding-agent: STOP

    Action: DO NOT create any files

    Returns:
    ❌ CONFLICT DETECTED
    Type: Keybinding conflict
    Requested: Ctrl+K for QuickSearch
    Existing: Ctrl+K for DifferentFeature
    Location: search-old.ts:45
    Severity: BLOCKER
    Alternatives: Ctrl+Shift+K, Ctrl+Alt+K, Ctrl+J
    User decision required"]

    ConflictCheck -->|"NO<br/>CONFLICT"| Impl["coding-agent: Implement

    Tools:
    - Write: search-contribution.ts
      (with Ctrl+K binding)
    - Edit: CODEBASE_STRUCTURE.md
      (Add 'Ctrl+K: QuickSearch')

    Returns:
    ✅ Implementation Complete
    Conflict check: Passed
    Structure updated"]

    Stop --> MT4["Main Thread
    Action: Present conflict to user"]

    --> UserDec{"User Decision:

    Choose alternative:
    A) Ctrl+Shift+K
    B) Ctrl+Alt+K
    C) Ctrl+J
    D) Override existing
    E) Different key"}

    UserDec --> MT5["Main Thread
    Action: Retry with user's choice

    Tool: Task(coding-agent,
    'Bind [resolved-key] to QuickSearch')"]

    --> Coding2["coding-agent: Retry

    Step 1: Check new key conflict
    Step 2: If OK → Implement
    Step 3: Update structure doc"]

    Coding2 --> Impl

    Impl --> MT6["Main Thread
    Tools:
    - Bash: npm run build
    - Bash: git add eg-desk_taehwa
    - Bash: git commit"]

    --> End(["Complete
    QuickSearch bound to resolved key
    Structure document updated

    Agents: 3 (PM → analyzer → coding x2)
    Duration: Multi-phase with user decision"])

    style User fill:#e1f5ff
    style ConflictCheck fill:#ffe1e1,stroke:#ff0000,stroke-width:2px
    style UserDec fill:#ffe1e1,stroke:#ff0000,stroke-width:3px
    style Coding1 fill:#ffe1f0
    style Coding2 fill:#ffe1f0
    style Stop fill:#ffe1f0
    style Impl fill:#ffe1f0
    style MT3 fill:#e1ffe1
    style MT4 fill:#e1ffe1
    style MT5 fill:#e1ffe1
    style MT6 fill:#e1ffe1
    style End fill:#e1ffe1
```

**Key:** Conflict prevention - coding-agent checks BEFORE implementing

---

## Scenario 5: Agent Creation

**User:** "We need an agent that analyzes Konva.js integration patterns"

```mermaid
flowchart TD
    User(["User Request:
    'Create Konva.js analyzer agent'"])

    --> MT1["Main Thread
    Action: Analyze request
    Decision: Agent creation task
    Route: claude-agent-sdk-analyzer"]

    --> Agent["claude-agent-sdk-analyzer-agent

    Step 1 - Read best practices:
    - Tool: Read subagent-best-practices.md
    - Learn: Agent design patterns

    Step 2 - Examine existing agents:
    - Tool: Glob .claude/agents/*-analyzer-agent.md
    - Read: theia-analyzer-agent.md
    - Read: infinite-canvas-analyzer-agent.md
    - Extract: Proven patterns

    Step 3 - Design architecture:
    - YAML frontmatter (name, tools, model)
    - Instruction structure

    Step 4 - Create agent file:
    - Tool: Write .claude/agents/konva-analyzer-agent.md
    - Content: Complete agent spec

    Returns:
    Summary: konva-analyzer-agent created
    Specifications:
    - Tools: Bash, Read, Glob, Grep, WebSearch
    - Model: inherit
    Best Practices Applied:
    - Evidence-based analysis
    - Contextual tool restriction
    - Standard reporting format
    File: .claude/agents/konva-analyzer-agent.md
    Note: Session restart may be needed"]

    --> MT2["Main Thread
    Action: Present to user
    Output: 'Agent created. Restart to use.'"]

    --> End(["Complete
    konva-analyzer-agent ready

    Agents: 1 (claude-agent-sdk-analyzer)
    Duration: Single invocation"])

    style User fill:#e1f5ff
    style MT1 fill:#e1ffe1
    style MT2 fill:#e1ffe1
    style Agent fill:#f0e1ff
    style End fill:#e1ffe1
```

---

## Scenario 6: Simple File Edit

**User:** "Fix the typo in the README - change 'intsall' to 'install'"

```mermaid
flowchart LR
    User(["User Request:
    'Fix typo: intsall → install'"])

    --> MT1["Main Thread
    Action: Analyze request
    Decision: Simple edit
    Route: coding-agent"]

    --> Coding["coding-agent

    Tools:
    - Read: README.md
      (Find typo)
    - Edit: README.md
      old: 'intsall'
      new: 'install'

    Returns:
    Fixed typo in README.md"]

    --> MT2["Main Thread
    Tools:
    - Bash: git add README.md
    - Bash: git commit -m 'docs: fix typo'

    Output: Committed"]

    --> End(["Complete

    Agents: 1 (coding-agent)
    Key: ALL file changes
    via coding-agent"])

    style User fill:#e1f5ff
    style MT1 fill:#e1ffe1
    style MT2 fill:#e1ffe1
    style Coding fill:#ffe1f0
    style End fill:#e1ffe1
```

**Key:** Even 1-line typo fixes go through coding-agent (consistent separation)

---

## Scenario 7a: Technology Research (Happy Path with 2 User Decisions)

**User:** "캔버스에 3D visualization을 추가하고 싶어. 어떤 framework가 좋을까?"

```mermaid
flowchart LR
    subgraph Phase0["PHASE 0: PM Diagnosis"]
        User(["User: Add 3D viz<br/>Which framework?"])
        PM1["PM: Gap Analysis<br/>Glob+Read tech stack<br/>GAP: Konva.js=2D only<br/>Need: True 3D<br/>Plan: 3 parallel"]
        User --> PM1
    end

    subgraph Dec1["USER DECISION 1"]
        UD1{"Proceed?<br/>A) Yes<br/>B) Modify<br/>C) No"}
    end

    Phase0 --> Dec1

    subgraph Phase2["PHASE 2: Parallel Research"]
        direction TB
        R1["Agent1: Three.js<br/>WebSearch+Fetch<br/>600KB, mature"]
        R2["Agent2: Babylon.js<br/>WebSearch+Fetch<br/>1.2MB, complex"]
        R3["Agent3: WebGL<br/>WebSearch<br/>2-3 months"]
    end

    Dec1 -->|A| Phase2
    Dec1 -->|B| PM2["PM Adjust<br/>(7c)"]
    Dec1 -->|C| E1([Stop])
    PM2 --> Dec1

    subgraph Phase3["PHASE 3: Organize"]
        Org["MT: Write 3 docs<br/>ideas&external_references/"]
    end

    Phase2 --> Phase3

    subgraph Phase4["PHASE 4: Analysis"]
        Analyzer["IC-analyzer<br/>Read docs<br/>Three.js: Medium<br/>Babylon: High"]
    end

    Phase3 --> Phase4

    subgraph Phase5["PHASE 5: PM Eval"]
        PM3["PM: Score<br/>Three.js: 4.2/5<br/>Babylon: 3.1/5<br/>WebGL: 2.5/5<br/>Recommend: Three.js"]
    end

    Phase4 --> Phase5

    subgraph Dec2["USER DECISION 2"]
        UD2{"Accept?<br/>A) Three.js<br/>B) Babylon<br/>C) More<br/>D) No"}
    end

    Phase5 --> Dec2

    subgraph Phase7["PHASE 7: Finalize"]
        PM4["PM: Update stack<br/>Edit: tech-stack.md<br/>Write: PRD"]
    end

    Dec2 -->|A or B| Phase7
    Dec2 -->|C| Phase2
    Dec2 -->|D| E2([Stop])

    subgraph Phase8["PHASE 8: Implementation"]
        direction TB
        FW1["theia-analyzer<br/>Webview integration"]
        FW2["electron-analyzer<br/>Security patterns"]
        Coding["coding-agent<br/>Implement"]
        FW1 --> Coding
        FW2 --> Coding
    end

    Phase7 --> Phase8
    Phase8 --> E3(["Complete<br/>Agents: 8<br/>Decisions: 2"])

    style User fill:#e1f5ff
    style UD1 fill:#ffe1e1,stroke:#ff0000,stroke-width:3px
    style UD2 fill:#ffe1e1,stroke:#ff0000,stroke-width:3px
    style PM1 fill:#fff4e1
    style PM2 fill:#fff4e1
    style PM3 fill:#fff4e1
    style PM4 fill:#fff4e1
    style R1 fill:#f0e1ff
    style R2 fill:#f0e1ff
    style R3 fill:#f0e1ff
    style Org fill:#e1ffe1
    style Analyzer fill:#f0e1ff
    style FW1 fill:#f0e1ff
    style FW2 fill:#f0e1ff
    style Coding fill:#ffe1f0
    style E1 fill:#f0f0f0
    style E2 fill:#f0f0f0
    style E3 fill:#e1ffe1
```

**Highlights:**
- **2 User Decision Points**: Approve research? / Accept recommendation?
- **Parallel execution**: 3 research agents + 2 framework agents (simultaneously)
- **PM waits for user approval** before updating technology-stack.md
- **Institutional memory**: Research docs preserved

---

## Scenario 7b: User Rejects PM Recommendation

(Embedded in Scenario 7a diagram - see UserDec2 → B: Choose Other option)

Flow continues with PM re-evaluating user's choice, documenting rationale, adjusting integration strategy.

---

## Scenario 7c: User Modifies Research Criteria

(Embedded in Scenario 7a diagram - see UserDec1 → B: Modify option)

PM adjusts research plan per user modifications (add options, change criteria), user confirms, then proceed.

---

## Common Patterns (Self-Documenting)

### Pattern 1: Quick Framework Question

```mermaid
flowchart LR
    User(["User: Framework question
    Example: 'How does Theia handle menus?'"])

    --> MT["Main Thread
    Action: Analyze request
    Decision: Single-framework question
    Route: Direct to framework agent
    No orchestration needed"]

    --> Agent["theia-analyzer-agent
    Tools:
    - Read: packages/core/src/browser/menu/
    - Grep: menu patterns
    - Read: Examples

    Returns:
    - Detailed explanation
    - Pattern examples
    - File references"]

    --> Answer(["Complete
    User sees answer

    Agents: 1
    Duration: Single invocation
    Efficiency: No orchestration overhead"])

    style User fill:#e1f5ff
    style MT fill:#e1ffe1
    style Agent fill:#f0e1ff
    style Answer fill:#e1ffe1
```

---

### Pattern 2: PM-Driven Development (Complete)

```mermaid
flowchart TD
    User(["User: Feature request"])

    --> PM["PM Agent
    - Discovers tech stack (Glob + Read)
    - Analyzes vision alignment
    - Checks implementation status
    - Selects technology
    - Specifies location
    - Creates phasing strategy
    - Writes PRD
    Returns: Strategic guide"]

    --> MT1["Main Thread
    - Plans investigation
    - Identifies framework agents needed
    - Creates mission prompts"]

    --> FW["Framework Agents
    - Analyze codebase/docs
    - Find patterns
    - Provide file lists
    Returns: Patterns + CREATE/MODIFY/REF"]

    --> MT2["Main Thread
    - Synthesizes PM guide + patterns
    - Creates implementation plan
    Format:
      From PM: [strategic guidance]
      From analyzers: [patterns]
      Tasks: CREATE/MODIFY/REF"]

    --> Coding["coding-agent
    - Reads files for details
    - Implements following patterns
    - Creates/edits files
    Returns: Status report"]

    --> MT3["Main Thread
    - Bash: npm run build
    - Bash: git commit
    Returns: Committed"]

    --> End(["Complete"])

    style User fill:#e1f5ff
    style PM fill:#fff4e1
    style FW fill:#f0e1ff
    style Coding fill:#ffe1f0
    style MT1 fill:#e1ffe1
    style MT2 fill:#e1ffe1
    style MT3 fill:#e1ffe1
    style End fill:#e1ffe1
```

---

### Pattern 3: Full Development Cycle (Parallel Validation)

```mermaid
flowchart TD
    User(["User: Complex feature request
    Example: 'Add custom menu with OS integration'"])

    --> MT["Main Thread
    Action: Orchestrate multi-agent workflow
    Identifies:
    - PM (vision validation)
    - theia-analyzer (menu patterns)
    - electron-analyzer (OS integration)"]

    --> Phase1["PHASE 1: Parallel Validation

    Main Thread sends single message, 2 Tasks:

    Task 1: egdesk-pm-agent
    'Validate custom menu vs vision'

    Task 2: theia-analyzer-agent
    'Analyze Theia menu system'

    Both run simultaneously"]

    --> PM1["PM Agent
    Tools: Glob + Read vision docs
    Returns: APPROVE + considerations"]

    --> FW1["theia-analyzer
    Tools: Read menu code + Grep patterns
    Returns: Menu registration patterns"]

    Phase1 --> PM1
    Phase1 --> FW1

    PM1 --> MT2["Main Thread
    Synthesizes: PM approval + patterns"]
    FW1 --> MT2

    --> Phase2["PHASE 2: Sequential Architecture

    Main Thread (after Phase 1):
    Task: electron-analyzer
    'Given Theia menu pattern,
    analyze OS integration'"]

    --> FW2["electron-analyzer
    Tools: Read Electron menu API
    Returns: OS integration patterns"]

    --> MT3["Main Thread
    Synthesizes ALL guidance:
    - PM: Vision-aligned
    - Theia: Menu patterns
    - Electron: OS integration"]

    --> Coding["PHASE 3: coding-agent

    Implements based on all guidance
    Returns: Status report"]

    --> MT4["Main Thread
    Bash: build, test, commit"]

    --> End(["Complete

    Agents: 4 (PM + 2 FW parallel + 1 FW sequential + coding)
    Duration: Multi-phase with parallelization"])

    style User fill:#e1f5ff
    style PM1 fill:#fff4e1
    style FW1 fill:#f0e1ff
    style FW2 fill:#f0e1ff
    style Coding fill:#ffe1f0
    style MT fill:#e1ffe1
    style MT2 fill:#e1ffe1
    style MT3 fill:#e1ffe1
    style MT4 fill:#e1ffe1
    style End fill:#e1ffe1
```

---

### Pattern 4: Agent Creation

```mermaid
flowchart LR
    User(["User: Need new specialist
    'Create Konva analyzer agent'"])

    --> MT["Main Thread
    Route: claude-agent-sdk-analyzer-agent"]

    --> SDK["claude-agent-sdk-analyzer

    Actions:
    - Read: subagent-best-practices.md
    - Glob + Read: Existing agents (extract patterns)
    - Design: YAML frontmatter + instructions
    - Write: .claude/agents/konva-analyzer-agent.md

    Returns:
    Agent file created
    Session restart may be needed"]

    --> End(["Complete
    New agent ready

    Agents: 1
    Who codes: claude-agent (agent file only)"])

    style User fill:#e1f5ff
    style MT fill:#e1ffe1
    style SDK fill:#f0e1ff
    style End fill:#e1ffe1
```

---

### Pattern 5: Large Implementation (Context Preservation)

```mermaid
flowchart TD
    User(["User: Large multi-file feature
    Example: 'Implement custom dashboard (10+ files)'"])

    --> MT1["Main Thread
    Decision: Large implementation
    Strategy: Delegate to coding-agent
    (preserve Main Thread context)"]

    --> Phases["Phase 1-N: Gather Insights

    Main Thread orchestrates:
    - PM: Vision validation
    - Framework agents: Patterns
    - Architecture guidance

    Main Thread's context stays clean
    (agents do heavy reading)"]

    --> MT2["Main Thread
    Synthesizes insights:

    What: Dashboard system
    Files: 10+ CREATE/MODIFY operations
    Patterns: From framework agents

    Format for coding-agent:
    - Direction (what to implement)
    - File list (detailed CREATE/MODIFY/REF)
    - Pattern references"]

    --> Coding["coding-agent (Separate Context)

    Actions:
    - Reads files for implementation details
    - Writes/Edits 10+ files
    - Follows patterns precisely
    - Returns: Status report

    Benefit: Main Thread context preserved
    Main Thread can continue orchestrating"]

    --> MT3["Main Thread
    After coding-agent returns:
    - Bash: npm run build
    - Bash: git commit

    Context still clean for next feature"]

    --> End(["Complete

    Benefit: Main Thread available for
    continued orchestration

    When to use:
    - 3+ files
    - Large edits (500+ lines)
    - Multiple features in session"])

    style User fill:#e1f5ff
    style MT1 fill:#e1ffe1
    style MT2 fill:#e1ffe1
    style MT3 fill:#e1ffe1
    style Coding fill:#ffe1f0
    style End fill:#e1ffe1
```

---

### Pattern 6: Conflict Detection & Resolution

```mermaid
flowchart TD
    User(["User: EG-DESK custom feature
    'Add keybinding Ctrl+K'"])

    --> MT["Main Thread orchestrates:
    - PM: Strategic guide (EG-DESK custom)
    - Framework agents: Patterns"]

    --> Coding1["coding-agent

    BEFORE implementing:

    Step 1: Discover EG-DESK codebase
    - Glob: eg-desk*/**/*.ts

    Step 2: Find structure doc
    - Glob: eg-desk*/CODEBASE_STRUCTURE.md

    Step 3: Read structure doc

    Step 4: Check conflicts
    - Grep: 'Ctrl+K' in structure"]

    --> ConflictCheck{Conflict?}

    ConflictCheck -->|Yes| Stop["coding-agent: STOP

    DO NOT implement

    Returns:
    ❌ CONFLICT DETECTED
    Alternatives: [list]
    User decision required"]

    ConflictCheck -->|No| Impl["coding-agent: Implement

    Tools:
    - Write: feature code
    - Edit: CODEBASE_STRUCTURE.md
      (Add keybinding + timeline)

    Returns:
    ✅ Complete + Structure updated"]

    Stop --> UserDec{User chooses<br/>alternative}

    UserDec --> Retry["coding-agent: Retry
    with resolved keybinding"]

    Retry --> Impl

    Impl --> MT2["Main Thread
    Bash: build, commit"]

    --> End(["Complete

    Benefit: Prevents duplicate implementations
    When: EG-DESK custom features
    Skip: Theia framework modifications"])

    style User fill:#e1f5ff
    style ConflictCheck fill:#ffe1e1,stroke:#ff0000,stroke-width:2px
    style UserDec fill:#ffe1e1,stroke:#ff0000,stroke-width:3px
    style Coding1 fill:#ffe1f0
    style Stop fill:#ffe1f0
    style Impl fill:#ffe1f0
    style Retry fill:#ffe1f0
    style MT fill:#e1ffe1
    style MT2 fill:#e1ffe1
    style End fill:#e1ffe1
```

---

### Pattern 7: Optional UX Flow Validation (Pre-Build)

```mermaid
flowchart TD
    Impl["coding-agent completed
    implementation"]

    --> MTDecide{"Main Thread Decision:

    Validate UX flow?

    Consider:
    ✅ Complex user flows?
    ✅ Race conditions possible?
    ✅ State management?
    ✅ Async operations?
    ✅ Critical feature?

    ❌ Simple CRUD?
    ❌ Pure styling?
    ❌ Obvious correctness?"}

    MTDecide -->|Validate| UXSim["ux-flow-simulator-agent

    Tools:
    - Read: Implemented files
    - Trace: Code execution paths

    Actions:
    - Simulate user flow
    - Predict runtime behavior
    - Check: Race conditions, null errors

    Returns:
    ✅ No issues predicted
    OR
    ⚠️ Issues found: [list]"]

    MTDecide -->|Skip| Build

    UXSim --> IssuesCheck{Issues?}

    IssuesCheck -->|Yes| Fix["coding-agent
    Fix issues based on
    UX simulator findings"]

    Fix --> UXSim

    IssuesCheck -->|No| Build["Main Thread
    Bash: npm run build"]

    --> End(["Complete

    Benefit: Catch runtime errors BEFORE build
    When: Complex flows, critical features
    Skip: Simple changes, styling"])

    style MTDecide fill:#ffe1e1,stroke:#ff0000,stroke-width:2px
    style IssuesCheck fill:#ffe1e1,stroke:#ff0000,stroke-width:2px
    style UXSim fill:#f0e1ff
    style Fix fill:#ffe1f0
    style Build fill:#e1ffe1
    style End fill:#e1ffe1
```

---

### Pattern 8: Technology Research (With User Control)

```mermaid
flowchart TD
    User(["User: Need new capability"])

    --> PM1["PM: Diagnose limitation
    - Read tech stack
    - Analyze capabilities
    - Identify gap
    Returns: RESEARCH_NEEDED
    + Criteria + Investigation scope"]

    --> UD1{"USER DECISION 1:
    Proceed with research?

    A) Approve
    B) Modify criteria
    C) Reject"}

    UD1 -->|A| MT1["Main Thread
    Execute parallel research
    (3+ agents simultaneously)"]

    UD1 -->|B| PM1b["PM: Adjust plan
    per user modifications"]

    PM1b --> UD1

    UD1 -->|C| End1([Stop])

    MT1 --> Agents["Research Agents
    - WebSearch + WebFetch
    - Gather findings
    Returns: Research results"]

    --> MT2["Main Thread
    - Write research docs
      (ideas&external_references/)
    - Query analyzer agents
    Returns: Organized findings"]

    --> PM2["PM: Evaluate Results
    - Read research docs
    - Score against criteria
    - Assess vision alignment
    Returns: Recommendation
    (WITHOUT updating stack yet)"]

    --> UD2{"USER DECISION 2:
    Accept recommendation?

    PM recommends: [Option X]
    Scoring: [breakdown]

    A) Approve
    B) Choose different
    C) More research
    D) Reject all"}

    UD2 -->|A or B| PM3["PM: Finalize
    - Update tech stack
    - Create PRD
    - Document rationale
    Returns: Adoption complete"]

    UD2 -->|C| MT1
    UD2 -->|D| End2([Stop])

    PM3 --> MT3["Main Thread
    Proceed to implementation
    (Follow Pattern 2)"]

    --> End3([Complete])

    style User fill:#e1f5ff
    style UD1 fill:#ffe1e1,stroke:#ff0000,stroke-width:3px
    style UD2 fill:#ffe1e1,stroke:#ff0000,stroke-width:3px
    style PM1 fill:#fff4e1
    style PM1b fill:#fff4e1
    style PM2 fill:#fff4e1
    style PM3 fill:#fff4e1
    style MT1 fill:#e1ffe1
    style MT2 fill:#e1ffe1
    style MT3 fill:#e1ffe1
    style Agents fill:#f0e1ff
    style End1 fill:#f0f0f0
    style End2 fill:#f0f0f0
    style End3 fill:#e1ffe1
```

---

## Code Ownership Principle

```mermaid
flowchart TD
    MT["Main Thread

    Orchestrates by:
    - Identifying agents (from system prompt)
    - Creating mission prompts
    - Planning execution phases
    - Invoking agents (Task tool)
    - Synthesizing results

    Exclusive capabilities:
    ✅ Bash (build, test, commit, git)
    ✅ Task (invoke agents)
    ✅ Commit and PR creation

    Code delegation (ALWAYS):
    → ALL file changes → coding-agent

    NEVER:
    ❌ Write/Edit application files directly"]

    MT -->|"Guidance only<br/>(read-only)"| Analyzers["Framework Analyzers

    - Analyze codebase/docs
    - Provide patterns
    - Return file references

    Tools:
    - Bash (READ-ONLY analysis)
    - Read, Glob, Grep
    - WebFetch, WebSearch

    NEVER:
    ❌ Write code
    ❌ Edit files
    ❌ Commit"]

    MT -->|"Strategic guide<br/>(vision + docs)"| PM["PM Agent

    - Discovers tech stack (dynamically)
    - Analyzes vision alignment
    - Provides strategic guide
    - Reviews plans
    - Manages documentation

    Tools:
    - Read, Glob, Grep (discovery)
    - Write, Edit (documentation ONLY)
      (PRDs, vision docs, tech stack)

    NEVER:
    ❌ Write application code
    ❌ Implement features"]

    MT -->|"Implementation<br/>(ONLY entity)"| Coding["coding-agent

    ONLY entity that writes application code

    Actions:
    - Reads files for implementation details
    - Creates new files (Write)
    - Edits existing files (Edit)
    - Follows patterns from analyzers
    - Checks conflicts (EG-DESK features)
    - Updates CODEBASE_STRUCTURE.md

    Tools:
    - Write, Edit (application code)
    - Read, Glob, Grep (context)

    NO:
    ❌ Bash (no builds/tests/commits)
    ❌ Architectural decisions

    Returns: Status report"]

    style MT fill:#e1ffe1
    style Analyzers fill:#f0e1ff
    style PM fill:#fff4e1
    style Coding fill:#ffe1f0
```

**Hierarchy:** PM guides → Main Thread orchestrates → Analyzers provide patterns → coding-agent implements

---

## Multi-turn Communication with PM

```mermaid
flowchart TD
    Initial["PM: Initial Strategic Guide

    Provides:
    - Framework choice
    - Code location
    - Implementation phasing
    - Considerations
    - Creates PRD"]

    --> MT["Main Thread: Executes Plan"]

    --> Need{"Main Thread:
    Need PM consultation?

    Situations:
    - Plan needs validation?
    - Guidance ambiguous?
    - Phase N complete, need Phase N+1?
    - Multiple approaches, which one?
    - Found conflicts?
    - Research needed?
    - Research complete, evaluate?"}

    Need -->|Plan Review| PM1["PM: Pattern A
    - Validate plan vs vision
    - Check completeness
    - Review agent reports
    - Suggest improvements
    Returns: PROCEED/REVISE/CONSULT USER"]

    Need -->|Clarification| PM2["PM: Pattern B
    - Re-read context
    - Provide clear direction
    - Give concrete examples
    Returns: Focused clarification"]

    Need -->|Progressive Phase| PM3["PM: Pattern C
    - Acknowledge Phase N results
    - Assess impact on strategy
    - Adjust Phase N+1 direction
    Returns: Updated guidance for next phase"]

    Need -->|Decision Support| PM4["PM: Pattern D
    - Evaluate options vs vision
    - Consider strategic fit
    - Recommend with rationale
    Returns: Clear choice + reasoning"]

    Need -->|Conflict Resolution| PM5["PM: Pattern E
    - Analyze existing implementation
    - Check vision docs
    - Resolve conflict
    Returns: Enhance/Replace/Separate"]

    Need -->|Research Planning| PM6["PM: Pattern F (NEW)
    - Diagnose current stack limitation
    - Define evaluation criteria
    - Structure investigation scope
    - Design parallel execution
    Returns: RESEARCH_NEEDED + plan"]

    Need -->|Research Evaluation| PM7["PM: Pattern G (NEW)
    - Read research docs
    - Score against criteria
    - Assess vision alignment
    - Recommend option
    Returns: Vision-aligned choice"]

    Need -->|Clear, Execute| Exec["Main Thread
    Execute plan directly"]

    PM1 --> Adjust["Main Thread
    Adjust plan based on PM feedback"]
    PM2 --> Adjust
    PM3 --> Adjust
    PM4 --> Adjust
    PM5 --> Adjust
    PM6 --> Adjust
    PM7 --> Adjust

    Adjust --> Exec
    Exec --> Continue([Continue])

    style Initial fill:#fff4e1
    style Need fill:#ffe1e1,stroke:#ff0000,stroke-width:2px
    style PM1 fill:#fff4e1
    style PM2 fill:#fff4e1
    style PM3 fill:#fff4e1
    style PM4 fill:#fff4e1
    style PM5 fill:#fff4e1
    style PM6 fill:#fff4e1
    style PM7 fill:#fff4e1
    style MT fill:#e1ffe1
    style Adjust fill:#e1ffe1
    style Exec fill:#e1ffe1
    style Continue fill:#e1ffe1
```

**7 Multi-turn Patterns:**
- 5 original + 2 NEW (Research Planning + Evaluation)
- PM is stateless - Main Thread includes full context each time

---

## Comparison Matrix (Request Routing)

```mermaid
flowchart TD
    Request{"User Request

    Analyze type:"}

    Request -->|"Simple Question<br/>(file list, git status)"| Route1["Direct Execution

    Main Thread handles directly:
    - Glob, Read, Bash

    Agents: 0
    Who codes: Nobody
    Example: 'List files in packages/core'"]

    Request -->|"File Edit<br/>(typo, version)"| Route2["coding-agent Only

    MT → coding-agent:
    - coding-agent: Read + Edit

    Agents: 1
    Who codes: coding-agent
    Example: 'Fix typo in README'"]

    Request -->|"Framework Question<br/>(How does X work?)"| Route3["Direct to Framework Agent

    MT → theia/electron-analyzer:
    - Agent: Read + Grep + Analyze

    Agents: 1
    Who codes: Nobody
    Example: 'How does Theia DI work?'"]

    Request -->|"Strategic Decision<br/>(Should we add X?)"| Route4["PM Agent Only

    MT → PM:
    - PM: Vision alignment check

    Agents: 1
    Who codes: Nobody (decision only)
    Example: 'Should we add floating AI?'"]

    Request -->|"Theia Implementation<br/>(Modify framework)"| Route5["PM → Analyzers → coding-agent

    Flow:
    - PM: Strategic guide
    - Analyzers: Patterns
    - coding-agent: Implements

    Agents: 2-3
    Who codes: coding-agent
    Conflict check: ❌ No (Theia packages/)
    Example: 'Modify Theia terminal service'"]

    Request -->|"EG-DESK Feature<br/>(Custom code)"| Route6["PM → Analyzers → coding-agent

    Flow:
    - PM: Strategic guide
    - Analyzers: Patterns
    - coding-agent: Implements
      + Conflict check via CODEBASE_STRUCTURE.md

    Agents: 2-3
    Who codes: coding-agent
    Conflict check: ✅ Yes
    Example: 'Add custom QuickSearch + Ctrl+K'"]

    Request -->|"New Technology<br/>(Which framework?)"| Route7["PM Diagnoses → MT Investigates → PM Evaluates

    Flow:
    - PM: Diagnose limitation + plan research
    - USER DECISION: Approve research?
    - MT: Parallel investigation (3+ agents)
    - MT: Organize findings
    - PM: Evaluate + recommend
    - USER DECISION: Accept recommendation?
    - PM: Finalize (update stack)

    Agents: 8+
    Who codes: coding-agent (after approval)
    User Decisions: 2
    Example: 'Add 3D viz - which framework?'"]

    Request -->|"Large Multi-Feature<br/>(Complex system)"| Route8["PM → Multiple Analyzers → coding-agent(s)

    Flow:
    - PM: Strategic guide
    - Multiple framework agents (parallel)
    - Multiple coding-agents (if large)

    Agents: 3-4+
    Who codes: coding-agent(s)
    Conflict check: ✅ If EG-DESK custom
    Example: 'Implement dashboard system'"]

    Request -->|"New Agent<br/>(Create specialist)"| Route9["claude-agent-sdk-analyzer

    Flow:
    - Reads best practices
    - Examines existing agents
    - Designs new agent
    - Writes agent file

    Agents: 1
    Who codes: claude-agent (agent file only)
    Example: 'Create Konva analyzer'"]

    style Request fill:#e1f5ff,stroke:#ff0000,stroke-width:2px
    style Route1 fill:#e1ffe1
    style Route2 fill:#ffe1f0
    style Route3 fill:#f0e1ff
    style Route4 fill:#fff4e1
    style Route5 fill:#fff4e1
    style Route6 fill:#fff4e1
    style Route7 fill:#fff4e1
    style Route8 fill:#fff4e1
    style Route9 fill:#f0e1ff
```

**Route Selection:** Analyze request type → Choose simplest effective approach

---

## Success Metrics Checklist

```mermaid
flowchart TD
    Success["Effective Agent Swarm Flow

    Has ALL these characteristics:"]

    Success --> M1["✅ Right Routing

    - Simple questions → Direct execution
    - Complex tasks → Orchestrated
    - No over-orchestration"]

    Success --> M2["✅ Parallel Execution

    - Independent analyses run simultaneously
    - Single message, multiple Tasks
    - 3x+ faster runtime"]

    Success --> M3["✅ Evidence-Based

    - All recommendations backed by file analysis
    - No assumptions
    - Read before recommend"]

    Success --> M4["✅ Clear Boundaries

    - Agents analyze (read-only)
    - coding-agent implements (writes code)
    - Main Thread orchestrates + builds
    - PM guides strategy"]

    Success --> M5["✅ Strategic Alignment

    - Vision validated before implementation
    - PM checks all EG-DESK features
    - Institutional memory maintained"]

    Success --> M6["✅ User Decision Points

    - Research approval required
    - Recommendation approval required
    - Vision evolution requires user authority
    - User can override PM"]

    Success --> M7["✅ No Redundancy

    - Each agent contributes unique value
    - No duplicate work
    - Clear role separation"]

    Success --> M8["✅ Conflict Prevention

    - coding-agent checks CODEBASE_STRUCTURE.md
    - BEFORE implementing EG-DESK features
    - Reports conflicts, user decides
    - Updates structure after implementation"]

    Success --> M9["✅ Dynamic Discovery

    - No hardcoded paths
    - Glob everything
    - Tech stack discovered from doc
    - Structure-agnostic"]

    Success --> M10["✅ Institutional Memory

    - Research docs preserved
    - Decision rationales documented
    - CODEBASE_STRUCTURE.md maintained
    - Technology stack tracked"]

    style Success fill:#e1ffe1
    style M1 fill:#e1f5ff
    style M2 fill:#e1f5ff
    style M3 fill:#e1f5ff
    style M4 fill:#e1f5ff
    style M5 fill:#e1f5ff
    style M6 fill:#ffe1e1,stroke:#ff0000,stroke-width:2px
    style M7 fill:#e1f5ff
    style M8 fill:#e1f5ff
    style M9 fill:#e1f5ff
    style M10 fill:#e1f5ff
```

---

## Agent Ecosystem Hierarchy

```mermaid
flowchart TD
    User(["User

    - Makes final decisions
    - Provides preferences
    - Approves/rejects plans
    - Can override any recommendation
    - Full authority"])

    --> MT["Main Thread (Orchestrator ONLY)

    Responsibilities:
    - Analyze and route requests
    - Identify agents (from system prompt)
    - Create mission prompts
    - Plan execution phases
    - Invoke agents (Task tool)
    - Synthesize agent outputs
    - Execute bash (build/test/commit)
    - Preserve context (delegate heavy reading)

    Tools: Bash, Task, Read, Glob, Grep
    NO: Write/Edit application code

    Delegation:
    ALL file changes → coding-agent"]

    MT -->|"Strategic<br/>guidance"| PM["PM Agent

    Responsibilities:
    - Discover tech stack (Glob + Read)
    - Analyze vision alignment
    - Diagnose stack limitations
    - Plan technology research
    - Evaluate research results
    - Check implementation status
    - Review execution plans
    - Manage documentation
    - Maintain institutional memory

    Tools: Bash (discovery), Read, Glob, Grep,
           Write/Edit (docs only), WebFetch/Search

    Writes: PRDs, vision docs, tech stack
    NEVER: Application code"]

    MT -->|"Technical<br/>patterns"| Analyzers["Framework Analyzer Agents
    (theia, electron, infinite-canvas, etc.)

    Responsibilities:
    - Analyze codebase/documentation
    - Provide evidence-based guidance
    - Explain framework patterns
    - Find proven patterns
    - Return file references

    Tools: Bash (READ-ONLY), Read, Glob, Grep,
           WebFetch, WebSearch

    Returns: Patterns + File lists
    (CREATE/MODIFY/DELETE/REFERENCE)

    NEVER: Write code, Edit files, Commit"]

    MT -->|"Code<br/>execution"| Coding["coding-agent

    ONLY entity that writes application code

    Responsibilities:
    - Execute code writing/editing
    - Read files for implementation details
    - Follow patterns from analyzers
    - Discover EG-DESK codebase (dynamically)
    - Check conflicts (CODEBASE_STRUCTURE.md)
    - STOP if conflict (report, user decides)
    - Update structure doc after implementation
    - Return status reports

    Tools: Write, Edit, Read, Glob, Grep
    NO: Bash (no builds/tests/commits)

    Receives: Direction + File list from MT
    Returns: Implementation report"]

    MT -->|"Agent<br/>creation"| SDK["claude-agent-sdk-analyzer

    Responsibilities:
    - Design new specialized agents
    - Read best practices
    - Examine existing agents
    - Write agent definition files
    - Provide SDK guidance

    Tools: Read, Write (agent files only),
           Glob, Grep, WebFetch/Search

    Writes: .claude/agents/*.md
    NEVER: Application code"]

    style User fill:#e1f5ff
    style MT fill:#e1ffe1
    style PM fill:#fff4e1
    style Analyzers fill:#f0e1ff
    style Coding fill:#ffe1f0
    style SDK fill:#f0e1ff
```

---

## Key Architectural Principles (Visual Summary)

```mermaid
flowchart LR
    P1["Principle 1:
    Metaphysical Separation

    Main Thread operates at META level:
    - Coordinates agents
    - Delegates domain reading
    - Preserves context

    Agents operate at DOMAIN level:
    - PM: Vision docs
    - Analyzers: Codebase
    - coding-agent: Writes code"]

    P2["Principle 2:
    Strict Code Delegation

    Main Thread NEVER writes code
    ALWAYS delegates to coding-agent

    Even 1-line typo fixes:
    MT → coding-agent → Edit

    Consistent separation of concerns"]

    P3["Principle 3:
    Dynamic Discovery

    NEVER hardcode paths:
    - PM: Glob tech stack (not hardcoded list)
    - Agents: Glob codebases
    - Structure-agnostic

    Always discover current state"]

    P4["Principle 4:
    User Authority

    PM guides, user decides:
    - Research approval required
    - Recommendation approval required
    - Can override PM
    - Can evolve vision

    2+ decision points in research workflow"]

    P5["Principle 5:
    Parallel Execution

    Independent tasks run simultaneously:
    - Single message, multiple Tasks
    - 3x+ faster runtime
    - Research agents in parallel
    - Framework agents in parallel"]

    P6["Principle 6:
    Institutional Memory

    Everything documented:
    - Research docs in ideas&external_references/
    - PRDs for features
    - Decision rationales preserved
    - CODEBASE_STRUCTURE.md maintained
    - Technology stack tracked"]

    style P1 fill:#e1f5ff
    style P2 fill:#ffe1f0
    style P3 fill:#e1ffe1
    style P4 fill:#ffe1e1,stroke:#ff0000,stroke-width:2px
    style P5 fill:#f0e1ff
    style P6 fill:#fff4e1
```

---

## Key Execution Principles (Detailed)

### Principle 1: Tool Ownership (Contextual Restriction)

```mermaid
flowchart TD
    Principle["Tool Ownership Principle:

    Tools restricted CONTEXTUALLY (via prompts)
    NOT mechanically removed

    Agents have tools, but role defines HOW to use"]

    Principle --> Analyzers["Framework Analyzer Agents

    Tool Access:
    - Bash, Glob, Grep, Read
    - WebFetch, WebSearch

    Contextual Restriction (via prompt):
    ✅ Bash for READ-ONLY analysis
       (inspect outputs, run tests to understand)
    ❌ NEVER for implementation
       (commits, builds, installations)

    Example:
    ✅ Bash: npm test (see test behavior)
    ❌ Bash: npm install (implementation)

    Enforced: Agent prompt instructions"]

    Principle --> Coding["coding-agent

    Tool Access:
    - Write, Edit, Read, Glob, Grep

    Contextual Restriction:
    ✅ Code execution ONLY
    ❌ NO Bash (no builds/tests/commits)

    Example:
    ✅ Write: new-service.ts
    ✅ Edit: existing-file.ts
    ❌ Bash: npm run build

    Enforced: Agent role description"]

    Principle --> MT["Main Thread

    Tool Access: ALL tools

    Usage:
    ✅ Bash: build, test, commit, git
    ✅ Task: invoke agents
    ✅ Read, Glob, Grep: orchestration
    ❌ Write/Edit: application code
       (always delegate to coding-agent)

    Full access, contextually appropriate"]

    style Principle fill:#e1ffe1
    style Analyzers fill:#f0e1ff
    style Coding fill:#ffe1f0
    style MT fill:#e1ffe1
```

**Why Contextual Works:**
- Agents understand their role from prompts
- More flexible (agents can investigate runtime behavior)
- No artificial tool removal
- Trust agent instructions to enforce boundaries

---

### Principle 2: When to Spawn Subagent (Context Preservation)

```mermaid
flowchart TD
    Task["Task arrives"]

    --> Decision{"Requires heavy
    domain-specific reading?"}

    Decision -->|Yes| Spawn["✅ SPAWN SUBAGENT

    Scenarios:
    - Extensive domain reading
      (vision docs, framework docs, large codebase)
    - Synthesize knowledge from references
    - Domain analysis needed
    - 'Learn this domain, then apply'
    - Would pollute Main Thread context

    Example:
    'Analyze Theia DI system across 50 files'
    → theia-analyzer reads 50 files
    → Returns 3-paragraph summary
    → Main Thread context: Clean (only summary)

    Benefit: Context preserved"]

    Decision -->|No| Direct["✅ MAIN THREAD DIRECT

    Scenarios:
    - Simple questions (system knowledge)
    - File operations (clear instructions)
    - Orchestration tasks
    - Already have guidance from agents

    Example:
    'List files in packages/core'
    → Glob directly
    → No agent needed

    Benefit: Fast, efficient"]

    Spawn --> Effect1["Effect: Main Thread Context

    WITHOUT subagent:
    MT reads 50 files → context polluted

    WITH subagent:
    Agent reads 50 files → returns summary
    MT context: Clean (only summary)"]

    Direct --> Effect2["Effect: Immediate

    No agent overhead
    Fast execution"]

    style Task fill:#e1f5ff
    style Decision fill:#ffe1e1,stroke:#ff0000,stroke-width:2px
    style Spawn fill:#e1ffe1,stroke:#00ff00,stroke-width:2px
    style Direct fill:#e1ffe1,stroke:#00ff00,stroke-width:2px
    style Effect1 fill:#f0e1ff
    style Effect2 fill:#e1ffe1
```

---

### Principle 3: Parallel vs Sequential Execution

```mermaid
flowchart TD
    Tasks["Multiple analyses needed"]

    --> Check{Independent<br/>analyses?}

    Check -->|Yes| Parallel["✅ PARALLEL

    Single message, multiple Tasks:

    Main Thread sends ONE message:
    - Task(agent: 'theia-analyzer', ...)
    - Task(agent: 'electron-analyzer', ...)
    - Task(agent: 'infinite-canvas-analyzer', ...)

    All THREE execute simultaneously

    Duration: max(T1, T2, T3)
    Example: T1=10s, T2=8s, T3=12s
    → Total: 12s (not 10+8+12=30s)

    Benefit: 3x+ faster"]

    Check -->|No| Sequential["✅ SEQUENTIAL

    Separate messages (dependencies):

    Message 1:
    - Task(agent: 'electron-analyzer',
           'Find security requirements')
    (Wait for response)

    Message 2:
    - Task(agent: 'theia-analyzer',
           'Given security reqs from previous,
            analyze Theia implementation')

    Duration: T1 + T2
    (Later agent needs earlier findings)

    Necessary: Dependent analyses"]

    Parallel --> ParallelWhen["When to use PARALLEL:

    ✅ Independent analyses
    ✅ 'How does each framework handle X?'
    ✅ Multiple research options
    ✅ No dependencies between tasks"]

    Sequential --> SeqWhen["When to use SEQUENTIAL:

    ✅ Later agent needs earlier findings
    ✅ 'Given security reqs, analyze implementation'
    ✅ Dependent information
    ✅ Phase N+1 needs Phase N results"]

    style Tasks fill:#e1f5ff
    style Check fill:#ffe1e1,stroke:#ff0000,stroke-width:2px
    style Parallel fill:#e1ffe1,stroke:#00ff00,stroke-width:2px
    style Sequential fill:#e1ffe1,stroke:#00ff00,stroke-width:2px
    style ParallelWhen fill:#e1f5ff
    style SeqWhen fill:#e1f5ff
```

---

### Principle 4: Conversational Re-query Pattern

```mermaid
flowchart TD
    Agent1["Agent returns report

    Report:
    ✓ Summary: Present
    ✓ Findings: Present
    ✗ Files Analyzed: MISSING"]

    --> MT1["Main Thread
    Action: Detect incomplete report

    Missing: Files Analyzed section
    (REQUIRED for implementation)"]

    --> Requery["Main Thread: Re-query Contextually

    Tool: Task(agent: 'theia-analyzer-agent',
          prompt: 'You previously provided this analysis:

    [ENTIRE PREVIOUS REPORT QUOTED]

    However, Files Analyzed section MISSING (REQUIRED).
    Please provide complete file list with file:line refs.
    No need to re-analyze - just add missing section.')

    Key: Agent is stateless
    BUT Main Thread provides full context"]

    --> Agent2["Agent (stateless but with context)

    Actions:
    - Reads own previous report (from prompt)
    - Extracts file list from analysis
    - Completes missing section

    Returns:
    Files Analyzed:
    - terminal-theme-service.ts:45
    - terminal-frontend-module.ts:32
    - workspace-service.ts:89"]

    --> MT2["Main Thread
    Now has complete report
    Can proceed to next phase"]

    --> End(["Continue

    Benefit:
    - Flexible (not strict validation)
    - Natural conversation
    - Agent can clarify/complete

    When to use:
    - Report missing sections
    - Need clarification
    - Want more detail
    - File list breakdown needed"])

    style Agent1 fill:#f0e1ff
    style MT1 fill:#e1ffe1
    style Requery fill:#e1ffe1
    style Agent2 fill:#f0e1ff
    style MT2 fill:#e1ffe1
    style End fill:#e1ffe1
```

---

### Principle 5: File List Format Enforcement

```mermaid
flowchart LR
    Agent["Framework analyzer
    returns file list"]

    --> Format["Required Format:

    1. CREATE:
       - path/to/new-file.ts (purpose)

    2. MODIFY:
       - path/to/existing.ts:45 (what to change)

    3. DELETE (if applicable):
       - path/to/deprecated.ts (why removing)

    4. REFERENCE (patterns, don't modify):
       - path/to/pattern-file.ts:89
         (Follow @injectable pattern)"]

    --> Benefit["Main Thread Benefits:

    - Extract exact action list
    - Know CREATE vs MODIFY vs DELETE
    - Identify pattern references separately
    - Precise coding-agent instructions:
      'CREATE 1, MODIFY 2, REF 1'"]

    --> CodingPrompt["coding-agent Prompt:

    From theia-analyzer:
    - Pattern: workspace-service.ts:89

    Tasks:
    1. CREATE packages/terminal/src/browser/switcher.ts:
       - Follow workspace-service.ts:89 @injectable pattern
       - Implement time-based logic

    2. MODIFY terminal-frontend-module.ts:36:
       - Add DI binding: bind(Switcher).toSelf()

    3. REFERENCE workspace-service.ts:89:
       - Pattern to follow (don't modify this file)"]

    style Agent fill:#f0e1ff
    style Format fill:#e1f5ff
    style Benefit fill:#e1ffe1
    style CodingPrompt fill:#ffe1f0
```

---

## Orchestration Guidelines Deep Dive

### How Main Thread Orchestrates (10-Step Process)

```mermaid
flowchart TD
    Start(["Complex Development Task"])

    --> Step1["Step 1: Identify Agents

    Source: System prompt knowledge
    (Task tool description lists all agents)

    Main Thread knows:
    - egdesk-pm-agent
    - theia-analyzer-agent
    - electron-analyzer-agent
    - infinite-canvas-analyzer-agent
    - coding-agent
    - ux-flow-simulator-agent
    - etc."]

    --> Step2["Step 2: Select Agents

    Based on capabilities from descriptions

    For 'Add custom menu with OS integration':
    - PM (vision validation)
    - theia-analyzer (menu patterns)
    - electron-analyzer (OS integration)
    - coding-agent (implementation)"]

    --> Step3["Step 3: Analyze Task Requirements

    From user request:
    - What does user want?
    - Which frameworks involved?
    - Complexity level?
    - Vision validation needed?"]

    --> Step4["Step 4: (Optional) Read Agent Details

    If needs specific implementation examples:
    - Read: .claude/agents/[agent].md

    Usually NOT needed:
    - Agent descriptions in system prompt sufficient"]

    --> Step5["Step 5: Create Mission Prompts

    For each agent, create detailed prompt:

    egdesk-pm-agent:
    'Validate custom menu feature against
    ambient AI workspace vision in whitepaper'

    theia-analyzer-agent:
    'Analyze Theia menu system at
    packages/core/src/browser/menu/
    to find registration patterns'"]

    --> Step6["Step 6: Plan Execution Phases

    Identify parallel vs sequential:

    Phase 1 (Parallel):
    - PM validation
    - Theia menu analysis

    Phase 2 (Sequential, after P1):
    - Electron OS integration
      (needs Theia patterns from P1)"]

    --> Step7["Step 7: Identify Decision Points

    Where user input needed:
    - After PM evaluation?
    - Conflict resolution?
    - Multiple options to choose?

    분기점 (Decision gates)"]

    --> Step8["Step 8: Invoke Agents

    Tool: Task

    Phase 1 (single message, 2 Tasks):
    - Task(agent: 'egdesk-pm-agent', ...)
    - Task(agent: 'theia-analyzer-agent', ...)

    Wait for both

    Phase 2 (after P1):
    - Task(agent: 'electron-analyzer-agent',
           prompt includes P1 findings)"]

    --> Step9["Step 9: Synthesize Agent Results

    Collect:
    - PM: APPROVE + considerations
    - theia-analyzer: Menu patterns
    - electron-analyzer: OS integration

    Synthesize into:
    - Implementation direction
    - File list (CREATE/MODIFY/REF)
    - Pattern references"]

    --> Step10["Step 10: Delegate Implementation

    Tool: Task(coding-agent,
          prompt: synthesized guidance)

    OR (if simple enough):
    Main Thread handles directly

    Then:
    - Bash: npm run build
    - Bash: git commit"]

    --> End(["Complete
    Orchestration Done"])

    style Start fill:#e1f5ff
    style Step1 fill:#e1ffe1
    style Step2 fill:#e1ffe1
    style Step3 fill:#e1ffe1
    style Step4 fill:#e1ffe1
    style Step5 fill:#e1ffe1
    style Step6 fill:#e1ffe1
    style Step7 fill:#ffe1e1,stroke:#ff0000,stroke-width:2px
    style Step8 fill:#e1ffe1
    style Step9 fill:#e1ffe1
    style Step10 fill:#e1ffe1
    style End fill:#e1ffe1
```

---

### File Reading Scope (Critical for Context Preservation)

```mermaid
flowchart TD
    MT["Main Thread (When Orchestrating)"]

    MT --> Can["✅ CAN Read:

    Meta-level files:
    - .claude/prompts/agent-orchestration.md
      (orchestration guidelines)
    - .claude/agents/*.md (OPTIONAL)
      (only when needs specific examples
       - agent discovery already in system prompt)

    Purpose: Orchestration guidance only"]

    MT --> Cannot["❌ CANNOT Read (When Orchestrating):

    Domain files:
    - ideas&external_references/eg-desk ideas/
      (That's PM agent's domain)
    - packages/
      (That's framework analyzer agents' domain)
    - Application/framework code
      (Delegate to agents to preserve context)
    - Vision/strategy documents
      (PM agent reads these)

    Why: Preserve Main Thread context
    Agents synthesize → return summary"]

    Cannot --> Effect["Effect of Context Preservation:

    WITHOUT delegation:
    MT reads 50 vision docs → context polluted

    WITH delegation:
    PM reads 50 docs → returns 3-paragraph summary
    MT context: CLEAN (only summary)

    Benefit:
    - Main Thread available for continued orchestration
    - Doesn't load unnecessary reference materials
    - Gets synthesized conclusions only"]

    Can --> Exception["Exception:

    Main Thread CAN read domain files when:
    ✅ Directly implementing
       (after receiving agent guidance)
    ✅ Handling simple tasks
       (no orchestration needed)

    Example:
    'Fix typo' - may read README directly
    before delegating to coding-agent"]

    style MT fill:#e1ffe1
    style Can fill:#e1ffe1,stroke:#00ff00,stroke-width:2px
    style Cannot fill:#ffe1e1,stroke:#ff0000,stroke-width:2px
    style Effect fill:#f0e1ff
    style Exception fill:#fff4e1
```

---

## Anti-Patterns to Avoid (BAD vs GOOD)

### ❌ Anti-Pattern 1: Over-Orchestration

```mermaid
flowchart TD
    Q["User: 'What files are in packages/core?'"]

    Q --> Bad["❌ BAD: Over-Orchestration

    Main Thread:
    - Creates elaborate plan
    - Invokes agents
    - Orchestrates unnecessarily

    Problem:
    - Wastes time
    - Unnecessary complexity
    - Agent overhead for simple task"]

    Q --> Good["✅ GOOD: Direct Execution

    Main Thread:
    - Tool: Glob packages/core/**/*
    - Returns: File list immediately

    Benefit:
    - Fast
    - Simple
    - No agent overhead"]

    Bad --> BadEnd([Slow, complex])
    Good --> GoodEnd([Fast, simple])

    style Q fill:#e1f5ff
    style Bad fill:#ffe1e1,stroke:#ff0000,stroke-width:2px
    style Good fill:#e1ffe1,stroke:#00ff00,stroke-width:2px
    style BadEnd fill:#f0f0f0
    style GoodEnd fill:#e1ffe1
```

---

### ❌ Anti-Pattern 2: Sequential When Parallel Possible

```mermaid
flowchart TD
    Task["Task: Analyze menus in
    Theia AND Electron"]

    Task --> Bad["❌ BAD: Sequential

    Phase 1: theia-analyzer analyzes menus
    (Wait for completion)

    Phase 2: electron-analyzer analyzes menus
    (Wait for completion)

    Problem:
    - These are INDEPENDENT analyses
    - No dependency between them
    - 2x slower (sequential execution)"]

    Task --> Good["✅ GOOD: Parallel

    Phase 1 (Single message, 2 Tasks):
    - Task 1: theia-analyzer
    - Task 2: electron-analyzer

    Both run SIMULTANEOUSLY

    Benefit:
    - 2x faster runtime
    - Efficient use of resources
    - Independent analyses in parallel"]

    Bad --> BadTime["Duration: T1 + T2
    (Sequential = slower)"]

    Good --> GoodTime["Duration: max(T1, T2)
    (Parallel = faster)"]

    style Task fill:#e1f5ff
    style Bad fill:#ffe1e1,stroke:#ff0000,stroke-width:2px
    style Good fill:#e1ffe1,stroke:#00ff00,stroke-width:2px
    style BadTime fill:#f0f0f0
    style GoodTime fill:#e1ffe1
```

---

### ❌ Anti-Pattern 3: Agent Writing Code

```mermaid
flowchart LR
    Task["Analyzer agent
    analyzing codebase"]

    Task --> Bad["❌ BAD: Agent Writes Code

    Agent returns:
    'Here's the code:
    class Foo {
      bar() { ... }
    }'

    Problem:
    - Agents should analyze, not implement
    - Violates role separation
    - Main Thread can't review/adjust"]

    Task --> Good["✅ GOOD: Agent Provides Patterns

    Agent returns:
    'Pattern at packages/core/foo.ts:45
    shows @injectable decorator usage.
    Follow this pattern for FooService.

    File List:
    - CREATE: foo-service.ts
    - REFERENCE: foo.ts:45'

    Benefit:
    - Clear role: agent analyzes
    - Main Thread decides implementation
    - coding-agent implements"]

    style Task fill:#f0e1ff
    style Bad fill:#ffe1e1,stroke:#ff0000,stroke-width:2px
    style Good fill:#f0e1ff,stroke:#00ff00,stroke-width:2px
```

---

### ❌ Anti-Pattern 4: Assumption-Based Guidance

```mermaid
flowchart LR
    Agent["Analyzer agent"]

    Agent --> Bad["❌ BAD: Assumptions

    'Theia probably uses
    dependency injection like Angular'

    Problem:
    - Not evidence-based
    - May be wrong
    - No file references
    - Guessing framework patterns"]

    Agent --> Good["✅ GOOD: Evidence-Based

    Tools Used:
    - Read: packages/core/src/common/di.ts

    Analysis:
    'Theia uses InversifyJS for DI.
    Example at di.ts:89 shows
    @injectable decorator usage.'

    Benefit:
    - Backed by actual file analysis
    - File:line references
    - Accurate guidance"]

    style Agent fill:#f0e1ff
    style Bad fill:#ffe1e1,stroke:#ff0000,stroke-width:2px
    style Good fill:#f0e1ff,stroke:#00ff00,stroke-width:2px
```

---

### ❌ Anti-Pattern 5: Main Thread Writing Code Directly

```mermaid
flowchart TD
    Task["User: 'Fix typo in README'"]

    Task --> Bad["❌ BAD: Main Thread Direct Edit

    Main Thread:
    - Tool: Edit README.md directly
      (old: 'intsall' → new: 'install')

    Problem:
    - Breaks separation of concerns
    - Inconsistent (sometimes MT, sometimes coding-agent)
    - Main Thread context polluted
    - NOT orchestrator behavior"]

    Task --> Good["✅ GOOD: Delegate to coding-agent

    Main Thread:
    - Tool: Task(coding-agent,
             'Fix typo: intsall → install
              File: MODIFY README.md')

    coding-agent:
    - Read: README.md
    - Edit: old → new
    - Return: Fixed

    Main Thread:
    - Bash: git commit

    Benefit:
    - Consistent delegation (ALWAYS)
    - Main Thread stays clean
    - Single responsibility principle
    - Even 1-line fixes go through coding-agent"]

    Bad --> BadEnd([Inconsistent,<br/>polluted context])
    Good --> GoodEnd([Consistent,<br/>clean separation])

    style Task fill:#e1f5ff
    style Bad fill:#ffe1e1,stroke:#ff0000,stroke-width:2px
    style Good fill:#e1ffe1,stroke:#00ff00,stroke-width:2px
    style BadEnd fill:#f0f0f0
    style GoodEnd fill:#e1ffe1
```

---

### ❌ Anti-Pattern 6: Hardcoding Paths

```mermaid
flowchart LR
    Agent["PM or coding-agent"]

    Agent --> Bad["❌ BAD: Hardcoded Paths

    PM: 'Implement in packages/terminal/src/browser/'
    coding-agent: Read('packages/terminal/src/browser/file.ts')

    Problem:
    - Assumes fixed structure
    - Breaks when directory renamed
    - Not structure-agnostic
    - Hard to refactor"]

    Agent --> Good["✅ GOOD: Dynamic Discovery

    PM:
    - Glob: eg-desk*/**/*.ts
    - Discovers: eg-desk_taehwa/
    - Recommends: 'eg-desk_taehwa/terminal/'

    coding-agent:
    - Glob: eg-desk*/CODEBASE_STRUCTURE.md
    - Discovers structure dynamically

    Benefit:
    - Structure-agnostic
    - Flexible (handles renames)
    - Always discovers current state"]

    style Agent fill:#fff4e1
    style Bad fill:#ffe1e1,stroke:#ff0000,stroke-width:2px
    style Good fill:#fff4e1,stroke:#00ff00,stroke-width:2px
```

---

### ❌ Anti-Pattern 7: Not Checking Conflicts (EG-DESK Features)

```mermaid
flowchart TD
    Task["User: 'Bind Ctrl+K to QuickSearch'"]

    Task --> Bad["❌ BAD: No Conflict Check

    coding-agent:
    - Implements immediately
    - Write: search-contribution.ts
      (with Ctrl+K binding)
    - NO conflict check

    Result:
    ❌ Ctrl+K already used elsewhere
    → User confusion
    → Discovered only after deployment
    → Hard to debug 'which feature uses Ctrl+K?'"]

    Task --> Good["✅ GOOD: Conflict Check FIRST

    coding-agent:

    Step 1: BEFORE implementing
    - Read: CODEBASE_STRUCTURE.md
    - Grep: 'Ctrl+K'

    Step 2: If conflict
    - STOP immediately
    - Report to user with alternatives
    - User chooses: Ctrl+Shift+K

    Step 3: Implement with resolved key
    - Write: search-contribution.ts (Ctrl+Shift+K)
    - Edit: CODEBASE_STRUCTURE.md (update registry)

    Benefit:
    - Prevents duplicate keybindings
    - User decides before implementation
    - Registry always up-to-date
    - No confusion"]

    Bad --> BadEnd([Duplicate keybinding<br/>User confusion<br/>Wasted time])

    Good --> GoodEnd([No conflicts<br/>Clean registry<br/>Prevented early])

    style Task fill:#e1f5ff
    style Bad fill:#ffe1e1,stroke:#ff0000,stroke-width:2px
    style Good fill:#ffe1f0,stroke:#00ff00,stroke-width:2px
    style BadEnd fill:#f0f0f0
    style GoodEnd fill:#e1ffe1
```

---

## Conclusion

This visual guide provides **self-documenting Mermaid diagrams** where each node contains complete information:
- **Tools used** (Bash, Read, Write, etc.)
- **Specific actions** (what the entity does)
- **Outputs** (what it returns)
- **Decisions** (APPROVE/REJECT/RESEARCH_NEEDED)

**You don't need to read external explanations** - the diagrams are self-contained.

**Key Takeaways:**
1. **User Decision Points** (red borders) are critical gates - user has control
2. **Parallel execution** (shown as simultaneous branches) = faster runtime
3. **PM guides, user decides** - all strategic decisions require user approval
4. **Main Thread never writes code** - always delegates to coding-agent
5. **Each node is self-documenting** - includes tools, actions, outputs
6. **Diagrams tell the complete story** - no need for separate documentation

For detailed prose explanations and comprehensive context, see [AGENT_SWARM_FLOW.md](./AGENT_SWARM_FLOW.md).
