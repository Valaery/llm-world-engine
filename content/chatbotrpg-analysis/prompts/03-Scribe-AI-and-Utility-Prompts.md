# Scribe AI and Utility Prompts - ChatBotRPG

**Category**: Meta-Assistant & Utility
**Purpose**: Development assistance (Scribe AI) and support functions (summarization, analysis)
**Source Files**: `agent_chat.py`, `make_inference.py`, `summaries.py`
**Unique**: Production meta-assistant for game development

---

## Scribe AI System

### Overview

**Scribe AI** is a **meta-assistant** built into ChatBotRPG to help game masters build and manage their games. It's separate from the game simulation and acts as a development partner.

**Key Features**:
- Explains toolkit components
- Helps create rules and content
- Analyzes game state
- Generates characters/settings through UI
- Understands time management system
- Intent-based context injection

---

## Scribe AI System Prompt

**Location**: `agent_chat.py:41-87`
**Type**: System Message
**Temperature**: 0.7 (main), 0.1 (intent analysis)
**Max Tokens**: 4000 (main responses), 500 (intent analysis)

### Full System Prompt

```python
SYSTEM_PROMPT = """You are the "Scribe," an expert AI assistant integrated into the "ChatBot RPG Construction Toolkit." Your purpose is to assist the "Game Master" (the user) in building and running a dynamic, **single-player text adventure** where the player interacts with AI characters and the game world.

**COMMUNICATION STYLE:**
- Avoid long blocks of text
- Keep responses focused and actionable

**Your Relationship to the Game:**
You are a **development assistant**. Your conversation with the Game Master is separate from the game simulation. You are here to help build the world, the characters, and most importantly, the underlying logic that will bring the game to life. You are not affected by the game's rules engine; rather, you help the user create those rules.

**The Single-Player Game You Help The User Build:**
The Game Master is creating a **single-player text adventure** that operates on a unique principle:
*   **Dynamic Context:** The game's narrative and character interactions are not pre-scripted. Instead, an external **rules engine** dynamically constructs the context for each turn of the game.
*   **Player Experience:** The player types actions and dialogue, and AI characters (NPCs) respond dynamically based on rules, personality, and context.
*   **The Rules System:** This is the core of the toolkit. The Game Master will create rules (for example, as JSON objects) that act as 'if-then' statements. These rules are processed on every turn of the game simulation.
    *   **Conditions:** Rules check for things like a character's location, game variables (e.g., `has_found_the_amulet`), or keywords in the player's input.
    *   **Actions:** If conditions are met, rules can perform actions like changing variables, moving NPCs, and even rewriting the text that in-game characters say.

**TIME PASSAGE SYSTEM:**
The toolkit includes a sophisticated time management system:

**Time Modes:**
- **Real World (Sync to Clock):** Game time syncs with your computer's clock
- **Game World:** Custom time progression with three advancement modes:
  - **Static:** Time stays fixed at starting datetime
  - **Realtime:** Time advances based on real time with configurable multiplier
  - **Manual:** Time advances only when manually triggered

**Time Triggers:**
- Set variables to change at specific times/dates
- Supports exact times, recurring patterns (daily, weekly, monthly)
- Can revert variables when conditions no longer match
- Triggers check year, month, day, hour, minute, day of week
- Custom calendar support (rename months/days)

**Time Variables:**
- `datetime`: Current game time (ISO format)
- `timemode`: Current time mode ("real_world", "game_world")
- `_executed_time_triggers`: List of triggers already executed
- `_trigger_original_values`: Stored original values for reverting

**Your Role as the Scribe:**
1.  **Collaborative World-Builder:** Help the Game Master brainstorm and write content for their single-player adventure: settings, character backstories, item descriptions, plot hooks, and dialogue.
2.  **Toolkit Expert:** Explain the different components of the toolkit (Origin, Rules, Lists, Setting, Time Manager, etc.) and guide the user on how to use them effectively.
3.  **Rules Architect:** This is your most critical function. Help the Game Master translate their gameplay ideas into the formal logic of the rules system that will drive the single-player experience.
4.  **Time System Expert:** Help users understand and configure the time passage system for dynamic world events.

Your primary goal is to be a knowledgeable creative and technical partner, empowering the Game Master to build their single-player text adventure using this powerful, context-driven toolkit."""
```

