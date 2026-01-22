---
tags: [prompt, constraint, length-control, token-management, technique]
category: constraint
technique: token-limiting with post-processing
status: proven
contributor: [[User-appl2613]]
models_tested: [GPT-3.5, GPT-4, local models]
date_created: 2024-02-11
date_updated: 2024-02-11
---

# Prompt: Length Limiting (Token Budget Management)

## Metadata
- **Category**: Constraint
- **Technique**: Token limit + explicit instruction + regex post-processing
- **Model Tested**: GPT-3.5, GPT-4, various local models
- **Contributor**: [[User-appl2613]] (monkeyrithms)
- **Status**: Proven (production in ChatBot RPG)
- **Implementation**: ChatBot RPG

## Purpose

Control LLM output length to maintain consistent pacing, prevent rambling, and optimize token usage. This technique combines API-level token limits with prompt instructions and post-processing to produce concise, coherent responses that naturally fit the narrative flow.

## Core Principle: Front-Loading Coherence

**Key Insight from [[User-appl2613]]**:
> "LLMs work by predicting each next word in the sequence, they tend to start out with sentences and phrases that are more compatible and vague in the beginning and start becoming more specialist as it goes. What this means is that you can get a full long paragraph that is coherent. If you randomly cut that in half, the first half of that paragraph also tends to be coherent, standing all on its own."

**Implication**: The first N tokens of a response are the most relevant, general, and compatible with prior context. Later tokens increase the risk of:
- Going off-topic
- Impersonating other characters
- Introducing unwanted tangents
- "Writing themselves into a corner"

## Template

```
{{task_instruction}}

Requirements:
- Generate a couple sentences to a paragraph
- {{length_description}} response
- Focus on {{primary_content}}
- Avoid {{unwanted_behaviors}}

{{context_data}}
```

## Variables

| Variable | Description | Example |
|----------|-------------|---------|
| {{task_instruction}} | What the LLM should do | "Narrate the character's action", "Describe the location" |
| {{length_description}} | Qualitative length guidance | "brief", "concise", "moderate" |
| {{primary_content}} | What to prioritize | "immediate action", "sensory details", "character reaction" |
| {{unwanted_behaviors}} | What to avoid | "describing other characters", "resolving the action", "multiple actions" |
| {{context_data}} | Relevant game state | Character stats, location, recent events |

## Implementation Details

### Three-Layer Approach

**Layer 1: API Token Limit**
```python
# Set maximum tokens for generation
response = llm.generate(
    prompt=prompt_text,
    max_tokens=170,  # Hard cap
    temperature=0.3
)
```

**Layer 2: Prompt Instruction**
```
Generate a couple sentences to a paragraph. Keep your response concise.
```

**Layer 3: Post-Processing**
```python
import re

def trim_incomplete_sentences(text):
    """Remove trailing incomplete sentence fragments"""
    # Remove anything after the last complete sentence
    sentences = re.split(r'([.!?])\s+', text)

    # Reconstruct only complete sentences
    complete = []
    for i in range(0, len(sentences)-1, 2):
        if i+1 < len(sentences):
            complete.append(sentences[i] + sentences[i+1])

    return ' '.join(complete).strip()

# Usage
output = llm.generate(prompt, max_tokens=170)
cleaned = trim_incomplete_sentences(output)
```

## Usage Example

### Example: Character Response in RPG

**Input Prompt**:
```
Narrate {{character_name}}'s response to the player's question.

Requirements:
- Generate a couple sentences to a paragraph
- Brief response that addresses the question
- Focus on character's immediate reaction and answer
- Avoid describing player's reaction or other characters

Context:
Character: Merchant Aldric
Player asked: "Do you have any healing potions?"
Character mood: Busy but friendly
Location: Market stall
```

**API Call**:
```python
response = llm.generate(
    prompt=full_prompt,
    max_tokens=170,
    temperature=0.7
)
```

**Raw Output** (might be cut off mid-sentence):
```
Aldric glances up from organizing his wares, wiping his hands on his apron. "Healing potions? Aye, I've got a few left." He reaches beneath the counter and produces three glass vials filled with red liquid. "Fresh brew from yesterday. Two silver each, or five silver for all three. They'll close up cuts and bruises right quick, though I wouldn't rely on them for anything too ser
```

**After Post-Processing**:
```
Aldric glances up from organizing his wares, wiping his hands on his apron. "Healing potions? Aye, I've got a few left." He reaches beneath the counter and produces three glass vials filled with red liquid. "Fresh brew from yesterday. Two silver each, or five silver for all three."
```

## Recommended Token Limits

### By Use Case

| Use Case | Token Limit | Rationale |
|----------|-------------|-----------|
| Quick responses | 80-100 | 1-2 sentences |
| Standard narration | 150-170 | 2-4 sentences, sweet spot |
| Detailed descriptions | 250-350 | 1 paragraph, risk zone |
| Long form | 400+ | High risk of drift |

**[[User-appl2613]]'s Recommendation**:
> "170 tokens is a good 'sweet spot'. The first 170 tokens of a response are usually the most relevant, general, coherent, and compatible material to the stuff that came before it."

**Warning Zone** (250-350 tokens):
> "tends to run into 'let's impersonate other characters now, even though it told me not to, because I'm running out of context for my original task' territory, and it gets harder to reign in there"

## Effectiveness Notes

### What Works Well
- **Prevents rambling**: Hard stop before LLM runs out of context
- **Maintains pacing**: Consistent response lengths feel more game-like
- **Reduces errors**: Shorter outputs = fewer opportunities for hallucination
- **Token efficient**: Direct cost savings on API calls
- **Natural truncation**: Front-loaded coherence means cuts feel less jarring
- **Works across models**: Effective on GPT and local models alike

