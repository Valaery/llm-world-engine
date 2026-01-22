# ChatBotRPG - Code Examples and Best Practices

**Analysis Date**: 2026-01-18
**Language**: Python 3.10+
**Type Hints**: Full type annotations

---

## Overview

This document contains production-ready code examples inferred from ChatBotRPG's design patterns and Discord discussions. All examples include:
- Full type hints
- Error handling
- Documentation
- Real-world usage examples

---

## Best Practice 1: Anti-Hallucination Constraint System

### Implementation

```python
from typing import Dict, Any, Tuple, List, Optional
from dataclasses import dataclass

@dataclass
class ValidationResult:
    """Result of action validation"""
    valid: bool
    reason: str
    suggested_narration: Optional[str] = None


class ConstraintValidator:
    """Validates LLM outputs against game rules to prevent hallucinations"""

    MAGIC_KEYWORDS = ["cast", "spell", "magic", "summon", "teleport",
                      "enchant", "charm", "hex", "curse_spell"]
    MAGIC_CLASSES = ["mage", "wizard", "sorcerer", "warlock", "cleric",
                     "druid", "paladin"]

    def __init__(self, character_sheet: Dict[str, Any],
                 world_state: Dict[str, Any]):
        self.character_sheet = character_sheet
        self.world_state = world_state

    def validate_action(self, action: str, method: str) -> ValidationResult:
        """
        Validate if action is possible given character abilities

        Args:
            action: The action being attempted (e.g., "attack", "cast spell")
            method: How the action is being performed (e.g., "sword", "fireball")

        Returns:
            ValidationResult with validity status and reason
        """

        # Check 1: Character has required ability
        if self._is_magical_action(action, method):
            if not self._has_magic_ability():
                return ValidationResult(
                    valid=False,
                    reason="You're not a spellcaster",
                    suggested_narration="You attempt to channel magical energy, but nothing happens. You lack the training to cast spells."
                )

        # Check 2: Required items present
        required_item = self._get_required_item(action, method)
        if required_item and not self._has_item(required_item):
            return ValidationResult(
                valid=False,
                reason=f"You don't have {required_item}",
                suggested_narration=f"You reach for your {required_item}, but it's not there. You can't perform this action without it."
            )

        # Check 3: Physics validation
        physics_check = self._physics_check(action, method)
        if not physics_check.valid:
            return physics_check

        # Check 4: Location constraints
        location_check = self._location_check(action)
        if not location_check.valid:
            return location_check

        return ValidationResult(valid=True, reason="Action allowed")

    def _is_magical_action(self, action: str, method: str) -> bool:
        """Check if action or method involves magic"""
        text = f"{action} {method}".lower()
        return any(keyword in text for keyword in self.MAGIC_KEYWORDS)

    def _has_magic_ability(self) -> bool:
        """Check if character can cast spells"""
        char_class = self.character_sheet.get("class", "").lower()
        return char_class in self.MAGIC_CLASSES

    def _has_item(self, item: str) -> bool:
        """Check if item is in character's inventory"""
        inventory = self.character_sheet.get("inventory", [])
        item_lower = item.lower()
        return any(item_lower in inv_item.lower() for inv_item in inventory)

    def _get_required_item(self, action: str, method: str) -> Optional[str]:
        """Map actions/methods to required items"""
        action_items = {
            "lockpick": "lockpicks",
            "unlock": "key",
            "shoot": "bow",
            "fire arrow": "bow",
            "drink potion": "potion",
            "read": "book",
            "light": "torch",
            "write": "quill"
        }

        text = f"{action} {method}".lower()
        for key, item in action_items.items():
            if key in text:
                return item
        return None

    def _physics_check(self, action: str, method: str) -> ValidationResult:
        """Validate against setting physics"""

        text = f"{action} {method}".lower()

        # Check for impossible physical actions
        impossible_actions = [
            ("fly", "flight", "You can't fly without wings or magic"),
            ("teleport", "teleportation", "You can't teleport without magic"),
            ("lift 500", None, "That's far too heavy to lift"),
            ("jump through wall", None, "You can't jump through solid walls"),
            ("walk through", None, "You can't walk through solid objects"),
            ("phase", None, "You can't pass through matter"),
            ("become invisible", "invisibility", "You don't have invisibility powers")
        ]

        for trigger, ability, reason in impossible_actions:
            if trigger in text:
                # Check if character has the required ability
                if ability and self._has_ability(ability):
                    continue  # Action is valid due to special ability

                return ValidationResult(
                    valid=False,
                    reason=reason,
                    suggested_narration=f"You attempt to {action}, but it's physically impossible. {reason}."
                )

        return ValidationResult(valid=True, reason="Physics check passed")

    def _location_check(self, action: str) -> ValidationResult:
        """Validate action is appropriate for current location"""

        location = self.world_state.get("player_location", "")
        action_lower = action.lower()

        # Location-specific restrictions
        restrictions = {
            "underwater": ["light fire", "shoot bow"],
            "in_air": ["pick up", "open door"],
            "crowded_tavern": ["practice swordplay", "shoot bow"]
        }

        for location_type, forbidden_actions in restrictions.items():
            if location_type in location.lower():
                for forbidden in forbidden_actions:
                    if forbidden in action_lower:
                        return ValidationResult(
                            valid=False,
                            reason=f"Can't {action} while {location_type}",
                            suggested_narration=f"You can't do that here. The environment doesn't allow it."
                        )

        return ValidationResult(valid=True, reason="Location check passed")

    def _has_ability(self, ability: str) -> bool:
        """Check if character has a special ability"""
        abilities = self.character_sheet.get("abilities", [])
        return ability.lower() in [a.lower() for a in abilities]


# Usage Example
def example_usage():
    """Demonstrate constraint validation"""

    # Character setup
    character = {
        "class": "Warrior",
        "inventory": ["iron_sword", "leather_armor", "10_gold"],
        "abilities": ["basic_swordplay", "shield_bash"]
    }

    world = {
        "player_location": "golden_oak_inn",
        "npcs": ["bartender", "guard"],
        "time": "evening"
    }

    validator = ConstraintValidator(character, world)

    # Test 1: Impossible action (casting spells as warrior)
    result = validator.validate_action("cast fireball", "arcane gesture")
    print(f"Test 1 - Magic as warrior:")
    print(f"  Valid: {result.valid}")
    print(f"  Reason: {result.reason}")
    print(f"  Narration: {result.suggested_narration}\n")
    # Output:
    # Valid: False
    # Reason: You're not a spellcaster
    # Narration: You attempt to channel magical energy...

    # Test 2: Missing required item
    result = validator.validate_action("lockpick the door", "carefully")
    print(f"Test 2 - Missing lockpicks:")
    print(f"  Valid: {result.valid}")
    print(f"  Reason: {result.reason}\n")
    # Output:
    # Valid: False
    # Reason: You don't have lockpicks

    # Test 3: Valid action
    result = validator.validate_action("attack guard", "with sword")
    print(f"Test 3 - Valid attack:")
    print(f"  Valid: {result.valid}")
    print(f"  Reason: {result.reason}\n")
    # Output:
    # Valid: True
    # Reason: Action allowed

    # Test 4: Physics violation
    result = validator.validate_action("fly to the ceiling", "flapping arms")
    print(f"Test 4 - Physics violation:")
    print(f"  Valid: {result.valid}")
    print(f"  Reason: {result.reason}")
    print(f"  Narration: {result.suggested_narration}\n")
    # Output:
    # Valid: False
    # Reason: You can't fly without wings or magic


if __name__ == "__main__":
    example_usage()
```