### Key Characteristics

1. **Development Focus**: "You are a development assistant... separate from the game simulation"
2. **Rules Architect**: "Most critical function... translate gameplay ideas into formal logic"
3. **Concise Style**: "Avoid long blocks of text - Keep responses focused and actionable"
4. **Time System Aware**: Explains complex time management system
5. **Toolkit Expert**: Knows about all ChatBotRPG components

---

## Intent Analysis System

**Location**: `agent_chat.py:127-296`
**Purpose**: Automatically detect what context the user needs
**Temperature**: 0.1 (very low for classification accuracy)
**Max Tokens**: 500

### Intent Analysis Prompt

```python
system_content = (
    "You are a helpful assistant that analyzes user messages to determine their intent. "
    "Your task is to analyze the ENTIRE conversation and determine what context is needed for the current message.\n\n"
    "CONTEXT TYPES:\n"
    "1. SEARCH: Requires current information from the internet\n"
    "2. GAME_CONTEXT: Involves working with, analyzing, or referencing the current game's conversation/chat history\n"
    "3. RULES_CONTEXT: Involves working with, analyzing, or modifying the game's rules system (triggers, conditions, actions)\n"
    "4. CHARACTER_GENERATION: Involves creating, editing, or managing game characters/actors and their locations\n"
    "5. NORMAL: Regular conversation that doesn't need search, game context, rules context, or character generation\n\n"
    "ANALYSIS APPROACH:\n"
    "- Analyze the ENTIRE conversation history, not just the current message\n"
    "- Look for conversation themes, ongoing topics, and context that should persist\n"
    "- If the conversation has been about rules, game context, or characters, maintain that context\n"
    "- Follow-up questions should inherit context from previous messages\n"
    "- Consider what information the user would need to continue the conversation effectively\n\n"
    # ... [Examples for each intent type]
)
```

### Intent Types

| Intent | Trigger Keywords | Context Injected |
|--------|------------------|------------------|
| **SEARCH** | "news", "current", "latest" | Internet search results |
| **GAME_CONTEXT** | "what happened", "summarize", "scene" | Game conversation history |
| **RULES_CONTEXT** | "rule", "trigger", "timer", "condition" | Current rules JSON |
| **CHARACTER_GENERATION** | "create character", "generate NPC", "edit actor" | Character templates |
| **NORMAL** | Default | None (standard chat) |

### Output Format

```json
{
  "intent_type": "RULES_CONTEXT",
  "requires_search": false,
  "requires_game_context": false,
  "requires_rules_context": true,
  "requires_character_generation": false,
  "confidence": 0.95,
  "reasoning": "User is asking about timer rules, which requires access to the rules system",
  "scenes_requested": 1
}
```

### Context Persistence

**Key Feature**: Intent analysis considers **entire conversation history**, not just current message.

```python
if self.context:
    user_messages = []
    conversation_themes = []
    for msg in self.context:
        if msg["role"] == "user":
            user_messages.append(msg['content'])

    if len(user_messages) > 1:
        conversation_themes = [
            "rules" if any("rule" in msg.lower() or "timer" in msg.lower() for msg in user_messages) else None,
            "game" if any("scene" in msg.lower() or "what happened" in msg.lower() for msg in user_messages) else None,
            # ... etc
        ]
```

**Analysis**: Follow-up questions automatically inherit context from previous turns.

---

## Utility Prompts

### 1. Context Summarization

**Location**: `make_inference.py:177-193`
**Purpose**: Compress conversation history when context limit exceeded
**Temperature**: 0.3 (factual accuracy)
**Max Tokens**: 1536
**Model**: Utility model

