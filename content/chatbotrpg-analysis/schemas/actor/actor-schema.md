---
tags: [schema, chatbotrpg, implementation, actor, character, npc]
created: 2026-01-20
---

# Actor Schema (Character/NPC)

**Primary Files**:
- `src/editor_panel/actor_manager.py` (UI, lines 1-1500+)
- `src/generate/generate_actor.py` (Generation, lines 10-24, 354-577)
- `src/core/utils.py` (Actor file operations, lines 841-898, 930-942)

**Storage Location**:
- Base: `{workflow_dir}/resources/data files/actors/*.json`
- Session (runtime): `{workflow_dir}/game/actors/*.json`

**Type Definition**: `typing.TypedDict` (lines 10-21 in generate_actor.py)

## Purpose

Represents a character or NPC in the game world, including the player character. Stores identity, stats, appearance, inventory, equipment, relationships, goals, and location data.

## Core Schema Structure

```python
class ActorData(typing.TypedDict, total=False):
    name: str
    description: str
    personality: str
    appearance: str
    status: str
    relations: dict[str, str]
    goals: str
    story: str
    equipment: dict[str, str]
    portrait: dict
    location: str
```

## Complete JSON Structure

```json
{
  "name": "Adventurer",
  "isPlayer": false,
  "description": "A weathered traveler seeking ancient artifacts in forgotten ruins.",
  "personality": "curious, cautious, determined, witty, loyal, pragmatic, resourceful",
  "appearance": "tall, lean build, weathered skin, short brown hair, piercing blue eyes, scar on left cheek",
  "status": "",
  "location": "Tavern",
  "goals": "Find the Crystal of Ages. Repay debt to mentor. Discover truth about family history.",
  "story": "Born in a small village, lost parents to raiders at age 12...",

  "relations": {
    "Guard Captain": "Mutual respect, sometimes works together",
    "Innkeeper": "Old friend, provides information",
    "Merchant": "Distrusts due to past dealings"
  },

  "left_hand_holding": "torch",
  "right_hand_holding": "rusty sword",

  "equipment": {
    "head": "leather cap",
    "neck": "wool scarf",
    "left_shoulder": "",
    "right_shoulder": "backpack strap",
    "left_hand": "leather glove, iron ring",
    "right_hand": "worn bracer",
    "upper_over": "travel cloak",
    "upper_outer": "linen tunic",
    "upper_middle": "",
    "upper_inner": "",
    "lower_outer": "canvas trousers",
    "lower_middle": "",
    "lower_inner": "smallclothes",
    "left_foot_inner": "wool socks",
    "right_foot_inner": "wool socks",
    "left_foot_outer": "leather boots",
    "right_foot_outer": "leather boots"
  },

  "variables": {
    "health": 85,
    "max_health": 100,
    "mana": 30,
    "max_mana": 50,
    "gold": 250,
    "reputation": 15,
    "quest_stage": 3,
    "has_key": true,
    "is_player": false
  },

  "portrait": {
    "path": "portraits/adventurer.png",
    "offset_x": 0,
    "offset_y": 0,
    "scale": 1.0,
    "tint_color": "#CCCCCC"
  },

  "npc_notes": "[2024-01-15 14:23] Met player at tavern\n[2024-01-15 15:45] Agreed to help find artifact\n[2024-01-16 09:12] Revealed secret about ruins",

  "schedule": [
    {
      "time": "08:00",
      "location": "Market Square",
      "activity": "Shopping for supplies"
    },
    {
      "time": "14:00",
      "location": "Training Grounds",
      "activity": "Sword practice"
    },
    {
      "time": "20:00",
      "location": "Tavern",
      "activity": "Evening meal and rest"
    }
  ]
}
```

## Field Documentation

