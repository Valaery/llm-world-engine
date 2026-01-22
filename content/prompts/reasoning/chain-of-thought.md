---
tags: [prompt, reasoning, cot, technique]
category: reasoning
technique: chain-of-thought
model-tested: [all-models]
contributor: 50h100a
status: proven
---

# Prompt: Chain of Thought (CoT)

## Metadata
- **Category**: Reasoning
- **Technique**: Chain of Thought
- **Model Tested**: All models (especially beneficial for smaller models)
- **Contributor**: [[User-50h100a]], community adoption
- **Status**: Proven technique

## Purpose

Force the LLM to "think through" a response step-by-step before generating the final output. This reduces impulsive/random responses, improves consistency, and helps the model consider context more carefully.

Particularly useful for:
- State-aware narration
- Validation tasks
- Multi-step reasoning
- Reducing hallucinations

## Core Concept

> "You just tell the llm to start every response with x,y, and z. It'll 'think' for itself." - [[User-underscore_x]]

The LLM naturally wants to "fill in the blank." By structuring prompts with incomplete patterns, you force the model to complete them before continuing, creating an implicit reasoning chain.

## Template

### Basic CoT Pattern
```
Task: {{TASK_DESCRIPTION}}

Before responding, consider:
1. {{CONSIDERATION_1}}
2. {{CONSIDERATION_2}}
3. {{CONSIDERATION_3}}

Now, {{FINAL_INSTRUCTION}}
```

### Implicit CoT (Fill in the Blank)
```
Character: {{CHARACTER_NAME}}
Location: {{LOCATION}}
Mood: {{MOOD}}
What {{CHARACTER_NAME}} wants: {{GOAL}}
What {{CHARACTER_NAME}} will say:
```

The prompt ends here, forcing the LLM to complete the pattern.

## Variables

| Variable | Description | Example |
|----------|-------------|---------|
| {{TASK_DESCRIPTION}} | What you need the LLM to do | "Narrate the character's action" |
| {{CONSIDERATION_N}} | Step in the reasoning chain | "Check if character has the ability" |
| {{CHARACTER_NAME}} | Name of character | "Bob" |
| {{LOCATION}} | Current location | "Tavern" |
| {{MOOD}} | Character's emotional state | "Nervous" |
| {{GOAL}} | Character's current intention | "Information about the quest" |
| {{FINAL_INSTRUCTION}} | What to output | "write the dialogue" |

## Usage Example

### Example 1: Statblock-Based Narration

**Prompt**:
```
Character: Theron the Warrior
Location: Dark Forest
Current Health: 45/100
Stamina: Exhausted
Mood: Anxious
Recent Events: Just escaped from bandits, wounded
Equipment: Sword (damaged), leather armor
Goal: Find shelter before nightfall

What Theron will do next:
```

**Output**:
```
Theron scans the treeline desperately, looking for any sign of refuge. His wounded leg throbs with each step, but fear of pursuit keeps him moving. He needs shelter, and fast.
```

### Example 2: Action Validation with CoT

**Prompt**:
```
Player attempts: "I cast a fireball spell"

Character Sheet:
- Class: Warrior
- Abilities: Sword proficiency, shield bash
- Magic abilities: None

Before responding, check:
1. Does the character have this ability? [No, warrior with no magic]
2. Is this action possible given their abilities? [No]
3. What is the appropriate outcome? [Action fails, explain why]

Narrate the outcome:
```

**Output**:
```
You raise your hands and shout dramatic words, but nothing happens. You're a warrior, not a mage - you have no magical training or innate spellcasting ability.
```

### Example 3: NPC Dialogue with CoT

**Prompt**:
```
NPC: Marcus the Innkeeper
Personality: Cautious, protective of regular customers
Current situation: Stranger (player) asking about local bandits
What Marcus knows: The bandits are dangerous, his daughter was threatened
What Marcus thinks: This stranger might be trouble, or might help
Marcus's goal: Assess if player can be trusted

What Marcus is thinking:
[CoT space for model to reason]

What Marcus says:
```

**Output**:
```
What Marcus is thinking:
This one's got the look of a fighter... Could be another bandit, or could be someone who can help. Best to be careful. I'll hint at the problem without saying too much.

What Marcus says:
"Bandits? Aye, they're around. Most folk here learned to keep their heads down and their doors locked." He studies you carefully, wiping the same spot on the bar twice.
```

## Effectiveness Notes

### What Works Well
- ✅ Reduces impulsive/random responses
- ✅ Forces consideration of context
- ✅ Especially helpful for smaller models (7B-9B)
- ✅ Creates implicit "reasoning" without explicit instruction
- ✅ Improves consistency across turns
- ✅ Natural way to structure prompts

### Known Limitations
- ⚠️ Slower generation (more tokens to produce)
- ⚠️ Increases token usage
- ⚠️ Can be verbose if not constrained
- ⚠️ Requires careful prompt structure
- ⚠️ Some models might skip the reasoning step

