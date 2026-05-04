---
name: era-tactics-agents
description: "Era Tactics development team - 8 specialized agents for game development"
---

# Era Tactics - Development Team Agents

This workspace uses a **professional game development team structure** with 8 specialized agents, each with distinct expertise and responsibilities.

## 👥 Team Overview

The agents function as a complete game development studio, with clear role separation, tool restrictions, and integration points.

## 🎯 Team Members

### 1. Technical Lead
**File**: `technical-lead.agent.md`
**Expertise**: Architecture, design patterns, system integration
**Use when**: 
- Making high-level design decisions
- Reviewing architectural impacts
- Planning system integration
- Mentoring on best practices

### 2. Systems Programmer
**File**: `systems-programmer.agent.md`
**Expertise**: Core gameplay, combat, movement, state management
**Use when**:
- Implementing game mechanics (movement, combat, timing)
- Managing game state and transitions
- Building visibility and terrain systems
- Ensuring robustness and correctness

### 3. Graphics Engineer
**File**: `graphics-engineer.agent.md`
**Expertise**: pygame rendering, animations, UI, performance
**Use when**:
- Building rendering pipeline
- Implementing animations and effects
- Creating UI systems
- Optimizing frame rate and draw calls

### 4. AI Programmer
**File**: `ai-programmer.agent.md`
**Expertise**: CPU behavior, decision trees, pathfinding, tactics
**Use when**:
- Implementing enemy AI
- Building decision-making logic
- Coding pathfinding algorithms
- Ensuring AI determinism

### 5. Tools Engineer
**File**: `tools-engineer.agent.md`
**Expertise**: Procedural generation, data pipelines, configuration
**Use when**:
- Building map generation systems
- Managing tech tree and configuration
- Creating data pipelines
- Building debugging tools

### 6. QA Engineer
**File**: `qa-engineer.agent.md`
**Expertise**: Testing, validation, quality assurance
**Use when**:
- Writing unit and integration tests
- Validating game mechanics
- Finding and preventing regressions
- Ensuring code coverage

### 7. Technical Writer
**File**: `technical-writer.agent.md`
**Expertise**: Documentation, docstrings, API clarity
**Use when**:
- Writing Google-style docstrings
- Documenting architectures
- Creating API documentation
- Maintaining code clarity

### 8. Production Manager
**File**: `production-manager.agent.md`
**Expertise**: Scope management, prioritization, design alignment
**Use when**:
- Defining feature scope and acceptance criteria
- Prioritizing work
- Reviewing design alignment
- Managing quality gates

## 🤝 How to Use the Team

### Example: Implementing Combat System

```
1. Production Manager: Define scope and success criteria
2. Systems Programmer: Implement core damage logic
3. Graphics Engineer: Visualize combat feedback
4. AI Programmer: Use combat in decision-making
5. QA Engineer: Write comprehensive tests
6. Technical Writer: Document the API
7. Technical Lead: Review architecture
```

### Example: Building Map Generation

```
1. Production Manager: Define what "good map" means
2. Tools Engineer: Implement procedural generation
3. Systems Programmer: Integrate with game world
4. QA Engineer: Validate reproducibility
5. Technical Writer: Document generation parameters
6. Technical Lead: Optimize performance
```

## ⚙️ Integration Points

```
Technical Lead (oversight & mentoring)
    ├── Systems Programmer (core mechanics)
    ├── Graphics Engineer (rendering)
    ├── AI Programmer (enemy behavior)
    ├── Tools Engineer (data pipelines)
    ├── QA Engineer (quality)
    ├── Technical Writer (documentation)
    └── Production Manager (coordination)
```

## 📋 Team Meeting Topics

**Weekly Standup**:
- Current focus per engineer
- Blockers and dependencies
- Integration points

**Design Review**:
- New features from Production Manager
- Technical design from Technical Lead + specialists
- Feasibility assessment

**Quality Review**:
- Test coverage metrics from QA Engineer
- Performance metrics from Graphics/Systems
- Documentation status from Technical Writer

**Retrospective**:
- What went well
- What could improve
- Process improvements

## 🚀 Getting Started

1. **Define your feature** with Production Manager
   - Acceptance criteria
   - Success metrics
   - Scope boundaries

2. **Assign to specialists** based on system area
   - Use `/technical-lead` for architecture questions
   - Use `/systems-programmer` for game mechanics
   - Use `/graphics-engineer` for rendering
   - etc.

3. **Coordinate integration** through Technical Lead
   - Ensure handoffs are clear
   - Validate contracts between systems
   - Flag potential issues

4. **Validate quality** with QA Engineer
   - Tests written before/during implementation
   - Coverage metrics tracked
   - Regressions prevented

5. **Document thoroughly** with Technical Writer
   - Every API documented
   - Architecture decisions recorded
   - Examples provided

## ✅ Quality Gates

Before merging any feature:

- [ ] Production Manager: Aligned with design vision
- [ ] Specialist engineers: Implementation complete
- [ ] Technical Lead: Architecture reviewed
- [ ] QA Engineer: Tests passing, coverage adequate
- [ ] Technical Writer: Documentation complete
- [ ] Technical Lead: Final approval

## 📊 Recommended Workflow

```
Feature Request
    ↓
Production Manager defines scope + acceptance criteria
    ↓
Technical Lead reviews architectural impact
    ↓
Specialist(s) implement
    ↓
QA Engineer writes/validates tests
    ↓
Technical Writer ensures documentation
    ↓
Peer review by relevant specialists
    ↓
Technical Lead final review
    ↓
Merge to development
```