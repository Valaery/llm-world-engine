# Prompt: Location Generation

#prompt #generation #world-building #template

## Metadata
- **Category**: Generation
- **Technique**: Template-based with weighted tables
- **Model Tested**: Various (GPT-4, Claude, local models)
- **Contributor**: Community discussions, [[User-monkeyrithms]]
- **Status**: Proven

## Purpose
Generate game locations (taverns, dungeons, towns, rooms) with consistent structure and modding-friendly output. Uses template system with weighted random tables to ensure variety while maintaining quality and game integration.

## Template
```
Generate a {{location_type}} with the following specifications:

Basic Information:
- Name: {{name_style}}
- Parent Location: {{parent_location}}
- Size: {{size}}
- Purpose: {{primary_purpose}}

Atmosphere and Style:
- Ambiance: {{ambiance}}
- Wealth Level: {{wealth_level}}
- Reputation: {{reputation}}
- Distinctive Feature: {{distinctive_feature}}

Functional Details:
- Security Level: {{security_level}}
- Opening Hours: {{operating_hours}}
- Common Patrons: {{patron_types}}
- Common Events: {{typical_events}}
- Services Available: {{services}}

Generate a structured description including:
1. **Name**: Creative but appropriate for {{genre}} setting
2. **Physical Description**: Architecture, layout, notable features (2-3 sentences)
3. **Atmosphere**: Sights, sounds, smells, general feeling (2-3 sentences)
4. **Function**: What happens here, who comes here, what services (1-2 sentences)
5. **Notable Features**: 3-5 specific details that make it memorable
6. **Hooks**: 2-3 potential quest hooks or interesting events that could occur here

Format as structured data:
\`\`\`json
{
  "name": "...",
  "type": "...",
  "description": {
    "physical": "...",
    "atmosphere": "...",
    "function": "..."
  },
  "features": ["...", "...", "..."],
  "hooks": ["...", "...", "..."],
  "stats": {
    "size": "...",
    "wealth": "...",
    "security": "...",
    "reputation": "..."
  }
}
\`\`\`
```

## Variables
| Variable | Description | Example |
|----------|-------------|---------|
| {{location_type}} | What kind of location | "tavern", "dungeon level", "shop", "city district" |
| {{name_style}} | Naming convention | "alliterative", "descriptive", "fantasy names", "themed" |
| {{parent_location}} | Where it exists | "merchant district", "haunted forest", "underground complex" |
| {{size}} | Scale | "cramped", "cozy", "spacious", "vast" |
| {{primary_purpose}} | Main function | "drinking establishment", "trading post", "monster lair" |
| {{ambiance}} | Mood | "warm and welcoming", "tense and watchful", "eerie and abandoned" |
| {{wealth_level}} | Economic status | "destitute", "modest", "prosperous", "opulent" |
| {{reputation}} | How it's known | "respectable", "notorious", "secretive", "renowned" |
| {{distinctive_feature}} | Unique element | "always has a fire burning", "covered in strange symbols" |
| {{security_level}} | Safety | "unguarded", "casual watch", "heavily protected" |
| {{operating_hours}} | When accessible | "dawn to dusk", "24 hours", "midnight to dawn only" |
| {{patron_types}} | Who visits | "merchants", "adventurers", "criminals", "scholars" |
| {{typical_events}} | What happens | "business deals", "gambling", "fights", "performances" |
| {{services}} | What's offered | "food and drink", "lodging", "equipment repair", "information" |
| {{genre}} | Setting style | "medieval fantasy", "dark fantasy", "steampunk", "sci-fi" |

## Usage Example

### Example 1: Tavern
```
Generate a tavern with the following specifications:

Basic Information:
- Name: Descriptive with occupational theme
- Parent Location: Merchant district of a medium-sized trading city
- Size: Spacious
- Purpose: Social gathering, business deals, evening meals

Atmosphere and Style:
- Ambiance: Warm and lively with professional atmosphere
- Wealth Level: Prosperous
- Reputation: Respectable, known for fair dealing
- Distinctive Feature: Collection of exotic items from distant lands

Functional Details:
- Security Level: Casual watch, occasional city guard presence
- Opening Hours: Noon to midnight
- Common Patrons: Traveling merchants, local business owners, off-duty guards
- Common Events: Business negotiations, storytelling, occasional performances
- Services Available: Quality food and drink, lodging, storage for goods, message service

Generate a structured description including:
1. **Name**: Creative but appropriate for medieval fantasy setting
2. **Physical Description**: Architecture, layout, notable features (2-3 sentences)
3. **Atmosphere**: Sights, sounds, smells, general feeling (2-3 sentences)
4. **Function**: What happens here, who comes here, what services (1-2 sentences)
5. **Notable Features**: 3-5 specific details that make it memorable
6. **Hooks**: 2-3 potential quest hooks or interesting events that could occur here

Format as structured data.
```