### Integration with LLM Pipeline

```python
def process_player_action_with_validation(
    player_input: str,
    character_sheet: Dict[str, Any],
    world_state: Dict[str, Any],
    llm_client: Any
) -> str:
    """
    Process player action with constraint validation before narration

    Args:
        player_input: Natural language player input
        character_sheet: Player's character data
        world_state: Current world state
        llm_client: LLM client for narration

    Returns:
        Narrated response
    """

    # Step 1: Extract intent from natural language
    intent = extract_intent(player_input, llm_client)

    # Step 2: Validate action against constraints
    validator = ConstraintValidator(character_sheet, world_state)
    validation = validator.validate_action(intent['action'], intent['method'])

    # Step 3: Generate narration
    if validation.valid:
        # Action allowed - narrate success/failure based on dice rolls, etc.
        event = execute_action(intent, world_state)
        return generate_narration(event, world_state, llm_client)
    else:
        # Action invalid - use suggested narration or generate failure
        if validation.suggested_narration:
            return validation.suggested_narration
        else:
            return generate_failure_narration(intent, validation.reason, llm_client)
```

---

## Best Practice 2: 170-Token Narration Limiter

### Implementation

```python
import re
from typing import Optional, Dict, Any
from dataclasses import dataclass


@dataclass
class NarrationConfig:
    """Configuration for narration generation"""
    max_tokens: int = 170
    target_sentences: int = 2
    temperature: float = 0.7
    enforce_complete_sentences: bool = True


class NarrationLimiter:
    """Enforces 170-token sweet spot for responsive narration"""

    def __init__(self, llm_client: Any, config: Optional[NarrationConfig] = None):
        self.llm = llm_client
        self.config = config or NarrationConfig()

    def generate_narration(self, ndl: str, context: Dict[str, Any],
                           override_max_tokens: Optional[int] = None) -> str:
        """
        Generate token-limited narration

        Args:
            ndl: Natural Description Language event string
            context: Game context (location, time, NPCs, etc.)
            override_max_tokens: Optional override for max tokens

        Returns:
            Formatted narration string
        """
        max_tokens = override_max_tokens or self.config.max_tokens

        # Build prompt with explicit length constraint
        prompt = self._build_prompt(ndl, context, max_tokens)

        # API-level token limit
        response = self.llm.complete(
            prompt,
            temperature=self.config.temperature,
            max_tokens=max_tokens
        )

        # Post-processing: Enforce sentence count and completeness
        if self.config.enforce_complete_sentences:
            cleaned = self._enforce_sentence_limit(response)
        else:
            cleaned = response.strip()

        return cleaned

    def _build_prompt(self, ndl: str, context: Dict[str, Any],
                      max_tokens: int) -> str:
        """Build prompt with length instructions"""

        location = context.get('location', 'Unknown location')
        time = context.get('time', 'Unknown time')
        weather = context.get('weather', '')

        weather_note = f"\nWeather: {weather}" if weather else ""

        return f"""You are a game narrator. Convert events to natural prose.

Setting: {location}
Time: {time}{weather_note}

Events: {ndl}

Write EXACTLY {self.config.target_sentences}-3 sentences. Be concise and punchy.
Maximum length: {max_tokens} tokens.
Use present tense and second person ("You swing...").
"""

    def _enforce_sentence_limit(self, text: str) -> str:
        """
        Post-process to enforce sentence count

        Args:
            text: Raw LLM output

        Returns:
            Cleaned text with proper sentence boundaries
        """
        # Split on sentence boundaries (. ! ?)
        sentence_pattern = r'(?<=[.!?])\s+'
        sentences = re.split(sentence_pattern, text.strip())

        # Keep first 2-3 sentences
        max_sentences = self.config.target_sentences + 1  # Allow up to 3
        if len(sentences) > max_sentences:
            sentences = sentences[:max_sentences]

        # Handle incomplete final sentence (common with token limits)
        # If last sentence doesn't end with punctuation, remove it
        if sentences and sentences[-1]:
            last_char = sentences[-1].strip()[-1] if sentences[-1].strip() else ''
            if last_char not in '.!?':
                sentences = sentences[:-1]

        # Join sentences
        result = ' '.join(s.strip() for s in sentences if s.strip())

        return result

    def generate_dialogue(self, speaker: str, listener: str,
                          topic: str, context: Dict[str, Any],
                          max_tokens: int = 50) -> str:
        """
        Generate short dialogue (even more constrained than narration)

        Args:
            speaker: Character speaking
            listener: Character being spoken to
            topic: What the dialogue is about
            context: Game context including NPC personality
            max_tokens: Token limit for dialogue (default 50)

        Returns:
            Dialogue string
        """
        personality = context.get('npcs', {}).get(speaker, {}).get('personality', 'neutral')
        mood = context.get('npcs', {}).get(speaker, {}).get('mood', 'neutral')

        prompt = f"""Character {speaker} speaks to {listener} about {topic}.

Personality: {personality}
Mood: {mood}

Write a single line of dialogue (1-2 sentences max). Match personality and mood.
Include quotation marks. No narration, just dialogue.
"""

        response = self.llm.complete(
            prompt,
            temperature=0.9,  # Higher for creative dialogue
            max_tokens=max_tokens
        )

        # Extract dialogue from quotes if present
        dialogue_match = re.search(r'"([^"]+)"', response)
        if dialogue_match:
            return f'"{dialogue_match.group(1)}"'

        return response.strip()

    def generate_combat_narration(self, attacker: str, defender: str,
                                   attack_type: str, hit: bool,
                                   damage: int, context: Dict[str, Any]) -> str:
        """
        Generate combat narration (optimized for fast-paced action)

        Args:
            attacker: Who is attacking
            defender: Who is being attacked
            attack_type: Type of attack (sword, spell, etc.)
            hit: Whether attack hit or missed
            damage: Damage dealt (0 if miss)
            context: Game context

        Returns:
            Combat narration string
        """
        result = "hit" if hit else "miss"
        damage_str = f" ({damage} damage)" if hit else " (Miss)"

        prompt = f"""Combat turn in fantasy RPG:

Attacker: {attacker}
Defender: {defender}
Attack: {attack_type}
Result: {result}

Write 2 sentences describing this combat turn.
Be dynamic and engaging. Include sensory details (sounds, visuals).
End with: {damage_str}
"""

        response = self.llm.complete(
            prompt,
            temperature=0.6,  # Moderate creativity
            max_tokens=120    # Slightly shorter than normal narration
        )

        # Ensure damage/miss indicator is present
        if damage_str not in response:
            response = response.strip() + f" {damage_str}"

        return self._enforce_sentence_limit(response)


# Usage Example
def example_usage():
    """Demonstrate narration limiting"""

    class MockLLM:
        """Mock LLM for testing"""
        def complete(self, prompt: str, temperature: float, max_tokens: int) -> str:
            # Simulate different response lengths
            if "combat" in prompt.lower():
                return ("You swing your sword at the guard with all your might. "
                        "The blade connects with a sharp clang, biting deep into his shoulder. "
                        "He staggers back, blood seeping through his armor.")
            else:
                return ("You enter the Golden Oak Inn. The warm firelight flickers across "
                        "worn wooden tables. The smell of roasted meat and ale fills the air. "
                        "A bard strums a lute in the corner. Patrons laugh and talk loudly. "
                        "The bartender waves you over.")

    limiter = NarrationLimiter(MockLLM())

    # Example 1: Regular narration
    ndl = 'do($player, "enter")->location($inn)'
    context = {
        "location": "Golden Oak Inn",
        "time": "Evening",
        "weather": "Rainy outside"
    }

    narration = limiter.generate_narration(ndl, context)
    print("Example 1 - Regular narration:")
    print(f"  {narration}\n")
    # Output: First 2-3 sentences only

    # Example 2: Dialogue
    dialogue = limiter.generate_dialogue(
        "Bartender",
        "Player",
        "drinks",
        context={
            "npcs": {
                "Bartender": {
                    "personality": "gruff but fair",
                    "mood": "tired"
                }
            }
        }
    )
    print("Example 2 - Dialogue:")
    print(f"  {dialogue}\n")

    # Example 3: Combat
    combat = limiter.generate_combat_narration(
        "You",
        "Guard",
        "sword slash",
        hit=True,
        damage=12,
        context={}
    )
    print("Example 3 - Combat:")
    print(f"  {combat}\n")


if __name__ == "__main__":
    example_usage()
```

