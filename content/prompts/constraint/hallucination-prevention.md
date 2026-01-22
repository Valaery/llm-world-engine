---
tags: [prompt, constraint, grounding, anti-hallucination, data-grounding]
category: constraint
technique: explicit-prohibitions
model-tested: [all-models]
contributor: yukidaore, veritasr
status: proven
---

# Prompt: Hallucination Prevention

## Metadata
- **Category**: Constraint
- **Technique**: Explicit prohibitions + data grounding
- **Model Tested**: All models (especially important for creative/local models)
- **Contributor**: [[User-yukidaore]], [[User-veritasr]], community consensus
- **Status**: Proven

## Purpose
Prevent LLMs from inventing content beyond provided data. Critical for game consistency - stops "diamond horses", spontaneous teleportation, phantom NPCs, and other hallucinated content that breaks game logic.

## Template
```
{{base_prompt_or_task}}

CONSTRAINTS - Follow these rules strictly:
- {{char}} is a logical and realistic {{genre}} game
- {{user}} has no equipment, items, powers, abilities, skills, or magic unless explicitly mentioned in: {{inventory_data}}
- Impossible actions must fail with realistic explanations
- Do not invent characters, locations, or items not provided in the data
- Do not add events or encounters not specified
- Do not assume player capabilities beyond what is stated
- Do not create new plot developments
- Do not introduce dialogue from characters not present
- If information is not provided, say "You don't know" or "You don't see that" rather than inventing

Available data:
{{provided_context}}

Generate response based ONLY on the data provided above.
```

## Variables
| Variable | Description | Example |
|----------|-------------|---------|
| {{base_prompt_or_task}} | The main instruction | "Narrate the player's action", "Describe the room" |
| {{char}} | System reference | "The game", "The system", "This world" |
| {{genre}} | Game type for grounding | "text adventure", "fantasy RPG", "sci-fi simulation" |
| {{user}} | Player reference | "The player", "You", "The character" |
| {{inventory_data}} | Current possessions | "worn cloak, rusty sword, 3 gold coins" |
| {{provided_context}} | All available information | Character stats, location desc, NPCs present, quest data |

## Usage Example

### Example 1: Combat Action
```
Narrate the result of the player attacking the bandit with their sword.

CONSTRAINTS - Follow these rules strictly:
- The game is a logical and realistic fantasy RPG
- The player has no equipment, items, powers, abilities, skills, or magic unless explicitly mentioned in: rusty iron sword, leather armor, 2 health potions
- Impossible actions must fail with realistic explanations
- Do not invent characters, locations, or items not provided in the data
- Do not add events or encounters not specified
- Do not assume player capabilities beyond what is stated
- Do not create new plot developments
- Do not introduce dialogue from characters not present
- If information is not provided, say "You don't know" or "You don't see that" rather than inventing

Available data:
Location: Forest road
Characters present: Player, Bandit (armed with club, leather armor, alert)
Player action: Attack with sword
Roll result: Success
Damage: Moderate hit to bandit's arm

Generate response based ONLY on the data provided above.
```

**Expected Output (Good)**:
```
Your rusty sword connects with the bandit's arm, cutting through his leather armor. He grunts in pain and staggers back, his club wavering in his grip.
```

**Would Prevent (Bad)**:
```
Your enchanted blade flares with holy fire as it strikes! [❌ Sword not enchanted]
The bandit calls for reinforcements! [❌ No reinforcements mentioned]
You follow up with a spinning kick! [❌ No second action authorized]
```

### Example 2: Exploration
```
The player examines the room.

CONSTRAINTS - Follow these rules strictly:
- The game is a logical and realistic dungeon crawler
- The player has no equipment, items, powers, abilities, skills, or magic unless explicitly mentioned in: torch, rope (50 ft), dagger
- Impossible actions must fail with realistic explanations
- Do not invent characters, locations, or items not provided in the data
- Do not add events or encounters not specified
- Do not assume player capabilities beyond what is stated
- Do not create new plot developments
- Do not introduce dialogue from characters not present
- If information is not provided, say "You don't know" or "You don't see that" rather than inventing

Available data:
Location: Stone chamber (20x20 ft)
Features: Wooden door (north), torch sconce (empty), cracked floor tiles
Hidden content: None detected (player has not searched)

Generate response based ONLY on the data provided above.
```

