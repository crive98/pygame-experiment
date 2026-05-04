---
name: design-feature
description: "Design a new feature with acceptance criteria, scope boundaries, and effort estimation"
---

# Design Feature

## Purpose

Help define a new feature's scope, acceptance criteria, risks, and effort before implementation begins.

## What You Provide

```
Feature Name: [Name of the feature]
Description: [What should it do? Why?]
Related Systems: [Which systems does it touch?]
```

## What I'll Generate

1. **Feature Specification**
   - Clear description of behavior
   - Success metrics
   - "Out of scope" boundaries

2. **Acceptance Criteria**
   - Specific, testable requirements
   - Checklist format
   - Definition of "done"

3. **Risk Assessment**
   - Technical risks
   - Dependencies
   - Integration points

4. **Effort Estimation**
   - Hours estimate
   - Complexity breakdown
   - Potential blockers

5. **Implementation Roadmap**
   - Suggested order of work
   - Dependencies between tasks
   - Suggested owner (which agent)

## Example Usage

```
Feature Name: Terrain Defense Modifiers

Description:
Different terrain types should provide defensive bonuses.
Mountains offer protection, forests provide cover, plains are exposed.
This adds tactical depth to positioning decisions.

Related Systems:
- Combat (damage calculation)
- Terrain system
- Configuration (balance)
- UI (display defense bonus)
```

## Output Format

### Specification
[Clear description]

### Acceptance Criteria
- [ ] [Specific requirement]
- [ ] [Specific requirement]
- [ ] All tests passing (100% coverage)
- [ ] Code reviewed
- [ ] Documentation complete
- [ ] Performance within budget

### Risk Assessment
- **Risk 1**: [Description] → **Mitigation**: [How to handle]
- **Risk 2**: [Description] → **Mitigation**: [How to handle]

### Effort Estimate
- Implementation: X hours
- Testing: Y hours
- Documentation: Z hours
- **Total**: X+Y+Z hours

### Implementation Order
1. [Task 1] (2 hours, Systems Programmer)
2. [Task 2] (1 hour, Graphics Engineer)
3. [Task 3] (1.5 hours, QA Engineer)
4. [Task 4] (0.5 hours, Technical Writer)
