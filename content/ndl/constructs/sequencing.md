---
tags: [ndl, construct, operator, sequencing, chaining]
date: 2026-01-17
related:
  - "[[do-action]]"
  - "[[wait]]"
  - "[[00-NDL-INDEX]]"
  - "[[01-lexical-elements]]"
---

# Sequencing Operator (->)

## Summary

The sequencing operator `->` chains actions and events in temporal order. It represents the flow of time and causality in NDL, allowing multiple actions, results, and states to be combined into coherent narrative sequences. The arrow operator is fundamental to NDL's ability to describe complex multi-step events that the LLM then narrates as flowing, connected prose.

## Syntax

```ndl
action1 -> action2 -> action3
statement1 -> statement2
do($"entity", "action") -> result("outcome") -> system_response("consequence")
```

### Characteristics

| Aspect | Description |
|--------|-------------|
| **Precedence** | Lower than `~` (manner), higher than properties |
| **Associativity** | Left-to-right |
| **Type** | Infix binary operator |
| **Purpose** | Temporal/causal sequencing |

## Description

The `->` operator (read as "then" or "followed by") creates sequences of events that happen in order. It's the primary mechanism for describing narratives that unfold over time rather than single isolated actions.

The sequencing operator serves multiple purposes:
1. **Temporal ordering**: Action A happens, then action B
2. **Causal chains**: Action A causes result B, which triggers consequence C
3. **Story flow**: Multiple events form a coherent narrative
4. **State transitions**: Track how situations evolve

> [!important] Design Insight
> The `->` operator transforms NDL from a single-action descriptor into a narrative sequencing language. It's what makes NDL suitable for complex game events rather than just simple commands.

## Parsing Behavior

From veritasr's Director implementation:

```python
Input String: do($"you", "write some NDL")->wait("response validation")
is_ndl: True
continue_last: False
end_scene: True
transition_scene_type: both
Actions: ['do', 'wait']
```

The parser:
- Detects `->` as an NDL marker
- Extracts all actions in the sequence
- Determines scene transitions
- Identifies continuation behavior

## Basic Examples

### Simple Two-Action Sequence

```ndl
do($"you", "write some NDL") -> wait("response validation")
```

**LLM Output**:
```
You write some NDL, then wait for the response to be validated.
```

**Explanation**: The `->` indicates these actions happen in order: writing first, then waiting.

### Real Example: Search with Result

```ndl
search("pile of wood", intention="find makeshift club") -> result()
```

**Actual LLM Output** (Gemma 2 9B IT):
```
Your fingers sift through the rough-hewn logs, searching for something sturdy and manageable. You find it - a thick, gnarled branch, its ends still bearing the marks of the axe. It's not much, but it's better than nothing.
```

**Explanation**: The search action sequences into a result, creating a complete search-and-find narrative.

### Real Example: Action with Failure Chain

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

**Explanation**: The sequence chains action → failure result → system message → description, creating a complete failure narrative.

## Common Patterns

### Action-Result Pattern

```ndl
do($"player", "attack") -> result("hit") -> damage(15)
```

The most common pattern: an action, its outcome, and the consequences.

### Action-Check-Result Pattern

```ndl
do($"player", "sneak") -> roll("stealth") -> result("success")
```

Incorporates dice rolling or skill checks between action and result.

### Multi-Action Sequence

```ndl
do($"player", "draw sword") -> do($"player", "charge") -> do($"player", "attack")
```

Multiple actions in succession, each following the previous.

### Conditional Branching

```ndl
do($"player", "attempt lockpick") -> roll("dexterity") -> result("failure")
-> do($"player", "try again") -> roll("dexterity") -> result("success")
```

Shows multiple attempts with different outcomes.

### Social Interaction Chain

```ndl
talk("Mark", "John"^"Sue") -> convey("desire to explore dungeon", perspective="adventurous")
```

Social actions that sequence conversation and content delivery.

### Scene Transition Sequence

```ndl
do($"party", "travel") -> wait("3 hours") -> arrive($"forest_clearing")
```

Movement across time and space.

## Advanced Usage

### Complex Combat Sequence

```ndl
do($"player", "feint") ~ "convincingly" intention="create opening"
-> wait("enemy reacts")
-> do($"player", "strike") ~ "swiftly" target=$"exposed_flank"
-> roll("attack") -> result("critical_hit") -> damage(24)
```

**LLM Output**:
```
You execute a convincing feint, drawing your opponent's guard out of position. As they react to the false threat, you strike swiftly at their exposed flank. Your blade finds its mark perfectly, dealing a devastating critical wound for 24 damage.
```

### Investigation Chain

```ndl
do($"player", "examine room") ~ "carefully"
-> search($"desk") -> result("find letter")
-> do($"player", "read letter") -> reveal("secret plot")
-> do($"player", "realize implications")
```

**LLM Output**:
```
You carefully examine the room, your attention drawn to the desk. Searching through the drawers, you discover a hidden letter. As you read its contents, a secret plot is revealed. The implications hit you like a thunderbolt - everything makes sense now.
```

### Failed Then Succeeded Attempt

