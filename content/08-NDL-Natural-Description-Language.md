---
tags: [thread, ndl, markup-language, narration, template-system, design-language]
date: 2026-01-15
source: Discord - LLM World Engine Channel
status: analyzed
---

# NDL - Natural Description Language

## Summary

NDL (Natural Description Language) is veritasr's innovative markup language designed to bridge the gap between programmatic game events and natural language narration. Rather than asking LLMs to make decisions and track state (which they do poorly), NDL provides a structured format for describing *what* should be narrated, allowing LLMs to focus purely on generating high-quality prose. It's essentially a "play-by-play" script that tells the LLM exactly what events occurred and how to present them as narrative text.

## Core Philosophy

> [!quote] veritasr [04:09]
> "Backend processes input and runs logic deciding the result. Then it dynamically generates the NDL prompt, passing in the action chain to the LLM for processing. The end result is what you see here. Basically narrative text. All the rest of the stuff on the backend is just complex logic to make decisions and enforce the rules on the game world. NDL is the piece that glues the two systems together.. Converting from natural text into something the backend can parse, and converting data from the backend into something readable by humans."

**Key Insight**: LLMs excel at autocomplete/narration but fail at consistent decision-making. NDL separates concerns: program decides, LLM narrates.

## The Problem NDL Solves

### Traditional Approach Issues
1. **LLM makes decisions** → Hallucinations, inconsistency, rule violations
2. **LLM tracks state** → Forgets, contradicts itself, loses context
3. **LLM describes outcomes** → May ignore game logic, "decides" arbitrarily

### NDL Approach
1. **Program makes decisions** → Deterministic, consistent, rule-following
2. **Program tracks state** → Perfect memory, no contradictions
3. **NDL describes events** → LLM narrates predetermined outcomes
4. **LLM generates prose** → Quality natural language from structured data

> [!success] Result
> "once you offload the decision making to a system that's actually capable of making sane decisions the hallucinations functionally disappear." - veritasr [04:15]

## NDL Syntax

### Basic Structure

**veritasr** [04:27]: "NDL represents the pieces as:
- what (action())
- how (~ (basically with / by))
- why(intention='')"

### Core Elements

#### Actions
```ndl
do($"character", "action_description")
```
Represents an action being taken. The `$` prefix indicates a reference to a game entity.

#### Modifiers (How)
```ndl
~ "method_or_tool"
```
The `~` operator specifies how an action is performed.

#### Intentions (Why)
```ndl
intention="motivation_or_reason"
```
Provides context for why an action is taken.

#### Sequencing
```ndl
->
```
The arrow operator chains actions sequentially.

#### Waiting/Conditions
```ndl
wait("condition")
```
Represents pausing or waiting for something.

### Example

**veritasr** [05:38]: "Basic NDL Processing."
```ndl
do($"you", "write some NDL")->wait("response validation")
```

**Interpretation**: You (the entity) are writing some NDL, then waiting for response validation.

**Backend Processing**:
```python
Input String:  do($"you", "write some NDL")->wait("response validation")
is_ndl:  True
continue_last:  False
end_scene:  True
transition_scene_type:  both
Actions:  ['do', 'wait']
```

## How NDL Works

### The NDL Pipeline

```
User Input (Natural Language)
    ↓
Input Parser (Backend)
    ↓
Game Logic Engine (Makes Decisions)
    ↓
NDL Generator (Creates Action Chain)
    ↓
Dynamic Prompt Builder (Adds NDL + Context)
    ↓
LLM (Narrates Based on NDL)
    ↓
Natural Language Output (Story Text)
```

### Bidirectional Translation

**Natural Language → NDL**:
```
User: "I try to sneak past the guard"
    ↓ Parser
NDL: do($"player", "sneak")~"quietly"->target($"guard")->check("stealth")
    ↓ Game Logic
Result: Success/Failure determined by game rules
    ↓ NDL Generator
NDL: do($"player", "sneak successfully")~"quietly"->pass($"guard")->result("undetected")
```

**NDL → Natural Language**:
```
NDL: do($"player", "sneak successfully")~"quietly"->pass($"guard")->result("undetected")
    ↓ LLM Narration
Output: "You carefully make your way past the guard, your soft footsteps masked by the ambient noise of the courtyard. He doesn't notice your passage."
```

