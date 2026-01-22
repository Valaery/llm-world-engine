---
tags: [prompt, reasoning, validation, parsing, binary-classification]
category: reasoning
technique: binary-classification
model-tested: [gpt-3.5, llama-3-8b, tinyllama, mistral-7b]
contributor: monkeyrithms
status: proven
---

# Prompt: Binary Classification (Yes/No Validation)

## Metadata
- **Category**: Reasoning
- **Technique**: Binary classification with CoT
- **Model Tested**: GPT-3.5, Llama 3 (8B), TinyLlama, Mistral 7B, various local models
- **Contributor**: [[User-monkeyrithms]], [[User-vali98]]
- **Status**: Proven in production (ChatBot RPG game engine)

## Purpose

Extract binary decisions from natural language input to drive game logic. Used for validating player actions, detecting intent, and triggering programmatic responses. This pattern enables deterministic game state management while using LLMs for flexible natural language understanding.

**Key Innovation**: Combines human-readable questions with machine-parseable answers. The LLM explains its reasoning (helpful for debugging) but commits to a binary [YES]/[NO] response that code can reliably parse.

## Template

```
{{input_context}}

Is {{condition_to_check}}? Answer with [YES] or [NO]. Explain your answer step-by-step.
```

### Grammar-Constrained Version
For backends that support grammar constraints (koboldcpp, llama.cpp):

```
Grammar: root::= "[YES]" | "[NO]"

{{input_context}}

Is {{condition_to_check}}?
```

## Variables

| Variable | Description | Example |
|----------|-------------|---------|
| {{input_context}} | The text to analyze | Player's action, dialogue, game log excerpt |
| {{condition_to_check}} | The binary question to answer | "player interacting with an object", "quest completed", "combat initiated" |

## Usage Examples

### Example 1: Object Interaction Detection

**Input:**
```
{{{INPUT}}}
"Hmm, that looks good. I think I want to eat that apple." Player eats the apple.

Is player interacting with an object right now? Answer with [YES] or [NO]. Explain your answer step-by-step.
```

**Expected Output:**
```
{{{OUTPUT}}}
[YES]: Yes, the player is interacting with an object. The text explicitly states "Player eats the apple," which is a direct interaction with the object "apple". The player is performing the action of eating, which requires physical interaction with the food item.
```

**Code Parsing:**
```python
def parse_binary_response(llm_output: str) -> bool:
    """Extract YES/NO from LLM response."""
    if llm_output.strip().startswith("[YES]"):
        return True
    elif llm_output.strip().startswith("[NO]"):
        return False
    else:
        # Fallback regex
        import re
        yes_match = re.search(r'\[YES\]', llm_output, re.IGNORECASE)
        if yes_match:
            return True
        return False

response = llm.generate(prompt)
is_interacting = parse_binary_response(response)

if is_interacting:
    # Extract item name with follow-up prompt
    item_name = extract_item(context)
    game_state.use_item(item_name)
```

### Example 2: Item Extraction

**Input:**
```
{{{INPUT}}}
"Hmm, that looks good. I think I want to eat that apple." Player eats the apple.

What is the item the Player is using? Answer with the item inside square brackets.
```

**Expected Output:**
```
{{{OUTPUT}}}
[apple]
```

### Example 3: Quest Completion Check

**Input:**
```
{{{INPUT}}}
Player: "Here is the ancient artifact you requested, wise elder."
Old Man: "Ah, thank you brave adventurer! You have saved our village."

Player hands over the Amulet of Zephyr to the Old Man.

Did the player return the artifact to the old man? Answer with [YES] or [NO].
```

**Expected Output:**
```
{{{OUTPUT}}}
[YES]: The player explicitly handed over the Amulet of Zephyr to the Old Man, which is the artifact requested. The Old Man's response confirms receipt.
```

**Code Integration:**
```python
if check_quest_completion(game_log, "return_artifact_to_elder"):
    game_state.complete_quest("village_savior")
    game_state.advance_quest_stage("main_story", stage=2)
```

