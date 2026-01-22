# `intention=` - Intent Property

#ndl #construct #property

[[00-NDL-INDEX|← Back to Index]]

## Overview

The `intention=` property specifies **why** an action is being performed - the underlying purpose or goal behind the action. It provides context that helps the LLM narrate not just what happened, but the motivation behind it.

## Syntax

```ndl
do(action) intention="intent_value"
```

### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `intent_value` | String | Yes | The purpose behind the action |

## Description

While manner (`~`) describes **how** an action is performed, `intention=` describes **why** it's performed. This distinction allows rich, contextual narration that captures both the physical execution and the underlying motivation.

The intention is determined by game logic - the program decides what the character is trying to accomplish, and NDL communicates that to the LLM for natural narration.

## Basic Usage

### Simple Intention
```ndl
do(attack) intention="intimidate"
```

**LLM Output Examples**:
- "The warrior attacks, clearly trying to intimidate his opponent."
- "She swings her weapon in an obviously intimidating display."
- "He strikes with the clear intent to frighten his foe."

### Real Example: Search with Intent

```ndl
search("pile of wood", intention="find makeshift club") -> result()
```

**Actual LLM Output** (Gemma 2 9B IT):
```
Your fingers sift through the rough-hewn logs, searching for something sturdy and manageable. You find it - a thick, gnarled branch, its ends still bearing the marks of the axe. It's not much, but it's better than nothing.
```

**Explanation**: The intention `"find makeshift club"` guides the LLM to narrate the search as looking for a weapon specifically, affecting both the description and what's found.

### Real Example: Failed Action with Intent

```ndl
do($"you", "swing makeshift club against table's surface", intention="test club's combat readiness")
-> result("failure")
-> system_response("something goes wrong when you try to perform this action")
-> describe()
```

**Actual LLM Output** (Gemma 2 9B IT):
```
You raise the gnarled branch, hefting it awkwardly in your hand. It's heavier than you anticipated, the rough bark digging into your palm. You bring it back over your head, aiming for the tabletop. But as you swing, your momentum falters. The branch veers off course, catching on the edge of the chair.

With a sickening crack, the makeshift club splinters, breaking into several pieces. The impact sends a shower of splinters flying, scattering across the floor. You stare down at the useless fragments in your hand, your heart sinking.
```

**Explanation**: The intention `"test club's combat readiness"` provides context for why the character is swinging the club, making the failure narration more meaningful - the test revealed the weapon's poor quality.

### Intention with Manner
```ndl
do(attack) ~ fiercely intention="intimidate"
```

**LLM Output**:
"With fierce, intimidating strikes, the warrior attacks, clearly trying to instill fear in his enemy."

## Common Intentions

### Combat Intentions
```ndl
intention="kill"
intention="intimidate"
intention="disable"
intention="disarm"
intention="wound"
intention="capture"
```

### Social Intentions
```ndl
intention="persuade"
intention="deceive"
intention="impress"
intention="befriend"
intention="insult"
intention="comfort"
```

### Information Intentions
```ndl
intention="gather_information"
intention="distract"
intention="reveal_truth"
intention="hide_evidence"
```

### Strategic Intentions
```ndl
intention="create_opening"
intention="buy_time"
intention="provoke_attack"
intention="retreat_safely"
```

## Advanced Usage

### Complex Social Interaction
```ndl
do(speak) ~ eloquently text="You make an excellent point" intention="gain_favor"
```

**LLM Output**:
"'You make an excellent point,' he says eloquently, clearly trying to curry favor."

### Deceptive Actions
```ndl
do(agree) ~ enthusiastically intention="deceive"
```

**LLM Output**:
"She agrees enthusiastically, though her true intent is to deceive."

### Combat Strategy
```ndl
do(feint) ~ convincingly intention="create_opening" ->
  wait(500ms) ->
  do(attack) ~ swiftly target="exposed_flank"
```

**LLM Output**:
"He executes a convincing feint, deliberately creating an opening. In the brief moment of opportunity, he strikes swiftly at the exposed flank."

## Design Patterns

### Intention vs. Outcome
Intention represents the **goal**, not the **result**:

```ndl
# Intention: What character tries to do
do(persuade) intention="convince" outcome="failure"

# LLM might output:
# "She attempts to convince him, but he remains unmoved."
```

### Layered Motivation
```ndl
# Surface action with hidden intent
do(help) ~ reluctantly intention="gain_trust"
```

**LLM Output**:
"He helps, though reluctantly, seemingly trying to build trust."

### Conflicting Signals
```ndl
# Actions that don't match words
do(speak) ~ warmly text="I'm so happy for you" intention="mask_jealousy"
```

**LLM Output**:
"'I'm so happy for you,' she says warmly, masking what seems to be jealousy."

## Common Intention Categories

### Direct Intentions
Clear, straightforward goals:
```ndl
intention="achieve_goal"
intention="solve_problem"
intention="complete_task"
```

### Hidden Intentions
Concealed motivations:
```ndl
intention="deceive"
intention="manipulate"
intention="distract"
intention="hide_truth"
```

### Emotional Intentions
Feeling-driven goals:
```ndl
intention="express_love"
intention="vent_anger"
intention="show_respect"
intention="humiliate"
```

### Tactical Intentions
Strategic goals:
```ndl
intention="gain_advantage"
intention="gather_intel"
intention="test_defenses"
intention="lure_trap"
```

## Anti-Patterns

### ❌ Redundant Intention
Don't state the obvious:
```ndl
do(kill) intention="kill"  # Redundant
do(kill) intention="send_message"  # Better - adds context
```

