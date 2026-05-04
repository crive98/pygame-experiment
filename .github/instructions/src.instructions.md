---
name: src-instructions
description: "Source code instructions for Era Tactics: production-grade standards, robustness, performance, error handling"
applyTo: "src/**/*.py"
---

# Source Code Instructions

## Scope

These instructions apply to all Python files in `src/`. They reinforce and extend the core standards in `copilot-instruction.md` with emphasis on production-grade robustness.

## Core Principles

### 1. Robustness First
Every public function must:
- **Validate inputs** at entry (type checking, bounds, null checks)
- **Fail fast**: Raise exceptions immediately on invalid state
- **Clear error messages**: Include context that helps debugging
- **No silent failures**: Never catch exceptions without handling

```python
# ✅ Good - Validates and fails fast
def set_unit_position(unit: Unit, pos: Position) -> None:
    """Set unit position with validation.
    
    Args:
        unit: Unit to move.
        pos: New position.
    
    Raises:
        ValueError: If position is invalid or out of bounds.
        TypeError: If inputs are wrong type.
    """
    if not isinstance(unit, Unit):
        raise TypeError(f"Expected Unit, got {type(unit)}")
    if not isinstance(pos, Position):
        raise TypeError(f"Expected Position, got {type(pos)}")
    
    if not self.is_valid_position(pos):
        raise ValueError(f"Position {pos} out of bounds")
    
    unit.position = pos

# ❌ Avoid - Silent failure
def set_position(u, p):
    try:
        u.pos = p
    except:
        pass  # Silent failure!
```

### 2. Type Safety
- **Type hints on all functions**: Parameter and return types
- **Use concrete types**: Avoid `Any` unless absolutely necessary
- **Union types when needed**: `Unit | None` instead of just `Unit`
- **Type aliases for clarity**: `Position = tuple[int, int]`

```python
# ✅ Good - Clear types
def get_visible_units(observer: Unit) -> list[Unit]:
    """Get units visible to observer.
    
    Args:
        observer: Unit checking visibility.
    
    Returns:
        List of visible enemy units.
    """
    visible: list[Unit] = []
    for unit in self.all_units:
        if observer.can_see(unit):
            visible.append(unit)
    return visible

# ❌ Avoid - Vague types
def get_visible(obs):
    return [u for u in units if obs.sees(u)]
```

### 3. State Consistency
- **Invariants preserved**: State changes maintain valid invariants
- **Atomic operations**: Related changes happen together
- **No partial updates**: Either fully update or roll back
- **Validate after changes**: Verify state is still valid

```python
# ✅ Good - Atomic state change
def eliminate_unit(unit: Unit) -> None:
    """Remove unit and handle faction consequences.
    
    If general dies, entire faction is eliminated.
    Orphaned units become rebels.
    """
    self.active_units.remove(unit)
    
    if unit.is_general:
        self._eliminate_faction(unit.faction)
    elif unit.faction in self.eliminated_factions:
        unit.faction = Faction.REBELS
    
    # Verify state is still valid
    self._validate_game_state()

# ❌ Avoid - Partial update
def kill_unit(u):
    units.remove(u)
    # Oops, forgot to handle general death!
```

### 4. Performance Consciousness
- **Avoid per-frame allocations**: Cache/reuse objects
- **Efficient data structures**: Choose for the access pattern
  - Set/dict for lookups
  - deque for queues (popleft is O(1))
  - List for iteration
- **Profile before optimizing**: Measure, identify bottleneck, fix
- **Document hot paths**: Mark performance-critical code

```python
# ✅ Good - Efficient lookup
self.units_by_position: dict[Position, Unit] = {}  # Cache

def get_unit_at(pos: Position) -> Unit | None:
    """Get unit at position (O(1) lookup)."""
    return self.units_by_position.get(pos)

# ✅ Good - Efficient queue
from collections import deque
action_queue: deque[Action] = deque()  # O(1) popleft

# ❌ Avoid - Inefficient lookup
def get_unit_at(pos):
    for u in all_units:  # O(n)!
        if u.position == pos:
            return u
```

### 5. Error Handling
- **Explicit exceptions**: Use specific exception types
- **Context in messages**: What failed and why
- **Recover or escalate**: Don't silent-catch
- **Log failures**: Debug information for post-mortems