### Example 4: Combat Detection

**Input:**
```
{{{INPUT}}}
Player: "I draw my sword and charge at the bandit!"
The player rushes forward with weapon raised.

Has combat been initiated? Answer with [YES] or [NO].
```

**Expected Output:**
```
{{{OUTPUT}}}
[YES]: Combat has been initiated. The player has drawn a weapon and is performing an aggressive action ("charge at the bandit"), which are clear indicators of entering combat.
```

## Effectiveness Notes

### What Works Well

1. **[YES]/[NO] Brackets**: Makes parsing trivial with `startswith()` checks
2. **"Explain step-by-step" suffix**: Improves accuracy on smaller models via Chain of Thought
3. **Token limiting**: Restrict to 10-50 tokens to prevent rambling while preserving reasoning
4. **Grammar constraints**: Forces exact `[YES]` or `[NO]` output on compatible backends
5. **Simple questions work best**: Binary, unambiguous conditions
6. **Context isolation**: Feed only relevant text (not entire game history)

### Known Limitations

1. **False positives on intent**: LLMs may answer YES if player *intends* to do something but hasn't done it yet
   - **Solution**: Exclude quoted dialogue, focus on action descriptions only

2. **Complex multi-part questions fail**: "Did player drop item AND pick up new item?"
   - **Solution**: Break into separate binary questions

3. **Negation confusion**: "Did player NOT eat the apple?" is harder than "Did player eat the apple?"
   - **Solution**: Frame questions positively

4. **Model-specific quirks**: Some questions are inexplicably hard for certain models
   - Example: "Did player drop item?" consistently fails on Llama 2 7B
   - **Solution**: Test each question, route difficult ones to stronger models

### The Question Tree Pattern

From [[User-monkeyrithms]]'s architecture:

Binary questions form a decision tree:
```
Q1: "Is player interacting with object?" → [YES]
  ↓
Q2: "What item?" → [apple]
  ↓
Q3: "Is [apple] in inventory?" → Check game state
  ↓
If YES: Remove item, update hunger status
If NO: Generate "You don't have that item" response
```

Each question is self-contained. The program parses answers and decides which question to ask next.

### Model-Specific Performance

| Model | Accuracy | Notes |
|-------|----------|-------|
| GPT-3.5 | 99%+ | Never fails, can handle complex questions |
| GPT-4 | 99%+ | Overkill for this task, use for complex reasoning |
| Llama 3 (8B) | 90-95% | Very good with clear questions |
| Mistral 7B | 85-90% | Good balance of speed and accuracy |
| TinyLlama | 70-80% | Works for simple questions, unreliable for complex |
| Llama 2 (7B) | 60-80% | Inconsistent, avoid for production |

## Variations

### Minimal (Token-Efficient)
```
{{context}}

{{question}}? [YES] or [NO]:
```

Output: `[YES]` or `[NO]` only (no explanation)

### Confidence Scoring
```
{{context}}

{{question}}? Answer format:
[YES/NO]: [Confidence 0-100%]: [Explanation]
```

Output: `[YES]: 95%: The text clearly states...`

### Multi-Choice (Beyond Binary)
```
{{context}}

What action is the player performing?
[ATTACK] [DEFEND] [MOVE] [INTERACT] [DIALOGUE] [NONE]
```

### Grammar-Constrained Item Extraction
```
Grammar: root::= "[" item_name "]"
         item_name::= [a-z]+

What item is the player using?
```

Forces output like `[apple]`, `[sword]`, etc.

## Integration Patterns

### Multi-Model Routing

From [[User-monkeyrithms]]'s approach:

```python
# Easy questions → Fast local model
if question_type == "object_interaction":
    response = local_model_7b.generate(prompt, max_tokens=10)

# Hard questions → Smarter model
elif question_type == "quest_logic":
    response = gpt35.generate(prompt, max_tokens=50)

# Creative narration → Creative model
elif question_type == "narration":
    response = hathor_13b.generate(prompt, max_tokens=200, temperature=0.9)
```

