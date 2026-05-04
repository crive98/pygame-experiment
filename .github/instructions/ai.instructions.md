---
name: ai-instructions
description: "AI implementation instructions: visibility constraints, determinism, decision trees, CPU behavior, parameterized difficulty"
applyTo: "src/ai/**/*.py"
---

# AI Instructions

## Scope

These instructions apply to CPU AI in `src/ai/`. The AI must be **deterministic, tactically coherent, and honor visibility constraints**.

## Core Principles

### 1. Visibility Constraint (Critical!)
AI can **only use information visible to the unit**:

```python
# ✅ Good - Only visible information
def select_target(unit: Unit, world: GameWorld) -> Unit | None:
    """Select attack target from visible enemies.
    
    Constraint: Only consider units the AI unit can see.
    """
    # Get ONLY visible enemies
    visible_enemies = [
        u for u in world.all_units 
        if u.faction != unit.faction and unit.can_see(u)
    ]
    
    if not visible_enemies:
        return None
    
    # Choose from visible options
    return min(visible_enemies, key=lambda u: u.health)

# ❌ Avoid - Omniscient AI
def select_target(unit, world):
    """BUG: Can see everything!"""
    # This violates fog of war
    return min(world.all_enemy_units, key=lambda u: u.health)
```

### 2. Determinism
Same game state **must always produce the same decision**:

```python
# ✅ Good - Deterministic decision
def decide_action(unit: Unit, world: GameWorld) -> Action:
    """Decide next action deterministically.
    
    Same game state always produces same action.
    """
    # Consistent priority order
    if self._should_attack(unit, world):
        target = self._select_target(unit, world)
        return AttackAction(target)
    elif self._should_move(unit, world):
        destination = self._select_destination(unit, world)
        return MoveAction(destination)
    else:
        return WaitAction()

# ❌ Avoid - Non-deterministic
def decide_action(unit, world):
    return random.choice([attack, move, wait])  # Random!
```

### 3. Clear Decision Logic
Decision trees must be **obvious and traceable**:

```python
# ✅ Good - Clear decision tree
def decide_action(unit: Unit, world: GameWorld) -> Action:
    """Decide next action with clear priority.
    
    Decision tree:
    1. If general health < 30% → retreat
    2. If enemy general visible → attack
    3. If weak enemy visible → attack
    4. Otherwise → move toward enemy
    """
    # Priority 1: Retreat if general endangered
    if unit.health < unit.max_health * 0.3:
        return self._retreat(unit, world)
    
    # Priority 2: Attack general if visible
    visible_enemies = self._get_visible_enemies(unit, world)
    generals = [u for u in visible_enemies if u.is_general]
    if generals:
        return AttackAction(generals[0])
    
    # Priority 3: Attack weakest visible enemy
    if visible_enemies:
        weakest = min(visible_enemies, key=lambda u: u.health)
        if unit.can_attack(weakest):
            return AttackAction(weakest)
    
    # Priority 4: Move toward enemy
    return self._move_toward_enemy(unit, world)
```

## Decision-Making Framework

### Information Available to AI
- **Visible units**: Only units this unit can see
- **Visible terrain**: Terrain in field of view
- **Own unit stats**: Health, attack range, movement range
- **Known faction positions**: Where factions start (not current positions if hidden)

### Information NOT Available
- ❌ Hidden enemy units
- ❌ Enemy unit stats (unless revealed)
- ❌ Hidden enemy movements
- ❌ Global map state (only local visibility)

### Example: Target Selection

```python
def select_target(unit: Unit, world: GameWorld) -> Unit | None:
    """Select target using only visible information.
    
    Args:
        unit: AI unit making decision.
        world: Game world with visibility.
    
    Returns:
        Best visible target, or None if no valid target.
    
    Priority:
        1. Enemy general (highest priority)
        2. Weakest visible enemy
        3. Closest visible enemy (as tiebreaker)
    """
    # Get visible enemies only
    visible_enemies = [
        u for u in world.all_units 
        if u.faction != unit.faction and unit.can_see(u)
    ]
    
    if not visible_enemies:
        return None
    
    # Priority 1: General (if visible)
    generals = [u for u in visible_enemies if u.is_general]
    if generals:
        return generals[0]
    
    # Priority 2: Weakest (minimize health)
    targets_by_health = sorted(
        visible_enemies, 
        key=lambda u: u.health
    )
    
    # Priority 3: Tiebreaker by distance (closest first)
    closest = min(
        targets_by_health,
        key=lambda u: world.distance(unit.position, u.position)
    )
    
    return closest
```

## Difficulty Levels

AI behavior is parameterized by difficulty:

```python
@dataclass
class AIDifficulty:
    """Difficulty parameters for CPU AI."""
    
    reaction_time: float       # Delay before responding to threats
    decision_quality: float    # 0.0 (random) to 1.0 (optimal)
    risk_tolerance: float      # 0.0 (cautious) to 1.0 (aggressive)
    aggression: float          # 0.0 (passive) to 1.0 (attacking)

# Easy: Slower, riskier decisions
EASY = AIDifficulty(
    reaction_time=0.5,
    decision_quality=0.4,
    risk_tolerance=0.3,
    aggression=0.3,
)

# Hard: Fast, calculated decisions
HARD = AIDifficulty(
    reaction_time=0.0,
    decision_quality=0.9,
    risk_tolerance=0.7,
    aggression=0.8,
)

def decide_action(unit: Unit, world: GameWorld) -> Action:
    """Decide action based on difficulty."""
    difficulty = world.get_ai_difficulty(unit.faction)
    
    # Easier AI makes suboptimal choices
    if random.random() > difficulty.decision_quality:
        return self._make_random_action(unit, world)
    
    # Normal decision-making
    return self._make_optimal_action(unit, world)
```

