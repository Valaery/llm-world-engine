# Generation Prompts - ChatBotRPG

**Category**: Procedural Generation
**Purpose**: Generate game content (characters, locations, items, templates)
**Source Files**: `generate_actor.py`, `generate_setting.py`, `generate_random_list.py`
**Pattern**: Template-based generation with retry logic and JSON extraction

---

## Overview

ChatBotRPG uses **template-driven prompts** for procedural content generation. All generation prompts share common patterns:

1. **Zero-shot** (no conversation history)
2. **Simple context** (user message only, no system prompt for most)
3. **JSON output** for structured data
4. **Automatic retry** with escalating instructions
5. **Fallback values** if generation fails

---

## Actor Generation Prompts

**Source**: `generate_actor.py`
**Temperature**: 0.7
**Max Tokens**: 256 (text fields), 512 (equipment)

### Prompt 1: Name Generation

**Location**: `generate_actor.py:77-92`

```python
if existing_name:
    name_prompt_instruction = f"The character's name is '{existing_name}'. Output just this name without any formatting or extra text."
else:
    name_prompt_instruction = f"Invent a new, creative name for a character based on the information below. Avoid simply repeating the existing name ('{existing_name}') if provided. Output just the name without any formatting or extra text."
    if not existing_name:
        name_prompt_instruction = "Create a name for a character based on the information below. Output just the name without any formatting or extra text."

prompt = f"""{instruction_prefix}{name_prompt_instruction}

Current Character Sheet:
{context}

New Name:"""
```

**Example Output**: `"Elara Moonwhisper"`

---

### Prompt 2: Description Generation

**Location**: `generate_actor.py:93-98`

```python
prompt = f"""{instruction_prefix}Given the following information about a character named {character_name}, write a single cohesive paragraph describing their background, role, and character essence. Focus on who they are, what they do, and their place in the world. Write in natural narrative style - no bullet points, lists, or fragmented sentences. Keep it to one well-developed paragraph that flows naturally.

Current Character Sheet:
{context}

Description:"""
```

**Key Constraints**:
- Single cohesive paragraph
- Natural narrative style
- No bullet points or lists
- Focus on essence, not details

**Example Output**:
```
"Elara is a seasoned scout who has spent most of her life in the ancient forests, learning their secrets and defending them from those who would do harm. Known for her keen senses and patient demeanor, she serves as both a guide and protector to travelers brave enough to venture into the woodland depths."
```

---

### Prompt 3: Personality Generation

**Location**: `generate_actor.py:99-103`

```python
prompt = f"""{instruction_prefix}Given the following information about a character named {character_name}, create a comprehensive comma-separated list of personality traits. Include both positive and negative traits, behavioral patterns, values, quirks, and how they interact with others. Focus on psychological and behavioral characteristics only. Do not write sentences or explanations - just traits separated by commas. Aim for 15-25 traits that capture the full personality spectrum.

Current Character Sheet:
{context}

Personality:"""
```

**Key Constraints**:
- Comma-separated list (not sentences)
- 15-25 traits
- Mix of positive and negative
- Psychological/behavioral only

**Example Output**:
```
"cautious, observant, protective of nature, patient, speaks in riddles, distrustful of strangers, loyal to forest, perfectionist about tracking, stubborn in beliefs, kind to animals, quick to anger when nature threatened, values silence, appreciates beauty, dislikes cities, respects ancient traditions, independent, self-sufficient, occasionally cryptic, mentoring, defensive, territorial, intuitive, spiritual"
```

---

### Prompt 4: Appearance Generation

**Location**: `generate_actor.py:104-108`

```python
prompt = f"""{instruction_prefix}Given the following information about a character named {character_name}, create a comprehensive comma-separated list of physical appearance traits. Include build, height, skin tone, hair color/style, eye color, facial features, distinguishing marks, posture, movement patterns, and any unique physical characteristics. Focus ONLY on physical traits - do not include clothing, accessories, or equipment. Do not write sentences or explanations - just physical descriptors separated by commas. Aim for 15-25 traits that capture the complete physical appearance.

Current Character Sheet:
{context}

Appearance:"""
```

