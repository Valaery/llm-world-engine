# NDL Pattern: Environmental Description

#ndl #pattern #environment #exploration #world-building

## Overview

Environmental description in NDL handles the generation of location descriptions, ambient details, and world-building elements. Rather than storing massive pre-written descriptions, NDL provides structured cues that let the LLM generate appropriate environmental prose based on context, ensuring descriptions match the current game state and player actions.

## Core Philosophy

Environmental description in NDL is **lazy** and **context-aware**:
- Generate descriptions only when needed (player enters/observes)
- Base descriptions on current world state (time, weather, previous actions)
- Adapt detail level to player focus (passing through vs. actively exploring)

From [[User-50h100a]]:
> "if it doesn't come up in the interaction, it doesn't exist. if it does come up in the interaction, we *only* care that it doesn't directly contradict something else"

## Basic Environmental Patterns

### Initial Scene Description

```ndl
system_response("scene: tavern, evening")
-> describe("location details")
```

**Generated Narrative**:
> You step into the tavern. The low-ceilinged room is thick with smoke and the smell of ale. A fire crackles in the stone hearth, casting dancing shadows across rough wooden tables. A few patrons huddle over their drinks, conversation low and wary.

### Room Transition with Description

```ndl
do($"player", "enter") -> target($"ancient_library")
-> wait("transition")
-> system_response("scene: ancient_library, dusty, abandoned")
-> describe("first impressions")
```

**Generated Narrative**:
> You push open the heavy oak door. Dust swirls in the beam of light from your lantern. Towering shelves stretch into shadow, filled with crumbling tomes. The air is thick with the scent of old parchment and decay. Silence presses in, broken only by the creak of ancient wood.

### Active Observation

```ndl
do($"player", "look around") -> target($"current_location")
-> describe("detailed observation")
```

**Generated Narrative**:
> You take a moment to survey your surroundings more carefully. The market square is busier than you first noticed—merchants calling out prices, children weaving through the crowd, a street performer juggling near the fountain. The buildings surrounding the square are a mix of timber and stone, their signs swinging in the breeze.

## Search-Driven Description

### Basic Search

From real Gemma 2 testing (Media/image-9648E.png):

```ndl
search("pile of wood", intention="find makeshift club") -> result()
```

**Generated Narrative**:
> Your fingers sift through the rough-hewn logs, searching for something sturdy and manageable. You find it - a thick, gnarled branch, its ends still bearing the marks of the axe. It's not much, but it's better than nothing.

**Pattern**: `search()` triggers focused environmental description of the searched area.

### Search with No Result

```ndl
search("hidden passage", intention="find secret door") -> result("nothing found")
-> describe("unsuccessful search area")
```

**Generated Narrative**:
> You run your hands along the stone wall, pressing at loose bricks, checking for hidden mechanisms. The wall is solid, unremarkable. If there's a secret here, you can't find it.

### Search with Complications

```ndl
search("chest", intention="find treasure") -> result("trapped")
-> describe("trap discovered")
```

**Generated Narrative**:
> You kneel before the ornate chest, examining its elaborate lock. As your fingers probe the mechanism, you notice something—a thin wire, nearly invisible, leading from the lock to a small hole in the chest's side. A needle trap. One wrong move and you'd have been poisoned.

## Context-Aware Description

### Time-Based Variation

```ndl
# Morning
system_response("scene: market, morning")
-> describe("location")
# "Morning light streams across the market square..."

# Evening
system_response("scene: market, evening")
-> describe("location")
# "Lanterns flicker to life across the market square..."

# Night
system_response("scene: market, night")
-> describe("location")
# "The market square lies empty under starlight..."
```

**Pattern**: Time context in `system_response()` guides descriptive tone and details.

### Weather-Influenced Description

```ndl
system_response("scene: courtyard, raining heavily")
-> describe("environment")
```

**Generated Narrative**:
> Rain hammers down on the courtyard's cobblestones, turning every surface slick and treacherous. Water streams from gargoyles' mouths, pooling in low spots. The few people visible hurry past with heads down, cloaks pulled tight.

### State-Dependent Description

```ndl
# Before battle
system_response("scene: throne_room, pristine")
-> describe("location")

# After battle
system_response("scene: throne_room, battle-damaged")
-> describe("location")
# Mentions: overturned furniture, blood stains, broken windows, etc.
```

**Pattern**: Scene state modifiers influence which environmental details the LLM emphasizes.

## Layered Description Depth

### Passing Through (Minimal Description)

```ndl
do($"player", "walk through") -> target($"corridor")
# Brief transition description
```

