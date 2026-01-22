# Prompt: Action Narration

#prompt #narration #combat #ndl

## Metadata
- **Category**: Narration
- **Technique**: NDL-based structured narration
- **Model Tested**: Gemma 2, Llama 3, Mistral 7B, Stheno, EstopianMaid-13B
- **Contributor**: [[User-veritasr]]
- **Status**: Proven

## Purpose
Convert structured action data (NDL format) into natural narrative descriptions. The backend has already determined what happens and the outcome - the LLM only narrates the execution. This eliminates LLM decision-making and hallucination.

## Template
```
Convert the following action data into natural narrative. Narrate what happens based on the provided information only.

Action Data:
action({{action_verb}})
~ manner({{how_performed}})
intention({{why_doing_it}})
by(character({{actor_name}}))
target({{target}})
roll({{skill_type}}, result={{success_or_failure}})
location({{current_location}})
context({{situational_context}})

Generate a {{length}} narration that:
1. Describes the action being attempted
2. Shows the manner/method (the "how")
3. Conveys the outcome (success/failure) through description, not statements
4. Maintains {{tone}} tone
5. Is {{sentence_count}} sentences

Do not:
- Add actions not in the data
- Change the outcome
- Invent new consequences
- Add dialogue unless specified
- Make the character do anything beyond the provided action
```

## Variables
| Variable | Description | Example |
|----------|-------------|---------|
| {{action_verb}} | What is being done | "attack", "sneak", "persuade", "cast_spell" |
| {{how_performed}} | Method/style | "quietly", "aggressively", "desperately", "carefully" |
| {{why_doing_it}} | Character's intention | "to avoid detection", "to intimidate", "to help" |
| {{actor_name}} | Who is doing it | "$player", "Guard Captain", "$npc_thief" |
| {{target}} | What/who it's done to | "locked door", "bandit", "merchant" |
| {{skill_type}} | What was rolled | "stealth", "strength", "diplomacy" |
| {{success_or_failure}} | Outcome | "$success", "$failure", "$critical_success" |
| {{current_location}} | Where this happens | "dimly lit hallway", "crowded marketplace" |
| {{situational_context}} | Relevant circumstances | "guards nearby", "weapon drawn", "exhausted" |
| {{length}} | Output size | "brief", "moderate", "detailed" |
| {{tone}} | Narrative style | "tense", "heroic", "matter-of-fact" |
| {{sentence_count}} | How many sentences | "2-3", "3-4", "4-5" |

## Usage Example

### Example 1: Stealth Success
```
Convert the following action data into natural narrative. Narrate what happens based on the provided information only.

Action Data:
action(sneak)
~ manner(quietly)
intention(to bypass guard)
by(character($player))
target(guard(castle_guard_01))
roll(stealth, result=$success)
location(dimly lit stone corridor)
context(guard facing away, sound of footsteps echoing from other hallways)

Generate a brief narration that:
1. Describes the action being attempted
2. Shows the manner/method (the "how")
3. Conveys the outcome (success/failure) through description, not statements
4. Maintains tense tone
5. Is 2-3 sentences

Do not:
- Add actions not in the data
- Change the outcome
- Invent new consequences
- Add dialogue unless specified
- Make the character do anything beyond the provided action
```

**Expected Output**:
```
You press your back against the cold stone wall, timing your movements with the echo of distant footsteps to mask any sound. The guard remains facing away, attention fixed on something down the opposite corridor. You slip past him, close enough to hear his breathing, and continue deeper into the castle undetected.
```

### Example 2: Attack Failure
```
Action Data:
action(attack)
~ manner(desperately)
intention(to defend self)
by(character($player))
target(bandit(scarred_raider))
roll(combat, result=$failure)
location(muddy forest path)
context(outnumbered, weapon slippery from rain)

Generate a moderate narration that:
1. Describes the action being attempted
2. Shows the manner/method (the "how")
3. Conveys the outcome (success/failure) through description, not statements
4. Maintains gritty tone
5. Is 3-4 sentences
```

**Expected Output**:
```
You swing your blade at the scarred raider, desperation lending speed but robbing precision. The weapon's grip, slick with rain, turns slightly in your hand as you strike. Your opponent sidesteps effortlessly, and your momentum carries you stumbling forward into the mud. The raider's laugh joins those of his companions as they close the circle tighter.
```

### Example 3: Spell Casting (Critical Success)
```
Action Data:
action(cast_spell)
~ manner(confidently)
intention(to halt advancing enemies)
by(character($player))
target(enemies(goblin_raiders))
roll(magic, result=$critical_success)
location(crumbling ruins entrance)
context(goblins charging, allies behind player needing time)

Generate a detailed narration that:
1. Describes the action being attempted
2. Shows the manner/method (the "how")
3. Conveys the outcome (success/failure) through description, not statements
4. Maintains heroic tone
5. Is 4-5 sentences
```

