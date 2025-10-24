# PM Progressive Phases Methodology

This methodology is loaded when you execute `/pm-progressive-phases`.

## When to Use This

**Main Thread Request Pattern**:
- "Phase 1 complete. Found [findings]. Guidance for Phase 2?"
- "Implementation revealed [constraint]. Adjust strategy?"

This loads Pattern C methodology for providing follow-up guidance when phases complete or new information emerges.

## Methodology

### Pattern C: Progressive Phases

**Main Thread Request Format**:
```
Previously you provided strategic guide with Phase 1-3.

Phase 1 is complete. Found:
- [Key finding 1 that changes understanding]
- [Constraint discovered]
- [New requirement revealed]

What's the guidance for Phase 2 given these findings?
```

**Your Response**:

1. **Acknowledge Phase 1 results**: What was learned?
   - Read Phase 1 implementation or report
   - Identify key findings and their implications
   - Validate if findings align with or diverge from predictions

2. **Assess impact**: Do findings change original strategy?
   - **Minor findings**: Original strategy still valid, proceed as planned
   - **Significant findings**: Strategy needs adjustment but direction unchanged
   - **Major findings**: Strategy fundamentally changed, need new direction

3. **Adjust Phase 2 direction**: Modify based on new information
   - **If minor**: Reconfirm Phase 2 as originally planned
   - **If significant**: Refine Phase 2 scope/approach based on learnings
   - **If major**: Redesign Phase 2 entirely, possibly split into sub-phases

4. **Provide updated considerations**: New factors to address
   - What Phase 1 revealed that Phase 2 must handle
   - New constraints or opportunities discovered
   - Integration points that emerged

5. **Decide if phasing needs change**: Should we split Phase 2 into 2a and 2b?
   - **Keep phasing**: If original scope still appropriate
   - **Split phase**: If Phase 2 now too large/complex given findings
   - **Merge phases**: If Phase 1 findings simplified remaining work
   - **Add phase**: If new requirement emerged

## Assessment Framework

### Evaluate Phase 1 Findings

**For each finding, ask**:

1. **Impact Level**: How significant is this finding?
   - **Low**: Tactical detail, doesn't affect strategy
   - **Medium**: Adjusts approach but not direction
   - **High**: Changes fundamental understanding, requires new direction

2. **Scope Impact**: Does this affect Phase 2 scope?
   - **No change**: Proceed as planned
   - **Narrow scope**: Original plan too broad, focus more
   - **Expand scope**: Original plan missed important aspect
   - **Redirect scope**: Need different approach entirely

3. **Risk Level**: Does this introduce new risks?
   - **No new risks**: Continue confidently
   - **Manageable risks**: Proceed with mitigation plan
   - **High risks**: Reassess if Phase 2 should proceed

### Decision Matrix for Phase Adjustment

| Finding Impact | Scope Change | Risk Level | Action |
|---------------|--------------|------------|---------|
| Low | No change | Low | Proceed as planned |
| Medium | Narrow/Expand | Manageable | Refine Phase 2 |
| High | Redirect | High | Redesign Phase 2 |
| High | Expand significantly | Any | Split into sub-phases |

## Multi-turn Context Management

**CRITICAL**: When providing follow-up guidance, always:

1. **Reference original guide**: Quote relevant parts of initial strategic guide
2. **Acknowledge findings**: Show you've read and understood Phase 1 results
3. **Explain adjustments**: Be explicit about what changed and why
4. **Maintain continuity**: Preserve consistent direction unless findings justify pivot

**Example**:
```markdown
**Original Phase 2 Guidance** (from initial strategic guide):
"Phase 2: Implement theme switching service using Theia's PreferenceService"

**Phase 1 Findings**:
- Discovered: Theia's PreferenceService has async initialization race condition
- Impact: Cannot reliably access preferences at theme service startup

**Adjusted Phase 2 Guidance**:
Given the async initialization issue, I'm adjusting Phase 2:
- **Phase 2a**: Implement preference caching layer to handle async race
- **Phase 2b**: Integrate theme switching with cached preferences

**Rationale**: Original approach assumed synchronous preference access. Phase 1 revealed async complexity requires buffering layer first.
```

## Output Format

**Updated strategic guide focused on next phase, reference original guide for continuity**

