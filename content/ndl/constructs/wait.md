---
tags: [ndl, construct, function, timing, pause, control-flow]
date: 2026-01-17
related:
  - "[[sequencing]]"
  - "[[do-action]]"
  - "[[00-NDL-INDEX]]"
  - "[[01-lexical-elements]]"
---

# wait() - Wait/Pause Function

## Summary

The `wait()` function introduces temporal pauses, delays, or waiting conditions in NDL sequences. It represents time passing, dramatic pauses, reaction delays, or conditions that must be met before the narrative continues. This construct enables NDL to describe not just what happens, but the timing and pacing of events.

## Syntax

```ndl
wait("duration")
wait("condition")
wait(time_value)
```

### Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| duration/condition | String or number | Time duration (e.g., "1s", "3 hours") or condition (e.g., "response", "enemy reacts") |

## Description

The `wait()` function serves multiple purposes in NDL:

1. **Time Delays**: Explicit duration (seconds, minutes, hours)
2. **Dramatic Pauses**: Narrative timing ("tense moment", "brief silence")
3. **Reaction Delays**: Waiting for NPC/world response
4. **Conditional Holds**: Waiting until something happens
5. **Pacing Control**: Slowing down or speeding up narrative flow

Unlike `do()` which represents actions, `wait()` represents the absence of action or the passage of time between actions.

> [!important] Narrative Timing
> The `wait()` function is crucial for pacing. It tells the LLM that time passes or there's a pause, preventing all actions from feeling rushed or simultaneous.

## Real Examples

### Basic Wait in Sequence

```ndl
do($"you", "write some NDL") -> wait("response validation")
```

**From veritasr's Director processing**:
```
Input String: do($"you", "write some NDL")->wait("response validation")
is_ndl: True
Actions: ['do', 'wait']
```

**Interpretation**: The character writes NDL, then waits for the response to be validated. This creates a two-beat narrative: action, then pause/anticipation.

## Common Wait Types

### Time Durations

```ndl
wait("1s")        # 1 second
wait("5m")        # 5 minutes
wait("3 hours")   # 3 hours
wait("overnight") # Abstract duration
wait("brief moment")
wait("long pause")
```

### Response/Reaction Waits

```ndl
wait("enemy reacts")
wait("NPC responds")
wait("response validation")
wait("answer")
wait("their move")
```

### Conditional Waits

```ndl
wait("door opens")
wait("reinforcements arrive")
wait("spell completes")
wait("sunrise")
wait("trap resets")
```

### Dramatic Pauses

```ndl
wait("tense silence")
wait("dramatic pause")
wait("moment of anticipation")
wait("breath held")
wait("heartbeat")
```

## Usage Patterns

### Action-Wait-Action

```ndl
do($"player", "ask question") -> wait("tense silence") -> do($"NPC", "respond")
```

**LLM Output**:
```
You ask the question. Silence hangs heavy in the air. Finally, after what feels like an eternity, the stranger responds.
```

The wait creates tension between question and answer.

### Action-Wait-Result

```ndl
do($"player", "attack") -> wait("enemy reacts") -> result("enemy dodges")
```

**LLM Output**:
```
You swing your blade. For a split second, time seems to slow. Then your opponent moves, side-stepping your attack with practiced ease.
```

### Time-Based Scene Transition

```ndl
do($"party", "rest at camp") -> wait("8 hours") -> new_scene("dawn")
```

**LLM Output**:
```
You make camp for the night, taking turns keeping watch. Eight hours pass. As the first light of dawn breaks over the horizon, you prepare to continue your journey.
```

### Combat Timing

```ndl
do($"player", "feint") -> wait("enemy commits") -> do($"player", "strike true target")
```

**LLM Output**:
```
You feint toward the goblin's left side. It takes the bait, shifting its defense. In that instant of commitment, you strike at its undefended right flank.
```

### Multiple Waits

```ndl
do($"cast spell") -> wait("1s") ~ "channeling energy" -> wait("2s") ~ "energy builds" -> wait("1s") ~ "final surge" -> result("spell releases")
```

