# PM Research Evaluation Methodology

This methodology is loaded when you execute `/pm-research-evaluation`.

## When to Use This

**Main Thread Request Pattern**:
- "Research complete. Findings in subagent_reports/. Evaluate and recommend."
- "I investigated [options]. Which aligns with vision?"

This loads Pattern G methodology and Type D output format for evaluating research results and recommending technology choices.

## Evaluation Process

### Pattern G: Research Results Evaluation

**Main Thread Request Format**:
```
Previously you requested research on [capability] with criteria [X, Y, Z].

I've investigated and organized findings in subagent_reports/:
- [YYYYMMDD_HHMM_tech-option-A-research.md]: [summary]
- [YYYYMMDD_HHMM_tech-option-B-research.md]: [summary]

Analyzer agents reported:
- [agent findings on integration, performance, etc.]

Evaluate and recommend which option aligns with vision.
```

**Your Response**:

1. **Read organized research**: Main Thread put findings in subagent_reports/ - read them

2. **Evaluate each option against criteria**: Score options on defined criteria

3. **Assess vision alignment**: Which option best fits EG-DESK principles?

4. **Consider integration impact**: Complexity, bundle size, breaking changes

5. **Provide clear recommendation**: Option X recommended because [detailed rationale]

6. **Explain tradeoffs**: Why chosen option is best despite tradeoffs

7. **Update technology-stack.md**: If approved, add new technology to stack doc

8. **Create research PRD**: Document the investigation and decision

**Critical**: You evaluate based on Main Thread's investigation, not your own assumptions. Main Thread did the research, you provide strategic assessment.

## Scoring Framework

Use 1-5 scale (5 = best) to evaluate options against criteria:

| Criterion | Option A | Option B | Option C | Weight | Notes |
|-----------|----------|----------|----------|--------|-------|
| Bundle Size | [score] | [score] | [score] | High | [Key finding] |
| Integration Complexity | [score] | [score] | [score] | High | [Key finding] |
| Performance | [score] | [score] | [score] | Critical | [Key finding] |
| Compatibility | [score] | [score] | [score] | Critical | [Key finding] |
| Maintenance | [score] | [score] | [score] | Medium | [Key finding] |
| **Total** | **[X]** | **[Y]** | **[Z]** | | |

**Scoring weights**:
- **Critical** (5x multiplier): Must meet requirement or option rejected
- **High** (3x multiplier): Significant factor in decision
- **Medium** (1x multiplier): Nice to have, but not deciding factor

## Output Format

### Type D: Research Results Evaluation Report

