---
tags: [ndl, construct, action, do]
date: 2026-01-16
source: Discord - LLM World Engine Channel
status: complete
---

# do() - Action Construct

[[00-NDL-INDEX|← Back to Index]]

## Overview

The `do()` construct is the fundamental building block of NDL. It describes an action performed by an entity in the game world.

## Syntax

```ndl
do($entity, "action_description")
```

### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `entity` | Entity Reference | Yes | The entity performing the action (with `$` prefix) |
| `action_description` | String | Yes | Natural language description of the action |

## Entity Reference Syntax

Entities are referenced with the `$` prefix:
- `$"player"` - Named player entity
- `$"you"` - Second-person reference
- `$"guard"` - Named NPC
- `$"goblin"` - Named creature
- `$` - Implicit entity (context-dependent)

## Examples

### Basic Usage

```ndl
do($"player", "attack")
```

**LLM Output**: "You attack with determination."

### Detailed Action Description

```ndl
do($"you", "swing makeshift club against table's surface")
```

**LLM Output**: "You raise the gnarled branch, hefting it awkwardly in your hand. You bring it back over your head, aiming for the tabletop. But as you swing, your momentum falters. The branch veers off course, catching on the edge of the chair."

### With Intention Property

```ndl
do($"you", "swing makeshift club against table's surface", intention="test club's combat readiness")
```

**LLM Output**: "You raise the makeshift club, testing its weight and balance. You bring it down against the table's surface to gauge its durability."

## Combined with Modifiers

### With Manner Modifier (~)

```ndl
do($"player", "walk") ~ "carefully"
```

**LLM Output**: "You carefully make your way forward, watching each step."

```ndl
do($"player", "attack") ~ "sword"
```

**LLM Output**: "You swing your sword in a wide arc."

### With Properties

```ndl
do($"player", "speak") intention="intimidate"
```

**LLM Output**: "You speak with a menacing tone, your words calculated to strike fear."

## Chaining Actions

```ndl
do($"player", "attack") -> do($"player", "follow up")
```

**LLM Output**: "You attack fiercely, then immediately follow up with another strike."

## Real-World Examples from Screenshots

### Example 1: Search Action
From the Gemma 2 test screenshot:

**NDL**:
```ndl
search("pile of wood", intention="find makeshift club") -> result()
```

**LLM Output**: "Your fingers sift through the rough-hewn logs, searching for something sturdy and manageable. You find it - a thick, gnarled branch, its ends still bearing the marks of the axe. It's not much, but it's better than nothing."

### Example 2: Failed Action with System Response

**NDL**:
```ndl
do($"you", "swing makeshift club against table's surface", intention="test club's combat readiness")
-> result("failure")
-> system_response("something goes wrong when you try to perform this action")
-> describe()
```

**LLM Output**: "You raise the gnarled branch, hefting it awkwardly in your hand. It's heavier than you anticipated, the rough bark digging into your palm. You bring it back over your head, aiming for the tabletop. But as you swing, your momentum falters. The branch veers off course, catching on the edge of the chair.

With a sickening crack, the makeshift club splinters, breaking into several pieces. The impact sends a shower of splinters flying, scattering across the floor. You stare down at the useless fragments in your hand, your heart sinking."

## Design Philosophy

### What, How, Why Structure

From veritasr [04:24]:
> "There's also a sort of recipe on the input, which is the same one you use as a GM for people who don't know how to play TTRPGs:
> - **what** (do you do) → `do(action)`
> - **how** (do you do it) → `~ manner`
> - **why** (are you doing it) → `intention="reason"`"

The `do()` construct answers the "what" question.

### LLM as Narrator

From veritasr [04:09]:
> "Backend processes input and runs logic deciding the result. Then it dynamically generates the NDL prompt, passing in the action chain to the LLM for processing. The end result is what you see here. Basically narrative text."

The `do()` construct tells the LLM **what happened**, not what **should happen**. The game logic has already made all decisions.

## Common Patterns

### Combat Actions

```ndl
do($"player", "attack") ~ "sword" -> target($"goblin") -> roll("hit") -> result("success")
```

### Movement Actions

```ndl
do($"player", "move") ~ "stealthily" -> target($"door")
```

### Social Actions

```ndl
do($"player", "persuade") ~ "charming words" -> target($"merchant") -> intention="get discount"
```

### Search/Investigation

```ndl
do($"player", "search") ~ "carefully" -> target($"room")
```

## Anti-Patterns

### Don't Ask LLM to Decide

```ndl
❌ do($"player", "try to attack")
✓ do($"player", "attack") -> result("success")
✓ do($"player", "attack") -> result("failure")
```

The outcome should be determined by game logic before NDL generation.

### Don't Use Vague Descriptions

```ndl
❌ do($"player", "do something")
✓ do($"player", "swing sword at enemy")
```

Be specific about the action.

### Don't Omit Entity Reference

```ndl
❌ do("attack")
✓ do($"player", "attack")
```

Always specify who is performing the action.

## Technical Details

### Action Categories

Based on veritasr's implementation, NDL supports 100+ action types across categories:
- **Physical**: 36 actions (movement, combat, manipulation)
- **Mental**: 35 actions (thinking, planning, problem-solving)
- **Social**: 43+ actions (conversation, persuasion, emotion)

### Processing Example

From veritasr's Director implementation:

```python
Input String:  do($"you", "write some NDL")->wait("response validation")
is_ndl:  True
continue_last:  False
end_scene:  True
transition_scene_type:  both
Actions:  ['do', 'wait']
```

The parser extracts:
- Action type: `do`
- Entity: `$"you"`
- Description: `"write some NDL"`

## Integration with Game Logic

### Flow

```
Game State Change
    ↓
Generate NDL: do($entity, "action")
    ↓
Build Dynamic Prompt
    ↓
Send to LLM
    ↓
Receive Narrative Text
    ↓
Display to Player
```

### Example

```
Player Input: "I attack the goblin with my sword"
    ↓
Parser: Extract intent, entity, target
    ↓
Game Logic: Roll attack, calculate hit/miss/damage
    ↓
Result: Hit for 8 damage
    ↓
NDL Generator: do($"player", "attack") ~ "sword" -> target($"goblin") -> result("hit") -> damage(8)
    ↓
Prompt Builder: Add context + NDL
    ↓
LLM: "You swing your sword in a powerful arc, the blade catching the goblin in its shoulder. It shrieks in pain as blood flows from the wound."
```

## Model Compatibility

From veritasr [04:21]:
> "And this is all on 8B / 9B LLMs. Nothing that really breaks the bank hardware wise."

The `do()` construct works reliably on small models:
- Gemma 8B/9B
- Llama 3 8B/9B

No need for GPT-4 or large models.

## Related Constructs

- [[manner-modifier|~ - Manner Modifier]] - Specifies how action is performed
- [[intention|intention= - Intent]] - Specifies why action is performed
- [[target|target() - Target]] - Specifies target of action
- [[result|result() - Result]] - Declares action outcome
- [[sequencing|-> - Sequencing]] - Chains actions together

## Related Documentation

- [[01-lexical-elements|Lexical Elements]]
- [[02-grammar|Grammar Specification]]
- [[combat-narration|Combat Narration Pattern]]

---

#ndl #construct #action #do
