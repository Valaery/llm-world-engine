---
tags: [schema, chatbotrpg, implementation, variables, state]
created: 2026-01-20
---

# Variables Schema (Global State)

**Primary File**: `src/core/utils.py` (line 13, BASE_VARIABLES_FILE constant)

**Storage Location**: `{workflow_dir}/game/variables.json`

**Purpose**: Stores global workflow-scoped variables accessible across all actors, settings, and rules.

## File Structure

```json
{
  "origin": "Starting Village",
  "current_chapter": 3,
  "world_time": "Day 15, 14:30",
  "global_reputation": 75,
  "main_quest_stage": "investigate_ruins",
  "alliance_with_merchants": true,
  "gates_open": false,
  "event_festival_active": true,
  "npc_count": 12,
  "player_level": 5,
  "world_state": "peacetime",
  "last_save_date": "2024-01-20T15:30:00Z"
}
```

## Variable Scopes

ChatBotRPG uses a four-tier variable scoping system:

### 1. Global Variables

**Storage**: `{workflow_dir}/game/variables.json`

**Access**: Available everywhere in the workflow

**Common Uses**:
- `origin`: Player's starting location
- `current_chapter`: Story progression
- `world_time`: Current game time
- `main_quest_stage`: Main quest progress
- Global flags (event_active, gates_open, etc.)

**Loading** (referenced in utils.py and main application):
```python
def _load_variables(self, tab_index):
    tab_data = self.tabs_data[tab_index]
    workflow_data_dir = tab_data.get('workflow_data_dir')
    variables_file = os.path.join(workflow_data_dir, 'game', BASE_VARIABLES_FILE)

    if os.path.exists(variables_file):
        with open(variables_file, 'r', encoding='utf-8') as f:
            return json.load(f)
    return {}
```

### 2. Character Variables

**Storage**: `{workflow_dir}/game/actors/{actor}.json` → `variables` field

**Access**: Scoped to specific actor

**Example**:
```json
{
  "name": "Adventurer",
  "variables": {
    "health": 85,
    "max_health": 100,
    "mana": 30,
    "level": 5,
    "experience": 1250,
    "gold": 250,
    "reputation_merchant_guild": 15,
    "quest_artifact_found": true,
    "met_wizard": false
  }
}
```

**Common Uses**:
- Character stats (health, mana, stamina)
- Character progression (level, experience, skills)
- Character state (buffs, debuffs, conditions)
- Character relationships (reputation with factions)
- Character quest flags

### 3. Player Variables

**Storage**: `{workflow_dir}/game/actors/{player_actor}.json` → `variables` field

**Access**: Same as Character Variables but specifically for the player actor (where `isPlayer=true`)

**Special Handling**: Player variables are commonly referenced in rules/timers with scope="Player"

**Example**:
```json
{
  "name": "Hero",
  "isPlayer": true,
  "variables": {
    "health": 100,
    "max_health": 100,
    "gold": 500,
    "main_quest_progress": 3,
    "has_special_key": true,
    "is_player": true
  }
}
```

### 4. Setting Variables

**Storage**: `{workflow_dir}/game/settings/**/*_setting.json` → `variables` field

**Access**: Scoped to specific setting

**Example**:
```json
{
  "name": "Tavern",
  "variables": {
    "tavern_reputation": 75,
    "ales_in_stock": 12,
    "rooms_available": 3,
    "is_open": true,
    "last_brawl_date": "2024-01-15",
    "travel_notes_to_market_square": "Short walk north",
    "travel_notes_to_harbor": "Quick eastward path",
    "discovered": true,
    "danger_level": 0
  }
}
```

**Common Uses**:
- Setting state (is_open, is_locked, is_discovered)
- Setting resources (stock levels, available services)
- Setting conditions (danger_level, light_level, temperature)
- Travel notes (auto-generated from connections)
- Discovery status

## Variable Types

All scopes support JSON-serializable types:

### Numeric
```json
{
  "health": 85,
  "max_health": 100,
  "gold": 250,
  "reputation": 15.5,
  "level": 5
}
```