**Expected Output (Good)**:
```
The stone chamber is roughly 20 feet square. A wooden door stands in the north wall, and an empty torch sconce juts from the eastern wall. Several floor tiles show cracks, though whether from age or something else, you can't immediately tell.
```

**Would Prevent (Bad)**:
```
You notice ancient runes carved into the walls [❌ Not in features list]
A skeleton lies in the corner [❌ Not in data]
You sense a magical presence [❌ Player has no magic sense]
The door is locked [❌ Door state not specified, don't assume]
```

### Example 3: Social Interaction
```
Generate the NPC's response to the player asking about the king.

CONSTRAINTS - Follow these rules strictly:
- The game is a logical and realistic medieval fantasy game
- The player has no equipment, items, powers, abilities, skills, or magic unless explicitly mentioned in: traveler's clothes, small coinpurse
- Impossible actions must fail with realistic explanations
- Do not invent characters, locations, or items not provided in the data
- Do not add events or encounters not specified
- Do not assume player capabilities beyond what is stated
- Do not create new plot developments
- Do not introduce dialogue from characters not present
- If information is not provided, say "You don't know" or "You don't see that" rather than inventing

Available data:
NPC: Innkeeper Mara (friendly, talkative, loyal to crown)
NPC knows about: Local gossip, tavern business, general kingdom news
NPC does NOT know: Specific court politics, military secrets, royal family details
Player asked: "What can you tell me about the king?"

Generate response based ONLY on the data provided above.
```

**Expected Output (Good)**:
```
"The king?" Mara wipes down the bar. "Good man, from what folks say. Keeps the roads safe, taxes aren't too heavy. That's about all I know - I run a tavern, not the royal court." She chuckles. "You want the latest gossip, I'm your woman. Court intrigue? You'd need to talk to someone who travels in those circles."
```

**Would Prevent (Bad)**:
```
"The king is actually a tyrant and..." [❌ Not in NPC knowledge]
"He's planning to invade the northern kingdom" [❌ Inventing plot]
Another patron leans in and whispers... [❌ New character not specified]
```

## Effectiveness Notes

### What Works Well
- **"Logical and realistic" preamble**: Sets baseline expectations
- **Explicit inventory listing**: "Unless mentioned in: X" is clearer than "only use what you have"
- **"Impossible actions must fail"**: Prevents physics-breaking stunts
- **"Do not invent" list**: Comprehensive prohibition list
- **"Say you don't know" clause**: Gives LLM acceptable fallback instead of hallucinating
- **Data-first structure**: Showing available data makes grounding explicit
- **Multiple complementary constraints**: Redundancy helps, especially on weaker models

### Known Limitations
- Creative models (Hathor, some Llama finetunes) may still push boundaries
- Very small models (< 7B) may struggle with long constraint lists
- Constraints must be included in EVERY prompt - LLMs don't "remember" rules
- Edge cases may need specific prohibitions added
- Some models interpret "realistic" differently

### Model-Specific Notes
- **GPT-4**: Excellent constraint following, rarely hallucinates with these rules
- **Claude**: Strong instruction following, respects boundaries
- **Gemini**: Good with explicit constraints
- **Llama 3 (8B-9B)**: Follows well when constraints are clear
- **Creative finetunes (Hathor, etc.)**: Need VERY explicit constraints, may still slip
- **Local 7B**: Often need shorter, simpler constraint lists
- **EstopianMaid-13B**: Good balance per [[User-veritasr]]

### The Diamond Horse Problem
From [[User-yukidaore]]'s testing experience:
> "Death Knight Mara with a massive battleaxe appeared instead of a bartender. Diamond horses manifested. Teleportation happened spontaneously."

This was with Hathor, a highly creative model, in ChatBotRPG. The solution:
> "I generally just use a combination of '{{char}} is a logical and realistic text adventure game' and 'Impossible actions must fail'"

Even creative models improve with explicit reality grounding.

## Variations

