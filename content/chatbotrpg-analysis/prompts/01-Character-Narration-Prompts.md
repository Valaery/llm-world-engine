# Character Narration Prompts - ChatBotRPG

**Category**: Core Narration
**Purpose**: Drive NPC/character actions and dialogue in third-person narrative style
**Source Files**: `character_inference.py`, `chatBotRPG.py`
**Complexity**: High (multi-stage context assembly)

---

## Overview

Character narration in ChatBotRPG uses a **complex multi-layered prompt system** that assembles context from multiple sources before generating each NPC action. The system enforces strict anti-hallucination constraints while allowing dynamic character behavior based on rules, memories, and scene context.

---

## Prompt 1: Character System Base

**Location**: `character_inference.py:264-274`
**Type**: System Message (Base Instruction)
**When Used**: Every NPC/character inference turn

### Full Prompt Text

```python
if character_system_context:
    system_msg_base_intro = character_system_context
else:
    system_msg_base_intro = (
        "You are in a third-person text RPG. "
        "You are responsible for writing ONLY the actions and dialogue of your assigned character, as if you are a narrator describing them. "
        "You must ALWAYS write in third person (using the character's name or 'he/she/they'), NEVER in first or second person. "
        "Assume other characters are strangers unless otherwise stated in special instructions. "
        "Write one single open-ended response (do NOT describe the OUTCOME of actions)."
    )
```

### Key Constraints

| Constraint | Purpose | Enforcement |
|------------|---------|-------------|
| Third-person only | Prevent first-person slip | "MUST ALWAYS write in third person" |
| No first/second person | Maintain narrator voice | "NEVER in first or second person" |
| Single character | Prevent multi-character control | "ONLY the actions... of your assigned character" |
| Strangers by default | Prevent relationship hallucination | "Assume other characters are strangers" |
| Open-ended | Preserve player agency | "do NOT describe the OUTCOME" |
| Single response | Prevent run-on narration | "Write one single open-ended response" |

### Cross-Reference

**Validates Discord Claim**: "We use third-person narration constraints to prevent the LLM from controlling the player"
**Pattern**: [[LLM World Engine/patterns/constraint/Constraint-Based-Prompting|Constraint-Based Prompting]]
**Technique**: [[LLM World Engine/prompts/constraint/03-Anti-Hallucination-Prompt|Anti-Hallucination Constraints]]

---

## Prompt 2: Character Turn Instruction

**Location**: `character_inference.py:522-524`
**Type**: User Message (Turn-Specific)
**When Used**: Final message in character inference context

### Full Prompt Text

```python
npc_context_for_llm.append({
    "role": "user",
    "content": f"(You are playing as: {char}. It is now {char}'s turn. What does {char} do or say next?)"
})
```

### Variable Substitution

```python
# Example substitution
char = "Elara"
# Becomes:
"(You are playing as: Elara. It is now Elara's turn. What does Elara do or say next?)"
```

### Purpose

1. **Remind model** of character assignment (reinforcement)
2. **Indicate turn** (when to respond)
3. **Prompt action** (open-ended question)
4. **Use character name** (reinforce third-person)

---

## Context Assembly Pipeline

**Location**: `character_inference.py:466-531`

### Full Assembly Sequence

