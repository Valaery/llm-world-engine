---
tags: [schema, chatbotrpg, implementation, setting, location, world]
created: 2026-01-20
---

# Setting Schema (World/Region/Location)

**Primary Files**:
- `src/editor_panel/setting_manager.py` (UI and management, lines 1-2000+)
- `src/generate/generate_setting.py` (LLM generation, lines 1-404)
- `src/editor_panel/world_editor/` (Visual map editor)

**Storage Location**:
- Base: `{workflow_dir}/resources/data files/settings/**/*_setting.json`
- Session (runtime): `{workflow_dir}/game/settings/**/*_setting.json`

**Hierarchy**: World → Region → Location → Setting

## Purpose

Represents a hierarchical spatial structure for the game world. Settings can be Worlds (continents), Regions (provinces), Locations (cities/dungeons), or specific places (rooms/areas) with connections forming a navigable map.

## Hierarchical Structure

```
World (Continent)
├── Region (Province/Territory)
│   ├── Location (City/Dungeon)
│   │   ├── Setting (Tavern)
│   │   ├── Setting (Market Square)
│   │   └── Setting (Guard House)
│   └── Location (Forest)
│       ├── Setting (Clearing)
│       └── Setting (Ancient Ruins)
└── Region (Another Province)
```

## Complete JSON Structure

```json
{
  "name": "Tavern",
  "description": "A warm, rustic tavern filled with the smell of ale and roasted meat. Wooden tables and chairs crowd the main room, with a large fireplace crackling in the corner.",

  "world": "Aethermoor",
  "region": "Merchant District",
  "location": "Port City",

  "features": [
    {
      "feature_name": "Fireplace",
      "feature_description": "Large stone fireplace, always burning, provides warmth"
    },
    {
      "feature_name": "Bar Counter",
      "feature_description": "Long oak bar with brass fixtures, bottles lined on shelves"
    },
    {
      "feature_name": "Staircase",
      "feature_description": "Wooden stairs leading to guest rooms on second floor"
    }
  ],

  "connections": {
    "Market Square": "North through main door onto cobblestone plaza",
    "Harbor": "East down narrow alley past crates and barrels",
    "Guard House": "West along paved street near city walls"
  },

  "characters": [
    "Innkeeper",
    "Bard",
    "Guard Captain",
    "Merchant",
    "Player"
  ],

  "inventory": [
    "Ale barrel",
    "Wine bottles",
    "Roasted meat",
    "Playing cards",
    "Guest registry book"
  ],

  "variables": {
    "tavern_reputation": 75,
    "ales_in_stock": 12,
    "rooms_available": 3,
    "is_open": true,
    "last_brawl_date": "2024-01-15",
    "travel_notes_to_market_square": "Short walk north through busy streets",
    "travel_notes_to_harbor": "Quick eastward path through merchant quarter"
  },

  "paths": [
    {
      "path_name": "Main Road",
      "path_description": "Wide cobblestone road suitable for wagons and carts"
    },
    {
      "path_name": "Alley",
      "path_description": "Narrow passage between buildings, dimly lit"
    }
  ],

  "world_map_data": {
    "position_x": 150,
    "position_y": 200,
    "icon": "tavern",
    "color": "#8B4513"
  }
}
```

## Field Documentation

| Field | Type | Required | Default | Description | UI Location |
|-------|------|----------|---------|-------------|-------------|
| **name** | string | Yes | - | Setting name (unique within hierarchy) | setting_manager.py:211-214 |
| **description** | string | No | "" | Descriptive text about the setting (1-2 paragraphs) | setting_manager.py:215-219 |
| **world** | string | No | "" | Parent world name | World list selector |
| **region** | string | No | "" | Parent region name | Region list selector |
| **location** | string | No | "" | Parent location name | Location list selector |
| **features** | array | No | [] | Notable features/objects in this setting | setting_manager.py:225-263 |
| **connections** | object | No | {} | Connected settings with path descriptions | Generated or manual |
| **characters** | array | No | [] | Actor names present in this setting | Managed by actor system |
| **inventory** | array | No | [] | Items/objects available in this setting | Generated or manual |
| **variables** | object | No | {} | Custom setting variables and travel notes | setting_manager.py:variables |
| **paths** | array | No | [] | Custom path types for this setting | setting_manager.py:265-308 |
| **world_map_data** | object | No | {} | Visual map editor data | world_editor.py |

## Hierarchical Levels

### World Schema

The top level of the hierarchy representing continents or realms:

