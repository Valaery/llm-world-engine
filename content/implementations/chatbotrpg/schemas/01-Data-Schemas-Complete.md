---
tags: [implementation, chatbotrpg, schemas, data-models, json]
created: 2026-01-21
agent: schema-archaeologist
status: complete
---

# ChatBotRPG Data Schemas - Complete Documentation

## Overview

Complete documentation of all JSON data schemas used in ChatBotRPG for persistence, configuration, and game state management.

**Analysis Date**: 2026-01-21
**Source**: ChatBotRPG `/src/editor_panel/` manager files
**Storage Format**: JSON files (no SQLite database)
**Persistence Model**: Two-tier (Resources + Game)

---

## Directory Structure

```
workflow_data_dir/
├── resources/data files/          # World-tier (immutable templates)
│   ├── actors/                    # Character templates
│   ├── settings/                  # Location/world templates
│   ├── items/                     # Item templates by category
│   ├── keywords/                  # Keyword definitions
│   ├── worldnotes.json           # World-level notes
│   └── time_passage.json         # Time configuration
├── game/                          # Playthrough-tier (mutable state)
│   ├── actors/                    # Active character states
│   ├── settings/                  # Active location states
│   ├── conversation.json          # Chat history
│   ├── variables.json             # Global variables
│   ├── rules.json                 # Active rules
│   └── timer_rules.json           # Timer-based rules
└── saves/                         # Save states (copies of game/)
    └── {save_name}/               # Named save directories
```

---

## Schema 1: Actor (Character)

**Purpose**: Represents NPCs and player characters with full state
**File Location**: `actors/{character_name}.json`
**Manager**: `actor_manager.py`

### Schema Definition

```json
{
  "name": "string",                    // Display name
  "isPlayer": boolean,                 // Player character flag
  "description": "string",             // Physical description
  "personality": "string",             // Personality traits
  "appearance": "string",              // Detailed appearance
  "portrait": "string",                // Portrait file path
  "portrait_offset": [int, int],       // Portrait display offset
  "portrait_scale": float,             // Portrait scale factor
  "portrait_tint": "string",           // Hex color for tint
  "location": "string",                // Current setting name
  "relationships": {                   // Character relationships
    "{character_name}": {
      "type": "string",                // Relationship type
      "description": "string"          // Relationship details
    }
  },
  "variables": {                       // Character-scoped variables
    "{var_name}": any                  // Any JSON type
  },
  "equipment": {                       // 16-slot layered equipment
    "head": "string",
    "neck": "string",
    "left_shoulder": "string",
    "right_shoulder": "string",
    "left_hand": "string",             // Worn items
    "right_hand": "string",            // Worn items
    "upper_outer": "string",
    "upper_middle": "string",
    "upper_inner": "string",
    "lower_outer": "string",
    "lower_middle": "string",
    "lower_inner": "string",
    "left_foot_inner": "string",
    "right_foot_inner": "string",
    "left_foot_outer": "string",
    "right_foot_outer": "string"
  },
  "inventory": [                       // Item instances
    {
      "item_id": "string",             // Unique instance ID
      "resource_item": "string",       // Item template name
      "resource_category": "string",   // Item category
      "quantity": int,                 // Stack size
      "durability": int,               // 0-100%
      "description": "string",         // Custom description
      "custom_properties": {},         // Any JSON object
      "contains": []                   // Nested items (containers)
    }
  ],
  "npc_notes": "string",               // Auto-generated NPC memories
  "schedule": [                        // Daily schedule
    {
      "time": "HH:MM",                 // 24-hour format
      "activity": "string",            // Activity description
      "location": "string"             // Location name
    }
  ],
  "follower_memory": [                 // Recent scene memories
    {
      "scene": int,                    // Scene number
      "summary": "string"              // Scene summary
    }
  ]
}
```

### Equipment System Details

