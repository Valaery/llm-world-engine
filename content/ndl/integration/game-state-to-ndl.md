---
tags: [ndl, integration, game-state, conversion, draft]
type: Integration Guide
category: NDL Implementation
status: draft
---

# Converting Game State to NDL

## Overview

**Purpose**: Guide for converting programmatic game state into NDL markup
**Status**: DRAFT - Awaiting detailed specification from ndl-reference-builder agent
**Use Case**: Backend developers implementing NDL generation

## Placeholder Structure

### Conversion Pipeline

```
Game Event → State Change → NDL Generator → NDL Markup → LLM
```

### Expected Content

**State to NDL Mapping**:
- Player actions → [action] constructs
- NPC behavior → [character] + [dialogue]
- World events → [scene] + [event]
- Combat outcomes → [combat] constructs
- Item interactions → [item] + [effect]

**Code Examples** (To be added):
```python
def game_state_to_ndl(event, state):
    # Convert Python game state to NDL markup
    pass
```

## Related Documentation

- [[ndl/specification/formal-grammar]] - NDL grammar
- [[ndl/integration/llm-integration]] - LLM integration
- [[ReallmCraft-Project]] - Reference implementation

## Implementation Notes

This is a placeholder stub. Full guide needs:
- Complete conversion patterns
- Code examples from ReallmCraft
- Best practices and gotchas
- Performance considerations
- Error handling

---

> [!warning] Draft Status
> This file is a stub placeholder. Run the ndl-reference-builder agent to extract complete specification from Discord transcripts and veritasr's implementation notes.
