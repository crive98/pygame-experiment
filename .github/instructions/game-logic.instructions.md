---
name: game-logic-instructions
description: "Game logic implementation instructions: core mechanics, determinism, robustness, combat and movement accuracy"
applyTo: "src/game/**/*.py"
---

# Game Logic Instructions

## Scope

These instructions apply to core gameplay mechanics in `src/game/`. This is where the game rules live—correctness and robustness are paramount.

## Core Principles

### 1. Correctness First
Game logic must be **provably correct**:
- Implement exactly as specified in README.md
- No approximations or simplifications
- Deterministic behavior (same input → same output)
- Fail fast if state is invalid

### 2. State Consistency
Game state must always be **valid and consistent**:
- Unit health: 0 ≤ health ≤ max_health
- Unit position: valid grid position, no two units at same spot
- Faction integrity: general alive ↔ faction active
- Timeline validity: action costs make sense

```python
# ✅ Good - Verify state after changes
def eliminate_unit(unit: Unit) -> None:
    """Remove unit and maintain state consistency."""
    if unit.is_general:
        # Eliminate faction BEFORE removing general
        self._eliminate_faction(unit.faction)
    
    # Remove from tracking
    self.active_units.remove(unit)
    del self.units_by_position[unit.position]
    
    # Convert to rebel if needed
    if unit.faction == Faction.REBELS:
        pass
    
    # Verify state is still valid
    self._assert_game_state_valid()
```

### 3. Timeline & Initiative Accuracy
The action cost system must be **exact**:
- Action costs increase correctly after each action
- Queue ordering is deterministic
- Initiative is calculated precisely
- Ties broken consistently

```python
# ✅ Good - Exact action cost calculation
def update_action_costs(self) -> None:
    """Update action costs for timeline system.
    
    Each unit's action cost represents when they act next.
    Lower cost → acts first. After acting, cost increases by
    action duration + unit.initiative.
    """
    for unit in self.active_units:
        # Action cost is deterministic based on:
        # - Last action duration
        # - Unit initiative (constant)
        # - Unit speed (constant)
        
        if unit.just_acted:
            action_duration = unit.last_action.duration
            initiative = unit.stats.initiative
            unit.action_cost += action_duration + initiative
            unit.just_acted = False
```

### 4. Visibility System Accuracy
Visibility must follow **exact rules**:
- Line of sight calculation is consistent
- Terrain blocks vision correctly
- Vision range is applied properly
- Fog of war matches visibility

```python
# ✅ Good - Exact line-of-sight calculation
def calculate_visibility(observer: Unit) -> set[Position]:
    """Calculate exact visible positions from observer's position.
    
    Visibility rules:
    1. Maximum distance from observer = vision_range
    2. Terrain blocks sight (mountains, dense forest)
    3. Line-of-sight to visible positions (no wrapping)
    4. Result is deterministic
    """
    visible = set()
    
    for pos in self.get_positions_in_range(
        observer.position, 
        observer.stats.vision_range
    ):
        if self._has_line_of_sight(observer.position, pos):
            visible.add(pos)
    
    return visible
```

### 5. Combat Accuracy
Damage calculation must **match the specification exactly**:
- All modifiers applied in correct order
- Visible target check enforced
- Terrain defense applied correctly
- Health bounds respected

```python
# ✅ Good - Combat follows specification exactly
def calculate_damage(
    attacker: Unit,
    defender: Unit,
    terrain: Terrain
) -> int:
    """Calculate damage with all modifiers.
    
    Formula (from game design):
    - Base damage: attacker.weapon.damage
    - Minus: terrain.defense_bonus (if defender in terrain)
    - Result: clamped to [0, infinity)
    - Visibility: 0 if defender not visible
    
    Args:
        attacker: Unit dealing damage.
        defender: Unit taking damage.
        terrain: Terrain defender is on.
    
    Returns:
        Final damage value.
    
    Raises:
        ValueError: If units not in valid state.
    """
    # Check visibility first
    if not attacker.can_see(defender):
        return 0
    
    # Calculate base damage
    base_damage = attacker.stats.weapon_damage
    
    # Apply terrain defense modifier
    defense_bonus = terrain.get_defense_bonus_for_unit(defender)
    final_damage = base_damage - defense_bonus
    
    # Clamp to valid range
    final_damage = max(0, final_damage)
    
    return final_damage
```

## Implementation Patterns

### State Change Pattern
For any operation that changes game state:

