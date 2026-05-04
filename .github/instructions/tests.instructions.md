---
name: tests-instructions
description: "Testing instructions for Era Tactics: unit tests, integration tests, comprehensive coverage, edge cases"
applyTo: "tests/**/*.py"
---

# Testing Instructions

## Scope

These instructions apply to all Python files in `tests/`. Testing is critical for maintaining code quality and preventing regressions in complex game systems.

## Testing Philosophy

### Test-First Mindset
1. **Write tests before/during implementation**
2. **Tests define the specification**
3. **100% coverage on critical systems** (combat, movement, state)
4. **Regression prevention**: Test every fixed bug

### Test Pyramid
```
UI/E2E Tests (10%)
    ↑
Integration Tests (30%)
    ↑
Unit Tests (60%)
```

Start with unit tests, add integration tests for multi-system flows, E2E tests sparingly.

## Unit Test Standards

### Naming Convention
```python
# Pattern: test_<function>_<scenario>_<expected_result>

# ✅ Good - Clear what's being tested
def test_damage_calculation_visible_enemy_deals_damage():
    ...

def test_damage_calculation_hidden_enemy_deals_zero():
    ...

def test_movement_pathfinding_blocked_path_returns_none():
    ...

# ❌ Avoid - Vague
def test_damage():
    ...

def test_works():
    ...
```

### AAA Pattern (Arrange-Act-Assert)
```python
def test_damage_calculation_visible_enemy():
    """Test: Visible enemy receives full damage."""
    # Arrange: Set up test state
    attacker = create_unit(damage=10, faction=Faction.PLAYER)
    defender = create_unit(health=20, faction=Faction.CPU1)
    terrain = Terrain.PLAINS
    
    world = GameWorld()
    world.add_unit(attacker, pos=(0, 0))
    world.add_unit(defender, pos=(1, 1))
    attacker.visibility.add_unit(defender)
    
    # Act: Execute the function under test
    damage = world.combat_system.calculate_damage(
        attacker, defender, terrain
    )
    
    # Assert: Verify result
    assert damage == 10, "Visible enemy should take full damage"
```

### Setup & Teardown
```python
import pytest

@pytest.fixture
def world():
    """Create a fresh game world for each test."""
    w = GameWorld(width=20, height=20)
    yield w
    w.cleanup()  # Cleanup after test

def test_something(world):
    # Test uses fresh world
    unit = Unit(faction=Faction.PLAYER)
    world.add_unit(unit, pos=(5, 5))
    
    assert unit.position == (5, 5)
```

### Factories for Test Objects
```python
# Create reusable factories for test fixtures
def create_unit(
    health: int = 100,
    damage: int = 10,
    faction: Faction = Faction.PLAYER,
    is_general: bool = False
) -> Unit:
    """Factory for creating test units."""
    unit = Unit(health=health, damage=damage)
    unit.faction = faction
    unit.is_general = is_general
    return unit

# Use in tests
def test_unit_death():
    unit = create_unit(health=5)
    apply_damage(unit, 10)
    assert unit.health == 0
```

## Critical Test Areas

### 1. Combat System (High Priority - 100% Coverage)

```python
def test_damage_calculation_visible_target():
    """Visible target receives full damage."""
    attacker = create_unit(damage=10)
    defender = create_unit(health=20)
    
    attacker.visibility.add_unit(defender)
    
    damage = calculate_damage(attacker, defender, Terrain.PLAINS)
    assert damage == 10

def test_damage_calculation_hidden_target():
    """Hidden target receives 0 damage."""
    attacker = create_unit(damage=10)
    defender = create_unit(health=20)
    
    # Don't add to visibility
    
    damage = calculate_damage(attacker, defender, Terrain.PLAINS)
    assert damage == 0

def test_damage_calculation_terrain_modifier():
    """Terrain reduces damage."""
    attacker = create_unit(damage=10)
    defender = create_unit(health=20)
    terrain = Terrain.MOUNTAIN  # +2 defense
    
    attacker.visibility.add_unit(defender)
    
    damage = calculate_damage(attacker, defender, terrain)
    assert damage == 8, "10 - 2 terrain defense = 8"

def test_damage_zero_when_defender_dead():
    """Cannot damage eliminated unit."""
    attacker = create_unit(damage=10)
    defender = create_unit(health=0)
    
    with pytest.raises(ValueError):
        calculate_damage(attacker, defender, Terrain.PLAINS)

def test_damage_negative_raises_error():
    """Negative damage should raise error."""
    attacker = create_unit(damage=-5)
    defender = create_unit(health=20)
    
    with pytest.raises(ValueError):
        calculate_damage(attacker, defender, Terrain.PLAINS)

def test_unit_elimination_general_eliminates_faction():
    """Killing general eliminates entire faction."""
    general = create_unit(is_general=True, faction=Faction.CPU1)
    minion = create_unit(faction=Faction.CPU1, is_general=False)
    
    world = GameWorld()
    world.add_unit(general, pos=(0, 0))
    world.add_unit(minion, pos=(1, 1))
    
    world.eliminate_unit(general)
    
    assert Faction.CPU1 not in world.active_factions
    assert minion.faction == Faction.REBELS
```

### 2. Movement System (High Priority - 100% Coverage)

