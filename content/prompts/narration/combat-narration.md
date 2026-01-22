# Prompt: Combat Narration

#prompt #combat #narration #turn-based

## Metadata
- **Category**: Narration
- **Technique**: Comma-separated constraint list
- **Model Tested**: GPT-3.5, GPT-4, various local models
- **Contributor**: [[User-appl2613]]
- **Status**: Proven (ChatBot RPG production)

## Purpose
Generate turn-based combat narration for text RPGs. The prompt constrains the LLM to produce single, grounded combat actions with rich sensory detail while preventing common problems like theatrics, acrobatics, multi-move sequences, and assuming outcomes.

## Template
```
Third-person RPG, turn-based combat, your turn, {{constraints}}, your character {{character_name}} is fighting {{opponent_list}}. Your character is holding: {{equipped_items}}, follow ALL SPECIAL INSTRUCTIONS when considering your move, {{movement_constraints}}

Constraints (comma-separated):
- one action per arm and leg only
- do NOT post two or more consecutive moves
- post damage if hit
- avoid dialogue
- avoid theatrics
- avoid acrobatics
- maintain firm footing
- describe single open-ended action
- attack, defense, or damage
- rich prose, sensory detail
- don't assume outcome
- one move only
- don't describe other characters

Special Instructions:
{{dynamic_combat_context}}
```

## Variables
| Variable | Description | Example |
|----------|-------------|---------|
| {{character_name}} | Fighter's name | "Theron", "Player" |
| {{opponent_list}} | Who they're fighting | "bandit chief", "two goblins and an orc" |
| {{equipped_items}} | What's in their hands | "longsword (right), shield (left)", "dagger (right), empty (left)" |
| {{constraints}} | Core constraint list | See template above |
| {{movement_constraints}} | Position/distance rules | "you can only move forwards or backwards in one turn - not both" |
| {{dynamic_combat_context}} | Situational modifiers | See "Special Instructions" section below |

## Usage Example

### Example 1: Melee Combat at Medium Range
```
Third-person RPG, turn-based combat, your turn, one action per arm and leg only, do NOT post two or more consecutive moves, post damage if hit, avoid dialogue, avoid theatrics, avoid acrobatics, maintain firm footing, describe single open-ended action, attack, defense, or damage, rich prose, sensory detail, don't assume outcome, one move only, don't describe other characters. Your character Theron is fighting scarred bandit. Your character is holding: longsword (right hand), shield (left hand), follow ALL SPECIAL INSTRUCTIONS when considering your move, you can only move forwards or backwards in one turn - not both.

#Extra Note: Your opponent is close enough to hit with your weapon, although they can also hit you! Be cautious about advancing any closer unless the opponent leaves themselves open.#
```

**Expected Output**:
```
Theron raises his shield to cover his left side and steps forward, driving his longsword toward the bandit's unprotected flank. The blade cuts through the air with a whistle, aimed at the gap between the enemy's ribs and hip.
```

### Example 2: Combat at Close Distance
```
Third-person RPG, turn-based combat, your turn, one action per arm and leg only, do NOT post two or more consecutive moves, post damage if hit, avoid dialogue, avoid theatrics, avoid acrobatics, maintain firm footing, describe single open-ended action, attack, defense, or damage, rich prose, sensory detail, don't assume outcome, one move only, don't describe other characters. Your character Lyra is fighting armored knight. Your character is holding: dagger (right hand), empty (left hand), follow ALL SPECIAL INSTRUCTIONS when considering your move, you can only move forwards or backwards in one turn - not both.

#Extra Note: You are very close, almost flush with your opponent. Longer weapons may become awkward. Striking with weapon hilts, lower ends near the hands, striking with hands or feet, or grappling are all viable options, but be careful not to leave yourself open for the opponent to do the same!#
```

**Expected Output**:
```
At this distance, Lyra can feel the knight's breath through the helmet visor. She drives her left palm against the inside of the knight's sword arm, shoving it away, while her dagger hand snakes low toward the gap between breastplate and tasset.
```