**Key Constraints**:
- Comma-separated list
- 15-25 traits
- Physical traits ONLY (no clothing)
- No sentences

**Example Output**:
```
"tall, lithe build, pale skin, silver hair worn long and loose, silver eyes that reflect moonlight, high cheekbones, pointed ears, small scar above left eyebrow, graceful movements, moves silently, light footsteps, wears calm expression, quick reflexes, sharp gaze, slender fingers, athletic build, upright posture, fluid gestures, minimal facial expressions, alert stance"
```

---

### Prompt 5: Goals Generation

**Location**: `generate_actor.py:109-110`

```python
prompt = f"""Given the following information about a character named {character_name}, list the character's main goals and motivations. Include both short-term and long-term ambitions, and explain why these goals matter to the character. Output plain text only, no markdown formatting.

Current Character Sheet:
{context}

Goals:"""
```

**Example Output**:
```
"Short-term: Track down the source of corruption spreading through the western forest. This matters because she's witnessed the slow death of trees and wildlife in that region.

Long-term: Become the next Forest Guardian, a title held by those who dedicate their lives to protecting the ancient woods. She seeks this role to honor her mentor who died defending the forest."
```

---

### Prompt 6: Story (Backstory) Generation

**Location**: `generate_actor.py:111-112`

```python
prompt = f"""Given the following information about a character named {character_name}, write a short backstory or narrative that explains how {character_name} became who they are. Focus on key events, relationships, and turning points. Output plain text only, no markdown formatting.

Current Character Sheet:
{context}

Story:"""
```

**Example Output**:
```
"Elara was born in the heart of the ancient forest to a family of scouts. Her childhood was spent learning to read tracks, identify plants, and move through the woods without leaving a trace. When she was sixteen, her village was attacked by bandits, and her mentor sacrificed himself to save her. This event changed Elara forever, making her fiercely protective of the forest and those who respect it. She vowed never to let another innocent fall on her watch."
```

---

### Prompt 7: Equipment Generation (Most Complex)

**Location**: `generate_actor.py:114-164`

**Unique Features**:
- Extensive few-shot examples
- Multi-genre adaptation
- Slot-based JSON structure
- Distinguishes WORN vs HELD

```python
prompt = (
    f"{instruction_prefix}You are an expert wardrobe designer. Given the following character information for {character_name}, "
    "generate a JSON object representing the character's worn equipment. "
    "The equipment should match the character's theme, type, and station. "
    "Respect the genre (e.g., medieval, modern, sci-fi)."
    "\n\n"
    "Current Character Sheet:\n"
    f"{context}"
    "\n\n"
    "The JSON object MUST contain exactly these keys: "
    f'{", ".join(EQUIPMENT_JSON_KEYS)}.'
    "\n\n"
    "For each key, provide a short description of the item worn/carried in that slot. "
    "The 'left_hand' and 'right_hand' slots are specifically for WORN items like gloves, rings, bracelets, etc. Do NOT put held items (weapons, shields, tools) here."
    "If multiple items are worn in the 'left_hand' or 'right_hand' slot, separate them with commas (e.g., \"leather gloves, silver ring\")."
    "Be very thorough, but if a slot is empty, use an empty string \"\"."
    "\n\n"
    "Examples (adapt to character & genre):\n"
    "  head: (Modern: baseball cap, sunglasses | Medieval: leather hood, metal helm | Empty: \"\")\n"
    "  neck: (Modern: chain necklace, scarf | Medieval: amulet, wool scarf | Empty: \"\")\n"
    "  left_shoulder/right_shoulder: (Modern: backpack strap, purse strap | Medieval: pauldron, cloak pin | Empty: \"\")\n"
    "  left_hand/right_hand (WORN): (Modern: watch, gloves, rings | Medieval: leather gloves, signet ring, bracers | Empty: \"\")\n"
    "  upper_over: (Modern: jacket, blazer | Medieval: cloak, leather armor | Empty: \"\")\n"
    "  upper_outer: (Modern: t-shirt, hoodie | Medieval: tunic, jerkin) [Usually not empty]\n"
    "  upper_middle: (Modern: undershirt, camisole | Medieval: chemise) [Often empty for males]\n"
    "  upper_inner: (Modern: bra | Medieval: bindings) [Often empty for males]\n"
    "  lower_outer: (Modern: jeans, skirt | Medieval: trousers, skirt) [Usually not empty]\n"
    "  lower_middle: (Modern: slip, bike shorts | Medieval: shorts, braies) [Often empty]\n"
    "  lower_inner: (Modern: boxers, panties | Medieval: smallclothes, loincloth) [Usually not empty]\n"
    "  left_foot_inner/right_foot_inner: (Modern: socks | Medieval: wool socks, foot wraps) [Often empty]\n"
    "  left_foot_outer/right_foot_outer: (Modern: sneakers, boots | Medieval: leather boots, sandals) [Usually not empty]\n"
    "\n\n"
    "Do NOT include hairstyles. Provide minimal visual description. Ensure all listed keys are present."
    "\n\n"
    "Output ONLY the JSON object:\n"
    "Example Output Format (using full key names):\n"
    "{\n"
    "  \"head\": \"worn leather cap\",\n"
    "  \"neck\": \"\",\n"
    "  \"left_shoulder\": \"\",\n"
    "  \"right_shoulder\": \"heavy backpack strap\",\n"
    "  \"left_hand\": \"leather glove, iron ring\",\n"
    "  \"right_hand\": \"worn bracer\",\n"
    "  ... (include all other keys using full names like left_foot_outer) ...\n"
    "}\n"
    "\n"
    "Equipment JSON:"
)
```