### Community Feedback

**50h100a**: "CoT with external state tracking, essentially"

**monkeyrithms**: "One-shotting combat doesn't work too well unless it's GPT-4 in my experience, but a CoT type thing does"

**appl2613**: "LLMs are super cliche and shallow on their own devices... but so would we if we just one-shot everything"

## CoT Patterns

### 1. Explicit Reasoning Steps
```
Before narrating, answer:
1. What is the current situation?
2. What is the character trying to do?
3. Is this action possible?
4. What is the most likely outcome?

Now narrate:
```

### 2. State-Checking Pattern
```
Current State:
- Health: {{HEALTH}}
- Location: {{LOCATION}}
- Equipment: {{EQUIPMENT}}
- Recent events: {{EVENTS}}

Given this state, describe what happens next:
```

### 3. Three-Question Framework (TTRPG-Inspired)
```
Action: {{PLAYER_INPUT}}

What (are they doing):
How (are they doing it):
Why (are they doing it):

Based on these, narrate the outcome:
```

This is the foundation of NDL's design philosophy.

### 4. Validation Chain
```
Player action: {{ACTION}}

Validation:
- Does character have required ability? {{CHECK_ABILITY}}
- Does character have required item? {{CHECK_ITEM}}
- Is action physically possible? {{CHECK_PHYSICS}}
- Does action follow game rules? {{CHECK_RULES}}

Result:
```

### 5. NPC Response Chain
```
NPC: {{NPC_NAME}}
Player said: {{PLAYER_DIALOGUE}}

NPC's internal reaction:
[What they actually think]

NPC's external response:
[What they choose to say]
```

Creates natural subtext in dialogue.

## Integration with Other Techniques

### CoT + NDL
```
NDL Events:
do($"player", "persuade")~"honeyed words"
->target($"guard")
->roll("charisma", result="failure")

Before narrating, consider:
- The player attempted persuasion
- They used a charming approach
- The dice roll failed
- How would the guard realistically react?

Narrate the outcome:
```

### CoT + Constraints
```
Player action: "I jump to the rooftop"

Constraint check:
- Character sheet: Normal human, no special abilities
- Physics: Rooftop is 15 feet high
- Conclusion: Impossible without equipment

Narrate the failed attempt:
```

### CoT + Few-Shot
```
Example 1:
Action: "I search the desk"
What I know: Desk has a hidden compartment
Perception check: Success
Outcome: [Narrate finding the compartment]

Example 2:
Action: "I search the desk"
What I know: Desk has a hidden compartment
Perception check: Failure
Outcome: [Narrate not finding anything unusual]

Now apply this pattern:
Action: {{PLAYER_ACTION}}
What I know: {{WORLD_STATE}}
Relevant check: {{CHECK_RESULT}}
Outcome:
```

## Temperature Settings

For CoT prompts:
- **Reasoning portion**: 0.3 - 0.5 (want logical, consistent thinking)
- **Narration portion**: 0.6 - 0.9 (want creative prose)

Some models allow setting different temperatures for different parts of generation.

## Advanced: Hidden CoT

You can use CoT internally without showing it to the player:

```python
# Generate with CoT
full_prompt = f"""
Character state: {state}
Action: {action}

Internal reasoning:
[Let model think here]

Player-facing narration:
"""

response = llm.generate(full_prompt)

# Parse and only show narration part
narration = extract_section(response, "Player-facing narration:")
return narration
```

This gives you CoT benefits without verbose output.

## When to Use CoT

### Good Use Cases:
- ✅ Complex decisions with multiple factors
- ✅ State validation
- ✅ NPC dialogue (thinking vs. saying)
- ✅ Action resolution with multiple checks
- ✅ Smaller models that need guidance
- ✅ Situations requiring consistency

### Poor Use Cases:
- ❌ Simple, straightforward narration
- ❌ When token budget is extremely tight
- ❌ Real-time responses (CoT is slower)
- ❌ When you're using NDL (already structured)

## Debugging with CoT

CoT makes debugging easier because you can see the model's "reasoning":

```
Action: {{PLAYER_ACTION}}

Debug trace:
1. Parsed action as: [parse]
2. Checked against abilities: [check]
3. Determined outcome: [outcome]
4. Reason: [explanation]

Narration:
```

This helps identify where things go wrong.

## Related Prompts
- [[statblock-cot]] - Specific pattern for state-aware narration
- [[ndl-to-narrative]] - Alternative that structures input instead of thinking
- [[query-formulation]] - CoT for RAG queries

## Source
- Discussion context: Early 2024 architecture discussions
- Key contributor: 50h100a's statblock experiments
- Referenced in: [[02-Prompt-Engineering]]
- Also called: "CoT with external state tracking"