### Validation Chain

```python
def validate_player_action(player_input: str, game_state: GameState) -> ActionResult:
    """Multi-step validation using binary classification."""

    # Step 1: Is this an action or just dialogue?
    is_action = classify_binary(
        f"{player_input}\n\nIs this a game action? [YES/NO]"
    )
    if not is_action:
        return ActionResult(type="dialogue", valid=True)

    # Step 2: Does action involve an item?
    uses_item = classify_binary(
        f"{player_input}\n\nDoes this action use an item? [YES/NO]"
    )

    if uses_item:
        # Step 3: Extract item name
        item_name = extract_item(player_input)

        # Step 4: Validate inventory
        if not game_state.player.has_item(item_name):
            return ActionResult(
                type="action",
                valid=False,
                error=f"You don't have {item_name}"
            )

    return ActionResult(type="action", valid=True, item=item_name)
```

## Temperature Settings

- **Recommended**: 0.1 - 0.3 (low temperature for deterministic binary output)
- **With CoT explanation**: 0.3 - 0.5 (allow some reasoning variation)
- **Grammar-constrained**: 0.0 (temperature irrelevant when output is forced)

## Testing Checklist

Test your binary classification prompts:

- [ ] Answers consistently start with `[YES]` or `[NO]`
- [ ] False positives: Doesn't confuse intent with action
- [ ] False negatives: Catches all valid cases
- [ ] Handles edge cases (empty input, ambiguous actions)
- [ ] Works on target model (test small models separately)
- [ ] Parsing code handles malformed responses gracefully
- [ ] Token limit prevents excessive output
- [ ] Questions are unambiguous and binary

## Performance Metrics

From ChatBot RPG production use:

- **Accuracy**: 95%+ with GPT-3.5, 85-90% with Llama 3 8B
- **Latency**: 50-200ms per classification (local 7B models)
- **Token cost**: 10-50 tokens per question (with explanation)
- **Token cost (grammar)**: 1-5 tokens (forced binary output)

## Related Prompts

- [[format-enforcement]] - General structured output
- [[chain-of-thought]] - Reasoning patterns
- [[action-narration]] - What happens after validation succeeds
- [[anti-hallucination]] - Prevents inventing false states

## Source

**Discussion context**: Lines 1330-1398 of transcript. [[User-monkeyrithms]] explaining the "question tree" approach used in ChatBot RPG to extract game state from natural language.

**Key insight from monkeyrithms**:
> "So its a combination of function calls, and being clever about prompting. What I mean by that last part is you really have to think everything through... asking the object/apple question would likely result in a false positive, especially with smaller 'dumber' models, because the context suggests that the intent might be to eat an apple. So in this case, you have to check all the text that is not the enclosed-quotes dialogue."

**On model quirks**:
> "What I've found is that the questions models find easier/harder to answer, are not the same as what -we- find easier or harder to answer. There's a lot of quirks you just have to work with. Like I have one question with multiple possible answers and multiple moving parts to it, so you'd think that'd be harder for a model to get right, but even the little ones do that one well. Then I have another question, 'did the player drop an item,' and for some unknown reason, LLMs just really really struggle with that one."

**Screenshot**: `Media/image-D3FB9.png` shows TinyLlama successfully handling binary classification with step-by-step explanation.

**Related threads**: [[01-Architecture-and-Design]], [[02-Prompt-Engineering]]

## Best Practices