```python
def test_pathfinding_direct_path():
    """Find optimal path when no obstacles."""
    world = GameWorld(width=10, height=10)
    
    path = world.movement_system.pathfind(
        start=(0, 0),
        goal=(5, 5)
    )
    
    assert len(path) > 0
    assert path[-1] == (5, 5)

def test_pathfinding_blocked_path():
    """Return None when path blocked."""
    world = GameWorld(width=10, height=10)
    
    # Block path
    for i in range(10):
        world.add_obstacle((5, i))
    
    path = world.movement_system.pathfind(
        start=(0, 0),
        goal=(9, 0)
    )
    
    assert path is None

def test_movement_terrain_cost():
    """Terrain affects movement cost."""
    world = GameWorld(width=10, height=10)
    unit = create_unit()
    
    cost_plains = world.movement_system.get_movement_cost(
        unit, Terrain.PLAINS
    )
    cost_mountain = world.movement_system.get_movement_cost(
        unit, Terrain.MOUNTAIN
    )
    
    assert cost_mountain > cost_plains
```

### 3. Visibility System (High Priority - 100% Coverage)

```python
def test_visibility_line_of_sight():
    """Can see unit in direct line of sight."""
    world = GameWorld(width=20, height=20)
    observer = create_unit()
    target = create_unit()
    
    world.add_unit(observer, pos=(0, 0))
    world.add_unit(target, pos=(5, 0))
    
    assert observer.can_see(target)

def test_visibility_blocked_by_terrain():
    """Mountain blocks line of sight."""
    world = GameWorld(width=20, height=20)
    observer = create_unit()
    target = create_unit()
    
    world.add_unit(observer, pos=(0, 0))
    world.add_unit(target, pos=(5, 0))
    world.set_terrain((2, 0), Terrain.MOUNTAIN)
    
    assert not observer.can_see(target)

def test_visibility_range_limit():
    """Cannot see beyond vision range."""
    world = GameWorld(width=20, height=20)
    observer = create_unit(vision_range=3)
    target = create_unit()
    
    world.add_unit(observer, pos=(0, 0))
    world.add_unit(target, pos=(10, 0))
    
    assert not observer.can_see(target)
```

### 4. State Management (Medium Priority)

```python
def test_game_state_consistency_after_update():
    """Game state remains valid after update."""
    world = GameWorld()
    
    # Add units
    for _ in range(10):
        unit = create_unit()
        world.add_unit(unit, pos=random_position())
    
    # Update
    world.update(delta_time=0.016)
    
    # Verify consistency
    assert len(world.active_units) > 0
    for unit in world.active_units:
        assert world.is_valid_position(unit.position)
        assert unit.health >= 0
```

## Integration Tests

Test multi-system interactions:

```python
def test_combat_with_visibility_integration():
    """Combat respects visibility system."""
    world = GameWorld()
    
    attacker = create_unit(damage=10, faction=Faction.PLAYER)
    defender = create_unit(health=20, faction=Faction.CPU1)
    
    world.add_unit(attacker, pos=(0, 0))
    world.add_unit(defender, pos=(10, 10))  # Out of visibility range
    
    # Attack should fail (can't see)
    result = world.perform_attack(attacker, defender)
    assert result.damage == 0

def test_movement_with_terrain_integration():
    """Movement path respects terrain costs."""
    world = GameWorld()
    
    unit = create_unit()
    world.add_unit(unit, pos=(0, 0))
    
    # Shortest path might go through expensive terrain
    path = world.movement_system.pathfind(
        start=(0, 0),
        goal=(5, 5),
        terrain_costs={
            Terrain.MOUNTAIN: 5,
            Terrain.PLAINS: 1
        }
    )
    
    # Path should avoid mountains when possible
    mountain_count = sum(
        1 for pos in path 
        if world.get_terrain(pos) == Terrain.MOUNTAIN
    )
    assert mountain_count <= 2
```

## Performance Tests

```python
def test_pathfinding_performance():
    """Pathfinding completes in < 1ms."""
    world = GameWorld(width=100, height=100)
    
    start = time.perf_counter()
    
    world.movement_system.pathfind(
        start=(0, 0),
        goal=(99, 99)
    )
    
    elapsed = time.perf_counter() - start
    assert elapsed < 0.001, f"Pathfinding took {elapsed}s"
```

## Test Organization

```
tests/
├── conftest.py (shared fixtures)
├── unit/
│   ├── test_combat_system.py
│   ├── test_movement_system.py
│   ├── test_visibility_system.py
│   ├── test_unit_state.py
│   └── test_game_state.py
├── integration/
│   ├── test_combat_with_visibility.py
│   ├── test_movement_with_terrain.py
│   └── test_full_turn_sequence.py
├── performance/
│   ├── test_pathfinding_perf.py
│   └── test_combat_perf.py
└── factories/
    ├── __init__.py
    ├── unit_factory.py
    └── world_factory.py
```

## Coverage Requirements

- **Critical systems** (combat, movement, state): 100%
- **Important systems** (visibility, AI): 90%+
- **Overall**: 80%+ coverage

Track with:
```bash
pytest --cov=src --cov-report=html
```

## Regression Prevention

**Every bug fix requires a test:**

```python
# Bug: Unit could be damaged after elimination
# Fix: Add validation

def test_regression_cannot_damage_eliminated_unit():
    """Regression test for bug #42.
    
    Previously, eliminated units could be damaged again,
    causing state corruption. Now should raise ValueError.
    """
    unit = create_unit(health=0)
    
    with pytest.raises(ValueError):
        apply_damage(unit, 10)
```

## Summary

Write tests that are:
- ✅ **Clear**: Test name describes exactly what's tested
- ✅ **Focused**: One assertion per test (generally)
- ✅ **Comprehensive**: 100% coverage on critical paths
- ✅ **Maintainable**: Use factories, fixtures, reusable helpers
- ✅ **Fast**: Unit tests < 1ms each
- ✅ **Reliable**: Pass/fail deterministically