**LLM Output**:
```
You begin casting, channeling arcane energy. The power builds second by second, swirling around you. The energy reaches a crescendo, and in a final surge, the spell releases in a brilliant flash.
```

## Semantic Interpretations

### Duration: Explicit Time

```ndl
wait("3 hours")
```

**LLM Interpretation**: Three hours pass. May summarize events, skip details, or transition scene.

### Condition: Event Occurs

```ndl
wait("door opens")
```

**LLM Interpretation**: Time passes until door opens. May describe anticipation, then the opening.

### Dramatic: Narrative Beat

```ndl
wait("heartbeat")
```

**LLM Interpretation**: Very brief pause for dramatic effect. Doesn't necessarily mean literal heartbeat duration.

### Response: Reactive Pause

```ndl
wait("NPC response")
```

**LLM Interpretation**: Pause while NPC formulates and delivers response. May include NPC deliberation.

## Integration with Other Constructs

### With Actions (Sequencing)

```ndl
do($"action1") -> wait("delay") -> do($"action2")
```

Most common pattern: action, pause, action.

### With Results

```ndl
do($"attack") -> wait("enemy reacts") -> result("miss")
```

Pause before outcome is revealed.

### With Manner

```ndl
do($"speak") -> wait("pause") ~ "uncomfortable" -> do($"continue")
```

The manner can modify the wait itself (uncomfortable pause, tense silence, etc.).

### With System Response

```ndl
do($"action") -> result("failure") -> wait("recover") -> system_response("You can try again")
```

Pause before meta-information.

## Scene Transitions

Long waits often trigger scene transitions:

### Time Skip

```ndl
wait("overnight") -> new_scene("next morning")
wait("several days") -> arrive($"destination")
```

### Travel Time

```ndl
do($"depart city") -> wait("3 days travel") -> arrive($"fortress")
```

**LLM Output**:
```
You leave the city gates behind. Three days of travel through rolling hills and dense forests bring you to the fortress, its dark towers looming against the sky.
```

### Healing/Recovery

```ndl
do($"rest") -> wait("full night") -> restore("health") -> wake($"refreshed")
```

## Implementation Notes

### Parsing Wait Functions

```python
import re

def parse_wait(ndl_string):
    """Extract wait value from NDL."""
    match = re.search(r'wait\("([^"]+)"\)', ndl_string)
    if match:
        return match.group(1)
    return None

# Usage
condition = parse_wait('do(action) -> wait("response")')
# Returns: "response"
```

### Interpreting Wait Durations

```python
def parse_duration(wait_value):
    """Parse wait duration into time units."""
    patterns = {
        r'(\d+)s': ('seconds', 1),
        r'(\d+)m': ('minutes', 60),
        r'(\d+)\s*hours?': ('hours', 3600),
        r'(\d+)\s*days?': ('days', 86400),
    }

    for pattern, (unit, multiplier) in patterns.items():
        match = re.match(pattern, wait_value)
        if match:
            amount = int(match.group(1))
            return {
                'amount': amount,
                'unit': unit,
                'seconds': amount * multiplier
            }

    # Non-numeric duration (dramatic pause, etc.)
    return {
        'amount': None,
        'unit': 'narrative',
        'description': wait_value
    }

# Usage
duration = parse_duration("3 hours")
# Returns: {'amount': 3, 'unit': 'hours', 'seconds': 10800}

dramatic = parse_duration("tense moment")
# Returns: {'amount': None, 'unit': 'narrative', 'description': 'tense moment'}
```

### Backend Wait Processing

```python
def process_wait(wait_condition, game_state):
    """Determine if wait condition is met."""
    if is_duration(wait_condition):
        # Time-based wait
        game_state.advance_time(parse_duration(wait_condition))
        return True

    elif is_condition(wait_condition):
        # Conditional wait
        return game_state.check_condition(wait_condition)

    else:
        # Dramatic/narrative wait (always proceeds)
        return True

# Usage
if process_wait("3 hours", state):
    continue_narrative()
```

