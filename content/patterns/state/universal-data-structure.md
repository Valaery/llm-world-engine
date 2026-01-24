---
tags: [pattern, state, data-structure, schema, draft]
category: State Patterns
status: placeholder
---

# Universal Data Structure Pattern

## Overview

**Problem**: Managing multiple entity types (NPCs, items, locations) with different schemas creates complexity and duplication.

**Solution**: Design a single universal data structure that can represent all entity types through flexible attribute systems.

**Status**: PLACEHOLDER - Needs extraction from Discord discussions and implementation examples.

## Intent

Create a unified entity schema that:
- Reduces code duplication
- Simplifies serialization/deserialization
- Enables generic entity operations
- Supports dynamic attribute addition
- Maintains type safety where needed

## Structure (Placeholder)

```python
class UniversalEntity:
    """
    Single entity structure for all game objects.
    """
    def __init__(self):
        self.id: str
        self.type: str  # 'npc', 'item', 'location', etc.
        self.attributes: Dict[str, Any]  # Flexible attributes
        self.metadata: Dict[str, Any]  # System data
```

## Expected Content

### Design Patterns
- Attribute-based entity system
- Type discrimination through attributes
- Schema validation for specific types
- Inheritance vs composition trade-offs

### Implementation Approaches
- JSON-based universal schema
- Class hierarchies with common base
- Entity-Component-System (ECS) architecture
- Hybrid approaches

### Trade-offs
- Flexibility vs type safety
- Performance vs ease of use
- Validation complexity
- Query efficiency

## Related Patterns

- [[state/template-driven-entities]] - Template instantiation
- [[state/scene-based-boundaries]] - State persistence
- [[generation/template-meta-generation]] - Template generation

## Related Projects

- [[ReallmCraft-Project]] - Template-based entity system
- [[ChatBotRPG-Project]] - JSON/TXT hybrid approach

---

> [!warning] Placeholder Status
> This pattern needs detailed extraction from Discord discussions. Run architecture-pattern-extractor agent to complete.

> [!todo] Required Content
> - Extract veritasr's entity schema discussions
> - Add appl2613's JSON approach from ChatBotRPG
> - Include code examples and best practices
> - Document trade-offs and decision criteria