**Equipment Slots** (16 total):
```python
EQUIPMENT_JSON_KEYS = [
    "head", "neck", "left_shoulder", "right_shoulder", "left_hand", "right_hand",
    "upper_over", "upper_outer", "upper_middle", "upper_inner",
    "lower_outer", "lower_middle", "lower_inner",
    "left_foot_inner", "right_foot_inner", "left_foot_outer", "right_foot_outer"
]
```

**Example Output**:
```json
{
  "head": "",
  "neck": "",
  "left_shoulder": "quiver strap",
  "right_shoulder": "cloak clasp",
  "left_hand": "leather gloves, silver ring",
  "right_hand": "leather gloves, archer's bracer",
  "upper_over": "forest-green cloak",
  "upper_outer": "leather tunic",
  "upper_middle": "",
  "upper_inner": "linen shirt",
  "lower_outer": "dark leather trousers",
  "lower_middle": "",
  "lower_inner": "cotton smallclothes",
  "left_foot_inner": "wool socks",
  "right_foot_inner": "wool socks",
  "left_foot_outer": "soft leather boots",
  "right_foot_outer": "soft leather boots"
}
```

---

## Setting Generation Prompts

**Source**: `generate_setting.py`
**Temperature**: 0.7
**Max Tokens**: 400 (name), 800 (description)

### Prompt 8: Setting Description

**Location**: `generate_setting.py:212`

```python
prompt = f"Generate a short, simple, and to-the-point description for a setting based on the following context. Limit your answer to 1-2 clear sentences. Avoid extra detail or atmosphere. Output plain text only, no markdown formatting.\n\nCONTEXT:\n{context}\n\nDESCRIPTION:"
```

**Context Includes**:
- World information (name, description)
- Region information (if nested)
- Location information (if nested deeper)
- Containing features
- Additional instructions

**Example Output**:
```
"A small tavern with wooden beams and a stone fireplace. Locals gather here to trade news and ale."
```

---

### Prompt 9: Setting Name

**Location**: `generate_setting.py:214`

```python
prompt = f"Suggest a single, concise, evocative name for a setting based on the following context. The name should be 1-4 words, no explanations, no lists, no punctuation, and no quotes. Return ONLY the name, nothing else. Output plain text only, no markdown formatting.\n\nCONTEXT:\n{context}\n\nSETTING NAME:"
```

