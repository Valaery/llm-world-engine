---
tags: [index, schemas, data-structures, reference]
date: 2026-01-17
status: complete
---

# State & Schema Reference Index

Quick reference to data structures and schemas documented across the LLM World Engine knowledge base.

## Overview

This index catalogs all data structures, state schemas, type systems, and API formats extracted from the Discord discussions. Rather than duplicating content, this index points to schemas in their natural context within patterns, prompts, and specifications.

**Philosophy**: Schemas are documented alongside the patterns that use them, ensuring context and preventing duplication.

## State Management Schemas

### Three-Tier Persistence
**Location**: [[patterns/state/three-tier-persistence]]

Complete state management schema with three layers:

```python
@dataclass
class WorldState:
    """Persistent world data (survives all sessions)"""
    locations: Dict[str, Location]
    npcs: Dict[str, NPC]
    world_facts: List[str]
    current_time: GameTime

@dataclass
class PlaythroughState:
    """Single playthrough data (character-specific)"""
    character: PlayerCharacter
    inventory: List[Item]
    completed_quests: List[str]
    relationships: Dict[str, int]

@dataclass
class SessionState:
    """Temporary session data (cleared on exit)"""
    current_location: str
    active_npcs: List[str]
    conversation_history: List[Message]
```

### Scene-Based State Boundaries
**Location**: [[patterns/state/scene-based-boundaries]]

Scene state structure:
- Scene metadata (id, type, location)
- Active entities and their states
- Scene-specific flags and variables
- Save/load point definitions

### Conditional Persistence
**Location**: [[patterns/state/conditional-persistence]]

Rules for what to persist:
- **Always persist**: Quest progress, inventory, relationships, world changes
- **Session-only**: Conversation history, temporary buffs, UI state
- **Conditional**: Combat state (persist if interrupted), generated content (persist if interacted with)

## API & Integration Schemas

### JSON API Abstraction Layer
**Location**: [[patterns/integration/api-abstraction-layer]]

Complete API schemas for LLM-game communication:

```json
{
  "action": "generate_narration",
  "params": {
    "event_type": "action",
    "ndl_markup": "do($player, 'attack') -> target($goblin) -> result('hit')",
    "context": {
      "location": "dark_cave",
      "nearby_npcs": ["goblin_warrior"],
      "recent_events": []
    }
  }
}
```

Response format:
```json
{
  "status": "success",
  "narration": "You swing your sword at the goblin...",
  "metadata": {
    "tokens_used": 45,
    "model": "gpt-3.5-turbo"
  }
}
```

### State-to-LLM Injection Format
**Location**: [[patterns/integration/state-to-llm-injection]]

How game state is formatted for LLM context:
- Entity serialization formats
- Context window management
- Priority-based state selection
- Compressed state representations

### Multi-Model Routing Schemas
**Location**: [[patterns/integration/multi-model-routing]]

Model selection and routing:
```python
TaskRoute = {
    "narration": "gpt-3.5-turbo",
    "generation": "gpt-4",
    "dialogue": "claude-sonnet",
    "validation": "local-7b"
}
```

## NDL Type System

### Parameter Types
**Location**: [[ndl/specification/04-type-system]]

Complete type system for NDL parameters:
- **EntityRef**: `$"entity_name"` - References to game entities
- **String**: Quoted strings for actions, descriptions
- **Identifier**: Unquoted identifiers for keywords
- **PropertyMap**: Key-value pairs `key="value"`

### Grammar Specification
**Location**: [[ndl/specification/02-grammar]]

Formal EBNF grammar defining NDL syntax structure.

### Semantic Rules
**Location**: [[ndl/specification/03-semantics]]

Type checking and validation rules for NDL constructs.

## Generation Output Schemas

### Character Generation Schema
**Location**: [[prompts/generation/character-generation]]

Complete character template structure:
```json
{
  "name": "Character Name",
  "role": "Job or archetype",
  "personality": "2-3 defining traits",
  "appearance": "Physical description",
  "background": "Brief backstory",
  "motivation": "What drives them",
  "equipment": ["item1", "item2"],
  "abilities": ["ability1", "ability2"],
  "hooks": ["quest hook 1", "quest hook 2"],
  "stats": {
    "level": 5,
    "health": 45,
    "class": "warrior"
  }
}
```

### Location Generation Schema
**Location**: [[prompts/generation/location-generation]]