```json
{
  "name": "Aethermoor",
  "description": "A vast continent of floating islands suspended by ancient magic, connected by ethereal bridges and skyship routes.",
  "features": [
    {
      "feature_name": "Ethereal Bridges",
      "feature_description": "Shimmering magical constructs connecting major islands"
    },
    {
      "feature_name": "Skyship Ports",
      "feature_description": "Floating docks where airships moor"
    }
  ],
  "paths": [
    {
      "path_name": "Skyway",
      "path_description": "Magical flight path for skyships"
    },
    {
      "path_name": "Ethereal Bridge",
      "path_description": "Glowing magical walkway between islands"
    }
  ]
}
```

### Region Schema

Mid-level representing provinces or territories:

```json
{
  "name": "Merchant District",
  "description": "Bustling commercial hub of the city, filled with shops, markets, and trading posts.",
  "world": "Aethermoor",
  "features": [
    {
      "feature_name": "Grand Bazaar",
      "feature_description": "Covered market with hundreds of stalls"
    }
  ]
}
```

### Location Schema

Specific places like cities, dungeons, or wilderness areas:

```json
{
  "name": "Port City",
  "description": "A thriving coastal city built around a natural harbor, serving as major trade hub.",
  "world": "Aethermoor",
  "region": "Merchant District",
  "features": [
    {
      "feature_name": "Harbor",
      "feature_description": "Deep water port with stone docks"
    }
  ]
}
```

### Setting Schema

The most specific level, representing individual rooms or areas:

```json
{
  "name": "Tavern",
  "description": "Warm rustic tavern...",
  "world": "Aethermoor",
  "region": "Merchant District",
  "location": "Port City"
}
```

## Features Schema

Array of notable objects or points of interest:

```json
{
  "feature_name": "Fireplace",
  "feature_description": "Large stone fireplace, always burning, provides warmth and light"
}
```

| Field | Type | Required | Description | UI |
|-------|------|----------|-------------|-----|
| feature_name | string | Yes | Name of the feature | setting_manager.py:250 |
| feature_description | string | Yes | Detailed description | setting_manager.py:253 |

**UI Management** (setting_manager.py:225-263):
- Displayed in list widget
- Add/Remove buttons
- Selected feature shows in editable fields
- Separate for World and Location levels

## Connections Schema

Connections define travel routes between settings:

```json
{
  "connections": {
    "Market Square": "North through main door onto cobblestone plaza",
    "Harbor": "East down narrow alley past crates and barrels"
  }
}
```

**Format**: `{ "DestinationName": "Travel description" }`

**Travel Notes Variables**: Connections automatically create variables for navigation memory:

```json
{
  "variables": {
    "travel_notes_to_market_square": "North through main door onto cobblestone plaza",
    "travel_notes_to_harbor": "East down narrow alley past crates and barrels"
  }
}
```

**Variable Name Pattern**: `travel_notes_to_{sanitized_destination_name}`

## Path Types Schema

Custom path definitions for map connections:

```json
{
  "path_name": "Main Road",
  "path_description": "Wide cobblestone road suitable for wagons and carts"
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| path_name | string | Yes | Type of path (e.g., "Main Road", "Forest Trail") |
| path_description | string | Yes | Description of path characteristics |

**Default Path Types** (setting_manager.py:276-278):
- "Large Path (Default)"
- "Medium Path (Default)"
- "Small Path (Default)"

**Usage**: World Editor uses path types to determine visual representation and connection properties.

## World Map Data Schema

Visual editor metadata for map display:

```json
{
  "world_map_data": {
    "position_x": 150,
    "position_y": 200,
    "icon": "tavern",
    "color": "#8B4513",
    "region_color": "#4A90E2",
    "polygon_points": [[100, 150], [200, 150], [200, 250], [100, 250]]
  }
}
```

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| position_x | number | 0 | X coordinate on world map |
| position_y | number | 0 | Y coordinate on world map |
| icon | string | "default" | Icon identifier |
| color | string | "#FFFFFF" | Color for this setting |
| region_color | string | null | Region boundary color |
| polygon_points | array | [] | Vertices for region/location boundaries |

## Variables Schema

Settings can store custom variables for state tracking:

```json
{
  "variables": {
    "tavern_reputation": 75,
    "ales_in_stock": 12,
    "rooms_available": 3,
    "is_open": true,
    "last_brawl_date": "2024-01-15",
    "travel_notes_to_market_square": "Short walk north",
    "discovered": true,
    "danger_level": 3
  }
}
```

**Variable Types**:
- **Numeric**: Counters, stats, levels
- **Boolean**: Flags (discovered, is_open, is_hostile)
- **String**: Descriptions, dates, names
- **Special**: `travel_notes_to_*` (auto-generated from connections)

**Variable Scope**: Setting-scoped variables accessible by Rule Engine and Timer System with scope="Setting".

## File Operations

### Finding Settings

Settings are organized hierarchically in the filesystem:

```
settings/
├── world_name/
│   ├── world_name_setting.json          # World definition
│   ├── region_name/
│   │   ├── region_name_setting.json      # Region definition
│   │   └── location_name/
│   │       ├── location_name_setting.json # Location definition
│   │       ├── tavern_setting.json        # Individual settings
│   │       └── market_square_setting.json
```

### Loading a Setting

```python
from core.utils import _find_setting_file_by_name, _load_json_safely