```python
npc_context_for_llm = []

# 1. Base System Prompt
npc_context_for_llm.append({"role": "system", "content": system_msg_base_intro})

# 2. Character Sheet
char_sheet_str = "(Character sheet not found)"
npc_file_path = _find_actor_file_path(self, workflow_data_dir, char)
if npc_file_path:
    npc_data = _load_json_safely(npc_file_path)
    if npc_data:
        relevant_fields = ['name', 'description', 'personality', 'appearance', 'goals', 'story', 'equipment', 'left_hand_holding', 'right_hand_holding']
        filtered_npc_data = {k: v for k, v in npc_data.items() if k in relevant_fields and v}
        char_sheet_content = json.dumps(filtered_npc_data, indent=2)
        char_sheet_str = f"Your character sheet (JSON format):\n```json\n{char_sheet_content}\n```"

npc_context_for_llm.append({"role": "user", "content": char_sheet_str})

# 3. Follower Memories (if applicable)
scenes_to_recall = 1
if npc_file_path:
    # Check if following player
    variables = npc_data.get('variables', {})
    if variables.get('following', '').strip().lower() == 'player':
        scenes_to_recall = 2

mem_summary = _get_follower_memories_for_context(self, workflow_data_dir, char, list(chars_in_scene), filtered_context, scenes_to_recall=scenes_to_recall)
if mem_summary:
    npc_context_for_llm.append({"role": "user", "content": mem_summary})

# 4. NPC Notes (memory system)
if npc_file_path:
    npc_notes = get_npc_notes_from_character_file(npc_file_path)
    if npc_notes:
        formatted_notes = format_npc_notes_for_context(npc_notes, char)
        if formatted_notes:
            npc_context_for_llm.append({"role": "user", "content": formatted_notes})

# 5. Setting Description
if setting_desc:
    setting_info_content = f"(The current setting of the scene is: {setting_desc}{setting_connections})"
    npc_context_for_llm.append({"role": "user", "content": setting_info_content})

# 6. Keyword Injection (context-aware)
current_scene = tab_data.get('scene_number', 1)
current_setting_name = _get_player_current_setting_name(workflow_data_dir) if workflow_data_dir else None
location_info = get_location_info_for_keywords(workflow_data_dir, current_setting_file) if workflow_data_dir and current_setting_file else None
is_narrator = False
npc_context_for_llm = inject_keywords_into_context(
    npc_context_for_llm, full_history_context, char,
    current_setting_name, location_info, workflow_data_dir,
    current_scene, is_narrator
)

# 7. Conversation History (filtered by scene)
for item in history_to_add:
    npc_context_for_llm.append(item)

# 8. Prepend System Modifications (from rules)
for mod in context_modifications:
    if mod.get('role') == 'system' and mod.get('position') == 'prepend':
        content = mod['content']
        if hasattr(self, '_substitute_variables_in_string'):
            content = self._substitute_variables_in_string(content, tab_data, char)
        npc_context_for_llm.insert(1, {"role": "system", "content": content})

# 9. Turn Instruction
npc_context_for_llm.append({
    "role": "user",
    "content": f"(You are playing as: {char}. It is now {char}'s turn. What does {char} do or say next?)"
})

# 10. Append System Modifications (from rules)
for mod in context_modifications:
    if mod.get('role') == 'system' and mod.get('position') == 'append':
        content = mod['content']
        if hasattr(self, '_substitute_variables_in_string'):
            content = self._substitute_variables_in_string(content, tab_data, char)
        npc_context_for_llm.append({"role": "system", "content": content})
```

### Assembly Diagram

```
┌─────────────────────────────────────┐
│  1. System Base Prompt              │  ← Anti-hallucination constraints
├─────────────────────────────────────┤
│  (Prepended System Modifications)   │  ← Rule-injected system messages
├─────────────────────────────────────┤
│  2. Character Sheet (JSON)          │  ← Static character data
├─────────────────────────────────────┤
│  3. Follower Memories (Optional)    │  ← If following player
├─────────────────────────────────────┤
│  4. NPC Notes (Optional)            │  ← Auto-generated memories
├─────────────────────────────────────┤
│  5. Setting Description             │  ← Current location context
├─────────────────────────────────────┤
│  6. Keyword-Injected Content        │  ← Dynamic context injection
├─────────────────────────────────────┤
│  7. Conversation History (Scene)    │  ← Filtered by current scene
├─────────────────────────────────────┤
│  8. Turn Instruction                │  ← "What does X do next?"
├─────────────────────────────────────┤
│  (Appended System Modifications)    │  ← Rule-injected system messages
└─────────────────────────────────────┘
```

---

## Parameters

### Inference Call

**Location**: `character_inference.py:679-685`

```python
thread = InferenceThread(
    context,                              # Assembled context
    character,                            # Character name
    model,                                # Model from rules or settings
    self.max_tokens,                      # Dynamic (default 8192)
    self.get_current_temperature()        # Dynamic (default 0.7)
)
```

### Parameter Values

