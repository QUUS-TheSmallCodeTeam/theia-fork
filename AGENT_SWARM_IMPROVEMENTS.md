# Agent Swarm System: Identified Issues and Potential Improvements

This document captures potential issues and improvements for the agent swarm orchestration system identified during architecture review.

## Critical Issues

### 1. Agent Discovery Overhead

**Issue:**
Current documentation says Main Thread "discovers agents by reading `.claude/agents/`" every time it orchestrates. However, Main Thread already knows all available agents through the system prompt (the Task tool description lists all agents with their descriptions).

**Impact:**
- Unnecessary file operations
- Slower orchestration startup
- Misleading documentation about how agent discovery works

**Improvement:**
Clarify that Main Thread:
- Uses built-in knowledge of agents from system prompt (primary method)
- Only Globs `.claude/agents/` when checking for newly added agents mid-session
- Reads agent definitions only when need to understand detailed capabilities

**Decision Required:**
Should we update orchestration guidelines to reflect this, or is re-reading agent files each time actually beneficial for freshness?

---

### 2. Metaphysical Separation Boundary Ambiguity

**Issue:**
Document states "Main Thread CANNOT read domain files when orchestrating" but also "CAN read when implementing directly." The boundary between these modes is unclear.

In Scenario 4, steps 14-19 show Main Thread reading and editing files directly - is this:
- a) Implementation phase (allowed)?
- b) Orchestration phase (violating the rule)?

**Impact:**
- Confusing guidelines
- Unclear when Main Thread can/cannot read domain files
- Risk of violating metaphysical separation principle

**Improvement Options:**

**Option A: Strict Separation**
```
Orchestration Phase (Meta-level):
├─ Main Thread reads: .claude/agents/, .claude/prompts/ only
├─ Main Thread creates: mission prompts
└─ Main Thread invokes: analyzer agents

Implementation Phase (Execution-level):
├─ Main Thread CAN read domain files
├─ Main Thread CAN write/edit code
└─ Main Thread builds, tests, commits
```

**Option B: Pragmatic Separation**
```
Main Thread can read domain files when:
✅ Implementing (after receiving all agent guidance)
✅ Simple tasks that don't need orchestration
✅ Verifying agent suggestions

Main Thread delegates domain reading when:
✅ Planning complex orchestration
✅ Need specialized analysis
✅ Unknown patterns need discovery
```

**Decision Required:**
Which separation model should we enforce?

---

### 3. SDK Analyzer Undefined Usage

**Issue:**
Document clarifies claude-agent-sdk-analyzer-agent is "for implementing Claude Code SDK into forked apps" but provides **zero scenarios** showing when/how this would actually be used.

**Impact:**
- Unclear value proposition for this agent
- No guidance on when to invoke it
- Potential misuse

**Missing Scenarios:**
- User wants to add slash commands to their fork
- User wants to implement custom hooks
- User wants to use Claude Code MCP servers
- User wants to understand subagent system for their fork

**Improvement:**
Add Scenario 7: "Implementing Claude Code Slash Commands in Fork"
```
User: "How do I add custom slash commands to this Theia fork?"

Main Thread → claude-agent-sdk-analyzer-agent

Agent analyzes:
├─ ideas&external_references/claude-agent-sdk/
├─ Finds slash command implementation patterns
└─ Returns: How to create .claude/commands/ structure

Main Thread implements based on guidance
```

**Decision Required:**
Should we add this scenario, or is SDK analyzer actually not needed (Main Thread can read SDK docs directly like it does for agent creation)?

---

### 4. coding-agent Error Handling

**Issue:**
No documentation for error correction flows:
- What if coding-agent's Edit fails (old_string not found)?
- What if coding-agent writes syntactically invalid code?
- Main Thread builds → fails → then what?

**Impact:**
- No clear recovery procedure
- Main Thread may not know how to handle coding-agent failures
- Could lead to endless retry loops

**Missing Patterns:**

**Pattern A: Edit Failure**
```
coding-agent → Edit fails (old_string not found)
coding-agent → Reports error in status
Main Thread → Reads file to understand issue
Main Thread → Re-invokes coding-agent with corrected instructions
```

