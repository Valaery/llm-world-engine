---
tags: [index, templates, generation, reference]
date: 2026-01-17
status: complete
---

# Template Reference Index

Quick reference to generation templates, prompt patterns, and structured output formats across the LLM World Engine knowledge base.

## Overview

This index catalogs all templates, prompt structures, and generation patterns extracted from the Discord discussions. Templates are documented alongside the prompts and patterns that use them, ensuring practical context.

**Philosophy**: Templates belong with their usage context. This index provides navigation to templates embedded throughout the documentation.

## Content Generation Templates

### Character Generation
**Location**: [[prompts/generation/character-generation]]

**Template Variables**:
```
{{SETTING}} - World/campaign setting
{{CHARACTER_TYPE}} - Role or archetype
{{RELATIONSHIP_TO_PLAYER}} - Ally/enemy/neutral
{{REQUIRED_ABILITIES}} - Specific skills needed
```

**Output Format**: JSON with fields:
- name, role, personality, appearance
- background, motivation, secrets
- equipment, abilities, stats
- hooks (quest/interaction opportunities)

**Variations**:
- Quick NPC (minimal detail)
- Companion Character (detailed)
- Antagonist Generation
- Context Inheritance (location-themed)

**Model Requirements**:
- GPT-4: One-shot generation
- GPT-3.5: Requires template structure
- 7B-13B: Needs tight constraints + validation

### Location Generation
**Location**: [[prompts/generation/location-generation]]

**Template Variables**:
```
{{LOCATION_TYPE}} - dungeon|city|wilderness|building
{{PARENT_REGION}} - Containing area
{{SETTING}} - Fantasy|Sci-fi|Modern|etc
{{ATMOSPHERE}} - Dark|cheerful|mysterious|etc
{{PURPOSE}} - Story function (quest hub, danger zone, etc)
```

**Output Format**: JSON with sections:
- Basic info (name, type, parent)
- Description and atmosphere
- Features (3-5 notable elements)
- Hooks (quest opportunities)
- Stats (size, wealth, security)
- Mechanics (traps, enemies, treasure)

**Generation Patterns**:
- Top-down: Region → District → Location
- Bottom-up: Location → District → Region
- Context inheritance: Child locations inherit parent themes

### Template Meta-Generation
**Location**: [[prompts/generation/template-generation]]

**Meta-Template**: Prompts that generate other templates

```
Generate a JSON template for {{CONTENT_TYPE}} in {{SETTING}}.

The template should:
1. Define all required fields with descriptions
2. Include {{VARIABLES}} in double braces
3. Specify validation rules
4. Provide 1-2 examples
5. Note model requirements (what works on small vs large models)

Output as JSON schema format.
```

**Use Cases**:
- Item generation templates
- Quest generation templates
- Event generation templates
- Dialogue tree templates

**Production Usage**: ReallmCraft uses this for JITG (Just-In-Time Generation) systems

## Narration Templates

### NDL-to-Narrative
**Location**: [[prompts/narration/ndl-to-narrative]]

**Core Template**:
```
Convert the following NDL markup to natural narrative prose:

{{NDL_MARKUP}}

Context:
- Location: {{LOCATION}}
- Time: {{TIME}}
- Nearby: {{ENTITIES}}

Requirements:
- Describe ONLY what the NDL markup specifies
- Do not add actions or events not in the markup
- Use {{TONE}} tone
- Keep to {{MAX_LENGTH}} words

Write the narrative:
```

**NDL Construct Patterns**:
- `do($entity, "action")` → "Entity performs action"
- `do() ~ "manner"` → "Entity performs action in [manner] way"
- `intention="intent"` → Adds subtext/motivation
- `result("outcome")` → Explicitly state result
- `->` sequencing → Chronological narration

### Scene Description
**Location**: [[prompts/narration/scene-description]]

**Template**:
```
Describe {{LOCATION_NAME}} as the player enters.

Location type: {{TYPE}}
Atmosphere: {{ATMOSPHERE}}
Notable features: {{FEATURES}}
Time of day: {{TIME}}

Include:
- Initial sensory impression (sight, sound, smell)
- 2-3 notable features that stand out
- Mood and atmosphere
- Potential points of interest

Do NOT include:
- Actions the player takes
- NPC dialogue
- Items not in the feature list

Length: {{MIN_LENGTH}}-{{MAX_LENGTH}} words
```