### Boolean
```json
{
  "has_key": true,
  "quest_complete": false,
  "gates_open": true,
  "discovered": false
}
```

### String
```json
{
  "faction": "Merchant Guild",
  "title": "Adventurer",
  "quest_stage": "investigate_ruins",
  "last_visit_date": "2024-01-15",
  "current_weather": "rainy"
}
```

### Null
```json
{
  "last_combat": null,
  "active_buff": null
}
```

### Arrays and Objects (less common but supported)
```json
{
  "completed_quests": ["tutorial", "first_delivery", "bandit_camp"],
  "party_members": ["Warrior", "Mage", "Cleric"],
  "buffs": {
    "strength": 10,
    "intelligence": 5
  }
}
```

## Variable Resolution Order

When a rule or timer references a variable by name, the system searches scopes in this order:

1. **Global Variables** (`variables.json`)
2. **Character Variables** (if character scope specified)
3. **Player Variables** (if player scope specified)
4. **Setting Variables** (if setting scope specified)

**Example from timer_manager.py (lines 154-229)**:

```python
def _evaluate_variable_condition(self, condition, tab_data, character_name=None):
    var_name = condition.get('name', '')
    scope = condition.get('scope', 'Global')

    if scope == 'Character':
        # Load character variables
        actor_data, _ = _get_or_create_actor_data(self.parent(), workflow_dir, character_name)
        var_value = actor_data['variables'].get(var_name)

    elif scope == 'Player':
        # Load player variables
        player_name = _get_player_character_name(workflow_dir)
        actor_data, _ = _get_or_create_actor_data(self.parent(), workflow_dir, player_name)
        var_value = actor_data['variables'].get(var_name)

    elif scope == 'Setting':
        # Load setting variables
        setting_name = _get_player_current_setting_name(workflow_dir)
        setting_file = _find_setting_file_by_name(session_settings_dir, setting_name)
        setting_data = json.load(open(setting_file))
        var_value = setting_data['variables'].get(var_name)

    else:  # Global
        # Load global variables
        variables = main_ui._load_variables(tab_index)
        var_value = variables.get(var_name)
```

## Special Variables

### origin (Global)

**Type**: String

**Purpose**: Tracks the player's starting location

**Usage**: Used to reset player location or determine starting setting

**Set**: During game initialization or new game creation

**Example**:
```json
{
  "origin": "Starting Village"
}
```

### is_player (Character)

**Type**: Boolean

**Purpose**: Marks which actor is the player character

**Usage**: Only one actor can have `is_player=true`

**Example**:
```json
{
  "name": "Hero",
  "variables": {
    "is_player": true
  }
}
```

### travel_notes_to_* (Setting)

**Type**: String

**Purpose**: Auto-generated travel descriptions for connections

**Pattern**: `travel_notes_to_{sanitized_destination_name}`

**Generated**: When setting connections are created via LLM

**Example**:
```json
{
  "variables": {
    "travel_notes_to_market_square": "North through main door onto cobblestone plaza",
    "travel_notes_to_harbor": "East down narrow alley past crates and barrels"
  }
}
```

## Variable Operations

### Reading Variables

```python
# Global
from core.utils import _load_variables
global_vars = _load_variables(tab_index)
value = global_vars.get('origin', 'Unknown')

# Character
from core.utils import _get_or_create_actor_data
actor_data, _ = _get_or_create_actor_data(self, workflow_dir, "Adventurer")
health = actor_data.get('variables', {}).get('health', 100)

# Setting
from core.utils import _find_setting_file_by_name, _load_json_safely
setting_file = _find_setting_file_by_name(settings_dir, "Tavern")
setting_data = _load_json_safely(setting_file)
is_open = setting_data.get('variables', {}).get('is_open', True)
```

### Writing Variables

```python
# Global
variables = _load_variables(tab_index)
variables['current_chapter'] = 4
with open(variables_file, 'w', encoding='utf-8') as f:
    json.dump(variables, f, indent=2)

# Character
actor_data['variables']['health'] = 50
_save_json_safely(actor_file, actor_data)

# Setting
setting_data['variables']['is_open'] = False
_save_json_safely(setting_file, setting_data)
```