**Expected Output**:
```json
{
  "name": "The Trader's Rest",
  "type": "tavern",
  "description": {
    "physical": "A three-story timber and stone building with broad windows overlooking the market square. The ground floor contains a spacious common room with a long oak bar, while sturdy tables and booths provide private spaces for negotiations. Display cases along the walls hold curiosities from distant lands—spices, silks, carved figurines, and nautical instruments.",
    "atmosphere": "The air is thick with the scents of roasted meat, mulled wine, and exotic spices. Conversations range from hushed business dealings to animated storytelling, creating a steady murmur punctuated by the clink of coins and tankards. Warm light from well-maintained lanterns and a central fireplace gives the space a welcoming but professional ambiance.",
    "function": "The Trader's Rest serves as both tavern and informal trade house, where merchants seal deals over quality meals and ale. The establishment offers clean lodging on the upper floors and secure storage for goods in the cellar, making it popular with traders who value discretion and security."
  },
  "features": [
    "Display cases of exotic goods serve as conversation pieces and subtle advertising",
    "Private booths with curtains for confidential negotiations",
    "Message board where merchants post buying/selling notices",
    "Cellar with locked strongboxes for rent",
    "Innkeeper keeps ledger of merchant reputations (quietly)"
  ],
  "hooks": [
    "A merchant who stayed here three nights ago left behind a locked case - hasn't returned to claim it",
    "Two rival trading companies both want to hire the same warehouse - tension is building",
    "The exotic items in the display case: one of them is stolen from a noble's collection"
  ],
  "stats": {
    "size": "spacious",
    "wealth": "prosperous",
    "security": "casual watch",
    "reputation": "respectable"
  }
}
```

### Example 2: Dungeon Room
```
Generate a dungeon level with the following specifications:

Basic Information:
- Name: Descriptive based on function
- Parent Location: Ancient temple complex, second level
- Size: Medium chamber with side passages
- Purpose: Tomb of temple guardians

Atmosphere and Style:
- Ambiance: Eerie and oppressive
- Wealth Level: Once prosperous, now looted
- Reputation: Dangerous, tales of the undead
- Distinctive Feature: Stone statues that seem to watch visitors

Functional Details:
- Security Level: Trapped and guarded by undead
- Opening Hours: Always accessible (but dangerous)
- Common Patrons: None (abandoned), occasionally tomb robbers
- Common Events: Undead patrols, trap triggers, treasure hunting
- Services Available: None

Generate a structured description including dungeon features, hazards, and potential treasures.

Format as structured data.
```

**Expected Output**:
```json
{
  "name": "Chamber of the Silent Sentinels",
  "type": "dungeon_room",
  "description": {
    "physical": "A rectangular stone chamber, 40 feet by 30 feet, with a vaulted ceiling 20 feet high. Six alcoves line the walls, each containing a stone statue of an armored warrior. The floor is set with geometric tiles, some cracked with age. Three passages exit: north, east, and a collapsed southern tunnel.",
    "atmosphere": "The air is stale and cold, carrying a faint scent of decay. The statues' empty eyes seem to follow movement through the room. Distant sounds—dripping water, settling stone—echo from elsewhere in the complex. Scratch marks on the floor suggest recent activity.",
    "function": "This chamber once served as a guard post for the temple's inner sanctum. The statues honor the guardians who served in life and were buried in the alcoves behind them. Looters have pried open three alcoves, but three remain sealed."
  },
  "features": [
    "Six statues: three alcoves behind them are open (empty), three are sealed (stone slabs with carved prayers)",
    "Geometric floor tiles: some are pressure plates triggering dart traps in the walls",
    "Scorch marks on the east passage wall (previous adventurers triggered something)",
    "Collapsed south tunnel: recent, rubble suggests deliberate cave-in",
    "Faint phosphorescent moss on ceiling near north passage (air flow from deeper levels)"
  ],
  "hooks": [
    "The sealed alcoves: one contains treasure, one contains a powerful undead guardian, one contains clues to temple history",
    "The recent cave-in: someone doesn't want people going that direction - why?",
    "Scratch marks and fresh blood on floor: something dragged a body through here within the last day"
  ],
  "stats": {
    "size": "medium",
    "wealth": "partially looted",
    "security": "trapped and patrolled",
    "reputation": "dangerous"
  },
  "mechanics": {
    "traps": "Pressure plate tiles (4 locations): trigger dart traps from walls (Dexterity save DC 14)",
    "enemies": "2 Skeleton Warriors patrol every 20 minutes",
    "treasure": "Sealed alcove #2 contains warrior's sword (+1 longsword) and 150gp in ancient coins"
  }
}
```

