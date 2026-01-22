---
tags: [person, developer, game-designer]
role: "ChatBot RPG Developer"
projects: "[[03-ChatBotRPG-Project]]"
aliases: [monkeyrithms, "Newbiks Cube", appl2613]
status: analyzed
---

# appl2613 (monkeyrithms / Newbiks Cube)

## Overview

**Aliases**: appl2613, monkeyrithms, Newbiks Cube (GitHub)
**Primary Project**: ChatBot RPG
**Technical Focus**: GUI development, rule engines, visual programming
**Key Innovation**: StarCraft-inspired trigger system for LLM games
**Active Period**: January 2024 - July 2025 (throughout transcript)
**Public Release**: July 13, 2025

## Background

Self-identified as "not a coder" early in project, but learned rapidly through experimentation. Brought unique perspective from non-technical background, focusing on user experience and intuitive design.

> [!quote] On Learning
> "as a not-coder, when i see brackets and indentations and stuff im like, 'thats for the devs lol'" (January 2024)

By project end: Proficient in Python, PyQt5, complex state management, and published working game engine on GitHub.

## ChatBot RPG Project

### Core Architecture

**Tech Stack**:
- Python 3.10.11
- PyQt5 for GUI
- JSON for data storage (save files, character sheets)
- TXT files for user-editable content
- Rule engine based on JSON trigger system

**Philosophy**: Visual programming over code
- Character sheets as TXT files (human-readable)
- Rules as JSON (programmatically enforceable)
- Separation of "coder stuff" from "content creator stuff"

### Key Features Developed

**1. Visual Rule Editor**
- Inspired by StarCraft campaign trigger editor
- If-Then-Else logic via JSON
- No coding required for quest design
- Trigger-based event system

**Example Trigger**:
```json
{
  "condition": "player_inventory_contains('ancient_artifact')",
  "actions": [
    "advance_quest_stage(2)",
    "spawn_npc('Old Man', 'relieved')",
    "add_item('gold_reward', 100)"
  ]
}
```

**2. Item System with Effects**
- Consumables with programmatic effects
- Hunger/thirst tracking
- Item-triggered state changes
- Magic effects (time stop, teleportation)

**3. Map/Scene System**
- Dynamic scene switching
- Adjacent location buttons
- Travel commands (`/enter`, `/go`, `/teleport`)
- Scene-specific state management

**4. Quest Staging System**
- Linear progression tracking
- Turn-by-turn pacing
- Stage-based prompt injection
- Prevents one-shot resolution

**5. Character Management**
- Dynamic NPC generation
- Context-aware behavior (at work vs not at work)
- Career/schedule system
- Programmatic personality traits

### Development Timeline

#### Q1 2024 (Jan-Mar): Foundation
- Initial architecture experiments
- Joined LLM World Engine discussions
- Started PyQt5 learning
- Built basic chat interface

#### Q2 2024 (Apr-Jun): Core Systems
- Rule engine implementation
- Item consumption mechanics
- Scene management
- Map editor prototype

