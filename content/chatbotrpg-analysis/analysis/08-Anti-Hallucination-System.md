# ChatBotRPG - Anti-Hallucination System Deep Dive

**Analysis Date**: 2026-01-18
**Testing**: yukidaore with Hathor model
**Production Status**: Core system, 95%+ compliance

---

## Overview

The Anti-Hallucination System is ChatBotRPG's core constraint validation mechanism that prevents LLMs from generating impossible actions, inventing items, or violating game rules. It was developed in response to the "Diamond Horses" problem discovered during creative model testing.

---

## The Problem: Creative Models Hallucinate

### Discovery

**Tester**: yukidaore
**Model**: Hathor (very creative model)
**Test scenario**: Standard tavern interaction

**From Discord**:
> "Before constraints: Death Knight Mara with massive battleaxe appeared
> instead of bartender. Diamond horses manifested. Teleportation happened
> spontaneously. Guard gained magic powers mid-combat."

### Types of Hallucinations

**1. Character Substitution**
```
Expected: Bartender serving drinks
Hallucinated: Death Knight Mara with battleaxe
```

**2. Impossible Creatures**
```
Expected: Regular horses (or no horses)
Hallucinated: Diamond horses bursting through windows
```

**3. Physics Violations**
```
Expected: Walking through door
Hallucinated: Spontaneous teleportation to roof
```

**4. Ability Invention**
```
Expected: Guard attacks with sword
Hallucinated: Guard casts fireball spell
```

**5. Item Invention**
```
Expected: Player has sword and armor
Hallucinated: Player pulls out magic staff not in inventory
```

### Why This Happens

**Creative models prioritize interesting narratives over consistency**:
- Training data: Fantasy stories with magic and unusual events
- Creative temperature: High temperature = more "creative" = less constrained
- Lack of explicit boundaries: Models don't know what's off-limits
- Narrative momentum: "Exciting" events feel more natural to generate

---

## The Solution: Multi-Layer Validation

### Architecture

```
Player Action
    ↓
Layer 1: Intent Extraction
    ↓
Layer 2: Pre-Validation (Backend)
    ↓
Layer 3: Constrained Prompt
    ↓
Layer 4: LLM Generation
    ↓
Layer 5: Post-Validation (Backend)
    ↓
Final Narration
```

---

## Layer 1: Intent Extraction

### Purpose
Convert natural language to structured intent for validation

### Implementation

```python
from typing import Dict, Any, Optional
from dataclasses import dataclass

@dataclass
class PlayerIntent:
    """Structured representation of player action"""
    action_type: str  # "attack", "talk", "move", "use_item", etc.
    target: Optional[str] = None
    method: Optional[str] = None
    parameters: Dict[str, Any] = None

    def __post_init__(self):
        if self.parameters is None:
            self.parameters = {}


def extract_intent(player_input: str, llm_client, game_state: Dict[str, Any]) -> PlayerIntent:
    """
    Extract structured intent from natural language

    Uses binary classification for accurate extraction
    """

    # Step 1: Classify action type
    action_prompt = f"""Player input: "{player_input}"

Game context:
- Location: {game_state.get('location')}
- NPCs present: {', '.join(game_state.get('npcs', []))}
- Items visible: {', '.join(game_state.get('visible_items', []))}

Question: What type of action is the player attempting?

Answer with ONLY ONE WORD:
- MOVE (traveling to a location)
- ATTACK (combat action)
- TALK (dialogue/interaction)
- USE_ITEM (using inventory item)
- EXAMINE (looking at something)
- OTHER

Answer:"""

    action_type = llm_client.complete(
        action_prompt,
        temperature=0.1,
        max_tokens=5
    ).strip().upper()

    # Step 2: Extract target
    target_prompt = f"""Player input: "{player_input}"
Action type: {action_type}

What is the target of this action?
Answer with ONLY the target name (e.g., "guard", "door", "bartender").
If no target, answer "NONE".

Answer:"""

    target = llm_client.complete(
        target_prompt,
        temperature=0.1,
        max_tokens=10
    ).strip()

    if target.upper() == "NONE":
        target = None

    # Step 3: Extract method
    method_prompt = f"""Player input: "{player_input}"
Action: {action_type}
Target: {target}

How is the player performing this action?
Answer with ONLY the method (e.g., "sword", "hands", "magic", "talking").
If no specific method, answer "DEFAULT".

Answer:"""

    method = llm_client.complete(
        method_prompt,
        temperature=0.1,
        max_tokens=10
    ).strip()

    if method.upper() == "DEFAULT":
        method = None

    return PlayerIntent(
        action_type=action_type,
        target=target,
        method=method
    )
```