## Pathfinding Integration

```python
def move_toward_goal(
    unit: Unit,
    goal: Position,
    world: GameWorld
) -> MoveAction | None:
    """Move toward goal using pathfinding.
    
    Args:
        unit: Unit to move.
        goal: Destination (might be approximated).
        world: Game world.
    
    Returns:
        Move action toward goal, or None if blocked.
    """
    # Pathfind to goal
    path = world.movement_system.pathfind(
        start=unit.position,
        goal=goal
    )
    
    if not path or len(path) < 2:
        return None  # No valid path
    
    # Move to first step of path
    next_position = path[1]
    
    return MoveAction(next_position)
```

## Common Decision Patterns

### Retreat Decision
```python
def should_retreat(unit: Unit, world: GameWorld) -> bool:
    """Determine if unit should retreat.
    
    Conditions:
    - Health < 30% of max
    - Outnumbered (more enemies visible than allies)
    - Weak relative to visible enemies
    """
    # Health check
    if unit.health < unit.max_health * 0.3:
        return True
    
    # Outnumbered check (visible units only)
    visible_allies = sum(
        1 for u in world.all_units 
        if u.faction == unit.faction and unit.can_see(u)
    )
    visible_enemies = sum(
        1 for u in world.all_units 
        if u.faction != unit.faction and unit.can_see(u)
    )
    
    return visible_enemies > visible_allies * 1.5
```

### Exploration Decision
```python
def select_exploration_direction(
    unit: Unit,
    world: GameWorld
) -> Position | None:
    """Select position to explore (expand visibility).
    
    Strategy: Move toward edges of known map that haven't
    been fully explored.
    """
    # Known positions within visibility range
    known_positions = set()
    for pos in world.get_positions_in_range(
        unit.position, 
        unit.stats.vision_range * 2
    ):
        if world.is_valid_position(pos):
            known_positions.add(pos)
    
    # Choose frontier position (edge of visibility)
    frontier = [p for p in known_positions 
                if not all_neighbors_explored(p, world)]
    
    if not frontier:
        return None
    
    return random.choice(frontier)
```

### Resource Defense
```python
def should_defend_general(unit: Unit, world: GameWorld) -> bool:
    """Decide if unit should protect the general.
    
    Units near general or general endangered should defend.
    """
    general = world.get_general(unit.faction)
    
    if general is None:
        return False  # General already dead
    
    # Distance to general
    distance = world.distance(unit.position, general.position)
    
    # Defend if:
    # 1. Close to general and enemy nearby
    # 2. General health low
    if distance < unit.stats.vision_range:
        enemy_nearby = any(
            u for u in world.all_units 
            if u.faction != unit.faction and unit.can_see(u)
        )
        if enemy_nearby:
            return True
    
    if general.health < general.max_health * 0.3:
        return True
    
    return False
```

## Testing AI

```python
def test_ai_respects_visibility():
    """AI only targets visible enemies."""
    world = GameWorld()
    ai_unit = create_unit(faction=Faction.CPU1)
    hidden_enemy = create_unit(faction=Faction.PLAYER)
    
    world.add_unit(ai_unit, pos=(0, 0))
    world.add_unit(hidden_enemy, pos=(20, 20))  # Far away
    
    # Make hidden enemy not visible
    ai_unit.visibility.clear()
    
    # AI should not target hidden enemy
    target = ai_unit.decide_target(world)
    assert target != hidden_enemy

def test_ai_deterministic():
    """Same state produces same decision."""
    world_state = GameWorld()
    ai_unit = create_unit()
    
    # Set up identical state twice
    decision1 = ai_unit.decide_action(world_state)
    decision2 = ai_unit.decide_action(world_state)
    
    assert decision1 == decision2
```

## Anti-Patterns

### ❌ Omniscience
```python
# DON'T: AI knows about all units
def select_target(unit, world):
    all_enemies = [u for u in world.all_units if u.is_enemy]
    return min(all_enemies, key=...)  # WRONG!
```

### ❌ Randomness Without Purpose
```python
# DON'T: Random decisions
def decide_action(unit, world):
    if random.random() > 0.5:
        return attack()
    else:
        return move()  # Non-deterministic!
```

### ❌ Complex Heuristics Without Documentation
```python
# DON'T: Magic formula
def priority(enemy):
    return 3 * enemy.health - 2 * distance + random_value()
    # What does this mean? Why these numbers?
```

## Summary

Write AI that is:
- ✅ **Constrained**: Only uses visible information
- ✅ **Deterministic**: Same input → same decision
- ✅ **Coherent**: Clear decision trees
- ✅ **Parameterized**: Difficulty levels
- ✅ **Tactical**: Makes reasonable choices
- ✅ **Documented**: Decision rationale clear
- ✅ **Tested**: Visibility and determinism validated
