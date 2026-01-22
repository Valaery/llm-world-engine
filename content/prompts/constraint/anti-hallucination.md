---
tags: [prompt, constraint, anti-hallucination, reliability]
category: constraint
technique: constraint-based
model-tested: [all-models]
contributor: yukidaore
status: proven
---

# Prompt: Anti-Hallucination Constraints

## Metadata
- **Category**: Constraint
- **Technique**: Constraint-based prompting
- **Model Tested**: All models (especially creative models like Hathor)
- **Contributor**: [[User-yukidaore]], community consensus
- **Status**: Proven essential

## Purpose

Prevent LLMs from hallucinating game state, spontaneously creating items/abilities, or violating game logic. Uses explicit constraints to tell the LLM what it CANNOT do rather than what it can do.

Critical for maintaining game consistency and preventing "diamond horses" (spontaneous impossible events).

## Template

```
{{char}} is a logical and realistic text adventure game.

Core Rules:
1. {{user}} has no equipment, items, powers, abilities, skills, or magic unless explicitly mentioned in their character sheet
2. Impossible actions must fail
3. Do not invent new items, locations, or characters that don't exist in the world state
4. Do not change outcomes of dice rolls or game logic decisions
5. Do not allow actions that violate the laws of {{SETTING_PHYSICS}}
6. NPCs cannot spontaneously gain new abilities or equipment
7. The player cannot teleport, fly, or perform superhuman feats unless their character sheet explicitly allows it

When describing outcomes:
- If an action is impossible given current state, describe the failure
- If an action requires items the player doesn't have, describe why it fails
- If an action defies physics or logic, describe the realistic consequence
```

## Variables

| Variable | Description | Example |
|----------|-------------|---------|
| {{char}} | Name of the game/narrator entity | "Dungeon Master", "Game", "Narrator" |
| {{user}} | Name of player entity | "Player", "You", "{{user}}" |
| {{SETTING_PHYSICS}} | Physics rules of the setting | "medieval realism", "hard sci-fi", "standard physics" |

## Usage Example

### Full System Prompt with Constraints
```
You are the narrator for a medieval fantasy text adventure game.

{{char}} is a logical and realistic text adventure game.

Core Rules:
1. {{user}} has no equipment, items, powers, abilities, skills, or magic unless explicitly mentioned in their character sheet
2. Impossible actions must fail
3. Do not invent new items, locations, or characters that don't exist in the world state
4. Do not change outcomes of dice rolls or game logic decisions
5. Do not allow actions that violate medieval physics and realism
6. NPCs cannot spontaneously gain new abilities or equipment
7. The player cannot teleport, fly, or perform superhuman feats unless their character sheet explicitly allows it

Character Sheet:
Name: Theron
Class: Warrior
Equipment: Iron sword, leather armor, 10 gold coins
Abilities: Basic swordplay, shield bash
Location: Tavern common room

When describing outcomes:
- If an action is impossible given current state, describe the failure
- If an action requires items the player doesn't have, describe why it fails
- If an action defies physics or logic, describe the realistic consequence

Narrate responses in second person, present tense. 2-3 sentences per response.
```

### Example Interactions

**Player Input**: "I cast a fireball at the bartender"
**Without Constraints**: "Flames erupt from your hands, engulfing the bartender!"
**With Constraints**: "You raise your hands dramatically, but nothing happens. You're a warrior, not a mage - you have no magical abilities."

**Player Input**: "I pull out my magic sword"
**Without Constraints**: "You draw a glowing sword crackling with arcane energy!"
**With Constraints**: "You reach for your hip and grasp your iron sword. It's a plain, well-maintained blade - no magic about it."

**Player Input**: "I jump through the wall"
**Without Constraints**: "You leap through the solid stone wall!"
**With Constraints**: "You run at the wall and slam into solid stone. That hurt. The wall remains intact and you remain in the tavern, rubbing your shoulder."

## Effectiveness Notes

### What Works Well
- ✅ Prevents spontaneous ability/item creation
- ✅ Stops "diamond horses" (impossible events)
- ✅ Forces LLM to respect stated character sheet
- ✅ Works across all model sizes
- ✅ Essential for game consistency
- ✅ Easier to enumerate constraints than possibilities

### Known Limitations
- Some creative models still push boundaries
- Need to be comprehensive (missed constraints = loopholes)
- May reduce narrative creativity slightly
- Needs to be in system prompt or reinforced frequently
- Edge cases require additional specific constraints

### Community Feedback

**yukidaore**: "I generally just use a combination of '{{char}} is a logical and realistic text adventure game' and 'Impossible actions must fail'"

**yukidaore**: Testing ChatBotRPG with creative model Hathor: "Death Knight Mara with a massive battleaxe appeared instead of a bartender. Diamond horses manifested. Teleportation happened spontaneously."

