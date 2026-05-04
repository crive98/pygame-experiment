---
name: rendering-instructions
description: "Graphics rendering instructions: pygame performance, frame budget, surface caching, animations, optimization"
applyTo: "src/graphics/**/*.py", "src/ui/**/*.py"
---

# Rendering Instructions

## Scope

These instructions apply to all rendering and UI code. The goal is **60 FPS performance with responsive, clear visuals**.

## Performance Budget

```
Frame time: 16.67ms @ 60 FPS

Game Logic:        5.0ms
Rendering:         10.0ms
UI:                1.5ms
Buffer:            0.2ms
```

Stay within budget or frame rate drops.

## Core Principles

### 1. Performance First
Optimize for speed before visual quality:

```python
# ✅ Good - Caches transformed surfaces
class UnitRenderer:
    def __init__(self):
        self._sprite_cache = {}  # Cache transformed sprites
    
    def render_unit(self, unit: Unit, screen: pygame.Surface):
        """Render unit with cached sprite."""
        # Get from cache (fast)
        sprite_key = (unit.sprite_id, unit.size)
        if sprite_key not in self._sprite_cache:
            # Transform once, cache result
            self._sprite_cache[sprite_key] = pygame.transform.scale(
                self._load_sprite(unit.sprite_id),
                unit.size
            )
        
        cached_sprite = self._sprite_cache[sprite_key]
        screen.blit(cached_sprite, unit.screen_position)

# ❌ Avoid - Transforms every frame
def render_unit(unit, screen):
    sprite = load_sprite(unit.sprite_id)
    transformed = pygame.transform.scale(sprite, unit.size)
    screen.blit(transformed, unit.pos)
```

### 2. Minimize Blit Operations
Fewer draw calls = faster rendering:

```python
# ✅ Good - Batches similar objects
def render_units(units: list[Unit], screen: pygame.Surface):
    """Render all units efficiently."""
    # Draw all units at once (same sprite)
    for unit in sorted(units, key=lambda u: u.depth):
        screen.blit(self._get_cached_sprite(unit), unit.screen_pos)

# ❌ Avoid - Unnecessary batching complexity
def render_units(units, screen):
    by_sprite = {}
    for unit in units:
        if unit.sprite not in by_sprite:
            by_sprite[unit.sprite] = []
        by_sprite[unit.sprite].append(unit)
    # Overcomplicated
```

### 3. Dirty Rectangle Optimization
Only update screen areas that changed:

```python
# ✅ Good - Partial update
def update_display(dirty_rects: list[pygame.Rect]):
    """Update only changed areas."""
    if dirty_rects:
        pygame.display.update(dirty_rects)
    else:
        pygame.display.update()  # Full update if needed

# Track dirty rects
dirty_rects = []
if unit_moved:
    dirty_rects.append(old_position_rect)
    dirty_rects.append(new_position_rect)

update_display(dirty_rects)

# ❌ Avoid - Always full refresh
pygame.display.flip()  # Every frame!
```

### 4. Resource Management
Load assets once, reuse forever:

```python
# ✅ Good - Load once, cache
class AssetManager:
    def __init__(self):
        self._sprites = {}
        self._fonts = {}
    
    def get_sprite(self, sprite_id: str) -> pygame.Surface:
        if sprite_id not in self._sprites:
            self._sprites[sprite_id] = pygame.image.load(
                f"assets/sprites/{sprite_id}.png"
            )
        return self._sprites[sprite_id]
    
    def get_font(self, size: int) -> pygame.font.Font:
        if size not in self._fonts:
            self._fonts[size] = pygame.font.Font(
                "assets/fonts/default.ttf", size
            )
        return self._fonts[size]

# ❌ Avoid - Load every frame
def render_text(text):
    font = pygame.font.Font("assets/fonts/default.ttf", 16)  # Loaded!
    return font.render(text, True, (255, 255, 255))
```

## Rendering Architecture

### Layered Rendering
```python
class Renderer:
    """Manages rendering in layers."""
    
    def render_frame(self, world: GameWorld, screen: pygame.Surface):
        """Render complete frame with proper layer order."""
        # Clear screen
        screen.fill((0, 0, 0))
        
        # Layer 1: Terrain (background)
        self._render_terrain(world, screen)
        
        # Layer 2: Units (mid)
        self._render_units(world, screen)
        
        # Layer 3: Effects (particles, etc.)
        self._render_effects(world, screen)
        
        # Layer 4: UI (foreground)
        self._render_ui(world, screen)
        
        # Update display
        pygame.display.flip()
```

### Camera System
```python
class Camera:
    """Manage viewport and world-to-screen conversion."""
    
    def __init__(self, width: int, height: int):
        self.x = 0
        self.y = 0
        self.zoom = 1.0
        self.width = width
        self.height = height
    
    def world_to_screen(self, world_pos: tuple[int, int]) -> tuple[int, int]:
        """Convert world position to screen coordinates."""
        screen_x = int((world_pos[0] - self.x) * self.zoom)
        screen_y = int((world_pos[1] - self.y) * self.zoom)
        return (screen_x, screen_y)
    
    def screen_to_world(self, screen_pos: tuple[int, int]) -> tuple[int, int]:
        """Convert screen position to world coordinates."""
        world_x = int(screen_pos[0] / self.zoom + self.x)
        world_y = int(screen_pos[1] / self.zoom + self.y)
        return (world_x, world_y)
    
    def pan_to_unit(self, unit: Unit):
        """Center camera on unit."""
        screen_center_x = self.width // 2
        screen_center_y = self.height // 2
        
        self.x = unit.position[0] - screen_center_x // self.zoom
        self.y = unit.position[1] - screen_center_y // self.zoom
        
        # Clamp to world bounds
        self.x = max(0, min(self.x, world_width - self.width // self.zoom))
        self.y = max(0, min(self.y, world_height - self.height // self.zoom))
```