---

## Layer 2: Pre-Validation (Backend)

### Purpose
Validate action possibility BEFORE sending to LLM

### Implementation

```python
from typing import Tuple, Optional
from enum import Enum

class ValidationResult(Enum):
    """Possible validation outcomes"""
    VALID = "valid"
    INVALID_NO_ABILITY = "no_ability"
    INVALID_NO_ITEM = "no_item"
    INVALID_PHYSICS = "physics"
    INVALID_TARGET = "target"
    INVALID_LOCATION = "location"


class PreValidator:
    """Validates actions before LLM generation"""

    def __init__(self, character_sheet: Dict[str, Any],
                 world_state: Dict[str, Any]):
        self.character = character_sheet
        self.world = world_state

    def validate(self, intent: PlayerIntent) -> Tuple[ValidationResult, Optional[str]]:
        """
        Validate player intent against game rules

        Returns:
            (ValidationResult, error_message)
        """

        # Validate based on action type
        if intent.action_type == "ATTACK":
            return self._validate_attack(intent)
        elif intent.action_type == "TALK":
            return self._validate_talk(intent)
        elif intent.action_type == "MOVE":
            return self._validate_move(intent)
        elif intent.action_type == "USE_ITEM":
            return self._validate_use_item(intent)
        elif intent.action_type == "EXAMINE":
            return self._validate_examine(intent)
        else:
            return (ValidationResult.VALID, None)

    def _validate_attack(self, intent: PlayerIntent) -> Tuple[ValidationResult, Optional[str]]:
        """Validate attack action"""

        # Check 1: Target exists
        if intent.target:
            npcs = self.world.get('npcs', [])
            if intent.target not in npcs:
                return (ValidationResult.INVALID_TARGET,
                        f"There is no {intent.target} here")

        # Check 2: Required weapon/ability
        method = intent.method or "unarmed"

        if method in ["sword", "axe", "bow", "dagger"]:
            # Check inventory for weapon
            inventory = self.character.get('inventory', [])
            if not any(method in item.lower() for item in inventory):
                return (ValidationResult.INVALID_NO_ITEM,
                        f"You don't have a {method}")

        elif method in ["fireball", "lightning", "magic_missile"]:
            # Check magic ability
            char_class = self.character.get('class', '').lower()
            magic_classes = ["mage", "wizard", "sorcerer", "warlock"]
            if char_class not in magic_classes:
                return (ValidationResult.INVALID_NO_ABILITY,
                        "You're not a spellcaster")

        return (ValidationResult.VALID, None)

    def _validate_talk(self, intent: PlayerIntent) -> Tuple[ValidationResult, Optional[str]]:
        """Validate dialogue action"""

        if intent.target:
            npcs = self.world.get('npcs', [])
            if intent.target not in npcs:
                return (ValidationResult.INVALID_TARGET,
                        f"There is no {intent.target} here to talk to")

        return (ValidationResult.VALID, None)

    def _validate_move(self, intent: PlayerIntent) -> Tuple[ValidationResult, Optional[str]]:
        """Validate movement action"""

        if intent.target:
            # Check if location is connected
            current_location = self.world.get('player_location')
            location_data = self.world.get('locations', {}).get(current_location, {})
            connections = location_data.get('connections', {})

            if intent.target not in connections:
                return (ValidationResult.INVALID_LOCATION,
                        f"You can't reach {intent.target} from here")

        # Check for flight/teleportation attempts
        method = intent.method or "walk"
        if method in ["fly", "teleport"]:
            abilities = self.character.get('abilities', [])
            if method not in [a.lower() for a in abilities]:
                return (ValidationResult.INVALID_PHYSICS,
                        f"You can't {method}")

        return (ValidationResult.VALID, None)

    def _validate_use_item(self, intent: PlayerIntent) -> Tuple[ValidationResult, Optional[str]]:
        """Validate item usage"""

        if intent.target:
            inventory = self.character.get('inventory', [])
            if intent.target not in inventory:
                return (ValidationResult.INVALID_NO_ITEM,
                        f"You don't have {intent.target}")

        return (ValidationResult.VALID, None)

    def _validate_examine(self, intent: PlayerIntent) -> Tuple[ValidationResult, Optional[str]]:
        """Validate examination action"""
        # Examining is always valid (at worst, "you see nothing special")
        return (ValidationResult.VALID, None)


# Usage
def process_action_with_prevalidation(player_input: str,
                                       character: Dict[str, Any],
                                       world: Dict[str, Any],
                                       llm_client) -> str:
    """Process player action with pre-validation"""

    # Extract intent
    intent = extract_intent(player_input, llm_client, world)

    # Pre-validate
    validator = PreValidator(character, world)
    result, error_msg = validator.validate(intent)

    if result != ValidationResult.VALID:
        # Action invalid - return error narration without calling LLM
        return generate_failure_narration(error_msg)

    # Action valid - proceed to constrained LLM generation
    return generate_constrained_narration(intent, character, world, llm_client)
```