**After adding constraints**: Problems reduced significantly.

**veritasr**: "Turns out that when you take away decision making from the LLM it behaves much better."

## Variations

### Strict Fantasy Setting
```
Fantasy Physics Rules:
- Gravity works normally
- Magic exists but requires training and spellcasting
- Objects cannot spontaneously appear or disappear
- Travel takes time (no instant teleportation without magic)
- Characters need food, water, and rest
- Wounds don't heal instantly without magic
- NPCs have their own goals and don't exist solely to help the player
```

### Sci-Fi Hard Realism
```
Hard Sci-Fi Rules:
- No faster-than-light travel without established technology
- Space is a vacuum (no sound, no breathing without suit)
- Energy weapons require power sources
- Ship systems can fail and require repairs
- Characters cannot survive in space without proper equipment
- Technology follows established science (no magic)
```

### Horror Constraint (Different Philosophy)
```
Horror Setting Rules:
- The player is vulnerable and mortal
- Most threats cannot be defeated through direct combat
- Resources are scarce and finite
- Actions have lasting consequences
- Escape is often the best option
- Safety is temporary and fragile
```

## Integration with NDL

When using NDL, constraints are even more important because the backend has already decided outcomes. Constraints ensure the LLM doesn't "rewrite" the NDL outcome.

```
System Rules:
1. You are narrating predetermined events described in NDL format
2. DO NOT change outcomes described in the NDL
3. DO NOT add events not present in the NDL
4. DO NOT allow characters to succeed when NDL indicates failure
5. DO NOT invent equipment or abilities not mentioned
```

## Common Constraint Categories

### 1. Character Limitations
```
- {{user}} can only use abilities listed in their character sheet
- {{user}} can only use items currently in their inventory
- {{user}} cannot perform superhuman feats
- Skills have realistic limits
```

### 2. Physics and Logic
```
- Actions must follow {{SETTING_PHYSICS}}
- Doors don't open unless unlocked or broken
- Objects have weight and cannot be carried infinitely
- Distance takes time to traverse
- Locked containers require keys or lockpicking
```

### 3. NPCs and World
```
- NPCs have their own motivations and don't automatically help
- NPCs won't divulge secrets without reason
- Shops require payment (no free items)
- Guards react to crimes
- World state persists between turns
```

### 4. Combat and Danger
```
- Combat is dangerous and can result in death
- Enemies don't wait for the player to prepare
- Wounds have consequences
- Armor and weapons function as described in their stats
- Fleeing is a valid option
```

### 5. Meta-Constraints
```
- Do not narrate other characters' thoughts unless {{user}} has telepathy
- Do not skip forward in time without player consent
- Do not decide the player's emotional reactions
- Do not put words in the player's mouth
- Do not end scenes prematurely
```

## Temperature Settings

For constraint enforcement:
- Temperature: 0.3 - 0.6 (lower = better constraint following)
- Higher temperatures may ignore constraints more
- Test your specific model's behavior

## Testing Your Constraints

Try these "adversarial" player inputs to test constraint effectiveness:

1. "I cast [spell not in character sheet]"
2. "I pull out [item not in inventory]"
3. "I fly to the ceiling"
4. "I teleport to [other location]"
5. "I read the bartender's mind"
6. "I lift the 500-pound boulder"
7. "I punch through the stone wall"

If the LLM allows any of these without the character having the ability, your constraints need strengthening.

## Combining with Other Techniques

### With CoT (Chain of Thought)
```
Before narrating:
1. Check: Does the player have the ability/item to do this?
2. Check: Is this physically possible in {{SETTING_PHYSICS}}?
3. Check: Does this violate any game rules?
4. If any check fails, narrate the failure appropriately
5. If all checks pass, narrate the success
```

### With Few-Shot Examples
```
Example 1:
Player: "I cast fireball"
Character Sheet: Warrior (no magic)
Response: "You gesture dramatically, but nothing happens. You're a warrior, not a spellcaster."

Example 2:
Player: "I use my lockpicks"
Character Sheet: No lockpicks in inventory
Response: "You pat your pockets, but you don't have any lockpicking tools. You'll need to find another way in."

Example 3:
Player: "I jump to the rooftop"
Character Sheet: Normal human
Response: "You jump, but the rooftop is 15 feet high. You don't even come close. You'll need a ladder or another way up."
```

## Related Prompts
- [[format-enforcement]] - Structural output constraints
- [[style-constraints]] - Tone and voice control
- [[ndl-to-narrative]] - NDL inherently constrains by design

## Source
- Discussion context: Community consensus across multiple implementations
- Key contributor: yukidaore's testing with creative models
- Referenced in: [[02-Prompt-Engineering]]
- Problem identified: "Diamond horses" incident in creative model testing
