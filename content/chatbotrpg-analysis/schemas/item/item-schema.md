---
tags: [schema, chatbotrpg, implementation, item, inventory, equipment]
created: 2026-01-20
---

# Item Schema (Inventory & Equipment)

**Primary Files**:
- `src/editor_panel/inventory_manager.py` (UI and management, lines 1-1500+)
- `src/editor_panel/actor_manager.py` (Actor equipment, lines 292-500)

**Storage Location**:
- Resource Items (templates): `{workflow_dir}/resources/data files/items/{category}/*.json`
- Item Instances (runtime): Stored in Actor JSON files and Setting JSON files

**Two-Level System**: Resource Items (templates) + Item Instances (with state)

## Purpose

ChatBotRPG uses a two-tier item system:

1. **Resource Items**: Template definitions stored in categorized directories
2. **Item Instances**: Actual items with quantity, durability, owner, and location stored inline in Actor or Setting data

## Resource Item Schema

Resource items are template definitions organized by category:

### Complete Resource Item JSON

```json
{
  "name": "Healing Potion",
  "description": "A small glass vial filled with glowing red liquid that heals wounds when consumed.",

  "properties": ["Consumable"],

  "containers": ["Glass Vial"],
  "liquid_containers": ["Glass Vial"],

  "variables": {
    "heal_amount": 50,
    "weight": 0.5,
    "rarity": "common",
    "magical": true,
    "price": 25
  },

  "base_value": 25,
  "weight": 0.5,
  "tags": ["healing", "potion", "consumable"],

  "equipment_slot": null,
  "readable_text": null
}
```

### Field Documentation

| Field | Type | Required | Default | Description | UI Location |
|-------|------|----------|---------|-------------|-------------|
| **name** | string | Yes | - | Item name (unique within category) | inventory_manager.py:318 |
| **description** | string | No | "" | Item description and appearance | Row 7 in table |
| **properties** | array | No | ["Consumable"] | Item type flags | inventory_manager.py:343-423 |
| **containers** | array | No | [] | Container names this item has | inventory_manager.py:425-479 |
| **liquid_containers** | array | No | [] | Subset of containers that hold liquids | inventory_manager.py:467-469 |
| **variables** | object | No | {} | Custom item variables | Row 3 in table |
| **base_value** | number | No | 0 | Base economic value | Row 4 in table |
| **weight** | number | No | 0 | Item weight (for encumbrance) | Row 5 in table |
| **tags** | array | No | [] | Searchable tags | Row 6 in table |
| **equipment_slot** | string | No | null | If wearable, which slot (see Actor equipment slots) | inventory_manager.py:369-377 |
| **readable_text** | string | No | null | If readable, the text content | inventory_manager.py:383-388 |

### Properties System

**Available Properties** (inventory_manager.py:350-365):
- **Consumable**: Can be consumed/used up (default)
- **Wearable**: Can be equipped in a body slot
- **Readable**: Contains text that can be read
- **Liquid**: Is a liquid substance

**Property UI** (inventory_manager.py:343-423):
```python
def _create_properties_widget(self, properties="Consumable"):
    # Checkboxes for each property type
    consumable_checkbox = QCheckBox("Consumable")
    wearable_checkbox = QCheckBox("Wearable")
    readable_checkbox = QCheckBox("Readable")
    liquid_checkbox = QCheckBox("Liquid")

    # Equipment slot combo (visible if Wearable)
    equipment_slot_combo = QComboBox()
    equipment_slot_combo.addItems([
        "Head", "Neck", "Left Shoulder", "Right Shoulder",
        "Left Hand", "Right Hand", "Upper Outer", "Upper Middle",
        "Upper Inner", "Lower Outer", "Lower Middle", "Lower Inner",
        "Left Foot Inner", "Right Foot Inner",
        "Left Foot Outer", "Right Foot Outer"
    ])

    # Readable text input (visible if Readable)
    readable_text_input = QTextEdit()
    readable_text_input.setPlaceholderText("Enter the text content of this readable item...")
```

### Containers System

Items can have containers (bags, bottles, boxes):

