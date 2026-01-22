---
tags: [ndl, construct, function, search, investigation]
date: 2026-01-17
related:
  - "[[do-action]]"
  - "[[result]]"
  - "[[intention]]"
  - "[[manner-modifier]]"
  - "[[00-NDL-INDEX]]"
---

# search() - Search Function

## Summary

The `search()` function represents investigation actions where a character examines a location, object, or area for items, clues, or secrets. It's a specialized action type that often combines with `intention=` (what you're looking for) and `result()` (what you find) to create complete investigation narratives.

## Syntax

```ndl
search("location_or_object")
search("target", intention="goal")
search("target") ~ "method"
search("target", intention="goal") ~ "method" -> result()
```

### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| target | String or Entity | Yes | What is being searched (location, object, person) |
| intention | Property | Optional | What the searcher hopes to find |
| manner | Modifier (~) | Optional | How the search is conducted |

## Description

The `search()` function is NDL's primary construct for investigation actions. It represents the deliberate act of looking for something, whether:
- Searching a location for items or clues
- Examining an object for hidden features
- Investigating a scene for evidence
- Looking for a specific item or person

Unlike general `do($"entity", "search")` actions, the `search()` function is a specialized keyword that signals investigation mechanics to both the parser and the LLM.

## Real Examples

### Basic Search with Intent and Result

```ndl
search("pile of wood", intention="find makeshift club") -> result()
```

**Actual LLM Output** (Gemma 2 9B IT):
```
Your fingers sift through the rough-hewn logs, searching for something sturdy and manageable. You find it - a thick, gnarled branch, its ends still bearing the marks of the axe. It's not much, but it's better than nothing.
```

**Analysis**:
- `search("pile of wood")` - specifies what is being searched
- `intention="find makeshift club"` - specifies what is sought
- `-> result()` - indicates success (finding something appropriate)
- LLM generates tactile, physical narration of the search and discovery

### Search with Manner

```ndl
search("room") ~ "carefully"
```

**LLM Output**:
```
You carefully search the room, methodically checking every corner, drawer, and surface. Nothing escapes your thorough examination.
```

**Analysis**: The manner `~ "carefully"` affects how the search is narrated (methodical, thorough vs hasty, cursory).

## Common Patterns

### Search with Intention

```ndl
search("debris", intention="find survivors")
search("desk", intention="find secret compartment")
search("corpse", intention="determine cause of death")
```

The intention guides what the searcher is looking for, affecting both game logic and narration.

### Search with Result

```ndl
search("room") -> result("find_hidden_door")
search("chest") -> result("find_nothing")
search("bookshelf") -> result("discover_secret_passage")
```

The result specifies what was found (or not found).

### Search with Method

```ndl
search("crime scene") ~ "forensic tools"
search("area") ~ "magical detection"
search("ruins") ~ "torch light"
```

The manner specifies tools or approach used in the search.

### Complete Search Chain

```ndl
search("ancient tome", intention="find ritual spell") ~ "carefully" -> result("success") -> gain("spell_knowledge")
```

Full context: what, why, how, outcome, and consequence.

## Search Targets

### Locations

```ndl
search("room")
search("forest clearing")
search("cave entrance")
search("crime scene")
search("battlefield")
```

### Objects

```ndl
search("desk")
search("chest")
search("corpse")
search("bookshelf")
search("pile of debris")
```

### Abstract Targets

```ndl
search("memories")
search("archives for records")
search("crowd for familiar face")
```

### Entity References

```ndl
search($"suspicious_crate")
search($"current_location")
```

## Intention in Searches

The intention property is particularly important for search actions:

### Specific Item Search

```ndl
search("room", intention="find key")
```

**LLM Focus**: Narration focuses on looking for key-like objects.

### General Investigation

```ndl
search("room", intention="find anything useful")
```

**LLM Focus**: Broader narration, examining various items.

### Clue Hunting

```ndl
search("scene", intention="determine what happened")
```

**LLM Focus**: Looking for contextual clues and evidence.

### Person Search

```ndl
search("tavern", intention="find informant")
```

**LLM Focus**: Scanning for people matching description.

## Manner in Searches

The manner modifier affects search approach and narration:

### Speed/Care

```ndl
search("room") ~ "hastily"  # Quick, might miss things
search("room") ~ "meticulously"  # Slow, thorough
search("room") ~ "carefully"  # Balanced approach
```

