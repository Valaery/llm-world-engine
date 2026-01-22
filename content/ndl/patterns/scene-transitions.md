# NDL Pattern: Scene Transitions

#ndl #pattern #scenes #transitions #state-management

## Overview

Scene transitions in NDL manage movement between locations, time skips, and mode changes (exploration → combat, dialogue → action, etc.). By explicitly marking scene boundaries with `wait()` and system markers, NDL enables clean state management, context switching, and narrative pacing without LLM confusion.

## Core Philosophy

Scenes are the fundamental unit of game state and context management. From [[User-monkeyrithms]]:

> "if we consider transitioning scenes to be checkpoints. We'd assume the player will be moving about the game frequently and a scene is usually a location that is visible to the eye from where you're at -- so maybe the interior of a house, to a vast open prairie. But the 'visibility' thing enforces a good way to enforce limits on what a scene is and help you to divide it into enough scenes to where the player can be expected to create checkpoints very often."

### Scene as Context Boundary

A scene defines:
- **Spatial context**: Where entities are located
- **Temporal context**: Current time of day, elapsed time
- **Social context**: Who is present and aware of each other
- **Mode context**: Combat, dialogue, exploration, etc.

Transitions are **not just narrative flavor**—they're state management checkpoints.

## Transition Types

### Spatial Transitions (Location Change)

#### Simple Room-to-Room Movement

```ndl
do($"player", "walk") ~ "through door" -> target($"kitchen")
-> wait("transition")
-> system_response("scene changed to: kitchen")
-> describe("new location")
```

**Generated Narrative**:
> You walk through the doorway into the kitchen. The warm smell of bread fills the air. Morning light streams through a window above the sink, illuminating dust motes dancing in the sunbeams.

#### Fast Travel / Teleportation

```ndl
do($"player", "cast") ~ "teleport spell" -> target($"distant city")
-> wait("travel time")
-> system_response("scene transition: place")
-> describe("arrival at city gates")
```

**Implementation Note** (from [[User-monkeyrithms]]):
> "Move scenes -- you can teleport if you want, skipping the requirement to pick one of the 'links' in and out of the room, later I'll add requirements that you possess that spell first"

Available movement commands trigger scene transitions:
```
/enter, /go, /walk, /jog, /join, /run, /scene, /transit, /travel,
/change, /switch, /move, /teleport, /warp, /fasttravel, /skip
```

### Temporal Transitions (Time Skip)

#### Short Time Skip

```ndl
wait("1 hour") -> system_response("time advanced")
-> describe("passage of time effects")
```

**Generated Narrative**:
> An hour passes. The sun has moved noticeably across the sky. You feel your energy returning after the brief rest.

#### Long Time Skip

```ndl
wait("overnight") -> system_response("scene transition: time, 8 hours elapsed")
-> describe("morning arrival")
```

**Generated Narrative**:
> You bed down for the night. Sleep comes quickly after the day's exertions.
>
> *(8 hours pass)*
>
> Dawn breaks. You wake refreshed, the events of yesterday feeling distant now in the morning light.

#### Travel Montage

```ndl
do($"player", "travel") ~ "on horseback" -> target($"distant_kingdom")
-> wait("3 days travel")
-> system_response("scene transition: time + place")
-> describe("journey highlights and arrival")
```

**Generated Narrative**:
> You mount your horse and set off toward the distant kingdom. The journey takes you through rolling hills and dense forests.
>
> *(3 days pass)*
>
> On the third day, you crest a ridge and see the kingdom's towers on the horizon. By evening, you reach the outer gates.

### Mode Transitions (Context Change)

#### Exploration → Combat

```ndl
do($"goblin", "ambush") -> target($"player")
-> system_response("combat initiated")
-> wait("combat mode")
```

**Implementation Detail** (from [[User-monkeyrithms]]):
> "when combat has begun. The game window will turn red as a visual indicator, and NPCs will be fed different kinds of prompts that facilitate that sort of scene better."

#### Combat → Aftermath

```ndl
do($"player", "strike final blow") -> target($"enemy") -> result("defeated")
-> system_response("combat ended")
-> wait("return to exploration")
-> describe("battlefield aftermath")
```

#### Dialogue → Stealth

```ndl
talk($"guard", $"partner") -> convey("patrol route")
-> do($"player", "hide") ~ "in shadows" -> result("undetected")
-> system_response("stealth mode")
-> wait("guards pass")
```

## Scene Boundary Detection

### NDL Parse Flags for Transitions

From [[User-veritasr]]'s Director implementation:

```python
# Scene transition detection in parser
transition_scene_types = ["time", "place", "both"]

# Parse result structure
{
    "is_ndl": True,
    "continue_last": False,  # True = same scene; False = new scene
    "end_scene": True,       # True = scene boundary detected
    "transition_scene_type": "time",  # or "place" or "both"
    "actions": [...]
}
```

### Automatic Scene Detection

```python
def detect_scene_transition(ndl: str) -> dict:
    """Detect scene boundaries from NDL structure."""

    # Time transitions
    if re.search(r'wait\("[^"]*(?:hour|day|night|week)', ndl):
        return {"end_scene": True, "type": "time"}

    # Place transitions
    if re.search(r'do\(\$[^,]+, "[^"]*(?:travel|enter|leave|go|move)', ndl):
        return {"end_scene": True, "type": "place"}

    # Mode transitions
    if "system_response" in ndl and any(mode in ndl for mode in
                                        ["combat", "stealth", "dialogue"]):
        return {"end_scene": True, "type": "mode"}

    # Explicit scene markers
    if "end_scene" in ndl or "transition" in ndl:
        return {"end_scene": True, "type": "explicit"}

    return {"end_scene": False, "type": None}
```

## Companion and Entity Persistence

### Keeping Companions Across Scenes

```ndl
do($"player", "enter") -> target($"new_area")
-> system_response("transitioning with companion: $companion")
-> wait("transition")
-> describe("arrival with companion")
```

**Implementation Note** (from [[User-monkeyrithms]]):
> "now if I transition scenes, she will come with me to the next scene"

### Entity Removal at Scene Boundary

```ndl
do($"enemy", "die") -> result("death")
-> system_response("entity removed: $enemy")
-> wait("transition")
# Enemy no longer exists in new scene context
```

**Death Handling**:
> "if a character dies, they are 'transported' (removed) to a folder called Afterlife, where they are no longer a part of the game, but they still exist, for any 'resurrection' mechanics later"

## Scene Connection Graphs

### Linked Locations

```ndl
system_response("current scene connections")
-> describe("available exits")
# North: Forest Path
# South: Village Square
# East: River Crossing

do($"player", "go north")
-> wait("transition")
-> system_response("scene: forest_path")
```

**Design Philosophy** (from [[User-irovos]]):
> "You don't get to conjure up a town's square. It is there and the llm has already fed it with connecting locations"

### Graph Traversal

```python
class SceneGraph:
    """Manages scene connections and transitions."""

    def __init__(self):
        self.nodes: dict[str, Scene] = {}
        self.edges: dict[str, list[str]] = {}

    def transition(self, from_scene: str, to_scene: str,
                   player: Entity) -> str:
        """Generate NDL for scene transition."""

        if to_scene not in self.edges.get(from_scene, []):
            return f'system_response("cannot reach {to_scene} from here")'

        ndl = f'do(${player.id}, "travel") -> target(${to_scene})'
        ndl += f' -> wait("transition")'
        ndl += f' -> system_response("scene changed to: {to_scene}")'

        return ndl
```

## Time and Pacing Management

### Action-Based Time Advancement

From [[User-irovos]]:
> "Programmatically. I count up time based on some default interactions. Right now the poc involves going from one part of town, to a wilderness, then gathering resources, each interaction adds to a tracker that informs the player that they're getting sleepy"

```ndl
do($"player", "gather") ~ "firewood" -> result("success")
-> system_response("time +15 minutes")
-> system_response("fatigue +5")

# After multiple actions...
system_response("you're feeling tired")
-> wait("player decision")
```

### Fatigue-Based Scene Transition

```ndl
system_response("exhaustion threshold reached")
-> do($"player", "collapse") -> result("forced rest")
-> wait("8 hours")
-> system_response("scene transition: time, dawn, full rest")
-> describe("waking up")
```

### Scene Duration Tracking

```python
def track_scene_duration(actions: list[str]) -> int:
    """Calculate elapsed time from actions in current scene."""

    time_costs = {
        "search": 10,      # minutes
        "talk": 5,
        "craft": 30,
        "travel_short": 15,
        "combat_turn": 1,  # ~6 seconds per turn
    }

    total_minutes = sum(time_costs.get(action_type, 1)
                       for action_type in actions)

    if total_minutes >= 60:  # 1 hour threshold
        return f'wait("{total_minutes} minutes") -> system_response("scene transition: time")'

    return ""
```

## State Persistence Across Transitions

### What Persists

```ndl
# Inventory persists
do($"player", "enter") -> target($"new_room")
-> wait("transition")
-> system_response("inventory maintained")

# Quest states persist
-> system_response("quest 'Find the Artifact': In Progress")

# Relationships persist
-> system_response("reputation with Guild: 75")
```

### What Resets