| Field | Type | Required | Default | Description | UI Location |
|-------|------|----------|---------|-------------|-------------|
| **name** | string | Yes | "Unnamed Actor" | Character name (max 80 chars) | actor_manager.py:167-168 |
| **isPlayer** | boolean | Yes | false | True if this is the player character (only one allowed) | actor_manager.py:159-162 |
| **description** | string | No | "" | Background, role, and character essence (1-2 paragraphs) | actor_manager.py:200-206 |
| **personality** | string | No | "" | Comma-separated personality traits (15-25 traits) | actor_manager.py:214-220 |
| **appearance** | string | No | "" | Comma-separated physical traits (15-25 descriptors) | actor_manager.py:228-234 |
| **status** | string | No | "" | Current status/condition | - |
| **location** | string | No | "" | Current location name (references Setting) | actor_manager.py:171-176 |
| **goals** | string | No | "" | Short-term and long-term ambitions | actor_manager.py:276-284 |
| **story** | string | No | "" | Backstory, key events, relationships | generate_actor.py:111-112 |
| **relations** | object | No | {} | Key: character name, Value: relationship description | Actor UI relations section |
| **left_hand_holding** | string | No | "" | Item held in left hand | actor_manager.py:322-327 |
| **right_hand_holding** | string | No | "" | Item held in right hand | actor_manager.py:329-335 |
| **equipment** | object | Yes | {} | Worn items by body slot (17 slots) | See Equipment Schema below |
| **variables** | object | No | {} | Custom character variables (health, stats, flags, etc.) | actor_manager.py:246-274 |
| **portrait** | object | No | {} | Portrait image configuration | Portrait UI section |
| **npc_notes** | string | No | "" | Timestamped interaction notes | utils.py:427-431 |
| **schedule** | array | No | [] | Daily schedule entries | Schedule UI section |

## Equipment Schema

**Slot Order** (constants in generate_actor.py:29-34):

```python
EQUIPMENT_JSON_KEYS = [
    "head", "neck",
    "left_shoulder", "right_shoulder",
    "left_hand", "right_hand",
    "upper_over", "upper_outer", "upper_middle", "upper_inner",
    "lower_outer", "lower_middle", "lower_inner",
    "left_foot_inner", "right_foot_inner",
    "left_foot_outer", "right_foot_outer"
]
```

**Equipment Slot Descriptions** (actor_manager.py:38-48):

| Slot | Display Name | Purpose | Examples |
|------|--------------|---------|----------|
| head | Head | Headwear/accessories | leather cap, metal helm, crown |
| neck | Neck | Neckwear | amulet, scarf, chain |
| left_shoulder | Left Shoulder | Left shoulder item | pauldron, backpack strap |
| right_shoulder | Right Shoulder | Right shoulder item | pauldron, cloak pin |
| left_hand | Left Hand (Worn) | **WORN items only** (gloves, rings, bracelets) | leather gloves, silver ring |
| right_hand | Right Hand (Worn) | **WORN items only** (gloves, rings, bracelets) | worn bracer, gold ring |
| upper_over | Upper Outer | Outermost upper body | jacket, cloak, leather armor |
| upper_outer | Upper Outer | Outer upper body clothing | t-shirt, tunic, jerkin |
| upper_middle | Upper Middle | Middle upper body layer | undershirt, camisole, chemise |
| upper_inner | Upper Inner | Innermost upper body | bra, bindings (often empty for males) |
| lower_outer | Lower Outer | Outer lower body clothing | jeans, trousers, skirt |
| lower_middle | Lower Middle | Middle lower body layer | slip, shorts, braies |
| lower_inner | Lower Inner | Innermost lower body | boxers, panties, smallclothes |
| left_foot_inner | Left Foot Inner | Left foot inner layer | socks, foot wraps |
| right_foot_inner | Right Foot Inner | Right foot inner layer | socks, foot wraps |
| left_foot_outer | Left Foot Outer | Left footwear | sneakers, boots, sandals |
| right_foot_outer | Right Foot Outer | Right footwear | sneakers, boots, sandals |

**Important Notes**:
- `left_hand` and `right_hand` slots are for **WORN items ONLY** (gloves, rings, bracelets)
- Held items (weapons, shields, tools) go in `left_hand_holding` and `right_hand_holding` fields
- Multiple items in one slot are comma-separated: `"leather gloves, silver ring"`
- Empty slots use empty string `""`
- All 17 slots must be present in the equipment object

## Variables Schema

The `variables` object stores custom character data with flexible structure:

```json
{
  "health": 85,
  "max_health": 100,
  "mana": 30,
  "max_mana": 50,
  "gold": 250,
  "reputation": 15,
  "quest_stage": 3,
  "has_magic_key": true,
  "faction": "Merchant Guild",
  "is_player": false
}
```

