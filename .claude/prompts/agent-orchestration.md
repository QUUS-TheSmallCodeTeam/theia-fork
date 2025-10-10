# Agent Orchestration Guidelines for Main Thread

When handling complex, multi-step development tasks that require coordinating multiple specialized agents, follow these orchestration principles.

## When to Orchestrate

Orchestrate when the user request requires:
- Implementing new EG-DESK features
- Multi-step development workflows
- Cross-framework integration (Theia AND Electron)
- Complex troubleshooting requiring multiple domains
- Strategic implementation planning

## Orchestration Methodology

### Step 1: Task Decomposition

Analyze the user request:
- **Primary objective**: What is the user ultimately trying to achieve?
- **Framework domains**: Which frameworks are involved? (Theia, Electron, both?)
- **Knowledge required**: What specific knowledge is needed?
- **Complexity level**: Simple lookup, analysis, or multi-step investigation?

### Step 2: Agent Selection & Execution Chain Planning

Read `.claude/agents/` to discover available agents and their capabilities.

Determine which agents to deploy AND how to chain them:
- **Single agent**: If task is within one framework domain
- **Multiple agents (parallel)**: If task requires independent analysis from multiple domains
- **Multiple agents (sequential)**: If later agents need earlier agents' findings
- **Mixed parallel + sequential chains**: Complex workflows with both independent and dependent steps
- **Plan until decision point (분기점)**: Stop planning when you reach a point that requires:
  - User decision or preference selection
  - Experimental validation before proceeding
  - Results inspection to determine next steps
  - External input or clarification

**Chain Planning Strategy:**
- Map out the entire execution chain up to the first decision point
- Group independent analyses for parallel execution
- Sequence dependent analyses that need prior results
- Identify where user input or validation becomes necessary
- Create complete execution plan for the deterministic portion

### Step 3: Mission Prompt Crafting

For each selected agent, create a mission prompt that includes:
- **Context**: Relevant background from user's request
- **Objective**: Specific goal for this agent
- **Deliverables**: Exact format of expected output
- **Constraints**: Any limitations or special considerations
- **File paths**: Specific files or directories to examine (if known)

### Step 4: Execute the Plan

**For Parallel Execution:**
Send ONE message with multiple Task calls:
```
Task(agent: "agent-1", prompt: "[mission]")
Task(agent: "agent-2", prompt: "[mission]")
Task(agent: "agent-3", prompt: "[mission]")
```

**For Sequential Execution:**
Wait for previous phase to complete, then invoke next agent with context from prior results.

### Step 5: Synthesize and Implement

- Combine insights from all agents
- Make implementation decisions
- Write code following agent guidance
- Build, test, commit

## Execution Plan Structure

When orchestrating, present the plan to yourself (internally) or the user (if complex):

```
## Task Analysis
[Brief analysis of what the user is asking for]
[Identify the first decision point/분기점 in the workflow]

## Execution Chain Plan

### Phase 1: [Name - Parallel/Sequential]
**Execution Mode**: Parallel/Sequential
**Agents**: [list agents for this phase]
**Dependencies**: None / [what this phase needs from previous phases]

**Agent Missions for Phase 1:**

**Agent: [agent-name-1]**
```
[Exact prompt to pass to Task tool]
```

**Agent: [agent-name-2]**
```
[Exact prompt to pass to Task tool]
```

**Synthesis after Phase 1**: [How to combine Phase 1 results]

---

### Phase 2: [Name - Parallel/Sequential]
**Execution Mode**: Parallel/Sequential
**Agents**: [list agents for this phase]
**Dependencies**: Requires results from Phase 1 agents: [specific agents]

[Continue pattern...]

---

## Decision Point (분기점)

**Stop here because**: [User decision needed / Experimental validation required / etc.]

**What user needs to decide/validate**:
- [Specific decision or experiment needed]

**Possible next paths**:
- Path A: If [condition], then [next steps]
- Path B: If [condition], then [next steps]

**Recommendation**: [What to check/try/decide]
```

## Critical Operating Principles

🎯 **BE DECISIVE**
- Quickly identify the right agents for the job
- Don't over-complicate simple tasks
- Trust specialized agents to handle their domains

📋 **BE SPECIFIC**
- Create detailed mission prompts with clear objectives
- Include all necessary context from the original request
- Specify exact deliverable formats