**16-Slot Layered System**:
- Head → Neck → Shoulders (2) → Hands (2, worn) → Upper Body (3 layers) → Lower Body (3 layers) → Feet (4, inner/outer per foot)
- **Inner-to-Outer Rendering**: Inner layers appear under outer layers
- **Used in Prompt**: Equipment descriptions generated with all 16 slots (512-token budget)

**Source**: `actor_manager.py` lines 38-57

---

## Schema 2: Setting (Location/World)

**Purpose**: Represents geographic locations with nested hierarchy
**File Location**: `settings/{world_name}/{region_name}/{location_name}_setting.json`
**Manager**: `setting_manager.py`

### Schema Definition

```json
{
  "name": "string",                    // Location name
  "description": "string",             // Location description
  "level": "string",                   // "World", "Region", "Location", "Setting"
  "parent": "string",                  // Parent location name
  "world": "string",                   // World name
  "region": "string",                  // Region name (if applicable)
  "characters": ["string"],            // Character names present
  "connections": [                     // Connected locations
    {
      "destination": "string",         // Destination setting name
      "description": "string"          // Connection description
    }
  ],
  "variables": {                       // Setting-scoped variables
    "{var_name}": any
  },
  "features": [                        // World map features
    {
      "type": "string",                // Feature type (e.g., "mountain")
      "position": [int, int],          // Grid coordinates
      "properties": {}                 // Feature-specific data
    }
  ],
  "grid_width": int,                   // World map width
  "grid_height": int,                  // World map height
  "tile_size": int                     // Tile pixel size
}
```

### Hierarchy Levels

1. **World**: Top-level geography (e.g., "Middle Earth")
2. **Region**: Sub-area of world (e.g., "The Shire")
3. **Location**: Specific place (e.g., "Bag End")
4. **Setting**: Player-occupiable scene (e.g., "Bilbo's Study")

**Source**: `setting_manager.py` lines 22-55

---

## Schema 3: Item Template

**Purpose**: Reusable item definitions
**File Location**: `items/{category}/{item_name}.json`
**Manager**: `inventory_manager.py`

### Schema Definition

```json
{
  "name": "string",                    // Item name
  "category": "string",                // Item category
  "description": "string",             // Item description
  "weight": float,                     // Weight in units
  "value": int,                        // Base value in currency
  "stackable": boolean,                // Can stack in inventory
  "max_stack": int,                    // Max stack size
  "durability_max": int,               // Max durability (0-100)
  "properties": {                      // Item-specific properties
    "{property_name}": any
  },
  "tags": ["string"]                   // Searchable tags
}
```

### Item Instance (in Actor inventory)

```json
{
  "item_id": "string",                 // UUID for instance
  "resource_item": "string",           // References item template
  "resource_category": "string",       // Category path
  "quantity": int,                     // Stack size
  "durability": int,                   // Current durability %
  "description": "string",             // Instance description
  "custom_properties": {               // Runtime properties
    "enchantment": "string",
    "crafted_by": "string"
  },
  "contains": [                        // Nested items (for containers)
    {/* nested item instance */}
  ]
}
```

**Container Support**: Items can contain other items (recursive nesting)
**Source**: `inventory_manager.py` lines 121-139

---

## Schema 4: Keyword Definition

**Purpose**: Context injection via keyword triggers
**File Location**: `keywords/{category}.json`
**Manager**: `keyword_manager.py`

### Schema Definition

```json
{
  "category": "string",                // Keyword category
  "keywords": {                        // Keyword map
    "{keyword}": [                     // Keyword text
      {
        "entry": "string",             // Context to inject
        "priority": int                // Injection order (higher first)
      }
    ]
  }
}
```

### Usage Pattern

1. User message contains keyword (case-insensitive match)
2. System finds all matching keywords across categories
3. Entries injected into context by priority
4. Multiple entries per keyword supported

**Source**: `keyword_manager.py` lines 16-100

---

## Schema 5: Rule Definition