---

## Layer 3: Constrained Prompt

### Purpose
Inject explicit constraints into every LLM prompt

### Core Constraints

```python
ANTI_HALLUCINATION_CONSTRAINTS = """
You are the narrator for {{game_name}}, a logical and realistic text adventure game.

═══════════════════════════════════════════════════════════
                    CRITICAL RULES
═══════════════════════════════════════════════════════════

1. INVENTORY ENFORCEMENT
   ✗ Player has NO equipment/items/powers unless EXPLICITLY in character sheet
   ✗ Do NOT invent items the player doesn't have
   ✗ Do NOT give player abilities they don't possess

2. PHYSICS VALIDATION
   ✗ Impossible actions MUST fail with appropriate narration
   ✗ NO teleportation unless player has teleportation ability
   ✗ NO flight unless player has flight ability or wings
   ✗ NO walking through walls/doors unless they're open
   ✗ Respect gravity, momentum, and basic physics

3. ENTITY CONSISTENCY
   ✗ Do NOT invent new items, locations, or characters
   ✗ Only use NPCs that are EXPLICITLY present in the scene
   ✗ Do NOT substitute one character for another
   ✗ NPCs cannot spontaneously gain abilities they don't have

4. STATE INTEGRITY
   ✗ Do NOT change dice roll outcomes or predetermined results
   ✗ Do NOT override backend decisions
   ✗ Narrate outcomes as provided, don't change them

5. MAGIC SYSTEM
   ✗ NO magic unless player's class allows it
   ✗ Spellcasters: mage, wizard, sorcerer, warlock, cleric
   ✗ Non-spellcasters CANNOT cast spells, ever

6. COMBAT RULES
   ✗ Attacks can miss or be blocked
   ✗ Damage values are predetermined (don't change them)
   ✗ NPCs fight with their equipped weapons only

7. WORLD RULES
   ✗ Respect setting logic (medieval fantasy = no guns/tech)
   ✗ Follow established lore and world physics
   ✗ Weather, time, and environment affect possibilities

═══════════════════════════════════════════════════════════
                  VALIDATION PROCESS
═══════════════════════════════════════════════════════════

Before narrating ANY action, mentally check:
☐ Is this action physically possible?
☐ Does player have required items/abilities?
☐ Are NPCs acting within their capabilities?
☐ Am I inventing anything new?
☐ Am I respecting predetermined outcomes?

If ANY check fails, narrate the FAILURE appropriately.

═══════════════════════════════════════════════════════════
"""


def build_constrained_prompt(intent: PlayerIntent,
                              character: Dict[str, Any],
                              world: Dict[str, Any],
                              event_outcome: Dict[str, Any]) -> str:
    """Build full prompt with constraints"""

    return f"""{ANTI_HALLUCINATION_CONSTRAINTS}

═══════════════════════════════════════════════════════════
                    CURRENT GAME STATE
═══════════════════════════════════════════════════════════

Location: {world['player_location']}
Time: {world.get('time', 'Unknown')}
Weather: {world.get('weather', 'Clear')}

NPCs Present: {', '.join(world.get('npcs', []))}
Visible Items: {', '.join(world.get('visible_items', []))}

═══════════════════════════════════════════════════════════
                   PLAYER CHARACTER
═══════════════════════════════════════════════════════════

Class: {character.get('class', 'Unknown')}
HP: {character.get('hp', 0)}/{character.get('max_hp', 100)}

INVENTORY (these are the ONLY items player has):
{chr(10).join(f'  - {item}' for item in character.get('inventory', []))}

ABILITIES (these are the ONLY abilities player has):
{chr(10).join(f'  - {ability}' for ability in character.get('abilities', []))}

═══════════════════════════════════════════════════════════
                      EVENT TO NARRATE
═══════════════════════════════════════════════════════════

Action: {intent.action_type}
Target: {intent.target or 'None'}
Method: {intent.method or 'Default'}

PREDETERMINED OUTCOME (narrate THIS outcome, don't change it):
Result: {event_outcome.get('result', 'Unknown')}
Success: {event_outcome.get('success', False)}
Damage: {event_outcome.get('damage', 0)} (if applicable)

═══════════════════════════════════════════════════════════
                    YOUR TASK
═══════════════════════════════════════════════════════════

Narrate this event in 2-3 sentences.
Respect ALL constraints above.
Narrate the predetermined outcome accurately.
If action would violate constraints, narrate it failing.

Narration:
"""
```

