# PM Decision Support Methodology

This methodology is loaded when you execute `/pm-decision-support`.

## When to Use This

**Main Thread Request Pattern**:
- "Framework agents suggested 3 approaches: [A, B, C]. Which one?"
- "Multiple valid implementations possible. Which aligns with vision?"

This loads Pattern D methodology for vision-based option evaluation when multiple technically valid choices exist.

## Methodology

### Pattern D: Decision Support

**Main Thread Request Format**:
```
Framework agents suggested 3 approaches:
A) [Approach A with tradeoffs]
B) [Approach B with tradeoffs]
C) [Approach C with tradeoffs]

Which aligns best with EG-DESK vision?
```

**Your Response**:

1. **Evaluate each option against vision**: Which principles does each support/violate?
   - Ambient AI workspace principles (non-intrusive, context-aware)
   - Spatial canvas UX (spatial metaphors, proximity-based)
   - Technical excellence (maintainability, performance)

2. **Consider strategic fit**: Long-term maintainability, user experience, architectural consistency
   - **Maintainability**: Which option easiest to evolve over time?
   - **User Experience**: Which option best serves knowledge workers?
   - **Architecture**: Which option fits existing patterns?

3. **Recommend**: Clear choice with detailed rationale
   - **Primary choice** with confidence level (High/Medium/Low)
   - **Evidence** from vision documents and technical analysis
   - **Specific reasons** why this option is best fit

4. **Explain tradeoffs**: Why chosen option is best despite tradeoffs
   - **What we gain**: Primary benefits of chosen option
   - **What we give up**: Tradeoffs we're accepting
   - **Why acceptable**: Rationale for accepting tradeoffs

5. **Provide fallback**: "If X constraint appears, switch to option B"
   - **Trigger condition**: What would invalidate primary choice
   - **Fallback option**: Alternative if constraint appears
   - **Decision point**: When to reassess

## Vision-Based Evaluation Framework

### Step 1: Extract Vision Principles

For each option, assess against core EG-DESK principles:

**Ambient AI Workspace**:
- Non-intrusive: Does it stay out of the way until needed?
- Context-aware: Does it adapt to user's current task?
- Assistive: Does it augment human capability vs replacing it?

**Spatial Canvas UX**:
- Spatial metaphors: Does it leverage spatial relationships?
- Proximity-based: Does it use nearness for relevance?
- Ephemeral: Does it allow temporary, exploratory work?

**Technical Excellence**:
- Maintainable: Can team easily evolve this over time?
- Performant: Does it meet performance requirements?
- Consistent: Does it follow existing architectural patterns?

### Step 2: Score Options

Use simple scoring:
- ✅ **Strong Support** (+2): Directly enables vision principle
- ✓ **Supports** (+1): Compatible with principle
- − **Neutral** (0): Neither supports nor violates
- ✗ **Conflicts** (−1): Somewhat contradicts principle
- ❌ **Violates** (−2): Directly contradicts principle

### Step 3: Strategic Fit Assessment

**Long-term Maintainability**:
- Code complexity: Simple vs complex
- Team familiarity: Known patterns vs new patterns
- Evolution path: Easy to extend vs rigid

**User Experience Impact**:
- Learning curve: Intuitive vs requires training
- Task flow: Smooth vs disruptive
- Delight factor: Exceeds expectations vs meets them

**Architectural Consistency**:
- Pattern alignment: Matches existing code vs new pattern
- Dependency impact: Minimal vs heavy dependencies
- Integration complexity: Simple vs complex

## Output Format

**Decision recommendation with vision-based rationale**