**Purpose**: If-then logic for dynamic narrative control
**File Location**: `game/rules.json`
**Manager**: `rules_manager.py`

### Schema Definition

```json
{
  "rules": [
    {
      "id": "string",                  // Unique rule ID
      "description": "string",         // Human-readable description
      "enabled": boolean,              // Active flag
      "start_conditions": [            // Conditions to check
        {
          "type": "string",            // Condition type (see below)
          "operator": "string",        // Comparison operator
          "value": any,                // Comparison value
          "variable": "string",        // Variable name (if applicable)
          "variable_scope": "string",  // "Global" | "Character" | "Player" | "Setting"
          "geography_name": "string"   // Location name (if applicable)
        }
      ],
      "actions": [                     // Actions to execute
        {
          "type": "string",            // Action type (see below)
          "mode": "string",            // "Prepend" | "Append" | "Replace"
          "scope": "string",           // "user_message" | "llm_reply" | "full_conversation"
          "tag": "string",             // Prompt tag
          "content": "string",         // Prompt content
          "variable": "string",        // Variable to modify
          "operator": "string",        // Math operator
          "value": any                 // New value
        }
      ],
      "trigger": "string"              // "before_send" | "after_receive" | "always"
    }
  ]
}
```

### Condition Types

1. **Setting**: Current location name matches
2. **Location**: Current location matches
3. **Region**: Current region matches
4. **World**: Current world matches
5. **Variable**: Variable comparison (`==`, `!=`, `>`, `<`, `>=`, `<=`, `contains`, `not contains`, `exists`, `not exists`)
6. **Scene Count**: Scene number comparison
7. **Game Time**: Game time comparison (seconds, minutes, hours, days)
8. **Post Dialogue**: Dialogue detection in last post

### Action Types

1. **Inject Prompt**: Add text to context
2. **Set Variable**: Modify variable value
3. **Move Character**: Change character location
4. **Trigger Event**: Fire custom event
5. **Screen Effect**: Visual effect (flash, shake, fade)
6. **Play Sound**: Audio playback

**Source**: `rules_manager.py` lines 1-150

---

## Schema 6: Timer Rule Definition

**Purpose**: Time-based automatic actions
**File Location**: `game/timer_rules.json`
**Manager**: `timer_manager.py` + `timer_rules_manager.py`

### Schema Definition

```json
{
  "timer_rules": [
    {
      "id": "string",                  // Unique timer ID
      "description": "string",         // Human-readable description
      "enabled": boolean,              // Active flag
      "character": "string",           // Character to trigger (or null for global)
      "time_mode": "string",           // "real_time" | "game_time"
      "interval": int,                 // Seconds (if not random)
      "interval_is_random": boolean,   // Use random interval
      "interval_min": int,             // Min seconds (if random)
      "interval_max": int,             // Max seconds (if random)
      "game_seconds": int,             // Game time seconds
      "game_minutes": int,             // Game time minutes
      "game_hours": int,               // Game time hours
      "game_days": int,                // Game time days
      "game_seconds_is_random": boolean,
      "game_seconds_min": int,
      "game_seconds_max": int,
      "game_minutes_is_random": boolean,
      "game_minutes_min": int,
      "game_minutes_max": int,
      "game_hours_is_random": boolean,
      "game_hours_min": int,
      "game_hours_max": int,
      "game_days_is_random": boolean,
      "game_days_min": int,
      "game_days_max": int,
      "conditions": [                  // Start conditions (same as regular rules)
        {/* condition object */}
      ],
      "actions": [                     // Actions to execute
        {/* action object */}
      ]
    }
  ]
}
```

### Timer Behavior

- **Real Time**: Interval in wall-clock seconds
- **Game Time**: Interval in game-world time (scaled by time multiplier)
- **Time Multiplier**: Configured in `time_passage.json` (default: 1.0)
- **Random Intervals**: Recalculated after each fire
- **Character Timers**: Trigger NPC actions (e.g., wandering, scheduled activities)