### Dynamic Prompt Generation

**veritasr** [10:30]: "configured the class to do dynamic prompt generation, meaning that it sort of builds my prompt up on the fly based on the actions taken."

The system constructs prompts dynamically:
```python
def build_ndl_prompt(actions, context):
    prompt = "Narrate the following events:\n\n"
    prompt += f"Setting: {context.location}\n"
    prompt += f"Characters present: {context.npcs}\n\n"
    prompt += f"Events: {actions_to_ndl(actions)}\n\n"
    prompt += "Write a natural description of these events."
    return prompt
```

## Evolution and Development

### Phase 1: Concept and Naming (June 2024)

**veritasr** [04:07]: "Yeah, the good thing about the DSL method is that it's basically giving the LLM a play by play of what to write. Which means that the BS where it doesn't want to describe characters is potentially something of the past."

**veritasr** [04:07]: "Guessing I should name the syntax."

**veritasr** [04:07]: "Hmm... NDL (Narrative Description Language)?"

> [!note] Origin
> NDL grew out of the realization that LLMs work better when told exactly what to describe rather than being asked to make creative decisions.

### Phase 2: Initial Testing (June 2024)

**veritasr** [03:59]: Testing NDL with example scenarios:
[Screenshot showing NDL-driven narration]

**veritasr** [04:01]: "let's test it with system messages to see how it would respond to roll results."

**veritasr** [04:04]: [Screenshot showing system message integration]

**veritasr** [04:04]: "Yup. Seems like it would work just fine."

> [!success] Validation
> Initial tests showed NDL could effectively communicate game outcomes to LLMs for narration, even with dice roll results.

### Phase 3: Philosophical Foundation (June 2024)

**veritasr** [04:14]: "The process itself isn't actually too different from how diffusion works or the old bart stuff. You provide the LLM with input and a task, then have it fill in the blanks based on what it knows. It processes the words that are given to it, along with the context and tries to string them together in a probabilistic way. Functions more in line with how LLMs actually work... as autocomplete engines."

**veritasr** [04:15]: "once you offload the decision making to a system that's actually capable of making sane decisions the hallucinations functionally disappear."

**veritasr** [04:16]: "Sorta what I've been saying all along"

> [!important] Core Insight
> NDL treats LLMs as what they actually are: sophisticated autocomplete engines. By providing structured "fill in the blanks" tasks instead of open-ended decision-making, reliability increases dramatically.

### Phase 4: TTRPG-Inspired Design (June 2024)

**veritasr** [04:21]: "And this is all on 8B / 9B LLMs. Nothing that really breaks the bank hardware wise. 🤷"

**veritasr** [04:24]: "There's also a sort of recipe on the input, which is the same one you use as a GM for people who don't know how to play TTRPGs:
- what (do you do)
- how (do you do it)
- why (are you doing it)

With those three pieces of information the LLM has the context it needs to generate text in a pretty reliable manner. The backend just serves to answer those questions for non-player controlled agents, inject content and events, and determine failure chance."

> [!tip] TTRPG Connection
> NDL mirrors how tabletop RPG GMs narrate: players declare intent (what/how/why), dice determine outcome, GM narrates the result. NDL codifies this pattern.

### Phase 5: Social Dynamics Expansion (June 2024)

**veritasr** [16:59]: "On another note, updated the NDL to encompass conversation content, opinions / perspectives, and describe how something is accomplished. Should capture the content of conversations now in a way that allows describing intent so that reasoning can be applied and characters can react with different viewpoints."

**veritasr** [19:47]: "hardest part so far has been thinking about how to model social dynamics. There's a whole lot of permutations on interactions between characters and how those interactions progress. NDL currently has 43 actions tied to social and is still growing. and 23 different exposed states just tied to social interaction."

> [!note] Complexity Growth
> NDL evolved from simple action descriptions to encompassing complex social interactions, opinions, and character perspectives.

### Phase 6: Implementation and Parsing (June-July 2024)

**veritasr** [05:27]: "System I'm looking at designing is a mixture of parsing NDL and checking various states in game to determine if substeps or conditions are met."