**Common Variable Types**:
- **Numeric**: health, max_health, mana, gold, reputation, level
- **Boolean**: flags (has_key, quest_complete, is_friendly)
- **String**: faction, title, class
- **Mixed**: Any JSON-serializable type

**Variable Scopes** (used in rules/timers):
- **Character**: Variables on this specific actor
- **Player**: Variables on the player actor
- **Setting**: Variables on the current location/setting
- **Global**: Variables in `{workflow_dir}/game/variables.json`

## Portrait Schema

```json
{
  "path": "portraits/character.png",
  "offset_x": 0,
  "offset_y": 0,
  "scale": 1.0,
  "tint_color": "#CCCCCC"
}
```

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| path | string | null | Relative path to portrait image |
| offset_x | number | 0 | Horizontal offset in pixels |
| offset_y | number | 0 | Vertical offset in pixels |
| scale | number | 1.0 | Scale multiplier (1.0 = 100%) |
| tint_color | string | "#CCCCCC" | Hex color tint |

## Schedule Schema

Array of daily activity entries:

```json
{
  "time": "08:00",
  "location": "Market Square",
  "activity": "Shopping for supplies"
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| time | string | Yes | Time in HH:MM format (24-hour) |
| location | string | Yes | Location name (references Setting) |
| activity | string | Yes | Description of activity |

## File Operations

### Loading an Actor

```python
from core.utils import _find_actor_file_path, _load_json_safely

# Find actor file (searches game/ first, then resources/)
actor_file_path = _find_actor_file_path(self, workflow_data_dir, "Adventurer")

# Load actor data
actor_data = _load_json_safely(actor_file_path)
```

### Saving an Actor

```python
from core.utils import _save_json_safely

actor_data = {
    "name": "New Character",
    "isPlayer": False,
    "description": "A mysterious figure...",
    # ... other fields
}

save_path = os.path.join(workflow_data_dir, 'game', 'actors', 'new_character.json')
_save_json_safely(save_path, actor_data)
```

### Actor Caching

The system maintains an in-memory cache for fast actor lookups (utils.py:842-898):

```python
# Cache structure
self._actor_name_to_file_cache = {
    "adventurer": "/path/to/adventurer.json",
    "guard_captain": "/path/to/guard_captain.json"
}

self._actor_name_to_actual_name = {
    "adventurer": "Adventurer",
    "guard_captain": "Guard Captain"
}
```

**Normalization**: Names are normalized to lowercase with underscores replacing spaces.

**Cache Invalidation**: Cache is cleared when actors are added/removed or on game state load.

## Serialization

### To JSON

```python
import json

with open(actor_file, 'w', encoding='utf-8') as f:
    json.dump(actor_data, f, indent=2, ensure_ascii=False)
```

### From JSON

```python
import json

with open(actor_file, 'r', encoding='utf-8') as f:
    actor_data = json.load(f)
```

### Filename Convention

Actors are saved with sanitized filenames (actor_manager.py:10-13):

```python
def sanitize_path_name(name):
    sanitized = re.sub(r'[^a-zA-Z0-9_\-\. ]', '', name).strip()
    sanitized = sanitized.replace(' ', '_').lower()
    return sanitized or 'untitled'