**Example**: Timer fires every 5-10 minutes (real time), making NPC post a message

**Source**: `timer_manager.py` lines 11-150

---

## Schema 7: Conversation History

**Purpose**: Complete chat history with metadata
**File Location**: `game/conversation.json`
**Manager**: `chatBotRPG.py` (main application)

### Schema Definition

```json
{
  "context": [
    {
      "role": "string",                // "system" | "user" | "assistant"
      "content": "string",             // Message text
      "scene": int,                    // Scene number
      "datetime": "string",            // Game datetime (ISO format)
      "metadata": {
        "character": "string",         // Speaking character
        "location": "string",          // Location name
        "token_count": int,            // Token count (if available)
        "model": "string",             // Model used
        "post_visibility": {           // Visibility rules
          "mode": "string",            // "Visible Only To" | "Hidden From"
          "condition_type": "string",  // "Name Match" | "Variable"
          "actor_names": ["string"],   // Character names
          "variable_conditions": []    // Variable conditions
        }
      }
    }
  ],
  "scene_number": int                  // Current scene
}
```

### Visibility System

**Purpose**: Control which characters "see" which messages
**Modes**:
- **Visible Only To**: Only listed characters see message
- **Hidden From**: All except listed characters see message

**Condition Types**:
- **Name Match**: By character name
- **Variable**: By variable condition

**Usage**: NPCs only receive context they should see (prevents meta-gaming)

**Source**: `utils.py` lines 565-627

---

## Schema 8: Global Variables

**Purpose**: Game-wide state tracking
**File Location**: `game/variables.json`
**Manager**: `chatBotRPG.py` (main application)

### Schema Definition

```json
{
  "{variable_name}": any,              // Global variables (any JSON type)
  "origin": "string",                  // Origin setting name
  "scene_number": int,                 // Current scene number
  "game_datetime": "string"            // Game datetime (ISO format)
}
```

### Special Variables

- **origin**: Starting location for player character
- **scene_number**: Current scene (increments on scene boundary)
- **game_datetime**: Current in-game date/time

**Source**: `utils.py` lines 978-1050

---

## Schema 9: Time Passage Configuration

**Purpose**: Control game time flow
**File Location**: `resources/data files/settings/time_passage.json`
**Manager**: `time_manager.py`

### Schema Definition

```json
{
  "time_multiplier": float,            // Real seconds per game second (e.g., 60.0 = 1 min real = 1 hr game)
  "current_datetime": "string",        // Current game datetime (ISO format)
  "start_datetime": "string",          // Game start datetime (ISO format)
  "format_string": "string"            // Display format (e.g., "%Y-%m-%d %H:%M")
}
```

### Time Multiplier Examples

- `1.0`: Real-time (1 real second = 1 game second)
- `60.0`: 1 real minute = 1 game hour
- `3600.0`: 1 real hour = 1 game day

**Source**: `timer_manager.py` lines 80-98

---

## Schema 10: World Notes

**Purpose**: World-level documentation
**File Location**: `resources/data files/worldnotes.json`
**Manager**: `notes_manager.py`

### Schema Definition

```json
{
  "fields": [
    {
      "name": "string",                // Field name
      "content": "string"              // Field content
    }
  ]
}
```

**Usage**: Free-form note-taking for world-building

**Source**: `notes_manager.py` lines 14-18

---

## Persistence Model

### Two-Tier Architecture

**Tier 1: Resources** (`resources/data files/`)
- Immutable templates
- Shared across all playthroughs
- Examples: Actor templates, item templates, world geography

**Tier 2: Game** (`game/`)
- Mutable playthrough state
- Player-specific data
- Examples: Active characters, conversation history, variables

### Save/Load System

**Save Operation**:
1. Copy entire `game/` directory to `saves/{save_name}/`
2. Include timer state
3. Atomic operation (rename old files before copy)

