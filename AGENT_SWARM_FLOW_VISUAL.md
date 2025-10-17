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
flowchart TD
    User(["User Request:
    'Add time-based terminal theme'"])

    --> MT1["Main Thread
    Action: Analyze request
    Decision: Development task
    Route: PM for strategic guide"]

    --> PM1["PHASE 0: PM Strategic Guide

    Tools:
    - Glob: ideas&external_references/eg-desk ideas/**/*.md
    - Glob: packages/*/package.json
    - Read: EG-DESK_Whitepaper.md
    - Read: packages/terminal/package.json
    - Read: packages/terminal/src/browser/terminal-theme-service.ts
    - Grep: 'theme', 'terminal'

    Analysis:
    ✅ Vision aligned (ambient AI principles)
    ✅ Framework: Theia (terminal = Theia domain)
    ✅ No duplicates found
    ✅ Existing: terminal-theme-service.ts

    Actions:
    - Creates PRD

    Returns:
    Decision: APPROVE
    Framework: Theia
    Location: packages/terminal/src/browser/
    Phasing: Phase 1 - analyze, Phase 2 - design, Phase 3 - implement
    Considerations: Manual override, preference persistence
    PRD: time-based-terminal-theme-prd.md"]

    --> MT2["PHASE 1: Main Thread
    Investigation Planning

    Action: Plan framework investigation
    Based on: PM's Phase 1 guidance

    Plan:
    - Query theia-analyzer for theme patterns
    - Query theia-analyzer for DI registration"]

    --> FW1["PHASE 2: theia-analyzer-agent
    Framework Investigation

    Tools:
    - Read: packages/terminal/src/browser/terminal-theme-service.ts
    - Read: packages/terminal/src/browser/terminal-frontend-module.ts
    - Grep: @injectable

    Analysis:
    - Theme registration pattern found
    - DI binding pattern identified

    Returns:
    Files Analyzed:
    - terminal-theme-service.ts:45
    - terminal-frontend-module.ts:32
    Pattern: ThemeService.register() with DI
    File List:
    - CREATE: time-based-theme-switcher.ts
    - MODIFY: terminal-frontend-module.ts:36
    - REFERENCE: workspace-service.ts:89
      (for @injectable pattern)"]

    --> MT3["PHASE 3: Main Thread
    Implementation Planning

    Action: Synthesize PM guide + framework patterns

    Implementation Plan:
    Direction: Create TimeBasedThemeSwitcher
    following Theia DI pattern

    File List:
    - CREATE: time-based-theme-switcher.ts
    - MODIFY: terminal-frontend-module.ts:36
    - MODIFY: terminal-contribution.ts:89
    - REFERENCE: workspace-service.ts:89
      (for @injectable pattern)"]

    --> MT4["Main Thread
    Action: Delegate to coding-agent

    Tool: Task
    Prompt includes:
    - Direction (what to implement)
    - File list (CREATE/MODIFY/REF)
    - Pattern references
    - 'You will read files for details'"]

    --> Coding1["PHASE 4: coding-agent
    Implementation Execution

    Tools:
    - Read: workspace-service.ts:89 (pattern)
    - Write: time-based-theme-switcher.ts
      (with @injectable, DI pattern)
    - Read: terminal-frontend-module.ts
    - Edit: Add DI binding
      (bind(TimeBasedThemeSwitcher)...)
    - Read: terminal-contribution.ts
    - Edit: Inject service
      (@inject(TimeBasedThemeSwitcher))

    Returns:
    Implementation complete: 1 CREATE, 2 MODIFY
    Following workspace-service.ts pattern"]

    --> MTDecide{"PHASE 4.5: Main Thread Decision

    Validate UX flow?

    Consider:
    ✅ Complex user flows?
    ✅ Race conditions?
    ✅ State management?
    ✅ Critical feature?

    ❌ Simple CRUD?
    ❌ Pure styling?"}

    MTDecide -->|"Validate<br/>(Complex)"| UXSim["ux-flow-simulator-agent

    Tools:
    - Read: time-based-theme-switcher.ts
    - Read: terminal-contribution.ts
    - Read: terminal-frontend-module.ts

    Actions:
    - Trace execution flow
    - Check: User opens terminal → theme switches
    - Predict: Race conditions, runtime errors

    Returns:
    ✅ No race conditions found
    ✅ Expected behavior correct
    OR
    ⚠️ Issues: [list] → Fix needed"]

    MTDecide -->|"Skip<br/>(Simple)"| MT5

    UXSim --> UXCheck{"Issues found?"}

    UXCheck -->|Yes| Fix["coding-agent
    Action: Fix issues
    Re-validate needed"]

    Fix --> UXSim

    UXCheck -->|No| MT5["PHASE 5: Main Thread
    Build & Commit

    Tools:
    - Bash: npm run build
    - Bash: git add .
    - Bash: git commit -m 'feat(terminal): ...'

    Output: Committed"]

    --> End(["Complete
    Feature implemented and committed

    Total Agents: 3-4
    - PM strategic guide
    - theia-analyzer
    - coding-agent
    - (optional) ux-flow-simulator

    Who Coded: coding-agent
    Duration: Multi-phase"])

    style User fill:#e1f5ff
    style MTDecide fill:#ffe1e1,stroke:#ff0000,stroke-width:2px
    style UXCheck fill:#ffe1e1,stroke:#ff0000,stroke-width:2px
    style PM1 fill:#fff4e1
    style FW1 fill:#f0e1ff
    style UXSim fill:#f0e1ff
    style Coding1 fill:#ffe1f0
    style Fix fill:#ffe1f0
    style MT1 fill:#e1ffe1
    style MT2 fill:#e1ffe1
    style MT3 fill:#e1ffe1
    style MT4 fill:#e1ffe1
    style MT5 fill:#e1ffe1
    style End fill:#e1ffe1
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
flowchart TD
    User(["User Request:
    'Add 3D visualization
    Which framework?'"])

    --> MT1["Main Thread
    Action: Analyze request
    Decision: New capability, may need new tech
    Route: PM for gap assessment"]

    --> PM1["PHASE 0: PM Diagnosis

    Tools:
    - Glob: ideas*/**/eg-desk*ideas*/*tech*.md
    - Read: technology-stack.md
    - Glob: ideas*/**/eg-desk*ideas*/**/*.md
    - Read: EG-DESK_Whitepaper.md

    Analysis:
    Current Stack:
    - Konva.js = 2D canvas library
    - Infinite Canvas = viewport transforms

    User Needs: True 3D rendering
    (lighting, shadows, depth)

    GAP FOUND:
    Konva.js limited to 2D shapes
    Pseudo-3D insufficient for complex 3D

    Vision Check: ✅ 3D viz aligns with
    spatial canvas principles

    Research Plan Created:
    Options: Three.js, Babylon.js, Custom WebGL
    Criteria:
    - Bundle size < 500KB
    - Integration complexity (w/ Infinite Canvas)
    - Performance: 1000 obj @ 60fps
    - Compatibility w/ existing stack

    Parallel Design:
    All 3 independent → run simultaneously

    Returns:
    Decision: RESEARCH_NEEDED
    Limitation: Konva.js is 2D-only
    Why: User needs true 3D w/ lighting
    Investigation Scope: 3 parallel
    Expected: MT returns with findings"]

    --> MT2["Main Thread
    Action: Present research plan to user"]

    --> UserDec1{"USER DECISION 1:
    Proceed with research?

    PM diagnosed:
    - Konva.js cannot do true 3D
    - Recommends investigating:
      Three.js, Babylon.js, Custom WebGL
    - Criteria: bundle, integration, perf

    Options:
    A) Approve → Proceed
    B) Modify → Adjust criteria/options
    C) Reject → Use current stack"}

    UserDec1 -->|A: Approve| MT3["Main Thread
    Action: Execute parallel research"]

    UserDec1 -->|B: Modify| PM2["PM: Adjust Plan
    (See Scenario 7c)"]

    UserDec1 -->|C: Reject| End1(["Stop
    Use current stack workaround"])

    PM2 --> UserConfirm{User: OK?}
    UserConfirm -->|Yes| MT3
    UserConfirm -->|No| End1

    MT3 --> Parallel["PHASE 2: Parallel Research

    Main Thread executes:
    Single message, 3 Tasks

    Task 1: general-purpose
    'Research Three.js: bundle, 3D caps,
    IC integration, performance'

    Task 2: general-purpose
    'Research Babylon.js: bundle, 3D caps,
    IC integration, performance'

    Task 3: general-purpose
    'Research Custom WebGL: feasibility,
    dev time, maintenance'

    → All 3 run SIMULTANEOUSLY"]

    --> Research["3 general-purpose agents

    Agent 1 - Three.js:
    - WebSearch: 'Three.js bundle size'
    - WebSearch: 'Three.js IC integration'
    - WebFetch: threejs.org/docs
    Returns: ~600KB, mature, good docs

    Agent 2 - Babylon.js:
    - WebSearch: 'Babylon.js bundle'
    - WebFetch: babylonjs.com/docs
    Returns: ~1.2MB, game-focused, complex

    Agent 3 - Custom WebGL:
    - WebSearch: 'WebGL integration'
    Returns: Feasible, 2-3 months dev time"]

    --> MT4["PHASE 3: Main Thread
    Research Organization

    Tools:
    - Write: ideas&external_references/
             threejs-research.md
             (Three.js findings)
    - Write: babylonjs-research.md
             (Babylon.js findings)
    - Write: custom-webgl-research.md
             (Custom WebGL findings)

    Output: 3 research docs created
    Institutional memory preserved"]

    --> MT5["Main Thread
    Action: Query analyzer for
    technical integration assessment"]

    --> Analyzer["PHASE 4: infinite-canvas-analyzer

    Tools:
    - Read: threejs-research.md
    - Read: babylonjs-research.md
    - Read: infinite-canvas/[integration code]

    Analysis:
    - Three.js: Camera sync with IC viewport
      Complexity: MEDIUM
    - Babylon.js: Scene graph conflicts
      Complexity: HIGH

    Returns:
    Three.js: Medium integration complexity
    Babylon.js: High complexity"]

    --> MT6["Main Thread
    Action: Return to PM with research results

    Provides:
    - Research docs: threejs, babylonjs, webgl
    - Analyzer findings: integration complexity"]

    --> PM3["PHASE 5: PM Evaluation

    Tools:
    - Read: threejs-research.md
    - Read: babylonjs-research.md
    - Read: custom-webgl-research.md

    Evaluation Against Criteria:

    Three.js:
    - Bundle: 600KB (vs 500KB target = minor over)
    - Integration: Medium
    - Performance: Excellent
    - Score: 4.2/5

    Babylon.js:
    - Bundle: 1.2MB (2x over target)
    - Integration: High complexity
    - Score: 3.1/5

    Custom WebGL:
    - Dev time: 2-3 months
    - Maintenance: High burden
    - Score: 2.5/5

    Vision Alignment:
    Three.js: Best fit (lightweight, proven)

    Returns (WITHOUT updating stack yet):
    Recommended: Three.js
    Rationale: Best balance
    Scoring: [detailed breakdown]
    Why not Babylon: Bundle too large
    Why not Custom: Too much dev time
    Integration Strategy: 3 phases"]

    --> MT7["Main Thread
    Action: Present PM recommendation"]

    --> UserDec2{"USER DECISION 2:
    Accept Three.js?

    PM evaluated 3 options:
    - Three.js: 4.2/5 (recommended)
    - Babylon.js: 3.1/5 (bundle large)
    - Custom: 2.5/5 (dev time)

    Options:
    A) Approve Three.js
    B) Choose Babylon.js
    C) More research
    D) Reject all"}

    UserDec2 -->|A: Approve| PM4["PM: Finalize Adoption

    Tools:
    - Edit: technology-stack.md
      (Add Three.js to 3D Rendering category
       Status: Approved for Integration
       Bundle: ~600KB
       Integration: Layers with IC)
    - Write: 3d-visualization-with-threejs-prd.md
      (Research summary + decision rationale
       + integration phases)

    Returns:
    Technology Stack Updated: Three.js added
    PRD Created: Integration plan
    Next Steps: Query framework agents"]

    UserDec2 -->|B: Choose Other| PM4b["PM: Re-evaluate
    (See Scenario 7b)"]

    UserDec2 -->|C: More| MT3

    UserDec2 -->|D: Reject| End2(["Stop
    No new tech added"])

    PM4 --> MT8["Main Thread
    Action: Proceed to implementation

    Next: Query theia-analyzer + electron-analyzer
    for integration patterns"]

    PM4b --> MT8

    --> FW["Framework Agents (Parallel)

    theia-analyzer:
    - Analyze webview integration
      for Three.js canvas

    electron-analyzer:
    - Electron security for Three.js
      in renderer process

    Both run simultaneously"]

    --> MT9["Main Thread
    Action: Synthesize + spawn coding-agent"]

    --> Coding["coding-agent
    Action: Implement Three.js integration
    Direction: From PM guide
    Files: From analyzer agents"]

    --> End3(["Complete
    3D visualization integrated

    Total Agents: 8
    - PM planning
    - 3 research (parallel)
    - 1 analyzer
    - PM evaluation
    - PM finalization
    - 2 framework (parallel)
    - coding-agent

    Duration: Multi-phase
    User Decisions: 2"])

    style User fill:#e1f5ff
    style UserDec1 fill:#ffe1e1,stroke:#ff0000,stroke-width:3px
    style UserDec2 fill:#ffe1e1,stroke:#ff0000,stroke-width:3px
    style UserConfirm fill:#ffe1e1,stroke:#ff0000,stroke-width:3px
    style PM1 fill:#fff4e1
    style PM2 fill:#fff4e1
    style PM3 fill:#fff4e1
    style PM4 fill:#fff4e1
    style PM4b fill:#fff4e1
    style MT3 fill:#e1ffe1
    style MT4 fill:#e1ffe1
    style MT5 fill:#e1ffe1
    style MT6 fill:#e1ffe1
    style MT7 fill:#e1ffe1
    style MT8 fill:#e1ffe1
    style MT9 fill:#e1ffe1
    style Parallel fill:#f0e1ff
    style Research fill:#f0e1ff
    style Analyzer fill:#f0e1ff
    style FW fill:#f0e1ff
    style Coding fill:#ffe1f0
    style End1 fill:#f0f0f0
    style End2 fill:#f0f0f0
    style End3 fill:#e1ffe1
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