```ndl
# Combat state resets
system_response("combat ended")
-> wait("transition")
# Initiative order, temporary buffs, positioning all reset

# Temporary scene effects reset
system_response("leaving cursed room")
-> wait("transition")
# Curse effects no longer apply
```

### Conditional Persistence

```ndl
do($"player", "light torch")
-> system_response("torch lit, 10 minutes remaining")
-> wait("transition to new scene")
-> system_response("torch still burning, 8 minutes remaining")
```

**Pattern**: Ongoing effects that span scenes must be tracked in system state.

## Scene Mode Categories

From [[User-veritasr]]'s scene classifier:

```python
scene_modes = [
    "Default",      # General exploration
    "Social",       # Conversation, negotiation
    "Combat",       # Fighting
    "Exploration",  # Active investigation
    "Travel",       # Moving between locations
    "Delving",      # Dungeon crawling
    "Downtime",     # Resting, crafting
    "Shopping",     # Merchant interaction
    "Resting",      # Sleep, recovery
    "Crafting",     # Item creation
    "Stealth",      # Sneaking, hiding
    "Training",     # Skill improvement
    "Searching",    # Looking for items/clues
    "Researching"   # Learning information
]
```

### Mode-Specific Prompting

```ndl
system_response("entering Stealth mode")
-> wait("mode switch")
# Prompt changes to emphasize:
# - Sound descriptions
# - Guard patrol patterns
# - Cover opportunities
# - Detection risk

do($"player", "sneak") ~ "carefully" -> result("undetected")
```

## Nested Scene Structures

### Subscenes (Rooms within Buildings)

```ndl
# Main scene: Castle
do($"player", "enter") -> target($"great_hall")
-> system_response("subscene: great_hall (parent: castle)")

# Still in castle context, different room
do($"player", "climb stairs") -> target($"throne_room")
-> system_response("subscene: throne_room (parent: castle)")
```

### Scene Hierarchy

From [[User-veritasr]]:
> "functionally it would be something like (world → continents → countries → cities → buildings → rooms and entities)"

```ndl
system_response("current location hierarchy")
# World: Aetheria
#   Continent: Westmarch
#     Country: Kingdom of Valoria
#       City: Capital (Silverpeak)
#         Building: Royal Palace
#           Room: Throne Room

# Transition up hierarchy (room → city)
do($"player", "leave palace")
-> wait("transition: exit building")
-> system_response("scene: Silverpeak (city level)")
```

## Special Transition Cases

### Dream Sequences

```ndl
wait("sleep")
-> system_response("entering dream sequence")
-> describe("dream begins")

# Dream content...
do($"player", "see visions") -> describe("prophetic imagery")

# Wake up
-> wait("dream ends")
-> system_response("returning to reality")
-> describe("waking in original location")
```

### Flashbacks

```ndl
system_response("flashback triggered")
-> wait("memory transition")
-> describe("past event", perspective="memory")

# Flashback content...

-> wait("return to present")
-> system_response("flashback ended")
```

### Astral Projection / Split Consciousness

```ndl
do($"player_body", "meditate")
-> do($"player_spirit", "project") -> target($"distant_location")
-> system_response("consciousness split: body remains, spirit travels")

# Spirit explores...

-> wait("return to body")
-> system_response("consciousness reunited")
```

## Transition Pacing and Rhythm

### Quick Cut (No Description)

```ndl
do($"player", "enter") -> target($"tavern")
# No wait, no describe - instant transition
```

### Montage (Condensed Time)

```ndl
do($"player", "train") ~ "intensively"
-> wait("1 month")
-> describe("training montage")
# Brief descriptions of multiple training scenes
-> system_response("skill increased: Swordsmanship +5")
```

### Lingering Transition (Detailed)

```ndl
do($"player", "walk into forest") ~ "cautiously"
-> describe("forest entrance details")
-> wait("continue deeper")
-> describe("forest deepens, light dims")
-> wait("reach clearing")
-> system_response("scene: forest_clearing")
-> describe("clearing description")
```

## Error Handling and Edge Cases

### Invalid Transition Attempt

```ndl
do($"player", "enter") -> target($"locked_door")
-> result("blocked")
-> system_response("door is locked, transition prevented")
```

### Forced Transition (Cutscene)

```ndl
system_response("cutscene triggered")
-> wait("player cannot act")
-> do($"villain", "capture") -> target($"player")
-> wait("forced transition")
-> system_response("scene: dungeon_cell")
-> describe("captured and imprisoned")
```

### Interrupted Transition

```ndl
do($"player", "walk toward exit")
-> do($"guard", "shout") -> convey("halt!", perspective="commanding")
-> result("transition interrupted")
-> wait("player response")
```

## Common Anti-Patterns

### ❌ Don't: Implicit Scene Changes