### DO:
- ✅ Use `[YES]` and `[NO]` with square brackets for easy parsing
- ✅ Ask for step-by-step explanation to improve accuracy (CoT)
- ✅ Limit tokens (10-50 range) to prevent rambling
- ✅ Frame questions positively ("Did X happen?" not "Did X not happen?")
- ✅ Test each question individually on target model
- ✅ Isolate relevant context (don't feed entire chat history)
- ✅ Use grammar constraints when available
- ✅ Route hard questions to stronger models

### DON'T:
- ❌ Ask multi-part questions ("Did X and Y happen?")
- ❌ Include player dialogue in action detection context
- ❌ Use Yes/No without brackets (harder to parse)
- ❌ Assume all models handle all questions equally
- ❌ Trust high temperature outputs
- ❌ Skip testing on production model
- ❌ Parse without fallback for malformed output

## Code Example: Complete Implementation

```python
from typing import Literal, Optional
import re

class BinaryClassifier:
    """LLM-based binary classification for game state validation."""

    def __init__(self, model, use_grammar: bool = False):
        self.model = model
        self.use_grammar = use_grammar

    def classify(
        self,
        context: str,
        question: str,
        explain: bool = True,
        max_tokens: int = 50
    ) -> tuple[bool, Optional[str]]:
        """
        Perform binary classification.

        Returns:
            (result: bool, explanation: Optional[str])
        """
        if self.use_grammar:
            prompt = f'{context}\n\n{question}?'
            grammar = 'root::= "[YES]" | "[NO]"'
            response = self.model.generate(
                prompt,
                grammar=grammar,
                max_tokens=10,
                temperature=0.0
            )
        else:
            explain_suffix = " Explain your answer step-by-step." if explain else ""
            prompt = f'{context}\n\n{question}? Answer with [YES] or [NO].{explain_suffix}'
            response = self.model.generate(
                prompt,
                max_tokens=max_tokens,
                temperature=0.2
            )

        # Parse response
        result = self._parse_binary(response)
        explanation = self._extract_explanation(response) if explain else None

        return result, explanation

    def _parse_binary(self, response: str) -> bool:
        """Extract YES/NO from response."""
        response = response.strip()

        # Direct prefix match (fastest)
        if response.startswith("[YES]"):
            return True
        if response.startswith("[NO]"):
            return False

        # Regex fallback
        if re.search(r'\[YES\]', response, re.IGNORECASE):
            return True
        if re.search(r'\[NO\]', response, re.IGNORECASE):
            return False

        # Default to False if unparseable
        return False

    def _extract_explanation(self, response: str) -> Optional[str]:
        """Extract explanation after [YES]/[NO]."""
        match = re.search(r'\[(YES|NO)\]:?\s*(.+)', response, re.IGNORECASE | re.DOTALL)
        if match:
            return match.group(2).strip()
        return None

# Usage
classifier = BinaryClassifier(model=llm, use_grammar=False)

player_input = '"That apple looks tasty." Player eats the apple.'
is_action, explanation = classifier.classify(
    context=player_input,
    question="Is player interacting with an object right now",
    explain=True
)

if is_action:
    print(f"Action detected: {explanation}")
    # Follow-up: extract item name
    item, _ = classifier.classify(
        context=player_input,
        question="What is the item the Player is using? Answer with the item inside square brackets"
    )
```

## Advanced: Question Tree Framework

```python
from dataclasses import dataclass
from typing import Callable, Optional

@dataclass
class Question:
    """A node in the validation question tree."""
    prompt: str
    on_yes: Optional['Question'] = None
    on_no: Optional['Question'] = None
    action_yes: Optional[Callable] = None
    action_no: Optional[Callable] = None

# Define question tree
q_interact = Question(
    prompt="Is player interacting with an object?",
    on_yes=Question(
        prompt="What is the item? Answer with [item_name]",
        action_yes=lambda item: game.use_item(item)
    ),
    action_no=lambda: game.generate_generic_narration()
)

# Execute tree
def execute_question_tree(root: Question, context: str, classifier: BinaryClassifier):
    """Traverse question tree based on LLM answers."""
    current = root

    while current:
        result, explanation = classifier.classify(context, current.prompt)

        if result:  # [YES]
            if current.action_yes:
                current.action_yes()
            current = current.on_yes
        else:  # [NO]
            if current.action_no:
                current.action_no()
            current = current.on_no
```