Complete location template structure:
```json
{
  "name": "Location Name",
  "type": "dungeon|city|wilderness|building",
  "parent_location": "Containing region",
  "description": "2-3 sentence overview",
  "atmosphere": "Mood and feeling",
  "features": [
    "Notable feature 1 with sensory details",
    "Notable feature 2 with gameplay implications"
  ],
  "hooks": [
    "Quest hook 1",
    "Mystery or secret"
  ],
  "stats": {
    "size": "small|medium|large",
    "wealth": "poor|modest|wealthy",
    "security": "none|light|heavy",
    "reputation": "unknown|neutral|famous"
  },
  "mechanics": {
    "traps": "Trap descriptions if any",
    "enemies": "Enemy encounters",
    "treasure": "Rewards and loot"
  }
}
```

### Template Meta-Generation
**Location**: [[prompts/generation/template-generation]]

Schemas for LLM-generated templates (meta-prompts):
- JSON schema definitions
- Template variable syntax
- Inheritance hierarchies
- Validation rules

## Action & Event Schemas

### Social Action Catalog
**Location**: [[ndl/patterns/social-dynamics]]

43+ cataloged social actions with parameters:
- Greeting patterns (wave, nod, bow, salute)
- Conversation starters (ask, inquire, question)
- Persuasion attempts (convince, bargain, intimidate)
- Relationship actions (befriend, flirt, insult)

Each with schema: `action(entity, target, manner?, intention?)`

### Combat Action Schema
**Location**: [[ndl/patterns/combat-narration]]

Combat sequence structure:
```ndl
do($attacker, "attack") ~ "manner"
  -> target($defender)
  -> result("hit"|"miss"|"critical")
  -> damage(value)
```

### Scene Transition Schema
**Location**: [[ndl/patterns/scene-transitions]]

Scene metadata structure:
- Scene ID and type (14 categories)
- Entry/exit conditions
- State preservation rules
- Transition triggers

## Constraint Formats

### Format Enforcement Schemas
**Location**: [[prompts/constraint/format-enforcement]]

Bracketed output formats for reliable parsing:
```
[ITEM1],[ITEM2],[ITEM3]
[YES] or [NO]
[LOCATION: name]
[ACTION: verb]
```

### Binary Classification Output
**Location**: [[prompts/reasoning/binary-classification]]

Question tree format:
```json
{
  "question": "Is the door locked?",
  "answer": "[YES]",
  "confidence": "high",
  "reasoning": "The description mentions a padlock."
}
```

## Validation & Testing Schemas

### Anti-Hallucination Rules
**Location**: [[prompts/constraint/anti-hallucination]]

Constraint rule format:
```
RULE: Impossible actions must fail
VALIDATION: Check against entity capabilities
FORMAT: result("failure") -> system_response("explanation")
```

### HyDE Query Schema
**Location**: [[prompts/retrieval/query-formulation-hyde]]

Hypothetical document format for improved retrieval:
```json
{
  "original_query": "How do I pick the lock?",
  "hypothetical_questions": [
    "What tools are needed for lockpicking?",
    "What skill check is required?",
    "What happens if lockpicking fails?"
  ],
  "search_terms": ["lockpicking", "tools", "skill check", "failure"]
}
```

## Usage Guidelines

### Finding Schemas

1. **For state management**: Check `patterns/state/` directory
2. **For generation**: Check `prompts/generation/` directory
3. **For NDL types**: Check `ndl/specification/` directory
4. **For API formats**: Check `patterns/integration/` directory

### Schema Evolution

All schemas are production-tested from:
- **ReallmCraft** (veritasr's Minecraft engine)
- **ChatBot RPG** (monkeyrithms' text RPG)
- **DirectorAPI** (Community experiments)

### Implementation Notes

- **Language**: Most examples use Python with type hints (TypedDict, dataclass)
- **Serialization**: JSON for API boundaries, native types for internal state
- **Validation**: Schema validation recommended at API boundaries only
- **Flexibility**: Schemas are guidelines, not rigid requirements

## Related Documentation

- [[patterns/00-PATTERN-INDEX]] - Architectural patterns using these schemas
- [[prompts/00-PROMPT-INDEX]] - Prompts that output these schemas
- [[ndl/00-NDL-INDEX]] - NDL language specification and types

## Contributing

When adding new schemas:
1. Document them in context (within relevant pattern/prompt)
2. Add reference to this index
3. Include working code examples
4. Specify which projects use this schema
5. Note validation requirements

---

**Last Updated**: 2026-01-17
**Coverage**: Complete (all identifiable schemas from 24 months of Discord discussions)
**Source**: LLM World Engine Discord (Jan 2024 - Dec 2025)