**Pattern B: Build Failure After Coding**
```
coding-agent → Implementation complete
Main Thread → npm run build → FAILS
Main Thread → Reads build errors
Main Thread → Options:
  ├─ Fix directly (if simple)
  ├─ Re-invoke coding-agent with error context
  └─ Invoke theia-analyzer-agent to understand error
```

**Pattern C: Incremental Fix**
```
Main Thread → coding-agent implements feature
coding-agent → Returns status
Main Thread → Builds → Error in file 2
Main Thread → Edits file 2 directly (small fix)
Main Thread → Builds → Success
```

**Improvement:**
Add "Error Correction Flows" section to AGENT_SWARM_FLOW.md

---

### 5. Decision Point Continuation Pattern

**Issue:**
When Main Thread hits a 분기점 and user provides a decision, the continuation flow is unclear:
- Does user send new request?
- Does Main Thread remember the context and continue?
- How does conversation resume after experimental validation?

**Impact:**
- User confusion about how to proceed after decision points
- Lost context if user doesn't know to reference previous discussion
- Inefficient restart of entire orchestration

**Missing Pattern:**
```
Main Thread → Reaches Decision Point:
  "DECISION POINT: User needs to implement prototype and test.
   After testing, report results and I'll continue orchestration."

User → Tests → "It works! The menu shows up."

Main Thread → Continues orchestration:
  Phase 4: Advanced features (was planned but waiting for validation)

OR

User → Tests → "Error: Menu doesn't respond to clicks"

Main Thread → Re-orchestrates:
  ├─ Invoke electron-analyzer-agent for event handling
  └─ Fix implementation
```

**Improvement:**
Add "Decision Point Continuation" section showing how Main Thread resumes work after user provides validation results.

---

## Potential Improvements

### 6. PM Agent Invocation Criteria

**Issue:**
Current scenarios show egdesk-pm-agent invoked for every feature implementation. However:
- Bug fixes typically don't need vision validation
- Refactoring might not need vision check
- Dependency updates don't need PM agent
- Only NEW user-facing features need PM validation

**Current:** PM agent always invoked in development tasks
**Improvement:** PM agent only for:
- ✅ New user-facing features
- ✅ UX/UI changes
- ✅ Architectural shifts
- ❌ Bug fixes (unless architectural)
- ❌ Code refactoring
- ❌ Dependency updates
- ❌ Test additions

**Benefit:** Faster orchestration for non-strategic tasks

**Decision Required:**
Should orchestration guidelines include PM agent invocation criteria?

---

### 7. Sequential Analyzer Dependency Pattern Missing

**Issue:**
Documentation shows parallel analyzer invocation well (Scenario 4, Phase 1) but lacks full walkthrough of sequential analyzer chaining where later agent needs earlier agent's **specific findings** (not just context).

**Existing:** Pattern 3 mentions it abstractly
**Missing:** Concrete scenario with actual file paths and data flow

**Example Missing Scenario:**
```
User: "I'm getting Electron security warning about Theia's preload script"

Phase 1:
electron-analyzer-agent → Analyzes security requirements
Returns: "contextIsolation: true requires preload script to use contextBridge API"

Phase 2 (uses Phase 1 SPECIFIC findings):
Main Thread → theia-analyzer-agent
Prompt: "Given that contextIsolation requires contextBridge API [from Phase 1],
        analyze how Theia's preload script in packages/core/src/electron-browser/preload.ts
        exposes APIs and whether it follows this security pattern."

Agent returns: Specific file locations that violate the pattern
Main Thread: Implements fix
```

**Improvement:**
Add Scenario 8 showing full sequential dependency with specific data passed between agents.

---

### 8. coding-agent Context Size Uncertainty

**Issue:**
Document assumes coding-agent gets 1M context via `model: inherit`, but SDK documentation states:

> "It's unclear from documentation and user testing whether `model: inherit` gives subagents access to the same context window size (e.g., 1M tokens) as the parent conversation, or if they get a fresh context window with the inherited model capabilities"

**Risk:**
If coding-agent doesn't actually get 1M context:
- Might fail on large implementations
- Defeats the purpose of context preservation
- Could hit limits unexpectedly

**Improvement Options:**

**Option A: Test and Document**
- Run experimental coding-agent session with large files
- Monitor if it completes or hits context limits
- Document actual behavior

