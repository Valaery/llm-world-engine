# `~` - Manner Modifier

#ndl #construct #modifier

[[00-NDL-INDEX|← Back to Index]]

## Overview

The manner modifier (`~`) specifies **how** an action is performed. It adds qualitative detail to actions, constraining the LLM to narrate the action in a specific way.

## Syntax

```ndl
do(action) ~ manner
```

### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `manner` | Identifier | Yes | Adverb or adverbial phrase |

## Description

The `~` operator modifies the immediately preceding action, telling the LLM that the action was performed in a particular manner. It represents a **constraint** on the narration - the LLM must incorporate this manner into the output.

Think of `~` as meaning "in the manner of" or "approximately like" (hence the similarity to the mathematical approximation symbol ≈).

## Basic Usage

### Simple Manner
```ndl
do(walk) ~ slowly
```

**LLM Output Examples**:
- "The character walks slowly."
- "She moves at a deliberate, unhurried pace."
- "He trudges forward slowly."

### Real Example: Physical Action with Tool

```ndl
search("pile of wood", intention="find makeshift club") ~ "with your hands"
```

**Actual LLM Output** (Gemma 2 9B IT):
```
Your fingers sift through the rough-hewn logs, searching for something sturdy and manageable. You find it - a thick, gnarled branch, its ends still bearing the marks of the axe. It's not much, but it's better than nothing.
```

This example from veritasr's testing shows how the manner `~ "with your hands"` tells the LLM this is a manual search, resulting in tactile, physical narration.

### Real Example: Failed Action

```ndl
do($"you", "swing makeshift club against table's surface") ~ "with club"
```

**Actual LLM Output** (Gemma 2 9B IT):
```
You raise the gnarled branch, hefting it awkwardly in your hand. It's heavier than you anticipated, the rough bark digging into your palm. You bring it back over your head, aiming for the tabletop. But as you swing, your momentum falters. The branch veers off course, catching on the edge of the chair.
```

Even in failure scenarios, the manner modifier `~ "with club"` provides the tool context that enables realistic failure narration.

### Multiple Word Manner
```ndl
do(speak) ~ with_confidence
```

**LLM Output Examples**:
- "She speaks with confidence."
- "He delivers his words confidently."
- "The character speaks in a self-assured tone."

## Common Manner Modifiers

### Speed/Pace
```ndl
~ slowly
~ quickly
~ rapidly
~ hastily
~ leisurely
```

### Emotion/Attitude
```ndl
~ angrily
~ happily
~ sadly
~ nervously
~ confidently
~ reluctantly
```

### Physical Quality
```ndl
~ carefully
~ clumsily
~ gracefully
~ forcefully
~ gently
~ roughly
```

### Social Quality
```ndl
~ politely
~ rudely
~ formally
~ casually
~ warmly
~ coldly
~ whisper
~ sarcastically
~ enthusiastically
```

### Real Example: Social Interaction
```ndl
talk("Mark", "John"^"Sue") -> convey("desire to explore dungeon", perspective="adventurous")
```

This example from veritasr shows how manner can apply to social interactions, specifying tone, volume, or style of communication.

### Intensity
```ndl
~ fiercely
~ mildly
~ intensely
~ half_heartedly
~ wholeheartedly
```

### Stealth/Visibility
```ndl
~ quietly
~ loudly
~ stealthily
~ openly
~ subtly
```

## Advanced Usage

### Combined with Properties
```ndl
do(attack) ~ fiercely damage=15 weapon="sword"
```

**LLM Output**:
"The warrior swings his sword fiercely, dealing 15 points of damage."