```ndl
do($"player", "pick lock") ~ "lockpicks" intention="open chest"
-> roll("sleight_of_hand") -> result("failure")
-> system_response("The lockpicks slip, you need a different approach")
-> do($"player", "force open") ~ "crowbar"
-> roll("strength") -> result("success")
-> describe("chest opens")
```

**LLM Output**:
```
You work the lockpicks carefully, trying to open the chest, but they slip from the mechanism - this lock is beyond your skill. Frustrated, you grab a crowbar and apply brute force. With a satisfying crack, the chest's lock breaks and the lid swings open.
```

### Dialogue Exchange

```ndl
do($"player", "speak") ~ "politely" text="Excuse me, have you seen a stranger?"
-> wait("NPC response")
-> do($"guard", "reply") ~ "suspiciously" text="Why do you ask?"
-> do($"player", "explain") intention="gain trust"
```

**LLM Output**:
```
"Excuse me, have you seen a stranger?" you ask politely. The guard eyes you with suspicion. "Why do you ask?" he replies. You begin to explain, trying to gain his trust.
```

## Sequencing Rules

### Left-to-Right Execution

```ndl
A -> B -> C
```

Executes as: A, then B, then C (left-to-right, sequential)

### Precedence with Other Operators

```ndl
do($"action") ~ manner -> next_action
```

Parses as: `(do($"action") ~ manner) -> next_action`

The manner modifier binds tighter than sequencing.

### Whitespace Flexibility

```ndl
# All equivalent
action1->action2
action1 -> action2
action1-> action2
action1 ->action2
```

Whitespace around `->` is optional and doesn't affect meaning.

### Chaining Length

There's no theoretical limit to sequence length:

```ndl
a -> b -> c -> d -> e -> f -> g -> ...
```

Practical limits depend on:
- LLM context window
- Narrative coherence
- Scene boundaries

## Semantic Considerations

### Temporal Flow

The `->` operator implies time passing between elements:

```ndl
do($"cast spell") -> wait(5s) -> effect("fireball explodes")
```

Even without explicit `wait()`, sequence implies temporal order.

### Causal Relationships

Sequences often represent causality:

```ndl
do($"knock over candle") -> result("fire starts") -> spread($"curtains")
```

Action causes result, which causes consequence.

### Narrative Coherence

The LLM interprets sequences as connected narrative:

```ndl
do($"enter tavern") -> do($"order drink") -> do($"sit at bar")
```

The LLM generates these as a flowing scene, not disconnected events.

### Scene Boundaries

From veritasr's implementation:

```python
end_scene: True
transition_scene_type: "both"  # time, place, or both
```

Long sequences or certain actions can trigger scene transitions.

## Anti-Patterns

### ❌ Contradictory Sequences

```ndl
do($"player", "die") -> do($"player", "stand up")
```

**Problem**: Actions contradict each other logically.

### ❌ Redundant Sequencing

```ndl
do($"walk") -> do($"walk")
```

**Problem**: Repeating the same action without purpose.

**Better**:
```ndl
do($"walk") ~ "slowly" -> do($"walk") ~ "faster"
```

### ❌ Overly Long Sequences

```ndl
a -> b -> c -> d -> e -> f -> g -> h -> i -> j -> k -> l -> m -> n -> o -> p
```

**Problem**: Too many actions in one sequence becomes confusing.

**Better**: Break into logical chunks or separate scenes.

### ❌ Missing Causal Links

```ndl
do($"swing sword") -> damage(50)
```

**Problem**: Missing the hit/miss check between action and result.

**Better**:
```ndl
do($"swing sword") -> roll("attack") -> result("hit") -> damage(50)
```

## When to Use Sequencing

### ✓ Use `->` When:

1. **Multiple actions occur**: More than one thing happens
2. **Actions have order**: The sequence matters
3. **Cause and effect**: One event leads to another
4. **Time progression**: Events unfold over time
5. **Complex narration**: Story requires multiple beats

### ✗ Single Action Cases:

```ndl
# No need for sequencing
do($"player", "sleep")

# Sequencing adds nothing
do($"player", "sleep") ->
```

## Scene Transitions

Sequences can trigger scene transitions:

### Time Transition

```ndl
do($"party", "rest") -> wait("8 hours") -> new_scene("morning")
```

### Place Transition

```ndl
do($"player", "enter portal") -> travel() -> arrive($"new_location")
```

### Time and Place

```ndl
do($"party", "embark on journey") -> wait("3 days") -> arrive($"distant_city")
```

From veritasr's implementation:
```python
transition_scene_type: "both"  # time AND place changed
```

## Implementation Notes

### Parser Implementation

```python
def parse_sequence(ndl_string):
    """Split NDL into sequence elements."""
    elements = ndl_string.split('->')
    return [elem.strip() for elem in elements]

# Usage
seq = parse_sequence("do(a) -> do(b) -> do(c)")
# Result: ['do(a)', 'do(b)', 'do(c)']
```

### Building Sequences Programmatically