**Option B: Plan for Limited Context**
- Add fallback: If coding-agent reports context issues, Main Thread implements directly
- Document context limits in coding-agent usage criteria
- Add "chunk implementation" pattern for very large features

**Decision Required:**
Should we test this before documenting it as reliable, or document the uncertainty as a caveat?

---

### 9. Parallel coding-agents Pattern Not Addressed

**Issue:**
Documentation doesn't address whether Main Thread can invoke multiple coding-agents in parallel for independent features.

**Potential Use Case:**
```
User: "Implement both dark mode toggle AND keyboard shortcuts manager"

Main Thread orchestrates:
Phase 1: Get guidance for both features (parallel analyzers)
Phase 2: Implement both features (parallel coding-agents?)

Could be:
Task(agent: "coding-agent", prompt: "Implement dark mode toggle based on [guidance]")
Task(agent: "coding-agent", prompt: "Implement keyboard shortcuts based on [guidance]")

Both run in parallel, separate contexts
```

**Questions:**
- Can two coding-agents run simultaneously?
- Would they conflict with file operations?
- How to name them for tracking (coding-agent-1, coding-agent-2)?

**Improvement:**
Either:
- Document that parallel coding-agents are supported with examples
- Or explicitly state coding-agent must be sequential (one at a time)

---

### 10. Main Thread Context Monitoring Infeasible

**Issue:**
Criteria says "Use coding-agent when Main Thread context > 200k tokens" but Main Thread has no visibility into its current token usage.

**Reality Check:**
Main Thread cannot check `if (myContext > 200000)` - this metric is not accessible.

**Practical Alternative:**
```
Use coding-agent when:
✅ Implementing 3+ files
✅ Large file edits (500+ lines total across files)
✅ Already orchestrated 5+ agent invocations in this conversation
✅ Notice responses getting slower (context pressure)
✅ Multiple features planned in single session
```

**Improvement:**
Replace token-count criteria with observable metrics Main Thread can actually assess.

---

### 11. No Rollback or Fix Pattern

**Issue:**
If implementation fails (build errors, test failures, incorrect behavior), there's no documented pattern for:
- Rolling back changes
- Fixing errors iteratively
- Re-invoking agents with error context

**Missing Scenario:**
```
coding-agent → Implements feature
Main Thread → Builds → ERROR: "Cannot find name 'TerminalWidget'"
Main Thread → Options:
  a) Read error, fix directly
  b) Invoke theia-analyzer-agent: "Why TerminalWidget undefined? Missing import?"
  c) Re-invoke coding-agent with error context
  d) Git reset and restart
```

**Improvement:**
Add Scenario 9: "Fixing Build Errors After Implementation"

---

### 12. Agent Report Format Standardization

**Issue:**
Analyzer agents are told to return "detailed reports" but no standardized format is enforced. Different agents might return:
- Bullet lists
- Prose explanations
- File paths with line numbers
- Code snippets

**Impact:**
- Harder for Main Thread to synthesize
- Inconsistent quality
- May need to re-query agents for clarification

**Improvement:**
Define standard report template:
```markdown
## Analysis: [Topic]

### Files Analyzed:
- path/to/file.ts:45-67
- path/to/other.ts:123

### Findings:
1. [Finding with evidence]
2. [Finding with evidence]

### Recommended Pattern:
[Specific pattern with file reference]

### Example Implementation:
[Code snippet or file reference showing pattern]

### Constraints:
- [Any limitations or warnings]
```

Then update all analyzer agent definitions to require this format.

---

## Summary

**High Priority (Affects Correctness):**
1. Metaphysical separation boundary (Issue #2)
4. Error handling (Issue #4)
5. Decision point continuation (Issue #5)
10. Context monitoring infeasibility (Issue #10)

**Medium Priority (Affects Usability):**
6. PM agent criteria (Issue #6)
7. Sequential analyzer pattern (Issue #7)
11. Rollback pattern (Issue #11)

**Low Priority (Documentation/Edge Cases):**
1. Agent discovery (Issue #1)
3. SDK analyzer examples (Issue #3)
8. Context size uncertainty (Issue #8)
9. Parallel coding-agents (Issue #9)
12. Report format (Issue #12)

**Recommendation:** Address high priority issues first, especially #2 (boundary ambiguity) and #4 (error handling), as these affect the core workflow reliability.