| Parameter | Value | Source | Purpose |
|-----------|-------|--------|---------|
| `context` | Assembled | Pipeline above | Full context |
| `character_name` | String | Character file | Metadata |
| `url_type` | String | Settings/Rules | Model selection |
| `max_tokens` | Integer | Settings | Output length limit |
| `temperature` | Float (0.7) | Settings | Creativity level |
| `is_utility_call` | False | Hardcoded | Not a utility call |

### Temperature Override

**Location**: `character_inference.py:436-440`

Rules can override temperature per-character:

```python
if action_type == 'Switch Model':
    switched_model = action_obj.get('value', '').strip()
    if switched_model:
        model_to_use = switched_model
        print(f"        [Action] Switched model for '{char}' to: {model_to_use}")
```

---

## Post-Processing

### Response Cleaning

**Location**: `character_inference.py:695-701`

```python
actual_character_name = get_actual_character_name(self, character_name)
if actual_character_name and isinstance(msg, str):
    prefix = f"{actual_character_name}:"
    if msg.strip().startswith(prefix):
        msg = msg.strip()[len(prefix):].lstrip()

# Remove CoT tags
if isinstance(msg, str):
    msg = re.sub(r'<think>[\s\S]*?</think>', '', msg, flags=re.IGNORECASE).strip()
```

### Post-Processing Steps

1. **Remove character name prefix** (e.g., "Elara: walks forward" → "walks forward")
2. **Remove CoT tags** (`<think>...</think>`)
3. **Deduplicate check** (detect repeated responses)
4. **Fallback retry** (if failure detected)

---

## Prompt 3: NPC Note Generation

**Location**: `character_inference.py:1279-1296`
**Type**: User Message
**Purpose**: Auto-generate memory notes from NPC responses
**When Used**: After each NPC response

### Full Prompt Text

```python
recent_messages = []
for msg in current_context[-5:]:
    if msg.get('role') == 'user':
        player_name = _get_player_name_for_context(workflow_data_dir)
        recent_messages.append(f"{player_name}: {msg.get('content', '')}")
    elif msg.get('role') == 'assistant':
        char_name = msg.get('metadata', {}).get('character_name', 'Unknown')
        recent_messages.append(f"{char_name}: {msg.get('content', '')}")

recent_messages.append(f"{character_name}: {npc_response}")
context_str = "\n".join(recent_messages)

note_prompt = f"""Based on this recent conversation, write a very brief note (1-2 sentences max) from {character_name}'s perspective about what just happened or what they learned. Focus on key events, discoveries, or important interactions. Write in first person as {character_name}.

Recent conversation:
{context_str}

Brief note from {character_name}'s perspective:"""

note_context = [
    {"role": "system", "content": "You are helping an NPC character write brief personal notes about recent events. Keep notes very concise and in first person."},
    {"role": "user", "content": note_prompt}
]
```

### Parameters

```python
model = tab_data.get('settings', {}).get('cot_model', get_default_cot_model())
thread = UtilityInferenceThread(
    chatbot_ui_instance=self,
    context=note_context,
    model_identifier=model,
    max_tokens=100,                  # Very short
    temperature=0.7                  # Moderate creativity
)
```

### Example

**Input**:
```
Player: Hello, do you know where the ancient ruins are?
Elara: I've heard rumors of ruins to the north, beyond the old bridge.
Player: Thank you! I'll check there.
Elara: Be careful. Strange things have been seen in that area lately.
```

**Generated Note**:
```
"I told a traveler about the ruins north of the bridge and warned them about the strange occurrences there. I hope they stay safe."
```

### Purpose

1. **Persistent memory** - NPCs remember past interactions
2. **Auto-generated** - No manual note-taking required
3. **First-person perspective** - Character's view of events
4. **Concise** - 1-2 sentences max

---

## Fallback System

**Location**: `character_inference.py:774-876`

### Trigger Conditions

Fallback activates when response starts with:
- "I'm"
- "sorry"
- "ext"

Or when response is a duplicate of previous post.

### Fallback Models

```python
FALLBACK_MODEL_1 = "cognitivecomputations/dolphin-mistral-24b-venice-edition:free"
FALLBACK_MODEL_2 = "thedrummer/anubis-70b-v1.1"
FALLBACK_MODEL_3 = "google/gemini-2.5-flash-lite-preview-06-17"
```

