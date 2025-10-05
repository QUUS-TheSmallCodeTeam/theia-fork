# Claude Code Subagent Best Practices

## Core Principles

### 1. Single Responsibility
- Give each subagent ONE clear, specific goal
- Avoid making one subagent do everything
- Keep descriptions action-oriented (e.g., "Use after spec exists; produce ADR and guardrails")

### 2. Write Detailed Prompts
- Include specific instructions, examples, and constraints
- More guidance = better performance
- Define the role, capabilities, and approach clearly

## Structure & Format

```yaml
---
name: descriptive-subagent-name
description: Concise, action-oriented description. Use PROACTIVELY if needed.
tools:
  - Read
  - Bash
  # Only list tools actually needed
model: inherit  # or haiku/sonnet/opus based on complexity
---

# Role and Expertise
Clear description of the subagent's specialized domain.

## When to Use
- Specific trigger conditions
- Clear handoff points

## Process
1. Step-by-step workflow
2. Specific actions to take
3. Clear output format

## Constraints
- What NOT to do
- Boundaries and limitations
- Quality criteria/checklist
```

## Best Practices

### Tool Management
- ⚠️ **Omitting `tools` = access to ALL tools** - be intentional
- Only grant necessary permissions
- Improves security and focus

### Proactive Delegation
- Use "PROACTIVELY" in description if agent should self-invoke
- Make triggers explicit
- Define clear "Definition of Done"

### Model Selection
- **haiku**: Quick, focused tasks with minimal reasoning
- **sonnet**: Standard development (default if omitted)
- **opus**: Complex reasoning/critical analysis
- **inherit**: Uses the same model as the main conversation (recommended for consistency)

## Example Patterns

### Good Description:
```
Use PROACTIVELY after test failures to analyze root cause and suggest fixes
```

### Good Process:
```
1. Read test output and error messages
2. Locate relevant source code
3. Identify root cause
4. Propose minimal fix
5. Return fix with explanation
```

### Good Constraints:
```
- Only modify test-related files
- Preserve existing test coverage
- Don't add new dependencies
```

## Workflow Tips

- Create modular, multi-stage pipelines
- Implement clear handoff points between agents
- Keep humans in the loop for critical decisions
- Use hooks for agent transitions
- Track status (e.g., `READY_FOR_REVIEW`, `READY_FOR_BUILD`)

## Context Available to Subagents

Subagents receive the following context when invoked:

1. **Working directory**: Current project path
2. **Git information**:
   - Whether it's a git repo
   - Current branch and main branch
   - Full git status (deleted, modified, untracked files)
   - Recent commit history (5 commits with hashes and messages)
3. **Platform info**: OS platform and version
4. **Current date**: Today's date
5. **Model information**: Model ID and token budget
6. **Knowledge cutoff**: Model's knowledge cutoff date

### What Subagents DON'T Get:
- Previous conversation history with the parent agent
- User interaction history across sessions
- Context from earlier conversations
- Any memory of prior tasks

Each subagent starts fresh with only the prompt you give it via the Task tool, plus the environment snapshot.

### Context Window & Model Inheritance

- When using `model: inherit`, the subagent uses the same model as the main conversation
- Each subagent has its own independent context window
- **Note**: It's unclear from documentation and user testing whether `model: inherit` gives subagents access to the same context window size (e.g., 1M tokens) as the parent conversation, or if they get a fresh context window with the inherited model capabilities
- Multiple subagents can run in parallel, each with their own context, allowing you to effectively gain additional context capacity for large codebases

## Quick Start

### Recommended Approach
Have Claude generate your initial subagent, then customize it:

```
"Create a subagent that reviews TypeScript code for common anti-patterns"
```

Then iterate on the generated agent to make it yours.

## Resources

- [Official Claude Code Subagents Documentation](https://docs.claude.com/en/docs/claude-code/sub-agents)
- [Claude Code Best Practices](https://www.anthropic.com/engineering/claude-code-best-practices)
- [Community Subagent Examples](https://github.com/wshobson/agents)
- [Production-Ready Subagents Collection](https://github.com/VoltAgent/awesome-claude-code-subagents)
