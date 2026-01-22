# Prompt: Dialogue Generation with Subtext

#prompt #narration #dialogue #psychology #ndl

## Metadata
- **Category**: Narration
- **Technique**: Psychology-layered generation with implicit subtext
- **Model Tested**: Gemma 2, Llama 3, Mistral 7B, Stheno, EstopianMaid-13B
- **Contributor**: [[User-veritasr]]
- **Status**: Proven (advanced technique, July 2025)

## Purpose
Generate natural character dialogue with authentic subtext by separating what the character believes, knows, wants to convey, and actually says. This creates realistic conversations where characters don't explicitly state their thoughts, but their beliefs influence word choice and tone.

## Template
```
Generate dialogue for {{character_name}} in this situation.

Character Psychology:
character_believes({{core_belief_about_situation}})
character_knows({{factual_information_they_have}})
character_wants_to_convey({{message_they_intend}})
character_emotional_state({{current_emotion}})
character_relationship_to_listener({{relationship_type}})

Social Context:
location({{where_conversation_happens}})
others_present({{who_else_is_there}})
power_dynamic({{who_has_social_power}})
recent_events({{what_just_happened}})

Constraints:
- Character NEVER states their beliefs outright
- Information "comes to mind" influences word choice subtly
- What they WISH to convey ≠ what they DO convey
- Dialogue reflects emotional state through tone, not statements
- {{sentence_count}} lines of dialogue maximum
- Include brief manner/action tags: *manner* or (manner)

Generate dialogue only. No narration outside the dialogue.
```

## Variables
| Variable | Description | Example |
|----------|-------------|---------|
| {{character_name}} | Who is speaking | "Merchant Aldric", "Guard Captain Thorne" |
| {{core_belief_about_situation}} | What they think is really going on | "The player is hiding something", "This stranger is dangerous" |
| {{factual_information_they_have}} | What they know for certain | "Player recently arrived from the capital", "Stranger has no visible weapons" |
| {{message_they_intend}} | What they're trying to communicate | "I'm watching you", "I'm not a threat", "I need help but can't ask directly" |
| {{current_emotion}} | How they feel | "suspicious", "nervous", "confident", "afraid but hiding it" |
| {{relationship_type}} | Connection to listener | "stranger", "old friend", "subordinate", "rival" |
| {{where_conversation_happens}} | Setting | "busy marketplace", "private study", "roadside" |
| {{who_else_is_there}} | Other people nearby | "crowd of shoppers", "none, isolated", "the character's guards" |
| {{who_has_social_power}} | Power balance | "character has authority", "player has leverage", "equal standing" |
| {{what_just_happened}} | Recent context | "player just helped character", "character saw player with enemy", "nothing, first meeting" |
| {{sentence_count}} | How much dialogue | "2-3", "3-5", "5-7" |

## Usage Example

### Example 1: Suspicious Merchant
```
Generate dialogue for Merchant Aldric in this situation.

Character Psychology:
character_believes(The player is hiding something about why they're asking about the old ruins)
character_knows(Player recently arrived from the capital; most travelers avoid the ruins)
character_wants_to_convey(I'm watching you and I'm not fooled)
character_emotional_state(suspicious but maintaining friendly merchant facade)
character_relationship_to_listener(stranger, potential customer)

Social Context:
location(busy marketplace stall surrounded by other shoppers)
others_present(crowd of townspeople browsing nearby stalls)
power_dynamic(equal standing, but merchant has home advantage)
recent_events(player asked specific questions about ruins that locals don't talk about)

Constraints:
- Character NEVER states their beliefs outright
- Information "comes to mind" influences word choice subtly
- What they WISH to convey ≠ what they DO convey
- Dialogue reflects emotional state through tone, not statements
- 3-5 lines of dialogue maximum
- Include brief manner/action tags: *manner* or (manner)

Generate dialogue only. No narration outside the dialogue.
```