### Tools/Methods

```ndl
search("area") ~ "with hands"  # Tactile search
search("area") ~ "with torch"  # Visual search with light
search("area") ~ "with divination magic"  # Magical detection
search("area") ~ "with metal detector"  # Specialized tool
```

### Stealth/Obviousness

```ndl
search("office") ~ "discreetly"  # Don't disturb, hide search
search("office") ~ "thoroughly disrupting everything"  # Obvious, messy
```

## Result Outcomes

### Success

```ndl
search("location") -> result("find item")
search("location") -> result("discover secret")
search("location") -> result("locate clue")
```

### Failure

```ndl
search("location") -> result("find nothing")
search("location") -> result("miss hidden item")
search("location") -> result("false_lead")
```

### Complications

```ndl
search("trapped chest") -> result("trigger trap")
search("guarded room") -> result("alert guards")
search("cursed object") -> result("activate curse")
```

### Partial Success

```ndl
search("complex scene") -> result("partial_information")
search("damaged records") -> result("fragmentary_clues")
```

## Usage Examples by Genre

### Fantasy RPG

```ndl
search("dragon's hoard", intention="find magic sword") ~ "carefully avoiding treasure guardian" -> result("find legendary blade")
```

### Mystery/Detective

```ndl
search("crime scene", intention="find murder weapon") ~ "forensic examination" -> result("discover fingerprints")
```

### Horror

```ndl
search("abandoned house", intention="find way out") ~ "frantically" -> result("trigger something sinister")
```

### Sci-Fi

```ndl
search("alien ruins", intention="find technology") ~ "scanner device" -> result("detect energy signature")
```

### Heist

```ndl
search("security office", intention="find vault codes") ~ "quickly and quietly" -> result("success") -> gain("access codes")
```

## Integration with Game Systems

### Skill Check Integration

```ndl
search("hidden door") -> roll("perception") -> result("success") -> reveal($"secret_passage")
```

The search triggers a perception check, which determines the result.

### Difficulty Modifiers

```ndl
# Easy search
search("obvious location") -> result("immediate_find")

# Hard search
search("expertly hidden compartment") ~ "thorough search" -> roll("investigation", difficulty=20) -> result("barely_find")
```

### Time Consumption

```ndl
search("large library", intention="find specific book") ~ "methodically" -> wait("2 hours") -> result("find tome")
```

Searches can take time, represented by `wait()`.

### Inventory Interaction

```ndl
search("chest") -> result("find treasure") -> gain_items(["gold", "potion", "gem"])
```

Successful searches add items to inventory.

## Advanced Patterns

### Multi-Stage Search

```ndl
search("room") -> result("notice strange bookshelf")
-> do($"examine bookshelf") -> result("find mechanism")
-> do($"activate mechanism") -> reveal($"secret_passage")
```

Search leads to discovery, which leads to further investigation.

### Group Search

```ndl
search($"party", "battlefield", intention="find survivors") ~ "spreading out" -> result("find wounded soldier")
```

Multiple characters searching together.

### Repeated Search

```ndl
search("room", intention="find evidence") -> result("failure")
-> wait("think about it")
-> search("room", intention="look in unusual places") ~ "different approach" -> result("success")
```

Initial failure, followed by new strategy.

### Competitive Search

```ndl
search($"player", "ancient site") -> roll("investigation") -> result("15")
-> search($"rival", "ancient site") -> roll("investigation") -> result("18")
-> system_response("The rival finds the artifact first")
```

Multiple parties searching, one succeeds first.

## Implementation Notes

### Parsing Search Actions

```python
import re

def parse_search(ndl_string):
    """Parse search function from NDL."""
    match = re.search(r'search\("([^"]+)"(?:,\s*intention="([^"]+)")?\)', ndl_string)

    if match:
        return {
            'target': match.group(1),
            'intention': match.group(2) if match.group(2) else None
        }
    return None

# Usage
search_data = parse_search('search("room", intention="find key")')
# Returns: {'target': 'room', 'intention': 'find key'}
```

### Generating Search NDL

```python
def create_search(target, intention=None, manner=None, result=None):
    """Generate search NDL."""
    parts = []

    # Base search
    if intention:
        parts.append(f'search("{target}", intention="{intention}")')
    else:
        parts.append(f'search("{target}")')

    # Manner
    if manner:
        parts[-1] += f' ~ "{manner}"'

    # Result
    if result:
        parts.append(f'result("{result}")')

    return ' -> '.join(parts)

# Usage
ndl = create_search("chest", intention="find gold", manner="carefully", result="success")
# Returns: 'search("chest", intention="find gold") ~ "carefully" -> result("success")'
```