```json
{
  "name": "Backpack",
  "containers": ["Main Compartment", "Side Pocket", "Front Pouch"],
  "liquid_containers": []
}
```

```json
{
  "name": "Waterskin",
  "containers": ["Leather Pouch"],
  "liquid_containers": ["Leather Pouch"]
}
```

**Container UI** (inventory_manager.py:425-500):
- Add/Remove buttons to manage containers
- Checkbox for "Holds Liquids" per container
- Nested item containment supported in Item Instances

### Equipment Slots

If property is "Wearable", the item can specify an equipment slot:

```json
{
  "name": "Iron Helmet",
  "properties": ["Wearable"],
  "equipment_slot": "Head"
}
```

**Valid Slots** (matching Actor equipment schema):
- Head, Neck
- Left Shoulder, Right Shoulder
- Left Hand, Right Hand
- Upper Outer, Upper Middle, Upper Inner
- Lower Outer, Lower Middle, Lower Inner
- Left Foot Inner, Right Foot Inner
- Left Foot Outer, Right Foot Outer

### Readable Items

If property is "Readable", the item contains text:

```json
{
  "name": "Ancient Scroll",
  "properties": ["Readable"],
  "readable_text": "The ancient prophecy speaks of a chosen one who will unite the fractured kingdoms..."
}
```

### Resource Item Variables

Custom variables for game mechanics:

```json
{
  "variables": {
    "heal_amount": 50,
    "duration_seconds": 300,
    "magical": true,
    "cursed": false,
    "required_level": 5,
    "damage": "2d6",
    "armor_class": 5,
    "enchantment": "Fire Resistance"
  }
}
```

## Item Instance Schema

Item instances represent actual items in the game world with state:

### Complete Item Instance JSON

```json
{
  "item_id": "a3f2e1d4",
  "resource_item": "Healing Potion",
  "resource_category": "Consumables",

  "quantity": 3,
  "durability": 100,

  "description": "A partially used healing potion, slightly warm to the touch",

  "owner": "player_id_123",
  "location": "setting_id_456",

  "custom_properties": {
    "blessed": true,
    "crafted_by": "Master Alchemist Elara",
    "creation_date": "2024-01-15"
  },

  "contains": [
    {
      "item_id": "b4g3f2e5",
      "resource_item": "Gold Coin",
      "resource_category": "Currency",
      "quantity": 50,
      "durability": 100,
      "owner": "player_id_123",
      "location": null,
      "custom_properties": {},
      "contains": []
    }
  ]
}
```

### Item Instance Fields

| Field | Type | Required | Description | Source |
|-------|------|----------|-------------|--------|
| **item_id** | string | Yes | Unique identifier (8-char hex) | inventory_manager.py:21-22 |
| **resource_item** | string | Yes | Name of resource item template | inventory_manager.py:132 |
| **resource_category** | string | Yes | Category directory name | inventory_manager.py:133 |
| **quantity** | number | Yes | Stack size (1+) | inventory_manager.py:134 |
| **durability** | number | Yes | Condition percentage (0-100) | inventory_manager.py:135 |
| **description** | string | No | Instance-specific description override | inventory_manager.py:136 |
| **owner** | string | No | Owner actor ID or name | inventory_manager.py:217-218 |
| **location** | string | No | Location setting ID or name | inventory_manager.py:218 |
| **custom_properties** | object | No | Instance-specific custom data | inventory_manager.py:137-138 |
| **contains** | array | No | Nested item instances (for containers) | inventory_manager.py:138-207 |

### Item ID Generation

```python
def generate_item_id():
    return f"{uuid.uuid4().hex[:8]}"
```

**Examples**:
- `a3f2e1d4`
- `b7c4d2f9`
- `e1f8a3b2`

### Nested Containment

Items can contain other items recursively:

```json
{
  "item_id": "backpack01",
  "resource_item": "Backpack",
  "resource_category": "Containers",
  "quantity": 1,
  "durability": 85,
  "contains": [
    {
      "item_id": "potion01",
      "resource_item": "Healing Potion",
      "resource_category": "Consumables",
      "quantity": 3,
      "durability": 100,
      "contains": []
    },
    {
      "item_id": "pouch01",
      "resource_item": "Coin Pouch",
      "resource_category": "Containers",
      "quantity": 1,
      "durability": 100,
      "contains": [
        {
          "item_id": "coin01",
          "resource_item": "Gold Coin",
          "resource_category": "Currency",
          "quantity": 50,
          "durability": 100,
          "contains": []
        }
      ]
    }
  ]
}
```

