---
name: pygame-experiment-standards
description: "Core coding standards for pygame-experiment: production-grade Python, camelCase classes, snake_case methods, Google-style docstrings, performance-optimized"
---

# Pygame Experiment - Coding Standards

## Language & Framework
- **Language**: Python (3.12)
- **Primary Framework**: pygame
- **Paradigm**: Object-oriented design with clear separation of concerns

## Code Organization

### Class Naming
- Classes: **PascalCase** (e.g., `PlayerController`, `GameState`, `InputHandler`)
- Files containing classes: snake_case matching the primary class name (e.g., `player_controller.py`)

### Method & Function Naming
- Methods and functions: **snake_case** (e.g., `update_position()`, `handle_input()`, `render_frame()`)
- Private methods: prefix with underscore (e.g., `_validate_state()`)
- Constants: UPPER_SNAKE_CASE

### File Structure
- One primary class per file when possible
- Related utility functions in dedicated modules
- Clear, logical grouping in `src/` subdirectories

## Documentation

### Google Style Docstrings
Every class, method, and module must have Google-style docstrings:

**Classes**: Include description, attributes, and key behavior
**Methods/Functions**: Include description, Args, Returns, Raises
**Modules**: Include file purpose and main exports

Example:
```python
def calculate_collision(rect1: pygame.Rect, rect2: pygame.Rect) -> bool:
    """Detect rectangular collision between two game objects.
    
    Args:
        rect1: First collision rectangle.
        rect2: Second collision rectangle.
    
    Returns:
        True if rectangles overlap, False otherwise.
    """
```

## Code Quality Standards

### Robustness
- Type hints on all function signatures
- Input validation in critical paths
- Explicit error handling (don't silent-fail)
- Fail-fast principle: catch errors early

### Clarity & Maintainability
- Single Responsibility Principle: one class/function does one thing well
- Methods keep focused, average length <50 lines
- Use descriptive variable names over single letters
- Avoid magic numbers—use named constants

### Performance Optimization
- Profile before optimizing
- Prefer efficient data structures (deque > list for queues, set for lookups)
- Minimize per-frame allocations in game loop
- Cache pygame surfaces and transformations when appropriate
- Document performance-critical sections

### Additional Standards
- Import organization: stdlib, third-party, local
- No wildcard imports
- Prefer explicit over implicit