**veritasr** [18:02]: "Also spent some time yesterday playing with the NDL, still seems to work effectively across multiple models. Get's tricky when there's back messages, since it sometimes tries to select previous messages, but I'll be doing it in a multi-step workflow, so that won't be a problem."

**veritasr** [03:45]: "NDL works on it. Pretty meh about the writing style though.."

> [!important] Multi-Model Compatibility
> NDL works across different LLMs (8B-9B models tested). Writing style varies by model but structure is reliable.

### Phase 7: Bidirectional Processing (July 2024)

**veritasr** [11:12]: "got conversion from input to NDL working."

**veritasr** [22:09]: "figured out how to process conversations in NDL without having to parse every word. Gonna continue this evening on building out templates."

> [!success] Bidirectional Support
> NDL supports both directions: Natural Language → NDL (parsing) and NDL → Natural Language (generation).

## Technical Implementation

### Detection and Parsing

```python
def is_ndl(input_string):
    # Check for NDL syntax markers
    ndl_markers = ['do(', '->', 'wait(', '~', 'intention=', '$']
    return any(marker in input_string for marker in ndl_markers)

def parse_ndl(input_string):
    results = {
        'is_ndl': is_ndl(input_string),
        'continue_last': check_continuation(input_string),
        'end_scene': check_scene_end(input_string),
        'transition_scene_type': determine_transition_type(input_string),
        'actions': extract_actions(input_string)
    }
    return results
```

### Dynamic Prompt Building

**veritasr** [10:30]: "configured the class to do dynamic prompt generation, meaning that it sort of builds my prompt up on the fly based on the actions taken."

The system builds context-aware prompts:
```python
class NDLPromptBuilder:
    def build_prompt(self, ndl_actions, world_state):
        prompt_parts = []

        # Add scene context
        prompt_parts.append(f"Setting: {world_state.location.name}")
        prompt_parts.append(f"Time: {world_state.time}")

        # Add character context
        for char in world_state.active_characters:
            prompt_parts.append(f"- {char.name}: {char.current_state}")

        # Add NDL action sequence
        prompt_parts.append("\nEvents occurring:")
        prompt_parts.append(ndl_actions)

        # Add narration instruction
        prompt_parts.append("\nNarrate these events naturally:")

        return "\n".join(prompt_parts)
```

### Scene Transitions

**veritasr** [05:36]: "Transitions either occur due to a change in time, place, or both."

```python
transition_types = {
    'time': "Time has passed",
    'place': "Location has changed",
    'both': "Time and location have changed"
}
```

## Benefits of NDL

### 1. Hallucination Reduction
By constraining LLM output to narrating pre-determined events, hallucinations about game state disappear.

### 2. Consistency
Game logic handles state, guaranteeing consistency across sessions.

### 3. Model Flexibility
Works on small models (8B-9B parameters) without requiring GPT-4.

### 4. Bidirectional Translation
Can parse natural language into NDL and generate natural language from NDL.

### 5. Composability
Actions can be chained (`->`) to create complex event sequences.

### 6. Extensibility
New action types can be added without changing core architecture.

### 7. Separation of Concerns
- **Backend**: Logic, rules, state management
- **NDL**: Event description, action sequencing
- **LLM**: Prose generation, style, flavor

## Use Cases

### Combat Narration
```ndl
do($"player", "attack")~"sword"->target($"goblin")->roll("hit")->result("success")->damage(8)
```

LLM Output: "You swing your sword in a wide arc, catching the goblin off-guard. Your blade bites deep into its side, dealing a grievous wound."

### Social Interaction
```ndl
do($"player", "persuade")~"charming words"->target($"merchant")->intention="get discount"->roll("charisma")->result("failure")
```

LLM Output: "You attempt to sweet-talk the merchant into lowering his prices, but he's heard it all before. 'These are fair prices,' he says firmly, crossing his arms."

### Exploration
```ndl
do($"player", "search")~"carefully"->target($"room")->result("find")->item($"hidden_key")
```

LLM Output: "You carefully search the room, running your fingers along the walls and under furniture. Behind a loose stone in the fireplace, you discover a tarnished brass key."

