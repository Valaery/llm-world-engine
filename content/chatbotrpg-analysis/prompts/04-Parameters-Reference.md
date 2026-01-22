# Prompt Parameters Reference - ChatBotRPG

**Purpose**: Comprehensive reference for all LLM inference parameters
**Source**: Extracted from actual `make_inference()` calls throughout codebase
**Usage**: Quick lookup for parameter configurations

---

## Parameter Overview

ChatBotRPG uses **consistent parameter patterns** across different use cases. Parameters are passed to `make_inference()` function which handles all LLM API calls.

### Core Parameters

| Parameter | Type | Purpose | Default |
|-----------|------|---------|---------|
| `context` | List[Dict] | Message array (role + content) | Required |
| `user_message` | String | Last user message (for logging) | Required |
| `character_name` | String | Character name (metadata) | Required |
| `url_type` | String | Model identifier | From config |
| `max_tokens` | Integer | Output length limit | 8192 |
| `temperature` | Float | Creativity (0.0-1.0) | 0.7 |
| `is_utility_call` | Boolean | Suppress verbose logging | False |
| `seed` | Integer | Random seed (optional) | None |
| `allow_summarization_retry` | Boolean | Auto-summarize on overflow | True |

---

## Temperature Strategies

### By Use Case

| Use Case | Temperature | Reasoning | Source Location |
|----------|-------------|-----------|-----------------|
| **Character Narration** | 0.7 | Balance creativity/consistency | chatBotRPG.py:684 |
| **Actor Generation** | 0.7 | Creative but coherent | generate_actor.py:180 |
| **Equipment Generation** | 0.7 | Creative within structure | generate_actor.py:180 |
| **Setting Generation** | 0.7 | Evocative but grounded | generate_setting.py:234 |
| **Random List Meta** | 0.7 | Creative table design | generate_random_list.py:66 |
| **NPC Note Generation** | 0.7 | Brief but personal | character_inference.py:1295 |
| **Context Summarization** | 0.3 | Factual accuracy | make_inference.py:26 |
| **Follower Summary** | 0.2 | Very factual | summaries.py:36 |
| **Intent Analysis** | 0.1 | Classification accuracy | agent_chat.py:258 |
| **Scribe AI Main** | 0.7 | Helpful/creative | agent_chat.py:124 |
| **Rule CoT Analysis** | Variable | Per-rule config | character_inference.py:382 |

### Temperature Ranges

```
0.1 ████ Very Low (Intent Classification)
0.2 ████ Low (Follower Summary)
0.3 ████ Medium-Low (Context Summarization)
0.7 ████████████ Standard (Most Creative Tasks)
0.9+ ███████████████ High (Not used in production)
```

**Pattern Discovery**: ChatBotRPG **never exceeds 0.7 temperature** in production code. Higher temperatures avoided for consistency.

---

## Token Limits

### By Use Case

| Use Case | Max Tokens | Reasoning | Source Location |
|----------|------------|-----------|-----------------|
| **Character Narration** | Dynamic | Settings-based (often 170) | chatBotRPG.py:153 |
| **NPC Note Generation** | 100 | Very brief notes only | character_inference.py:1294 |
| **Actor Name** | 256 | Short name output | generate_actor.py:179 |
| **Actor Description** | 256 | Single paragraph | generate_actor.py:179 |
| **Actor Personality** | 256 | 15-25 traits | generate_actor.py:179 |
| **Actor Appearance** | 256 | 15-25 traits | generate_actor.py:179 |
| **Actor Goals** | 256 | Brief goals | generate_actor.py:179 |
| **Actor Story** | 256 | Short backstory | generate_actor.py:179 |
| **Actor Equipment** | 512 | JSON with 16+ slots | generate_actor.py:179 |
| **Setting Name** | 400 | 1-4 words | generate_setting.py:232 |
| **Setting Description** | 800 | 1-2 sentences | generate_setting.py:232 |
| **Setting Connections** | 400 | JSON paths | generate_setting.py:232 |
| **Setting Inventory** | 400 | JSON array 3-7 items | generate_setting.py:232 |
| **Intent Analysis** | 500 | JSON classification | agent_chat.py:256 |
| **Follower Summary** | 512 | Concise memory | summaries.py:35 |
| **Context Summarization** | 1536 | Detailed compression | make_inference.py:19 |
| **Random List Meta** | 4000 | Full generator JSON | generate_random_list.py:65 |
| **Scribe AI Main** | 4000 | Detailed assistance | agent_chat.py:122 |
| **Rule CoT** | 50 | Brief classification | character_inference.py:382 |

### Token Strategy Patterns