### Generating Wait NDL

```python
def create_wait(duration_or_condition):
    """Create wait NDL."""
    return f'wait("{duration_or_condition}")'

# Usage
ndl = create_wait("enemy responds")
# Returns: 'wait("enemy responds")'

# In sequence
seq = f'{action1} -> {create_wait("1s")} -> {action2}'
```

## Anti-Patterns

### ❌ Waiting Without Context

```ndl
wait("wait")
```

**Problem**: Unclear what is being waited for.

**Better**:
```ndl
wait("response")  # or "1s" or "door opens"
```

### ❌ Redundant Waits

```ndl
wait("pause") -> wait("pause") -> wait("pause")
```

**Problem**: Multiple identical waits with no intervening action.

**Better**:
```ndl
wait("long pause")
```

### ❌ Contradictory Wait

```ndl
do($"immediate action") -> wait("never ends")
```

**Problem**: Semantically contradictory.

### ❌ Missing Wait for Pacing

```ndl
do($"ask life-or-death question") -> do($"NPC answers")
```

**Problem**: No dramatic pause where one is expected.

**Better**:
```ndl
do($"ask life-or-death question") -> wait("tense silence") -> do($"NPC answers")
```

## Best Practices

### 1. Use Waits for Dramatic Pacing

```ndl
# Add tension
do($"light fuse") -> wait("fuse burns") -> result("explosion")

# Build anticipation
do($"knock on door") -> wait("footsteps approach") -> door_opens()
```

### 2. Specify Duration Clearly

```ndl
✓ wait("3 hours")
✓ wait("brief moment")
✗ wait("time")  # Too vague
```

### 3. Match Wait to Context

```ndl
# Combat: Short, dramatic
wait("split second")

# Travel: Long, summarized
wait("several days")

# Dialogue: Medium, tense
wait("uncomfortable pause")
```

### 4. Use Waits for NPC Reactions

```ndl
do($"player", "say shocking thing") -> wait("NPC processes") -> do($"NPC", "respond")
```

This gives NPCs realistic reaction time.

### 5. Combine with Scene Transitions

```ndl
wait("overnight") -> new_scene("morning") -> describe("well-rested")
```

## Design Philosophy

### Narrative Pacing

Wait controls the tempo of narration:
- Fast-paced: Few or no waits
- Deliberate: Frequent short waits
- Epic scope: Long waits (days, weeks)

### Realism

Characters don't act instantaneously:
```ndl
# Realistic
do($"ask") -> wait("NPC thinks") -> do($"NPC answers")

# Unrealistic
do($"ask") -> do($"NPC answers")
```

### LLM Guidance

The `wait()` function signals to the LLM:
- "Slow down the narration here"
- "Time passes"
- "Create a pause for effect"
- "Show anticipation or reaction time"

## Scene Boundary Detection

From veritasr's implementation, certain waits trigger scene transitions:

```python
is_ndl: True
end_scene: True
transition_scene_type: "both"  # time, place, or both
```

Long waits often indicate:
- `transition_scene_type: "time"` - Hours/days pass
- `transition_scene_type: "place"` - Travel time
- `transition_scene_type: "both"` - Journey over time and space

## Related Constructs

- [[sequencing]] - The `->` operator chains waits with actions
- [[do-action]] - Actions that waits separate
- [[result]] - Outcomes that waits precede
- [[system_response]] - Meta-information after waits

## See Also

- [[00-NDL-INDEX]] - Main NDL reference
- [[01-lexical-elements]] - Function syntax
- [[02-grammar]] - Wait in formal grammar
- [[patterns/scene-transitions]] - Using wait for transitions

---

> [!summary] Key Takeaway
> The `wait()` function is NDL's mechanism for controlling narrative pacing and representing the passage of time. By explicitly specifying pauses, delays, and timing between events, NDL enables the LLM to generate properly paced narration with realistic reaction times, dramatic tension, and scene transitions. Without `wait()`, all actions would feel simultaneous and rushed. With it, stories flow naturally with proper timing.