⚡ **BE EFFICIENT**
- Use parallel execution when agents don't depend on each other
- Minimize back-and-forth by providing complete context upfront
- Only invoke agents when they add clear value

🔗 **BE COORDINATED**
- Consider dependencies between agent outputs
- Plan synthesis strategy before execution
- Ensure mission prompts align with final goal

⛓️ **PLAN IN CHAINS**
- Think in phases: parallel groups + sequential dependencies
- Map the entire workflow up to the first decision point (분기점)
- Identify where user input or experimental validation is required
- Create complete execution chains for all deterministic steps
- Stop planning when uncertainty requires user involvement

## Example Orchestration Patterns

### Pattern 1: Single Framework Deep Dive
```
User asks: "How does Theia implement the file watcher system?"

Analysis: Single framework (Theia), requires codebase investigation
Execution: Single phase, single agent
Decision Point: None (straightforward analysis)

Execute:
Task(agent: "theia-analyzer-agent",
     prompt: "Analyze packages/filesystem to understand how Theia implements the file watcher system")
```

### Pattern 2: Cross-Framework Integration (Parallel)
```
User asks: "How should I integrate Electron's native dialogs with Theia's dialog service?"

Analysis: Both frameworks, independent analyses can run in parallel
Execution: Single phase, parallel agents
Decision Point: After getting both analyses, user may need to choose integration approach

Execute (single message, multiple Tasks):
Task(agent: "theia-analyzer-agent",
     prompt: "Analyze Theia's dialog service architecture in packages/core/src/browser/dialogs")
Task(agent: "electron-analyzer-agent",
     prompt: "Research Electron's native dialog APIs and security best practices")

Then: Synthesize both patterns and present integration options to user
```

### Pattern 3: Sequential Investigation
```
User asks: "I'm getting a security warning in my Electron app related to Theia's preload script"

Analysis: Electron security first, then Theia implementation
Execution: Two sequential phases
Decision Point: None (deterministic troubleshooting flow)

Phase 1:
Task(agent: "electron-analyzer-agent",
     prompt: "Analyze Electron security requirements for preload scripts")

Phase 2 (after Phase 1):
Task(agent: "theia-analyzer-agent",
     prompt: "Given Electron's security requirements [from Phase 1], analyze how Theia implements preload scripts")

Then: Provide fix based on both analyses
```

### Pattern 4: Complex Multi-Phase Chain
```
User asks: "Help me add a custom menu system with native OS integration to EG-DESK"

Analysis: Multiple frameworks, multiple steps, experimental validation needed
Execution: Mixed parallel + sequential, stops at validation point

Phase 1 (Parallel - Independent research):
Task(agent: "egdesk-pm-agent",
     prompt: "Validate: Does custom native menu align with EG-DESK vision?")
Task(agent: "theia-analyzer-agent",
     prompt: "Analyze how Theia's menu system works")
Task(agent: "electron-analyzer-agent",
     prompt: "Research Electron's Menu API patterns")

Phase 2 (Sequential - Integration analysis):
Task(agent: "theia-analyzer-agent",
     prompt: "Given Theia's menu system [Phase 1] and Electron's Menu API [Phase 1], analyze how to extend Theia menus with native integration")

Phase 3 (Parallel - Security + Examples):
Task(agent: "electron-analyzer-agent",
     prompt: "Security best practices for native menus")
Task(agent: "theia-analyzer-agent",
     prompt: "Find existing menu extension examples")

DECISION POINT: User needs to implement basic prototype and test before proceeding
- User should implement based on Phase 1-3 insights
- Test the basic menu integration
- Verify it works before adding advanced features

Possible next paths after validation:
  - Path A: If working → proceed with advanced features (keyboard shortcuts, dynamic menus)
  - Path B: If issues → troubleshoot with targeted agent queries
```

## Constraints

- **Never skip to implementation**: Always get agent guidance first for complex tasks
- **Stay focused**: Only deploy agents that clearly add value
- **Be explicit**: Provide exact prompts with clear objectives
- **Trust specialists**: Don't second-guess agent capabilities
- **Synthesize**: Combine agent insights before implementing

## Success Metrics

An effective orchestration has:
- ✅ Right agents selected for the domains involved
- ✅ Parallel execution maximized (independent tasks in single message)
- ✅ Clear mission prompts with specific deliverables
- ✅ Decision points identified and communicated to user
- ✅ Complete synthesis of agent guidance before implementation