```ndl
# WRONG - no transition marker
do($"player", "walk")
# Now what? Are we in a new scene? Still in old scene?
```

### ✅ Do: Explicit Transition Markers

```ndl
# CORRECT - clear transition
do($"player", "walk") -> target($"new_location")
-> wait("transition")
-> system_response("scene changed")
```

### ❌ Don't: Lose Entity Context

```ndl
# WRONG - companion disappears inexplicably
do($"player", "enter new room")
-> wait("transition")
# Where did companion go?
```

### ✅ Do: Handle Entity Persistence

```ndl
# CORRECT - explicitly handle companions
do($"player", "enter new room")
-> system_response("companion $ally follows")
-> wait("transition")
-> describe("arrival with ally")
```

### ❌ Don't: Skip Time Without Consequences

```ndl
# WRONG - time passes but nothing changes
wait("7 days")
# What happened during those 7 days? Status effects? Quests?
```

### ✅ Do: Update State for Time Passage

```ndl
# CORRECT - time has consequences
wait("7 days")
-> system_response("quest 'Rescue' failed: time limit exceeded")
-> system_response("wounds fully healed")
-> system_response("rations consumed: 7")
-> describe("week's passage")
```

## Integration with Save Systems

### Scene as Save Point

```ndl
do($"player", "rest at inn")
-> wait("transition: save point")
-> system_response("progress saved at: Resting at Inn, Day 15")
```

### Checkpoint Transitions

```python
def create_checkpoint(scene_id: str, player_state: dict) -> str:
    """Create save checkpoint at scene transition."""

    ndl = f'wait("checkpoint created")'
    ndl += f' -> system_response("game saved: {scene_id}")'

    # Save player state, inventory, quests, etc.
    save_game_state(scene_id, player_state)

    return ndl
```

## Performance Considerations

### Context Management

Transitions are opportunities to:
- Clear old scene context from LLM memory
- Load only relevant information for new scene
- Reset temporary state
- Trim conversation history

```python
def transition_context(old_scene: str, new_scene: str) -> str:
    """Manage context during scene transition."""

    # Summarize old scene
    summary = summarize_scene_events(old_scene)

    # Clear detailed old context
    # Load new scene context
    new_context = load_scene_context(new_scene)

    ndl = f'wait("transition from {old_scene} to {new_scene}")'
    ndl += f' -> system_response("previous scene: {summary}")'
    ndl += f' -> system_response("current scene: {new_context}")'

    return ndl
```

### Load Time Management

For JIT-generated scenes:

> "For example, I need a character to be the merchant at the stand that I'm currently selling loot at.. Do I wait ~1.5m for it to gen a new character to interact with because I didn't have a merchant avail?" - [[User-veritasr]]

**Solution**: Generate NPCs during transition delays:

```ndl
do($"player", "enter market")
-> wait("loading new scene and NPCs")
# Parallel: Generate merchant NPC during wait time
-> system_response("scene loaded: market")
-> describe("market with merchant")
```

## Testing Scene Transitions

### Validation Checklist

- [ ] Entity positions updated correctly
- [ ] Time advancement reflected in world state
- [ ] Companions/followers persist as expected
- [ ] Inaccessible scenes properly blocked
- [ ] Mode switches trigger appropriate prompt changes
- [ ] Quest states updated for time passage
- [ ] Inventory and equipment maintained
- [ ] Temporary effects cleared appropriately

## Real-World Implementation

From [[User-monkeyrithms]] ReallmCraft:
- Multiple movement commands trigger scene transitions
- Scene hierarchy: locations contain subscenes
- Visual indicators for mode changes (red screen for combat)
- Companions track across scenes
- Death triggers special "Afterlife" scene transition

From [[User-veritasr]] DirectorAPI:
- Parser detects scene boundaries via `wait()` analysis
- Scene transition types: time, place, both
- State checkpointing at scene boundaries

## Related Patterns

- [[wait]] - Core timing and transition construct
- [[scene-based-boundaries]] - Scene management architecture pattern
- [[sequencing]] - Action ordering across scenes
- [[environmental-description]] - Describing new scenes
- [[combat-narration]] - Mode transition to combat

## Further Reading

- [[08-NDL-Natural-Description-Language]] - Full NDL discussion
- [[User-veritasr]] - NDL creator, scene detection implementation
- [[User-monkeyrithms]] - Scene system in ReallmCraft
- [[User-irovos]] - Scene connection philosophy

---

**Pattern Status**: Validated in Production (ReallmCraft, DirectorAPI)
**Model Compatibility**: All models (transitions are state management, not generation-heavy)
**Complexity**: Intermediate
**Use Cases**: Open-world games, exploration, time management, mode switching