### Cost Comparison

```python
def calculate_cost_savings():
    """Demonstrate cost savings from 170-token limit"""

    # Pricing (Gemini 2.5 Flash Lite Preview, Jan 2026)
    cost_per_1k_output_tokens = 0.001  # $0.001 per 1K tokens

    # Scenarios
    scenarios = {
        "No limit (avg 500 tokens)": 500,
        "300-token limit": 300,
        "170-token limit": 170
    }

    turns_per_session = 200  # Average game session

    print("Cost Analysis for 200-turn game session:\n")
    for scenario, tokens_per_turn in scenarios.items():
        total_tokens = tokens_per_turn * turns_per_session
        cost = (total_tokens / 1000) * cost_per_1k_output_tokens
        print(f"{scenario}:")
        print(f"  Tokens per turn: {tokens_per_turn}")
        print(f"  Total tokens: {total_tokens:,}")
        print(f"  Cost: ${cost:.4f}\n")

    # Savings
    no_limit_cost = (500 * turns_per_session / 1000) * cost_per_1k_output_tokens
    with_limit_cost = (170 * turns_per_session / 1000) * cost_per_1k_output_tokens
    savings = no_limit_cost - with_limit_cost
    savings_pct = (savings / no_limit_cost) * 100

    print(f"Savings with 170-token limit:")
    print(f"  Absolute: ${savings:.4f}")
    print(f"  Percentage: {savings_pct:.1f}%")


# Output:
# Cost Analysis for 200-turn game session:
#
# No limit (avg 500 tokens):
#   Tokens per turn: 500
#   Total tokens: 100,000
#   Cost: $0.1000
#
# 300-token limit:
#   Tokens per turn: 300
#   Total tokens: 60,000
#   Cost: $0.0600
#
# 170-token limit:
#   Tokens per turn: 170
#   Total tokens: 34,000
#   Cost: $0.0340
#
# Savings with 170-token limit:
#   Absolute: $0.0660
#   Percentage: 66.0%
```

---

## Best Practice 3: Three-Tier Persistence

### Implementation