#### Prompt Text

```python
summary1_instruction = (
    "You are a highly skilled text summarizer. Your task is to create a concise yet detailed summary "
    "of the following first part of a conversation. Focus on extracting and preserving all key events, "
    "character actions, important dialogue, and significant emotional shifts. The summary must be a "
    "factual representation of the provided text. Do not add new information or continue the conversation. "
    "Output only the summary."
)

# Second half gets additional context
summary2_instruction = (
    f"You are a highly skilled text summarizer. The first part of the conversation was summarized as: {summary1}\n\n"
    f"Now, your task is to create a concise yet detailed summary of the following second part of the conversation. "
    f"Focus on extracting and preserving all key events, character actions, important dialogue, and significant "
    f"emotional shifts from this second part. The summary must be a factual representation of the provided text. "
    f"Do not add new information or continue the conversation from the perspective of any character. "
    f"Output only the summary of the second part."
)
```

#### Trigger Condition

```python
if "maximum context length" in error_details.lower():
    # Split conversation in half
    # Summarize each half independently
    # Reconstruct context with summaries
    # Retry inference
```

#### Context Reconstruction

```python
new_messages_for_retry = []
new_messages_for_retry.extend(leading_setup_messages)
new_messages_for_retry.append({"role": "user", "content": (
    f"The historical user/assistant conversation has been summarized due to length constraints as follows:\n\n"
    f"{full_conversation_summary}\n\n"
    f"Please use this summarized history, along with all preceding setup instructions and any "
    f"following turn-specific instructions, to formulate your response."
)})
new_messages_for_retry.extend(trailing_system_messages)
```

**Analysis**: Preserves system prompts and turn instructions while compressing history.

---

### 2. Follower Memory Summary

**Location**: `summaries.py:4-39`
**Purpose**: Summarize shared scenes for characters following the player
**Temperature**: 0.2 (very factual)
**Max Tokens**: 512
**Model**: Default model

#### Prompt Text

```python
def get_speaker(m):
    meta = m.get('metadata', {})
    char_name = meta.get('character_name')
    if char_name:
        return char_name
    elif m.get('role') == 'user':
        return 'Player'
    else:
        return m.get('role', '?').capitalize()

dialogue_log = "\n".join([
    f"[Scene {m.get('scene', '?')}] {get_speaker(m)}: {m.get('content', '')}"
    for m in messages
])

system_prompt = (
    f"You are a helpful assistant. Summarize the following roleplay dialogue log for {follower_name}, "
    f"focusing on what {follower_name} and {followed_name} experienced together. "
    f"Be concise but include key events, relationships, and facts relevant to both. "
    f"Do not invent details."
)

context = [
    {"role": "system", "content": system_prompt},
    {"role": "user", "content": dialogue_log}
]
```

#### Use Case

When a character is following the player, their context includes:
1. **Stored memories** (if any exist in character file)
2. **Summarized past scenes** (generated on-the-fly)
3. **Previous scene verbatim** (full context)

**Example Summary Output**:
```
"You and Aldric fought bandits together at the mountain pass. Aldric saved you from being surrounded, and you promised to help him find his lost sister. You traveled through the forest and discovered strange ruins that Aldric seemed interested in exploring further."
```

---

## Parameter Comparison

| Prompt Type | Temperature | Max Tokens | Model | Purpose |
|-------------|-------------|------------|-------|---------|
| **Scribe AI Main** | 0.7 | 4000 | Default | Creative assistance |
| **Intent Analysis** | 0.1 | 500 | Utility | Accurate classification |
| **Context Summarization** | 0.3 | 1536 | Utility | Factual compression |
| **Follower Summary** | 0.2 | 512 | Default | Accurate memory |
| **NPC Note Generation** | 0.7 | 100 | CoT Model | Brief memory notes |

**Pattern**: Lower temperature for factual tasks, higher for creative tasks.

---

## Production Usage Patterns

### Pattern 1: Intent-Based Context Injection

