---
tags: [ndl, construct, function, result, outcome]
date: 2026-01-17
related:
  - "[[do-action]]"
  - "[[sequencing]]"
  - "[[system_response]]"
  - "[[00-NDL-INDEX]]"
---

# result() - Result Function

## Summary

The `result()` function specifies the outcome of an action or skill check. It represents what actually happened after an action was attempted, whether success or failure. This construct is central to NDL's deterministic approach - the backend decides the outcome, and the LLM narrates it.

## Syntax

```ndl
result()
result("outcome")
result("success")
result("failure")
```

### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| outcome | String | Optional | The result of the action (success, failure, hit, miss, critical, etc.) |

## Description

The `result()` function communicates to the LLM what actually occurred. It appears in NDL sequences after actions and skill checks to specify the determined outcome. This prevents the LLM from inventing results that contradict game state.

The result function is typically used in conjunction with:
- Actions (`do()`) - showing what happened
- Skill checks (`roll()`) - showing check results
- Combat (`attack`, `defend`) - showing hits/misses
- Social interactions - showing persuasion success/failure
- Investigation - showing what was found/not found

> [!important] Core Principle
> The backend determines results through game logic (dice rolls, skill checks, state evaluation). The LLM never decides success or failure - it only narrates the predetermined outcome.

## Real Examples

### Search with Result

```ndl
search("pile of wood", intention="find makeshift club") -> result()
```

**Actual LLM Output** (Gemma 2 9B IT):
```
Your fingers sift through the rough-hewn logs, searching for something sturdy and manageable. You find it - a thick, gnarled branch, its ends still bearing the marks of the axe. It's not much, but it's better than nothing.
```

**Explanation**: The `result()` indicates the search succeeded. The LLM knows to narrate finding something appropriate.

### Failed Action Result

```ndl
do($"you", "swing makeshift club against table's surface", intention="test club's combat readiness")
-> result("failure")
-> system_response("something goes wrong when you try to perform this action")
```

**Actual LLM Output** (Gemma 2 9B IT):
```
You raise the gnarled branch, hefting it awkwardly in your hand. It's heavier than you anticipated, the rough bark digging into your palm. You bring it back over your head, aiming for the tabletop. But as you swing, your momentum falters. The branch veers off course, catching on the edge of the chair.

With a sickening crack, the makeshift club splinters, breaking into several pieces. The impact sends a shower of splinters flying, scattering across the floor. You stare down at the useless fragments in your hand, your heart sinking.
```

**Explanation**: The `result("failure")` tells the LLM the action failed, enabling realistic failure narration (weapon breaking).

## Common Result Values

### Binary Outcomes

```ndl
result("success")
result("failure")
result("pass")
result("fail")
```

### Combat Results

```ndl
result("hit")
result("miss")
result("critical_hit")
result("critical_miss")
result("glancing_blow")
```

### Degrees of Success

```ndl
result("overwhelming_success")
result("success")
result("partial_success")
result("failure")
result("catastrophic_failure")
```

### Investigation Results

```ndl
result("find_clue")
result("find_nothing")
result("discover_secret")
result("trigger_trap")
```

### Social Results

```ndl
result("convinced")
result("skeptical")
result("offended")
result("neutral")
```

## Usage Patterns

### Action → Result

```ndl
do($"player", "attack") -> result("hit")
```

Most basic pattern: action followed by outcome.

### Action → Check → Result

```ndl
do($"player", "persuade merchant") -> roll("charisma") -> result("success")
```

Incorporates the skill check between action and result.

### Action → Result → Consequence

```ndl
do($"player", "attack") -> result("hit") -> damage(15)
```

Result leads to further effects.

### Action → Result → System Message

```ndl
do($"player", "lockpick") -> result("failure") -> system_response("The lock is too complex")
```

Result is explained with meta-information.

### Action → Result → Description

```ndl
do($"player", "search") -> result("find treasure") -> describe("You discover a hidden compartment")
```

Result followed by explicit narration guidance.

## Success vs Failure

### Success Narration

When `result("success")` or similar:

```ndl
do($"player", "climb wall") -> result("success")
```

**LLM Output**:
```
You scale the wall successfully, finding handholds and footholds with practiced ease. Reaching the top, you pull yourself over the edge.
```

The LLM narrates achievement, completion, and positive outcome.

### Failure Narration

When `result("failure")` or similar:

```ndl
do($"player", "climb wall") -> result("failure")
```

**LLM Output**:
```
You attempt to climb the wall, but the stones are slippery and your grip fails. You slide back down, frustrated but unharmed.
```

The LLM narrates the attempt, the failure point, and consequences.

## Combining with Other Constructs

### With Intention

```ndl
do($"player", "intimidate") ~ "menacing" intention="make guards flee" -> result("partial_success")
```

**LLM Output**:
```
You advance menacingly, trying to frighten the guards. They step back nervously but don't flee entirely - you've shaken them, but not broken their resolve.
```

The intention shows the goal, the result shows partial achievement.

### With Target

```ndl
do($"player", "shoot arrow") -> target($"goblin") -> result("hit") -> damage(8)
```

**LLM Output**:
```
You loose your arrow at the goblin. It flies true, striking the creature in the shoulder for 8 points of damage.
```

### With Manner

```ndl
do($"player", "negotiate") ~ "diplomatically" -> result("success") -> gain("trade agreement")
```

**LLM Output**:
```
You negotiate diplomatically, choosing your words carefully. Your measured approach pays off - the other party agrees to a mutually beneficial trade agreement.
```

## Empty vs Valued Result

### Empty Result: `result()`

```ndl
search($"room") -> result()
```

The LLM infers the outcome from context. Generally implies success or neutral outcome.

### Explicit Result: `result("value")`

```ndl
search($"room") -> result("find secret door")
```

Provides specific outcome for the LLM to narrate.

### When to Use Each

**Use `result()`**:
- Outcome is obvious from context
- Simple success/completion
- Chaining needs a result marker

**Use `result("value")`**:
- Outcome needs specification
- Success/failure determination
- Specific degree or type of outcome

## Result in Sequences

### Linear Success Chain

```ndl
attempt -> result("success") -> consequence -> new_state
```

### Branching on Result

```ndl
attempt1 -> result("failure")
-> attempt2 -> result("failure")
-> attempt3 -> result("success")
```

Shows multiple attempts with different results.

### Result with Recovery

```ndl
action -> result("failure") -> fallback_action -> result("success")
```

Failure prompts alternative approach.

### Cascading Results

```ndl
action1 -> result("success")
-> enables(action2) -> result("success")
-> enables(action3) -> result("critical_success")
```

Success chains enable further successes.

## Implementation Notes

### Backend Determination

```python
def determine_result(action, character, target, difficulty):
    """Backend determines result based on game logic."""
    roll = random.randint(1, 20) + character.skill_bonus

    if roll >= difficulty + 10:
        return "critical_success"
    elif roll >= difficulty:
        return "success"
    elif roll >= difficulty - 5:
        return "partial_success"
    else:
        return "failure"

# Usage
outcome = determine_result("persuade", player, merchant, 15)
ndl = f'do($"player", "persuade") -> result("{outcome}")'
```

### Parsing Results

```python
import re

def parse_result(ndl_string):
    """Extract result value from NDL."""
    match = re.search(r'result\("([^"]+)"\)', ndl_string)
    if match:
        return match.group(1)

    # Empty result
    if 'result()' in ndl_string:
        return None  # or "success" as default

    return None

# Usage
outcome = parse_result('do(action) -> result("hit")')
# Returns: "hit"
```

### Generating Result NDL

```python
def add_result(action_ndl, outcome=None):
    """Add result to action NDL."""
    if outcome:
        return f'{action_ndl} -> result("{outcome}")'
    else:
        return f'{action_ndl} -> result()'

# Usage
ndl = add_result('do($"attack")', "critical_hit")
# Returns: 'do($"attack") -> result("critical_hit")'
```

## Anti-Patterns

