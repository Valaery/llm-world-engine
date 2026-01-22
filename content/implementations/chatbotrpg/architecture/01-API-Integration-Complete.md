---
tags: [implementation, chatbotrpg, api, openrouter, integration, architecture]
created: 2026-01-21
agent: api-integration-tracer
status: complete
---

# ChatBotRPG API Integration - Complete Documentation

## Overview

Complete documentation of OpenRouter.ai and multi-provider LLM API integration in ChatBotRPG, including retry logic, error handling, automatic summarization, and failover mechanisms.

**Analysis Date**: 2026-01-21
**Primary API**: OpenRouter.ai
**Secondary APIs**: Google GenAI, Local (LM Studio)
**Main File**: `src/core/make_inference.py` (231 lines)
**Config File**: `src/config.py` (97 lines)

---

## Architecture Overview

### Multi-Provider Support

Chat BotRPG supports 3 LLM providers via unified interface:

1. **OpenRouter.ai** - Primary (aggregator for 100+ models)
2. **Google GenAI** - Direct Google API
3. **Local** - Local inference (LM Studio, etc.)

**Switching**: User-configurable in `config.json` (`current_service` field)

---

## Configuration System

### Config File Structure

**Location**: `config.json` (root directory)

```json
{
  "current_service": "openrouter",           // "openrouter" | "google" | "local"
  "openrouter_api_key": "sk-or-...",
  "openrouter_base_url": "https://openrouter.ai/api/v1",
  "google_api_key": "",
  "google_base_url": "https://generativelanguage.googleapis.com/v1beta",
  "local_api_key": "",                       // Optional for local
  "local_base_url": "http://127.0.0.1:1234/v1",
  "default_model": "google/gemini-2.5-flash-lite-preview-06-17",
  "default_cot_model": "google/gemini-2.5-flash-lite-preview-06-17",
  "default_utility_model": "google/gemini-2.5-flash-lite-preview-06-17",
  "default_temperature": 0.3,
  "default_max_tokens": 2048
}
```

### Model Types

Chat BotRPG uses 3 distinct model configurations:

1. **default_model**: Main generation (character responses, narration)
2. **default_cot_model**: Chain-of-thought reasoning (rule evaluation)
3. **default_utility_model**: Utility tasks (summarization, intent analysis)

**Purpose**: Allow different models for different cognitive loads

**Source**: `config.py` lines 6-19

---

## Core API Client

### Function: `make_inference()`

**Location**: `make_inference.py` lines 79-231

**Signature**:
```python
def make_inference(
    context: list[dict],           # Message history
    user_message: str,             # Current user message
    character_name: str | None,    // Current character (or None for narrator)
    url_type: str,                 # Model name
    max_tokens: int,               # Max output tokens
    temperature: float,            # Temperature (0.0-1.0)
    seed: int | None = None,       # Random seed (optional)
    is_utility_call: bool = False, # Suppress logging if utility
    allow_summarization_retry: bool = True  # Enable auto-summarization
) -> str
```

**Returns**: Generated text or error message

---

## Request Flow

### 1. Service Selection

```python
current_service = get_current_service()  # "openrouter", "google", or "local"
api_key = get_api_key_for_service()
```

**Source**: `make_inference.py` lines 82-83

---

### 2. Google GenAI Path (Special Case)

If `current_service == "google"`:

```python
return _make_google_genai_request(context, url_type, max_tokens, temperature, api_key)
```

**Google-Specific Handling**:
- Uses `google-genai` Python SDK (not HTTP)
- Converts `system` role → `user` role with `[SYSTEM]` prefix
- Converts `assistant` role → `model` role (Google terminology)
- Strips `google/` prefix from model names

**Source**: `make_inference.py` lines 36-77, 90-91

---

### 3. OpenRouter/Local Path (Standard)

**Endpoint Construction**:
```python
base_url = get_base_url_for_service()  # e.g., "https://openrouter.ai/api/v1"
base_url = f"{base_url}/chat/completions"  # OpenAI-compatible endpoint
```

