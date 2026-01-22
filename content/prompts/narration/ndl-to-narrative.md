---
tags: [prompt, narration, ndl, primary-technique]
category: narration
technique: ndl
model-tested: [Gemma2-9B, Llama3-8B, Mistral-7B, EstopianMaid-13B, Stheno, GPT-4]
contributor: veritasr
status: production-ready
---

# Prompt: NDL to Narrative Translation

## Metadata
- **Category**: Narration
- **Technique**: NDL (Natural Description Language)
- **Model Tested**: Gemma 2 9B, Llama 3 8B, Mistral 7B, EstopianMaid 13B, Stheno, GPT-4
- **Contributor**: [[User-veritasr]]
- **Status**: Production-ready (PRIMARY TECHNIQUE)

## Purpose

Convert structured NDL (Natural Description Language) markup into natural narrative prose. NDL provides a "play-by-play" of what happened programmatically, and the LLM's job is to narrate it engagingly without making any game logic decisions.

This is the core technique used in ReallmCraft and represents the community's consensus approach to reliable LLM narration.

## Template

```
{{SYSTEM_PROMPT}}

Setting: {{LOCATION_NAME}}
Time: {{TIME_OF_DAY}}
Weather: {{WEATHER_CONDITIONS}}

Characters Present:
{{#each NPCS}}
- {{name}}: {{current_state}}
{{/each}}

Events occurring:
{{NDL_ACTION_SEQUENCE}}

Narrate these events naturally, focusing on {{NARRATIVE_FOCUS}}. Write 2-4 sentences.
```

## Variables

| Variable | Description | Example |
|----------|-------------|---------|
| {{SYSTEM_PROMPT}} | Base instructions for narration engine | "You are a narrative engine for a fantasy game..." |
| {{LOCATION_NAME}} | Current scene location | "Rusty Tankard Tavern" |
| {{TIME_OF_DAY}} | Current time | "Evening" |
| {{WEATHER_CONDITIONS}} | Weather if relevant | "Light rain outside" |
| {{NPCS}} | Array of NPCs in scene | [{name: "Bartender", state: "wiping glasses"}] |
| {{NDL_ACTION_SEQUENCE}} | The NDL markup to narrate | See NDL syntax below |
| {{NARRATIVE_FOCUS}} | Optional focus area | "tension and atmosphere" |

## NDL Syntax Reference

### Basic Action
```
do($"character_name", "action_description")
```

### Action with Manner (How)
```
do($"character", "action")~"method"
```

### Full Action with Intention
```
do($"character", "action")~"method"->intention="why"
```

### Sequential Actions
```
do($"player", "sneak")~"quietly"->target($"guard")->check("stealth", result="success")
```

### Common Patterns
- `->target($"entity")` - Specify target of action
- `->check("skill", result="success/failure")` - Dice roll outcome
- `->result("outcome")` - Final outcome state
- `wait("condition")` - Pause or conditional
- `->` - Sequence operator

## Usage Example

### Input (NDL)
```
do($"player", "sneak")~"carefully"
->target($"guard")
->check("stealth", result="success")
->move_to($"treasure_room")
->remain("undetected")
```

### Full Prompt
```
You are a narrative engine for a fantasy RPG. Your job is to convert structured event descriptions into natural, engaging prose. Do not invent new events or outcomes - only narrate what is described.

Setting: Castle Hallway
Time: Midnight
Weather: Clear, moonlight through windows

Characters Present:
- Player: Attempting stealth
- Guard: On patrol, unaware

Events occurring:
do($"player", "sneak")~"carefully"
->target($"guard")
->check("stealth", result="success")
->move_to($"treasure_room")
->remain("undetected")

Narrate these events naturally, focusing on tension and stealth. Write 2-4 sentences.
```

### Expected Output
```
You press yourself against the cold stone wall, moving with practiced silence. The guard shifts his weight but doesn't turn as you slip past his position. Heart pounding, you reach the treasure room door undetected.
```

## System Prompt Template

```
You are a narrative engine for a {{GAME_GENRE}} game. Your role is to convert structured event descriptions (written in NDL format) into natural, engaging prose.

Important rules:
1. ONLY narrate events explicitly described in the NDL
2. DO NOT invent new events, outcomes, or character actions
3. DO NOT change the outcome of dice rolls or checks
4. DO NOT add information not present in the event description
5. Focus on vivid sensory details within the described events
6. Maintain a {{TONE}} tone
7. Keep responses concise (2-4 sentences unless specified otherwise)

The backend has already determined what happens. Your job is to make it read well.
```

## Effectiveness Notes

### What Works Well
- ✅ Eliminates hallucinations about game state (LLM can't invent outcomes)
- ✅ Works reliably on small models (7B-9B parameters)
- ✅ Consistent output across different models
- ✅ Separates decision-making from narration cleanly
- ✅ Easy to debug (NDL is human-readable)
- ✅ Modding-friendly (NDL syntax is learnable)

### Known Limitations
- Requires backend to generate NDL (additional dev work)
- Writing quality depends on model choice
- Complex social interactions need extensive NDL vocabulary (43+ actions for social alone)
- Backreferences in conversation can be tricky
- Need to establish NDL vocabulary for your game's needs

### Community Feedback

**veritasr** (creator): "once you offload the decision making to a system that's actually capable of making sane decisions the hallucinations functionally disappear."

**veritasr**: "Achieved consistent subtext in narrative. Controlled emotional beats. Reliable dialogue with implicit intent. Works across multiple models (Gemma, Llama3, Mistral, Stheno)."

**Model-specific observations**:
- **Gemma 2**: Works but "meh" writing style
- **Llama 3**: Clean, accurate narration
- **EstopianMaid 13B**: Good balance, veritasr's primary model
- **GPT-4**: Overkill but works excellently
- **7B-9B models**: All tested models work adequately

### Token Efficiency
Context windows typically stay under 3k tokens with NDL + RAG system.

## Variations

### Combat Narration
```
do($"player", "attack")~"sword"
->target($"goblin")
->roll("hit", result="success")
->damage(8)
->target_state("heavily wounded")
```

Output: "You swing your sword in a wide arc, catching the goblin off-guard. Your blade bites deep into its side, dealing a grievous wound."

### Social Interaction
```
do($"player", "persuade")~"charming words"
->target($"merchant")
->intention="get discount"
->roll("charisma", result="failure")
->merchant_responds("firmly refuses")
```

Output: "You attempt to sweet-talk the merchant into lowering his prices, but he's heard it all before. 'These are fair prices,' he says firmly, crossing his arms."

### Multi-Step Sequence
```
do($"player", "examine")~"carefully"
->target($"bookshelf")
->find($"secret_lever")
->pull($"secret_lever")
->trigger("hidden_door_opens")
```

Output: "You run your fingers along the bookshelf, searching for irregularities. Behind a worn tome, you discover a hidden lever. As you pull it, a section of wall grinds open to reveal a dark passage."

## Temperature & Sampling Settings

**Recommended for NDL Narration**:
- Temperature: 0.6 - 0.9
- Top-p: 0.9 - 0.95
- Repetition Penalty: 1.15 - 1.3

Balance creativity with staying on task.

## Related Techniques
- [[few-shot-examples]] - Teaching LLM the NDL syntax
- [[dynamic-prompt-building]] - Programmatic prompt construction
- [[chain-of-thought]] - Can combine with NDL for complex narration

## Source
- Discussion context: June 2024 NDL formalization by veritasr
- Proven in: ReallmCraft production implementation
- Referenced in: [[08-NDL-Natural-Description-Language]]
