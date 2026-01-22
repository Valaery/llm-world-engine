# NDL (Natural Description Language) - Index

#ndl #language-spec #llm-world-engine

## Overview

**NDL (Natural Description Language)** is a declarative language created by [[User-veritasr]] for bridging deterministic game logic with LLM-based narrative generation. It serves as a structured instruction format that eliminates LLM hallucinations by removing decision-making from the AI's responsibilities.

## Philosophy

> "Program decides → NDL describes → LLM narrates"

NDL's core insight: By separating game state decisions from narrative generation, you eliminate the primary source of AI hallucinations. The program makes all decisions; NDL describes what happened; the LLM only provides natural language output.

## Quick Reference

### Basic Syntax

```ndl
do($entity, "action")                # Basic action with entity
do($entity, "action") ~ "manner"     # Action with manner modifier
do($entity, "action") intention="intent"  # Action with intent
action1 -> action2 -> action3        # Sequential actions
wait("condition")                    # Pause/delay
target($entity)                      # Specify target
result("outcome")                    # Declare result
search("object", intention="goal")   # Search action
convey("message", perspective="view") # Communication
system_response("message")           # System messages
```

### Entity Reference Syntax

Entities are referenced with the `$` prefix:
- `$"player"` - Direct entity reference
- `$"guard"` - Named NPC reference
- `$` - Implicit entity (context-dependent)

### Common Patterns

```ndl
# Combat narration
do($"player", "attack") ~ "sword" -> target($"goblin") -> result("hit") -> damage(8)

# Failed action with system response
do($"you", "swing makeshift club against table's surface", intention="test club's combat readiness")
-> result("failure")
-> system_response("something goes wrong when you try to perform this action")
-> describe()

# Search with intention
search("pile of wood", intention="find makeshift club") -> result()

# Social interaction (conversation)
talk("Mark", "John"^"Sue") -> convey("desire to explore dungeon", perspective="adventurous")

# Basic NDL processing example
do($"you", "write some NDL") -> wait("response validation")
```

## Documentation Structure

### 📋 [Specification](specification/)
Formal language definition, grammar, and parsing rules
- [[01-lexical-elements]] - Tokens, keywords, operators
- [[02-grammar]] - Formal BNF grammar
- [[03-semantics]] - Execution model and meaning
- [[04-type-system]] - Parameter types and validation

### 🔧 [Constructs](constructs/)
Individual language elements with examples
- [[do-action]] - The fundamental action construct
- [[manner-modifier]] - How actions are performed
- [[intention]] - Intent and emotional context
- [[sequencing]] - Temporal ordering of actions
- [[wait]] - Timing and pauses
- [[properties]] - Key-value attributes

### 🎯 [Patterns](patterns/)
Common usage patterns and idioms
- [[combat-narration]] - Fighting and action sequences
- [[dialogue-generation]] - Character speech patterns
- [[scene-transitions]] - Moving between scenes
- [[environmental-description]] - Describing settings
- [[character-actions]] - Character behavior patterns

### 🔗 [Integration](integration/)
How NDL connects to the broader system
- [[ndl-to-llm-flow]] - How NDL becomes LLM prompts
- [[game-state-to-ndl]] - Generating NDL from game logic
- [[parsing-implementation]] - Parser design considerations

### 📜 [Evolution](evolution/)
Historical development of the language
- [[version-history]] - How NDL changed over time
- [[design-decisions]] - Rationale behind syntax choices

### 📚 [Appendix](appendix/)
Reference materials
- [[complete-syntax]] - Full syntax reference
- [[glossary]] - Terms and definitions
- [[faq]] - Frequently asked questions

## Key Concepts

### Determinism First
NDL describes outcomes that have already been determined by game logic. It never asks the LLM to make decisions.

### Layered Information
NDL provides multiple layers of context:
1. **What happened** - The action itself (`do(attack)`)
2. **How it happened** - The manner (`~ fiercely`)
3. **Why it happened** - The intention (`intention="intimidate"`)

### Composability
NDL constructs combine cleanly using operators like `->` for sequencing, enabling complex narrative descriptions from simple building blocks.

## Use Cases

- **[[ReallmCraft]]** - Primary implementation in veritasr's Minecraft engine
- **Turn-based games** - Action narration with full context
- **Visual novels** - Scene and dialogue description
- **Procedural storytelling** - Dynamic narrative generation
- **Game state transitions** - Bridging mechanics to narrative

## Related Topics

- [[02-Prompt-Engineering]] - How NDL fits into prompting strategies
- [[04-World-Generation]] - Using NDL for procedural content
- [[User-veritasr]] - Creator and primary contributor

## Status

**Version**: Active Development (as of 2023-2024 discussions)
**Implementations**: [[ReallmCraft]] (Minecraft with Fabric/Quilt)
**Community**: LLM World Engine Discord channel

---

*This documentation synthesized from the LLM World Engine Discord channel discussions, primarily contributions by [[User-veritasr]].*
