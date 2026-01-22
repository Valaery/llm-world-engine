# ChatBotRPG - Discovered Prompts Index

**Status**: Initial Extraction Complete
**Date**: 2026-01-20
**Source**: ChatBotRPG Source Code Analysis
**Total Prompts Discovered**: 15

---

## Overview

This folder contains **extracted prompt text** from the ChatBotRPG codebase. Every prompt template, system message, and LLM interaction has been documented with:

- **Exact prompt text** from source code
- **Variable substitution** patterns
- **Parameters** (temperature, max_tokens, model)
- **Code location** with line numbers
- **Usage context** and examples
- **Cross-references** to Discord discussions

---

## Prompt Categories

### 1. Core Narration Prompts
**Purpose**: Drive character-based narration and NPC dialogue

- [[01-Character-System-Prompt]] - Base system prompt for NPC/character inference (character_inference.py)
- [[02-Character-Turn-Prompt]] - Turn-specific prompt for character actions (character_inference.py)
- [[03-NPC-Note-Generation-Prompt]] - Generate memory notes for NPCs (character_inference.py)

**Parameters**:
- Temperature: 0.7 (narration), 0.7 (note generation)
- Max tokens: Variable (main inference), 100 (note generation)
- Model: Dynamic (from settings)

---

### 2. Generation Prompts
**Purpose**: Procedurally generate game content

#### Actor Generation
- [[04-Actor-Name-Generation]] - Generate character names (generate_actor.py)
- [[05-Actor-Description-Generation]] - Generate character descriptions (generate_actor.py)
- [[06-Actor-Personality-Generation]] - Generate comma-separated personality traits (generate_actor.py)
- [[07-Actor-Appearance-Generation]] - Generate comma-separated appearance traits (generate_actor.py)
- [[08-Actor-Goals-Generation]] - Generate character goals/motivations (generate_actor.py)
- [[09-Actor-Story-Generation]] - Generate character backstory (generate_actor.py)
- [[10-Actor-Equipment-Generation]] - Generate character equipment JSON (generate_actor.py)

#### Setting Generation
- [[11-Setting-Description-Generation]] - Generate location descriptions (generate_setting.py)
- [[12-Setting-Name-Generation]] - Generate location names (generate_setting.py)
- [[13-Setting-Connections-Generation]] - Generate path descriptions as JSON (generate_setting.py)
- [[14-Setting-Inventory-Generation]] - Generate location items as JSON array (generate_setting.py)

#### Random List Generation
- [[15-Random-List-Meta-Generation]] - Template meta-generation system prompt (generate_random_list.py)

---

### 3. Utility Prompts
**Purpose**: Support functions (summarization, analysis, intent detection)

- [[16-Context-Summarization-Prompt]] - Summarize conversation history (make_inference.py)
- [[17-Follower-Memory-Summary-Prompt]] - Summarize shared scenes for followers (summaries.py)
- [[18-Intent-Analysis-Prompt]] - Analyze user intent for Scribe AI (agent_chat.py)

---

### 4. Scribe AI System
**Purpose**: Meta-assistant for game development

- [[19-Scribe-System-Prompt]] - Complete Scribe AI assistant system prompt (agent_chat.py)
- [[20-Character-Generation-Context-Intent]] - Intent classifier for character generation (agent_chat.py)

---

## Prompt Statistics

| Category | Prompt Count | Temperature Range | Token Range | Model Types |
|----------|--------------|-------------------|-------------|-------------|
| **Narration** | 3 | 0.7 | 100-8192 | Dynamic |
| **Generation** | 11 | 0.7-0.9 | 256-800 | Utility model |
| **Utility** | 3 | 0.1-0.3 | 50-1536 | Utility model |
| **Scribe AI** | 2 | 0.1-0.7 | 500-4000 | Default/Search |

---

## Key Findings

### 1. Anti-Hallucination Constraints
**Location**: `character_inference.py:268-273`

```python
system_msg_base_intro = (
    "You are in a third-person text RPG. "
    "You are responsible for writing ONLY the actions and dialogue of your assigned character, as if you are a narrator describing them. "
    "You must ALWAYS write in third person (using the character's name or 'he/she/they'), NEVER in first or second person. "
    "Assume other characters are strangers unless otherwise stated in special instructions. "
    "Write one single open-ended response (do NOT describe the OUTCOME of actions)."
)
```

**Analysis**: This is the **core anti-hallucination constraint** for character inference. It:
- Enforces third-person narration
- Prevents character assumptions
- Limits to single response (no chaining)
- Prevents outcome declaration (preserves agency)