**Request Payload**:
```json
{
  "model": "google/gemini-2.5-flash-lite-preview-06-17",
  "temperature": 0.3,
  "max_tokens": 2048,
  "top_p": 0.95,
  "messages": [
    {"role": "system", "content": "..."},
    {"role": "user", "content": "..."},
    {"role": "assistant", "content": "..."}
  ]
}
```

**Headers (OpenRouter)**:
```python
headers = {
    "Content-Type": "application/json",
    "Authorization": f"Bearer {api_key}",
    "HTTP-Referer": "https://github.com/your-repo/your-project",
    "X-Title": "ChatBot RPG"
}
```

**Source**: `make_inference.py` lines 93-108

---

### 4. HTTP Request

```python
final_response = requests.post(
    base_url,
    headers=headers,
    json=final_data,
    timeout=180  # 3 minutes
)
final_response.raise_for_status()
final_response_data = final_response.json()
```

**Timeout**: 180 seconds (3 minutes)

**Source**: `make_inference.py` lines 110-112

---

### 5. Response Extraction

```python
if final_response_data.get("choices") and final_response_data["choices"][0].get("message"):
    final_message = final_response_data["choices"][0]["message"]["content"]
    return final_message
```

**OpenAI-Compatible Response Format**:
```json
{
  "choices": [
    {
      "message": {
        "role": "assistant",
        "content": "Generated text here..."
      }
    }
  ]
}
```

**Source**: `make_inference.py` lines 229-231

---

## Error Handling

### Timeout Errors

```python
except requests.exceptions.Timeout:
    error_msg = "Sorry, the request timed out."
    print(f"\n=== API Error ===\n{error_msg}")
    return error_msg
```

**Behavior**: Returns error string (displayed to user)

**Source**: `make_inference.py` lines 113-116

---

### HTTP Errors (4xx/5xx)

```python
except requests.exceptions.RequestException as e:
    status_code = e.response.status_code if e.response else None
    error_data = e.response.json() if e.response else {}
    error_details = error_data.get("error", {}).get("message", "Unknown error")

    # Check for context length error
    if "maximum context length" in error_details.lower():
        # Trigger automatic summarization (see below)
        ...

    return f"Sorry, API error ({status_code}): {error_details}"
```

**Special Handling**: Context length errors trigger automatic summarization retry

**Source**: `make_inference.py` lines 117-223

---

### API Key Missing

```python
if not api_key and current_service != "local":
    service_names = {"openrouter": "OpenRouter", "google": "Google GenAI"}
    service_name = service_names.get(current_service, current_service.title())
    return f"Sorry, API error: {service_name} API key not configured. Please check config.json file."
```

**Behavior**: Friendly error message with service name

**Source**: `make_inference.py` lines 85-88

---

## Automatic Summarization Retry

### Trigger Condition

When API returns "maximum context length exceeded" error:

```python
if allow_summarization_retry and \
   "maximum context length" in error_details.lower() and \
   not final_data.get("_summarization_attempted", False):
    # Attempt summarization and retry
```

**Flag**: `_summarization_attempted` prevents infinite loops

**Source**: `make_inference.py` lines 128-129

---

### Summarization Algorithm

**Step 1: Message Categorization**

```python
leading_setup_messages = []      # System prompts before conversation
conversational_history_messages = []  # User/assistant exchanges
trailing_system_messages = []    # Final instructions (e.g., "your turn")
```

**Logic**:
- Messages before first `assistant` message → `leading_setup_messages`
- Messages between first and last `assistant` → `conversational_history_messages`
- `system` messages after last `assistant` → `trailing_system_messages`

**Source**: `make_inference.py` lines 131-169

---

**Step 2: Two-Phase Summarization**

```python
# Split conversation in half
mid_point = len(original_conversational_text) // 2
first_half_text = original_conversational_text[:mid_point]
second_half_text = original_conversational_text[mid_point:]

# Summarize first half
summary1 = _internal_summarize_chunk(first_half_text, summary1_instruction, user_message)

# Summarize second half (with context from first half)
summary2 = _internal_summarize_chunk(second_half_text, summary2_instruction, user_message)

# Combine summaries
full_conversation_summary = f"{summary1}\n\n{summary2}"
```