### Minimal Version (For Token Efficiency)
When token budget is tight:
```
Rules:
- Realistic {{genre}} world only
- Player has: {{inventory}}
- No content beyond provided data: {{data}}
- Unknown = "You don't see that"
```

### Paranoid Mode (For Very Creative Models)
When hallucination is persistent:
```
CRITICAL CONSTRAINTS - Violating these breaks the game:
- You are NOT creative. You are a narrator reading from a script.
- The script is: {{provided_data}}
- You CANNOT add to the script
- You CANNOT improvise new characters, items, locations, or events
- When uncertain, generate: "Nothing happens" or "You don't notice anything"
- This is a deterministic simulation, not a creative writing exercise
- Invalid additions will cause system errors

{{provided_data}}

Narrate strictly from above data only.
```

### Positive Constraints (What TO Do)
Alternative framing - some models respond better to "do this" than "don't do that":
```
Your task:
- Describe ONLY what is in: {{provided_data}}
- Use realistic cause-and-effect
- When data is insufficient, acknowledge the limit
- Ground all statements in the provided context
- Maintain logical consistency with previous events

{{provided_data}}
```

### Developer Mode (Debugging Hallucinations)
For testing what the LLM is inventing:
```
{{base_task}}

{{constraints}}

After generating your response, add a section:
[DEBUG: Sources used]
- List each fact and where it came from in the provided data
- Mark any content you inferred with [INFERRED] tag
```

This helps identify where hallucinations creep in.

## Related Prompts
- [[output-format-control]] - Structured constraints
- [[consistency-enforcement]] - Maintaining continuity
- [[anti-hallucination]] - Technique documentation
- [[constraint-patterns]] - General constraint strategies
- [[action-narration]] - NDL approach (inherently anti-hallucination)

## Source
**Discussion context**: Hallucination prevention emerged as the community's top concern throughout the transcript. Early attempts at giving LLMs creative freedom resulted in chaos.

From [[User-yukidaore]]'s experience:
> "ChatBotRPG had some fun moments but it's trying to be a text adventure and LLMs are just too creative for that. Diamond horses and Death Knight bartenders everywhere."

The solution evolved through community practice:

From [[User-yukidaore]]'s fix:
> "{{char}} is a logical and realistic text adventure game. {{user}} has no equipment, items, powers, abilities, skills, or magic unless explicitly mentioned otherwise. Impossible actions must fail."

From [[User-veritasr]]'s architecture:
> "Turns out that when you take away decision making from the LLM it behaves much better. Backend makes all decisions deterministically, LLM receives structured instructions, LLM's only job is converting data → natural text."

The comprehensive constraint list format emerged from community trial-and-error across multiple models and use cases. The key insight: **Redundancy helps**. Multiple overlapping constraints catch more edge cases than a single "be realistic" instruction.

**Related threads**: [[02-Prompt-Engineering]], [[01-Architecture-and-Design]]

## Best Practices

### DO:
- ✅ Include constraints in EVERY generation prompt
- ✅ List available data explicitly
- ✅ Provide fallback phrases ("You don't know/see")
- ✅ Be specific about what NOT to add
- ✅ Ground genre/realism expectations upfront
- ✅ Test with creative models to find edge cases
- ✅ Add specific prohibitions as you discover issues

### DON'T:
- ❌ Assume the LLM "remembers" constraints from earlier
- ❌ Rely on single constraint - use multiple overlapping ones
- ❌ Use vague terms like "stay realistic" without defining realistic
- ❌ Skip inventory/capability listing
- ❌ Let LLM improvise when data is missing
- ❌ Trust that "smart" models won't hallucinate
- ❌ Forget that creative finetunes need extra constraints

## Testing Checklist

Use this to test your hallucination prevention:

- [ ] LLM doesn't add items to inventory
- [ ] LLM doesn't introduce new NPCs
- [ ] LLM doesn't create plot events
- [ ] LLM doesn't give player abilities they lack
- [ ] LLM doesn't invent locations
- [ ] LLM doesn't assume success/failure beyond provided results
- [ ] LLM admits lack of knowledge when appropriate
- [ ] LLM respects physics/realism of genre
- [ ] LLM doesn't add dialogue for characters not present
- [ ] LLM doesn't elaborate beyond provided details
