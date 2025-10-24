# PM Plan Review Methodology

This methodology is loaded when you execute `/pm-plan-review`.

## When to Use This

**Main Thread Request Pattern**:
- "I created this plan: [plan]. Review against vision."
- "Framework agents reported [findings]. Does my plan incorporate correctly?"

This loads Type B consultation methodology for validating execution plans.

## Methodology

**When Main Thread consults**: "I created this plan: [detailed plan]. Review it against vision and project structure."

**Your process**:

### Step 1: Understand the Plan
1. Read Main Thread's proposed execution plan
2. Identify planned phases, agent queries, implementation steps
3. Note framework agent reports included (if any)

### Step 2: Validate Against Vision
1. Check if plan aligns with vision documents
2. Verify framework choices match architectural decisions
3. Ensure code locations follow project structure

### Step 3: Assess Completeness
1. Are all necessary phases included?
2. Are dependencies properly sequenced?
3. Are there missing considerations?
4. Are framework agents being queried appropriately?

### Step 4: Review Framework Agent Reports (if provided)
1. Check if reports reveal new insights requiring plan adjustment
2. Validate that plan incorporates agent findings correctly
3. Identify any conflicts between agent reports and vision

### Step 5: Provide Plan Review
- **Assessment**: Plan is solid / Needs revision / Major issues
- **Gaps identified**: Missing steps or considerations
- **Vision conflicts**: Any divergence from documented direction
- **Recommendations**: Specific improvements to the plan
- **Approval**: Can proceed / Revise first / Consult user

## Multi-turn Pattern: Plan Review

**Main Thread Request Format**:
```
Previously you provided this strategic guide:
[QUOTES YOUR ENTIRE PREVIOUS GUIDE]

I've created this execution plan:
[DETAILED PLAN]

Framework agents reported:
[AGENT FINDINGS]

Review this plan against vision and suggest improvements.
```

**Your Response**:
1. **Validate against vision**: Check if plan aligns with documented principles
2. **Assess completeness**: Are all necessary steps included? Dependencies correct?
3. **Review agent findings**: Do reports reveal insights requiring plan adjustment?
4. **Identify gaps**: Missing steps, unconsidered edge cases, unclear requirements
5. **Provide specific recommendations**: "Add Phase 2.5 for X", "Query Y agent about Z"
6. **Approval decision**: PROCEED / REVISE FIRST / CONSULT USER

## Output Format

### Type B: Plan Review Report (Plan Validation)

```markdown
## EG-DESK PM: Plan Review

### Summary (Concise)
[2-3 sentences: Plan assessment, major findings, approval status]

### Plan Assessment

**Overall Evaluation**: [Solid / Needs Minor Revision / Needs Major Revision / Reject]

**Vision Alignment**:
✅ **Aligned aspects:**
- [How plan aligns with vision]

⚠️ **Concerns:**
- [Any vision conflicts or risks]

**Completeness Check**:
✅ **Well-covered:**
- [Phases/steps that are well-planned]

❌ **Gaps identified:**
- [Missing steps or considerations]

### Framework Agent Report Review (if provided)

**Reports Analyzed**:
- theia-analyzer-agent report: [Key findings]
- electron-analyzer-agent report: [Key findings]

**Insights from Reports**:
- [Important insight that should adjust plan]
- [Pattern discovered that changes approach]

**Plan Integration**:
✅ [Plan correctly incorporates report X]
⚠️ [Plan misses insight Y from report Z - needs adjustment]

### Recommendations (Actionable)

**Plan Revisions Needed**:
1. [Specific change 1 with rationale]
2. [Specific change 2 with rationale]

**Additional Queries Suggested**:
- [Framework agent to query for missing info]
- [Specific question to ask that agent]

**User Clarification Needed** (if applicable):
- [Question for user that requires decision]

**Approval Status**: [PROCEED / REVISE FIRST / CONSULT USER]

### References for Main Thread

**Vision Documents Referenced**:
- [Docs used to validate plan]

**Project Structure Verified**:
- [Packages/files checked against plan]
```

## Example

**Request**: Review plan with agent findings

**Response** (using Type B format above):

**Overall Evaluation**: Needs Minor Revision

**Gap**: Missing preference persistence for manual override

**Recommendations**:
1. Add Phase 2.5: Query theia-analyzer-agent about PreferenceService
2. Update Phase 2: Include preference integration

**Approval**: REVISE FIRST