**Load Operation**:
1. Clear current tab state (memory, context, UI)
2. Rename existing `game/` files (backup)
3. Copy `saves/{save_name}/` contents to `game/`
4. Reload conversation, variables, timers
5. Clean up backup files after 10 minutes

**Source**: `utils.py` lines 19-513

---

## Data Migration

### Evolution: JSON → Current System

**Historical Context** (from git-history-miner):
- Original system used monolithic JSON files
- Migrated to directory-based structure for scalability
- Equipment system added later (16-slot layered system)
- Timer rules separated from regular rules
- Visibility system added for multi-character support

**No SQL Database**: Despite `.world` and `.save` filename patterns discussed in Discord, the actual implementation uses **JSON files only**.

---

## Schema Validation

### JSON Format Enforcement

**Loading Pattern** (all managers):
```python
try:
    with open(file_path, 'r', encoding='utf-8') as f:
        content = f.read().strip()
        if not content:
            return {}
        data = json.loads(content)
        if not isinstance(data, dict):
            return {}
        return data
except (json.JSONDecodeError, IOError, OSError):
    return {}
```

**Saving Pattern**:
```python
try:
    with open(file_path, 'w', encoding='utf-8') as f:
        json.dump(data, f, indent=2, ensure_ascii=False)
        f.flush()
        os.fsync(f.fileno())  # Force write to disk
    return True
except (IOError, OSError):
    return False
```

**Source**: `utils.py` lines 900-928

---

## Performance Considerations

### File I/O Optimization

1. **Lazy Loading**: Files loaded only when editor panel opens
2. **Debounced Saving**: 1-second timer before saving edits
3. **Caching**: Actor file paths cached for fast lookup
4. **Atomic Writes**: fsync() ensures data integrity

### Memory Management

1. **Conversation Cleanup**: Old messages cleared on load
2. **Actor Cache Rebuild**: Cache rebuilt on location changes
3. **Timer Persistence**: Timer state saved on tab switch

**Source**: Multiple manager files

---

## Cross-References

### Related Discord Analysis Files

- [[Three-Tier Persistence]] - Discusses 3-tier model (Session tier implicit here)
- [[World-Playthrough-Session Pattern]] - Architecture pattern
- [[Equipment Generation Prompt]] - Uses 16-slot equipment schema
- [[NPC Memory System]] - Uses `npc_notes` and `follower_memory` fields
- [[Variable Scoping]] - Uses variable scopes in rules/conditions

### Related ChatBotRPG Analysis Files

- [[01-Repository-Overview]] - Architecture overview
- [[02-Pattern-Implementation]] - Persistence pattern implementation
- [[03-Prompt-Implementation]] - Prompts that use these schemas
- [[04-Code-Examples]] - Code examples for schema manipulation

---

## Key Insights

### Schema Design Principles

1. **Flat JSON**: No complex nesting (except inventory containers)
2. **String-Based References**: Entity references by name (not IDs)
3. **Flexible Variables**: Generic `variables` dict for extensibility
4. **Hierarchical Geography**: World → Region → Location → Setting
5. **Layered Equipment**: 16-slot system for detailed descriptions

### Production-Tested Patterns

1. **Debounced Saves**: Prevents excessive file I/O
2. **Backup-Before-Overwrite**: Atomic save/load operations
3. **Fallback Defaults**: Empty dict/array if file missing
4. **Normalized Names**: Lowercase, underscore-separated for lookups

---

## Statistics

**Total Schemas Documented**: 10 core schemas
**Additional Sub-Schemas**: 8 (conditions, actions, items, etc.)
**Lines of Schema Code**: ~1,500 lines analyzed
**Manager Files**: 10 files
**Storage Format**: JSON (no SQL)
**Persistence Tiers**: 2 (Resources + Game)

---

## Tags

#schemas #data-models #json #persistence #two-tier #chatbotrpg #production-validated
