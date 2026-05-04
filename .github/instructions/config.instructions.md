---
name: config-instructions
description: "Configuration file instructions: YAML/JSON validation, balance parameters, game constants, schema consistency"
applyTo: "config/**/*.yaml", "config/**/*.yml", "config/**/*.json"
---

# Configuration Instructions

## Scope

These instructions apply to all configuration files in `config/`. Configuration is the **single source of truth** for game balance and parameters.

## Core Principles

### 1. Configuration as Data
- Configuration **values** belong in files
- **Logic** belongs in code
- Never hardcode game balance parameters in code

```yaml
# ✅ Good - Configuration file
units:
  warrior:
    base_health: 100
    base_damage: 15
    movement_range: 5
  archer:
    base_health: 60
    base_damage: 20
    movement_range: 6
```

```python
# ✅ Good - Load from configuration
@dataclass
class UnitStats:
    health: int
    damage: int
    movement_range: int

def load_unit_config(path: str) -> dict[str, UnitStats]:
    with open(path) as f:
        data = yaml.safe_load(f)
    
    return {
        name: UnitStats(**stats)
        for name, stats in data['units'].items()
    }
```

### 2. Validation
Every configuration file must be **validated before use**:

```python
# ✅ Good - Validate schema
def validate_unit_config(config: dict) -> None:
    """Validate unit configuration schema."""
    required_fields = {'health', 'damage', 'movement_range'}
    
    for unit_name, stats in config.items():
        missing = required_fields - set(stats.keys())
        if missing:
            raise ValueError(
                f"Unit '{unit_name}' missing fields: {missing}"
            )
        
        # Validate values
        if stats['health'] <= 0:
            raise ValueError(
                f"Unit '{unit_name}' health must be > 0"
            )
        if stats['damage'] < 0:
            raise ValueError(
                f"Unit '{unit_name}' damage cannot be negative"
            )

# ❌ Avoid - No validation
def load_units(path):
    with open(path) as f:
        return yaml.safe_load(f)
    # What if file is malformed?
```

### 3. Clear Comments
Configuration values should **explain the intent**:

```yaml
# ✅ Good - Explains design decision
terrain:
  plains:
    movement_cost: 1      # Base movement cost
    defense_bonus: 0      # No defense benefit
    vision_modifier: 1.0  # No vision penalty
  
  mountain:
    movement_cost: 3      # 3x slower to cross
    defense_bonus: 2      # +2 defense (hard to reach)
    vision_modifier: 0.8  # 20% reduced vision range

# ❌ Avoid - Magic numbers
terrain:
  plains:
    mc: 1
    db: 0
    vm: 1.0
```

## File Organization

### Recommended Structure
```
config/
├── units/
│   ├── base_units.yaml      # Unit definitions by era
│   ├── unit_evolutions.yaml # Evolution chains
│   └── unit_abilities.yaml  # Special abilities
├── terrain/
│   └── terrain_types.yaml   # Terrain properties
├── balance/
│   ├── game_balance.yaml    # Core balance parameters
│   ├── combat.yaml          # Combat modifiers
│   └── economy.yaml         # Resource values
├── tech/
│   ├── tech_tree.yaml       # Technology DAG
│   └── era_progression.yaml # Era unlocks
└── factions/
    └── faction_start.yaml   # Faction starting compositions
```

## Configuration Templates

### Units Configuration
```yaml
# units/base_units.yaml
units:
  warrior:
    name: "Warrior"
    description: "Tough melee fighter"
    era: IRON_AGE
    
    # Combat Stats
    base_health: 100
    base_damage: 15
    attack_range: 1
    attack_speed: 1.0
    
    # Movement
    movement_range: 5
    movement_speed: 1.0
    
    # Visibility
    vision_range: 6
    
    # Modifiers
    terrain_affinity:
      plains: 1.0      # No modifier
      forest: 0.9      # 10% slower in forest
      mountain: 0.7    # 30% slower on mountain
  
  archer:
    name: "Archer"
    description: "Ranged attacker"
    era: IRON_AGE
    
    base_health: 60
    base_damage: 20
    attack_range: 4
    attack_speed: 0.8
    
    movement_range: 6
    movement_speed: 1.1
    
    vision_range: 8
```