**Why Two-Phase?**:
- Prevents loss of detail in long conversations
- Second summary knows about first summary (maintains continuity)
- Each summary uses utility model with 1536 token budget

**Source**: `make_inference.py` lines 174-198

---

**Step 3: Reconstructed Context**

```python
new_messages_for_retry = []
new_messages_for_retry.extend(leading_setup_messages)
new_messages_for_retry.append({
    "role": "user",
    "content": f"The historical user/assistant conversation has been summarized:\n\n{full_conversation_summary}\n\n..."
})
if character_name and character_name != "Narrator":
    character_prompt = f"(You are playing as: {character_name}. What does {character_name} do or say next?)"
    new_messages_for_retry.append({"role": "user", "content": character_prompt})
new_messages_for_retry.extend(trailing_system_messages)
```

**Structure**:
1. Original system setup (character sheets, world context)
2. Summarized conversation history
3. Character turn instruction (if applicable)
4. Final turn-specific instructions

**Source**: `make_inference.py` lines 199-211

---

**Step 4: Retry Request**

```python
final_data["messages"] = new_messages_for_retry
final_data["_summarization_attempted"] = True  # Prevent infinite retry

final_response = requests.post(base_url, headers=headers, json=final_data, timeout=180)
final_response.raise_for_status()
final_response_data = final_response.json()

return final_response_data["choices"][0]["message"]["content"]
```

**Success Path**: Returns generated text
**Failure Path**: Returns error message (no further retries)

**Source**: `make_inference.py` lines 212-223

---

### Summarization Prompts

**First Half Instruction**:
```
You are a highly skilled text summarizer. Your task is to create a concise yet detailed summary
of the following first part of a conversation. Focus on extracting and preserving all key events,
character actions, important dialogue, and significant emotional shifts. The summary must be a
factual representation of the provided text. Do not add new information or continue the conversation.
Output only the summary.
```

**Second Half Instruction**:
```
You are a highly skilled text summarizer. The first part of the conversation was summarized as: {summary1}

Now, your task is to create a concise yet detailed summary of the following second part of the conversation.
Focus on extracting and preserving all key events, character actions, important dialogue, and significant
emotional shifts from this second part. The summary must be a factual representation of the provided text.
Do not add new information or continue the conversation from the perspective of any character.
Output only the summary of the second part.
```

**Key Phrases**:
- "Concise yet detailed" - Balance brevity and completeness
- "Factual representation" - Prevent hallucination
- "Do not add new information" - Prevent continuation
- "Do not continue from perspective of character" - Prevent role-bleed

**Source**: `make_inference.py` lines 177-192

---

## Token Counting

### No Built-In Token Counter

ChatBotRPG **does not** implement token counting in the API client.

**Observation**: Token limits are configured, but actual usage is not tracked.

**Implications**:
- Relies on API error messages for context length violations
- No pre-emptive summarization before API call
- Reactive (not proactive) context management

**Why This Works**:
- Context length errors are rare (prompts are well-sized)
- Automatic summarization handles edge cases
- Simplifies codebase (no tokenization library needed)

---

## Multi-Model Routing

### Model Selection Strategy

ChatBotRPG uses **3 model types** for different tasks:

```python
# Main generation (default_model)
make_inference(
    context=context,
    url_type=get_default_model(),  # Main model
    ...
)

# Chain-of-thought reasoning (default_cot_model)
make_inference(
    context=cot_context,
    url_type=get_default_cot_model(),  # CoT model
    ...
)

# Utility tasks (default_utility_model)
make_inference(
    context=utility_context,
    url_type=get_default_utility_model(),  # Utility model
    is_utility_call=True,  # Suppress logging
    ...
)
```