**Generated Narrative**:
> You pass through a long corridor lit by sputtering torches.

### Casual Observation (Medium Description)

```ndl
do($"player", "enter") -> target($"corridor")
-> describe("notable features")
```

**Generated Narrative**:
> You enter a long corridor. Torches in iron sconces light the way, their flames casting wavering shadows. The walls are bare stone, cold and damp to the touch. Somewhere distant, you hear the drip of water.

### Active Investigation (Detailed Description)

```ndl
do($"player", "examine") ~ "carefully" -> target($"corridor")
-> describe("fine details")
```

**Generated Narrative**:
> You examine the corridor with careful attention. The torches, you notice, are recently lit—the wood isn't charred all the way through. The stone walls bear tool marks, crude but functional. At intervals, you spot small drainage holes near the floor, currently dry but suggesting this passage sometimes floods. The air carries a faint metallic scent, like old blood or rust.

## Dynamic Environmental Elements

### Populated Locations

```ndl
system_response("scene: tavern, crowded, evening")
-> describe("location and occupants")
```

**Generated Narrative**:
> The tavern is packed tonight. Every table is occupied—merchants arguing over prices, off-duty soldiers playing dice, a few hooded figures keeping to shadowed corners. The barkeep works frantically, pulling ales as fast as he can pour them. The noise is a constant hum of conversation, laughter, and clinking mugs.

### Active Events in Environment

```ndl
system_response("scene: market_square")
-> system_response("event: festival in progress")
-> describe("location with event")
```

**Generated Narrative**:
> The market square has been transformed for the harvest festival. Colorful banners stretch between buildings, and temporary stalls line every available space. Musicians play near the fountain, and the smell of roasting meat and sweet pastries fills the air. Everywhere you look, people are celebrating.

### Environmental Hazards

```ndl
system_response("scene: volcano_path, lava flows nearby")
-> describe("dangerous environment")
```

**Generated Narrative**:
> The path clings to the mountainside, a narrow ledge of cooled basalt. To your left, the volcano's flank rises steep and black. To your right, nothing—a sheer drop into a river of molten rock. Heat rises in waves, making the air shimmer. Every breath burns your lungs.

## Location Hierarchies

### Broad to Specific (Zoom In)

```ndl
# Continent level
system_response("location: Westmarch Continent")
-> describe("continental overview")

# Country level
-> system_response("location: Kingdom of Valoria")
-> describe("kingdom characteristics")

# City level
-> system_response("location: Capital city Silverpeak")
-> describe("city description")

# District level
-> system_response("location: Market District")
-> describe("district atmosphere")

# Building level
-> system_response("location: Goldsmith's Shop")
-> describe("shop interior")
```

**Pattern** (from [[User-veritasr]]):
> "functionally it would be something like (world → continents → countries → cities → buildings → rooms and entities)"

### Context-Aware Detail Level

```python
def generate_location_description(location_level: str,
                                 detail_focus: str) -> str:
    """Generate appropriate description based on hierarchy level."""

    if location_level == "world":
        return 'describe("overview: major continents and geography")'

    elif location_level == "continent":
        return 'describe("nations, climate zones, major landmarks")'

    elif location_level == "country":
        return 'describe("culture, major cities, terrain")'

    elif location_level == "city":
        return 'describe("districts, notable buildings, atmosphere")'

    elif location_level == "building":
        return 'describe("interior layout, furnishings, occupants")'

    elif location_level == "room":
        return f'describe("specific details: {detail_focus}")'
```

## Procedural Environment Generation

### Template-Based Description

```ndl
system_response("location type: forest_clearing")
-> system_response("features: ancient_tree, pond, moss")
-> describe("generate from template")
```

**Generated Narrative**:
> You emerge into a clearing. At its center stands an ancient oak, its trunk so wide three people couldn't encircle it with joined hands. A small pond reflects the canopy above, its surface dotted with lily pads. Thick moss carpets every surface, muffling your footsteps.

### Keyword-Driven Generation

```python
def generate_location_ndl(location_type: str,
                         features: list[str],
                         mood: str) -> str:
    """Generate NDL for procedural location description."""

    ndl = f'system_response("scene: {location_type}, {mood}")'
    ndl += f' -> system_response("features: {", ".join(features)}")'
    ndl += ' -> describe("location based on features and mood")'

    return ndl

# Example usage
ndl = generate_location_ndl(
    location_type="abandoned_temple",
    features=["crumbling_pillars", "altar", "faded_murals", "vines"],
    mood="eerie, forgotten"
)
```