# Find setting by name (searches recursively)
settings_dir = os.path.join(workflow_data_dir, 'resources', 'data files', 'settings')
setting_file = _find_setting_file_by_name(settings_dir, "Tavern")

# Load setting data
setting_data = _load_json_safely(setting_file)
```

### Saving a Setting

```python
from generate.generate_setting import _save_json

setting_data = {
    "name": "New Location",
    "description": "A mysterious place...",
    "world": "Aethermoor",
    # ... other fields
}

save_path = os.path.join(settings_dir, "world/region/new_location_setting.json")
_save_json(save_path, setting_data)
```

### Setting Filename Convention

Settings use the suffix `_setting.json`:

```python
def sanitize_path_name(name):
    sanitized = re.sub(r'[^a-zA-Z0-9_\-\. ]', '', name).strip()
    sanitized = sanitized.replace(' ', '_').lower()
    return sanitized or 'untitled'

filename = f"{sanitize_path_name(setting_name)}_setting.json"
```

**Examples**:
- "Tavern" → `tavern_setting.json`
- "Market Square" → `market_square_setting.json`
- "The Ancient Ruins" → `the_ancient_ruins_setting.json`

## LLM Generation

Settings can be generated by LLM (generate_setting.py):

### Generatable Fields

```python
ordered_fields = []
if options.get("description", False): ordered_fields.append("description")
if options.get("name", False): ordered_fields.append("name")
if options.get("connections", False): ordered_fields.append("connections")
if options.get("inventory", False): ordered_fields.append("inventory")
```

### Generation Context

The LLM receives hierarchical context:

```python
def _prepare_context_for_description(self) -> str:
    context_parts = []
    context_parts.append(f"WORLD INFORMATION:")
    context_parts.append(f"World Name: {self.world_data.get('name')}")
    context_parts.append(f"World Description: {self.world_data.get('description')}")

    if self.region_data:
        context_parts.append(f"REGION INFORMATION:")
        context_parts.append(f"Region Name: {self.region_data.get('name')}")
        context_parts.append(f"Region Description: {self.region_data.get('description')}")

    if self.location_data:
        context_parts.append(f"LOCATION INFORMATION:")
        context_parts.append(f"Location Name: {self.location_data.get('name')}")
        context_parts.append(f"Location Description: {self.location_data.get('description')}")

    if self.containing_features_data:
        context_parts.append(f"CONTAINING FEATURES:")
        for feature in self.containing_features_data:
            context_parts.append(f"- {feature.get('feature_name')}: {feature.get('feature_description')}")

    return "\n".join(context_parts)
```

### Generation Prompts

**Description** (lines 212):
- Output: 1-2 clear sentences, short and simple
- Focus: Core essence of the setting
- Example: "A warm, rustic tavern filled with the smell of ale..."

**Name** (lines 214):
- Output: 1-4 words, concise and evocative
- No explanations, lists, or punctuation
- Example: "The Rusty Anchor"

**Connections** (lines 216):
- Output: JSON object with location names as keys
- Format: `{"LocationName": "Brief path description"}`
- Focus: Unique, memorable path descriptions
- Excludes: Cardinal directions (north/south/etc.)
- Example: `{"Market Square": "Narrow dirt path", "Forest": "Steep rocky trail"}`

**Inventory** (lines 218):
- Output: JSON array of strings
- Focus: 3-7 notable items or objects
- Purpose: Interactable, collectible, or atmospheric
- Example: `["Ale barrel", "Wine bottles", "Playing cards"]`

### Generation Configuration

```python
make_inference(
    context=[{"role": "user", "content": prompt}],
    user_message=prompt,
    character_name=setting_name,
    url_type=url_type_to_use,
    max_tokens=800 if field == "description" else 400,
    temperature=0.7,
    is_utility_call=True
)
```

### Auto-Variable Creation

When connections are generated, travel notes variables are automatically created (lines 105-112):

```python
if field == "connections" and isinstance(generated_value, dict):
    variables = {}
    for connected_setting, description in generated_value.items():
        var_name = f"travel_notes_to_{sanitize_path_name(connected_setting)}"
        variables[var_name] = description

    if "variables" not in self.current_setting_data:
        self.current_setting_data["variables"] = {}

    self.current_setting_data["variables"].update(variables)