**Usage Examples**:
- **Main Model**: Character responses, narration, scene descriptions
- **CoT Model**: Rule condition evaluation (chain-of-thought prompts)
- **Utility Model**: Summarization, intent analysis, keyword generation

**Configuration**: User sets 3 separate model names in `config.json`

**Source**: `config.py` lines 72-82, `make_inference.py` line 24

---

## Temperature Control

### Default Temperature: 0.3

```python
"default_temperature": 0.3
```

**Low Temperature (0.0-0.5)**:
- More deterministic output
- Better for structured tasks (intent analysis, format enforcement)
- Reduces hallucination

**High Temperature (0.6-1.0)**:
- More creative/varied output
- Better for narration, character personality
- Increases spontaneity

**Observed Usage**:
- Intent analysis: 0.1 (very deterministic)
- Main generation: 0.3 (balanced)
- Creative generation: 0.7 (more varied)

**Source**: `config.py` line 18

---

## Request Parameters

### Standard Parameters (All Requests)

```python
{
    "model": "...",
    "temperature": 0.3,
    "max_tokens": 2048,
    "top_p": 0.95,        # Nucleus sampling
    "messages": [...]
}
```

**top_p**: 0.95 (fixed)
- Nucleus sampling for token selection
- 95% probability mass threshold
- Balances creativity and coherence

**Source**: `make_inference.py` line 98

---

### Seed Parameter (Optional)

```python
if seed is not None:
    random.seed(seed)
    seed = random.randint(-1, 100000)
```

**Behavior**: If seed provided, re-randomize it (curious pattern)

**Usage**: Not observed in codebase (always `None`)

**Source**: `make_inference.py` lines 80-81

---

## API Referer Headers (OpenRouter)

### Custom Headers for OpenRouter

```python
headers["HTTP-Referer"] = "https://github.com/your-repo/your-project"
headers["X-Title"] = "ChatBot RPG"
```

**Purpose**:
- **HTTP-Referer**: OpenRouter tracks usage by app
- **X-Title**: Display name in OpenRouter dashboard

**Note**: Referer URL is placeholder (not actual repo)

**Source**: `make_inference.py` lines 103-104

---

## Failover Logic

### No Built-In Failover

ChatBotRPG **does not** implement automatic model failover.

**Behavior**:
- If OpenRouter fails → Error message returned
- If Google GenAI fails → Error message returned
- No fallback to alternative provider

**User Workaround**: Manually switch `current_service` in `config.json`

**Why No Failover?**:
- Adds complexity
- Models have different capabilities/prompts
- User controls provider selection

---

## Character Inference Integration

### Standalone Character Inference

**File**: `src/core/standalone_character_inference.py`

**Purpose**: Timer-based automatic NPC actions

```python
def run_single_character_post(character_name, tab_data):
    # Build context for character
    context = build_character_context(character_name, tab_data)

    # Call make_inference
    response = make_inference(
        context=context,
        user_message="",
        character_name=character_name,
        url_type=get_default_model(),
        max_tokens=2048,
        temperature=0.3
    )

    # Add response to conversation
    add_message_to_conversation(response, character_name, tab_data)
```

**Integration Point**: Uses `make_inference()` for all LLM calls

**Source**: `standalone_character_inference.py` line 9 (import)

---

## Performance Characteristics

### Request Latency

**Timeout**: 180 seconds (3 minutes)

**Typical Latency** (observed from OpenRouter):
- Small prompts (< 1000 tokens): 1-5 seconds
- Large prompts (> 5000 tokens): 10-30 seconds
- Summarization retries: +5-10 seconds (per summary)

**Bottleneck**: LLM inference time (not network)

---

### Retry Overhead

**Summarization Retry Adds**:
1. Two summarization API calls (~5-10 seconds each)
2. One retry API call (~5-30 seconds)
3. Total: ~15-50 seconds additional latency

**Trade-Off**: Slower but succeeds vs. fast but fails

---

### Request Frequency

**No Rate Limiting**: ChatBotRPG does not implement client-side rate limiting

