# PM Conflict Resolution Methodology

This methodology is loaded when you execute `/pm-conflict-resolution`.

## When to Use This

**Main Thread Request Pattern**:
- "Found existing feature at [location] that conflicts with user request."
- "User idea conflicts with vision doc [X]. How to resolve?"

This loads Pattern E methodology for resolving vision conflicts and existing implementation conflicts.

## Methodology

### Pattern E: Conflict Resolution

**Main Thread Request Format**:
```
Found existing feature at [location] that partially implements this.

Options:
- Enhance existing feature
- Replace existing with new implementation
- Create separate feature

Vision documents seem to suggest different directions. How should we proceed?
```

**Your Response**:

1. **Analyze existing implementation**: Read the found code, understand intent
   - What was the original purpose?
   - Who created it and when?
   - What design decisions were made?

2. **Check vision docs**: Look for related decisions
   - Are there PRDs or brainstorms related to existing feature?
   - What vision principles guided original implementation?
   - Has vision evolved since original implementation?

3. **Resolve apparent conflict**: Often vision evolved, or documents cover different aspects
   - **Vision evolution**: "Original implementation aligned with vision at that time, but vision has evolved to emphasize [new principle]"
   - **Different aspects**: "Document A covers [aspect X], Document B covers [aspect Y] - both valid, no actual conflict"
   - **True conflict**: "Document A says [X], Document B says [opposite of X] - requires user clarification"

4. **Recommend path forward**: Enhance vs replace vs separate (with clear rationale)
   - **Enhance**: When existing implementation is fundamentally sound, just needs extension
   - **Replace**: When existing implementation conflicts with current vision, clean slate needed
   - **Separate**: When use cases are different enough to warrant independent features

5. **Update docs if needed**: If vision actually conflicts, flag for user to clarify
   - Note the conflict in brainstorming doc
   - Request user decision on which direction to take
   - Update vision docs once user clarifies

## Conflict Types

### Type 1: Existing Implementation Conflict

**Scenario**: User requests feature that overlaps with existing implementation

**Process**:
1. Read existing implementation code
2. Identify overlap vs unique aspects
3. Assess quality and alignment of existing code
4. Decide: Enhance / Replace / Separate

**Decision Matrix**:
- **Enhance if**:
  - Existing code is well-designed
  - User request is superset of existing feature
  - Minimal breaking changes needed

- **Replace if**:
  - Existing code is poorly designed or outdated
  - User request fundamentally different approach
  - Clean slate simpler than refactoring

- **Separate if**:
  - Different use cases (both features valuable)
  - Minimal overlap in functionality
  - User explicitly wants both options

### Type 2: Vision Document Conflict

**Scenario**: Multiple vision documents seem to contradict each other

**Process**:
1. Read both/all conflicting documents
2. Check timestamps (vision may have evolved)
3. Analyze context (different aspects vs actual conflict)
4. Resolve or escalate to user

**Resolution Strategies**:
- **Evolution**: "Document A (older) said X, Document B (newer) says Y - vision evolved, use Document B"
- **Context**: "Document A applies to [context X], Document B applies to [context Y] - both valid"
- **True Conflict**: "Documents genuinely conflict - user must decide which direction to take"

### Type 3: User Request vs Vision Conflict

**Scenario**: User request contradicts documented vision principles

**Process**:
1. Identify which vision principle is violated
2. Understand user's intent and reasoning
3. Assess if vision should evolve or user should adjust
4. Provide insight (not rejection)

**Response Framework**:
```markdown
**Conflict Identified**:
- User request: [What user wants]
- Vision principle: [Which principle conflicts]
- Tension: [Why they conflict]

**Analysis**:
- User's reasoning: [Why user thinks this is valuable]
- Vision's reasoning: [Why principle exists]

**Options**:

1. **Modify Request** (Preserve Vision):
   - Adjusted approach: [Alternative that aligns with vision]
   - Achieves user goal: [How it meets user's need]
   - Maintains vision: [How it preserves principle]

2. **Evolve Vision** (If User Has Strong Rationale):
   - User insight: [What user sees that vision missed]
   - Vision evolution: [How principle could be refined]
   - Updated approach: [New direction that incorporates both]

3. **User Decision Required**:
   - Question: [What user must clarify]
   - Context: [Information to help user decide]

**Recommendation**: [Which option and why]
```

## Output Format

**Conflict analysis + recommended resolution + rationale**

```markdown
## EG-DESK PM: Conflict Resolution

### Summary
[2-3 sentences: What conflict exists, recommended resolution]

### Conflict Analysis

**Conflict Type**: [Existing Implementation / Vision Document / User Request vs Vision]

**Parties in Conflict**:
- Party A: [Existing feature / Document A / User request] - Position: [What it says/wants]
- Party B: [User request / Document B / Vision principle] - Position: [What it says/wants]

**Nature of Conflict**:
[Describe why they conflict - technical, philosophical, timing]

### Investigation Results

**Code Analysis** (if applicable):
- File: [path to existing implementation]
- Purpose: [Original intent]
- Quality: [Assessment of implementation]
- Overlap: [How much overlaps with user request]

**Vision Document Analysis** (if applicable):
- Document A: [Path and key principles]
- Document B: [Path and key principles]
- Timeline: [When each was written]
- Context: [What each document covers]

**Conflict Resolution**:
- **If Evolution**: "Vision evolved from [old] to [new], use latest direction"
- **If Context**: "Both valid in different contexts: [explain contexts]"
- **If True Conflict**: "Genuine conflict exists, requires user clarification"

### Recommended Resolution

**Recommendation**: [ENHANCE / REPLACE / SEPARATE / EVOLVE VISION / USER CLARIFY]

**Rationale**:
1. [Primary reason with evidence]
2. [Secondary reason with evidence]
3. [Vision alignment argument]

**Implementation Path** (if ENHANCE/REPLACE/SEPARATE):
- **Phase 1**: [First step]
- **Phase 2**: [Next step]
- **Integration**: [How it fits with existing code/vision]

**User Decision Needed** (if USER CLARIFY):
- **Question**: [What user must decide]
- **Context**: [Information to help decision]
- **Options**:
  - Option A: [Description and implications]
  - Option B: [Description and implications]

### Documentation Updates Needed

**If vision conflict resolved**:
- Update: [Which document needs updating]
- Changes: [What to add/change]
- Rationale: [Why this resolves conflict]

**If user clarification needed**:
- Create brainstorm: [Note conflict for future reference]
- Flag: [Which documents need user input]
```

## Example

**Request**: "Found existing feature at packages/terminal/theme-switcher.ts. User wants time-based theme. Enhance or replace?"

**Response**:

## EG-DESK PM: Conflict Resolution

**Summary**: Existing manual theme switcher found. User wants automatic time-based switching. Recommend ENHANCE - add time-based mode to existing switcher.

**Conflict**: Existing (manual switching) vs User Request (automatic time-based)

**Analysis**:
- Existing code: Well-designed manual switcher with preference persistence
- User request: Add automatic mode based on time of day
- Overlap: Both switch themes, different triggering mechanisms

**Recommendation**: ENHANCE

**Rationale**:
1. Existing code is high quality, no need to replace
2. User request is superset (automatic + manual)
3. Vision alignment: Give users choice (manual override + automatic default)

**Implementation**:
- Phase 1: Add time-based detection service
- Phase 2: Integrate with existing switcher (add "auto" mode)
- Phase 3: Preference UI (manual / auto toggle)
