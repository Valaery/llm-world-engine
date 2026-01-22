# NDL Pattern: Dialogue Generation

#ndl #pattern #dialogue #conversation #social

## Overview

Dialogue generation in NDL handles character speech, conversation flow, and social interaction. By combining `talk()`, `convey()`, and social action constructs with entity references, NDL enables dynamic multi-character conversations while maintaining deterministic control over who speaks, what they're communicating, and the social context.

## Core Philosophy

Unlike traditional dialogue systems that use branching trees or pre-written lines, NDL describes the *intent and structure* of dialogue, letting the LLM generate actual words while the game controls the conversation flow.

### Separation of Concerns

1. **Game logic determines**:
   - Who is speaking to whom
   - What topic/information is being conveyed
   - Conversation flow and turn order
   - Emotional context and social dynamics

2. **NDL describes**:
   - Speaker and audience references
   - Message intent or perspective
   - Conversational manner (formal, casual, aggressive, etc.)

3. **LLM generates**:
   - Actual dialogue text
   - Character voice and personality
   - Appropriate language for context

## Basic Dialogue Patterns

### Simple Statement

```ndl
talk($"John", $"Mary") -> convey("quest information")
```

**Generated Narrative**:
> **John**: "Mary, I've learned something important about the artifact. The merchant told me it's hidden in the old temple ruins, guarded by ancient wards."

### Multi-Person Dialogue

```ndl
talk($"Mark", $"John"^$"Sue") -> convey("desire to explore dungeon", perspective="adventurous")
```

**Design Notes**:
- `^` operator (conjunction) specifies multiple listeners
- `perspective` modifier colors how the message is delivered
- All audience members receive same message in same turn

**Generated Narrative**:
> **Mark**: "Hey, what do you say we check out that dungeon everyone's been talking about? I heard there's treasure deeper down, and I'm itching for some action!"

### Conversation Exchange

```ndl
talk($"player", $"guard") -> convey("request to enter city")
-> wait("response")
-> talk($"guard", $"player") -> convey("denial", perspective="authoritative")
-> wait("player decision")
```

**Generated Narrative**:
> **You**: "I need to enter the city. I have urgent business with the council."
>
> **Guard**: "No one passes without proper papers. Council's orders. Move along."

## Advanced Dialogue Patterns

### Persuasion Attempt

```ndl
talk($"player", $"merchant") -> convey("lower price request", perspective="persuasive")
-> roll("persuasion") -> result("success")
-> talk($"merchant", $"player") -> convey("agreement", perspective="reluctant")
```

**Generated Narrative**:
> **You**: "Come on, I'm a regular customer. Surely you can give me a better deal on this?"
>
> *[Roll: Persuasion - Success]*
>
> **Merchant**: "Alright, alright. I suppose I can knock off a few coins. But don't go telling everyone, or I'll lose business."

### Failed Social Interaction

```ndl
talk($"player", $"noble") -> convey("introduce self", perspective="casual")
-> result("offended")
-> talk($"noble", $"player") -> convey("dismissal", perspective="insulted")
```

**Generated Narrative**:
> **You**: "Hey there! Name's Jack. What's yours?"
>
> **Noble**: "How dare you address me so informally! I am Lord Blackwood, and you will show proper respect or remove yourself from my presence immediately!"

### Group Conversation

```ndl
talk($"narrator", $"party") -> convey("situation description")
-> talk($"wizard", $"party") -> convey("analysis", perspective="intellectual")
-> talk($"warrior", $"party") -> convey("action plan", perspective="direct")
-> talk($"rogue", $"party") -> convey("alternative approach", perspective="sneaky")
```

**Pattern**: Round-robin discussion where each character contributes their perspective on the same topic.

## Dialogue with Emotional Context

### Manner Modifiers in Dialogue

```ndl
talk($"player", $"enemy") ~ "threateningly" -> convey("demand for surrender")
```

```ndl
talk($"npc", $"player") ~ "whispering" -> convey("secret information")
```

```ndl
talk($"lover", $"player") ~ "tenderly" -> convey("affection")
```

**Pattern**: The `~` modifier affects *how* something is said, influencing tone, volume, and delivery style.

### Intention in Dialogue

```ndl
talk($"player", $"suspect") -> convey("casual question", intention="gather information without raising suspicion")
```