### Variable Conditions (Rules/Timers)

Variables can be tested with operators:

**Operators**:
- `==` : Equal
- `!=` : Not equal
- `>` : Greater than
- `<` : Less than
- `>=` : Greater than or equal
- `<=` : Less than or equal
- `contains` : String contains
- `not contains` : String does not contain
- `exists` : Variable exists (not null)
- `not exists` : Variable does not exist (is null)

**Example Condition**:
```python
condition = {
    "name": "health",
    "operator": "<",
    "value": 20,
    "scope": "Player"
}

# Evaluates: player_health < 20
```

**Numeric Comparison** (timer_manager.py:287-300):
```python
def _compare_numeric(self, a, b, operator):
    if operator == "==":
        return a == b
    elif operator == "!=":
        return a != b
    elif operator == ">":
        return a > b
    elif operator == "<":
        return a < b
    elif operator == ">=":
        return a >= b
    elif operator == "<=":
        return a <= b
    return False
```

## Variable Substitution

Variables can be substituted into strings using placeholders:

**Pattern**: `{variable_name}` or `[variable_name]`

**Example**:
```python
template = "You have {gold} gold coins."
# With gold=250: "You have 250 gold coins."

message = "Quest stage: [main_quest_stage]"
# With main_quest_stage="investigate_ruins": "Quest stage: investigate_ruins"
```

## Integration Points

### With Rules Engine

Rules can read/write variables and use them in conditions:

```python
# Condition: Check if player has enough gold
{
  "condition_type": "Variable",
  "variable_conditions": [
    {
      "name": "gold",
      "operator": ">=",
      "value": 100,
      "scope": "Player"
    }
  ]
}

# Action: Modify variable
{
  "action_type": "Modify Variable",
  "variable_name": "gold",
  "operation": "subtract",
  "value": 100,
  "scope": "Player"
}
```

### With Timer System

Timers can trigger based on variable conditions:

```python
# Timer fires when health < 20
{
  "rule_id": "low_health_warning",
  "interval": 60,
  "variable_conditions": [
    {
      "name": "health",
      "operator": "<",
      "value": 20,
      "scope": "Player"
    }
  ]
}
```

### With Post Visibility

Messages can be filtered based on variable conditions (utils.py:589-626):

```python
visibility_data = {
  "mode": "Visible Only To",
  "condition_type": "Variable",
  "variable_conditions": [
    {
      "name": "has_magic_key",
      "operator": "==",
      "value": true,
      "scope": "Player"
    }
  ]
}

# Message only visible if player has magic key
```

## Validation Rules

1. **Keys**: Any valid JSON string (but avoid special characters for consistency)
2. **Values**: Any JSON-serializable type (string, number, boolean, null, array, object)
3. **Scope**: Must be one of: Global, Character, Player, Setting
4. **Operators**: Must be valid operator string (see Variable Conditions)

## Save/Load Behavior

### On Save

1. Global variables saved to `game/variables.json`
2. Character/Player variables saved inline in actor JSON
3. Setting variables saved inline in setting JSON
4. Save file copies entire `game/` directory including all variable states

### On Load

1. Restore `game/variables.json` from save
2. Restore all actor JSON files (with variables)
3. Restore all setting JSON files (with variables)
4. All variable states restored atomically

## Cross-References

- **Used By**: [[Rule Evaluator]], [[Timer System]], [[Post Visibility]]
- **Stored In**: [[Actor Schema]], [[Setting Schema]], [[Save File Format]]
- **Validates**: Discord claims about variable scoping system
- **Related Patterns**: [[State Management]], [[Conditional Logic]]

## Notes

- Variables.json stores only global workflow-level state
- Character/Player/Setting variables are stored inline in their respective JSON files
- The four-scope system (Global/Character/Player/Setting) provides flexible state management
- `travel_notes_to_*` variables are auto-generated and provide contextual navigation for LLM
- Variable names are case-sensitive
- Variable substitution in strings allows dynamic message generation
- Operator evaluation supports both numeric and string comparisons with type coercion