## Storage Locations

### Resource Items (Templates)

```
resources/data files/items/
├── Consumables/
│   ├── healing_potion.json
│   ├── mana_potion.json
│   └── antidote.json
├── Weapons/
│   ├── iron_sword.json
│   ├── wooden_bow.json
│   └── steel_dagger.json
├── Armor/
│   ├── leather_armor.json
│   ├── iron_helmet.json
│   └── steel_boots.json
├── Containers/
│   ├── backpack.json
│   ├── coin_pouch.json
│   └── waterskin.json
└── Misc/
    ├── torch.json
    ├── rope.json
    └── lockpick.json
```

### Item Instances

Item instances are **NOT** stored in separate files. They are stored inline in:

#### Actor Inventory

In Actor JSON (`game/actors/*.json`):

```json
{
  "name": "Adventurer",
  "inventory": [
    {
      "item_id": "a3f2e1d4",
      "resource_item": "Healing Potion",
      "resource_category": "Consumables",
      "quantity": 3,
      "durability": 100,
      "owner": "Adventurer",
      "location": null,
      "custom_properties": {},
      "contains": []
    }
  ]
}
```

#### Setting Inventory

In Setting JSON (`game/settings/**/*_setting.json`):

```json
{
  "name": "Tavern",
  "inventory": [
    "Ale barrel",
    "Wine bottles",
    "Roasted meat"
  ]
}
```

**Note**: Setting inventory is simpler - just an array of item names (strings), not full Item Instances.

## Item Instance UI

**Item Instance Dialog** (inventory_manager.py:24-139):

```python
class ItemInstanceDialog(QDialog):
    def __init__(self, parent=None, workflow_data_dir=None, existing_item_data=None):
        # UI for creating/editing item instances
        self.resource_category_combo = QComboBox()  # Select category
        self.resource_item_combo = QComboBox()      # Select item template
        self.quantity_spin = QSpinBox()             # 1-999
        self.durability_spin = QSpinBox()           # 0-100
        self.description_edit = QTextEdit()         # Custom description
        self.custom_properties_edit = QTextEdit()   # JSON for custom properties
```

**Item Tree Widget** (inventory_manager.py:169-211):
- Displays items in tree structure (supports nested containers)
- Columns: Item, Quantity, Durability, Description
- Double-click to edit item instance
- Shows containment hierarchy

## File Operations

### Loading Resource Items

```python
# Load all items from a category
resource_items_dir = os.path.join(workflow_data_dir, 'resources', 'data files', 'items')
category_path = os.path.join(resource_items_dir, 'Consumables')

for filename in os.listdir(category_path):
    if filename.lower().endswith('.json'):
        file_path = os.path.join(category_path, filename)
        with open(file_path, 'r', encoding='utf-8') as f:
            item_data = json.load(f)
            item_name = item_data.get('name')
```

### Creating Item Instance

```python
def generate_item_id():
    return f"{uuid.uuid4().hex[:8]}"

item_instance = {
    "item_id": generate_item_id(),
    "resource_item": "Healing Potion",
    "resource_category": "Consumables",
    "quantity": 1,
    "durability": 100,
    "description": "",
    "owner": actor_name,
    "location": None,
    "custom_properties": {},
    "contains": []
}
```

### Adding Item to Actor

```python
# Load actor
actor_data = _load_json_safely(actor_file)

# Ensure inventory array exists
if 'inventory' not in actor_data:
    actor_data['inventory'] = []

# Add item instance
actor_data['inventory'].append(item_instance)

# Save actor
_save_json_safely(actor_file, actor_data)
```

### Removing Item from Actor

```python
def _remove_item_from_list(target_item_id, items_list):
    """Recursively search and remove item by ID"""
    for i, item_data in enumerate(items_list):
        if item_data.get('item_id') == target_item_id:
            items_list.pop(i)
            return True

        # Check nested containers
        contains = item_data.get('contains', [])
        if _remove_item_from_list(target_item_id, contains):
            return True

    return False
```

