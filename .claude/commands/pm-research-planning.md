# PM Research Planning Methodology

This methodology is loaded when you execute `/pm-research-planning`.

## When to Use This

**Main Thread Request Pattern**:
- "User wants [capability]. Does current stack support this?"
- "Current tech X insufficient for Y. Research alternatives?"

This loads Pattern F methodology and Type C output format for tech gap assessment and research planning.

## Limitation Diagnosis Framework

**Core Competency**: Diagnose when current stack has limitations for requirements

**Process**:
1. **Identify user requirement**: What specific capability is needed?
2. **Analyze current stack**: What technologies are currently used for related functionality?
3. **Assess capability gap**: Why does current tech NOT meet requirement?
4. **Articulate research necessity**: Explain WHY investigation needed (not just "we need X")

**Example**:
- Requirement: "True 3D object rendering with lighting and shadows"
- Current tech: "Konva.js for 2D canvas rendering"
- Limitation: "Konva.js is 2D-only, cannot render true 3D with lighting/shadows"
- Research needed: "User needs true 3D manipulation with realistic lighting. Konva.js pseudo-3D (perspective tricks) insufficient for complex 3D scenes."

## Research Planning Process

### Pattern F: Research Planning

**Main Thread Request Format**:
```
User wants [feature with specific capabilities].

Current technology stack analysis:
- [Current tech X] used for [purpose]
- User requirement: [specific capability needed]

Does current stack support this? Or do we need new technology research?
```

**Your Response**:

1. **Diagnose current stack limitations**: "Current framework X has Y limitation for Z requirement"

2. **Articulate research necessity**: Explain WHY new framework investigation needed (not just "we need X")

3. **Define evaluation criteria**: Detailed criteria for Main Thread to use:
   - Bundle size impact (< X MB preferred)
   - Integration complexity (must work with current stack: Infinite Canvas, Konva.js, etc.)
   - Performance (render Y objects at Z FPS)
   - Learning curve (team familiarity)
   - License compatibility
   - Active maintenance and community

4. **Structure investigation scope**: What questions need answering?
   - "Can framework A achieve [specific capability]?"
   - "What's the integration approach with [existing tech]?"
   - "What's the performance profile for [use case]?"

5. **Request multi-option comparison**: "Investigate options: [Option A], [Option B], [Option C]"

6. **Design for parallel execution**: Structure questions so Main Thread can run multiple agents simultaneously

**Critical**: You do NOT choose which framework. You diagnose the gap and define investigation criteria. Main Thread will investigate and return findings.

## Output Format

### Type C: Research Planning Report