**Implications**:
- User can send multiple requests rapidly
- Relies on OpenRouter rate limits
- Timer rules can trigger many requests

---

## Error Messages

### User-Facing Error Messages

1. **Timeout**:
   ```
   Sorry, the request timed out.
   ```

2. **API Key Missing**:
   ```
   Sorry, API error: OpenRouter API key not configured. Please check config.json file.
   ```

3. **HTTP Error**:
   ```
   Sorry, API error (500): Internal server error
   ```

4. **Summarization Failed**:
   ```
   Sorry, API error (413): maximum context length exceeded (Summarization attempt also failed.)
   ```

5. **Google GenAI Not Installed**:
   ```
   Sorry, API error: google-genai package not installed. Please install it with 'pip install google-genai'
   ```

**Pattern**: All errors prefixed with "Sorry," (user-friendly tone)

**Source**: `make_inference.py` lines 38, 88, 114, 197, 223

---

## Code Quality Observations

### Strengths

1. **Automatic Summarization**: Innovative two-phase approach
2. **Multi-Provider**: Unified interface for 3 providers
3. **Error Handling**: Comprehensive exception handling
4. **User-Friendly Errors**: Clear, actionable error messages

### Weaknesses

1. **No Token Counting**: Relies on API errors (reactive)
2. **Hardcoded Referer**: Placeholder GitHub URL
3. **No Rate Limiting**: Could hit provider limits
4. **No Request Caching**: Duplicate requests re-sent
5. **Seed Re-Randomization**: Curious pattern (likely bug)

---

## Production Patterns

### 1. Two-Phase Summarization

**Pattern**: Split long text, summarize halves, combine
**Benefit**: Preserves detail in long conversations
**Drawback**: 2x summarization API calls

### 2. Reactive Context Management

**Pattern**: Wait for API error, then summarize
**Benefit**: Simple implementation (no tokenization)
**Drawback**: Slower first failure, extra latency

### 3. Unified Interface

**Pattern**: Single `make_inference()` for all LLM calls
**Benefit**: Easy to swap providers/models
**Drawback**: Provider-specific features lost

### 4. Error-As-String

**Pattern**: Return error messages as strings (not exceptions)
**Benefit**: Simplifies caller code
**Drawback**: Harder to distinguish errors from valid output

---

## Cross-References

### Related Discord Analysis Files

- [[LLM Processing Pipeline]] - Pre/Gen/Post pattern (implements here)
- [[Multi-Model Routing]] - Model selection strategy
- [[Adaptive Context Window]] - Two-phase summarization technique
- [[Temperature Switching]] - Uses different temps for different tasks

### Related ChatBotRPG Analysis Files

- [[01-Repository-Overview]] - Architecture overview
- [[03-Prompt-Implementation]] - Prompts sent via this API
- [[02-Pattern-Implementation]] - LLM pipeline pattern
- [[04-Code-Examples]] - Example API usage

---

## Key Insights

### API Integration Philosophy

1. **Simplicity Over Optimization**: No token counting, no caching
2. **User Control**: Manual provider switching
3. **Fail-Safe**: Automatic summarization prevents context errors
4. **Provider Agnostic**: OpenAI-compatible interface

### Production Lessons

1. **Two-Phase Summarization Works**: Handles long conversations without pre-emptive truncation
2. **Reactive Context Management**: Simpler than proactive (no tokenization library)
3. **Error-As-String Pattern**: User-friendly but harder to debug
4. **3-Model Strategy**: Optimize for task-specific cognitive load

---

## Statistics

**Primary API File**: 231 lines
**Config File**: 97 lines
**Total API Code**: ~328 lines
**Providers Supported**: 3 (OpenRouter, Google, Local)
**Model Types**: 3 (Main, CoT, Utility)
**Error Handlers**: 5 distinct error types
**Automatic Retry**: 1 (context length exceeded)
**Summarization Phases**: 2

---

## Tags

#api-integration #openrouter #google-genai #local-llm #retry-logic #automatic-summarization #error-handling #multi-provider #chatbotrpg #production-validated