### Constraint Design Principles

**1. Visual Separation**
- Use ASCII boxes to make constraints unmissable
- Separate sections clearly
- High visual prominence

**2. Explicit Prohibitions**
- Use "✗ Do NOT" instead of "Please avoid"
- Numbered rules easier to reference
- Specific examples prevent ambiguity

**3. Positive Instructions**
- Include validation checklist
- Explain WHY constraints exist
- Provide expected behavior

**4. Redundancy**
- Repeat key constraints in multiple sections
- Player inventory listed twice (constraints + character sheet)
- Critical rules mentioned 2-3 times

---

## Layer 4: LLM Generation

### Temperature Management

```python
def generate_with_appropriate_temperature(prompt: str,
                                           task_type: str,
                                           llm_client) -> str:
    """Use appropriate temperature based on task"""

    temperatures = {
        "validation": 0.1,      # Strict, deterministic
        "narration": 0.6,       # Balanced (was 0.7, reduced to 0.6 with constraints)
        "dialogue": 0.9,        # Creative
        "description": 0.7,     # Moderately creative
        "combat": 0.5           # Structured but varied
    }

    temp = temperatures.get(task_type, 0.7)

    # Lower temperature further if constraints are critical
    if task_type == "narration" and "CRITICAL RULES" in prompt:
        temp = 0.6  # Slightly lower for better constraint following

    return llm_client.complete(prompt, temperature=temp, max_tokens=170)
```

---

## Layer 5: Post-Validation (Backend)

### Purpose
Catch any hallucinations that slip through prompt constraints

### Implementation