### Action Narration
**Location**: [[prompts/narration/action-narration]]

**Template**:
```
Narrate this action:

Action: {{ACTION_TYPE}}
Actor: {{ENTITY}}
Target: {{TARGET}}
Result: {{SUCCESS|FAILURE}}
Manner: {{MANNER}}

Context:
- Location: {{LOCATION}}
- Recent events: {{HISTORY}}

Describe the action with:
- How it's performed (the manner)
- The immediate result
- Relevant sensory details

Length: {{LENGTH}} words
```

### Dialogue Generation
**Location**: [[prompts/narration/dialogue-generation]]

**Template**:
```
Generate dialogue for {{CHARACTER_NAME}}.

Context:
- Character personality: {{PERSONALITY_TRAITS}}
- Current emotion: {{EMOTION}}
- Speaking to: {{TARGET}}
- Topic: {{TOPIC}}
- Relationship: {{RELATIONSHIP_LEVEL}}

The dialogue should:
- Match character personality
- Reflect current emotion
- Be appropriate to relationship level
- Include subtext if {{INCLUDE_SUBTEXT}}

Format: "Dialogue here" [optional action beat]
```

**NDL Format**:
```ndl
talk("Speaker", "Listener") -> convey("message", perspective="viewpoint")
```

### Combat Narration
**Location**: [[prompts/narration/combat-narration]]

**Template**:
```
Narrate this combat action:

Attacker: {{ATTACKER}}
Defender: {{DEFENDER}}
Action: {{ATTACK_TYPE}}
Result: {{HIT|MISS|CRITICAL}}
Damage: {{DAMAGE_VALUE}}
Weapon: {{WEAPON}}

Describe:
- The attack attempt (manner and style)
- The result (hit/miss)
- Damage dealt if hit
- Defender's reaction

Keep to {{MAX_LENGTH}} words.
Use {{TONE}} tone (gritty|heroic|tactical).
```

**Combat NDL Pattern**:
```ndl
do($attacker, "attack") ~ "manner"
  -> target($defender)
  -> result("hit")
  -> damage(8)
```

## Action Templates

### Social Action Catalog
**Location**: [[ndl/patterns/social-dynamics]]

**43+ Social Actions** organized by category:

**Greeting (8 actions)**:
- wave, nod, bow, salute, greet, introduce, acknowledge, hail

**Conversation (12 actions)**:
- ask, tell, explain, inquire, question, answer, discuss, debate, argue, gossip, chat, converse

**Persuasion (8 actions)**:
- convince, persuade, bargain, negotiate, intimidate, threaten, bribe, coerce

**Relationship (15+ actions)**:
- befriend, flirt, compliment, insult, praise, criticize, apologize, forgive, trust, betray, console, comfort, encourage, discourage, mock

**Template Format**:
```ndl
do($entity, "{{ACTION}}") ~ "{{MANNER}}"
  -> target($target)
  -> intention="{{INTENT}}"
  -> result("{{OUTCOME}}")
```

### Environmental Interaction
**Location**: [[ndl/patterns/environmental-description]]

**Search Template**:
```ndl
search("{{OBJECT}}", intention="{{GOAL}}")
  -> result("found"|"not_found")
  -> describe("{{WHAT_WAS_FOUND}}")
```

**Examples**:
- `search("pile of wood", intention="find makeshift club")`
- `search("desk drawers", intention="find key")`
- `search("bookshelf", intention="research topic")`

### Scene Transition Templates
**Location**: [[ndl/patterns/scene-transitions]]

**14 Scene Mode Categories**:

1. **Exploration** - Free-form world traversal
2. **Combat** - Turn-based fighting
3. **Dialogue** - Conversation mode
4. **Investigation** - Puzzle-solving
5. **Crafting** - Item creation
6. **Trading** - Commerce
7. **Stealth** - Sneaking
8. **Cutscene** - Non-interactive narrative
9. **Rest** - Camping/recovery
10. **Fast Travel** - Skip traversal
11. **Minigame** - Special mechanics
12. **Dream/Vision** - Abstract sequences
13. **Flashback** - Past events
14. **Social** - Relationship building