### Example 3: Long Range (Too Far)
```
Third-person RPG, turn-based combat, your turn, one action per arm and leg only, do NOT post two or more consecutive moves, post damage if hit, avoid dialogue, avoid theatrics, avoid acrobatics, maintain firm footing, describe single open-ended action, attack, defense, or damage, rich prose, sensory detail, don't assume outcome, one move only, don't describe other characters. Your character Marcus is fighting archer. Your character is holding: battleaxe (both hands), follow ALL SPECIAL INSTRUCTIONS when considering your move, you can only move forwards or backwards in one turn - not both.

#Extra Note: Your opponent is TOO FAR AWAY to engage right now unless it is a ranged attack. You must advance to close the distance before you can attack. Step forward cautiously with your guard up and in front of you!#
```

**Expected Output**:
```
Marcus keeps his battleaxe raised before him, using the haft as a mobile barrier as he advances across the clearing. Each step forward is measured, weight shifting smoothly to maintain balance, eyes locked on the archer's movements.
```

## Special Instructions (Dynamic Context)

### Proximity-Based Instructions
From [[User-appl2613]]'s implementation in ChatBot RPG:

```python
if int(proximity_value) >= 3:
    postfix_instructions += f"\n#Extra Note: Your opponent is TOO FAR AWAY to engage right now unless it is a ranged attack. You must advance to close the distance before you can attack. Step forward cautiously with your guard up and in front of you!#"

if int(proximity_value) == 2:
    postfix_instructions += f"\n#Extra Note: Your opponent is close enough to hit with your weapon, although they can also hit you! Be cautious about advancing any closer unless the opponent leaves themselves open.#"

if int(proximity_value) == 1:
    postfix_instructions += f"\n#Extra Note: Your opponent is now close enough to hit with your weapon, although they can also hit you! Be cautious about advancing any closer unless the opponent leaves themselves open.#"

if int(proximity_value) == 0:
    postfix_instructions += f"\n#Extra Note: You are very close, almost flush with your opponent. Longer weapons may become awkward. Striking with weapon hilts, lower ends near the hands, striking with hands or feet, or grappling are all viable options, but be careful not to leave yourself open for the opponent to do the same!#"
```

### Weapon-Specific Instructions
```
#Weapon Note: Your {{weapon_type}} has {{range}} reach. {{special_properties}}.#
```

Examples:
- Longsword: "Your longsword has medium reach. It's versatile for both attack and defense."
- Dagger: "Your dagger has short reach. It excels at close quarters but requires you to get very close."
- Polearm: "Your polearm has long reach. It's most effective at medium distance but becomes awkward if the opponent closes in."

### Status Effect Instructions
```
#Status Note: You are {{status_effect}}. {{effect_description}}.#
```

Examples:
- "You are wounded (left arm). Avoid actions that require full use of your left arm."
- "You are fatigued. Your movements are slower and less precise than usual."
- "You are enraged. You fight with reckless aggression but reduced defensive awareness."

## Effectiveness Notes

### What Works Well
- **Comma-separated format**: Saves tokens, models parse it well (likely from CSV training data)
- **Explicit "don't" list**: Prevents theatrical flourishes, backflips, multi-move sequences
- **"One move only" repetition**: Reinforced constraint prevents LLM from narrating entire exchanges
- **"Don't assume outcome"**: Keeps narration grounded - "aims at" not "strikes through"
- **Proximity system**: Dynamic instructions based on combat range create tactical considerations
- **Rich sensory detail**: Balances constraint with engagement
- **Works in production**: Proven in ChatBot RPG turn-based combat

### Known Limitations
- May still produce slightly theatrical language on creative models
- Small models (< 7B) may occasionally slip in extra moves
- "Avoid dialogue" doesn't always prevent internal thoughts
- Can become repetitive over long combats (needs narrative variety injection)
- Assumes turn-based structure; not suitable for real-time combat simulation