1. **Very Brief (50-100)**: Rule classifications, NPC notes
2. **Short (256-512)**: Character generation fields, summaries
3. **Medium (800-1536)**: Setting descriptions, context compression
4. **Long (4000)**: Meta-generation, development assistance

---

## Model Selection

### Configuration Hierarchy

```python
# 1. Rule-specific model (highest priority)
if rule_has_model:
    model = rule.get('model')

# 2. Character-specific model switch
elif action_type == 'Switch Model':
    model = action_obj.get('value')

# 3. Settings model
else:
    model = tab_data['settings'].get('model', get_default_model())
```

### Model Types

| Model Type | Config Key | Use Case | Location |
|------------|------------|----------|----------|
| **Default Model** | `default_model` | Main narration | config.py:74 |
| **CoT Model** | `default_cot_model` | Rule evaluation | config.py:77 |
| **Utility Model** | `default_utility_model` | Generation, summaries | config.py:81 |
| **Search Model** | Hardcoded | Scribe AI search | agent_chat.py:24 |

### Default Configuration

**Location**: `config.py:14-17`

```python
DEFAULT_CONFIG = {
    "default_model": "google/gemini-2.5-flash-lite-preview-06-17",
    "default_cot_model": "google/gemini-2.5-flash-lite-preview-06-17",
    "default_utility_model": "google/gemini-2.5-flash-lite-preview-06-17",
}
```

**Note**: All default to same model, but can be configured independently.

### Fallback Models

**Location**: `chatBotRPG.py:36-38`

```python
FALLBACK_MODEL_1 = "cognitivecomputations/dolphin-mistral-24b-venice-edition:free"
FALLBACK_MODEL_2 = "thedrummer/anubis-70b-v1.1"
FALLBACK_MODEL_3 = "google/gemini-2.5-flash-lite-preview-06-17"
```

**Usage**: Automatic retry on character inference failure.

---

## Context Construction Patterns

### Pattern 1: System + User (Simple)

**Used By**: Generation prompts

```python
context = [
    {"role": "user", "content": prompt}
]
```

**Characteristics**:
- No system message
- Single user prompt
- No conversation history

---

### Pattern 2: System + User (Utility)

**Used By**: Summarization, intent analysis

```python
context = [
    {"role": "system", "content": system_prompt},
    {"role": "user", "content": user_prompt}
]
```

**Characteristics**:
- System message defines role
- Single user prompt
- No history

---

### Pattern 3: Multi-Stage Assembly (Complex)

**Used By**: Character inference, Scribe AI

```python
context = []
context.append({"role": "system", "content": base_system})
# ... add character sheet
# ... add memories
# ... add setting
# ... add keyword context
# ... add history
context.append({"role": "user", "content": turn_instruction})
```

**Characteristics**:
- Incremental building
- Multiple information sources
- Conversation history included
- Dynamic system modifications

---

### Pattern 4: System + History + User (Conversational)

**Used By**: Scribe AI

```python
messages = [{"role": "system", "content": SYSTEM_PROMPT}]
if self.context:
    messages.extend(self.context)
messages.append({"role": "user", "content": self.user_message})
```

**Characteristics**:
- Persistent system prompt
- Full conversation history
- New user message

---

## API Call Patterns

### Standard Call

**Location**: `make_inference.py:98-99`

```python
final_data = {
    "model": url_type,
    "temperature": temperature,
    "max_tokens": max_tokens,
    "top_p": 0.95,                    # Fixed
    "messages": context
}
```

**Fixed Parameters**:
- `top_p`: Always 0.95 (no configuration option)

---

### Headers (OpenRouter)

**Location**: `make_inference.py:101-104`

```python
if current_service == "openrouter":
    headers["Authorization"] = f"Bearer {api_key}"
    headers["HTTP-Referer"] = "https://github.com/your-repo/your-project"
    headers["X-Title"] = "ChatBot RPG"
```

---

### Timeout

**Location**: `make_inference.py:110`

```python
final_response = requests.post(base_url, headers=headers, json=final_data, timeout=180)
```

**Value**: 180 seconds (3 minutes) for all requests

---

## Retry Strategies

### Strategy 1: Automatic Summarization Retry

**Trigger**: Context too long error
**Location**: `make_inference.py:128-223`

```python
if "maximum context length" in error_details.lower():
    # Split conversation
    # Summarize each half
    # Reconstruct context
    # Retry once
```

**Parameters for Summarization**:
- Temperature: 0.3
- Max tokens: 1536
- Model: Utility model
- Attempts: 1 (no further retry)

---

### Strategy 2: Generation Field Retry

**Trigger**: Empty or invalid response
**Location**: `generate_actor.py:72-223`