**Constraints**:
- 1-4 words
- No explanations
- No punctuation
- Just the name

**Example Output**:
```
"The Rusty Tankard"
```

---

### Prompt 10: Connections (Path Descriptions)

**Location**: `generate_setting.py:216`

```python
prompt = f"For a setting with the following context, generate a brief, direct phrase for each path leading to connected locations. Include a small amount of detail or atmosphere, preferably something unique and memorable, but keep it short. Do NOT include directions such as north or south.\n\nCONTEXT:\n{context}\n\nDescribe each connection as a JSON object with location names as keys and short path descriptions as values. Example:\n{{\n  \"Forest Clearing\": \"Narrow dirt path\",\n  \"Mountain Pass\": \"Steep rocky trail\"\n}}\n\nOutput ONLY the JSON object:\nCONNECTIONS:"
```

**Example Output**:
```json
{
  "Forest Road": "Muddy path lined with ancient oaks",
  "Town Square": "Cobblestone street lit by oil lamps",
  "Old Mill": "Overgrown trail following the creek"
}
```

---

### Prompt 11: Inventory (Location Items)

**Location**: `generate_setting.py:218`

```python
prompt = f"Generate a list of notable items or objects that would be found in this setting based on the context. These items might be interactable, collectible, or simply part of the atmosphere. Include 3-7 items that make sense for this type of location.\n\nCONTEXT:\n{context}\n\nItems present in this setting (output as a JSON array of strings):\nINVENTORY:"
```

**Example Output**:
```json
[
  "Old ledger book behind the bar",
  "Dartboard on the back wall",
  "Worn deck of playing cards",
  "Copper mugs hanging from hooks",
  "Fireplace poker leaning against stones"
]
```

---

## Random List Meta-Generation

**Source**: `generate_random_list.py`
**Purpose**: Generate weighted random tables for procedural generation
**Temperature**: 0.7
**Max Tokens**: 4000

### System Prompt

**Location**: `generate_random_list.py:12-33`

```python
DEFAULT_SYSTEM_PROMPT = """
You are an expert game development assistant helping create random generators for a single-player text adventure game. Your task is to analyze instructions and create or modify JSON generator files that follow a specific format.

A generator consists of one or more tables, each with a title and a list of items. Each item has a name and weight.
Format:
{
  "name": "Generator Name",
  "tables": [
    {
      "title": "Table Name",
      "items": [
        {"name": "Item Name", "weight": 1},
        {"name": "Another Item", "weight": 3}
      ]
    }
  ]
}

Each table represents a category of elements (like "Professions", "Personalities", etc). Items in the tables are weighted options, where higher weights make them more likely to be selected.

Remember to ensure valid JSON. Weights should be positive integers. Be creative but relevant to the instructions.
"""
```

### User Prompt Template

**Location**: `generate_random_list.py:94-111`

```python
user_prompt = f"""
Create a random generator based on these instructions: "{instructions}"

{name_instruction}
Create multiple tables as needed with at least 10-15 varied items per table with appropriate weights.

Respond ONLY with the complete, valid JSON object.
"""
```

**Example Instructions**: `"Create a generator for fantasy character occupations with appropriate weights"`

**Example Output**:
```json
{
  "name": "Fantasy Occupations Generator",
  "tables": [
    {
      "title": "Common Professions",
      "items": [
        {"name": "Blacksmith", "weight": 3},
        {"name": "Farmer", "weight": 5},
        {"name": "Merchant", "weight": 4},
        {"name": "Guard", "weight": 3},
        {"name": "Innkeeper", "weight": 2}
      ]
    },
    {
      "title": "Rare Professions",
      "items": [
        {"name": "Alchemist", "weight": 1},
        {"name": "Wizard", "weight": 1},
        {"name": "Assassin", "weight": 1}
      ]
    }
  ]
}
```

---

## Common Patterns

### 1. Retry Logic

**Location**: All generation files