**Expected Output**:
```
You plant your feet at the ruins entrance and raise your hands, words of power flowing from your lips with practiced confidence. The air crackles with energy as arcane symbols materialize, spinning faster with each syllable. A wall of shimmering force erupts across the threshold, and the charging goblins slam into it with bone-jarring impact. Several bounce backward, stunned, while others scramble to find a way around the barrier that wasn't there a moment ago. Behind you, your allies use the precious seconds to regroup.
```

## Effectiveness Notes

### What Works Well
- **Eliminates hallucination**: Backend provides the outcome, LLM just describes it
- **Consistent outcomes**: Success always reads as success, failure as failure
- **Natural "show don't tell"**: "Sidesteps effortlessly" instead of "You miss"
- **Manner integration**: The "how" creates variety even with repeated actions
- **Works on small models**: 7B-9B models handle this reliably
- **Fast generation**: Typically 50-150 tokens, quick even on local models

### Known Limitations
- Requires NDL or similar structured format from backend
- Prose quality varies by model (Gemma serviceable, Llama 3 better)
- May need different templates for different action types (combat vs. social)
- Very small models (< 7B) sometimes slip in extra actions
- Critical successes/failures may need expanded context to feel appropriately impactful

### Model-Specific Notes
- **Llama 3 (7B-9B)**: Clean narration, rarely embellishes beyond data
- **Gemma 2 (9B)**: Follows structure well but prose is plain
- **EstopianMaid-13B**: [[User-veritasr]]'s main choice, good balance
- **Mistral 7B**: Tends toward slightly flowery language
- **GPT-4**: Excellent but overkill for this task
- **Claude**: Very good at "show don't tell", may elaborate slightly

### Combat-Specific Notes
From [[User-monkeyrithms]]:
> "One-shotting combat doesn't work too well unless it's GPT-4 in my experience, but a CoT type thing does"

For complex combat, use this prompt iteratively:
1. One action per narration pass
2. Update game state
3. Next action narration
4. Repeat until combat resolves

Don't try to narrate entire combat in one prompt.

## Variations

### Minimal Narration (Speed Mode)
For rapid action sequences:
```
Action: {{action_verb}} ~ {{manner}} → {{result}}
Narrate in ONE sentence, {{tone}} tone.
```

Output: "You slip past the guard unnoticed." or "Your blade glances off the raider's armor."

### Dialogue Integration
When action includes speech:
```
Action Data:
action({{action_verb}})
dialogue("{{what_they_say}}")
~ manner({{how_said}})
emotion({{emotional_state}})
by(character({{actor_name}}))
```

### Multi-Action Sequence
For connected actions (use carefully, can lead to hallucination):
```
Action Sequence:
1. action({{action1}}) → result({{result1}})
2. action({{action2}}) → result({{result2}})
3. action({{action3}}) → result({{result3}})

Narrate as a flowing sequence, {{sentence_count}} total sentences.
```

### Chain-of-Thought Variant
From [[User-50h100a]]'s technique:
```
Action Analysis:
- What {{actor_name}} is trying to do: {{action_verb}}
- How they're doing it: {{manner}}
- Why they're doing it: {{intention}}
- What happens: {{result}}

Now narrate:
```

This forces the LLM to "think through" the action before generating, can improve quality on weaker models.

## Related Prompts
- [[scene-description]] - Setting up the environment
- [[dialogue-generation]] - Character speech
- [[status-narration]] - Health/condition updates after actions
- [[chain-of-thought]] - CoT technique for reasoning
- [[ndl-reference-builder]] - Full NDL documentation

## Source
**Discussion context**: This prompt structure is the core of [[User-veritasr]]'s NDL (Natural Description Language) architecture. The fundamental insight came from failures with LLM-driven decision-making.

From the transcript:
> "Turns out that when you take away decision making from the LLM it behaves much better." - [[User-veritasr]]

The three-question framework (What, How, Why) is borrowed from tabletop GM practice:
> "With those three pieces of information the LLM has the context it needs to generate text in a pretty reliable manner. The backend just serves to answer those questions for non-player controlled agents, inject content and events, and determine failure chance." - [[User-veritasr]]

The NDL syntax evolved from January 2024 through July 2025, with the final architecture proven across multiple 7B-9B models. The result: consistent narration with sub-3k token context windows.

**Related threads**: [[02-Prompt-Engineering]], [[08-NDL-Natural-Description-Language]]