```python
import json
import sqlite3
from dataclasses import dataclass, asdict, field
from typing import Dict, List, Any, Optional
from datetime import datetime
from pathlib import Path


@dataclass
class WorldState:
    """Persistent world data (.world file)"""
    world_id: str
    name: str
    setting: str  # "fantasy", "sci-fi", etc.
    locations: Dict[str, Any] = field(default_factory=dict)
    npcs: Dict[str, Any] = field(default_factory=dict)
    items: Dict[str, Any] = field(default_factory=dict)
    rules: List[Dict[str, Any]] = field(default_factory=list)
    templates: Dict[str, Any] = field(default_factory=dict)
    lore: List[str] = field(default_factory=list)
    created_at: datetime = field(default_factory=datetime.now)
    version: str = "1.0"


@dataclass
class PlaythroughState:
    """Current game session (.save file)"""
    save_id: str
    world_id: str
    player_state: Dict[str, Any] = field(default_factory=dict)
    location_states: Dict[str, Any] = field(default_factory=dict)
    npc_states: Dict[str, Any] = field(default_factory=dict)
    quest_progress: Dict[str, Any] = field(default_factory=dict)
    inventory: List[str] = field(default_factory=list)
    game_time: int = 0  # In-game time (minutes since start)
    flags: Dict[str, bool] = field(default_factory=dict)
    turn_count: int = 0
    created_at: datetime = field(default_factory=datetime.now)
    last_played: datetime = field(default_factory=datetime.now)


class SessionState:
    """Runtime memory (not persisted)"""

    def __init__(self, max_context_window: int = 10):
        self.context_window: List[Dict[str, str]] = []
        self.max_context_window = max_context_window
        self.pending_events: List[str] = []
        self.cached_descriptions: Dict[str, str] = {}
        self.llm_temperature: float = 0.7
        self.current_scene: Optional[str] = None

    def add_to_context(self, event: Dict[str, str]):
        """Add event to context window with automatic cleanup"""
        self.context_window.append(event)

        # Keep only last N turns
        if len(self.context_window) > self.max_context_window:
            self.context_window.pop(0)

    def get_context(self) -> List[Dict[str, str]]:
        """Get current context window"""
        return self.context_window.copy()

    def clear_context(self):
        """Clear context window (e.g., when changing scenes)"""
        self.context_window.clear()


class PersistenceManager:
    """Manages three-tier persistence system"""

    def __init__(self, db_directory: str = "./saves"):
        self.db_directory = Path(db_directory)
        self.db_directory.mkdir(exist_ok=True)
        self.session = SessionState()
        self._ensure_schema()

    def _get_db_path(self, filename: str) -> Path:
        """Get full path for database file"""
        return self.db_directory / filename

    def _ensure_schema(self):
        """Ensure database schema exists for world/save files"""
        # Schema is created per-file, not globally
        pass

    def _init_world_db(self, db_path: Path):
        """Initialize world database schema"""
        conn = sqlite3.connect(str(db_path))
        cursor = conn.cursor()

        cursor.execute("""
            CREATE TABLE IF NOT EXISTS world_meta (
                world_id TEXT PRIMARY KEY,
                name TEXT NOT NULL,
                setting TEXT NOT NULL,
                version TEXT NOT NULL,
                created_at TIMESTAMP NOT NULL
            )
        """)

        cursor.execute("""
            CREATE TABLE IF NOT EXISTS locations (
                location_id TEXT PRIMARY KEY,
                data TEXT NOT NULL
            )
        """)

        cursor.execute("""
            CREATE TABLE IF NOT EXISTS npcs (
                npc_id TEXT PRIMARY KEY,
                data TEXT NOT NULL
            )
        """)

        cursor.execute("""
            CREATE TABLE IF NOT EXISTS items (
                item_id TEXT PRIMARY KEY,
                data TEXT NOT NULL
            )
        """)

        cursor.execute("""
            CREATE TABLE IF NOT EXISTS rules (
                rule_id TEXT PRIMARY KEY,
                data TEXT NOT NULL
            )
        """)

        cursor.execute("""
            CREATE TABLE IF NOT EXISTS lore (
                lore_id INTEGER PRIMARY KEY AUTOINCREMENT,
                content TEXT NOT NULL
            )
        """)

        conn.commit()
        conn.close()

    def _init_save_db(self, db_path: Path):
        """Initialize save database schema"""
        conn = sqlite3.connect(str(db_path))
        cursor = conn.cursor()

        cursor.execute("""
            CREATE TABLE IF NOT EXISTS save_meta (
                save_id TEXT PRIMARY KEY,
                world_id TEXT NOT NULL,
                created_at TIMESTAMP NOT NULL,
                last_played TIMESTAMP NOT NULL,
                turn_count INTEGER NOT NULL,
                game_time INTEGER NOT NULL
            )
        """)

        cursor.execute("""
            CREATE TABLE IF NOT EXISTS player_state (
                key TEXT PRIMARY KEY,
                value TEXT NOT NULL
            )
        """)

        cursor.execute("""
            CREATE TABLE IF NOT EXISTS location_states (
                location_id TEXT PRIMARY KEY,
                state TEXT NOT NULL
            )
        """)

        cursor.execute("""
            CREATE TABLE IF NOT EXISTS npc_states (
                npc_id TEXT PRIMARY KEY,
                state TEXT NOT NULL
            )
        """)

        cursor.execute("""
            CREATE TABLE IF NOT EXISTS flags (
                flag_name TEXT PRIMARY KEY,
                value INTEGER NOT NULL
            )
        """)

        conn.commit()
        conn.close()

    def save_world(self, world: WorldState) -> Path:
        """
        Persist world state to .world file

        Args:
            world: WorldState to persist

        Returns:
            Path to created .world file
        """
        db_path = self._get_db_path(f"{world.world_id}.world")
        self._init_world_db(db_path)

        conn = sqlite3.connect(str(db_path))
        cursor = conn.cursor()

        # Save metadata
        cursor.execute("""
            INSERT OR REPLACE INTO world_meta
            (world_id, name, setting, version, created_at)
            VALUES (?, ?, ?, ?, ?)
        """, (world.world_id, world.name, world.setting,
              world.version, world.created_at))

        # Save locations
        for loc_id, loc_data in world.locations.items():
            cursor.execute("""
                INSERT OR REPLACE INTO locations (location_id, data)
                VALUES (?, ?)
            """, (loc_id, json.dumps(loc_data)))

        # Save NPCs
        for npc_id, npc_data in world.npcs.items():
            cursor.execute("""
                INSERT OR REPLACE INTO npcs (npc_id, data)
                VALUES (?, ?)
            """, (npc_id, json.dumps(npc_data)))

        # Save items
        for item_id, item_data in world.items.items():
            cursor.execute("""
                INSERT OR REPLACE INTO items (item_id, data)
                VALUES (?, ?)
            """, (item_id, json.dumps(item_data)))

        # Save rules
        for rule in world.rules:
            cursor.execute("""
                INSERT OR REPLACE INTO rules (rule_id, data)
                VALUES (?, ?)
            """, (rule['rule_id'], json.dumps(rule)))

        # Save lore
        cursor.execute("DELETE FROM lore")  # Clear existing
        for lore_entry in world.lore:
            cursor.execute("""
                INSERT INTO lore (content) VALUES (?)
            """, (lore_entry,))

        conn.commit()
        conn.close()

        print(f"World saved to: {db_path}")
        return db_path

    def load_world(self, world_id: str) -> WorldState:
        """
        Load world state from .world file

        Args:
            world_id: ID of world to load

        Returns:
            WorldState object

        Raises:
            FileNotFoundError: If world file doesn't exist
        """
        db_path = self._get_db_path(f"{world_id}.world")
        if not db_path.exists():
            raise FileNotFoundError(f"World file not found: {db_path}")

        conn = sqlite3.connect(str(db_path))
        cursor = conn.cursor()

        # Load metadata
        cursor.execute("SELECT * FROM world_meta WHERE world_id = ?", (world_id,))
        meta = cursor.fetchone()
        if not meta:
            raise ValueError(f"World {world_id} not found in database")

        # Load locations
        locations = {}
        cursor.execute("SELECT location_id, data FROM locations")
        for loc_id, data in cursor.fetchall():
            locations[loc_id] = json.loads(data)

        # Load NPCs
        npcs = {}
        cursor.execute("SELECT npc_id, data FROM npcs")
        for npc_id, data in cursor.fetchall():
            npcs[npc_id] = json.loads(data)

        # Load items
        items = {}
        cursor.execute("SELECT item_id, data FROM items")
        for item_id, data in cursor.fetchall():
            items[item_id] = json.loads(data)

        # Load rules
        rules = []
        cursor.execute("SELECT data FROM rules")
        for (data,) in cursor.fetchall():
            rules.append(json.loads(data))

        # Load lore
        lore = []
        cursor.execute("SELECT content FROM lore")
        for (content,) in cursor.fetchall():
            lore.append(content)

        conn.close()

        return WorldState(
            world_id=meta[0],
            name=meta[1],
            setting=meta[2],
            version=meta[3],
            created_at=datetime.fromisoformat(meta[4]) if isinstance(meta[4], str) else meta[4],
            locations=locations,
            npcs=npcs,
            items=items,
            rules=rules,
            lore=lore
        )

    def save_playthrough(self, playthrough: PlaythroughState) -> Path:
        """
        Persist playthrough state to .save file

        Args:
            playthrough: PlaythroughState to persist

        Returns:
            Path to created .save file
        """
        playthrough.last_played = datetime.now()

        db_path = self._get_db_path(f"{playthrough.save_id}.save")
        self._init_save_db(db_path)

        conn = sqlite3.connect(str(db_path))
        cursor = conn.cursor()

        # Save metadata
        cursor.execute("""
            INSERT OR REPLACE INTO save_meta
            (save_id, world_id, created_at, last_played, turn_count, game_time)
            VALUES (?, ?, ?, ?, ?, ?)
        """, (playthrough.save_id, playthrough.world_id,
              playthrough.created_at, playthrough.last_played,
              playthrough.turn_count, playthrough.game_time))

        # Save player state
        cursor.execute("DELETE FROM player_state")
        for key, value in playthrough.player_state.items():
            cursor.execute("""
                INSERT INTO player_state (key, value) VALUES (?, ?)
            """, (key, json.dumps(value)))

        # Save location states
        for loc_id, state in playthrough.location_states.items():
            cursor.execute("""
                INSERT OR REPLACE INTO location_states (location_id, state)
                VALUES (?, ?)
            """, (loc_id, json.dumps(state)))

        # Save NPC states
        for npc_id, state in playthrough.npc_states.items():
            cursor.execute("""
                INSERT OR REPLACE INTO npc_states (npc_id, state)
                VALUES (?, ?)
            """, (npc_id, json.dumps(state)))

        # Save flags
        for flag_name, value in playthrough.flags.items():
            cursor.execute("""
                INSERT OR REPLACE INTO flags (flag_name, value)
                VALUES (?, ?)
            """, (flag_name, 1 if value else 0))

        conn.commit()
        conn.close()

        print(f"Save file created: {db_path}")
        return db_path

    def load_playthrough(self, save_id: str) -> PlaythroughState:
        """
        Load playthrough state from .save file

        Args:
            save_id: ID of save to load

        Returns:
            PlaythroughState object

        Raises:
            FileNotFoundError: If save file doesn't exist
        """
        db_path = self._get_db_path(f"{save_id}.save")
        if not db_path.exists():
            raise FileNotFoundError(f"Save file not found: {db_path}")

        conn = sqlite3.connect(str(db_path))
        cursor = conn.cursor()

        # Load metadata
        cursor.execute("SELECT * FROM save_meta WHERE save_id = ?", (save_id,))
        meta = cursor.fetchone()
        if not meta:
            raise ValueError(f"Save {save_id} not found in database")

        # Load player state
        player_state = {}
        cursor.execute("SELECT key, value FROM player_state")
        for key, value in cursor.fetchall():
            player_state[key] = json.loads(value)

        # Load location states
        location_states = {}
        cursor.execute("SELECT location_id, state FROM location_states")
        for loc_id, state in cursor.fetchall():
            location_states[loc_id] = json.loads(state)

        # Load NPC states
        npc_states = {}
        cursor.execute("SELECT npc_id, state FROM npc_states")
        for npc_id, state in cursor.fetchall():
            npc_states[npc_id] = json.loads(state)

        # Load flags
        flags = {}
        cursor.execute("SELECT flag_name, value FROM flags")
        for flag_name, value in cursor.fetchall():
            flags[flag_name] = bool(value)

        conn.close()

        return PlaythroughState(
            save_id=meta[0],
            world_id=meta[1],
            created_at=datetime.fromisoformat(meta[2]) if isinstance(meta[2], str) else meta[2],
            last_played=datetime.fromisoformat(meta[3]) if isinstance(meta[3], str) else meta[3],
            turn_count=meta[4],
            game_time=meta[5],
            player_state=player_state,
            location_states=location_states,
            npc_states=npc_states,
            flags=flags
        )

    def update_session(self, event: Dict[str, str]):
        """
        Update runtime session state (not persisted)

        Args:
            event: Event to add to context window
        """
        self.session.add_to_context(event)

    def get_session_context(self) -> List[Dict[str, str]]:
        """Get current session context window"""
        return self.session.get_context()

    def export_world_for_distribution(self, world_id: str, output_dir: str = "./exports"):
        """
        Export .world file for distribution (copy to output directory)

        Args:
            world_id: ID of world to export
            output_dir: Directory to export to
        """
        source = self._get_db_path(f"{world_id}.world")
        if not source.exists():
            raise FileNotFoundError(f"World file not found: {source}")

        output_path = Path(output_dir)
        output_path.mkdir(exist_ok=True)

        dest = output_path / f"{world_id}.world"
        import shutil
        shutil.copy2(source, dest)

        print(f"World exported to: {dest}")
        return dest


# Usage Example
def example_usage():
    """Demonstrate three-tier persistence"""

    pm = PersistenceManager()

    # Tier 1: Create world state (.world file)
    world = WorldState(
        world_id="fantasy_001",
        name="The Forgotten Realm",
        setting="high_fantasy",
        locations={
            "golden_oak_inn": {
                "name": "Golden Oak Inn",
                "description": "A cozy tavern with warm firelight",
                "connections": {
                    "market": {"travel_time": 10},
                    "castle": {"travel_time": 30}
                },
                "npcs": ["bartender"]
            }
        },
        npcs={
            "bartender": {
                "name": "Old Tom",
                "personality": "gruff but fair",
                "schedule": {
                    "8:00-22:00": "golden_oak_inn",
                    "22:00-8:00": "upstairs_room"
                }
            }
        },
        items={
            "health_potion": {
                "name": "Health Potion",
                "effect": "heal",
                "value": 20
            }
        },
        rules=[
            {
                "rule_id": "midnight_ghost",
                "name": "Midnight Haunting",
                "conditions": [{"type": "time", "value": "00:00"}],
                "actions": [{"type": "spawn_npc", "npc": "ghost"}]
            }
        ],
        lore=[
            "Long ago, dragons ruled these lands.",
            "The ancient ruins hold forgotten secrets."
        ]
    )

    world_path = pm.save_world(world)
    print(f"World saved: {world_path}\n")

    # Tier 2: Create playthrough state (.save file)
    playthrough = PlaythroughState(
        save_id="playthrough_001",
        world_id="fantasy_001",
        player_state={
            "name": "Adventurer",
            "hp": 100,
            "max_hp": 100,
            "level": 1,
            "location": "golden_oak_inn"
        },
        npc_states={
            "bartender": {
                "location": "golden_oak_inn",
                "mood": "neutral",
                "met_player": False
            }
        },
        inventory=["iron_sword", "leather_armor"],
        flags={"started_quest": False}
    )

    save_path = pm.save_playthrough(playthrough)
    print(f"Save created: {save_path}\n")

    # Tier 3: Update session state (runtime only)
    pm.update_session({
        "turn": 1,
        "player_input": "I enter the tavern",
        "narration": "You push open the heavy wooden door..."
    })
    pm.update_session({
        "turn": 2,
        "player_input": "I talk to the bartender",
        "narration": "The bartender looks up and nods..."
    })

    context = pm.get_session_context()
    print(f"Session context: {len(context)} events in memory\n")

    # Load world back
    loaded_world = pm.load_world("fantasy_001")
    print(f"Loaded world: {loaded_world.name}")
    print(f"  Locations: {len(loaded_world.locations)}")
    print(f"  NPCs: {len(loaded_world.npcs)}")
    print(f"  Rules: {len(loaded_world.rules)}\n")

    # Load save back
    loaded_save = pm.load_playthrough("playthrough_001")
    print(f"Loaded save: {loaded_save.save_id}")
    print(f"  Player HP: {loaded_save.player_state['hp']}/{loaded_save.player_state['max_hp']}")
    print(f"  Inventory: {loaded_save.inventory}")

    # Export for distribution
    export_path = pm.export_world_for_distribution("fantasy_001")
    print(f"\nExported for distribution: {export_path}")


if __name__ == "__main__":
    example_usage()
```