**Expected Output**:
```
*adjusts the wares on his stall, not quite meeting your eyes* "The old ruins, you say? Now that's an... interesting curiosity for someone fresh from the capital." *pauses, studying your face* "Most folk who come through here want to know about the harvest festival, the good hunting spots, the comfortable inns." *picks up a brass trinket, turning it over slowly* "But I suppose everyone has their reasons for asking about places best left alone. No judgment, of course." *the smile doesn't reach his eyes* "Anything else I can help you with today?"
```

### Example 2: Guard Trying Not to Show Fear
```
Generate dialogue for Guard Captain Thorne in this situation.

Character Psychology:
character_believes(The player is far more dangerous than they appear)
character_knows(Player single-handedly dealt with the bandit camp; rumors say without breaking a sweat)
character_wants_to_convey(I'm in control here and you must follow the law)
character_emotional_state(afraid but determined to maintain authority)
character_relationship_to_listener(official authority to civilian, but knows civilian is more powerful)

Social Context:
location(city gate checkpoint)
others_present(two junior guards watching their captain)
power_dynamic(guard has legal authority but player has physical superiority)
recent_events(player just arrived at gate; junior guards heard the rumors and look nervous)

Constraints:
- Character NEVER states their beliefs outright
- Information "comes to mind" influences word choice subtly
- What they WISH to convey ≠ what they DO convey
- Dialogue reflects emotional state through tone, not statements
- 2-3 lines of dialogue maximum
- Include brief manner/action tags: *manner* or (manner)

Generate dialogue only. No narration outside the dialogue.
```

**Expected Output**:
```
*hand resting on sword hilt, but fingers not quite gripping* "Standard procedure - all travelers state their business in the city." *clears throat* "I'm sure someone of your... capabilities... has legitimate reasons for visiting. The law applies to everyone, of course, but uh... we appreciate cooperation from responsible individuals."
```

### Example 3: NPC Seeking Help But Too Proud
```
Generate dialogue for Scholar Lyris in this situation.

Character Psychology:
character_believes(Asking for help is admitting weakness)
character_knows(Player has access to the royal library; the book she needs is there)
character_wants_to_convey(I need that book but I'm not begging)
character_emotional_state(desperate but masked with academic pride)
character_relationship_to_listener(peer scholars, but player has better connections)

Social Context:
location(university library reading room)
others_present(few other scholars studying quietly)
power_dynamic(equal intellectual standing, player has political connections)
recent_events(Lyris's research has stalled without the restricted book)

Constraints:
- Character NEVER states their beliefs outright
- Information "comes to mind" influences word choice subtly
- What they WISH to convey ≠ what they DO convey
- Dialogue reflects emotional state through tone, not statements
- 3-5 lines of dialogue maximum
- Include brief manner/action tags: *manner* or (manner)

Generate dialogue only. No narration outside the dialogue.
```

**Expected Output**:
```
*looks up from her notes, a bit too casually* "Oh, I didn't realize you had returned from the capital. I trust you had access to... adequate research materials?" *traces a finger along the margin of her page* "Some of us make do with the university's collection. Comprehensive for most purposes." *pause* "Though I've heard the royal library acquired a rare Telvanni treatise on planar resonance. Second era, if I recall. Not that it's essential to my work, merely... professionally interesting." *meets your eyes briefly* "I don't suppose you happened across it?"
```

## Effectiveness Notes

### What Works Well
- **Natural subtext**: Characters feel "real" because they don't say exactly what they mean
- **Emotion through manner**: Nervousness shows in pauses and fidgeting, not "I'm nervous"
- **Belief influences language**: Suspicious character uses careful, probing words
- **Works on small models**: Even 7B-9B models can follow this structure
- **Consistent personality**: Same character generates similar patterns across conversations
- **Action tags integrate naturally**: *manner* format works well across models

### Known Limitations
- Requires well-defined character psychology from backend
- More complex than simple dialogue generation
- Can be subtle to the point of vague on weaker models
- May need few-shot examples for very small models (< 7B)
- Action tags sometimes placed awkwardly by some models
- Long conversations may lose psychological thread (refresh context periodically)

