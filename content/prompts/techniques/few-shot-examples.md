---
tags: [technique, few-shot, prompting-method, examples]
category: technique
status: essential
---

# Technique: Few-Shot Prompting

## Overview

Few-shot prompting provides the LLM with examples of desired input/output pairs before asking it to perform the task. This "teaches by example" rather than by explanation.

**Also called**: One-shot (1 example), few-shot (2-5 examples), many-shot (6+ examples)

## When to Use

✅ **Use few-shot when**:
- Teaching LLM a custom format (like NDL)
- Enforcing specific output structure
- Showing desired response style
- Constraining creativity to pattern
- Working with smaller models
- Format is easier to show than explain

❌ **Don't use few-shot when**:
- Task is self-explanatory
- Examples would be longer than instructions
- You need maximum creativity (examples constrain)
- Working with very limited token budget

## Basic Pattern

```
Here are examples of the desired format:

Example 1:
Input: {{EXAMPLE_INPUT_1}}
Output: {{EXAMPLE_OUTPUT_1}}

Example 2:
Input: {{EXAMPLE_INPUT_2}}
Output: {{EXAMPLE_OUTPUT_2}}

Now convert: {{ACTUAL_INPUT}}
Output:
```

## Community Insights

**50h100a**: "note that this requires few-shot or at least one-shot unless you are clever with formatting"

**monkeyrithms**: "gpt-4 was very good, its the only model that can kinda-sorta one-shot a good RP with all the rules just added to the context"

**Implication**: Smaller models need more examples. GPT-4 can often work with just instructions. Local 7B-9B models benefit greatly from 2-3 examples.

## Example 1: Teaching NDL Format

### Prompt
```
Convert natural language actions into NDL format.

Example 1:
Input: "I draw my sword"
Output: do($"player", "draw")~"weapon"->item($"sword")

Example 2:
Input: "She whispers nervously"
Output: do($"npc_name", "speak")~"whisper"->emotion("nervous")

Example 3:
Input: "He attacks the goblin with his axe"
Output: do($"npc_name", "attack")~"axe"->target($"goblin")

Now convert: "The player sneaks past the guard quietly"
Output:
```

### Expected Output
```
do($"player", "sneak")~"quietly"->target($"guard")
```

## Example 2: Enforcing Parseable Format

### Prompt
```
Generate character equipment in exactly this format.

Example 1:
Input: Medieval warrior
Output:
EQUIPMENT:
- Iron Longsword
- Wooden Shield
- Leather Armor
- Belt Pouch (contains 10 gold coins)

Example 2:
Input: Fantasy wizard
Output:
EQUIPMENT:
- Oak Staff
- Spellbook
- Blue Robes
- Component Pouch (contains spell reagents)

Now generate equipment for: Post-apocalyptic scavenger
Output:
```

### Expected Output
```
EQUIPMENT:
- Makeshift Knife
- Patched Backpack
- Gas Mask
- Canteen (half-full)
```

## Example 3: Constraint Enforcement

### Prompt
```
Respond to impossible actions by describing realistic failure.

Example 1:
Player: "I cast fireball"
Character: Warrior (no magic abilities)
Response: You raise your hands dramatically, but nothing happens. You're a warrior, not a mage - you have no magical training.

Example 2:
Player: "I jump to the rooftop"
Character: Normal human
Setting: Rooftop is 15 feet high
Response: You jump as high as you can, but the rooftop is far too high. You don't even come close. You'll need a ladder or another way up.

Example 3:
Player: "I pull out my lockpicks"
Character Inventory: Sword, shield, 10 gold (no lockpicks)
Response: You pat your pockets, but you don't have any lockpicking tools. You'll need to find some or use another approach.

Now respond to:
Player: "I fly across the chasm"
Character: Knight in heavy armor
Setting: Chasm is 30 feet across
Response:
```

### Expected Output
```
You take a running start toward the chasm, but your heavy armor weighs you down. You manage a few feet before plummeting toward the bottom. You're a knight, not a bird - flying is impossible without magical assistance or equipment.
```

## Example 4: Dialogue Style

### Prompt
```
Write tavern NPC dialogue in medieval fantasy style.

Example 1:
Context: Bartender greeting regular customer
Dialogue: "Well met, Gareth! The usual ale? Just tapped a fresh cask this morning."

Example 2:
Context: Suspicious guard questioning stranger
Dialogue: "State your business, outsider. We don't get many travelers through here, especially ones with that look about them."

Example 3:
Context: Merchant haggling over price
Dialogue: "Twenty silver? For this fine blade? My friend, I couldn't part with it for less than thirty - feel the balance yourself!"

Now write:
Context: Nervous innkeeper when asked about local bandits
Dialogue:
```

### Expected Output
```
"Bandits? Aye, they're around..." The innkeeper glances toward the door, lowering his voice. "Most folk here learned to keep their heads down and their doors locked. Best you do the same."
```

## Optimizing Few-Shot Examples

### Quality over Quantity
- 2-3 good examples > 10 mediocre examples
- Each example should show a different aspect
- Examples should be representative of edge cases