### Terrain Configuration
```yaml
# terrain/terrain_types.yaml
terrain:
  plains:
    name: "Plains"
    movement_cost: 1
    defense_bonus: 0
    vision_modifier: 1.0
    description: "Open grassland (fast movement)"
  
  forest:
    name: "Forest"
    movement_cost: 2       # 2x slower
    defense_bonus: 1       # +1 defense (cover)
    vision_modifier: 0.8   # 20% reduced vision (dense)
    description: "Dense forest (slow, defensive)"
  
  mountain:
    name: "Mountain"
    movement_cost: 3       # 3x slower
    defense_bonus: 2       # +2 defense (elevated)
    vision_modifier: 0.6   # 40% reduced vision (blocked sight)
    description: "Mountain peak (very slow, very defensive)"
  
  water:
    name: "Water"
    movement_cost: null    # Impassable
    defense_bonus: 0
    vision_modifier: 1.0
    description: "Water (impassable for ground units)"
```

### Combat Configuration
```yaml
# balance/combat.yaml
combat:
  # Damage calculation
  base_damage_scaling: 1.0
  
  # Modifiers
  visibility_bonus: 0.0       # No bonus for visible targets
  hidden_penalty: 1.0         # Invisible = 100% damage reduction
  
  # Terrain
  terrain_defense_bonus:
    plains: 0
    forest: 1
    mountain: 2
  
  # Health mechanics
  overkill_tracking: true     # Track excess damage
  minimum_damage: 0           # Minimum damage dealt (0 = no minimum)
  
  # Status effects
  critical_multiplier: 1.5    # 1.5x damage on critical hit
  critical_chance: 0.1        # 10% base critical chance
```

### Technology Tree Configuration
```yaml
# tech/tech_tree.yaml
technologies:
  bronze_working:
    name: "Bronze Working"
    era: BRONZE_AGE
    cost: 100
    duration: 10  # turns
    description: "Unlock bronze tools and weapons"
    prerequisites: []  # No prerequisites
  
  iron_smelting:
    name: "Iron Smelting"
    era: IRON_AGE
    cost: 200
    duration: 20
    description: "Unlock iron weapons (stronger than bronze)"
    prerequisites: [bronze_working]  # Requires bronze first
  
  steel_forging:
    name: "Steel Forging"
    era: RENAISSANCE
    cost: 400
    duration: 30
    description: "Unlock steel equipment (strongest)"
    prerequisites: [iron_smelting]

# Validate: No cycles, all prerequisites exist
```

### Faction Configuration
```yaml
# factions/faction_start.yaml
factions:
  player:
    faction_type: PLAYER
    starting_era: STONE_AGE
    starting_position: [10, 10]  # World position
    starting_units:
      - unit_type: warrior
        count: 2
      - unit_type: scout
        count: 1
    starting_resources:
      gold: 50
      population: 5
  
  cpu_easy:
    faction_type: CPU
    difficulty: EASY
    starting_era: STONE_AGE
    starting_position: [5, 15]
    starting_units:
      - unit_type: warrior
        count: 1
      - unit_type: scout
        count: 1
    starting_resources:
      gold: 40
      population: 3
```

## Validation Patterns

### Schema Validation
```python
import jsonschema

UNIT_SCHEMA = {
    "type": "object",
    "properties": {
        "name": {"type": "string"},
        "base_health": {"type": "integer", "minimum": 1},
        "base_damage": {"type": "integer", "minimum": 0},
        "movement_range": {"type": "integer", "minimum": 1},
    },
    "required": ["name", "base_health", "base_damage"],
}

def validate_unit_config(config: dict) -> None:
    """Validate against schema."""
    try:
        jsonschema.validate(config, UNIT_SCHEMA)
    except jsonschema.ValidationError as e:
        raise ValueError(f"Invalid unit config: {e.message}")
```