```python
def apply_damage(unit: Unit, damage: int) -> DamageResult:
    """Apply damage and return result.
    
    Pattern:
    1. Validate inputs
    2. Calculate changes
    3. Apply changes atomically
    4. Handle consequences (death, etc.)
    5. Verify state consistency
    """
    # 1. Validate
    if not isinstance(unit, Unit):
        raise TypeError(f"Expected Unit, got {type(unit)}")
    if damage < 0:
        raise ValueError(f"Damage cannot be negative: {damage}")
    
    # 2. Calculate
    new_health = max(0, unit.health - damage)
    overkill = damage - unit.health if new_health == 0 else 0
    
    # 3. Apply atomically
    old_health = unit.health
    unit.health = new_health
    
    # 4. Handle consequences
    result = DamageResult(
        damage_dealt=old_health - new_health,
        overkill=overkill,
        unit_eliminated=unit.health == 0
    )
    
    if unit.health == 0:
        self.eliminate_unit(unit)
    
    # 5. Verify
    self._assert_game_state_valid()
    
    return result
```

### Query Pattern
For read-only operations:

```python
def get_visible_enemies(unit: Unit) -> list[Unit]:
    """Get all visible enemies.
    
    Does not modify state. Safe to call multiple times.
    Result is deterministic.
    """
    visible = []
    
    for other in self.all_units:
        if other.faction != unit.faction:  # Enemy
            if unit.can_see(other):  # Visible
                visible.append(other)
    
    # Sort for determinism (in case order matters)
    return sorted(visible, key=lambda u: u.id)
```

### Validation Pattern
For critical invariants:

```python
def _assert_game_state_valid(self) -> None:
    """Verify game state invariants.
    
    Raises AssertionError if any invariant violated.
    """
    # Every unit must have valid position
    for unit in self.all_units:
        assert self.is_valid_position(unit.position), \
            f"Unit {unit.id} at invalid position {unit.position}"
    
    # No two units at same position
    positions = [u.position for u in self.all_units]
    assert len(positions) == len(set(positions)), \
        "Multiple units at same position"
    
    # All generals must be alive
    for faction in self.active_factions:
        generals = [u for u in self.all_units 
                   if u.is_general and u.faction == faction]
        assert len(generals) == 1, \
            f"Faction {faction} should have exactly 1 general"
    
    # Health bounds
    for unit in self.all_units:
        assert 0 <= unit.health <= unit.max_health, \
            f"Unit {unit.id} health out of bounds: {unit.health}"
```

## No Approximations

Game logic must be **exact**. Don't approximate:

```python
# ❌ Avoid - Approximation
def is_in_range(attacker: Unit, defender: Unit) -> bool:
    dist = attacker.distance_to(defender)
    return dist < attacker.attack_range + 1  # Fuzzy!

# ✅ Good - Exact
def is_in_range(attacker: Unit, defender: Unit) -> bool:
    """Check if defender is exactly in attack range."""
    # Use Chebyshev distance (as specified in game design)
    dist = max(
        abs(attacker.position[0] - defender.position[0]),
        abs(attacker.position[1] - defender.position[1])
    )
    return dist <= attacker.stats.attack_range
```

## Determinism

All calculations must be **deterministic**:

```python
# ✅ Good - Deterministic
def get_unit_priority(unit: Unit) -> tuple:
    """Get sort key for deterministic ordering.
    
    Same unit state always produces same priority.
    """
    return (
        unit.action_cost,  # Lower cost acts first
        unit.id,           # Tie-breaker (unit ID)
    )

# ❌ Avoid - Non-deterministic
def get_unit_priority(unit):
    return unit.action_cost + random.random()  # Random!
```

## Performance Considerations

- **Combat calculation**: Should be < 0.1ms per hit
- **Visibility update**: < 1ms per unit
- **Pathfinding**: < 1ms per unit
- **State update**: < 5ms per frame

Profile and optimize if hitting budgets.

## Error Handling

Be strict in game logic:

```python
# ✅ Good - Strict validation
def move_unit(unit: Unit, destination: Position) -> None:
    """Move unit to destination.
    
    Raises:
        ValueError: If destination invalid or unit can't move there.
    """
    if not self.is_valid_position(destination):
        raise ValueError(f"Invalid position: {destination}")
    
    if not self._can_move_to(unit, destination):
        raise ValueError(f"Unit cannot move to {destination}")
    
    # Move
    old_pos = unit.position
    unit.position = destination
    
    # Update tracking
    del self.units_by_position[old_pos]
    self.units_by_position[destination] = unit
```

## Testing

Every function in game logic must have tests:
- Unit tests for individual calculations
- Integration tests for multi-system flows
- Edge case tests for boundary conditions

See `tests.instructions.md` for detailed testing standards.

## Summary

Write game logic that is:
- ✅ **Correct**: Implements specification exactly
- ✅ **Consistent**: Maintains invariants
- ✅ **Deterministic**: Same input → same output
- ✅ **Validated**: Checks invariants after changes
- ✅ **Tested**: 100% coverage on critical paths
- ✅ **Performant**: Within frame budgets
- ✅ **Clear**: Well-documented, obvious intent