### Example Selection Strategy
1. **Prototypical**: Show the most common case
2. **Edge case**: Show an unusual but important case
3. **Failure mode**: Show what NOT to do (if using negative examples)

### Diverse Examples
```
Example 1: Simple case
Example 2: Complex case
Example 3: Edge case

This covers more ground than 3 similar examples.
```

## Zero-Shot vs One-Shot vs Few-Shot

### Zero-Shot (No Examples)
```
Convert the player's action into structured format:
Player action: "I sneak past the guard"
Structured format:
```

Works for: GPT-4, Claude, very clear tasks

### One-Shot (1 Example)
```
Convert the player's action into structured format.

Example:
Input: "I draw my sword"
Output: action(draw, weapon=sword, actor=player)

Now convert:
Input: "I sneak past the guard"
Output:
```

Works for: Medium models, straightforward tasks

### Few-Shot (2-5 Examples)
```
Convert the player's action into structured format.

Example 1:
Input: "I draw my sword"
Output: action(draw, weapon=sword, actor=player)

Example 2:
Input: "She casts a spell"
Output: action(cast, spell=unknown, actor=npc)

Example 3:
Input: "He attacks the goblin"
Output: action(attack, target=goblin, actor=npc)

Now convert:
Input: "I sneak past the guard"
Output:
```

Works for: Smaller models (7B-9B), complex formats, ensuring consistency

## Common Mistakes

### 1. Examples Too Similar
**Bad**:
```
Example 1: "I attack the goblin" -> action(attack, goblin)
Example 2: "I attack the orc" -> action(attack, orc)
Example 3: "I attack the dragon" -> action(attack, dragon)
```

All examples show the same pattern. No edge cases covered.

**Good**:
```
Example 1: "I attack the goblin" -> action(attack, target=goblin)
Example 2: "I drink a potion" -> action(consume, item=potion)
Example 3: "She whispers to him" -> action(communicate, method=whisper, target=npc)
```

Diverse actions showing different aspects of the format.

### 2. Examples Too Complex
**Bad**:
```
Example 1: [500 token complex scenario]
```

**Good**:
```
Example 1: [50 token focused scenario]
Example 2: [50 token focused scenario]
```

Short, focused examples are better.

### 3. Inconsistent Examples
**Bad**:
```
Example 1: action(attack, goblin)
Example 2: ACTION{attack; target=orc}
Example 3: do_action("attack", "dragon")
```

Different formats confuse the model.

**Good**:
```
Example 1: action(attack, target=goblin)
Example 2: action(defend, method=shield)
Example 3: action(cast, spell=fireball)
```

Consistent format across all examples.

## Integration with Other Techniques

### Few-Shot + CoT
```
Example 1:
Input: "I cast fireball"
Reasoning: Character is a warrior -> No magic -> Impossible
Output: "You have no magical abilities."

Example 2:
Input: "I draw my sword"
Reasoning: Character has a sword -> Simple action -> Possible
Output: "You draw your sword from its scabbard."

Now process:
Input: {{ACTION}}
Reasoning:
```

### Few-Shot + Constraints
```
Example 1:
Action: "I fly"
Constraint: No flight ability
Output: "You can't fly."

Example 2:
Action: "I teleport"
Constraint: No teleportation
Output: "Teleportation is impossible."

Now with same constraints:
Action: {{NEW_ACTION}}
Output:
```

### Few-Shot + NDL
```
Example 1:
NDL: do($"player", "attack")~"sword"->target($"goblin")->result("hit")
Narration: "You swing your sword at the goblin, striking true."

Example 2:
NDL: do($"player", "sneak")~"quietly"->check("stealth", result="success")
Narration: "You move silently through the shadows, undetected."

Now narrate:
NDL: {{NEW_NDL}}
Narration:
```

## Model-Specific Considerations

### Small Models (7B-9B)
- Need 2-3 examples minimum
- Examples should be very clear
- Format should be simple
- More examples = better consistency

### Medium Models (13B-30B)
- 1-2 examples usually sufficient
- Can handle more complex formats
- Examples can be more nuanced

### Large Models (GPT-4, Claude)
- Often work zero-shot
- Use few-shot for precision/style
- Can learn from just 1 example
- Examples set tone rather than teach format

## Temperature Settings

When using few-shot:
- Lower temperature (0.1-0.4) for following format strictly
- Higher temperature (0.6-0.8) for creative content within format

## Testing Few-Shot Effectiveness

Try edge cases to see if pattern was learned:

```
Test cases:
1. Typical case (should match examples)
2. Unusual case (shows generalization)
3. Ambiguous case (reveals understanding)
4. Invalid case (shows constraint learning)
```

## Related Techniques
- [[chain-of-thought]] - Can combine with few-shot
- [[ndl-to-narrative]] - Uses few-shot to teach NDL
- [[format-enforcement]] - Few-shot is a format enforcement method

## Source
- Discussion context: Early 2024 format teaching for NDL
- Referenced throughout: [[02-Prompt-Engineering]]
- Critical for: Small model success with custom formats