**Generated Narrative**:
> Crumbling pillars rise from the jungle floor, half-consumed by vines. Between them, you can make out the shape of an altar, its surface stained dark. Faded murals cover the walls, their subjects long since indecipherable. The air is still, heavy with the weight of abandonment. This place hasn't seen worshippers in centuries.

## Sensory Description

### Multi-Sensory Cues

```ndl
system_response("scene: bakery, morning")
-> system_response("sensory focus: smell, warmth, sound")
-> describe("multi-sensory description")
```

**Generated Narrative**:
> The bakery envelops you in warmth and the rich scent of fresh bread. You hear the rhythmic thump of dough being kneaded in the back room. Flour dust hangs in the air, caught in the morning light streaming through the window. Your mouth waters at the smell of cinnamon rolls just out of the oven.

### Specific Sense Focus

```ndl
do($"player", "listen") ~ "carefully"
-> describe("auditory environment")
```

**Generated Narrative**:
> You focus on the sounds around you. The distant crash of waves against cliffs. Wind whistling through broken windows. Somewhere deeper in the ruins, a rhythmic dripping—water, or something else? And beneath it all, so faint you almost miss it: footsteps.

## Contrast and Change

### Before and After

```ndl
# Before event
system_response("scene: peaceful_village")
-> describe("idyllic setting")

# After event
do($"dragon", "attack") -> target($"village")
-> result("village destroyed")
-> describe("devastation")
```

**Generated Narrative** (after):
> The village you remember is gone. Charred timbers jut from piles of ash. The fountain in the square, once a gathering place, is cracked and dry. Bodies lie in the streets, unburied. Smoke still rises from a few smoldering buildings. The only sound is the wind, carrying the smell of death.

### Transformation Over Time

```ndl
# First visit
system_response("scene: wizard_tower, abandoned")
-> describe("state: neglected")

# After player claims it
wait("1 month of repairs")
-> system_response("scene: wizard_tower, renovated")
-> describe("state: restored")
```

## Environmental Storytelling

### Implied Narrative in Description

```ndl
do($"player", "examine") -> target($"battlefield")
-> describe("environmental story")
```

**Generated Narrative**:
> The battlefield tells its story without words. Two armies met here—you can see where the lines clashed by the density of the dead. Someone tried to flee; there's a trail of bodies leading toward the tree line, cut down from behind. Near a broken standard, the earth is churned and blackened—a spell, devastating and indiscriminate. This was slaughter, not battle.

### Environmental Clues

```ndl
search("crime scene", intention="find evidence")
-> result("clues discovered")
-> describe("forensic details")
```

**Generated Narrative**:
> You examine the room methodically. The struggle started near the door—overturned chair, scuff marks. It moved toward the desk. Blood spatter on the wall suggests the fatal blow was struck here. But something's odd: the window is open, but there's no mud on the sill despite last night's rain. Someone cleaned it. Recently.

## Scale and Perspective

### Intimate Spaces

```ndl
system_response("scene: prison_cell, cramped, dark")
-> describe("claustrophobic environment")
```

**Generated Narrative**:
> The cell is barely large enough to lie down in. Rough stone walls press close on all sides. A thin slit high up admits a weak shaft of light, enough to reveal the filth on the floor but not to provide any real illumination. The air is stale, close, thick with the smell of unwashed bodies and despair.

### Vast Spaces

```ndl
system_response("scene: canyon, massive scale, exposed")
-> describe("awe-inspiring environment")
```

**Generated Narrative**:
> The canyon stretches before you, impossibly vast. Its walls rise hundreds of feet, layered stone in bands of red and ochre. The river at the bottom is a thin silver thread from this height. You feel exposed, insignificant—a tiny figure on the edge of something ancient and uncaring. Wind howls through the gap, carrying the cries of distant birds.

### Transition Between Scales

```ndl
do($"player", "climb to vantage point")
-> describe("perspective shift: close to distant")
```

**Generated Narrative**:
> As you climb, your perspective shifts. The cramped streets you were navigating moments ago now spread out below like a map. You can see the city's layout clearly—the way the river divides it, the districts pressing against the walls, roads radiating from the central plaza like spokes. Smoke rises from a thousand chimneys. From up here, the chaos becomes order.

## Ambient Background Detail

### Minimal Intrusion

```ndl
do($"player", "talk") -> target($"merchant")
-> convey("negotiations")
-> system_response("ambient: market continues around you")
```

**Embedded in Narrative**:
> You haggle with the merchant, eventually settling on a price. Around you, the market continues its business—vendors shouting, customers arguing, the occasional clink of coins changing hands.

### Building Atmosphere

```ndl
wait("observe scene")
-> describe("atmospheric details")
```