### Scene Transitions
```ndl
end_scene()->transition("place")->new_location($"forest_path")->time_passage("3 hours")
```

LLM Output: "Three hours of walking brings you deep into the forest. The city walls are long behind you now, replaced by towering trees and the sounds of wildlife."

## Comparison to Other Approaches

| Approach | Decision Making | Narration | Consistency | Model Requirements |
|----------|----------------|-----------|-------------|-------------------|
| **Pure LLM** | LLM decides everything | LLM narrates | Poor | GPT-4 or equivalent |
| **Function Calling** | LLM calls functions | LLM narrates | Medium | GPT-3.5+ |
| **NDL** | Program decides | LLM narrates | Excellent | 8B models sufficient |
| **Template-Only** | Program decides | Templates | Excellent | No LLM needed |

NDL sits in the sweet spot: programmatic reliability with LLM-quality narration.

## Related Concepts

### DSL (Domain-Specific Language)
NDL is a DSL for game events. It's specialized for describing narrative actions in game contexts.

### Prompt Templates
NDL is dynamically generated prompt templates, but structured as a language rather than static templates.

### AIML / ChatScript
Earlier chatbot markup languages. NDL is similar in spirit but designed specifically for game narration.

### Markdown / HTML
Markup languages for documents. NDL is a markup language for events/actions.

## Design Principles

1. **Simplicity**: Syntax should be minimal and clear
2. **Composability**: Actions chain naturally with `->`
3. **Expressiveness**: Can describe complex multi-step events
4. **Parsability**: Both machines and humans can read NDL
5. **Model-Agnostic**: Works across different LLMs
6. **Bidirectional**: Can parse from and generate to natural language
7. **Game-Focused**: Designed specifically for game event description

## Limitations and Challenges

### 1. Complexity Management
**veritasr** [19:47]: "hardest part so far has been thinking about how to model social dynamics. There's a whole lot of permutations on interactions between characters and how those interactions progress. NDL currently has 43 actions tied to social and is still growing."

As scope increases, NDL vocabulary grows. Need to balance expressiveness vs complexity.

### 2. Backreferences
**veritasr** [18:02]: "Get's tricky when there's back messages, since it sometimes tries to select previous messages"

Referencing previous events in conversation is challenging. Workflow-based approach helps.

### 3. Writing Quality
**veritasr** [03:45]: "NDL works on it. Pretty meh about the writing style though.."

Writing quality depends on underlying model. Smaller models produce functional but less creative prose.

### 4. Learning Curve
Developers must learn NDL syntax. Documentation and tooling needed.

### 5. Overkill for Simple Cases
For simple narration, templates might suffice. NDL shines in complex, multi-action sequences.

## Future Directions

### Potential Enhancements
1. **NDL Editor**: Visual tool for building action sequences
2. **Standard Library**: Common action patterns as reusable modules
3. **Validation**: Schema/grammar for validating NDL syntax
4. **Debugging**: Tools to visualize NDL → Narration pipeline
5. **Compression**: Shorthand notation for common patterns
6. **Templates**: Pre-built NDL sequences for common scenarios
7. **Cross-Game Standard**: Standardize NDL across multiple engines

## Related Topics

- [[01-Architecture-and-Design]] - NDL fits into program-first architecture
- [[02-Prompt-Engineering]] - NDL is a prompting technique
- [[03-RAG-and-Memory]] - NDL events can be stored for retrieval
- [[05-State-Management]] - NDL bridges state and narration
- [[07-Models-and-APIs]] - NDL works on small models
- [[User-veritasr]] - NDL creator and primary implementer

## Key Insights

1. **Treat LLMs as narrators, not decision-makers** - NDL enforces this separation
2. **Structured event description eliminates hallucinations** - No room for LLM to invent game state
3. **Works on small models** - 8B-9B models sufficient when decisions are pre-made
4. **TTRPG pattern works for games** - What/How/Why structure from tabletop translates to code
5. **Bidirectionality is powerful** - Parse player input to NDL, generate narration from NDL
6. **Dynamic prompt building scales** - Build prompts from action sequences as needed
7. **Social dynamics are complex** - 43+ action types for social alone shows depth needed

## Open Questions

