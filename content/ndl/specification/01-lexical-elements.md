---
tags: [ndl, specification, lexical, tokens]
date: 2026-01-16
source: Discord - LLM World Engine Channel
status: complete
---

# NDL Lexical Elements

## Overview

This document defines the lexical elements (tokens, keywords, operators, and literals) that make up the NDL (Natural Description Language) syntax.

## Keywords

NDL keywords are reserved action identifiers that have special meaning in the language.

### Core Action Keywords

| Keyword | Category | Description | Example |
|---------|----------|-------------|---------|
| `do` | Action | Primary action constructor | `do($"player", "attack")` |
| `wait` | Control | Pause or wait for condition | `wait("response validation")` |
| `search` | Action | Search for objects | `search("pile of wood")` |
| `talk` | Social | Initiate conversation | `talk("Mark", "John")` |
| `convey` | Social | Communicate message/intent | `convey("desire to explore")` |
| `target` | Modifier | Specify action target | `target($"goblin")` |
| `result` | Outcome | Declare action outcome | `result("success")` |
| `describe` | Output | Trigger narrative description | `describe()` |
| `system_response` | Meta | System-level messages | `system_response("error")` |
| `roll` | Mechanics | Dice/check result | `roll("stealth")` |
| `damage` | Mechanics | Damage value | `damage(8)` |

### Properties/Attributes

These are key-value pair identifiers used to add context:

| Property | Description | Example |
|----------|-------------|---------|
| `intention` | Why an action is taken | `intention="test club's combat readiness"` |
| `perspective` | Point of view or opinion | `perspective="adventurous"` |

### Social Dynamics Keywords

NDL has **43+ action types** dedicated to social interactions (as of June 2024):
- Conversation initiation
- Emotional expression
- Opinion sharing
- Relationship management
- Persuasion attempts
- Conflict resolution

*Note: Full list of social action keywords not fully documented in transcripts, but system tracks 23 exposed states for social interaction.*

## Operators

### Primary Operators

| Operator | Name | Precedence | Associativity | Description |
|----------|------|------------|---------------|-------------|
| `$` | Entity Reference | 1 | Prefix | References game entities |
| `~` | Manner Modifier | 2 | Infix | Describes how action is performed |
| `=` | Assignment | 3 | Infix | Assigns property values |
| `->` | Sequence | 4 | Left | Chains actions sequentially |
| `^` | Conjunction | 5 | Infix | Connects multiple targets (AND) |

### Operator Details

#### `$` - Entity Reference Operator

**Purpose**: Identifies game entities (characters, objects, locations)

**Syntax**:
```ndl
$"entity_name"    # Named entity
$                 # Implicit entity (context-dependent)
```

**Examples**:
```ndl
do($"player", "attack")     # Player entity
target($"guard")            # Guard entity
$"you"                      # Second-person reference
```

#### `~` - Manner Modifier Operator

**Purpose**: Specifies how an action is performed (method, tool, style)

**Syntax**:
```ndl
action ~ "manner_description"
```

**Examples**:
```ndl
do($"player", "attack") ~ "sword"           # Attack with sword
do($"player", "sneak") ~ "carefully"        # Sneak carefully
do($"player", "speak") ~ "menacingly"       # Speak menacingly
```

**Quote from veritasr** [04:27]:
> "~ (basically with / by)"

#### `=` - Assignment Operator

**Purpose**: Assigns values to properties/attributes

**Syntax**:
```ndl
property="value"
```

**Examples**:
```ndl
intention="intimidate"
perspective="adventurous"
```

#### `->` - Sequence Operator

**Purpose**: Chains actions in temporal order (left-to-right execution)

**Syntax**:
```ndl
action1 -> action2 -> action3
```

**Examples**:
```ndl
do($"player", "attack") -> roll("hit") -> result("success")
search("wood") -> result() -> describe()
```

**Semantics**: Each action completes before the next begins. Order is preserved.

#### `^` - Conjunction Operator

**Purpose**: Connects multiple entities or targets (AND relationship)

**Syntax**:
```ndl
entity1 ^ entity2 ^ entity3
```