```markdown
## EG-DESK PM: Progressive Phases Guidance

### Summary
[2-3 sentences: What Phase completed, key findings, Phase N guidance]

### Phase [N-1] Review

**Original Plan** (from initial strategic guide):
[Quote relevant parts of original Phase N-1 guidance]

**Actual Results**:
- **Finding 1**: [What was discovered]
- **Finding 2**: [Another discovery]
- **Finding 3**: [Constraint/opportunity revealed]

**Impact Assessment**:
| Finding | Impact Level | Scope Change | Risk Level |
|---------|--------------|--------------|------------|
| Finding 1 | [Low/Medium/High] | [None/Narrow/Expand/Redirect] | [Low/Manageable/High] |
| Finding 2 | [Level] | [Change] | [Risk] |
| Finding 3 | [Level] | [Change] | [Risk] |

**Overall Assessment**: [Minor findings / Significant findings / Major findings requiring pivot]

### Strategy Adjustment

**Original Strategy** (from initial guide):
[Quote original phasing plan]

**Adjusted Strategy**:

**Decision**: [PROCEED AS PLANNED / REFINE PHASE N / REDESIGN PHASE N / SPLIT PHASE N]

**Rationale for Adjustment**:
1. [Primary reason based on findings]
2. [Secondary reason or new constraint]
3. [Vision alignment check - still aligns with original direction?]

### Phase [N] Guidance

**Phase [N] Scope** (Updated):
[What Phase N should accomplish given new information]

**Approach**:
- **Step 1**: [First action, considering Phase N-1 findings]
- **Step 2**: [Next action, addressing discovered constraints]
- **Step 3**: [Final action, leveraging discovered opportunities]

**Critical Considerations** (New):
- [Consideration 1 based on Phase N-1 findings]
- [Consideration 2 addressing revealed constraint]
- [Consideration 3 for newly discovered integration point]

**Success Criteria**:
- [Criterion 1 - what Phase N must achieve]
- [Criterion 2 - validation that approach is working]
- [Criterion 3 - readiness for next phase]

### Updated Phasing (If Changed)

**Original Phasing**:
- Phase 1: [Original]
- Phase 2: [Original]
- Phase 3: [Original]

**Adjusted Phasing**:
- Phase 1: ✅ **Complete** - [Summary of results]
- Phase 2a: [New sub-phase if split]
- Phase 2b: [New sub-phase if split]
- Phase 3: [Updated if needed]

**Rationale for Phasing Change**:
[Why splitting/merging/reordering phases based on findings]

### References for Main Thread

**Original Strategic Guide**:
[Reference to initial guide for context]

**Phase [N-1] Results**:
[Files/reports from completed phase]

**Vision Documents** (if consulted for adjustment):
[Any vision docs referenced in making adjustment decision]

### Next Steps for Main Thread

1. **Execute Phase [N]**: [Specific first action]
2. **Monitor for**: [What to watch for that might require further adjustment]
3. **Return to PM if**: [Conditions that would require another progressive guidance consultation]
```

## Example

**Request**: "Phase 1 complete. Found Theia PreferenceService has async race condition. Guidance for Phase 2?"

**Response**:

## EG-DESK PM: Progressive Phases Guidance

**Summary**: Phase 1 revealed async initialization race in PreferenceService. Adjusting Phase 2: Split into 2a (preference caching) + 2b (theme integration).

**Phase 1 Review**:
- **Finding**: PreferenceService async initialization race condition
- **Impact**: High (changes Phase 2 approach)
- **Risk**: Manageable (solved with caching layer)

**Adjusted Strategy**: SPLIT PHASE 2

**Rationale**: Original plan assumed synchronous preference access. Async complexity requires buffering layer.

**Updated Phasing**:
- Phase 1: ✅ Complete - Theme service foundation
- Phase 2a: **NEW** - Preference caching layer (handles async race)
- Phase 2b: Theme switching integration (uses cache)
- Phase 3: (unchanged) - UI preferences panel

**Phase 2a Guidance**:
1. Implement PreferenceCache service
2. Buffer preference reads until async init complete
3. Integrate with theme service startup

**Critical Considerations**:
- Cache invalidation strategy when preferences change
- Fallback behavior if preferences fail to load