```python
# ✅ Good - Explicit handling
def apply_damage(target: Unit, damage: int) -> None:
    """Apply damage to target.
    
    Raises:
        ValueError: If damage < 0 or target eliminated.
    """
    if damage < 0:
        raise ValueError(f"Damage cannot be negative: {damage}")
    if target.is_eliminated:
        raise ValueError(f"Cannot damage eliminated unit: {target.id}")
    
    new_health = max(0, target.health - damage)
    target.health = new_health
    
    if target.health == 0:
        logger.info(f"Unit {target.id} eliminated (overkill: {-new_health})")
        self.eliminate_unit(target)

# ❌ Avoid - Silent catch
def apply_damage(t, d):
    try:
        t.health -= d
    except:
        return  # What went wrong?
```

## Code Organization

### Module Structure
```
src/
├── __init__.py
├── main.py (entry point)
├── game/
│   ├── __init__.py
│   ├── world.py (WorldState class)
│   ├── unit.py (Unit class)
│   ├── combat.py (CombatSystem class)
│   ├── movement.py (MovementSystem class)
│   └── visibility.py (VisibilitySystem class)
├── ai/
│   ├── __init__.py
│   ├── cpu_controller.py
│   └── decision_maker.py
├── graphics/
│   ├── __init__.py
│   ├── renderer.py
│   └── camera.py
└── utils/
    ├── __init__.py
    ├── types.py (type aliases, enums)
    └── constants.py (game constants)
```

### One Class Per File (Generally)
- Exception: Closely related utility classes (2-3)
- Keep files focused and maintainable
- Easy to locate code you need

## Documentation Requirements

Every public class, method, and module must have Google-style docstrings:

```python
"""Module docstring explaining file purpose."""

class CombatSystem:
    """Handles all combat mechanics and damage application.
    
    This system manages damage calculation, application, and unit
    elimination. Combat is deterministic—same input always produces
    same damage value.
    
    Attributes:
        world: Reference to world state for unit queries.
        damage_log: Record of all damage events for debugging.
    """
    
    def calculate_damage(
        self, 
        attacker: Unit, 
        defender: Unit, 
        terrain: Terrain
    ) -> int:
        """Calculate damage with all modifiers applied.
        
        Applies attacker damage, terrain defense bonus, and
        visibility constraint (0 if target not visible).
        
        Args:
            attacker: Unit dealing damage.
            defender: Unit taking damage.
            terrain: Terrain defender is on.
        
        Returns:
            Final damage value (0 if defender not visible).
        
        Raises:
            ValueError: If units not in valid state.
        """
```

## Testing Expectations

- Every public function should have corresponding unit tests
- Critical paths (combat, movement, state) require 100% coverage
- Integration tests for multi-system interactions
- See `tests.instructions.md` for detailed testing standards

## Performance Constraints

- **Game loop**: 60 FPS target (16.67ms per frame)
- **AI decision**: < 5ms per faction turn
- **Pathfinding**: < 1ms per unit movement
- **State update**: < 5ms per frame
- **Total render**: < 10ms per frame

Profile and optimize if exceeding budgets.

## Determinism & Reproducibility

When using randomness:
- **Use seed-based RNG**: Same seed → same output
- **Document randomness**: Explain why it's needed
- **Avoid `random.random()`**: Use numpy.random or controlled seeding

```python
# ✅ Good - Controlled randomness
def generate_map(seed: int, config: MapConfig) -> WorldMap:
    """Generate procedural map with seed control."""
    rng = np.random.RandomState(seed)
    
    noise = rng.uniform(0, 1, (config.width, config.height))
    terrain = self._apply_threshold(noise, config.thresholds)
    
    return WorldMap(terrain)

# ❌ Avoid - Uncontrolled randomness
def random_terrain():
    return random.choice([MOUNTAIN, FOREST, PLAINS])
```

## No Magic Numbers

Use named constants, not hardcoded values:

```python
# ✅ Good
TERRAIN_DEFENSE_BONUS = 2
MAX_VISIBILITY_RANGE = 10
INITIAL_HEALTH = 100

def calculate_defense(terrain):
    return TERRAIN_DEFENSE_BONUS if terrain.is_mountain else 0

# ❌ Avoid
def calculate_defense(t):
    return 2 if t.is_mountain else 0  # Magic number!
```

## Summary

Write code that is:
- ✅ **Robust**: Validates, fails fast, handles errors
- ✅ **Type-safe**: Full type hints
- ✅ **Consistent**: Maintains invariants
- ✅ **Performant**: Efficient structures, profiled
- ✅ **Clear**: Descriptive names, documented
- ✅ **Deterministic**: Reproducible behavior
- ✅ **Maintainable**: Well-organized, tested