### Multiple Modifiers
Note: Only one manner per action (you can't do something both slowly AND quickly):
```ndl
do(speak) ~ nervously text="I'm sorry" emotion="fear"
```

**LLM Output**:
"With clear nervousness and fear in her voice, she stammers, 'I'm sorry.'"

### Manner with Intention
```ndl
do(persuade) ~ eloquently intention="convince"
```

**LLM Output**:
"He speaks eloquently, clearly attempting to convince his audience."

## Design Patterns

### Action Characterization
Use manner to add personality:
```ndl
# Timid character
do(enter) ~ hesitantly

# Bold character
do(enter) ~ confidently

# Sneaky character
do(enter) ~ stealthily
```

### Combat Dynamics
```ndl
do(attack) ~ desperately    # Losing fight
do(attack) ~ confidently    # Winning fight
do(defend) ~ frantically    # Overwhelmed
```

### Social Nuance
```ndl
do(agree) ~ reluctantly     # Forced agreement
do(agree) ~ enthusiastically # Genuine agreement
```

### Emotional Arc
```ndl
do(speak) ~ nervously ->
  wait(1s) ->
  do(speak) ~ more_confidently ->
  wait(1s) ->
  do(speak) ~ boldly
```

## Anti-Patterns

### ❌ Contradictory Manner
```ndl
do(walk) ~ quickly ~ slowly  # Can't be both!
```

### ❌ Manner Without Action
```ndl
~ slowly  # What is being done slowly?
```

### ❌ Inappropriate Manner
Be semantically sensible:
```ndl
do(die) ~ happily  # Unusual but technically valid
do(sleep) ~ loudly # Semantically questionable
```

### ❌ Over-Specification
Don't duplicate information:
```ndl
# Redundant
do(whisper) ~ quietly  # Whispering is already quiet

# Better
do(whisper)  # Let LLM interpret appropriately
```

## Semantic Considerations

### Manner vs. Properties
Use manner for **how**, properties for **what**:

```ndl
# Manner: HOW it's done
do(speak) ~ nervously

# Property: WHAT is said
do(speak) text="Hello"

# Combined
do(speak) ~ nervously text="Hello"
```

### Manner vs. Intention
```ndl
# Manner: HOW you do it
do(threaten) ~ menacingly

# Intention: WHY you do it
do(threaten) intention="intimidate"

# Both provide context
do(threaten) ~ menacingly intention="intimidate"
```

## Symbol Choice Rationale

### Why `~`?
From [[User-veritasr]]'s design discussions:

1. **Visual similarity** to approximation symbol (≈)
2. **Short** - single character
3. **Rare** in natural text - unlikely to conflict
4. **Distinct** from other operators
5. **Suggests** "in the manner of" or "approximately like"

### Alternatives Considered
- `@` - Too overloaded in programming
- `:` - Common in many syntaxes
- `with` - Too verbose
- `in` - English word, could conflict

## Examples by Context

### Combat
```ndl
do(attack) ~ ferociously
do(dodge) ~ acrobatically
do(block) ~ desperately
do(strike) ~ precisely
```

### Dialogue
```ndl
do(speak) ~ sarcastically
do(ask) ~ innocently
do(respond) ~ defensively
do(interrupt) ~ rudely
```

### Movement
```ndl
do(run) ~ frantically
do(walk) ~ purposefully
do(sneak) ~ carefully
do(stride) ~ confidently
```

### Magic/Special Abilities
```ndl
do(cast_spell) ~ dramatically
do(channel_energy) ~ intensely
do(summon) ~ ceremoniously
```

## Implementation

### Generating Manner Modifiers
```python
def add_manner(action_ndl: str, manner: str) -> str:
    """Add manner modifier to action."""
    return f"{action_ndl} ~ {manner}"

# Usage
base = "do(walk)"
with_manner = add_manner(base, "slowly")
# Result: "do(walk) ~ slowly"
```

### Parsing Manner Modifiers
```python
import re

def parse_manner(ndl: str) -> str | None:
    """Extract manner from NDL string."""
    match = re.search(r'~ (\w+)', ndl)
    return match.group(1) if match else None

# Usage
manner = parse_manner("do(walk) ~ slowly")
# Result: "slowly"
```

### Multi-Word Manner
```python
def format_manner(words: list[str]) -> str:
    """Format multi-word manner with underscores."""
    return "_".join(words).lower()

# Usage
manner = format_manner(["with", "determination"])
# Result: "with_determination"

ndl = f"do(fight) ~ {manner}"
# Result: "do(fight) ~ with_determination"
```

## LLM Integration

### In System Prompt
```
When you see "do(action) ~ manner", narrate the action being
performed in the specified manner. For example:
- "do(walk) ~ slowly" → "The character walks slowly"
- "do(attack) ~ fiercely" → "He attacks with fierce determination"

The manner MUST be reflected in your narration.
```

### In Few-Shot Examples
```
NDL: do(speak) ~ nervously text="I didn't do it"
Output: "I didn't do it," she says nervously, voice trembling.

NDL: do(charge) ~ recklessly target="enemy"
Output: He charges recklessly toward the enemy, throwing caution aside.
```

## Related Constructs

- [[do-action]] - The action being modified
- [[intention]] - Why action is performed (vs. how)
- [[properties]] - Additional attributes

## Related Documentation

- [[01-lexical-elements]] - Manner as a token type
- [[02-grammar]] - Manner in the grammar
- [[03-semantics]] - Meaning of manner modifiers

---

*From [[User-veritasr]]'s NDL design in the LLM World Engine Discord.*