```

## State Transitions

### Valid Transitions

- **characters**: Updated when actors move via `location` field change
- **inventory**: Modified by item pickup/drop actions
- **variables**: Updated by rules, timers, or script actions
- **connections**: Static (set during creation or editing)
- **description/name**: Can be regenerated or manually edited

### Character Movement

When an actor's `location` field changes:

1. Remove actor name from old setting's `characters` array
2. Add actor name to new setting's `characters` array
3. Update actor's `location` field
4. Save both setting files

```python
# Example from generate_actor.py:667-739
def _add_actor_to_setting(actor_name, location, workflow_data_dir):
    # Find setting file
    setting_file, setting_data = find_setting_file(location)

    # Update characters array
    characters = setting_data.get('characters', [])
    if actor_name not in characters:
        characters.append(actor_name)
        setting_data['characters'] = characters
        _save_json(setting_file, setting_data)
```

## Player Current Setting

The game tracks which setting contains the player:

```python
from core.utils import _get_player_current_setting_name

current_setting_name = _get_player_current_setting_name(workflow_data_dir)
# Returns: "Tavern" or "Unknown Setting"
```

**Logic** (utils.py:997-1050):
1. Find player character file (actor with `isPlayer=true`)
2. Search session settings (`game/settings/`) for setting containing player name in `characters` array
3. If not found in session, search base settings (`resources/data files/settings/`)
4. If still not found, return `origin` from `variables.json`
5. If no origin, return "Unknown Setting"

## Setting Rename Handling

When a setting name changes, all references are updated (generate_setting.py:313-361):

```python
def _handle_setting_rename(self, old_name, new_name):
    # Rename file
    new_filepath = os.path.join(dir_path, f"{sanitize_path_name(new_name)}_setting.json")
    shutil.move(self.setting_filepath, new_filepath)

    # Update all connections referencing this setting
    self._update_setting_references(old_name, new_name)

def _update_setting_references(self, old_name, new_name):
    # Search all setting files
    for file_path in all_setting_files:
        setting_data = _load_json(file_path)
        connections = setting_data.get('connections', {})

        # Update connection key
        if old_name in connections:
            connections[new_name] = connections.pop(old_name)
            setting_data['connections'] = connections

            # Update travel notes variable
            old_var_name = f"travel_notes_to_{sanitize_path_name(old_name)}"
            if old_var_name in setting_data['variables']:
                new_var_name = f"travel_notes_to_{sanitize_path_name(new_name)}"
                setting_data['variables'][new_var_name] = setting_data['variables'].pop(old_var_name)

            _save_json(file_path, setting_data)
```

## Integration Points

### With Actors

Settings maintain actor presence via `characters` array:

```python
# Check which actors are in a setting
characters_present = setting_data.get('characters', [])

# Move actor to new setting
from generate.generate_actor import _add_actor_to_setting, _remove_actor_from_setting
_remove_actor_from_setting("Adventurer", "Tavern", workflow_data_dir)
_add_actor_to_setting("Adventurer", "Market Square", workflow_data_dir)
```

### With World Editor

Visual map editor stores spatial data in `world_map_data`:
- Position (x, y coordinates)
- Visual appearance (icon, color)
- Region boundaries (polygon points)
- Connection paths between settings

### With Rules/Timers

Setting variables are accessible with scope="Setting":

```python
# Rule condition checking setting variable
if setting_data['variables'].get('is_open', True):
    allow_entry()
```

### With Variables System

Settings integrate with the global variables system:
- `travel_notes_to_*` variables for navigation context
- Custom setting state (is_open, danger_level, etc.)
- Cross-referenced by player location tracking

## Validation Rules

1. **name**: Required, unique within workflow, max 64 characters
2. **world/region/location**: Must reference existing parent setting (or be omitted for top-level)
3. **characters**: Array of strings referencing actor names
4. **connections**: Keys must be valid setting names
5. **features**: Array of objects with feature_name and feature_description
6. **variables**: Any JSON-serializable key-value pairs
7. **inventory**: Array of strings (item names)

## Cross-References

- **Used By**: [[Game Engine]], [[Actor Manager]], [[World Editor]], [[Rule Evaluator]]
- **References**: [[Actor Schema]], [[Variables System]], [[World Map Editor]]
- **Validates**: Discord claims about three-tier structure (World/Region/Location)
- **Related Patterns**: [[Three-Tier Persistence Pattern]], [[JIT Generation]]

## Notes

- Settings use hierarchical directories matching World → Region → Location structure
- Filename must end with `_setting.json` (required for discovery)
- Session settings (`game/settings/`) override base settings with same name and path
- Connections are unidirectional (A→B doesn't imply B→A)
- `travel_notes_to_*` variables provide contextual travel descriptions for LLM
- World Editor provides visual map interface but stores data in standard JSON
- The `characters` array is the source of truth for actor locations