```python
retry_count = 0
max_retries = 5
while not success and retry_count < max_retries:
    # Generate
    if retry_count > 0:
        prompt += f"\n\nThis is retry #{retry_count}. Please ensure..."
    # Validate
    if valid:
        success = True
    else:
        retry_count += 1
```

**Escalation**: Adds increasingly stern instructions

---

### Strategy 3: Model Fallback Retry

**Trigger**: Failure response patterns ("I'm", "sorry", "ext")
**Location**: `character_inference.py:774-876`

```python
for fallback_model in [FALLBACK_MODEL_1, FALLBACK_MODEL_2, FALLBACK_MODEL_3]:
    # Retry with fallback model
    if still_fails:
        continue  # Try next
    else:
        break  # Success
```

**Parameters**: Same context, same temp, different model

---

## Parameter Validation

### Required Parameters

**Location**: `make_inference.py:79-88`

```python
if not api_key and current_service != "local":
    return f"Sorry, API error: {service_name} API key not configured."
```

**Checks**:
1. API key exists (unless local)
2. Base URL configured
3. Model identifier valid

---

### Optional Parameters

```python
if seed is not None:
    random.seed(seed)
    seed = random.randint(-1, 100000)
```

**Note**: Seed is randomized even when provided (for variation).

---

## Performance Considerations

### Token Budget Strategy

**Discovery**: ChatBotRPG uses **dynamic token limits** based on use case, not fixed limits.

| Priority | Token Budget | Use Cases |
|----------|--------------|-----------|
| **Critical** | 100-512 | NPC notes, classifications |
| **Standard** | 256-800 | Generation, descriptions |
| **Extended** | 1536-4000 | Summarization, assistance |
| **Dynamic** | Settings-based | Main narration (170 common) |

---

### Temperature vs Token Tradeoff

**Pattern**: Lower temperature often paired with lower tokens

```
High Temperature + High Tokens = Expensive + Risky
Low Temperature + Low Tokens = Cheap + Consistent ✓
Low Temperature + High Tokens = Factual + Detailed ✓
High Temperature + Low Tokens = Creative + Brief ✓
```

**ChatBotRPG Strategy**: Match temperature to purpose, tokens to output needs.

---

## Cross-References

### Validates Discord Claims
✅ **170-token sweet spot** - Confirmed as common narration limit
✅ **Temperature 0.7 standard** - Most creative tasks use this
✅ **Dynamic token limits** - Different limits per use case
✅ **Low temperature for facts** - 0.2-0.3 for summarization

### Related Documentation
- [[01-Character-Narration-Prompts|Character Narration]] - Narration parameters
- [[02-Generation-Prompts|Generation Prompts]] - Generation parameters
- [[03-Scribe-AI-and-Utility-Prompts|Utility Prompts]] - Utility parameters
- [[chatbotrpg-analysis/analysis/05-Production-Lessons|Production Lessons]] - Performance insights

---

## Quick Reference Table

### Complete Parameter Matrix

| Use Case | Temp | Tokens | Model | Retry | Context Type |
|----------|------|--------|-------|-------|--------------|
| Character Narration | 0.7 | Dynamic | Settings | Fallback | Multi-stage |
| NPC Notes | 0.7 | 100 | CoT | No | System+User |
| Actor Name | 0.7 | 256 | Utility | 5x | User only |
| Actor Desc | 0.7 | 256 | Utility | 5x | User only |
| Actor Personality | 0.7 | 256 | Utility | 5x | User only |
| Actor Appearance | 0.7 | 256 | Utility | 5x | User only |
| Actor Goals | 0.7 | 256 | Utility | 5x | User only |
| Actor Story | 0.7 | 256 | Utility | 5x | User only |
| Actor Equipment | 0.7 | 512 | Utility | 5x | User only |
| Setting Desc | 0.7 | 800 | Utility | 3x | User only |
| Setting Name | 0.7 | 400 | Utility | 3x | User only |
| Setting Connect | 0.7 | 400 | Utility | 3x | User only |
| Setting Inventory | 0.7 | 400 | Utility | 3x | User only |
| Random List Meta | 0.7 | 4000 | Utility | No | System+User |
| Intent Analysis | 0.1 | 500 | Utility | No | System+User |
| Context Summary | 0.3 | 1536 | Utility | No | System+User |
| Follower Summary | 0.2 | 512 | Default | No | System+User |
| Scribe AI Main | 0.7 | 4000 | Default | No | System+History+User |
| Rule CoT | Varies | 50 | Per-rule | No | System+User |

---

## Tags

#chatbotrpg #parameters #temperature #tokens #model-selection #configuration #reference #performance