> [!question] Unresolved Issues
> 1. What's the optimal NDL vocabulary size? (Too small = not expressive, too large = complex)
> 2. How to handle ambiguous natural language → NDL parsing?
> 3. Can NDL be standardized across different engines?
> 4. Is there a visual programming interface for NDL?
> 5. How to version NDL as it evolves?
> 6. Can LLMs learn to write NDL directly (for mod creators)?

## Example: Complete Flow

```
Player Input: "I sneak past the guard to reach the treasure room"
    ↓
Backend Parser: Determines intent, checks stealth skill
    ↓
Game Logic: Rolls stealth check → Success
    ↓
NDL Generator:
    do($"player", "sneak")~"carefully"
    ->target($"guard")
    ->check("stealth", result="success")
    ->move_to($"treasure_room")
    ->remain("undetected")
    ↓
Dynamic Prompt:
    Setting: Castle Hallway
    Characters: Player, Guard (unaware)

    Events: do($"player", "sneak")~"carefully"->target($"guard")
            ->check("stealth", result="success")
            ->move_to($"treasure_room")->remain("undetected")

    Narrate these events naturally, focusing on tension and stealth:
    ↓
LLM (8B model):
    "You press yourself against the cold stone wall, moving with
    practiced silence. The guard shifts his weight but doesn't
    turn as you slip past his position. Heart pounding, you reach
    the treasure room door undetected."
    ↓
Output to Player
```

## Timeline

- **June 2024**: NDL concept and naming
- **June 2024**: Initial testing and validation
- **June 2024**: Philosophical foundation established
- **June 2024**: TTRPG-inspired design patterns
- **June 2024**: Social dynamics expansion (43+ actions)
- **June-July 2024**: Multi-model compatibility testing
- **July 2024**: Bidirectional processing implementation
- **July 2024**: Scene transition handling
- **Ongoing**: Vocabulary expansion and refinement

---

> [!success] Core Achievement
> NDL represents a paradigm shift in LLM game engines: instead of asking LLMs to be game masters (which they can't do reliably), ask them to be novelists (which they excel at). By separating decision-making from narration through a structured markup language, NDL achieves the reliability of programmatic systems with the quality of LLM-generated prose. It works on small models (8B-9B parameters), eliminates hallucinations about game state, and provides bidirectional translation between natural language and structured events.

## Related Threads

- [[01-Architecture-and-Design]] - How NDL fits into architecture
- [[02-Prompt-Engineering]] - NDL prompting techniques
- [[05-State-Management]] - NDL for state transitions
- [[User-veritasr]] - NDL creator and primary developer

## Related Enrichment Outputs

### NDL Specification (Complete Reference)
- [[ndl/00-NDL-INDEX]] - Complete NDL language index
- [[ndl/specification/01-lexical-elements]] - Tokens, keywords, operators
- [[ndl/specification/02-grammar]] - Formal EBNF grammar
- [[ndl/specification/03-semantics]] - Execution model and meaning
- [[ndl/specification/04-type-system]] - Parameter types and validation

### NDL Constructs
- [[ndl/constructs/do-action]] - The fundamental action construct
- [[ndl/constructs/search]] - Search actions with intentions
- [[ndl/constructs/wait]] - Timing and pauses
- [[ndl/constructs/result]] - Outcome specification
- [[ndl/constructs/sequencing]] - Sequential action chaining (->)
- [[ndl/constructs/manner-modifier]] - How actions are performed (~)
- [[ndl/constructs/intention]] - Why actions are performed

### NDL Patterns
- [[ndl/patterns/combat-narration]] - Combat sequence patterns
- [[ndl/patterns/dialogue-generation]] - Conversation patterns
- [[ndl/patterns/scene-transitions]] - Moving between scenes
- [[ndl/patterns/environmental-description]] - Location descriptions
- [[ndl/patterns/social-dynamics]] - 43+ social action catalog

### NDL Integration
- [[ndl/integration/ndl-to-llm-flow]] - NDL → LLM prompt pipeline

### Prompt Library
- [[prompts/narration/ndl-to-narrative]] - Primary NDL-to-narrative prompt

## Further Reading

For implementation details, see veritasr's ReallmCraft repository (referenced in discussions but implementation details not fully public at time of analysis).
