---
name: code-review
description: "Review code for compliance with standards, robustness, performance, and best practices"
---

# Code Review

## Purpose

Conduct a pre-merge code review checking compliance with Era Tactics standards, robustness, and quality gates.

## What You Provide

```
File(s): [Path(s) to review, e.g., src/game/combat.py]
Focus Areas: [Optional: robustness, performance, type safety, all?]
Context: [Optional: What problem does this solve?]
```

## What I'll Review

1. **Standards Compliance**
   - Google-style docstrings present
   - Type hints on all functions
   - Naming conventions (PascalCase classes, snake_case methods)
   - Import organization

2. **Robustness**
   - Input validation
   - Error handling (no silent failures)
   - State consistency checks
   - Invariant preservation

3. **Type Safety**
   - All parameters typed
   - Return types specified
   - No `Any` unless justified
   - Union types where appropriate

4. **Code Quality**
   - Single Responsibility Principle
   - Method length < 50 lines
   - Descriptive variable names
   - No magic numbers

5. **Performance**
   - No per-frame allocations
   - Efficient data structures
   - Caching where appropriate
   - Within performance budget

6. **Testing**
   - Unit tests exist
   - Edge cases covered
   - Integration tests for multi-system changes
   - Coverage adequate

7. **Testing Requirements Met**
   - [ ] All tests passing
   - [ ] Coverage >= 80% (100% for critical)
   - [ ] No regressions
   - [ ] Edge cases tested

## Example Usage

```
File(s): src/game/combat.py

Focus Areas: Robustness, Type Safety

Context: New combat system with damage calculation, visibility checks,
and terrain modifiers. Critical for game balance.
```

## Output Format

### Summary
✅ **Overall**: [PASS/FAIL]
- [X] Standards compliance
- [X] Robustness
- [X] Type safety
- [ ] Performance (needs optimization)
- [X] Testing

### Detailed Findings

#### Standards Compliance
- ✅ Docstrings: All public functions documented
- ✅ Type hints: 100% coverage
- ✅ Naming: Follows conventions
- ❌ Issue: Import organization (stdlib, third-party, local order wrong)
  - **Fix**: Reorder imports per PEP 8

#### Robustness
- ✅ Input validation: All parameters validated
- ✅ Error handling: No silent failures
- ✅ State consistency: Verified after changes
- ❌ Issue: Missing null check on `defender` parameter
  - **Fix**: Add `if defender is None: raise TypeError(...)`

#### Type Safety
- ✅ Parameters typed: 100% coverage
- ✅ Return types: All specified
- ❌ Issue: Function `calculate_damage()` uses `Any` for terrain parameter
  - **Fix**: Use `Terrain` type instead

#### Code Quality
- ✅ SRP: Each function has clear purpose
- ✅ Method length: Average 12 lines
- ✅ Variable names: Descriptive
- ❌ Issue: Magic number `2` in line 45 (terrain defense bonus)
  - **Fix**: Extract to named constant `TERRAIN_DEFENSE_BONUS = 2`

#### Performance
- ✅ No per-frame allocations
- ✅ Efficient lookups (set/dict usage)
- ⚠️ Issue: `calculate_damage()` called 1000x per frame
  - **Current time**: 0.5ms (within budget)
  - **Recommendation**: Cache visibility results if possible

#### Testing
- ✅ Unit tests: 12 tests, 100% coverage
- ✅ Integration tests: Multi-system tested
- ✅ Edge cases: All identified cases covered
- ⚠️ Issue: Missing performance test
  - **Add**: `test_damage_calculation_performance()`

### Quality Gate Checklist

Before merge:

- [X] All standards followed
- [X] Robustness verified
- [X] Type safe
- [X] Performant (or acceptable reason)
- [X] Tested (100% coverage)
- [X] Documented
- [X] No regressions
- [X] Approved by relevant specialist

### Blockers (If Any)

**MUST FIX before merge:**
- [ ] Missing null check on defender
- [ ] Magic number not extracted
- [ ] Performance regression > 10%

**NICE TO FIX:**
- [ ] Add performance test
- [ ] Consider caching visibility

### Recommendation

**✅ APPROVED** with minor notes (nice-to-fix items can be addressed in follow-up)

**Suggested Next Steps:**
1. Merge current PR
2. Follow-up: Add performance test (low priority)
3. Monitor gameplay for balance feedback

## Review Standards

- Focus on **correctness, robustness, and clarity**
- Point out **anti-patterns** clearly
- Suggest **concrete fixes**
- Prioritize **blockers** vs **nice-to-have**
- Be **constructive and specific**

## Common Issues to Watch

- ❌ Silent exceptions (catch without handling)
- ❌ Missing type hints
- ❌ No input validation
- ❌ Magic numbers/strings
- ❌ > 50 line methods
- ❌ Missing docstrings
- ❌ Insufficient test coverage
- ❌ Performance regressions