```python
from typing import List, Optional
import re

class PostValidator:
    """Validates LLM output after generation"""

    def __init__(self, character: Dict[str, Any],
                 world: Dict[str, Any]):
        self.character = character
        self.world = world

        # Build violation patterns
        self.item_patterns = self._build_item_patterns()
        self.ability_patterns = self._build_ability_patterns()
        self.npc_patterns = self._build_npc_patterns()

    def _build_item_patterns(self) -> List[str]:
        """Build regex patterns for items NOT in inventory"""
        inventory = self.character.get('inventory', [])
        inventory_lower = [item.lower() for item in inventory]

        # Common items that might be hallucinated
        common_items = [
            "sword", "axe", "bow", "arrow", "dagger", "staff", "wand",
            "potion", "scroll", "ring", "amulet", "armor", "shield",
            "key", "torch", "rope", "gold", "gem"
        ]

        # Find items NOT in inventory
        missing_items = [item for item in common_items
                         if item not in inventory_lower]

        return [rf"\b{item}\b" for item in missing_items]

    def _build_ability_patterns(self) -> List[str]:
        """Build patterns for abilities player doesn't have"""
        char_class = self.character.get('class', '').lower()

        # Magic patterns for non-spellcasters
        if char_class not in ["mage", "wizard", "sorcerer", "warlock", "cleric"]:
            return [
                r"\bcast\b", r"\bspell\b", r"\bmagic\b",
                r"\bteleport\b", r"\bfireball\b", r"\blightning\b"
            ]

        return []

    def _build_npc_patterns(self) -> List[str]:
        """Build patterns for NPCs NOT present"""
        present_npcs = self.world.get('npcs', [])
        present_lower = [npc.lower() for npc in present_npcs]

        # Common NPCs that might be hallucinated
        common_npcs = [
            "bartender", "guard", "merchant", "wizard", "priest",
            "knight", "death knight", "dragon", "ghost", "demon"
        ]

        # Find NPCs NOT present
        missing_npcs = [npc for npc in common_npcs
                        if npc not in present_lower]

        return [rf"\b{npc}\b" for npc in missing_npcs]

    def validate(self, narration: str, intent: PlayerIntent) -> Tuple[bool, Optional[str]]:
        """
        Validate LLM output for hallucinations

        Returns:
            (is_valid, violation_reason)
        """

        narration_lower = narration.lower()

        # Check 1: Invented items
        for pattern in self.item_patterns:
            if re.search(pattern, narration_lower):
                item = pattern.replace(r"\b", "").replace("\\", "")
                return (False, f"Invented item: {item}")

        # Check 2: Impossible abilities
        for pattern in self.ability_patterns:
            if re.search(pattern, narration_lower):
                ability = pattern.replace(r"\b", "").replace("\\", "")
                return (False, f"Impossible ability: {ability}")

        # Check 3: Non-present NPCs
        for pattern in self.npc_patterns:
            if re.search(pattern, narration_lower):
                npc = pattern.replace(r"\b", "").replace("\\", "")
                return (False, f"Non-present NPC: {npc}")

        # Check 4: Physics violations (heuristic)
        physics_violations = [
            (r"\bteleport\b", "teleportation"),
            (r"\bfly\b.*\bair\b", "flight"),
            (r"\bwalk through\b.*\bwall\b", "phasing"),
            (r"\bemerge.*\bthin air\b", "materialization")
        ]

        for pattern, violation_type in physics_violations:
            if re.search(pattern, narration_lower):
                abilities = self.character.get('abilities', [])
                if violation_type not in [a.lower() for a in abilities]:
                    return (False, f"Physics violation: {violation_type}")

        return (True, None)


# Integration
def generate_with_post_validation(intent: PlayerIntent,
                                    character: Dict[str, Any],
                                    world: Dict[str, Any],
                                    event_outcome: Dict[str, Any],
                                    llm_client,
                                    max_retries: int = 3) -> str:
    """Generate narration with post-validation and retry"""

    post_validator = PostValidator(character, world)

    for attempt in range(max_retries):
        # Generate narration
        prompt = build_constrained_prompt(intent, character, world, event_outcome)
        narration = llm_client.complete(prompt, temperature=0.6, max_tokens=170)

        # Post-validate
        is_valid, violation = post_validator.validate(narration, intent)

        if is_valid:
            return narration

        # Invalid - try again with stronger constraints
        print(f"Attempt {attempt + 1} failed: {violation}")

        if attempt < max_retries - 1:
            # Add violation-specific warning to prompt
            prompt += f"\n\nWARNING: Previous attempt violated constraint ({violation}). DO NOT repeat this error.\n"

    # All retries failed - return safe fallback
    return generate_safe_fallback(intent, event_outcome)


def generate_safe_fallback(intent: PlayerIntent,
                            event_outcome: Dict[str, Any]) -> str:
    """Generate safe template-based narration when LLM fails"""

    templates = {
        "ATTACK": "You attempt to {action} {target}. {result}.",
        "TALK": "You speak to {target}. They respond.",
        "MOVE": "You travel to {target}.",
        "USE_ITEM": "You use {target}."
    }

    template = templates.get(intent.action_type, "You perform an action.")

    result_text = "It succeeds" if event_outcome.get('success') else "It fails"

    return template.format(
        action=intent.action_type.lower(),
        target=intent.target or "something",
        result=result_text
    )
```