### Business Logic Validation
```python
def validate_tech_tree(config: dict) -> None:
    """Validate technology tree is acyclic and consistent."""
    techs = config.get('technologies', {})
    
    # Check no cycles
    for tech_name, tech in techs.items():
        visited = set()
        if _has_cycle(tech_name, techs, visited):
            raise ValueError(f"Technology cycle detected at {tech_name}")
    
    # Check all prerequisites exist
    for tech_name, tech in techs.items():
        for prereq in tech.get('prerequisites', []):
            if prereq not in techs:
                raise ValueError(
                    f"Tech '{tech_name}' requires unknown tech '{prereq}'"
                )

def _has_cycle(tech_name: str, techs: dict, visited: set) -> bool:
    """Check if tech has circular dependency."""
    if tech_name in visited:
        return True
    
    visited.add(tech_name)
    
    for prereq in techs[tech_name].get('prerequisites', []):
        if _has_cycle(prereq, techs, visited.copy()):
            return True
    
    return False
```

## Loading & Caching

```python
class ConfigManager:
    """Manage game configuration loading and caching."""
    
    def __init__(self, config_dir: str):
        self.config_dir = config_dir
        self._cache = {}
    
    def load_config(self, filename: str) -> dict:
        """Load and cache configuration."""
        if filename in self._cache:
            return self._cache[filename]
        
        path = os.path.join(self.config_dir, filename)
        
        with open(path) as f:
            if filename.endswith('.yaml') or filename.endswith('.yml'):
                config = yaml.safe_load(f)
            elif filename.endswith('.json'):
                config = json.load(f)
            else:
                raise ValueError(f"Unknown config format: {filename}")
        
        # Validate
        self._validate(filename, config)
        
        # Cache
        self._cache[filename] = config
        
        return config
    
    def _validate(self, filename: str, config: dict):
        """Validate configuration."""
        if 'units' in filename:
            validate_unit_config(config)
        elif 'terrain' in filename:
            validate_terrain_config(config)
        elif 'tech' in filename:
            validate_tech_tree(config)
        # ... etc
```

## Version Control

- ✅ **Commit config files**: They're data + design decisions
- ✅ **Add comments**: Explain why values changed
- ✅ **Review balance changes**: Config affects game feel
- ✅ **Tag releases**: Mark stable configurations

Example commit:
```
Increase warrior damage from 15 to 18

Players reported warriors underpowered compared to archers.
This brings warrior DPS closer to archer baseline while
maintaining ranged advantage through attack range.

Changed: config/units/base_units.yaml
- warrior.base_damage: 15 → 18
```

## Best Practices

1. **Single Responsibility**: Each file has one purpose
2. **No Magic Numbers**: Every parameter has a reason (in comments)
3. **Validation**: Always validate before use
4. **Caching**: Load once, reuse throughout game
5. **Comments**: Explain the *why* of values
6. **Defaults**: Provide sensible defaults for missing values
7. **Consistency**: Naming and structure consistent across files

## Testing Configuration

```python
def test_unit_config_valid():
    """Configuration loads and validates."""
    config_mgr = ConfigManager("config")
    units = config_mgr.load_config("units/base_units.yaml")
    
    assert "warrior" in units
    assert units["warrior"]["base_health"] > 0

def test_tech_tree_acyclic():
    """Technology tree has no cycles."""
    config_mgr = ConfigManager("config")
    techs = config_mgr.load_config("tech/tech_tree.yaml")
    
    # Should not raise
    validate_tech_tree(techs)

def test_all_configs_load():
    """All configuration files load without error."""
    config_dir = "config"
    
    for root, dirs, files in os.walk(config_dir):
        for file in files:
            if file.endswith(('.yaml', '.yml', '.json')):
                path = os.path.join(root, file)
                
                # Should load without error
                with open(path) as f:
                    if file.endswith('.json'):
                        json.load(f)
                    else:
                        yaml.safe_load(f)
```

## Summary

Write configurations that are:
- ✅ **Validated**: Schema and logic checks before use
- ✅ **Documented**: Comments explain all parameters
- ✅ **Organized**: Clear folder structure
- ✅ **Consistent**: Naming and format uniform
- ✅ **Cached**: Loaded once, reused
- ✅ **Testable**: Configuration integrity verified
- ✅ **Versionable**: Track balance changes