### Model-Specific Notes
- **EstopianMaid-13B**: [[User-veritasr]]'s main model, handles subtext well
- **Llama 3 (8B-9B)**: Clean, understands the psychology structure
- **Gemma 2 (9B)**: Functional but subtext is less sophisticated
- **Mistral 7B**: Sometimes too flowery, can over-dramatize
- **GPT-4**: Excellent subtext, but overkill
- **Claude**: Very strong at psychological nuance

### Advanced Technique: Implicit Intent
From [[User-veritasr]]'s July 2025 breakthrough:
> "Achieved consistent subtext in narrative. Controlled emotional beats. Reliable dialogue with implicit intent. Works across multiple models (Gemma, Llama3, Mistral, Stheno)."

The key rules:
1. **Never state beliefs outright** - "He seems suspicious" ❌ vs. showing suspicious behavior ✓
2. **Information "comes to mind"** - What they know influences word choice unconsciously
3. **Intent ≠ Execution** - What they want to say ≠ what actually comes out
4. **Emotion = manner, not statement** - Show nervousness through behavior, not "I'm nervous"

## Variations

### Simple Dialogue (Without Subtext)
For straightforward exchanges:
```
Character {{character_name}} says to {{target}}:
Topic: {{topic}}
Emotion: {{emotion}}
Relationship: {{relationship}}

Generate 1-2 lines of dialogue with manner tags.
```

### Group Conversation
For multiple speakers:
```
Conversation between {{character_list}}:
Topic: {{topic}}
Each character wants to: {{their_goals}}
Social dynamics: {{power_relationships}}

Generate one line per character, showing their personality through what/how they speak.
```

### Revelation Dialogue
When character must reveal information:
```
Character Psychology:
character_knows({{secret_information}})
character_must_reveal_because({{reason_they_tell}})
character_feels_about_telling({{emotional_conflict}})

Generate dialogue where they reveal {{information}} but their manner shows {{emotion}}.
```

### Separation of Action and Dialogue
From [[User-irovos]]'s principle:
```
[Action]: {{what_character_does}}
[Dialogue]: {{what_character_says}}
[Manner]: {{how_they_say_it}}
```

Parse separately for programmatic use.

## Related Prompts
- [[action-narration]] - Physical action descriptions
- [[character-creation]] - Creating the psychology data this uses
- [[scene-description]] - Setting for dialogue
- [[ndl-reference-builder]] - Full NDL documentation
- [[consistency-enforcement]] - Keeping character voices stable

## Source
**Discussion context**: This advanced technique represents the culmination of [[User-veritasr]]'s NDL development, achieved in July 2025. The breakthrough came from applying tabletop RPG GM principles about NPC portrayal.

From the transcript:
> "Character belief layer: What the character believes influences their word choice. What they WISH to convey ≠ what they DO convey. Never state beliefs outright. Information that 'comes to mind' influences word choice." - [[User-veritasr]]

The three-layer structure (believes/knows/wants) creates separation between:
1. **Internal truth** (what they believe)
2. **Factual awareness** (what they know)
3. **Social presentation** (what they try to convey)

This separation is what creates authentic subtext - characters are trying to communicate one thing while their beliefs unconsciously influence how they do it.

> "Achieved consistent subtext in narrative. Controlled emotional beats. Reliable dialogue with implicit intent. Works across multiple models (Gemma, Llama3, Mistral, Stheno)." - [[User-veritasr]]

**Related threads**: [[02-Prompt-Engineering]], [[08-NDL-Natural-Description-Language]]

## Best Practices

### DO:
- ✅ Define clear character psychology before generating
- ✅ Separate belief from knowledge
- ✅ Show emotion through manner and word choice
- ✅ Use action tags for body language
- ✅ Let intent and execution differ naturally
- ✅ Include social context and power dynamics

### DON'T:
- ❌ Have characters state their thoughts directly
- ❌ Use "he thought to himself" narration
- ❌ Over-explain emotions ("I'm very angry!")
- ❌ Generate both dialogue and surrounding narration together
- ❌ Skip the belief/knowledge/intent separation
- ❌ Ignore the social context