## Effectiveness Notes

### What Works Well
- **Structured JSON output**: Easy to parse and use in games
- **Template consistency**: Locations have predictable structure
- **Hook generation**: Built-in quest potential for each location
- **Modding-friendly**: All parameters can be adjusted by game designers
- **Separation of description and mechanics**: Narration separate from stats
- **Weighted tables approach**: Backend can use weighted random selection for parameters

### Known Limitations
- Generic prompting produces generic locations (need specific parameters)
- Quality varies significantly by model
- May need multiple generation passes for complex locations
- Hook quality can be hit-or-miss, often needs curation
- Feature lists can become formulaic without variation
- Small models (< 13B) struggle with creative, coherent hooks

### Model-Specific Notes
- **GPT-4**: Excellent at creative but appropriate names and hooks
- **Claude**: Strong at atmospheric descriptions and logical layout
- **Gemini**: Fast, good structure, sometimes generic descriptions
- **Local 70B models**: Serviceable for basic locations
- **Local 7-13B models**: Best with very specific templates, struggle with hooks

### Iterative Refinement Approach
From [[User-monkeyrithms]] and community discussion:

1. **Generate high-level** (city/region)
2. **Generate districts/areas** within that region
3. **Generate specific locations** within districts
4. **Generate NPCs** for those locations
5. **Generate relationships** between NPCs

Each step uses previous outputs as context, building coherence.

## Variations

### Quick Location (Speed Mode)
For rapid generation during gameplay:
```
Generate a {{location_type}} in {{parent_location}}.
Include:
- Name
- 1 sentence description
- 1 distinctive feature
- 1 potential hook

Format: JSON
```

### NPC-Focused Location
When the people matter more than the place:
```
Generate a {{location_type}} designed around these NPCs:
{{npc_list}}

The location should:
- Reflect their personalities and relationships
- Provide space for their activities
- Create opportunities for player interaction

Include description and hooks based on NPCs.
```

### Conditional Location Generation
From [[User-monkeyrithms]]'s work system:
```
if at_workplace() and during_work_hours():
    Generate location description including:
    - {{npc_name}} at work
    - Their professional activities
    - Work-appropriate interaction opportunities
else:
    Generate location description for off-hours:
    - Different atmosphere
    - Leisure activities
    - More personal interaction
```

### Procedural Template System
Backend fills template variables from tables:
```json
{
  "tavern_names": ["The {{adjective}} {{noun}}", "{{person}}'s {{establishment}}"],
  "adjectives": ["Rusty", "Golden", "Prancing", "Sleeping"],
  "nouns": ["Tankard", "Dragon", "Crown", "Anchor"],
  "distinctive_features_table": [
    {"feature": "always has a fire burning", "weight": 5},
    {"feature": "covered in ship figureheads", "weight": 2},
    {"feature": "built around a massive tree", "weight": 1}
  ]
}
```

Backend randomly selects from weighted tables, LLM expands into full description.

## Related Prompts
- [[scene-description]] - Narrating locations once generated
- [[character-creation]] - Populating locations with NPCs
- [[item-generation]] - Filling locations with objects
- [[quest-generation]] - Using location hooks as quests
- [[template-harvester]] - Extracting reusable templates

## Source
**Discussion context**: Location generation emerged from world-building discussions throughout the transcript. The template-based approach prioritizes modding and data-driven design.

Key insights:

**Iterative refinement** from community consensus:
> "Generate high-level world description, then specific region within world, then locations of interest in region, then characters for each location, then relationships between characters. Each step informs the next, building coherence."

**Conditional generation** from [[User-monkeyrithms]]:
> "NPCs at work: if at_workplace() and during_work_hours(), inject_career_description(), else use_standard_description()"

**Template philosophy**: Locations should be modding-friendly with clear structure for non-programmers to adjust. JSON output enables easy editing and procedural generation.

The hook system (2-3 per location) ensures every location has quest potential, following the principle that "every location should give players something to do."

**Related threads**: [[04-World-Generation]], [[02-Prompt-Engineering]]

## Best Practices

### DO:
- ✅ Use structured output (JSON/XML) for game integration
- ✅ Include hooks for quest potential
- ✅ Separate description from mechanics
- ✅ Make templates modding-friendly
- ✅ Generate iteratively (broad to specific)
- ✅ Provide specific parameters for unique locations
- ✅ Include sensory details (not just visual)

### DON'T:
- ❌ Generate everything at once (causes inconsistency)
- ❌ Let LLM decide mechanical stats (backend's job)
- ❌ Skip the distinctive feature (locations blur together)
- ❌ Forget hooks (reduces gameplay value)
- ❌ Make locations too generic (need specific parameters)
- ❌ Mix narration and stats in same output
- ❌ Rely on small models for creative, coherent generation