### Model-Specific Notes
- **GPT-4**: Excellent adherence, but overkill for this task
- **GPT-3.5**: Good balance of cost and quality for combat
- **Local 7B-9B**: Usually works but needs stricter repetition of constraints
- **Creative models**: May fight against "avoid theatrics" - consider different model

### Community Feedback
From [[User-appl2613]]:
> "i used to have this long, flowery paragraph, you know the type, where it's basically an accumulation of you trying different ways to urge the model and it grows and grows into this big, specific thing... for my generic combat prompt
>
> but it's since evolved (become simpler, more concise, comma-list basically) to this: [the comma-separated format]
>
> this seems to work just as well, if not better, than writing all that stuff out more naturally and it saves on tokens"

From [[User-monkeyrithms]]:
> "One-shotting combat doesn't work too well unless it's GPT-4 in my experience, but a CoT type thing does"

**Recommendation**: Use this prompt iteratively, one turn at a time, rather than trying to narrate entire combat sequences.

## Variations

### Ranged Combat Variant
```
Add to constraints:
- ranged attack
- describe aiming and release
- account for distance
- environmental factors (wind, cover)
```

Remove movement constraints about closing distance.

### Magic Combat Variant
```
Add to constraints:
- casting time and gestures
- mana cost acknowledged
- spell effects grounded
- avoid deus ex machina outcomes
```

### Defensive Turn Variant
When player chooses to defend:
```
Focus on:
- defensive positioning
- reading opponent's movements
- preparing counter-opportunities
- maintaining guard
```

### Critical Hit/Miss Variant
Inject outcome if backend determines it:
```
#Critical Result: {{critical_type}}. Your action {{succeeds_spectacularly / fails_catastrophically}}.#
```

## Integration with Game Systems

### Backend Workflow
1. **Player declares action** (e.g., "I attack with my sword")
2. **Backend calculates**:
   - Proximity value
   - Hit/miss roll
   - Damage calculation
   - Status effects
3. **Generate combat prompt** with dynamic instructions
4. **LLM narrates** the execution (not the outcome determination)
5. **Backend applies results** to game state
6. **Repeat for next turn**

### Post-Processing
After LLM generation:
- Check for multiple actions (red flag)
- Verify no outcome assumptions
- Ensure one-sentence-per-action structure
- May need regeneration if constraints violated

## Related Prompts
- [[action-narration]] - General action description
- [[status-narration]] - Health/condition updates after combat
- [[chain-of-thought]] - Multi-step reasoning for complex actions
- [[hallucination-prevention]] - Constraint techniques
- [[narration-engine-system]] - Overall system prompt integration

## Source
**Discussion context**: This combat prompt evolved through [[User-appl2613]]'s development of ChatBot RPG, a turn-based text adventure with LLM-narrated combat. The key insight was that verbose, paragraph-style constraints were less effective and more expensive than comma-separated lists.

From the transcript (November 2025):
> "i used to have this long, flowery paragraph... but it's since evolved (become simpler, more concise, comma-list basically)... this seems to work just as well, if not better, than writing all that stuff out more naturally and it saves on tokens" - [[User-appl2613]]

The proximity-based dynamic instructions came from testing where players would advance/retreat strategically. Without distance management, combats became static or unrealistic.

The "one move only, don't post two or more consecutive moves" constraint addresses a common LLM tendency to narrate back-and-forth exchanges:
❌ "He swings, you dodge, he swings again, you parry, then you counter-strike..."
✓ "He swings his blade in a wide arc toward your head."

**Related threads**: [[02-Prompt-Engineering]], [[01-Architecture-and-Design]]

## Testing Checklist
- [ ] Single action only (no sequences)
- [ ] No assumed outcomes ("aims at" not "pierces")
- [ ] No theatrics (backflips, dramatic poses)
- [ ] Sensory details present
- [ ] Respects proximity constraints
- [ ] Equipped items mentioned when relevant
- [ ] No dialogue from character
- [ ] Grounded movement (no acrobatics)
- [ ] Doesn't describe opponent's action
- [ ] Rich prose without being flowery