### ❌ LLM Deciding Result

```ndl
do($"attack")  # No result specified
```

**Problem**: LLM might invent hit/miss, contradicting game state.

**Better**:
```ndl
do($"attack") -> result("hit")
```

### ❌ Contradictory Results

```ndl
do($"attack") intention="kill" -> result("failure") -> damage(50)
```

**Problem**: Can't deal damage if attack failed.

**Better**:
```ndl
do($"attack") intention="kill" -> result("failure") -> damage(0)
```

### ❌ Vague Results

```ndl
result("something happens")
```

**Problem**: Too vague for meaningful narration.

**Better**:
```ndl
result("partial_success")  # or specific outcome
```

### ❌ Missing Result After Check

```ndl
do($"persuade") -> roll("charisma")  # No result
```

**Problem**: Roll outcome not specified.

**Better**:
```ndl
do($"persuade") -> roll("charisma") -> result("success")
```

## Best Practices

### 1. Always Specify Critical Results

```ndl
# Combat: Always show hit/miss
do($"attack") -> result("hit")

# Social: Always show convince/fail
do($"persuade") -> result("success")

# Investigation: Always show find/miss
do($"search") -> result("find clue")
```

### 2. Match Result Granularity to Context

```ndl
# Simple context: binary
result("success")

# Complex context: degrees
result("overwhelming success with minor complication")
```

### 3. Use Result to Guide LLM Tone

```ndl
# Critical failure: LLM should narrate dramatically
result("catastrophic_failure")

# Minor success: LLM should be matter-of-fact
result("barely_succeed")
```

### 4. Chain Results with Consequences

```ndl
action -> result("success") -> gain("reward")
action -> result("failure") -> lose("item")
```

### 5. Differentiate Attempt from Outcome

```ndl
# Attempt (action)
do($"lockpick")

# Outcome (result)
-> result("failure")

# Explanation (system)
-> system_response("The lock is beyond your skill")
```

## Result Types by Context

### Combat

```ndl
result("hit")
result("miss")
result("critical_hit")
result("fumble")
result("glancing_blow")
result("blocked")
result("parried")
result("dodged")
```

### Social

```ndl
result("convinced")
result("skeptical")
result("befriended")
result("offended")
result("neutral")
result("intrigued")
result("hostile")
```

### Skill Checks

```ndl
result("success")
result("failure")
result("partial_success")
result("critical_success")
result("critical_failure")
```

### Investigation

```ndl
result("find_clue")
result("discover_secret")
result("trigger_trap")
result("find_nothing")
result("false_lead")
```

### Crafting

```ndl
result("masterwork")
result("success")
result("flawed")
result("broken")
result("exceptional")
```

## Design Philosophy

### Determinism

The backend computes results:
- Dice rolls
- Skill checks
- State evaluation
- NPC reactions

NDL communicates these to the LLM.

### Separation of Concerns

```
Backend: Determines IF action succeeds
NDL: Communicates THAT it succeeded
LLM: Narrates HOW it succeeded
```

### No Hallucination Space

By specifying results explicitly, NDL eliminates the LLM's ability to invent outcomes that contradict game state.

> [!success] Outcome
> "once you offload the decision making to a system that's actually capable of making sane decisions the hallucinations functionally disappear." - veritasr

## Related Constructs

- [[do-action]] - The action whose result is shown
- [[roll]] - Often precedes result
- [[damage]] - Often follows combat result
- [[system_response]] - Explains result
- [[sequencing]] - Chains results with actions

## See Also

- [[00-NDL-INDEX]] - Main NDL reference
- [[01-lexical-elements]] - Function syntax
- [[02-grammar]] - Result in formal grammar
- [[ndl-to-llm-flow]] - How results flow to LLM

---

> [!summary] Key Takeaway
> The `result()` function is NDL's mechanism for communicating predetermined outcomes to the LLM. By explicitly stating what happened (success, failure, degrees of success), NDL ensures the LLM narrates the actual game state rather than inventing its own version of events. This is fundamental to eliminating hallucinations in LLM-driven game narration.
