---
tags:
- prompt
- techniques
- ndl
- few-shot
- thread
date: 2026-01-15
participants:
- '[[User-veritasr]]'
- '[[User-50h100a]]'
- '[[User-appl2613]]'
- '[[User-yukidaore]]'
status: analyzed
---

# Thread: Prompt Engineering and LLM Control

## Summary

This thread documents the evolution of prompt engineering techniques specifically for game engines and narrative generation. The community moved from relying on LLM decision-making to treating LLMs as controlled narration engines, with prompting serving to constrain rather than empower the AI.

## Core Philosophy Shift

### Early Phase: LLM as Decision Maker
Initial attempts gave LLMs significant autonomy:
- Parse player input and update state
- Make game logic decisions
- Generate events and consequences

**Result**: Unreliable, inconsistent, prone to hallucinations

### Later Phase: LLM as Narration Layer
Final consensus architecture:
- Backend makes all decisions deterministically
- LLM receives structured instructions
- LLM's only job is converting data → natural text

> [!quote] Core Insight
> "Turns out that when you take away decision making from the LLM it behaves much better." - [[User-veritasr]]

## Key Techniques

### 1. Chain of Thought (CoT) Prompting

**Concept**: Have the LLM "think through" the response step-by-step before generating final output.

**Early Implementation** ([[User-50h100a]]):
```
Health:
```
If prompt ends here, LLM instinctively fills in the blank.

**Character State Example**:
```
> Character: Bob
> Location: Tavern
> Mood: Nervous
> What Bob wants: Information about the quest
> What Bob will say:
```

**Benefits**:
- Forces LLM to consider context
- Reduces impulsive/random responses
- Creates implicit "reasoning" chain

**Downsides**:
- Slower generation (more tokens)
- Requires careful prompt structure
- Can be verbose

> [!tip] CoT Best Practice
> "You just tell the llm to start every response with x,y, and z. It'll 'think' for itself." - [[User-underscore_x]]

### 2. Few-Shot and One-Shot Prompting

**Purpose**: Provide examples of desired output format

**Basic Example**:
```
Example 1:
Input: "I draw my sword"
Output: action(draw) ~ item($sword) by(character($player))

Example 2:
Input: "She whispers nervously"
Output: action(speak) ~ manner($whisper) emotion($nervous) by(character($npc_name))

Now convert: "[user input]"
```

**When to Use**:
- Teaching LLM new formats (like NDL)
- Constraining output structure
- Preventing rambling responses

**Discussion from Transcript**:

[[User-50h100a]]: "note that this requires few-shot or at least one-shot unless you are clever with formatting"

[[User-monkeyrithms]]: "gpt-4 was very good, its the only model that can kinda-sorta one-shot a good RP with all the rules just added to the context"

### 3. Statblock Prompting

**Two Approaches**:

**Prefix (Guide the LLM)**:
```
[Current Status]
Health: 85/100
Stamina: Tired
Mood: Anxious
Recent Events: Just escaped from guards

[Continue the story...]
```
LLM's output is influenced by these states.

**Postfix (For the Reader)**:
```
[Story continues...]

--- Status Update ---
Health: 75/100
Thoughts: "I need to find shelter"
Arousal: 0
```
LLM summarizes for reader, doesn't affect generation.

**Consensus**: Prefix is more useful for game logic.

### 4. Constraint-Based Prompting

**Philosophy**: Tell LLM what it CAN'T do rather than what it CAN do.

**Example from [[User-yukidaore]]**:
```
{{char}} is a logical and realistic text adventure game
{{user}} has no equipment, items, powers, abilities, skills, or magic
unless explicitly mentioned otherwise
Impossible actions must fail
```

**Why This Works**:
- Easier to enumerate constraints than possibilities
- Prevents creative hallucination
- Aligns with how LLMs pattern-match

**Application**: Prevents diamond horses and spontaneous teleportation.

### 5. Function-Like Keywords

**Early Proposal** ([[User-50h100a]], January 2024):
```
FunctionRoll(Willpower, Ram Girl Lyre)
→ Stop generation
→ Perform roll programmatically
→ Inject result
→ Continue generation
```

This predated mainstream function-calling but pointed toward the same concept.

**Evolution**: By mid-2024, most APIs supported native function calling.

### 6. Separation of Actions and Dialogue

**Problem**: LLMs blend narration, action, and dialogue inconsistently.

**Solution** ([[User-irovos]]):
> "From the basic get go, i think you need to separate actions and dialogue"

**Implementation Examples**:

**Approach A - Structured Format**:
```
[Action]: Character draws sword
[Dialogue]: "Stand back!"
[Narration]: The blade gleams in the firelight
```

**Approach B - Parsing Markers**:
```
*draws sword dramatically* "Stand back!" The blade gleams...
```
Then parse asterisks for actions, quotes for dialogue.