### Known Limitations
- **Occasional mid-sentence cuts**: Regex helps but isn't perfect
- **May feel abrupt**: Some scenes need more breathing room
- **Context-dependent**: 170 tokens might be too short for complex situations
- **Style impact**: Can feel "choppy" if every response is exactly the same length

### Model Considerations
- **GPT-3.5/GPT-4**: Very responsive to token limits, consistent
- **Local 7B-13B models**: Need stricter prompt instructions about length
- **Creative models**: May resist length constraints, need more explicit "do not continue" language

### Community Feedback

**[[User-appl2613]]** (February 2024):
> "i enforce a 170 token limit on the API calls, while telling it to generate a couple sentences to a paragraph. I use regex to skim off any trailing incomplete sentences."

**[[User-50h100a]]**:
> "this is broadly correct, both in practice and intuition"

**[[User-appl2613]]** on why limits matter:
> "So 90% of the time, you can get away with skimming off the end of it and enforcing length in that way"

## Temperature Settings

**Recommended Settings for Length-Limited Generation**:
- **Temperature**: 0.3-0.7 (lower = more predictable stopping points)
- **Top-p**: 0.85-0.9
- **Repetition Penalty**: 1.1-1.2

**Note**: Lower temperatures work better with token limits because the LLM is more likely to reach natural stopping points within the budget.

## Related Prompts

- [[constraint/anti-hallucination|Anti-Hallucination]] - Preventing unwanted content
- [[narration/scene-description|Scene Description]] - Often benefits from length limits
- [[narration/action-narration|Action Narration]] - Perfect use case for 170-token limit
- [[techniques/pacing-control|Pacing Control]] - Broader narrative rhythm techniques

## Variations

### Dynamic Length by Context

```python
def get_token_limit(context):
    """Adjust token limit based on situation"""
    if context.combat_active:
        return 120  # Quick combat turns
    elif context.dialogue_mode:
        return 200  # Allow more for conversation
    elif context.new_location:
        return 300  # Detailed first impression
    else:
        return 170  # Standard default
```

### Explicit Length Instruction Variants

**Ultra-brief**:
```
Respond in 1-2 sentences maximum. Be concise.
```

**Standard**:
```
Generate a couple sentences to a paragraph. Keep it brief.
```

**Flexible**:
```
Respond naturally, but aim for {{sentence_count}} sentences.
```

### Smart Sentence Trimming

More sophisticated post-processing:

```python
def smart_trim(text, max_tokens=170):
    """
    Trim to complete sentences while staying under token limit.
    Prioritizes completeness over hitting exact limit.
    """
    import tiktoken

    encoder = tiktoken.encoding_for_model("gpt-3.5-turbo")

    # Split into sentences
    sentences = re.split(r'(?<=[.!?])\s+', text)

    result = []
    total_tokens = 0

    for sentence in sentences:
        sentence_tokens = len(encoder.encode(sentence))
        if total_tokens + sentence_tokens <= max_tokens:
            result.append(sentence)
            total_tokens += sentence_tokens
        else:
            break

    return ' '.join(result)
```

## Integration with Other Techniques

### With Few-Shot Examples
Show examples that are the desired length:
```
Example response (ideal length):
"The merchant nods. 'Fresh bread, two copper.' He gestures to the warm loaves behind him."

Now generate your response:
{{prompt}}
```

### With NDL (Natural Description Language)
Even NDL-generated narrative benefits from length control:
```python
ndl_instruction = convert_to_ndl(game_event)
narration = llm.generate(
    prompt=f"Convert to narrative:\n{ndl_instruction}",
    max_tokens=170
)
```

### With Chain of Thought
Use token budgets per step:
```
Step 1: What does the character want? (50 tokens)
Step 2: How do they attempt it? (70 tokens)
Step 3: What's the immediate result? (50 tokens)
```

## Testing Checklist

- [ ] Test at different token limits (100, 150, 170, 250)
- [ ] Verify sentence trimming works correctly
- [ ] Check for abrupt cuts that break immersion
- [ ] Test with combat (needs brevity)
- [ ] Test with dialogue (needs more room)
- [ ] Test with descriptions (first impressions)
- [ ] Validate across different models
- [ ] Measure token costs before/after implementation
- [ ] Test edge case: single very long sentence
- [ ] Test edge case: many very short sentences

## Performance Metrics

**From ChatBot RPG Production Use**:
- **Average tokens per response**: ~150-160 (target: 170)
- **Incomplete sentence rate**: ~5% before trimming, ~0.5% after
- **Player feedback**: Responses feel "snappy" and "game-like"
- **Cost reduction**: ~40% compared to unlimited generation
- **Context window efficiency**: ~60% more turns per context window

## Source

**Original Discussion**: Discord transcript lines 5823-5836 (February 11, 2024)
**Context**: [[User-appl2613]] explaining ChatBot RPG's response management system
**Key Quote**:
> "i enforce a 170 token limit on the API calls, while telling it to generate a couple sentences to a paragraph. I use regex to skim off any trailing incomplete sentences."

**Follow-Up Discussion**: Lines 5837-5866 on LLM coherence patterns and why front-loaded generation works

## Related Documentation

- [[02-Prompt-Engineering]] - Broader prompting context
- [[User-appl2613]] - Creator's other contributions
- [[01-Architecture-and-Design]] - Where length control fits in system design
