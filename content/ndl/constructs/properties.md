---
tags: [ndl, construct, properties, attributes, draft]
type: NDL Construct
category: Core Syntax
status: draft
---

# NDL Properties Construct

## Overview

**Purpose**: Define key-value attributes for entities, scenes, and other NDL elements
**Status**: DRAFT - Awaiting detailed specification from ndl-reference-builder agent
**Category**: Core NDL syntax construct

## Placeholder Structure

### Basic Syntax
```ndl
[entity:character_name]
  [properties]
    key: value
    attribute: data
  [/properties]
[/entity]
```

### Expected Use Cases
- Entity attributes (health, stats, states)
- Scene properties (lighting, weather, mood)
- Item metadata (weight, value, durability)
- Dynamic property updates during gameplay

## Related Constructs

- [[ndl/constructs/entity]] - Entity definitions
- [[ndl/constructs/scene]] - Scene structure
- [[ndl/constructs/state]] - State management

## Implementation Notes

This is a placeholder stub. Full specification needs:
- Complete syntax definition
- Property types and validation
- Inheritance and scoping rules
- Examples from ReallmCraft implementation
- Integration with LLM processing

---

> [!warning] Draft Status
> This file is a stub placeholder. Run the ndl-reference-builder agent to extract complete specification from Discord transcripts.