**Demo 1 Released**: June 2024
- Basic tavern scene
- NPC interaction (Mara Emberlight)
- Item consumption
- Quest prototype (Old Man's artifact)

#### Late 2024 (Jul-Dec): Refinement
- Addressed small model compatibility
- Improved rule reliability
- Character generation system
- Career/schedule logic
- Extensive testing with Hathor, Mixtral

#### 2025 (Jan-Jul): Polish and Release
- Stability improvements
- Documentation
- GitHub repository setup
- Public release July 13, 2025

## Technical Innovations

### 1. Programmatic Magic Effects

**Time Stop Implementation**:
```python
if magic_effect == "time_freeze":
    for entity in scene.entities:
        entity.frozen = True
        entity.prompt = entity.frozen_prompt

    scene.description = scene.frozen_description
```

**Result**: All NPCs and setting adhere to frozen time programmatically, preventing LLM inconsistency.

**Testing Notes**:
> "setting is programmed to respond when time restarts, so that it gives input on the environment popping back to life"

### 2. Slash Command System

Extensive command synonyms for user convenience:

```python
if re.match(r'^/(enter|go|walk|jog|join|run|scene|transit|
             travel|change|switch|move|teleport|warp|
             fasttravel|skip) ', message):
    change_scene(parse_location(message))
```

> "Im a big fan of multiple things meaning the same thing"

### 3. Context-Aware NPC Prompts

**Career System Example**:
```python
def get_npc_prompt(npc, current_time, current_location):
    if (npc.location == current_location and
        current_time in npc.work_hours):
        return npc.career_prompt
    else:
        return npc.standard_prompt
```

**Application**: Cashier only acts as cashier when at work during work hours.

### 4. Turn Counting for Pacing

**Problem**: LLMs one-shot everything

**Solution**:
```python
quest_stages = {
    1: "Introduce problem vaguely",
    2: "Reveal stakes",
    3: "Provide solution path",
    4: "Allow player action"
}

current_stage = get_quest_stage(quest_id)
inject_stage_prompt(current_stage)
```

**Result**: Story develops gradually over multiple turns.

## Design Philosophy

### Programmatic Control Over LLM

> "this 'magic effect' is scripted and programmatically controlled, so the LLM and all agents in the scene adhere to the world being frozen"

**Observation**:
> "if this is not programmatically controlled... various agents will still describe things moving or ignore the effect entirely while others adhere to it, and occasionally it will write that time starts again, randomly"

### Separation of Data Types

**For Non-Coders** (TXT files):
```
NAME
Bob
DESCRIPTION
Bob is a happy guy.
APPEARANCE
Bob wears jeans and a t-shirt.
INVENTORY
apple, water
```

**For System** (JSON files):
```json
{
  "gameHour": 14,
  "location": "Golden Oak Inn",
  "questStage": 2,
  "frozen": false
}
```

**Reasoning**: "I expect them to want to play with and add char sheets a lot. That's like, a whole 'thing'"

### Multiple Inference Passes

**Concept**: Don't one-shot complex decisions

> "the brain has multiple regions that calculate for different things... we as human beings don't always one-shot our answers to things, sometimes there is a quick deliberation or setup in the brain first"

**Application**:
1. Parse player input (inference 1)
2. Check rule violations (inference 2)
3. Generate NPC response (inference 3)
4. Update quest stage if needed (inference 4)

**Trade-off**: Slower with local models, but more reliable.

### Obsessive Detail Orientation

**Examples**:
- 20+ synonyms for scene transition commands
- Extensive testing across models
- Detailed error logging
- Clear feedback messages

> "call me obsessive" (on extensive command synonyms)

## Technical Challenges Overcome

### 1. PyQt5 GUI Complexity

**Issue** (February 2024):
> "once again, GUI stuff becomes my achilles heel. it is such a twisted puzzle trying to get it to simply update the buttons when the scene changes"

**Resolution**: Eventually figured out signal/slot patterns after ~20 hours debugging

### 2. Model Compatibility

**Problem**: What works on Mixtral fails on Hathor

**Testing from [[User-yukidaore]]**:
- Hathor: Too creative, generated diamond horses and teleportation
- Mixtral: Followed rules reliably
- Small 7B models: Needed more explicit constraints

**Solution**: Model-specific prompt adjustments, fallback to stricter models for critical logic

### 3. Scene State Persistence

**Issue**: Scene resets failed to restore inventory

**Root Cause**: Missing backup files in demo distribution

**Learning**: Need "Original" template files for proper resets

### 4. Item Tracking Bugs

**Observed**: Items duplicated, disappeared, or transferred incorrectly

**Fix**: Explicit inventory management with validation checks

### 5. Character Naming Conventions

**Problem**: LLM falls back on cliches

> "Here is an example of 'unhinged' naming where the LLM is allowed to fall back on its own cliches... Each of these is a one-shot name given to an 'archer' character... you can quickly see the problem, it's almost cheeky with how bad it is"

**Solution**: Constrained name generation with cultural/regional templates

## Interaction Style

### Collaborative and Supportive
- Actively tested others' projects
- Provided detailed bug reports
- Shared learnings from failures
- Asked clarifying questions

### Experimental and Iterative
- Willing to try new approaches
- Learned from mistakes
- Documented findings
- Iterated based on feedback

### User-Focused Design
- Prioritized accessibility
- Clear documentation
- Intuitive commands
- Forgiving error handling

## Notable Quotes

### On Development Process
> "well like all good wine, I gotta make sure it doesn't have a lot of last minute bugs"

### On Model Behavior
> "LLMs don't really have much concept of 'time' -- each time they're prompted... to their perspective, they're posting the very first time"

### On Learning to Code
> "So today is the day... my first major public project released on Github which I learned how to use like yesterday"

### On Project Scope
> "I started getting kind of bored with all the 'dummy' NPCs and locations... I added in some more fun in-game world stuff... and having actual world stuff to play with made the project more fun"

### On Multi-Step Inference
> "LLMs are super cliche and shallow on their own devices... but so would we if we just one-shot everything"

### On Community
> "its definitely not [a competition], lol, feels more like were all a team in here"

## Testing and QA Process

### Self-Testing Checklist
- Lorebook Keywords ✅
- Character Sheets ✅
- Item Consumption / Hunger / Thirst ✅
- Map Editor ✅
- In-Game Map Viewer ❌ (Borked)
- Activate/Read Readable Items ✅
- Switch Scenes 🐛 (Works but buggy)
- Quest + Stages ✅

### Community Testing
- [[User-yukidaore]]: Extensive Hathor testing, found sequence breaking
- [[User-hermokratesthelate]]: Windows compatibility testing
- Multiple iterations based on feedback

## Visual Documentation (May 2025 Reveal)

### First Public Screenshots

**appl2613** [May 26, 2025]: Shared comprehensive screenshots showcasing all major features:

#### Main Game Interface
![[Media/ChatbotRPG1-B43DA.jpg]]

**Features shown**:
- Retro CRT green-on-black terminal aesthetic (customizable color)
- Scene tabs for quick navigation (New RPG, Goldsprings, Combat Arena, Post-Apocalyptic)
- Turn counter display (upper right)
- "TEST" ASCII art branding
- Save/Load functionality
- Clear separation of player input and AI responses

**Design Notes**:
> "the theme, yes I know its all one color. Its supposed to have an old crt monitor theme... paying respects to the original text-adventures like Zork and that whole retro era"

#### NPC Interaction System
![[Media/ChatbotRPG2-203E4.jpg]]

**Features demonstrated**:
- Natural conversation flow with Jane Doe NPC
- Party mechanics: "Follow" command integration
- Scene transitions between House and other locations
- Context-aware dialogue showing memory persistence
- Multi-character interactions in same scene

#### Visual Rule Editor
![[Media/ChatbotRPG3-F77A8.jpg]]

**Core innovation on display**:
- Chat Triggers vs Timer Triggers tabs
- Condition builder with variable types (Variable, following, ischatting)
- Scope selectors: Global, Player, Character, Scene, Setting
- "Update ?", "Duplicate ?", "Clear/New ?", "Delete ?" buttons for rule management
- Model selection dropdown
- Full JSON integration without requiring code knowledge

**Philosophical statement**:
> "trigger-based 'if this, then that' system. Provides an entire RPG's worth of flexibility without having to write code"

#### Map Editor & Scene System
![[Media/ChatbotRPG4-423EF.jpg]]

**Navigation architecture revealed**:
- Visual map showing Goldsprings, Combat Arena, and Post-Apocalyptic zones
- Scene relationship diagram
- Route connections between locations
- "New RPG" starting location
- Integrated map viewer showing player position

**Technical approach**:
> "can define your whole game world and how all that stuff links together in a working map editor... the game will use natural language to determine if you're headed towards any of the exits in the current scene"

#### Character Creation Interface
![[Media/ChatbotRPG5-64BE7.jpg]]

**LLM-assisted content generation**:
- Complete character sheet for Jane Doe
- Sections: Name, Description, Personality, Appearance, Goals, Equipment, Abilities, Story
- "Generate Description", "Generate Personality", "Generate Appearance" buttons
- "Generate Goals", "Generate Equipment", "Generate Abilities" buttons
- "Generate Story" for background narrative
- Full integration between manual editing and AI generation

**JSON-based architecture**:
> "all the building-blocks of the game essentially--are in json format, the LLM will soon become involved in the process of writing these rules, auto-generating characters, etc."

### Design Philosophy (Explained)

**On theme consistency**:
> "Its supposed to have an old crt monitor theme (you can change the color). Reasons span from it being relatively easy for me to extend as Im far more focused on the game than the theme at the moment, to paying respects to the original text-adventures like Zork"

**On data architecture**:
> "not using nested folders and .json files for everything anymore either. no more mess on the computer. Everything is neatly tucked away in SQLite files. An entire 'game' comes in a .world file, and saves for current playthroughs of a game are .save files"

**On UX philosophy**:
> "a rapid series of shorter messages, but a more responsive world, and characters who talk in short lines or in a conversational manner that feels very realtime... its so much more real-feeling when the architecture is there for it"
>
> "things feel so much more alive when the length and fluidity of posts and their turn-orders can fluctuate depending on whether you're in a bar, a liminal space, or a boss battle"

## GitHub Release (July 13, 2025)

**Repository**: https://github.com/NewbiksCube/ChatBotRPG

**Initial Release Contents**:
- Complete engine
- Small demo scene (Golden Oak Inn)
- Character generation
- Rule engine
- Map editor
- Installation instructions

**Release Statement**:
> "this is the first release of the (mostly finished) ChatBot RPG engine, on which I intend to build some cool things on top of, and at least one epic fantasy world"

**Known Issues at Release**:
- Missing some features from established AI RP platforms (character portraits, backgrounds)
- Requires OpenRouter initially (planned to expand)
- Relatively small demo scene

**Vision**:
> "I think it'd be awesome to collab with others in the worldbuilding process on top of it and multiple people contribute to the development of the same game world"

## Architectural Comparisons

### vs ReallmCraft
- **ChatBotRPG**: GUI-first, visual rule editor
- **ReallmCraft**: Backend-first, Flask + React

### Shared Principles
- LLM as narration layer only
- Programmatic state management
- Template-based content
- Modular/extensible architecture

### Unique to ChatBotRPG
- Visual trigger system
- PyQt5 desktop application
- User-friendly TXT file editing
- StarCraft-inspired design

## Influence on Community

- Demonstrated non-programmer can build complex system
- Proved visual programming approach viable
- Showed importance of pacing (turn counting)
- Validated rule-based triggers for LLM games
- First to publicly release working engine

## Lessons Shared

### On Item Systems
> "you do want consumables to have the capability of imparting 'something happens' to the player... it would be nice if the Thing the potion is supposed to do was the same between scene A and scene B"

### On Model Selection
> "gpt-4 was very good, its the only model that can kinda-sorta one-shot a good RP with all the rules... a big model can handle a huge one-shot validation question with tons of detail"

### On Development Motivation
> "I need to do some worldbuilding for myself as I go or I will get tired of just code all day"

### On Upscaling vs One-Shotting
> "what I'm really getting at here is the concept of upscaling vs. one-shotting high resolution... If the answer is 'very important', upscale your posts instead of one-shotting them"

## Related Pages

- [[03-ChatBotRPG-Project]] - Full project documentation
- [[01-Architecture-and-Design]] - Architectural contributions
- [[02-Prompt-Engineering]] - Pacing and turn-counting techniques
- [[User-veritasr]] - Parallel engine developer
- [[User-yukidaore]] - Primary tester

## Timeline of Major Milestones

### January 2024
- Joined Discord channel
- Began learning Python and PyQt5
- Started project planning

### February-April 2024
- Built core chat interface
- Implemented item system
- Developed rule engine prototype

### May-June 2024
- First public demo
- Received community feedback
- Iterated on stability

### July-December 2024
- Advanced features (magic, careers, quests)
- Extensive model testing
- Refined rule engine

### January-July 2025
- Final stability pass
- Documentation
- Public GitHub release

---

> [!note] Impact Assessment
> appl2613 demonstrated that complex LLM game engines are accessible to non-programmers willing to learn. The visual rule editor and StarCraft-inspired trigger system represent a unique approach distinct from backend-first architectures. The public release provides a working reference implementation for the community.

> [!success] Achievement Unlocked
> First member of the community to publicly release a complete, working LLM game engine on GitHub (July 13, 2025).
