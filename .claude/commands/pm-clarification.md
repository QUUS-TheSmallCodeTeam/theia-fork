# PM Clarification Methodology

This methodology is loaded when you execute `/pm-clarification`.

## When to Use This

**Main Thread Request Pattern**:
- "Your guide said [X]. I'm unclear on [specific part]."
- "Does 'use Theia' mean modify packages/ or create custom service?"

This loads Pattern B methodology for providing focused clarifications on previous guidance.

## Methodology

### Pattern B: Clarification

**Main Thread Request Format**:
```
Your guide said: "[SPECIFIC PART OF GUIDE]"

I'm unclear on:
- Does "use Theia" mean modify packages/terminal/ or create custom service in eg-desk_taehwa/?
- Should this be Phase 1 or Phase 2?

Please clarify.
```

**Your Response**:

1. **Re-read context**: Review what you said and what Main Thread understood
   - Read your previous strategic guide (from file-based report or MT's quote)
   - Identify the ambiguous statement
   - Understand what Main Thread is confused about

2. **Identify ambiguity**: Which part was unclear?
   - **Terminology ambiguity**: "Use Theia" could mean multiple things
   - **Scope ambiguity**: "Integrate with X" - how deeply?
   - **Location ambiguity**: "Modify terminal" - packages/ or custom code?
   - **Timing ambiguity**: "After setup" - Phase 1 or Phase 2?

3. **Provide specific clarification**: Clear, unambiguous direction
   - **Be concrete**: Use exact file paths, not vague references
   - **Be decisive**: "Do X, not Y" rather than "X or Y depending on..."
   - **Be actionable**: Main Thread should know exactly what to do next

4. **Give concrete examples**: "Create custom service at eg-desk_taehwa/terminal/time-based-switcher.ts"
   - **File paths**: Full absolute paths when possible
   - **Code structure**: Class names, method signatures
   - **Integration points**: Specific services to inject

5. **Reference vision**: "Per whitepaper section X, we prefer custom services over framework modifications"
   - **Cite specific docs**: Help Main Thread understand the "why"
   - **Explain principle**: Clarify the guiding principle behind direction
   - **Build understanding**: Not just "do this", but "do this because..."

## Clarification Types

### Type 1: Code Location Ambiguity

**Ambiguous Statement**: "Modify the terminal package"

**Clarification**:
```markdown
**Clarification**: Create custom service, do NOT modify Theia framework code.

**Specific Path**: `eg-desk_taehwa/terminal/time-based-theme-switcher.ts`

**Rationale** (per EG-DESK vision):
- We prefer custom services over framework modifications (easier to maintain, clearer separation)
- Only modify `packages/` when absolutely necessary (framework bug fixes)
- Custom services allow us to iterate without upstream conflicts

**Integration**: Inject Theia's `TerminalThemeService` from `packages/terminal/` into your custom service
```

### Type 2: Phasing Ambiguity

**Ambiguous Statement**: "Implement preference integration after basic setup"

**Clarification**:
```markdown
**Clarification**: Preference integration is **Phase 2**, after Phase 1 (basic theme switching).

**Phase 1 Scope**:
- Implement TimeBasedThemeSwitcher service
- Basic theme switching logic (no preferences yet)
- Hardcoded time thresholds (6am = light, 6pm = dark)

**Phase 2 Scope**:
- Add PreferenceService integration
- User-configurable time thresholds
- Manual override toggle

**Rationale**: Phase 1 validates core concept with minimal dependencies. Phase 2 adds user control.
```

### Type 3: Integration Approach Ambiguity

**Ambiguous Statement**: "Integrate with theme service"

**Clarification**:
```markdown
**Clarification**: Inject `TerminalThemeService` and call its `setTheme()` method.

**Concrete Implementation**:
```typescript
@injectable()
export class TimeBasedThemeSwitcher {
  @inject(TerminalThemeService)
  protected readonly themeService: TerminalThemeService;

  protected applyTheme(themeId: string): void {
    this.themeService.setTheme(themeId);
  }
}
```

**Rationale**: Use existing Theia service rather than reimplementing theme switching logic. Single responsibility: Your service determines WHICH theme, Theia service applies it.
```

### Type 4: Scope Ambiguity

**Ambiguous Statement**: "Add AI integration"

**Clarification**:
```markdown
**Clarification**: For Phase 1, AI integration means **calling Claude API for code analysis only** - NOT full conversational AI.

**In Scope** (Phase 1):
- Send code snippet to Claude API
- Receive structured analysis response
- Display results in panel

**Out of Scope** (Future phases):
- Conversational chat interface
- Context management across sessions
- Multi-turn AI interactions

**Rationale**: Phase 1 validates AI-assisted analysis workflow. Full conversational features come later after validating core value.
```

## Output Format

**Focused clarification (not full report), reference previous guide**

```markdown
## PM Clarification

### Original Statement (From Previous Guide)
"[Quote the ambiguous statement from your previous guide]"

### Main Thread Confusion
[What Main Thread is unclear about]

### Clarification

**Clear Answer**: [Direct, unambiguous answer to the question]

**Concrete Details**:
- **If code location**: Full file path: `eg-desk_taehwa/[feature]/[file].ts`
- **If phasing**: Phase X includes [specific scope], Phase Y includes [other scope]
- **If integration**: [Exact code pattern or service to use]
- **If scope**: [What's in scope vs out of scope]

**Example** (if helpful):
```typescript
// Concrete code example demonstrating clarification
```

**Rationale** (Reference Vision):
- Vision document: [cite specific doc and section]
- Principle: [explain guiding principle]
- Why this approach: [connect to vision]

### Updated Guidance (If Needed)

**If clarification reveals original guide was unclear**, provide brief updated summary:

**Revised Direction**: [Clearer statement of what to do]

**No change to overall strategy** - just clarifying execution details.

### Next Steps for Main Thread

1. [Specific first action now that ambiguity is resolved]
2. [Return to implementation / Continue with Phase X]
```

## Example

**Request**: "Your guide said 'use Theia'. Does this mean modify packages/terminal/ or create custom service?"

**Response**:

## PM Clarification

**Original Statement**: "Use Theia's theme system for terminal theming"

**Main Thread Confusion**: Unclear if this means modifying packages/terminal/ (Theia framework) or creating custom service in eg-desk_taehwa/

**Clarification**:

**Clear Answer**: Create custom service in eg-desk_taehwa/, do NOT modify packages/terminal/

**Concrete Details**:
- **File path**: `eg-desk_taehwa/terminal/time-based-theme-switcher.ts`
- **Integration**: Inject Theia's `TerminalThemeService` from packages/terminal/
- **Pattern**: Custom service wraps/extends Theia service, doesn't replace it

**Example**:
```typescript
// File: eg-desk_taehwa/terminal/time-based-theme-switcher.ts
@injectable()
export class TimeBasedThemeSwitcher {
  @inject(TerminalThemeService) // Use Theia's service
  protected readonly themeService: TerminalThemeService;

  // Your custom logic
}
```

**Rationale** (Per Vision):
- **Vision doc**: architecture-decisions.md - "Prefer composition over modification"
- **Principle**: Keep framework code clean, put custom logic in eg-desk_taehwa/
- **Why**: Easier to maintain, clearer separation, no upstream conflicts

**Next Steps**:
1. Create `eg-desk_taehwa/terminal/` directory
2. Implement TimeBasedThemeSwitcher service
3. Register in terminal-frontend-module.ts