```python
def build_sequence(*actions):
    """Build NDL sequence from multiple actions."""
    return ' -> '.join(actions)

# Usage
seq = build_sequence(
    'do($"player", "attack")',
    'roll("attack")',
    'result("hit")',
    'damage(15)'
)
# Result: 'do($"player", "attack") -> roll("attack") -> result("hit") -> damage(15)'
```

### Sequence Execution

```python
class NDLSequence:
    def __init__(self, ndl_string):
        self.elements = ndl_string.split('->')
        self.index = 0

    def next(self):
        """Get next element in sequence."""
        if self.index < len(self.elements):
            elem = self.elements[self.index].strip()
            self.index += 1
            return elem
        return None

    def has_more(self):
        """Check if sequence has more elements."""
        return self.index < len(self.elements)

# Usage
seq = NDLSequence("do(a) -> do(b) -> do(c)")
while seq.has_more():
    action = seq.next()
    process_action(action)
```

## Relationship to Other Constructs

### With `do()` Actions

```ndl
do($"action1") -> do($"action2")
```

Chain multiple actions in sequence.

### With `wait()` Function

```ndl
action -> wait("1s") -> next_action
```

Introduce time delays between sequence elements.

### With `result()` Function

```ndl
action -> result("outcome")
```

Show the consequence of an action.

### With `roll()` Function

```ndl
action -> roll("check") -> result("success/failure")
```

Incorporate randomness/skill checks in the flow.

### With `system_response()`

```ndl
action -> result("outcome") -> system_response("game message")
```

Add meta-game information to the sequence.

## Design Philosophy

### Narrative Flow Over Commands

NDL isn't a list of discrete commands; it's a narrative flow:

```ndl
# Not: "Do A. Do B. Do C."
# Instead: "A happens, leading to B, resulting in C"
do(A) -> do(B) -> result(C)
```

The `->` operator creates continuity and flow.

### Mimicking Natural Storytelling

Stories have "and then" structures:
- "She entered the room and then saw the body and then screamed"

NDL's `->` mimics this:
```ndl
do($"enter room") -> do($"see body") -> do($"scream")
```

### Backend Control with LLM Narration

The backend determines the sequence of events:
```python
# Backend decides what happens
sequence = [
    'do($"player", "attack")',
    'roll("attack", 15)',
    'result("hit")',
    'damage(8)'
]
ndl = ' -> '.join(sequence)
```

The LLM narrates the predetermined sequence smoothly.

## Best Practices

### 1. Logical Grouping

Group related actions:
```ndl
# Good: Related combat actions
do($"draw weapon") -> do($"charge") -> do($"attack")

# Less good: Unrelated actions
do($"sneeze") -> do($"attack") -> do($"read book")
```

### 2. Clear Causality

Make cause-effect clear:
```ndl
do($"knock over lantern") -> result("fire starts") -> spread($"rapidly")
```

### 3. Appropriate Granularity

```ndl
# Too granular
do($"lift arm") -> do($"move arm forward") -> do($"extend fingers") -> do($"grasp object")

# Better
do($"reach for object") -> do($"grasp object")
```

### 4. Use Waiting for Dramatic Pauses

```ndl
do($"ask question") -> wait("tense silence") -> do($"NPC responds")
```

### 5. End Sequences at Natural Boundaries

```ndl
# Good: Ends at scene conclusion
enter_room -> investigate -> find_clue -> realize_truth

# Less good: Cuts off mid-action
enter_room -> investigate -> find_cl
```

## Evolution History

The `->` operator was part of NDL from its earliest conception as a sequencing language. It's fundamental to NDL's purpose of describing event chains for narration.

### Early Testing (June 2024)

> [!quote] veritasr (June 24, 2024)
> Basic NDL Processing:
> ```
> Input String: do($"you", "write some NDL")->wait("response validation")
> is_ndl: True
> Actions: ['do', 'wait']
> ```

This early example shows `->` already in use for chaining actions.

### Multi-Model Compatibility

> [!quote] veritasr (July 2024)
> "Also spent some time yesterday playing with the NDL, still seems to work effectively across multiple models."

The `->` operator works consistently across different LLMs (Gemma 8B/9B, Llama 3 8B/9B) because it structures the prompt rather than requiring LLM understanding of special syntax.

## Related Constructs

- [[do-action]] - The primary construct being sequenced
- [[wait]] - Introduces time delays in sequences
- [[result]] - Shows outcomes in sequences
- [[system_response]] - Adds meta-information to sequences
- [[roll]] - Adds randomness to sequences
- [[manner-modifier]] - Has higher precedence than `->`
- [[intention]] - Provides context for actions in sequences

## See Also

- [[00-NDL-INDEX]] - Main NDL reference
- [[01-lexical-elements]] - Operator reference
- [[02-grammar]] - Formal grammar for sequencing
- [[ndl-to-llm-flow]] - How sequences are converted to prompts

---

> [!summary] Key Takeaway
> The `->` operator is NDL's narrative glue. It transforms discrete actions into flowing stories, enabling complex multi-step events to be described deterministically by the backend and narrated naturally by the LLM. Without sequencing, NDL would be limited to single-action descriptions. With it, NDL can describe entire scenes, conversations, and event chains.