**Transition Template**:
```ndl
{{CURRENT_SCENE}}
  -> wait("transition_trigger")
  -> system_response("{{TRANSITION_MESSAGE}}")
  -> {{NEW_SCENE}}
```

## Constraint Templates

### Anti-Hallucination Rules
**Location**: [[prompts/constraint/anti-hallucination]]

**Core Constraint Template**:
```
CRITICAL RULES:
1. You are a {{GAME_TYPE}} game. Impossible actions must fail.
2. Players cannot use abilities they don't have
3. Items not in inventory cannot be used
4. NPCs not in scene cannot be interacted with
5. Locations not connected cannot be traveled to

When action is impossible:
- State why it fails
- Describe the failed attempt
- Do NOT make up abilities/items/NPCs to make it work
```

**NDL Failure Pattern**:
```ndl
do($entity, "{{IMPOSSIBLE_ACTION}}")
  -> result("failure")
  -> system_response("{{WHY_IT_FAILED}}")
  -> describe("{{FAILED_ATTEMPT_NARRATION}}")
```

### Format Enforcement
**Location**: [[prompts/constraint/format-enforcement]]

**Bracketed Output Template**:
```
Extract {{INFORMATION_TYPE}} from the context.

Format your response as:
[{{ITEM1}}],[{{ITEM2}}],[{{ITEM3}}]

Do NOT include any other text.
Do NOT explain your choices.
ONLY output the bracketed items.
```

**Examples**:
- Items: `[sword],[potion],[rope]`
- Locations: `[LOCATION: tavern]`
- Binary: `[YES]` or `[NO]`
- Actions: `[ACTION: attack],[ACTION: defend]`

### Length Limiting
**Location**: [[prompts/constraint/length-limiting]]

**170-Token Technique** (ChatBot RPG):
```
{{YOUR_INSTRUCTIONS}}

CRITICAL: Your response must be EXACTLY 170 tokens or less.
Token budget: 170 tokens maximum.

Write your response now (170 tokens max):
```

**Implementation**:
1. Set API `max_tokens=170`
2. Add prompt instruction
3. Regex post-processing to trim at sentence boundary if needed

**Why 170?**: Front-loaded coherence + forces conciseness + 40% cost savings

## Reasoning Templates

### Chain of Thought
**Location**: [[prompts/reasoning/chain-of-thought]]

**Template**:
```
Task: {{TASK_DESCRIPTION}}

Think step-by-step:
1. First, identify {{ASPECT_1}}
2. Then, consider {{ASPECT_2}}
3. Next, evaluate {{ASPECT_3}}
4. Finally, decide {{DECISION}}

Your reasoning:
```

**Function Keyword Pattern** (from 50h100a):
```python
def generate_response():
    """Think through the problem step by step before responding"""
    # Step 1: Analyze context
    # Step 2: Identify constraints
    # Step 3: Generate options
    # Step 4: Select best option
    return response
```

### Binary Classification
**Location**: [[prompts/reasoning/binary-classification]]

**Question Tree Template**:
```
Answer the following questions to determine {{GOAL}}:

Q1: {{QUESTION_1}}
A1: [YES] or [NO]

Q2: {{QUESTION_2}}
A2: [YES] or [NO]

Q3: {{QUESTION_3}}
A3: [YES] or [NO]

Based on your answers:
- If Q1=YES and Q2=YES → {{OUTCOME_A}}
- If Q1=YES and Q2=NO → {{OUTCOME_B}}
- If Q1=NO → {{OUTCOME_C}}

Final determination: [{{ANSWER}}]
```

**Production Usage**: ChatBot RPG uses this for state extraction from player input (95%+ accuracy)

## Retrieval Templates

### HyDE Query Formulation
**Location**: [[prompts/retrieval/query-formulation-hyde]]

**Hypothetical Document Template**:
```
Generate 3-5 hypothetical questions that content about "{{TOPIC}}" might answer.

Topic: {{TOPIC}}
Context: {{CONTEXT}}

Format as JSON:
{
  "questions": [
    "What is {{ASPECT_1}}?",
    "How does {{ASPECT_2}} work?",
    "Why would someone {{ASPECT_3}}?"
  ],
  "key_terms": ["term1", "term2", "term3"]
}
```