**Examples**:
```ndl
talk("Mark", "John"^"Sue")    # Mark talks to both John and Sue
```

## Literals

### String Literals

Strings are enclosed in double quotes:

```ndl
"this is a string"
"attack with great force"
"find makeshift club"
```

**Usage**:
- Action descriptions
- Entity names
- Property values
- Messages and dialogue

**Escaping**: Not explicitly documented, but standard escaping likely applies.

### Numeric Literals

Numbers appear in function-like constructs:

```ndl
damage(8)         # Integer
wait(1s)          # Duration (integer + unit)
```

**Types**:
- **Integer**: Whole numbers (e.g., `8`, `100`)
- **Duration**: Number + time unit (e.g., `1s`, `500ms`)

### Entity Literals

Entities use the `$` prefix with string notation:

```ndl
$"player"
$"guard"
$"goblin"
$"you"
```

## Delimiters

| Delimiter | Purpose | Example |
|-----------|---------|---------|
| `(` `)` | Function/action parameters | `do($"player", "attack")` |
| `"` | String boundaries | `"attack fiercely"` |
| `,` | Parameter separator | `do($entity, "action")` |

## Whitespace

Whitespace (spaces, tabs, newlines) is generally ignored except:
- Within string literals (preserved)
- As token separators

## Comments

Comments are not explicitly documented in the transcripts. Implementation may or may not support comments.

## Identifiers

Identifiers follow general programming conventions:
- Function names: `do`, `wait`, `search`, `talk`, etc.
- Property names: `intention`, `perspective`, `result`
- Entity names: Strings within `$"..."` notation

## Detection Markers

The NDL parser uses these markers to detect NDL syntax:

```python
ndl_markers = ['do(', '->', 'wait(', '~', 'intention=', '$']
```

**Source**: veritasr's implementation (transcript line 31881-31913)

If any of these markers appear in input, it's likely NDL.

## Parse Flags

The parser extracts these flags from NDL input:

| Flag | Type | Description |
|------|------|-------------|
| `is_ndl` | boolean | Is input valid NDL? |
| `continue_last` | boolean | Continue previous scene? |
| `end_scene` | boolean | End current scene? |
| `transition_scene_type` | string | Transition type: "time", "place", "both" |
| `actions` | array | List of action keywords found |

**Example Output**:
```python
Input String:  do($"you", "write some NDL")->wait("response validation")
is_ndl:  True
continue_last:  False
end_scene:  True
transition_scene_type:  both
Actions:  ['do', 'wait']
```

## Reserved Patterns

### TTRPG-Inspired Structure

veritasr [04:24]: "There's also a sort of recipe on the input, which is the same one you use as a GM for people who don't know how to play TTRPGs:
- **what** (do you do) → `do(action)`
- **how** (do you do it) → `~ manner`
- **why** (are you doing it) → `intention="reason"`"

This three-part structure is fundamental to NDL's design philosophy.

## Lexical Conventions

1. **Case Sensitivity**: Keywords appear to be case-sensitive (lowercase)
2. **Entity References**: Always use `$` prefix
3. **String Quoting**: Always use double quotes `"`
4. **Operator Spacing**: Whitespace around operators is flexible
5. **Chaining**: No limit on `->` chain length

## Implementation Notes

### Multi-Model Compatibility

veritasr [04:21]: "And this is all on 8B / 9B LLMs. Nothing that really breaks the bank hardware wise."

veritasr [18:02]: "NDL works effectively across multiple models. Get's tricky when there's back messages, since it sometimes tries to select previous messages, but I'll be doing it in a multi-step workflow, so that won't be a problem."

**Models Tested**:
- Gemma (8B/9B)
- Llama 3 (8B/9B)

### Dynamic Vocabulary Growth

veritasr [19:47]: "NDL currently has 43 actions tied to social and is still growing. and 23 different exposed states just tied to social interaction."

NDL vocabulary continues to expand as new game mechanics are added.

## Related Documentation

- [[02-grammar|Grammar Specification]]
- [[03-semantics|Semantic Rules]]
- [[do-action|do() Construct]]
- [[manner-modifier|~ Operator]]

---

#ndl #specification #lexical #syntax