### ❌ Conflicting Information
Ensure consistency:
```ndl
# Questionable
do(help) intention="harm"  # How does helping harm?

# Better
do(help) intention="gain_trust" -> do(betray)
```

### ❌ Outcome as Intention
Intention is the goal, not the result:
```ndl
do(attack) intention="success"  # Wrong - not an intent
do(attack) intention="kill" outcome="success"  # Better
```

## Intention vs. Other Modifiers

### Intention vs. Manner
```ndl
# How: Physical execution
do(speak) ~ loudly

# Why: Underlying purpose
do(speak) intention="intimidate"

# Combined: Full context
do(speak) ~ loudly intention="intimidate"
```

### Intention vs. Emotion
```ndl
# Current feeling
do(speak) emotion="anger"

# Desired outcome
do(speak) intention="intimidate"

# Both
do(speak) emotion="anger" intention="vent_frustration"
```

### Intention vs. Target
```ndl
# Who/what
do(speak) target="guard"

# Why
do(speak) intention="distract"

# Both
do(speak) target="guard" intention="distract"
```

## Examples by Genre

### Fantasy RPG
```ndl
do(cast_spell) ~ dramatically spell="Fireball" intention="destroy_barrier"
```

### Mystery/Detective
```ndl
do(question) ~ casually target="suspect" intention="catch_in_lie"
```

### Political Intrigue
```ndl
do(propose_alliance) ~ formally intention="weaken_rival"
```

### Heist/Stealth
```ndl
do(create_distraction) ~ convincingly intention="allow_infiltration"
```

## Implementation

### Generating Intentions
```python
def add_intention(action_ndl: str, intention: str) -> str:
    """Add intention property to action."""
    return f'{action_ndl} intention="{intention}"'

# Usage
base = "do(attack) ~ fiercely"
with_intent = add_intention(base, "intimidate")
# Result: do(attack) ~ fiercely intention="intimidate"
```

### Intention from Game State
```python
def determine_intention(character, action, context) -> str:
    """Derive intention from game state."""
    if context.in_combat and character.health < 0.3:
        return "survive"
    elif action == "attack" and character.wants_to_intimidate:
        return "intimidate"
    elif action == "help" and character.has_hidden_agenda:
        return "gain_trust"
    else:
        return "accomplish_goal"

# Usage
intention = determine_intention(player, "attack", combat_state)
ndl = f"do(attack) ~ fiercely intention=\"{intention}\""
```

### Parsing Intentions
```python
import re

def parse_intention(ndl: str) -> str | None:
    """Extract intention from NDL string."""
    match = re.search(r'intention="([^"]*)"', ndl)
    return match.group(1) if match else None

# Usage
intent = parse_intention('do(attack) intention="intimidate"')
# Result: "intimidate"
```

## LLM Integration

### In System Prompt
```
When you see intention="value", incorporate the character's
underlying motivation into your narration. For example:

- intention="intimidate" → show the character trying to frighten
- intention="deceive" → suggest dishonest motives
- intention="help" → convey genuine assistance

The intention represents what the character is TRYING to do,
which may or may not succeed.
```

### In Few-Shot Examples
```
NDL: do(help) ~ eagerly intention="gain_trust"
Output: She helps eagerly, seemingly trying to build trust.

NDL: do(attack) ~ viciously intention="kill"
Output: He attacks with vicious, lethal intent.

NDL: do(speak) ~ calmly text="Everything is fine" intention="hide_panic"
Output: "Everything is fine," he says calmly, trying to hide his inner panic.
```

## Psychology of Intention

### Observable Intent
Some intentions are visible:
```ndl
do(threaten) ~ menacingly intention="intimidate"
# Clear to observer: trying to intimidate
```

### Hidden Intent
Others are concealed:
```ndl
do(smile) ~ warmly intention="deceive"
# Not obvious to observer: deception hidden
```

### Mixed Signals
Actions can reveal or conceal:
```ndl
do(help) ~ reluctantly intention="appear_cooperative"
# Reluctance reveals lack of genuine desire to help
```

## TTRPG Inspiration

> [!quote] veritasr on TTRPG Structure (June 24, 2024)
> "NDL represents the pieces as:
> - what (action())
> - how (~ (basically with / by))
> - why(intention='')"

This what/how/why structure mirrors how GMs teach new TTRPG players to describe their actions. The intention property completes the triad, giving the LLM complete context.

> [!quote] veritasr on Social Dynamics Expansion (June 23, 2024)
> "On another note, updated the NDL to encompass conversation content, opinions / perspectives, and describe how something is accomplished. Should capture the content of conversations now in a way that allows describing intent so that reasoning can be applied and characters can react with different viewpoints."

This expansion made intention even more critical for the 43+ social actions in NDL.

## Design Philosophy

### Game Decides Intent
The game logic determines what the character is trying to accomplish:

```python
# Game logic
if enemy.intimidate_check(player):
    intention = "intimidate"
else:
    intention = "kill"

ndl = f"do(attack) intention=\"{intention}\""
```

### LLM Narrates Intent
The LLM translates the intent into natural language:
- Not: "The character's intention is to intimidate"
- Better: "Clearly trying to frighten his opponent..."

### Intent vs. Success
Intention doesn't guarantee success:
```ndl
do(persuade) intention="convince" outcome="failure"
# Tried to convince, but failed
```

## Related Constructs

- [[do-action]] - The action with intention
- [[manner-modifier]] - How vs. why
- [[properties]] - Other contextual properties

## Related Documentation

- [[03-semantics]] - Meaning of intentions
- [[04-type-system]] - Intention value types
- [[game-state-to-ndl]] - Determining intentions from state

---

*From [[User-veritasr]]'s NDL design in the LLM World Engine Discord.*
