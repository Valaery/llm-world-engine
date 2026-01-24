---
tags: [pattern, generation, context, inheritance, draft]
category: Generation Patterns
status: placeholder
---

# Context Inheritance Pattern

## Overview

**Problem**: Generated child elements (items in locations, NPCs in scenes) should reflect parent context without explicit specification.

**Solution**: Child elements automatically inherit contextual properties from their parent container (location, scene, world).

**Status**: PLACEHOLDER - Needs extraction from Discord discussions and implementation examples.

## Intent

Enable context-aware generation where:
- NPCs in a tavern inherit "tavern" cultural context
- Items in a dungeon inherit "ancient" or "dangerous" themes
- Locations in regions inherit geographical/cultural traits
- Reduce explicit context specification in templates

## Structure (Placeholder)

```python
def generate_npc(location):
    """
    Generate NPC with inherited context from location.
    """
    context = {
        'culture': location.culture,
        'tech_level': location.tech_level,
        'economic_status': location.wealth,
        'climate': location.climate
    }

    npc = llm_generate_character(base_template, context)
    return npc
```

## Expected Content

### Inheritance Mechanisms
- Explicit property inheritance
- Implicit context in prompts
- Template composition
- Contextual defaults

### Hierarchy Levels
```
World → Region → Location → Scene → Entity
  ↓       ↓         ↓         ↓       ↓
Culture, Climate, Wealth, Mood, Attributes
```

### Implementation Patterns
- Cascading properties
- Context stacking
- Override mechanisms
- Conflict resolution

### Use Cases
- NPC generation in context
- Item crafting by location
- Dialogue tone from scene mood
- Quest generation from region

## Related Patterns

- [[generation/hierarchical-cascade]] - Top-down generation
- [[generation/just-in-time-generation]] - When to inherit
- [[state/template-driven-entities]] - Template inheritance

## Related Projects

- [[ReallmCraft-Project]] - Hierarchical world generation
- [[ChatBotRPG-Project]] - Context-aware NPC prompts

## Related Discussions

- [[04-World-Generation]] - Procedural content generation thread
- [[01-Architecture-and-Design]] - Template system design

---

> [!warning] Placeholder Status
> This pattern needs detailed extraction from Discord discussions. Run architecture-pattern-extractor agent to complete.

> [!todo] Required Content
> - Extract veritasr's hierarchical generation discussions
> - Document context stacking approaches
> - Add working code examples from ReallmCraft
> - Include prompt engineering techniques for inheritance
> - Cover edge cases and conflict resolution