```python
retry_count = 0
max_retries = 5  # or 3 for settings
success = False

while not success and retry_count < max_retries:
    llm_response = make_inference(...)

    if retry_count > 0:
        prompt += f"\n\nThis is retry #{retry_count}. Please ensure your response is valid!"

    # Try to parse/validate response
    if valid:
        success = True
    else:
        retry_count += 1

if not success:
    # Use fallback default
```

---

### 2. JSON Extraction

**Pattern**: Try direct parse, then markdown fence

```python
try:
    data = json.loads(llm_response)
except:
    match = re.search(r'```(?:json)?\s*([\s\S]+?)\s*```', llm_response, re.IGNORECASE)
    if match:
        data = json.loads(match.group(1))
```

---

### 3. Context Preparation

**Location**: `generate_actor.py:242-273`

```python
def _prepare_context(self, field_to_exclude: str = None) -> str:
    context_parts = []

    # Always include name first
    name = self.actor_data.get('name', '').strip()
    if name:
        context_parts.append(f"Name: {name}")

    # Include held items
    left_holding = self.actor_data.get('left_hand_holding', '').strip()
    right_holding = self.actor_data.get('right_hand_holding', '').strip()
    if left_holding:
        context_parts.append(f"Left Hand Holding: {left_holding}")
    if right_holding:
        context_parts.append(f"Right Hand Holding: {right_holding}")

    # Include all other fields except the one being generated
    for field_name, field_value in self.actor_data.items():
        if field_name in ['name', 'left_hand_holding', 'right_hand_holding']:
            continue
        if field_name == field_to_exclude:
            continue
        if not field_value or (isinstance(field_value, str) and not field_value.strip()):
            continue

        # Format based on type
        if isinstance(field_value, dict):
            formatted_value = json.dumps(field_value, indent=2)
            context_parts.append(f"{field_name.replace('_', ' ').title()}:\n{formatted_value}")
        elif isinstance(field_value, list):
            if field_value:
                formatted_items = '\n'.join([f"  - {item}" for item in field_value])
                context_parts.append(f"{field_name.replace('_', ' ').title()}:\n{formatted_items}")
        else:
            context_parts.append(f"{field_name.replace('_', ' ').title()}: {field_value}")

    return "\n\n".join(context_parts)
```

---

## Parameters Summary

| Prompt Type | Temperature | Max Tokens | Model | Retry |
|-------------|-------------|------------|-------|-------|
| Actor Name | 0.7 | 256 | Utility | 5 |
| Actor Description | 0.7 | 256 | Utility | 5 |
| Actor Personality | 0.7 | 256 | Utility | 5 |
| Actor Appearance | 0.7 | 256 | Utility | 5 |
| Actor Goals | 0.7 | 256 | Utility | 5 |
| Actor Story | 0.7 | 256 | Utility | 5 |
| Actor Equipment | 0.7 | 512 | Utility | 5 |
| Setting Description | 0.7 | 800 | Utility | 3 |
| Setting Name | 0.7 | 400 | Utility | 3 |
| Setting Connections | 0.7 | 400 | Utility | 3 |
| Setting Inventory | 0.7 | 400 | Utility | 3 |
| Random List | 0.7 | 4000 | Utility | N/A |

---

## Cross-References

### Validates Discord Claims
✅ **Template-based generation** - Confirmed
✅ **JSON output for structured data** - All generation uses JSON
✅ **Retry logic** - Implemented across all generators
✅ **Few-shot examples** - Equipment generation uses extensive examples

### Related Patterns
- [[LLM World Engine/patterns/generation/Template-Meta-Generation|Template Meta-Generation]]
- [[LLM World Engine/patterns/generation/Just-In-Time-Generation|Just-In-Time Generation]]

### Related Prompts
- [[LLM World Engine/prompts/generation/02-Character-Generation|Character Generation (General)]]
- [[LLM World Engine/prompts/generation/03-Location-Generation|Location Generation]]
- [[LLM World Engine/prompts/techniques/07-Few-Shot-Examples|Few-Shot Examples]]

---

## Tags

#chatbotrpg #generation #procedural #actor #setting #character #location #json #retry-logic #few-shot #template-generation