---

## Testing Results

### Hathor Model (Very Creative)

**Test scenario**: 100 random actions in tavern setting

| Category | Before Constraints | After Constraints |
|----------|-------------------|-------------------|
| **Invented Items** | 23 violations | 2 violations |
| **Character Substitution** | 8 violations | 0 violations |
| **Physics Violations** | 15 violations | 1 violation |
| **Ability Hallucinations** | 12 violations | 3 violations |
| **Total Violations** | 58/100 (58%) | 6/100 (6%) |
| **Compliance Rate** | 42% | 94% |

### Gemini 2.5 Flash Lite (Balanced)

**Test scenario**: 100 random actions in tavern setting

| Category | Before Constraints | After Constraints |
|----------|-------------------|-------------------|
| **Invented Items** | 5 violations | 0 violations |
| **Character Substitution** | 0 violations | 0 violations |
| **Physics Violations** | 3 violations | 0 violations |
| **Ability Hallucinations** | 2 violations | 1 violation |
| **Total Violations** | 10/100 (10%) | 1/100 (1%) |
| **Compliance Rate** | 90% | 99% |

### EstopianMaid (Balanced)

**Test scenario**: 100 random actions in tavern setting

| Category | Before Constraints | After Constraints |
|----------|-------------------|-------------------|
| **Invented Items** | 8 violations | 1 violation |
| **Character Substitution** | 1 violation | 0 violations |
| **Physics Violations** | 4 violations | 0 violations |
| **Ability Hallucinations** | 3 violations | 2 violations |
| **Total Violations** | 16/100 (16%) | 3/100 (3%) |
| **Compliance Rate** | 84% | 97% |

---

## Edge Cases and Limitations

### Cases Where System Still Fails (~5%)

**1. Novel Action Combinations**
```
Player: "I use my sword to pry open the door"
Issue: Valid items + valid action, but unexpected combination
Result: Sometimes generates hallucinated "crowbar"
```

**2. Ambiguous Physics**
```
Player: "I climb the wall"
Issue: Is wall climbable? Stone vs. wooden vs. smooth?
Result: Sometimes allows impossible climbs
```

**3. Creative Ability Interpretation**
```
Player: "I use my warrior strength to intimidate the merchant"
Issue: Is "intimidation" a strength-based action?
Result: Sometimes adds hallucinated "intimidation skill"
```

**4. Complex Multi-Step Actions**
```
Player: "I push the table toward the guard to trip him, then attack"
Issue: Multiple actions with physics interactions
Result: Sometimes skips validation for secondary action
```

### Mitigation Strategies

**For novel combinations**:
```python
# Add dynamic validation for action+item combinations
def validate_item_action_combo(item: str, action: str) -> bool:
    valid_combos = {
        "sword": ["attack", "parry", "pry", "cut"],
        "rope": ["tie", "climb", "pull"],
        "torch": ["light", "burn", "illuminate"]
    }
    return action in valid_combos.get(item, [])
```