```markdown
## EG-DESK PM: Research Planning

### Summary (Concise)
[2-3 sentences: What capability is needed, why current stack insufficient, research scope requested]

### Current Stack Limitation Diagnosis

**User Requirement**:
[Specific capability user requested - e.g., "true 3D object rendering with lighting and shadows"]

**Current Stack Analysis**:
- **Technology in Use**: [Current tech - e.g., "Konva.js for 2D canvas rendering"]
- **Capabilities**: [What current tech can do]
- **Limitation**: [Why it cannot meet requirement - e.g., "Konva.js is 2D-only, cannot render true 3D with lighting/shadows"]
- **Gap**: [Specific missing capability]

**Why Research Needed**:
[Clear articulation - not just "we need X", but WHY current approach won't work]
- Example: "User needs true 3D object manipulation with realistic lighting. Konva.js pseudo-3D (perspective tricks) insufficient for complex 3D scenes with multiple light sources and shadows."

### Research Investigation Scope

**Evaluation Criteria** (for Main Thread to use):

**Criterion 1: Bundle Size Impact**
- Target: < 500 KB additional (gzipped)
- Critical: Avoid bloating application bundle
- Measure: npm package size + dependencies

**Criterion 2: Integration Complexity**
- Must work with: Infinite Canvas (viewport transforms), Konva.js (2D layer)
- Integration point: How to layer 3D rendering with existing 2D canvas?
- API complexity: Learning curve for team

**Criterion 3: Performance**
- Target: Render 1000+ 3D objects at 60 FPS
- Critical: Must not degrade existing 2D canvas performance
- Measure: Benchmark with realistic scene

**Criterion 4: Compatibility with Existing Stack**
- Must integrate with: Theia webview, Electron renderer process
- Security: Compatible with Electron contextIsolation
- Dependencies: No conflicts with existing packages

**Criterion 5: Maintenance & Community**
- Active development (commits in last 6 months)
- Strong community support (GitHub stars, Stack Overflow)
- TypeScript support (type definitions quality)

**Investigation Questions** (Main Thread will research):

**Question Set 1: [Option A - e.g., Three.js]**
- Can [Option A] achieve [specific capability]?
- What's the bundle size (core + required modules)?
- How does [Option A] integrate with Infinite Canvas viewport transforms?
- Performance profile for [use case]?
- TypeScript support quality?

**Question Set 2: [Option B - e.g., Babylon.js]**
- Can [Option B] achieve [specific capability]?
- What's the bundle size (core + required modules)?
- How does [Option B] integrate with Infinite Canvas viewport transforms?
- Performance profile for [use case]?
- TypeScript support quality?

**Question Set 3: [Option C - e.g., Custom WebGL]**
- Feasibility of custom WebGL solution?
- Development time estimate?
- Maintenance burden?

### Parallel Investigation Design

**Structure for Simultaneous Execution**:

Main Thread should run **3 parallel investigations**:

**Investigation 1: [Option A]** (agent: general-purpose + WebSearch)
- Research [Option A] capabilities, bundle size, integration approach
- Organize findings in: `subagent_reports/YYYYMMDD_HHMM_[option-a]-research.md`
- **IMPORTANT**: Write to subagent_reports/ first (temporary workspace)
- PM will review and move to ideas&external_references/ after user approval
- Rationale: Prevents polluting institutional memory with rejected research

**Investigation 2: [Option B]** (agent: general-purpose + WebSearch)
- Research [Option B] capabilities, bundle size, integration approach
- Organize findings in: `subagent_reports/YYYYMMDD_HHMM_[option-b]-research.md`
- **IMPORTANT**: Write to subagent_reports/ first (temporary workspace)
- PM will review and move to ideas&external_references/ after user approval
- Rationale: Prevents polluting institutional memory with rejected research

**Investigation 3: [Option C]** (agent: general-purpose)
- Assess feasibility of [Option C] implementation
- Organize findings in: `subagent_reports/YYYYMMDD_HHMM_[option-c]-research.md`
- **IMPORTANT**: Write to subagent_reports/ first (temporary workspace)
- PM will review and move to ideas&external_references/ after user approval
- Rationale: Prevents polluting institutional memory with rejected research

**All 3 can run simultaneously** - no dependencies between investigations.

### Expected Outcome

**Main Thread will return with**:
- 3 research documents organized in `subagent_reports/`
- Analyzer agent reports on integration complexity, performance
- Request for PM evaluation: "Which option aligns with vision?"

**Then you will**:
- Read all research documents
- Evaluate against criteria
- Provide vision-aligned recommendation
- Update technology-stack.md if approved

### Next Steps for Main Thread

1. **Investigate options in parallel**: Run 3 agents simultaneously to research options
2. **Organize findings**: Create research documents in `subagent_reports/` (temporary workspace)
3. **Query analyzer agents**: Get technical assessment of integration complexity
4. **Return to PM**: Present findings for evaluation and recommendation
```

## Example

**Request**: "User wants 3D rendering. Konva.js is 2D-only. Research needed?"

**Response** (using Type C format above):

**Summary**: User needs true 3D rendering with lighting/shadows. Konva.js is 2D-only. Research Three.js, Babylon.js, Custom WebGL.

**Current Stack Limitation**:
- Konva.js: 2D canvas only
- User needs: True 3D with realistic lighting
- Gap: Cannot render 3D objects with multiple light sources and shadows

**Research Scope**: Investigate 3 options with criteria:
1. Bundle size < 500KB
2. Integration with Infinite Canvas
3. Performance: 1000+ objects @ 60 FPS
4. Compatibility with Theia webview
5. Active maintenance

**Parallel Investigations**: Three.js, Babylon.js, Custom WebGL (all simultaneous)