### Backend Search Logic

```python
def process_search(character, target, intention, difficulty):
    """Backend determines search outcome."""
    # Modify difficulty based on conditions
    if intention and intention in target.contains:
        difficulty -= 5  # Easier if you know what you're looking for

    # Roll perception/investigation
    roll = dice.d20() + character.perception_bonus

    if roll >= difficulty:
        # Success: find something
        if intention:
            item = target.find_specific(intention)
        else:
            item = target.find_random()

        return {
            'result': 'success',
            'found': item,
            'ndl': f'search("{target.name}", intention="{intention}") -> result("find {item.name}")'
        }
    else:
        # Failure: find nothing
        return {
            'result': 'failure',
            'found': None,
            'ndl': f'search("{target.name}", intention="{intention}") -> result("find_nothing")'
        }
```

## Anti-Patterns

### ❌ Missing Target

```ndl
search()  # What is being searched?
```

**Problem**: No target specified.

**Better**:
```ndl
search("room")
```

### ❌ Result Without Search

```ndl
result("find key")  # How was it found?
```

**Problem**: Result without the search action.

**Better**:
```ndl
search("desk") -> result("find key")
```

### ❌ Contradictory Intention and Result

```ndl
search("room", intention="find weapon") -> result("find spell scroll")
```

**Problem**: Found something different from what was sought (unless that's the point).

**Better**:
```ndl
search("room", intention="find weapon") -> result("find sword")
# or
search("room", intention="find weapon") -> result("find spell scroll instead")
```

### ❌ Overly Specific Search

```ndl
search("room where John hid the ancient ruby key behind the loose floorboard near the fireplace")
```

**Problem**: Too much detail makes search trivial.

**Better**:
```ndl
search("room", intention="find hidden key") ~ "checking floor near fireplace"
```

## Best Practices

### 1. Specify Target Clearly

```ndl
✓ search("suspect's office")
✗ search("place")
```

### 2. Use Intention for Focused Searches

```ndl
# Specific goal
search("library", intention="find forbidden tome")

# General investigation
search("library")
```

### 3. Match Manner to Context

```ndl
# Stealth scenario
search("guard room") ~ "quietly"

# Archaeology
search("ruins") ~ "archaeological methods"

# Emergency
search("rubble") ~ "frantically"
```

### 4. Always Provide Result for Important Searches

```ndl
✓ search("crime scene") -> result("find murder weapon")
✗ search("crime scene")  # LLM might invent what's found
```

### 5. Chain Search with Consequences

```ndl
search("desk") -> result("find letter") -> reveal("conspiracy") -> trigger($"plot_advancement")
```

## Design Philosophy

### Player Agency

Searches represent player choice to investigate:
```
Player: "I search the room"
Backend: Determines what is found (if anything)
NDL: search("room") -> result("find clue")
LLM: Narrates the search and discovery
```

### Systematic Investigation

Search is a game mechanic, not narrative fiat:
- Player initiates search
- Backend applies rules (perception check, difficulty, etc.)
- Result is deterministic
- LLM provides flavor text

### Reward for Thoroughness

Searches reward player attentiveness:
```ndl
# Player explicitly searches: finds item
search("bookshelf") -> result("find secret button")

# Player doesn't search: misses item
# (item remains hidden until searched)
```

## Related Constructs

- [[do-action]] - Generic action, search is specialized
- [[intention]] - Specifies search goal
- [[manner-modifier]] - Specifies search method
- [[result]] - Shows search outcome
- [[roll]] - Skill check for search success
- [[reveal]] - What search discovers

## See Also

- [[00-NDL-INDEX]] - Main NDL reference
- [[01-lexical-elements]] - Function syntax
- [[02-grammar]] - Search in formal grammar
- [[patterns/environmental-description]] - Search in exploration

---

> [!summary] Key Takeaway
> The `search()` function is NDL's specialized construct for investigation actions. By explicitly specifying what is being searched, why (intention), how (manner), and what is found (result), it enables deterministic, game-logic-driven investigation while allowing the LLM to generate evocative narration of discovery, examination, and revelation. This prevents the LLM from inventing items or clues that don't exist in the game world.