---

### 2. Temperature Strategy
**Discovered Pattern**: Different temperatures for different purposes

| Purpose | Temperature | Reasoning |
|---------|-------------|-----------|
| Narration | 0.7 | Balance creativity/consistency |
| Generation | 0.7-0.9 | Higher for character variety |
| Summarization | 0.2-0.3 | Low for factual accuracy |
| Intent Analysis | 0.1 | Very low for classification |
| Rules/COT | Model-specific | Configured per rule |

---

### 3. Token Limiting Patterns

**170-Token Sweet Spot** (from Discord analysis):
- **Narration**: Dynamic (controlled by settings, default 170)
- **Generation**: 256-512 (enough for detailed content)
- **Summarization**: 512-1536 (factual compression)
- **Scribe AI**: 4000 (long-form assistance)

**Location**: `chatBotRPG.py:153`
```python
self.max_tokens = 8192  # Maximum, but rarely used
```

Actual usage is context-dependent and set per inference call.

---

### 4. Context Assembly Patterns

#### Pattern A: Incremental Builder
**Location**: `character_inference.py:466-531`

```python
npc_context_for_llm = []
npc_context_for_llm.append({"role": "system", "content": system_msg_base_intro})
npc_context_for_llm.append({"role": "user", "content": char_sheet_str})
# ... follower memories
# ... NPC notes
# ... setting description
# ... keyword injection
# ... history
# ... prepend system modifications
npc_context_for_llm.append({"role": "user", "content": f"(You are playing as: {char}...)"})
# ... append system modifications
```

**Analysis**: Context is built incrementally with:
1. Base system prompt
2. Character sheet
3. Memory/notes
4. Setting info
5. Keyword-injected content
6. Conversation history
7. Turn-specific instructions

#### Pattern B: Simple System+User
**Location**: `generate_actor.py:174-182`

```python
llm_response = make_inference(
    context=[{"role": "user", "content": prompt}],
    user_message=prompt,
    character_name=character_name,
    url_type=url_type,
    max_tokens=512 if field == 'equipment' else 256,
    temperature=0.7,
    is_utility_call=True
)
```

**Analysis**: Generation prompts use simple user-only context (no system message).

---

### 5. Automatic Context Summarization
**Location**: `make_inference.py:128-223`

ChatBotRPG implements **automatic context window management**:
1. Detects "maximum context length" errors
2. Splits conversation history in half
3. Summarizes each half independently
4. Reconstructs context with summaries
5. Retries inference

**Summarization Prompt** (line 177-183):
```python
summary1_instruction = (
    "You are a highly skilled text summarizer. Your task is to create a concise yet detailed summary "
    "of the following first part of a conversation. Focus on extracting and preserving all key events, "
    "character actions, important dialogue, and significant emotional shifts. The summary must be a "
    "factual representation of the provided text. Do not add new information or continue the conversation. "
    "Output only the summary."
)
```

**Parameters**:
- Temperature: 0.3 (factual)
- Max tokens: 1536
- Model: Utility model

---

### 6. Few-Shot Examples (Equipment Generation)
**Location**: `generate_actor.py:135-149`

The equipment generation prompt includes **extensive few-shot examples** across multiple genres:

```python
"Examples (adapt to character & genre):\n"
"  head: (Modern: baseball cap, sunglasses | Medieval: leather hood, metal helm | Empty: \"\")\n"
"  neck: (Modern: chain necklace, scarf | Medieval: amulet, wool scarf | Empty: \"\")\n"
"  left_shoulder/right_shoulder: (Modern: backpack strap, purse strap | Medieval: pauldron, cloak pin | Empty: \"\")\n"
"  left_hand/right_hand (WORN): (Modern: watch, gloves, rings | Medieval: leather gloves, signet ring, bracers | Empty: \"\")\n"
# ... etc
```

**Analysis**: Uses **multi-genre few-shot prompting** to:
- Guide slot-based equipment generation
- Distinguish WORN vs HELD items
- Show empty slot handling
- Demonstrate genre adaptation

---

### 7. JSON Extraction Patterns

**Pattern**: Flexible JSON extraction with fallback
**Location**: Multiple files (generate_actor.py, generate_setting.py, generate_random_list.py)

```python
try:
    equipment_dict = json.loads(llm_response)
except Exception:
    # Try extracting from markdown code fence
    match = re.search(r'```(?:json)?\s*([\s\S]+?)\s*```', llm_response, re.IGNORECASE)
    if match:
        json_str = match.group(1)
        equipment_dict = json.loads(json_str)
```