**Approach C - NDL (veritasr's solution)**:
Separate completely: Backend generates structured action data, LLM only narrates.

## Natural Description Language (NDL)

### Concept

**Creator**: [[User-veritasr]]
**Purpose**: Convert programmatic game events into structured prompts for LLM narration

**Architecture**:
```
Game Events (Backend) → NDL Markup → LLM → Natural Narrative
```

**Core Insight**: LLM functions as translator, not decision-maker

### NDL Syntax

**Basic Action Structure**:
```
action($verb) ~ manner($how) intention($why) by(character($who))
```

**Example**:
```
do($"you", "write some NDL") -> wait("response validation")
```

**Components**:
- **action()** - What happens (verb)
- **~** - How it happens (with/by)
- **intention=""** - Why it happens
- **by()** - Who does it
- **→** or **->** - Triggers/consequences

### Three-Question Framework

**Borrowed from Tabletop GM Practice**:
1. **What** (do you do) - `action()`
2. **How** (do you do it) - `~ manner()`
3. **Why** (are you doing it) - `intention=""`

**[[User-veritasr]]'s Explanation**:
> "With those three pieces of information the LLM has the context it needs to generate text in a pretty reliable manner. The backend just serves to answer those questions for non-player controlled agents, inject content and events, and determine failure chance."

### NDL Prompting Workflow

**Step 1: Backend Processes Input**
```python
# User types: "I sneak past the guard"
player_action = parse_input("sneak past guard")
guard_perception = roll_check(guard.perception, player.stealth)
result = "success" if player wins else "failure"
```

**Step 2: Generate NDL**
```
do($player, "sneak")
~ manner($quiet)
target(guard($guard_name))
roll($stealth, result=$success)
location($hallway)
lighting($dim)
```

**Step 3: LLM Converts to Narrative**
Input: NDL markup above
Output: "You crouch low, moving with practiced silence through the dimly lit hallway. The guard's attention is focused elsewhere, and you slip past unnoticed."

**Step 4: System Message for Consequences** (optional)
```
[Guard did not notice you. You are now in the courtyard.]
```

### Advanced NDL: Subtext and Implication

**Late-Stage Development** (July 2025):

[[User-veritasr]] achieved natural subtext by structuring character psychology:

**Character Belief Layer**:
```
character_beliefs($"The player is hiding something")
character_knows($"Player recently arrived")
character_wants_to_convey($"I'm watching you")
character_actually_says ~ controlled_by(intention + social_context)
```

**Rules**:
- Never state beliefs outright
- Information that "comes to mind" influences word choice
- What they WISH to convey ≠ what they DO convey

**Result**: Natural subtext without explicit dialogue scripting.

**[[User-veritasr]]'s Report**:
> "Achieved consistent subtext in narrative. Controlled emotional beats. Reliable dialogue with implicit intent. Works across multiple models (Gemma, Llama3, Mistral, Stheno)."

### NDL Testing Results

**Models Tested**:
- Gemma 2 (9B)
- Llama 3 (7B-9B)
- Mistral 7B
- Stheno
- EstopianMaid-13B

**Success Rate**: High consistency across models (7B-9B range)

**Token Efficiency**: Context windows stayed under 3k tokens typically

**Observations**:
- Gemma: Serviceable but "meh" writing style
- Llama 3: Clean, accurate narration
- EstopianMaid-13B: veritasr's main model, good balance

## Prompt Structure Patterns

### Template-Based Prompting

**Concept**: Use templates with placeholders for dynamic content

**Example - Tavern Description**:
```
Describe a {{size}} {{location_label}} called {{name}}
in a {{location_subtype}} {{location_type}} setting.

This establishment has a {{reputation}} reputation
and a {{overall_wealth}} level of wealth.
The atmosphere is {{ambiance}}.

Include details about:
* The tavern's appearance and notable features
* The types of patrons (drawn from {{common_patrons}})
* Typical events or activities (from {{common_events}})

Ensure the description reflects the {{security_level}}
security level and {{opening_hours}} operating hours.
```

**Benefits**:
- Consistent structure
- Easy to modify
- Data-driven approach
- Modding-friendly

### Conditional Prompting

**Use Case**: Different prompts based on game state

**Example from [[User-veritasr]]**:
```python
if character.state == "frozen_in_time":
    prompt = character.frozen_prompt
else:
    prompt = character.normal_prompt
```

**[[User-monkeyrithms]]** implemented this for NPCs at work:
```python
if at_workplace() and during_work_hours():
    inject_career_description()
else:
    use_standard_description()
```

### Iterative Refinement Prompting

**Pattern**: Progressively refine output through multiple passes

**World Generation Example**:
1. Generate high-level world description
2. Generate specific region within world
3. Generate locations of interest in region
4. Generate characters for each location
5. Generate relationships between characters

Each step informs the next, building coherence.

### Hypothetical Question Generation

**Technique**: Improve RAG retrieval by generating questions the text could answer

**[[User-veritasr]]'s Implementation**:
```
Instead of storing: "The Sword of Kings is a legendary blade forged in dragon fire"

Store: "What is the Sword of Kings? A legendary blade forged in dragon fire"
Or: "How was the Sword of Kings created? It was forged in dragon fire"
```

**Why**: Semantic similarity better matches questions to questions than descriptions to questions.

**Called**: HyDE (Hypothetical Document Embeddings)

## Model-Specific Prompting Challenges

### Small Model Limitations (7B-9B)

**Issues**:
- More literal interpretation
- Less able to "fill in gaps"
- Requires tighter constraints
- Can't handle complex multi-step reasoning

**Solutions**:
- More explicit instructions
- Shorter, simpler prompts
- More few-shot examples
- Stricter formatting

### Creative Models (e.g., Hathor)

**Problem**: Too creative for structured games

**Example from [[User-yukidaore]]** testing ChatBotRPG:
> "Death Knight Mara with a massive battleaxe appeared instead of a bartender. Diamond horses manifested. Teleportation happened spontaneously."

**Solutions**:
- Much stricter constraints
- Explicit "impossible actions must fail"
- Longer list of prohibited behaviors
- May need different model entirely

### Instruction-Following Models (Mixtral, GPT-3.5)

**Strengths**:
- Follow rules reliably
- Handle structured output well
- Good for game logic validation

**Weaknesses**:
- Can be less creative/engaging
- More robotic prose
- May need creative boost in prompts

### Larger Models (GPT-4, Claude)

**[[User-monkeyrithms]]**:
> "gpt-4 was very good, its the only model that can kinda-sorta one-shot a good RP with all the rules just added to the context and it follows them all."

**Trade-offs**:
- Expensive
- Slower
- Overkill for simple narration
- Better for complex reasoning

## Pacing and Time Management

### The One-Shot Problem

**Issue**: LLMs have no concept of time progression

**[[User-monkeyrithms]]**:
> "LLMs don't really have much concept of 'time' -- each time they're prompted with their next turn, to their perspective, they're posting the very first time to a list of pre-existing text that could have been all written a second ago"

**Solution**: Quest Stages and Turn Counting

**Implementation**:
```
Turn 1: Introduction to quest giver
Turn 2: Quest giver mentions the problem
Turn 3: Quest giver reveals the stakes
Turn 4: Decision point for player
```

Forces narrative to develop gradually rather than resolving immediately.

### Upscaling vs. One-Shotting

**Concept** ([[User-monkeyrithms]]):

**One-Shot**: Generate entire response at once (high token count)
**Upscaling**: Generate response incrementally with multiple prompts

**When to Upscale**:
- Complex scenes
- Important narrative beats
- Character introductions
- Combat sequences

**When to One-Shot**:
- Simple actions
- Routine descriptions
- Status updates

### Combat Prompting

**Consensus**: One-shot combat doesn't work well

**[[User-monkeyrithms]]**:
> "One-shotting combat doesn't work too well unless it's GPT-4 in my experience, but a CoT type thing does"

**Preferred Approach**:
```
1. Describe combat start
2. Player chooses action
3. LLM narrates action execution
4. System calculates results
5. LLM narrates consequences
6. Repeat until combat ends
```

**Alternative**: Backend handles all combat, LLM only narrates outcome.

## Anti-Patterns and Pitfalls

### 1. Assuming the LLM Understands Context

**Problem**: LLM doesn't "remember" previous rules unless in context

**Solution**: Reinforce critical rules in every prompt or use system message

### 2. Vague Instructions

**Bad**: "Describe the scene"
**Good**: "Describe the tavern's appearance, focusing on lighting, sounds, and the bartender's demeanor. 2-3 sentences."

### 3. Giving LLM Too Much Freedom

**Result**: Diamond horses, teleportation, spontaneous powers

**[[User-yukidaore]]'s Fix**:
> "I generally just use a combination of '{{char}} is a logical and realistic text adventure game' and 'Impossible actions must fail'"

### 4. Not Testing Across Models

**Issue**: Prompts that work on GPT-4 often fail on local 7B models

**Best Practice**: Test on target model early and often

### 5. Over-Engineering Prompts

**Trap**: Adding more and more instructions to fix edge cases

**Result**: Prompt bloat, slower generation, increased costs

**Solution**: Fix problems programmatically, not through prompting

## Temperature and Sampling Settings

### General Recommendations

**For Structured Output (Actions, Parsing)**:
- Temperature: 0.1 - 0.3
- Top-p: 0.8 - 0.9
- Repetition Penalty: 1.1 - 1.2

**For Creative Narration**:
- Temperature: 0.6 - 0.9
- Top-p: 0.9 - 0.95
- Repetition Penalty: 1.15 - 1.3

**For Dialogue**:
- Temperature: 0.7 - 1.0
- Top-p: 0.9 - 0.95
- Repetition Penalty: 1.2 - 1.3

### Dynamic Temperature

**Concept**: Adjust temperature based on task

```python
if task == "validate_action":
    temp = 0.2  # Need accuracy
elif task == "generate_dialogue":
    temp = 0.9  # Want variety
elif task == "narrate_event":
    temp = 0.7  # Balance
```

## Lorebook and Worldinfo Integration

### Keyword Triggering

**Pattern**: Insert relevant lore when keywords detected

**Example**:
```
User: "Tell me about the Sword of Kings"
→ Trigger: "Sword of Kings" lorebook entry
→ Inject into context
→ Generate response with lore available
```

### Issues with Keyword Matching

**Problem**: Requires exact match or close regex

**[[User-veritasr]]**:
> "Bummer is that you need an exact match or you end up with stuff that's not really relevant."

**Solution**: Hybrid approach (exact + semantic search)

### Semantic Lorebook Retrieval

**Better Approach**: Use embedding similarity

1. User asks question
2. Encode question as vector
3. Find closest lorebook entries by similarity
4. Inject top N matches
5. Generate response

**Trade-off**: More complex, requires vector DB, but more flexible

## Prompt Evolution Timeline

### January 2024: Experimentation Phase
- Testing LLM as state manager (failed)
- Exploring CoT patterns
- Statblock experiments
- Function-like keyword proposals

### February-April 2024: Refinement
- Separation of actions and dialogue
- Template-based prompting
- Constraint-based approaches
- Early NDL concepts

### May-June 2024: Stabilization
- NDL formalization
- Quest stage systems
- Turn counting
- Pacing techniques

### July 2024-July 2025: Advanced Techniques
- Subtext and implication
- Character psychology layers
- Modding frameworks
- Cross-model compatibility

## Best Practices Summary

### DO:
1. ✅ Keep LLM role narrow (narration only)
2. ✅ Use templates for consistency
3. ✅ Test on target model early
4. ✅ Separate actions from dialogue
5. ✅ Provide clear constraints
6. ✅ Use few-shot for new formats
7. ✅ Consider pacing (not one-shot everything)
8. ✅ Adjust temperature per task

### DON'T:
1. ❌ Let LLM make game logic decisions
2. ❌ Assume it understands implicit rules
3. ❌ Over-complicate prompts
4. ❌ Forget to test on small models
5. ❌ Mix multiple tasks in one prompt
6. ❌ One-shot complex sequences
7. ❌ Give too much creative freedom
8. ❌ Ignore model-specific quirks

## Related Threads

- [[01-Architecture-and-Design]] - How prompting fits into architecture
- [[03-RAG-and-Memory]] - Retrieval for prompt context
- [[04-World-Generation]] - Prompts for content creation
- [[08-NDL-Natural-Description-Language]] - Detailed NDL documentation
- [[User-veritasr]] - Primary NDL developer
- [[User-appl2613]] - Quest staging and pacing techniques

## Related Enrichment Outputs

### Prompt Library
- [[prompts/00-PROMPT-INDEX]] - Complete prompt template library
- [[prompts/narration/ndl-to-narrative]] - NDL-to-narrative conversion (PRIMARY)
- [[prompts/constraint/anti-hallucination]] - Core constraint rules
- [[prompts/constraint/hallucination-prevention]] - Explicit prohibitions with data grounding
- [[prompts/reasoning/chain-of-thought]] - CoT implementation
- [[prompts/techniques/few-shot-examples]] - Few-shot prompting patterns

### Pattern Library
- [[patterns/00-PATTERN-INDEX]] - Complete pattern library
- [[patterns/control/constraint-based-prompting]] - Constraint pattern implementation
- [[patterns/control/chain-of-thought]] - CoT control pattern with function keywords

### NDL Specification
- [[ndl/00-NDL-INDEX]] - Complete NDL language reference
- [[ndl/specification/02-grammar]] - Formal NDL grammar
- [[ndl/constructs/do-action]] - Core do() construct

## Key Quotes

> "CoT with external state tracking, essentially" - [[User-50h100a]]

> "LLMs are super cliche and shallow on their own devices... but so would we if we just one-shot everything" - [[User-appl2613]]

> "Prompt engineering to wrangle it into a thing that works most of the time" - [[User-veritasr]]

> "The answer was 'security'" - [[User-lyrcaxis]] (on why backend work takes longer than expected)

---

## Future Directions

- Integration with Model Context Protocol (MCP)
- Automated prompt optimization
- Multi-agent prompting strategies
- Adaptive prompting based on player behavior
- Better subtext and implication techniques

