---
name: generate-tests
description: "Generate comprehensive test scaffold for a new function or system"
---

# Generate Tests

## Purpose

Automatically generate unit tests, integration tests, and test factories for a new function or system.

## What You Provide

```
Function/System: [Name and signature]
File: [Where it lives, e.g., src/game/combat.py]
Coverage Target: [80%, 100%, critical paths only?]
Edge Cases: [Any known edge cases to test?]
```

## What I'll Generate

1. **Test Scaffold**
   - Unit test template
   - Fixture setup
   - Test factory functions

2. **Test Cases**
   - Happy path test
   - Error condition tests
   - Boundary condition tests
   - Edge case tests

3. **Coverage Analysis**
   - Which lines are covered
   - Which paths need tests
   - Coverage percentage

4. **Integration Test Suggestions**
   - Multi-system tests
   - State consistency checks
   - Performance tests

## Example Usage

```
Function/System: calculate_damage(attacker, defender, terrain)

File: src/game/combat.py

Coverage Target: 100%

Edge Cases:
- Attacker can't see defender (visibility check)
- Terrain provides defense bonus
- Damage result should be >= 0
- Overkill damage (excess over target health)
```

## Output Format

### Unit Tests

```python
def test_damage_calculation_visible_target():
    """Test: Visible target receives full damage."""
    # Arrange
    attacker = create_unit(damage=10)
    defender = create_unit(health=20)
    
    # Act
    damage = calculate_damage(attacker, defender, Terrain.PLAINS)
    
    # Assert
    assert damage == 10

def test_damage_calculation_hidden_target():
    """Test: Hidden target receives 0 damage."""
    # Arrange
    attacker = create_unit(damage=10)
    defender = create_unit(health=20)
    
    # Act (defender not visible)
    damage = calculate_damage(attacker, defender, Terrain.PLAINS)
    
    # Assert
    assert damage == 0
```

### Test Factory

```python
def create_unit(
    health: int = 100,
    damage: int = 10,
    faction: Faction = Faction.PLAYER
) -> Unit:
    """Factory for creating test units."""
    unit = Unit(health=health, damage=damage)
    unit.faction = faction
    return unit
```

### Integration Tests

```python
def test_combat_with_terrain_integration():
    """Test: Combat respects terrain defense."""
    # Test that combat system properly applies terrain modifiers
    ...
```

### Coverage Report

```
test_damage_calculation_visible_target: Lines 45-52 (8 lines)
test_damage_calculation_hidden_target: Lines 54-61 (8 lines)
test_damage_calculation_terrain: Lines 63-70 (8 lines)

Total Coverage: 24/24 lines = 100%
```

## Test Naming Convention

- `test_<function>_<scenario>_<expected_result>`
- Example: `test_damage_calculation_visible_target_deals_damage`
- Clear intent from name alone

## Important Notes

- Generate tests **before or during** implementation
- Aim for 100% coverage on critical systems
- Use AAA pattern (Arrange-Act-Assert)
- Include edge cases and error conditions
- Use descriptive assertion messages