**Analysis**: Handles both:
- Direct JSON responses
- JSON wrapped in markdown code fences
- Automatic retry with improved prompts

---

### 8. Retry Logic
**Location**: `generate_actor.py:72-223`

Generation prompts implement **automatic retry with escalating instructions**:

```python
retry_count = 0
max_retries = 5
while not success and retry_count < max_retries:
    # ... generate
    if retry_count > 0:
        if field == 'equipment':
            prompt += f"\n\nThis is retry #{retry_count}. Please ensure your response is a valid JSON object with ALL required keys!"
        else:
            prompt += f"\n\nThis is retry #{retry_count}. Please ensure your response is not empty!"
```

**Analysis**: Retries include:
- Escalating instruction clarity
- Field-specific guidance (JSON validation)
- Maximum 5 attempts
- Fallback to default values

---

### 9. Model Fallback Chain
**Location**: `character_inference.py:774-876` (fallback retry)

```python
FALLBACK_MODEL_1 = "cognitivecomputations/dolphin-mistral-24b-venice-edition:free"
FALLBACK_MODEL_2 = "thedrummer/anubis-70b-v1.1"
FALLBACK_MODEL_3 = "google/gemini-2.5-flash-lite-preview-06-17"
```

ChatBotRPG implements **automatic model fallback** when:
- Response starts with "I'm", "sorry", "ext" (failure indicators)
- Response is duplicate of previous post

**Analysis**: Production resilience through cascading fallbacks.

---

### 10. Variable Substitution
**Location**: Throughout `character_inference.py`

Prompts support **runtime variable substitution**:

```python
if hasattr(self, '_substitute_variables_in_string'):
    value = self._substitute_variables_in_string(value, tab_data, char)
```

**Example Variables**:
- `{character_name}`
- `{location}`
- `{time}`
- `{datetime}`
- Custom game variables

---

## Cross-References to Discord

### Validated Claims
✅ **170-Token Sweet Spot** - Confirmed in code, dynamic configuration
✅ **Anti-Hallucination Constraints** - Exact wording found in character_inference.py
✅ **Third-Person Enforcement** - Core system prompt requirement
✅ **Temperature Switching** - Dynamic temperature based on use case
✅ **Automatic Summarization** - Full implementation in make_inference.py
✅ **Few-Shot Equipment** - Extensive examples in generate_actor.py

### Newly Discovered
🆕 **Scribe AI Meta-Assistant** - Complete development assistant system (not discussed in Discord)
🆕 **Intent Analysis System** - Automatic context detection for Scribe AI
🆕 **Model Fallback Chain** - Automatic retry with 3 fallback models
🆕 **Random List Meta-Generation** - Template generation system
🆕 **Setting Generation** - Complete location generation pipeline

---

## Next Steps

### Recommended Follow-Up Agents
1. **prompt-diff-analyzer** - Track prompt evolution via git history
2. **implementation-validator** - Verify all Discord claims against actual prompts
3. **metrics-extractor** - Find actual performance metrics in logs/comments

### Gaps to Fill
- [ ] Timer rule prompts (if any exist)
- [ ] Validation rule prompts
- [ ] Combat narration variations
- [ ] Game over condition prompts
- [ ] Intro sequence text generation

---

## File Locations Reference

| Prompt Type | Source File | Lines |
|-------------|-------------|-------|
| Character System | character_inference.py | 268-273 |
| Character Turn | character_inference.py | 522-524 |
| NPC Notes | character_inference.py | 1279-1296 |
| Actor Generation | generate_actor.py | 78-167 |
| Setting Generation | generate_setting.py | 212-218 |
| Random List Meta | generate_random_list.py | 12-33 |
| Context Summarization | make_inference.py | 177-193 |
| Follower Summary | summaries.py | 20-29 |
| Scribe System | agent_chat.py | 41-87 |
| Intent Analysis | agent_chat.py | 146-245 |

---

## Tags

#chatbotrpg #prompts #prompt-engineering #extracted #production #llm #system-prompts #generation #narration #anti-hallucination

---

## Related Documentation

- [[chatbotrpg-analysis/analysis/01-Repository-Overview|Repository Overview]] - Architecture context
- [[chatbotrpg-analysis/analysis/03-Prompt-Implementation|Prompt Implementation Analysis]] - High-level prompt patterns
- [[LLM World Engine/prompts/00-PROMPT-INDEX|General Prompt Library]] - Compare with general patterns
- [[LLM World Engine/patterns/00-PATTERN-INDEX|Pattern Library]] - Architectural context
