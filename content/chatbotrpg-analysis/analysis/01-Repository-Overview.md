# ChatBotRPG - Repository Overview

**Repository**: https://github.com/NewbiksCube/ChatBotRPG
**Developer**: [[User-appl2613|appl2613]] (GitHub: NewbiksCube)
**License**: GNU GPL v3.0
**Status**: 80-90% feature complete (July 2025 release)

---

## Core Problem Addressed

ChatBotRPG solves fundamental issues with traditional chatbot roleplay:

### Problems with Traditional Chatbot RP
- **Inconsistent character names** - "Elara" bias in LLMs
- **Forgotten lore** - Environmental details lost over time
- **Hallucinations** - Invalid actions accepted (teleportation, spontaneous abilities)
- **Lack of programmatic control** - No enforcement of game rules

### Solution
Move "past simple roleplay chats with personas" toward **campaign-level games** with:
- ✅ Programmatic state management
- ✅ LLM-only narration (no game logic)
- ✅ Structured world data
- ✅ Rule enforcement

---

## Technology Stack

```yaml
Language: Python (99.9%)
UI Framework: PyQt5 (desktop application)

LLM Services:
  - OpenRouter.ai (primary)
  - Google GenAI (supported)
  - Local models (planned)

Recommended Model: Gemini 2.5 Flash Lite Preview

Data Storage:
  - Initial: JSON files (nested folders)
  - Current: SQLite database
  - Format: .world files (complete game data)
           .save files (individual playthroughs)

Version Control: Git
License: GNU GPL v3.0
```

---

## Architectural Evolution

### Phase 1: JSON-Based (Initial)
```
/data/
  /worlds/
    /fantasy_realm/
      settings.json
      /locations/
        tavern.json
        castle.json
      /npcs/
        bartender.json
        guard.json
```

**Issues**:
- File system clutter
- Difficult to distribute games
- No relational queries
- Version control conflicts

### Phase 2: SQLite-Based (Current)
```
/saves/
  fantasy_realm.world    # Complete game data
  playthrough_001.save   # Individual game session
```

**Benefits**:
- ✅ Single-file distribution
- ✅ Relational data queries
- ✅ Clean file management
- ✅ Portable game files
- ✅ Easier version control

---

## Core Features

### 1. Settings/Worlds
**Interconnected map nodes** with:
- Travel times between locations
- Dynamic descriptions based on time/weather
- Location-specific NPCs and items

```python
# Conceptual structure
location = {
    "id": "golden_oak_inn",
    "name": "Golden Oak Inn",
    "description": "A cozy tavern with warm firelight",
    "connections": {
        "castle": {"travel_time": 30, "method": "walk"},
        "market": {"travel_time": 10, "method": "walk"}
    },
    "npcs": ["bartender", "patron_1"],
    "items": ["rusty_key"]
}
```

### 2. Time Passage
- **Real-time sync**: Game time matches computer clock
- **Simulated time**: Accelerated or manual time control
- **Dynamic events**: Trigger based on time of day

Use cases:
- NPCs follow daily schedules
- Shops open/close
- Day/night cycle events
- Timed quests

### 3. Character Management
**NPC scheduling system** with:
- Movement between locations
- Time-based schedules
- Dynamic presence

```python
# Example NPC schedule
bartender_schedule = {
    "6:00-8:00": "kitchen",      # Preparing breakfast
    "8:00-14:00": "tavern",      # Serving customers
    "14:00-16:00": "market",     # Shopping
    "16:00-22:00": "tavern",     # Evening shift
    "22:00-6:00": "home"         # Sleeping
}
```

### 4. Rules Engine
**Conditional if-then-else logic** executing on:
- **Timers**: Every N turns or at specific times
- **Per-turn**: Check conditions each turn
- **Triggers**: Event-based execution

**Inspiration**: StarCraft map editor triggers

```
IF:
  - Player location == "haunted_mansion"
  - Time == midnight
  - Flag "ghost_defeated" == False
THEN:
  - Spawn NPC "ghost"
  - Set flag "haunting_started" = True
  - Display message "You feel a chill..."
```

### 5. Keywords Matching
**Context injection** based on lore recognition:
- Player mentions "dragon" → Inject dragon lore
- Player enters "ancient ruins" → Inject location history
- NPC name mentioned → Inject character background

**Similar to**: RAG (Retrieval-Augmented Generation)

### 6. Inventory System
**Full RPG item management**:
- Player inventory
- NPC inventories
- Item properties (weight, value, effects)
- Equipment slots
- Trade/give/drop mechanics

### 7. Random Lists
**Weighted randomization** to eliminate algorithmic bias:

```python
# Example random encounter list
encounters = {
    "goblin_patrol": 40,    # 40% chance
    "merchant": 30,          # 30% chance
    "wolf_pack": 20,         # 20% chance
    "dragon": 10             # 10% chance (rare)
}
```

### 8. Scribe (AI Agent)
**Built-in AI agent** for automated content generation:
- Character templates
- Location descriptions
- Rule creation
- Event generation

**Purpose**: Speed up worldbuilding with LLM assistance

---

## User Interface

### PyQt5 Desktop Application

**Components**:
1. **Main Window**: Chat interface for game narration
2. **World Editor**: Visual editing of locations, NPCs, items
3. **Rule Editor**: StarCraft-inspired trigger system
4. **Character Sheet**: Player stats and inventory
5. **Map View**: Visual representation of world connections

### Design Philosophy

**Consequential Decision-Making**:
> "This tool is a game engine, not an experimental chatbot interface"

**Deliberately restricted features**:
- ❌ No copy/paste of chat history
- ❌ No redo/undo for player actions
- ❌ No save scumming (limited save slots)