**Improvement**: 40-80% better recall vs. standard semantic search

## System Prompts

### Narration Engine System Prompt
**Location**: [[prompts/system/narration-engine-system]]

**Base System Template**:
```
You are the narrator for {{GAME_NAME}}, a {{GENRE}} game.

Your role:
- Convert game events (provided as NDL markup) into natural narrative prose
- Describe ONLY what the game logic has determined happened
- Do NOT make decisions about success/failure of actions
- Do NOT add events not specified in the NDL markup
- Do NOT introduce new items, NPCs, or locations not in the context

Tone: {{TONE}}
Style: {{STYLE}}
Target length: {{LENGTH_GUIDELINE}}

The game will provide:
1. NDL markup describing what happened
2. Context (location, entities, history)
3. Any special instructions

Your job is to narrate, not decide.
```

## Few-Shot Example Templates

### Few-Shot Format
**Location**: [[prompts/techniques/few-shot-examples]]

**General Template**:
```
Here are examples of the desired format:

Example 1:
Input: {{INPUT_1}}
Output: {{OUTPUT_1}}

Example 2:
Input: {{INPUT_2}}
Output: {{OUTPUT_2}}

Example 3:
Input: {{INPUT_3}}
Output: {{OUTPUT_3}}

Now process this:
Input: {{ACTUAL_INPUT}}
Output:
```

**Best Practices**:
- 1-3 examples (more isn't better)
- Examples should cover edge cases
- Show failure cases too
- Use diverse examples

## Pattern-Specific Templates

### Just-In-Time Generation
**Location**: [[patterns/generation/jit-generation]]

**Lazy Generation Template**:
```python
def get_location_description(location_id: str) -> str:
    """Generate location description only when first visited"""

    if location_id in cache:
        return cache[location_id]

    # Generate on-demand
    prompt = f"""
    Generate a {{LOCATION_TYPE}} with ID {location_id}
    Parent region: {{PARENT}}
    Theme: {{THEME}}
    [Template continues...]
    """

    generated = llm.generate(prompt)
    cache[location_id] = generated
    return generated
```

### Hierarchical Cascade
**Location**: [[patterns/generation/hierarchical-cascade]]

**Top-Down Template**:
```
1. Generate world/region overview
2. For each region, generate districts/areas (using region context)
3. For each area, generate specific locations (using area context)
4. For each location, generate NPCs (using location context)
5. For each NPC, generate dialogue/quests (using NPC context)
```

Each level uses previous level's output as context/constraints.

## Usage Guidelines

### Selecting Templates

**For Narration**: Use NDL-based templates (most reliable)
**For Generation**: Use structured JSON templates
**For Constraints**: Use rule-based templates
**For Small Models**: Use tighter constraints + few-shot examples
**For Large Models**: Can work with just instructions

### Template Variables

Standard variable naming:
- `{{UPPERCASE}}` - User-provided parameters
- `{{lowercase}}` - Generated/derived values
- `{{PascalCase}}` - Type/class names

### Model-Specific Adaptations

**GPT-4 / Claude Opus**:
- Can work with instructions alone
- Few-shot optional
- Creative freedom OK

**GPT-3.5 / Claude Sonnet**:
- Needs template structure
- 1-2 examples helpful
- Moderate constraints

**7B-13B Local Models**:
- Requires tight constraints
- 2-3 examples essential
- Validation mandatory
- Shorter prompts better

## Related Documentation

- [[prompts/00-PROMPT-INDEX]] - Full prompt library
- [[patterns/00-PATTERN-INDEX]] - Architectural patterns
- [[ndl/00-NDL-INDEX]] - NDL language specification
- [[schemas-index]] - Data structure reference

## Contributing

When adding new templates:
1. Document in context (within relevant prompt/pattern file)
2. Add reference to this index
3. Provide complete working example
4. Test on target model(s)
5. Note effectiveness and limitations
6. Include variable definitions

---

**Last Updated**: 2026-01-17
**Coverage**: Complete (all identifiable templates from 24 months of Discord discussions)
**Source**: LLM World Engine Discord (Jan 2024 - Dec 2025)
**Production Tested**: ReallmCraft, ChatBot RPG, DirectorAPI