```

**Examples**:
- "Adventurer" → `adventurer.json`
- "Guard Captain" → `guard_captain.json`
- "Sir Edward III" → `sir_edward_iii.json`

## LLM Generation

Actors can be generated by LLM (generate_actor.py):

### Generatable Fields

```python
GENERATABLE_FIELDS = [
    "name", "description", "personality",
    "appearance", "goals", "story", "equipment"
]
```

### Generation Order

Fields are generated sequentially in this order to maintain coherence:

1. **name** - Character name
2. **description** - Background and role
3. **personality** - Personality traits
4. **appearance** - Physical description
5. **goals** - Motivations and ambitions
6. **story** - Backstory
7. **equipment** - Worn items

### Generation Prompts

**Name** (lines 78-92):
- Output: Just the name, no formatting
- Example: "Elara Moonwhisper"

**Description** (lines 94-98):
- Output: Single cohesive paragraph
- Focus: Who they are, what they do, place in world
- Example: "A weathered traveler seeking ancient artifacts..."

**Personality** (lines 100-103):
- Output: 15-25 comma-separated traits
- Focus: Traits, behaviors, values, quirks
- Example: "curious, cautious, determined, witty, loyal..."

**Appearance** (lines 104-108):
- Output: 15-25 comma-separated physical descriptors
- Focus: Build, height, hair, eyes, features, marks
- Excludes: Clothing and equipment
- Example: "tall, lean build, weathered skin, blue eyes..."

**Goals** (lines 109-110):
- Output: Plain text list
- Focus: Short-term and long-term ambitions with reasons

**Story** (lines 111-112):
- Output: Plain text narrative
- Focus: Key events, relationships, turning points

**Equipment** (lines 114-165):
- Output: JSON object with all 17 slots
- Context-aware: Matches character theme, genre, station
- Special handling: Multi-retry with validation

### Generation Configuration

```python
make_inference(
    context=[{"role": "user", "content": prompt}],
    user_message=prompt,
    character_name=character_name,
    url_type=url_type,
    max_tokens=512 if field == 'equipment' else 256,
    temperature=0.7,
    is_utility_call=True
)
```

## State Transitions

### Valid Transitions

- **location**: Changed via move action, validated against world settings
- **left_hand_holding / right_hand_holding**: Updated via pickup/drop actions
- **equipment slots**: Modified via equip/unequip actions
- **variables**: Updated by rules, timers, or script actions
- **npc_notes**: Appended with timestamped entries
- **relations**: Modified through interactions

### Player Character Special Rules

1. **Only one player**: `isPlayer=true` allowed on exactly one actor
2. **Origin tracking**: Player's original location stored in `variables.json:origin`
3. **Location sync**: Player location determines current setting for UI display

## Integration Points

### With Settings

Actors reference settings via the `location` field:

```python
# Find player's current setting
from core.utils import _get_player_current_setting_name

current_setting = _get_player_current_setting_name(workflow_data_dir)
# Returns: "Tavern" or "Unknown Setting"
```

Settings maintain a `characters` array listing actors present:

```json
{
  "name": "Tavern",
  "characters": ["Adventurer", "Innkeeper", "Guard Captain"]
}
```

### With Items/Inventory

- `left_hand_holding` / `right_hand_holding`: References item names
- `equipment` slots: References wearable item names
- Full inventory system managed separately (see Item Schema)

### With Rules/Timers

Actor variables are readable/writable by Rules Engine and Timer system with scope resolution:

```python
# Rule condition checking actor variable
if actor_data['variables'].get('health', 100) < 20:
    trigger_low_health_event()
```

### With Memory System

Actor data is cached in memory for performance (utils.py:841-898):
- Cache built on first access
- Searches session (`game/actors/`) before base (`resources/data files/actors/`)
- Normalized name lookup for case-insensitive matching

## Validation Rules

Based on code inspection:

1. **Name**: Required, max 80 characters, unique per workflow
2. **isPlayer**: Boolean, only one actor can have `true`
3. **location**: Must reference existing setting name
4. **equipment**: All 17 slots must be present (can be empty strings)
5. **relations**: Keys are character names, values are relationship descriptions
6. **variables**: Any JSON-serializable key-value pairs
7. **schedule**: Time format HH:MM, location must exist

## Cross-References

- **Used By**: [[Game Engine]], [[Rule Evaluator]], [[Timer System]], [[Character Inference]]
- **References**: [[Setting Schema]], [[Item Schema]], [[Variables System]]
- **Validates**: Discord claims about character management in ChatBotRPG
- **Related Patterns**: [[Three-Tier Persistence Pattern]], [[JIT Generation]]

## Notes

- Actors exist in two directories: `resources/data files/actors/` (base templates) and `game/actors/` (session instances)
- Session actors override base actors with same name
- Equipment system separates WORN items (in equipment slots) from HELD items (in hand_holding fields)
- Portrait system supports customization with offset, scale, and tint
- Schedule system enables time-based NPC behavior
- Relations are one-directional (not bidirectional links)
- The `npc_notes` field stores persistent conversation memory with timestamps