## Animation System

```python
@dataclass
class Animation:
    """Simple animation for unit movement."""
    unit: Unit
    start_pos: tuple[int, int]
    end_pos: tuple[int, int]
    duration: float  # seconds
    elapsed: float = 0.0
    
    def update(self, delta_time: float):
        """Update animation progress."""
        self.elapsed += delta_time
        
        if self.elapsed >= self.duration:
            self.unit.position = self.end_pos
            return True  # Animation complete
        
        # Interpolate position
        progress = self.elapsed / self.duration
        current_pos = (
            int(self.start_pos[0] + 
                (self.end_pos[0] - self.start_pos[0]) * progress),
            int(self.start_pos[1] + 
                (self.end_pos[1] - self.start_pos[1]) * progress),
        )
        self.unit.screen_position = current_pos
        
        return False  # Still animating

class AnimationManager:
    """Manage multiple active animations."""
    
    def __init__(self):
        self.active_animations: list[Animation] = []
    
    def play_movement(self, unit: Unit, end_pos: tuple[int, int]):
        """Play unit movement animation."""
        anim = Animation(
            unit=unit,
            start_pos=unit.position,
            end_pos=end_pos,
            duration=0.3  # 300ms movement
        )
        self.active_animations.append(anim)
    
    def update(self, delta_time: float):
        """Update all active animations."""
        for anim in self.active_animations[:]:
            if anim.update(delta_time):
                self.active_animations.remove(anim)
```

## UI Rendering

```python
class UIPanel:
    """Base class for UI panels."""
    
    def __init__(self, rect: pygame.Rect):
        self.rect = rect
        self.surface = pygame.Surface((rect.width, rect.height))
    
    def draw(self, parent_surface: pygame.Surface):
        """Draw panel to parent surface."""
        self.surface.fill((50, 50, 50))  # Background
        self._render_content(self.surface)
        parent_surface.blit(self.surface, self.rect)
    
    def _render_content(self, surface: pygame.Surface):
        """Override to draw panel content."""
        pass

class HUD:
    """Heads-up display with game info."""
    
    def __init__(self, screen_width: int, screen_height: int):
        self.health_panel = UIPanel(pygame.Rect(10, 10, 200, 60))
        self.turn_counter = UIPanel(pygame.Rect(screen_width - 210, 10, 200, 60))
    
    def render(self, screen: pygame.Surface, game_state: GameState):
        """Render HUD to screen."""
        self._render_health_info(self.health_panel.surface, game_state)
        self._render_turn_info(self.turn_counter.surface, game_state)
        
        self.health_panel.draw(screen)
        self.turn_counter.draw(screen)
```

## Performance Monitoring

```python
class PerformanceMonitor:
    """Track rendering performance."""
    
    def __init__(self):
        self.frame_times = []
        self.max_history = 60
    
    def measure_frame(self, delta_time: float):
        """Record frame time."""
        self.frame_times.append(delta_time * 1000)  # Convert to ms
        
        if len(self.frame_times) > self.max_history:
            self.frame_times.pop(0)
    
    def get_average_fps(self) -> float:
        """Get average FPS over recent frames."""
        if not self.frame_times:
            return 0
        
        avg_time = sum(self.frame_times) / len(self.frame_times)
        return 1000 / avg_time if avg_time > 0 else 0
    
    def is_frame_slow(self) -> bool:
        """Check if frame exceeded budget."""
        if not self.frame_times:
            return False
        
        budget_ms = 16.67  # 60 FPS
        return self.frame_times[-1] > budget_ms
```

## Profiling Checklist

Before optimization:
- [ ] Measure current performance (profile)
- [ ] Identify bottleneck (frame profiler)
- [ ] Estimate improvement
- [ ] Implement optimization
- [ ] Verify improvement
- [ ] Document optimization

## Common Optimizations

### Sprite Sheet Usage
```python
# ✅ Use sprite sheets instead of individual files
sprite_sheet = pygame.image.load("assets/units_spritesheet.png")
unit_sprite = sprite_sheet.subsurface(pygame.Rect(0, 0, 32, 32))
```

### Surface Pre-rendering
```python
# ✅ Pre-render static text
unit_health_text = font.render("HP: 100", True, (255, 0, 0))
# Reuse each frame instead of rendering

# ❌ Avoid re-rendering every frame
for unit in units:
    text = font.render(f"HP: {unit.health}", True, (255, 0, 0))
```

### Viewport Culling
```python
# ✅ Only render visible units
def render_units_culled(self, units, screen, camera):
    for unit in units:
        # Check if in viewport
        if not self._is_visible(unit, camera):
            continue
        
        screen.blit(unit.sprite, camera.world_to_screen(unit.pos))
```

## Summary

Write rendering code that is:
- ✅ **Fast**: Hits 60 FPS budget
- ✅ **Cached**: Reuses expensive calculations
- ✅ **Batched**: Minimizes draw calls
- ✅ **Responsive**: Updates at frame rate
- ✅ **Clear**: Layer structure obvious
- ✅ **Profiled**: Performance measured
- ✅ **Tested**: Visual correctness verified