## Item Categories

Resource items are organized into categories (directories):

**Common Categories**:
- **Consumables**: Potions, food, scrolls
- **Weapons**: Swords, bows, daggers, staffs
- **Armor**: Helmets, chestplates, boots, shields
- **Containers**: Bags, pouches, boxes, bottles
- **Currency**: Coins, gems, trade goods
- **Tools**: Lockpicks, torches, rope, climbing gear
- **Quest Items**: Unique items for quests
- **Misc**: Miscellaneous items

## Equipment vs. Inventory vs. Holding

ChatBotRPG distinguishes three ways characters interact with items:

### 1. Equipped (Worn)

Items in Actor's `equipment` object (17 body slots):

```json
{
  "equipment": {
    "head": "Iron Helmet",
    "upper_outer": "Leather Tunic",
    "lower_outer": "Canvas Trousers",
    "left_foot_outer": "Leather Boots",
    "right_foot_outer": "Leather Boots"
  }
}
```

**Source**: Simple string names referencing resource items.

### 2. Holding (In Hands)

Items in Actor's hand holding fields:

```json
{
  "left_hand_holding": "Torch",
  "right_hand_holding": "Iron Sword"
}
```

**Source**: Simple string names (NOT item instances).

### 3. Inventory (Carried)

Items in Actor's `inventory` array:

```json
{
  "inventory": [
    {
      "item_id": "a3f2e1d4",
      "resource_item": "Healing Potion",
      "quantity": 3,
      "contains": []
    }
  ]
}
```

**Source**: Full Item Instance objects with state.

## Integration Points

### With Actors

Actors have three item-related fields:
- `equipment`: Worn items (string names)
- `left_hand_holding` / `right_hand_holding`: Held items (string names)
- `inventory`: Carried items (Item Instances)

### With Settings

Settings have simple inventory:
- `inventory`: Array of item name strings (not Item Instances)

Used for environmental items that can be picked up.

### With Rules/Timers

Item variables can be checked in rules:

```python
# Check if actor has specific item
for item in actor_data.get('inventory', []):
    if item['resource_item'] == 'Magic Key':
        unlock_door()
```

## Validation Rules

### Resource Items

1. **name**: Required, unique within category
2. **properties**: Array of valid property strings
3. **containers**: Array of container name strings
4. **liquid_containers**: Subset of containers
5. **variables**: Any JSON-serializable key-value pairs
6. **equipment_slot**: If Wearable, must be valid slot name
7. **readable_text**: If Readable, must be string

### Item Instances

1. **item_id**: Required, unique 8-character hex string
2. **resource_item**: Required, must reference existing resource item name
3. **resource_category**: Required, must be valid category directory
4. **quantity**: Integer >= 1
5. **durability**: Integer 0-100
6. **owner**: Must reference existing actor name (or null)
7. **location**: Must reference existing setting name (or null)
8. **contains**: Array of valid Item Instance objects (recursive)

## State Transitions

- **quantity**: Decreases on consumption/use, increases on stack merge
- **durability**: Decreases on use/damage, restored by repair
- **owner**: Changes on trade/drop/pickup
- **location**: Changes when dropped/picked up
- **contains**: Modified when items added/removed from container

## Cross-References

- **Used By**: [[Actor Manager]], [[Setting Manager]], [[Rule Evaluator]]
- **References**: [[Actor Schema]], [[Setting Schema]]
- **Validates**: Discord claims about equipment and inventory systems
- **Related Patterns**: [[Template System]], [[Resource Management]]

## Notes

- Two-tier system: Resource Items (templates) vs Item Instances (with state)
- Item Instances are stored **inline** in Actor and Setting JSON files
- Resource Items organized by category directories for easy management
- Nested containment allows bags-within-bags-within-bags
- Equipment slots (worn) are separate from hand holding (wielded) and inventory (carried)
- Setting inventory is simplified (just string names, not full instances)
- Item IDs are 8-character hex strings generated from UUID
- Custom properties allow per-instance state without modifying resource template