### Fallback Logic

```python
for i, fallback_model in enumerate(fallback_models):
    print(f"[FALLBACK] Attempting fallback model {i+1}: {fallback_model}")

    fallback_thread = InferenceThread(
        original_context,
        character_name,
        fallback_model,
        self.max_tokens,
        self.get_current_temperature()
    )

    # If this fails too, try next model
    if isinstance(msg, str) and any(msg.strip().lower().startswith(failure_start) for failure_start in ['i\'m', 'sorry', 'ext']):
        if i < len(fallback_models) - 1:
            continue  # Try next
        else:
            # All models failed
            error_msg = f"{fallback_char_name} seems to be having trouble responding right now."
            _queue_npc_message(self, error_msg, fallback_char_name, fallback_tag, {})
```

---

## Production Examples

### Example 1: Basic Character Action

**Context**:
- Character: Elara (elf scout)
- Setting: Forest clearing
- Previous: Player asked about path north

**Assembled Prompt** (simplified):
```
System: You are in a third-person text RPG. You are responsible for writing ONLY the actions and dialogue of your assigned character...

User: Your character sheet (JSON format):
{
  "name": "Elara",
  "description": "Elf scout",
  "personality": "Cautious, observant, protective",
  "appearance": "Silver eyes, leaf cloak"
}

User: (The current setting of the scene is: A small clearing in the ancient forest. Sunlight filters through the canopy above.)

User: Player: Is there a path north from here?

User: (You are playing as: Elara. It is now Elara's turn. What does Elara do or say next?)
```

**Generated Response**:
```
Elara studies the treeline carefully, her silver eyes scanning for any signs of danger. "There's a deer trail that heads north," she says quietly, pointing toward a gap in the underbrush. "But it's been disturbed recently. Something large passed through."
```

### Example 2: Character with Follower Memory

**Context**:
- Character: Kael (warrior, following player)
- Previous shared scenes: Fought bandits together

**Assembled Prompt** (simplified):
```
System: [Base system prompt]

User: [Character sheet]

User: (Your memories of adventures with Aldric):
You fought bandits together at the mountain pass. Aldric showed courage in battle and saved your life when you were outnumbered.

User: [Setting description]

User: [Conversation history]

User: (You are playing as: Kael. It is now Kael's turn. What does Kael do or say next?)
```

**Generated Response**:
```
Kael steps forward, his hand resting on his sword hilt. "I've got your back, Aldric," he says firmly, remembering how the traveler had once saved him from certain death.
```

---

## Performance Metrics

### Discovered from Code Comments

**Location**: `character_inference.py:690-692`

```python
# Fallback check for failure responses
if isinstance(msg, str) and any(msg.strip().lower().startswith(failure_start) for failure_start in ['i\'m', 'sorry', 'ext']):
```

**Analysis**: System actively detects and handles model refusals/failures, suggesting these are common enough to warrant automated handling.

---

## Cross-References

### Validates Discord Claims
✅ **Third-person narration** - Exact constraint found
✅ **Anti-hallucination** - Core system prompt
✅ **Temperature 0.7** - Default for narration
✅ **Dynamic token limiting** - Configurable per-inference
✅ **Context assembly** - Multi-stage pipeline confirmed

### Related Patterns
- [[LLM World Engine/patterns/integration/LLM-Processing-Pipeline|LLM Processing Pipeline]]
- [[LLM World Engine/patterns/control/Constraint-Based-Prompting|Constraint-Based Prompting]]
- [[LLM World Engine/patterns/state/Scene-Based-State-Boundaries|Scene-Based State Boundaries]]

### Related Prompts
- [[LLM World Engine/prompts/narration/01-NDL-to-Narrative|NDL-to-Narrative]] (comparison)
- [[LLM World Engine/prompts/constraint/03-Anti-Hallucination-Prompt|Anti-Hallucination Constraints]]
- [[LLM World Engine/prompts/system/01-Narration-Engine-System-Prompt|Narration Engine System Prompt]]

---

## Tags

#chatbotrpg #character-narration #system-prompt #anti-hallucination #third-person #npc #context-assembly #fallback #production