```markdown
## EG-DESK PM: Decision Support

### Summary
[2-3 sentences: Options compared, recommended choice, key rationale]

### Options Analysis

**Option A: [Name/Description]**
- **Approach**: [How it works]
- **Strengths**: [Key advantages]
- **Weaknesses**: [Key limitations]
- **Tradeoffs**: [What you gain vs what you lose]

**Option B: [Name/Description]**
- **Approach**: [How it works]
- **Strengths**: [Key advantages]
- **Weaknesses**: [Key limitations]
- **Tradeoffs**: [What you gain vs what you lose]

**Option C: [Name/Description]**
- **Approach**: [How it works]
- **Strengths**: [Key advantages]
- **Weaknesses**: [Key limitations]
- **Tradeoffs**: [What you gain vs what you lose]

### Vision Alignment Scoring

| Principle | Option A | Option B | Option C | Rationale |
|-----------|----------|----------|----------|-----------|
| **Ambient AI** |  |  |  |  |
| Non-intrusive | [✅/✓/−/✗/❌] | [score] | [score] | [Why scored this way] |
| Context-aware | [score] | [score] | [score] | [Reasoning] |
| Assistive | [score] | [score] | [score] | [Reasoning] |
| **Spatial Canvas** |  |  |  |  |
| Spatial metaphors | [score] | [score] | [score] | [Reasoning] |
| Proximity-based | [score] | [score] | [score] | [Reasoning] |
| **Technical** |  |  |  |  |
| Maintainable | [score] | [score] | [score] | [Reasoning] |
| Performant | [score] | [score] | [score] | [Reasoning] |
| Consistent | [score] | [score] | [score] | [Reasoning] |
| **Total** | **[+X]** | **[+Y]** | **[+Z]** |  |

### Strategic Fit Analysis

**Long-term Maintainability**:
- Option A: [Assessment - Simple/Moderate/Complex]
- Option B: [Assessment]
- Option C: [Assessment]
- **Winner**: [Which option, why]

**User Experience Impact**:
- Option A: [Assessment - Excellent/Good/Acceptable/Poor]
- Option B: [Assessment]
- Option C: [Assessment]
- **Winner**: [Which option, why]

**Architectural Consistency**:
- Option A: [Assessment - Perfect fit/Good fit/New pattern]
- Option B: [Assessment]
- Option C: [Assessment]
- **Winner**: [Which option, why]

### Recommendation

**Recommended Option**: [Option X]

**Confidence Level**: [High / Medium / Low]

**Primary Rationale**:
1. **Vision Alignment** (Score: [+X]): [How it best supports EG-DESK principles]
2. **Strategic Fit**: [Specific advantage - maintainability/UX/architecture]
3. **Tradeoffs Acceptable**: [Why tradeoffs are acceptable given vision]

**Detailed Reasoning**:
- [Elaboration on why this option is best choice]
- [Specific vision principles it enables]
- [Long-term strategic benefits]

**Tradeoffs Accepted**:
- **Give up**: [What we sacrifice by choosing this option]
- **Acceptable because**: [Why this tradeoff aligns with vision]
- **Mitigation**: [How to minimize impact of tradeoff]

**Why Not Option B**:
- [Specific reason with vision/strategic argument]
- [Evidence from scoring or analysis]

**Why Not Option C**:
- [Specific reason with vision/strategic argument]
- [Evidence from scoring or analysis]

### Fallback Plan

**Trigger Condition**: [What would invalidate primary recommendation]
- Example: "If Option A performance testing reveals < 30 FPS with 1000 objects"

**Fallback Option**: [Option B/C]
- **Why fallback makes sense**: [Rationale given trigger condition]
- **Decision point**: [When to reassess - after POC / after Phase 1 / etc.]

### Next Steps for Main Thread

1. **Proceed with recommended option**: [Specific implementation guidance]
2. **Validate assumptions**: [What to test/verify before committing fully]
3. **Monitor for trigger conditions**: [What to watch for that would require fallback]
```

## Example

**Request**: "Framework agents suggested 3 approaches: A) Theia contribution, B) Custom service, C) Electron main process. Which one?"

**Response**:

## EG-DESK PM: Decision Support

**Summary**: Compared 3 implementation approaches. Recommend Option B (Custom service) for vision alignment (ambient AI, non-intrusive) + maintainability.

**Vision Scoring**:
- Option A (Theia contribution): +3 (good technical fit, but framework coupling)
- Option B (Custom service): +7 (strong vision alignment, maintainable, flexible)
- Option C (Electron main): +1 (performant but rigid, poor UX)

**Recommendation**: Option B (Custom service)

**Rationale**:
1. Vision alignment: Non-intrusive, easily extended with AI features
2. Maintainability: Decoupled from framework, team can iterate quickly
3. Tradeoffs acceptable: Slight performance overhead vs flexibility

**Fallback**: If performance < 30 FPS → Switch to Option C (Electron main)