**Generated Narrative**:
> **You**: "So, you've been living in town long? I'm new here myself." *(You're trying to get them talking without seeming suspicious)*

### Combined Manner and Intention

```ndl
talk($"diplomat", $"warlord") ~ "formally" -> convey("peace proposal", intention="prevent war while showing strength")
```

**Generated Narrative**:
> **Diplomat**: "My lord, we come bearing terms from the High Council. We seek peace between our nations, but make no mistake—we are prepared to defend our borders if necessary. Let us find a path that honors both our peoples."

## Conversation Flow Control

### Interruption

```ndl
talk($"villain", $"player") -> convey("monologue about plan")
-> do($"player", "interrupt") ~ "suddenly"
-> talk($"player", $"villain") -> convey("challenge", perspective="defiant")
```

### Silence / No Response

```ndl
talk($"player", $"stranger") -> convey("greeting")
-> wait("response")
-> result("no response")
-> describe("stranger ignores you")
```

### Overheard Conversation

```ndl
talk($"conspirator1", $"conspirator2") -> convey("plot details")
-> system_response("player is listening from hiding")
-> describe("player learns information")
```

**Pattern**: Speaker and audience don't include player, but player entity gains knowledge.

## Quest Dialogue Integration

### Quest Stage-Dependent Dialogue

From [[User-monkeyrithms]] implementation:

```ndl
# Quest not started
talk($"questgiver", $"player") -> convey("quest offer", perspective="concerned")

# Quest in progress
talk($"questgiver", $"player") -> convey("encouragement", perspective="hopeful")

# Quest completed
talk($"questgiver", $"player") -> convey("gratitude and reward", perspective="relieved")
```

**Implementation Note**:
> "I drew most of my game design inspiration from Elder Scrolls games, which use a 'quest stage' variable that updates according to pre-existing conditions, and it changes the dialogue prompts that NPCs are given throughout the game. It works really well when I combined it with LLM+function call."

### Information Gating

```ndl
talk($"player", $"innkeeper") -> convey("ask about local rumors")
-> search($"innkeeper knowledge base", intention="find rumor about dragon")
-> result("rumor found")
-> talk($"innkeeper", $"player") -> convey("dragon rumor", perspective="gossipy")
```

## Auto-Chat / Background Conversations

### NPC-to-NPC Dialogue (Player Observing)

From [[User-monkeyrithms]] implementation:

```ndl
system_response("auto-chat mode activated")
-> talk($"npc1", $"npc2") -> convey("philosophy discussion")
-> wait("response")
-> talk($"npc2", $"npc1") -> convey("counter-argument")
-> wait("response")
-> talk($"npc1", $"npc2") -> convey("agreement", perspective="reluctant")
```

**Design Notes**:
> "the fact that you could tell two LLMs to talk to each other about philosophy and they actually will, for as long as you want, without a script, is kind of a cool narrative device (although they do tend to go off the rails after a while)"

**Prompting Tip**:
> "right now they are just told to talk about stuff and they decided to talk about markets. I probably need to include phrases like don't start with the other character's name, because that got repetitive"

### Scripted NPC Events

```ndl
# Player observes two NPCs talking
talk($"guard", $"merchant") -> convey("inspection notice")
-> wait("merchant response")
-> talk($"merchant", $"guard") -> convey("complaint", perspective="annoyed")
-> wait("conversation concludes")
-> system_response("player learned about increased patrols")
```

**Use Case**: Ambient worldbuilding - player overhears conversations that reveal setting details or quest hooks.

## Social Actions Beyond Talk

### Non-Verbal Communication

```ndl
do($"player", "gesture") ~ "pointing at map" -> target($"guide")
-> convey("indicate location")
```

```ndl
do($"npc", "signal") ~ "hand sign" -> target($"ally")
-> convey("danger warning")
```

### Emotive Actions

```ndl
do($"character", "laugh") ~ "mockingly" -> target($"player")
-> convey("derision")
```

```ndl
do($"player", "nod") ~ "respectfully" -> target($"elder")
-> convey("understanding and respect")
```

## Conversation State Management

### Tracking Conversation Topics

```python
def generate_dialogue_ndl(speaker: Entity, listener: Entity,
                         conversation_state: ConversationState) -> str:
    """Generate NDL for dialogue based on conversation state."""

    # Determine topic based on relationship and quest states
    if conversation_state.first_meeting:
        topic = "introduction"
        perspective = "cautious"
    elif conversation_state.active_quest:
        topic = f"quest update: {conversation_state.active_quest.name}"
        perspective = "focused"
    elif conversation_state.recent_conflict:
        topic = "apology" if speaker.disposition > 50 else "grievance"
        perspective = "tense"
    else:
        topic = "small talk"
        perspective = "friendly"

    ndl = f'talk(${speaker.id}, ${listener.id})'
    ndl += f' -> convey("{topic}", perspective="{perspective}")'

    return ndl
```

### Conversation Branching

```ndl
talk($"npc", $"player") -> convey("question with two implications")
-> wait("player choice")
# Game presents: [Agree] or [Refuse]

# If player chooses Agree:
-> talk($"player", $"npc") -> convey("agreement")
-> talk($"npc", $"player") -> convey("next quest step")

# If player chooses Refuse:
-> talk($"player", $"npc") -> convey("refusal")
-> talk($"npc", $"player") -> convey("disappointment")
-> result("relationship decreased")
```

## Dynamic Conversation Generation

### Context Steering

From [[User-veritasr]]:

```ndl
talk($"nobles", $"assembly") -> convey("current affairs discussion")
-> system_response("context: recent law about trade tariffs")
-> wait("topic shift to law")
```

**Implementation Note**:
> "now lets see if we can wiggle the context in the background and get them to steer the conversation..maybe, the latest law coming into effect"

### Personality-Driven Dialogue

```ndl
# Brave character
talk($"hero", $"party") -> convey("encouragement before battle", perspective="courageous")

# Cowardly character
talk($"coward", $"party") -> convey("express fear", perspective="terrified")

# Sarcastic character
talk($"rogue", $"party") -> convey("observation", perspective="sarcastic")
```

**Pattern**: `perspective` acts as a personality lens for dialogue generation.

## Dialogue Mode Transitions

### Normal → Conversation Mode

```ndl
do($"player", "approach") -> target($"npc")
-> system_response("entering conversation mode")
-> talk($"player", $"npc") -> convey("greeting")
```

### Conversation → Combat Mode

```ndl
talk($"player", $"bandit") -> convey("refusal to pay toll", perspective="defiant")
-> result("negotiation failed")
-> system_response("combat initiated")
-> do($"bandit", "attack") -> target($"player")
```

### Group Conversation End Condition

From [[User-veritasr]]:

> "So my question is.. How does it know when the conversation is over? What keeps it from chatting forever in the background?"

**Solution Pattern**:

```ndl
talk($"npc1", $"npc2") -> convey("topic discussion")
-> wait("response")
-> talk($"npc2", $"npc1") -> convey("conclusion", perspective="final")
-> system_response("conversation ended")
```

Or use a turn/topic counter:

```python
def auto_chat(npc1: Entity, npc2: Entity, max_turns: int = 5):
    for turn in range(max_turns):
        if turn % 2 == 0:
            ndl = f'talk(${npc1.id}, ${npc2.id}) -> convey("topic {turn//2 + 1}")'
        else:
            ndl = f'talk(${npc2.id}, ${npc1.id}) -> convey("response to topic {turn//2 + 1}")'

        yield ndl + ' -> wait("response")'

    # End conversation
    yield 'system_response("conversation concluded")'
```

## Common Anti-Patterns

### ❌ Don't: Let LLM Choose Speaker

```ndl
# WRONG - ambiguous speaker
convey("information about quest")
# Who is speaking? LLM might hallucinate the wrong character
```

### ✅ Do: Always Specify Speaker and Listener

```ndl
# CORRECT - explicit speaker and audience
talk($"questgiver", $"player") -> convey("information about quest")
```

### ❌ Don't: Mix Multiple Intents in One Convey

```ndl
# WRONG - too much in one statement
talk($"npc", $"player") -> convey("greeting and quest offer and personal backstory")
```

### ✅ Do: Separate Conversational Beats

```ndl
# CORRECT - one topic per turn
talk($"npc", $"player") -> convey("greeting")
-> wait("player response")
-> talk($"player", $"npc") -> convey("ask about quest")
-> wait("npc response")
-> talk($"npc", $"player") -> convey("quest offer")
```

### ❌ Don't: Assume Dialogue Order Without Sequencing

```ndl
# WRONG - no sequence control
talk($"a", $"b") -> convey("question")
talk($"b", $"a") -> convey("answer")
# LLM might generate both simultaneously, breaking conversation flow
```

### ✅ Do: Use Sequencing for Turn Order

```ndl
# CORRECT - explicit turn order
talk($"a", $"b") -> convey("question")
-> wait("response")
-> talk($"b", $"a") -> convey("answer")
```

## Genre-Specific Variations

### Fantasy RPG Dialogue

```ndl
talk($"wizard", $"apprentice") ~ "arcane terminology"
-> convey("magical instruction", perspective="scholarly")
```

### Noir Detective Dialogue

```ndl
talk($"detective", $"informant") ~ "cynical tone"
-> convey("demand for information", intention="intimidate into honesty")
```

### Sci-Fi Diplomatic Exchange

```ndl
talk($"captain", $"alien ambassador") ~ "universal translator"
-> convey("peaceful intentions", perspective="formal diplomatic protocol")
```

### Horror / Sanity Dialogue

```ndl
talk($"player", $"apparition") -> convey("question reality")
-> result("no coherent response")
-> describe("words echo strangely, filled with static")
-> system_response("sanity decreased")
```

## Integration with Social Systems

### Relationship Tracking

```ndl
talk($"player", $"npc") -> convey("compliment", perspective="sincere")
-> result("positive reaction")
-> system_response("relationship +10")
```

### Reputation Effects

```ndl
talk($"player", $"guild_master") ~ "formally" -> convey("report successful mission")
-> result("impressed")
-> system_response("guild reputation +25")
-> talk($"guild_master", $"player") -> convey("promotion offer")
```

### Social Skill Checks

```ndl
talk($"player", $"guard") -> convey("convincing lie", intention="gain entry")
-> roll("deception") -> result("failure")
-> talk($"guard", $"player") -> convey("suspicion and alarm", perspective="hostile")
-> system_response("guards alerted")
```

## Performance Considerations

### Token Efficiency

Dialogue NDL is lightweight:
- Simple exchange: ~20-30 tokens
- Complex multi-turn: ~50-80 tokens
- Generated output: 40-100 tokens per speaker turn

### Model Requirements

- **Minimum**: 7B-8B models work adequately for dialogue
- **Recommended**: 8B-13B for personality consistency
- **Optimal**: 13B+ for complex multi-character conversations

### Caching Strategies

For repeated conversations with the same NPC:
```python
# Cache character voice/personality with first dialogue
@cache_voice(speaker_id)
def generate_dialogue(speaker: Entity, message: str) -> str:
    ndl = f'talk(${speaker.id}, $"player") -> convey("{message}")'
    return process_ndl(ndl)
```

## Testing Dialogue Patterns

### Validation Checklist

- [ ] Correct speaker attribution (no mixed-up character voices)
- [ ] Appropriate dialogue for character personality
- [ ] Conversational flow feels natural (no abrupt topic jumps)
- [ ] Perspective modifiers affect tone consistently
- [ ] Multi-character conversations maintain distinct voices
- [ ] Emotional context (manner, intention) reflected in word choice
- [ ] Quest state changes reflected in NPC dialogue
- [ ] Conversation endings feel natural, not abrupt

### Common Edge Cases

1. **Character speaks to self**:
   ```ndl
   talk($"player", $"player") -> convey("internal monologue")
   # Some implementations use "think()" instead
   ```

2. **Broadcast message (speaker to all present)**:
   ```ndl
   talk($"crier", $"crowd") -> convey("public announcement")
   ```

3. **Silent protagonist response**:
   ```ndl
   talk($"npc", $"player") -> convey("question")
   -> wait("player chooses response")
   -> system_response("player selected: [dialogue option 2]")
   -> talk($"npc", $"player") -> convey("response to option 2")
   ```

## Real-World Implementation

From [[User-monkeyrithms]] ReallmCraft:
- Quest dialogue changes based on stage variable
- Auto-chat mode for NPC-to-NPC conversations
- Schedule-based conversations trigger when player observes
- Conversation prompts injected with relationship context

From [[User-veritasr]] philosophy:
> "having concrete objectives helps the llm to define your intention.. I'm using this now in ST to guide generation in a specific direction.. Because I want THESE types of events to occur. It causes it to generate certain TYPES of stories."

## Related Patterns

- [[talk]] - Core dialogue construct (to be documented)
- [[convey]] - Message content and perspective (to be documented)
- [[manner-modifier]] - How dialogue is delivered
- [[intention]] - Why character is speaking
- [[social-dynamics]] - Broader social interaction patterns
- [[scene-transitions]] - Mode changes affecting dialogue

## Further Reading

- [[08-NDL-Natural-Description-Language]] - Full NDL discussion
- [[User-veritasr]] - NDL creator
- [[User-monkeyrithms]] - Quest and dialogue implementation
- [[02-Prompt-Engineering]] - Dialogue prompting strategies

---

**Pattern Status**: Validated in Production (ReallmCraft)
**Model Compatibility**: 7B+ models, optimal at 13B+
**Complexity**: Intermediate to Advanced
**Use Cases**: RPGs, interactive fiction, social simulation, quest systems