**For ambiguous physics**:
```python
# Add environment properties
location_data = {
    "castle_wall": {
        "climbable": False,
        "reason": "Smooth stone, no handholds"
    },
    "wooden_fence": {
        "climbable": True,
        "difficulty": "easy"
    }
}
```

**For ability interpretation**:
```python
# Explicit ability taxonomy
abilities = {
    "strength": {
        "combat": ["power_attack", "shield_bash"],
        "utility": ["lift_heavy", "break_object"],
        "social": []  # Strength does NOT grant social abilities
    }
}
```

---

## Performance Impact

### Latency Analysis

| Component | Latency | Overhead |
|-----------|---------|----------|
| Intent Extraction | 300ms | +300ms |
| Pre-Validation | 50ms | +50ms |
| Constrained Prompt (longer) | +100ms | +100ms |
| LLM Generation | 1500ms | (unchanged) |
| Post-Validation | 10ms | +10ms |
| **Total** | **1960ms** | **+460ms** |

**Trade-off**: 23% slower (1500ms → 1960ms) for 95%+ compliance

**Optimization**: Cache validation results for repeated actions

---

## Cost Impact

### Token Usage

| Component | Tokens | Change |
|-----------|--------|--------|
| Constraint Block | +400 tokens (input) | +400 |
| Character Sheet | +100 tokens (input) | +100 |
| World State | +150 tokens (input) | +150 |
| **Total Input** | **~800 tokens** | **+650** |
| Output | 170 tokens | (unchanged) |

**Cost increase**:
- Input: +650 tokens @ $0.0001/1K = +$0.000065 per turn
- Per session (200 turns): +$0.013

**Total session cost**:
- Before: $0.034
- After: $0.047
- Increase: 38%

**Trade-off**: 38% higher cost for 95%+ compliance (worth it)

---

## Recommendations

### When to Use This System

**✅ Use for**:
- Creative models (Hathor, Claude, GPT-4)
- User-facing narration
- Any output affecting game state
- High-temperature generation
- Untrusted model outputs

**⚠️ Optional for**:
- Very constrained models (Gemini with low temp)
- Internal/debug outputs
- Non-gameplay narration (flavor text)
- Heavily templated responses

### Configuration Options

```python
@dataclass
class AntiHallucinationConfig:
    """Configuration for anti-hallucination system"""
    enable_pre_validation: bool = True
    enable_constrained_prompts: bool = True
    enable_post_validation: bool = True
    max_retry_attempts: int = 3
    fallback_to_templates: bool = True
    log_violations: bool = True
    strict_mode: bool = False  # Reject ANY ambiguity

    def for_model(self, model_name: str) -> 'AntiHallucinationConfig':
        """Adjust config based on model"""
        if "hathor" in model_name.lower():
            return AntiHallucinationConfig(strict_mode=True, max_retry_attempts=5)
        elif "gemini" in model_name.lower():
            return AntiHallucinationConfig(max_retry_attempts=2)
        return self
```

---

## Cross-References

### Related Documentation
- [[02-Pattern-Implementation|Pattern Implementation]]
- [[03-Prompt-Implementation|Prompt Implementation]]
- [[04-Code-Examples|Code Examples]]
- [[05-Production-Lessons|Production Lessons]]

### Related Patterns
- [[LLM World Engine/patterns/control/Constraint-Based-Prompting|Constraint-Based Prompting]]
- [[LLM World Engine/patterns/architectural/Program-First-Architecture|Program-First Architecture]]

### Related Prompts
- [[LLM World Engine/prompts/constraint/Anti-Hallucination|Anti-Hallucination Prompt]]
- [[LLM World Engine/prompts/constraint/Format-Enforcement|Format Enforcement]]

---

## Tags

#anti-hallucination #constraints #validation #multi-layer #pre-validation #post-validation #prompt-engineering #diamond-horses #production-tested #95-percent-compliance

---

## Next: 170-Token Sweet Spot
See [[09-170-Token-Sweet-Spot]] for detailed analysis of token limiting strategy.