---

## Best Practice 4: Visual Rule Engine

### Implementation

```python
from dataclasses import dataclass, field
from typing import List, Dict, Any, Callable, Optional
from enum import Enum
import json


class TriggerType(Enum):
    """Types of trigger conditions"""
    TIME = "time"
    LOCATION = "location"
    ACTION = "action"
    FLAG = "flag"
    ITEM = "item"
    NPC_STATE = "npc_state"
    STAT = "stat"


class Operator(Enum):
    """Comparison operators"""
    EQUAL = "=="
    NOT_EQUAL = "!="
    GREATER = ">"
    LESS = "<"
    GREATER_EQUAL = ">="
    LESS_EQUAL = "<="
    CONTAINS = "contains"
    NOT_CONTAINS = "not_contains"


@dataclass
class Condition:
    """A single condition to check"""
    trigger_type: TriggerType
    operator: Operator
    value: Any
    description: str = ""  # Human-readable description

    def evaluate(self, game_state: Dict[str, Any]) -> bool:
        """
        Evaluate condition against game state

        Args:
            game_state: Current game state dictionary

        Returns:
            True if condition is met, False otherwise
        """
        if self.trigger_type == TriggerType.TIME:
            current_time = game_state.get("time", 0)
            return self._compare(current_time, self.value, self.operator)

        elif self.trigger_type == TriggerType.LOCATION:
            player_location = game_state.get("player_location", "")
            return self._compare(player_location, self.value, self.operator)

        elif self.trigger_type == TriggerType.FLAG:
            flag_value = game_state.get("flags", {}).get(self.value, False)
            return flag_value if self.operator == Operator.EQUAL else not flag_value

        elif self.trigger_type == TriggerType.ITEM:
            inventory = game_state.get("inventory", [])
            has_item = self.value in inventory
            return has_item if self.operator == Operator.CONTAINS else not has_item

        elif self.trigger_type == TriggerType.NPC_STATE:
            # Value format: "npc_id.property"
            npc_id, prop = self.value.split(".", 1) if "." in self.value else (self.value, "present")
            npc_states = game_state.get("npc_states", {})
            npc_state = npc_states.get(npc_id, {})
            actual_value = npc_state.get(prop, False)
            return self._compare(actual_value, True, self.operator)

        elif self.trigger_type == TriggerType.STAT:
            # Value format: "stat_name:threshold"
            stat_name, threshold = self.value.split(":", 1) if ":" in self.value else (self.value, 0)
            player_stats = game_state.get("player_stats", {})
            actual_value = player_stats.get(stat_name, 0)
            return self._compare(actual_value, int(threshold), self.operator)

        return False

    def _compare(self, actual: Any, expected: Any, operator: Operator) -> bool:
        """Compare values based on operator"""
        if operator == Operator.EQUAL:
            return actual == expected
        elif operator == Operator.NOT_EQUAL:
            return actual != expected
        elif operator == Operator.GREATER:
            return actual > expected
        elif operator == Operator.LESS:
            return actual < expected
        elif operator == Operator.GREATER_EQUAL:
            return actual >= expected
        elif operator == Operator.LESS_EQUAL:
            return actual <= expected
        elif operator == Operator.CONTAINS:
            return expected in str(actual)
        elif operator == Operator.NOT_CONTAINS:
            return expected not in str(actual)
        return False


@dataclass
class Action:
    """An action to execute when conditions met"""
    action_type: str
    parameters: Dict[str, Any]
    description: str = ""  # Human-readable description

    def execute(self, game_state: Dict[str, Any]) -> Dict[str, Any]:
        """
        Execute action and return state changes

        Args:
            game_state: Current game state

        Returns:
            Dictionary of state changes to apply
        """
        if self.action_type == "set_flag":
            return {"flags": {self.parameters["flag"]: self.parameters.get("value", True)}}

        elif self.action_type == "spawn_npc":
            npc_id = self.parameters["npc_id"]
            location = self.parameters["location"]
            return {"npc_states": {npc_id: {"location": location, "spawned": True}}}

        elif self.action_type == "trigger_event":
            return {"events": [self.parameters["event_id"]]}

        elif self.action_type == "modify_stat":
            stat = self.parameters["stat"]
            value = self.parameters["value"]
            operation = self.parameters.get("operation", "set")  # set, add, subtract

            if operation == "add":
                return {"player_stats": {stat: game_state.get("player_stats", {}).get(stat, 0) + value}}
            elif operation == "subtract":
                return {"player_stats": {stat: game_state.get("player_stats", {}).get(stat, 0) - value}}
            else:  # set
                return {"player_stats": {stat: value}}

        elif self.action_type == "give_item":
            return {"inventory_add": [self.parameters["item_id"]]}

        elif self.action_type == "remove_item":
            return {"inventory_remove": [self.parameters["item_id"]]}

        elif self.action_type == "display_message":
            return {"messages": [self.parameters["message"]]}

        elif self.action_type == "start_quest":
            quest_id = self.parameters["quest_id"]
            return {"quests": {quest_id: {"status": "active", "step": 0}}}

        return {}


@dataclass
class Rule:
    """A complete rule with conditions and actions (StarCraft-inspired)"""
    rule_id: str
    name: str
    conditions: List[Condition] = field(default_factory=list)
    actions: List[Action] = field(default_factory=list)
    condition_logic: str = "AND"  # AND or OR
    enabled: bool = True
    frequency: str = "once"  # once, always, per_turn
    priority: int = 0  # Higher priority rules execute first
    description: str = ""  # Human-readable description

    def evaluate(self, game_state: Dict[str, Any]) -> bool:
        """
        Check if all conditions are met

        Args:
            game_state: Current game state

        Returns:
            True if conditions met, False otherwise
        """
        if not self.enabled:
            return False

        if not self.conditions:
            return True  # No conditions = always true

        results = [c.evaluate(game_state) for c in self.conditions]

        if self.condition_logic == "AND":
            return all(results)
        else:  # OR
            return any(results)

    def execute(self, game_state: Dict[str, Any]) -> Dict[str, Any]:
        """
        Execute all actions and return state changes

        Args:
            game_state: Current game state

        Returns:
            Dictionary of state changes
        """
        state_changes: Dict[str, Any] = {}

        for action in self.actions:
            changes = action.execute(game_state)

            # Merge changes
            for key, value in changes.items():
                if key in state_changes:
                    if isinstance(value, dict):
                        state_changes[key].update(value)
                    elif isinstance(value, list):
                        state_changes[key].extend(value)
                    else:
                        state_changes[key] = value
                else:
                    state_changes[key] = value

        return state_changes


class RuleEngine:
    """Manages and executes rules (StarCraft trigger system inspired)"""

    def __init__(self):
        self.rules: List[Rule] = []
        self.executed_once: set = set()  # Track "once" rules
        self.execution_history: List[Dict[str, Any]] = []

    def add_rule(self, rule: Rule):
        """Add rule to engine"""
        self.rules.append(rule)
        # Sort by priority (higher first)
        self.rules.sort(key=lambda r: r.priority, reverse=True)

    def remove_rule(self, rule_id: str):
        """Remove rule from engine"""
        self.rules = [r for r in self.rules if r.rule_id != rule_id]

    def process_turn(self, game_state: Dict[str, Any]) -> List[Dict[str, Any]]:
        """
        Process all rules for current turn

        Args:
            game_state: Current game state

        Returns:
            List of state change dictionaries
        """
        state_changes = []

        for rule in self.rules:
            if rule.evaluate(game_state):
                # Check frequency
                if rule.frequency == "once":
                    if rule.rule_id in self.executed_once:
                        continue
                    self.executed_once.add(rule.rule_id)

                # Execute and collect changes
                changes = rule.execute(game_state)
                if changes:
                    execution_record = {
                        "rule_id": rule.rule_id,
                        "rule_name": rule.name,
                        "changes": changes
                    }
                    state_changes.append(execution_record)
                    self.execution_history.append(execution_record)

        return state_changes

    def export_visual_format(self, rule: Rule) -> Dict[str, Any]:
        """
        Export rule in visual editor format (JSON)

        Args:
            rule: Rule to export

        Returns:
            Dictionary suitable for visual editor
        """
        return {
            "id": rule.rule_id,
            "name": rule.name,
            "description": rule.description,
            "enabled": rule.enabled,
            "priority": rule.priority,
            "frequency": rule.frequency,
            "condition_logic": rule.condition_logic,
            "conditions": [
                {
                    "type": c.trigger_type.value,
                    "operator": c.operator.value,
                    "value": c.value,
                    "description": c.description
                }
                for c in rule.conditions
            ],
            "actions": [
                {
                    "type": a.action_type,
                    "params": a.parameters,
                    "description": a.description
                }
                for a in rule.actions
            ]
        }

    def import_visual_format(self, data: Dict[str, Any]) -> Rule:
        """
        Import rule from visual editor format

        Args:
            data: Dictionary from visual editor

        Returns:
            Rule object
        """
        conditions = [
            Condition(
                trigger_type=TriggerType(c["type"]),
                operator=Operator(c["operator"]),
                value=c["value"],
                description=c.get("description", "")
            )
            for c in data.get("conditions", [])
        ]

        actions = [
            Action(
                action_type=a["type"],
                parameters=a["params"],
                description=a.get("description", "")
            )
            for a in data.get("actions", [])
        ]

        return Rule(
            rule_id=data["id"],
            name=data["name"],
            conditions=conditions,
            actions=actions,
            condition_logic=data.get("condition_logic", "AND"),
            enabled=data.get("enabled", True),
            frequency=data.get("frequency", "once"),
            priority=data.get("priority", 0),
            description=data.get("description", "")
        )

    def save_to_file(self, filename: str):
        """Save all rules to JSON file"""
        data = [self.export_visual_format(rule) for rule in self.rules]
        with open(filename, 'w') as f:
            json.dump(data, f, indent=2)

    def load_from_file(self, filename: str):
        """Load rules from JSON file"""
        with open(filename, 'r') as f:
            data = json.load(f)

        self.rules.clear()
        for rule_data in data:
            rule = self.import_visual_format(rule_data)
            self.add_rule(rule)


# Usage Example
def example_usage():
    """Demonstrate visual rule engine"""

    engine = RuleEngine()

    # Rule 1: Midnight Haunting
    midnight_rule = Rule(
        rule_id="midnight_ghost",
        name="Midnight Haunting",
        description="Spawn ghost at midnight if player is in haunted mansion",
        conditions=[
            Condition(
                trigger_type=TriggerType.TIME,
                operator=Operator.EQUAL,
                value=0,  # Midnight (00:00)
                description="Time is midnight"
            ),
            Condition(
                trigger_type=TriggerType.LOCATION,
                operator=Operator.EQUAL,
                value="haunted_mansion",
                description="Player is in haunted mansion"
            ),
            Condition(
                trigger_type=TriggerType.FLAG,
                operator=Operator.EQUAL,
                value="ghost_defeated",
                description="Ghost hasn't been defeated yet"
            )
        ],
        actions=[
            Action(
                action_type="spawn_npc",
                parameters={"npc_id": "ghost", "location": "haunted_mansion"},
                description="Spawn ghost NPC"
            ),
            Action(
                action_type="set_flag",
                parameters={"flag": "haunting_started", "value": True},
                description="Mark haunting as started"
            ),
            Action(
                action_type="display_message",
                parameters={"message": "A chill runs down your spine as a ghostly figure materializes before you."},
                description="Display atmospheric message"
            )
        ],
        frequency="once",
        priority=10
    )

    engine.add_rule(midnight_rule)

    # Rule 2: Reward for Defeating Ghost
    reward_rule = Rule(
        rule_id="ghost_reward",
        name="Reward for Defeating Ghost",
        description="Give player magic sword after defeating ghost",
        conditions=[
            Condition(
                trigger_type=TriggerType.FLAG,
                operator=Operator.EQUAL,
                value="ghost_defeated",
                description="Ghost has been defeated"
            )
        ],
        actions=[
            Action(
                action_type="give_item",
                parameters={"item_id": "magic_sword"},
                description="Give magic sword to player"
            ),
            Action(
                action_type="start_quest",
                parameters={"quest_id": "cleanse_mansion"},
                description="Start quest to cleanse the mansion"
            ),
            Action(
                action_type="display_message",
                parameters={"message": "The ghost dissipates, leaving behind a glowing sword."},
                description="Display reward message"
            )
        ],
        frequency="once",
        priority=5
    )

    engine.add_rule(reward_rule)

    # Rule 3: Low Health Warning
    health_warning = Rule(
        rule_id="low_health_warning",
        name="Low Health Warning",
        description="Warn player when health drops below 25%",
        conditions=[
            Condition(
                trigger_type=TriggerType.STAT,
                operator=Operator.LESS,
                value="hp:25",
                description="HP less than 25"
            )
        ],
        actions=[
            Action(
                action_type="display_message",
                parameters={"message": "You're badly wounded! Find healing soon!"},
                description="Display warning"
            )
        ],
        frequency="per_turn",
        priority=100  # High priority (safety warning)
    )

    engine.add_rule(health_warning)

    # Test with game state
    game_state = {
        "time": 0,  # Midnight
        "player_location": "haunted_mansion",
        "flags": {"ghost_defeated": False},
        "inventory": [],
        "player_stats": {"hp": 100}
    }

    print("=== Turn 1: Entering haunted mansion at midnight ===")
    changes = engine.process_turn(game_state)
    for change in changes:
        print(f"Rule triggered: {change['rule_name']}")
        print(f"Changes: {change['changes']}\n")

    # Export rule to visual format
    print("=== Visual Editor Format ===")
    visual_format = engine.export_visual_format(midnight_rule)
    print(json.dumps(visual_format, indent=2))

    # Save to file
    engine.save_to_file("rules.json")
    print("\nRules saved to rules.json")


if __name__ == "__main__":
    example_usage()
```

---

## Cross-References

### Pattern Documentation
- [[LLM World Engine/patterns/00-PATTERN-INDEX|Pattern Library]]
- [[LLM World Engine/patterns/architectural/Program-First-Architecture|Program-First Architecture]]
- [[LLM World Engine/patterns/state/Three-Tier-Persistence|Three-Tier Persistence]]

### Implementation Details
- [[02-Pattern-Implementation|Pattern Implementation Analysis]]
- [[03-Prompt-Implementation|Prompt Implementation Analysis]]
- [[08-Anti-Hallucination-System|Anti-Hallucination Deep Dive]]

---

## Tags

#code-examples #best-practices #python #type-hints #production-ready #anti-hallucination #token-limiting #persistence #rule-engine #starcraft-triggers

---

## Next: Production Lessons
See [[05-Production-Lessons]] for key insights and lessons learned from real-world ChatBotRPG usage.