```markdown
## EG-DESK PM: Research Results Evaluation

### Summary (Concise)
[2-3 sentences: What was researched, which option recommended, key rationale]

### Research Context Recall

**Original Request**:
[What capability was needed]

**Evaluation Criteria**:
[List criteria from Research Planning Report]

**Options Investigated**:
- Option A: [Technology A]
- Option B: [Technology B]
- Option C: [Technology C]

### Research Findings Summary

**Option A: [Technology A]** (from Main Thread's research)
- **Bundle Size**: [X MB - from research doc]
- **Integration Complexity**: [Assessment from analyzer agent]
- **Performance**: [Benchmark results from research]
- **Compatibility**: [Integration findings]
- **Maintenance**: [Community health metrics]
- **Strengths**: [Key advantages]
- **Weaknesses**: [Key limitations]

**Option B: [Technology B]** (from Main Thread's research)
- **Bundle Size**: [Y MB - from research doc]
- **Integration Complexity**: [Assessment from analyzer agent]
- **Performance**: [Benchmark results from research]
- **Compatibility**: [Integration findings]
- **Maintenance**: [Community health metrics]
- **Strengths**: [Key advantages]
- **Weaknesses**: [Key limitations]

**Option C: [Technology C]** (from Main Thread's research)
- **Bundle Size**: [Z MB - from research doc]
- **Integration Complexity**: [Assessment from analyzer agent]
- **Performance**: [Benchmark results from research]
- **Compatibility**: [Integration findings]
- **Maintenance**: [Community health metrics]
- **Strengths**: [Key advantages]
- **Weaknesses**: [Key limitations]

### Evaluation Against Criteria

**Scoring** (1-5 scale, 5 = best):

| Criterion | Option A | Option B | Option C | Weight | Notes |
|-----------|----------|----------|----------|--------|-------|
| Bundle Size | [score] | [score] | [score] | High | [Key finding] |
| Integration Complexity | [score] | [score] | [score] | High | [Key finding] |
| Performance | [score] | [score] | [score] | Critical | [Key finding] |
| Compatibility | [score] | [score] | [score] | Critical | [Key finding] |
| Maintenance | [score] | [score] | [score] | Medium | [Key finding] |
| **Total** | **[X]** | **[Y]** | **[Z]** | | |

**Key Findings**:
- [Most important discovery from research]
- [Critical differentiator between options]
- [Unexpected finding that changes assessment]

### Vision Alignment Assessment

**Option A vs EG-DESK Vision**:
- ✅ **Aligns**: [How it supports ambient AI, spatial canvas principles]
- ⚠️ **Concerns**: [Any tension with vision]
- **Strategic Fit**: [Long-term maintainability, user experience impact]

**Option B vs EG-DESK Vision**:
- ✅ **Aligns**: [How it supports principles]
- ⚠️ **Concerns**: [Any tension]
- **Strategic Fit**: [Assessment]

**Option C vs EG-DESK Vision**:
- ✅ **Aligns**: [How it supports principles]
- ⚠️ **Concerns**: [Any tension]
- **Strategic Fit**: [Assessment]

### Recommendation (Vision-Aligned)

**Recommended Option**: [Technology X]

**Rationale** (Detailed):

**Primary Reasons**:
1. [Reason 1 with evidence from research]
2. [Reason 2 with evidence from research]
3. [Reason 3 with vision alignment argument]

**Tradeoffs Accepted**:
- **Tradeoff 1**: [What we give up] - Acceptable because [why]
- **Tradeoff 2**: [What we give up] - Acceptable because [why]

**Why Not Option B**:
[Specific reason with evidence - e.g., "Bundle size 2x larger, unacceptable for performance goals"]

**Why Not Option C**:
[Specific reason with evidence - e.g., "Custom implementation = 3 months dev time + ongoing maintenance burden"]

**Vision Alignment**:
[How recommended option best supports EG-DESK's ambient AI workspace principles, spatial canvas UX, competitive positioning]

**Integration Strategy**:
- **Phase 1**: Proof of concept (integrate [Technology X] in isolated test)
- **Phase 2**: Infinite Canvas integration (layer with existing 2D canvas)
- **Phase 3**: Production deployment (optimize bundle, performance tuning)

### Documentation Actions Taken

**Technology Stack Updated**:
- File: `ideas&external_references/eg-desk ideas/technology-stack.md`
- Added: **[Technology X]** (3D Rendering, Status: Approved for Integration)
- Capabilities: [List capabilities]
- Integration Notes: [How it works with existing stack]

**Research PRD Created**:
- File: `ideas&external_references/eg-desk ideas/features/[capability]-with-[tech-x]-prd.md`
- Content: Research findings, decision rationale, integration approach

**Research Documents Management** (after user approval):

```bash
# Move approved research to institutional memory
Bash: mv subagent_reports/YYYYMMDD_HHMM_[approved-tech]-research.md ideas&external_references/[approved-tech]-research.md

# Delete rejected research (temporary notes no longer needed)
Bash: rm subagent_reports/YYYYMMDD_HHMM_[rejected-tech-A]-research.md
Bash: rm subagent_reports/YYYYMMDD_HHMM_[rejected-tech-B]-research.md
```

**Result**:
- **Preserved**: `ideas&external_references/[approved-tech]-research.md` (institutional memory)
- **Cleaned**: Deleted rejected options from subagent_reports/
- **Rationale**: Preserve institutional memory of why [approved-tech] chosen over alternatives

### Next Steps for Main Thread

1. **Proceed with integration**: Follow Phase 1-3 integration strategy above
2. **Query framework analyzer**: "How to integrate [Technology X] with Theia webview + Electron renderer?"
3. **Spawn coding-agent**: Implement proof of concept based on analyzer guidance
4. **User decision point**: After POC, confirm approach before full integration
```

## Example

**Request**: "Researched Three.js, Babylon.js. Findings in subagent_reports/. Evaluate."

**Response** (using Type D format above):

**Summary**: Researched Three.js, Babylon.js, Custom WebGL. Recommend Three.js for bundle size (450KB with tree-shaking) + strong TypeScript support.

**Scoring**:
- Three.js: 4.2/5 (bundle size good, integration moderate, performance excellent)
- Babylon.js: 3.5/5 (bundle size large, integration complex, performance excellent)
- Custom WebGL: 2.8/5 (bundle size minimal, dev time prohibitive, maintenance burden)

**Recommendation**: Three.js

**Rationale**:
1. Bundle size meets budget with tree-shaking (450KB vs 500KB target)
2. Proven integration patterns with similar canvas systems
3. Vision alignment: Lightweight, non-intrusive (ambient AI principle)

**Tradeoffs**:
- Slightly larger than Custom WebGL but saves 3 months dev time
- More complex than Babylon.js but 2.5x smaller bundle

**Tech Stack Updated**: Added Three.js to technology-stack.md (3D Rendering category)