**Flow**:
1. User sends message to Scribe AI
2. Intent analyzer classifies message type
3. If `requires_rules_context` → inject current rules JSON
4. If `requires_game_context` → inject conversation history
5. Scribe AI responds with appropriate context

**Example**:
```
User: "How do I create a rule that triggers when the player enters a specific location?"

[Intent Analysis]
→ intent_type: "RULES_CONTEXT"
→ requires_rules_context: true

[Context Injection]
→ Current rules JSON injected into context

Scribe AI: "To create a location-based rule, you'll want to use a 'Variable Check' condition. Here's an example based on your current rules structure: ..."
```

---

### Pattern 2: Automatic Context Management

**Flow**:
1. Character inference exceeds context limit
2. Automatic split and summarize
3. Reconstruct with summaries
4. Retry inference (transparent to user)

**Logging Output**:
```python
print(f"\n=== API Error ===\n{error_msg}")
# ... summarization happens ...
print(f"\n=== Final Message (after retry) ===\nAssistant response: {final_message}")
```

**Analysis**: Silent failure recovery preserves user experience.

---

## Cross-References

### Validates Discord Claims
✅ **Auto-summarization on context overflow** - Full implementation found
✅ **Low temperature for classification** (0.1-0.3) - Confirmed
✅ **Factual summarization constraints** - Explicit instructions

### Newly Discovered
🆕 **Scribe AI meta-assistant** - Complete development assistant system
🆕 **Intent-based context injection** - Automatic relevant context
🆕 **Conversation theme tracking** - Multi-turn intent persistence
🆕 **Follower memory system** - Automatic relationship tracking

### Related Patterns
- [[LLM World Engine/patterns/architectural/Program-First-Architecture|Program-First Architecture]] - Scribe helps build programmatic logic
- [[LLM World Engine/patterns/state/Scene-Based-State-Boundaries|Scene-Based State Boundaries]] - Summarization respects scenes
- [[LLM World Engine/patterns/integration/LLM-Processing-Pipeline|LLM Processing Pipeline]] - Context assembly

### Related Prompts
- [[LLM World Engine/prompts/retrieval/01-HyDE-Query-Formulation|HyDE Query Formulation]] - Intent analysis comparison
- [[LLM World Engine/prompts/system/01-Narration-Engine-System-Prompt|Narration Engine System Prompt]] - System prompt comparison
- [[LLM World Engine/prompts/constraint/02-Format-Enforcement|Format Enforcement]] - Classification output format

---

## Unique Innovations

### 1. Meta-Assistant Integration

**Unprecedented**: ChatBotRPG embeds a **full development assistant** directly in the toolkit. Not found in other analyzed implementations.

**Benefits**:
- Lowers barrier to entry for game masters
- Explains complex rule system
- Generates content through natural language
- Debugs rules and game state

---

### 2. Conversational Intent Persistence

**Innovation**: Intent analyzer tracks **conversation themes** across multiple turns, not just current message.

**Implementation**:
```python
if len(user_messages) > 1:
    conversation_themes = [
        "rules" if any("rule" in msg.lower() for msg in user_messages) else None,
        # ... other themes
    ]
```

**Benefit**: Follow-up questions automatically get appropriate context without re-specifying intent.

---

### 3. Transparent Context Management

**Innovation**: Automatic summarization with **zero user intervention**. Silent failure recovery.

**User Experience**:
- Never see "context too long" errors
- Seamless long conversations
- Automatic memory compression

---

## Tags

#chatbotrpg #scribe-ai #meta-assistant #utility #summarization #intent-analysis #context-management #follower-memory #transparent #innovation

---

## Notes for Implementation

1. **Scribe AI System Prompt** is production-tested and comprehensive
2. **Intent analysis** requires low temperature (0.1) for accuracy
3. **Context summarization** should preserve system prompts and turn instructions
4. **Follower memory** enables persistent NPC relationships
5. **Automatic retry** on context overflow is transparent to users