**Generated Narrative**:
> You pause, taking in the scene. Fog rolls through the graveyard, wrapping around headstones. An owl calls from a nearby tree. The gate creaks on rusted hinges, moving in a breeze you can't feel. This place sets your teeth on edge.

## Anti-Patterns

### ❌ Don't: Generate Massive Description Walls

```ndl
# WRONG - prompts overly long description
describe("everything in the room in extreme detail")
# Results in: 500-word description block
```

### ✅ Do: Layered, Interaction-Driven Detail

```ndl
# CORRECT - initial overview, detail on demand
describe("room overview")
# Player can then: examine specific objects, search areas, etc.
```

### ❌ Don't: Contradict Previous Descriptions

```ndl
# WRONG - scene already described as empty
system_response("scene: throne_room, empty")
-> describe()
# Then later in same scene:
-> describe("crowds of courtiers")
# Contradiction!
```

### ✅ Do: Maintain Scene State Consistency

```ndl
# CORRECT - update scene state explicitly
system_response("scene: throne_room, empty")
-> describe()
# If change occurs:
-> system_response("event: courtiers arrive")
-> describe("crowds of courtiers entering")
```

### ❌ Don't: Describe Impossible Knowledge

```ndl
# WRONG - player can't see through walls
describe("what's happening in the next room")
```

### ✅ Do: Limit Description to Observable

```ndl
# CORRECT - describe what player can perceive
describe("current room and nearby sounds from adjacent room")
```

## Integration Strategies

### With Procedural Generation

```python
class LocationGenerator:
    """Generate environmental descriptions from templates."""

    def generate_location_ndl(self, template: dict,
                             state: dict) -> str:
        """Create NDL for location description."""

        location_type = template['type']
        features = template.get('features', [])
        mood = state.get('mood', 'neutral')
        time = state.get('time_of_day', 'day')

        ndl = f'system_response("scene: {location_type}, {time}, {mood}")'

        if features:
            ndl += f' -> system_response("features: {", ".join(features)}")'

        ndl += ' -> describe("location")'

        return ndl
```

### With Player Actions

```python
def contextual_description(action: str, target: str,
                          location_state: dict) -> str:
    """Generate description based on player action."""

    if action == "search":
        return f'search("{target}") -> describe("searched area")'

    elif action == "examine":
        return f'do($"player", "examine") -> target("{target}") -> describe("detailed view")'

    elif action == "look around":
        return 'do($"player", "look around") -> describe("full scene")'

    else:
        return 'describe("environmental response to action")'
```

## Testing Environmental Description

### Validation Checklist

- [ ] Initial scene description includes key features
- [ ] Time/weather context affects descriptive language
- [ ] Descriptions don't contradict established facts
- [ ] Detail level appropriate to player focus
- [ ] Environmental changes reflected in descriptions
- [ ] Sensory details beyond just visual
- [ ] Scale conveyed appropriately
- [ ] Mood and atmosphere consistent

### Edge Cases

1. **Empty rooms**: Don't generate false details
   ```ndl
   system_response("scene: empty_cell, bare")
   -> describe("emptiness itself")
   ```

2. **Revisiting locations**: Acknowledge familiarity
   ```ndl
   system_response("location: tavern, previously visited")
   -> describe("familiar place, note any changes")
   ```

3. **Darkness/obscured vision**: Limit description
   ```ndl
   system_response("scene: cave, darkness, no light source")
   -> describe("what can be felt/heard, not seen")
   ```

## Real-World Implementation

From [[User-monkeyrithms]] ReallmCraft:
- Location headers contain base information
- Narrator character has detailed scene knowledge
- Player characters receive less detailed context
- Scene descriptions generated on entry, updated on state change

From [[User-veritasr]]'s generation approach:
- Location type templates with feature lists
- Keyword-driven generation
- Context injection for consistency

## Related Patterns

- [[search]] - Investigation-driven description
- [[scene-transitions]] - Initial location descriptions
- [[do-action]] - Action-prompted environmental responses
- [[describe]] - Core description construct (to be documented)

## Further Reading

- [[08-NDL-Natural-Description-Language]] - Full NDL discussion
- [[04-World-Generation]] - Procedural location generation
- [[User-veritasr]] - NDL creator, location generation system
- [[User-50h100a]] - Environmental existence philosophy

---

**Pattern Status**: Validated in Production (ReallmCraft, DirectorAPI)
**Model Compatibility**: 8B+ models; 13B+ for detailed environmental storytelling
**Complexity**: Beginner to Intermediate
**Use Cases**: Exploration games, dungeon crawlers, open-world RPGs, environmental storytelling
