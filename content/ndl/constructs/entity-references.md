---
tags: [ndl, construct, entity, references, draft]
type: NDL Construct
category: Core Syntax
status: draft
---

# NDL Entity References

## Overview

**Purpose**: Reference and link entities within NDL markup
**Status**: DRAFT - Awaiting detailed specification from ndl-reference-builder agent
**Category**: Core NDL syntax for entity linking

## Placeholder Structure

### Basic Syntax
```ndl
[scene:location_name]
  [character:@character_id]
    ...
  [/character]

  [action|actor:@character_id|target:@other_entity]
    ...
  [/action]
[/scene]
```

### Expected Use Cases
- Reference NPCs in scenes
- Link actors to actions
- Cross-reference entities
- Track relationships between entities
- Maintain entity coherence across scenes

## Related Constructs

- [[ndl/constructs/character]] - Character definitions
- [[ndl/constructs/entity]] - Entity structure
- [[ndl/constructs/relationships]] - Relationship tracking

## Implementation Notes

This is a placeholder stub. Full specification needs:
- Reference syntax and resolution
- Entity ID formats and conventions
- Scoping rules for references
- Examples from ReallmCraft
- Integration with RAG retrieval

---

> [!warning] Draft Status
> This file is a stub placeholder. Run the ndl-reference-builder agent to extract complete specification from Discord transcripts.