**Rationale**: Enforce time-pressure and consequences
- Players treat decisions more seriously
- Natural tension in gameplay
- No "optimal path" min-maxing

---

## File Structure

```
ChatBotRPG/
├── main.py                 # Application entry point
├── data/                   # Game data directory
│   ├── worlds/             # World definitions
│   └── templates/          # Content templates
├── saves/                  # Player save files
│   ├── *.world             # Complete game worlds
│   └── *.save              # Individual playthroughs
├── ui/                     # PyQt5 interface components
├── engine/                 # Game logic
│   ├── rules.py            # Rule engine
│   ├── state.py            # State management
│   ├── narrator.py         # LLM integration
│   └── persistence.py      # Database operations
├── llm/                    # LLM service integrations
│   ├── openrouter.py       # OpenRouter.ai client
│   └── google_genai.py     # Google GenAI client
└── utils/                  # Utility functions
```

---

## Distribution Model

### .world Files
**Complete game packages** containing:
- All locations and connections
- NPC definitions and schedules
- Item catalog
- Rules and triggers
- Lore and keywords
- Template definitions

**Distribution**: Share single .world file

### .save Files
**Individual playthroughs**:
- Player state (stats, inventory, location)
- World state changes (NPC positions, flags, quests)
- Time progression
- Rule execution history

**Persistence**: Local to player's machine

---

## OpenRouter.ai Integration

### Why OpenRouter?
- **Multi-model access**: Single API for multiple LLMs
- **Cost-effective**: Competitive pricing
- **Model flexibility**: Easy model switching
- **Fallback support**: Automatic failover

### Recommended Model
**Gemini 2.5 Flash Lite Preview**:
- Fast response times
- Good instruction following
- Cost-effective for narration
- Strong creative writing

### Alternative Models Tested
- **Hathor** (yukidaore): Very creative but requires strong constraints
- **EstopianMaid**: Balanced creativity and following rules

---

## Development Status (July 2025)

### Completed Features ✅
- ✅ Core game loop
- ✅ World editor
- ✅ Rule engine
- ✅ SQLite persistence
- ✅ OpenRouter.ai integration
- ✅ Inventory system
- ✅ NPC scheduling
- ✅ Keyword matching
- ✅ Scribe AI agent

### In Progress 🔄
- 🔄 Local model support
- 🔄 Advanced combat system
- 🔄 Quest journal
- 🔄 Map visualization improvements

### Planned Features 📋
- 📋 Multiplayer (limited scope)
- 📋 Plugin system
- 📋 Community world marketplace
- 📋 Mobile companion app

**Overall Completion**: 80-90%

---

## Key Design Decisions

### 1. Desktop vs. Web
**Choice**: Desktop application (PyQt5)

**Rationale**:
- Direct file system access
- Offline capability
- Visual editor feasibility
- Faster iteration for game designers

**Trade-offs**:
- No multiplayer (initially)
- Installation friction
- Platform-specific builds

### 2. SQLite vs. NoSQL
**Choice**: SQLite

**Rationale**:
- Relational data (locations ↔ NPCs)
- Single-file distribution
- Zero configuration
- SQL query power

### 3. LLM as Narrator Only
**Choice**: Backend makes all game logic decisions

**Rationale**:
- Consistent rules enforcement
- No hallucinations affecting game state
- Predictable behavior
- Cost-effective (fewer LLM calls)

**See**: [[LLM World Engine/patterns/architectural/Program-First-Architecture|Program-First Architecture]]

### 4. Visual Rule Editor
**Choice**: GUI-based trigger system (StarCraft-inspired)

**Rationale**:
- Accessible to non-programmers
- Visual debugging
- Faster iteration
- Game designer-friendly

---

## Performance Characteristics

### Response Times
- **Average narration**: 1-3 seconds (Gemini 2.5 Flash Lite)
- **World generation**: 10-30 seconds (Scribe AI)
- **Rule execution**: < 100ms (local Python)

### Cost Estimates
- **Per narration**: ~$0.0001-0.0003 (170 tokens @ Gemini pricing)
- **Per game session**: ~$0.01-0.03 (100-300 narrations)
- **World generation**: ~$0.005-0.02 (Scribe AI batch generation)

### Token Usage
- **Narration output**: 170 tokens (enforced limit)
- **Context window**: 2,000-4,000 tokens (recent turns + state)
- **World generation**: 2,000-5,000 tokens (Scribe AI)

---

## Cross-References

### Related Patterns
- [[LLM World Engine/patterns/architectural/Program-First-Architecture|Program-First Architecture]]
- [[LLM World Engine/patterns/architectural/Separation-of-Concerns|Separation of Concerns]]
- [[LLM World Engine/patterns/state/Three-Tier-Persistence|Three-Tier-Persistence]]
- [[LLM World Engine/patterns/control/Event-Driven-Design|Event-Driven Design]]

### Related Prompts
- [[LLM World Engine/prompts/narration/NDL-to-Narrative|NDL-to-Narrative]]
- [[LLM World Engine/prompts/constraint/Anti-Hallucination|Anti-Hallucination]]
- [[LLM World Engine/prompts/generation/Character-Generation|Character Generation]]
- [[LLM World Engine/prompts/generation/Location-Generation|Location Generation]]

### Related Discussions
- [[LLM World Engine/topics/01-Architecture-and-Design#Program-First vs LLM-First|Architecture Discussions]]
- [[User-appl2613|appl2613's Contributions]]

---

## Tags

#repository-overview #chatbotrpg #architecture #technology-stack #features #desktop-app #pyqt5 #sqlite #openrouter #game-engine

---

## Next: Pattern Implementation
See [[chatbotrpg-analysis/analysis/02-Pattern-Implementation|02-Pattern-Implementation]] for detailed analysis of architectural patterns used in ChatBotRPG.
