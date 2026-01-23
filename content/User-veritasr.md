---
tags: [person, developer, architect]
role: Primary Architect & Developer
projects: [[02-ReallmCraft-Project]]
status: analyzed
---

# veritasr

## Overview

**Primary Contributor**: Most active participant in the channel
**Primary Project**: ReallmCraft (LLM World Engine)
**Key Innovation**: Natural Description Language (NDL)
**Technical Focus**: Backend architecture, RAG systems, constraint programming
**Active Period**: January 2024 - July 2025 (throughout entire transcript)

## Background & Approach

veritasr brought significant software engineering experience to the project, often referencing day-job practices:
- Forced to provide 3 separate solutions during design phase at work
- Strong emphasis on modularity and separation of concerns
- Preference for programmatic control over LLM decision-making

> [!quote] Philosophy
> "Turns out that when you take away decision making from the LLM it behaves much better."

## Major Contributions

### 1. ReallmCraft Engine

**Architecture**: Flask-based backend with React frontend
**Key Features**:
- Modular world generation
- Template-based entity system
- RAG retrieval for context management
- Custom scripting language for game logic
- Mod framework support

**Development Timeline**:
- **Jan 2024**: Initial architecture discussions
- **Feb 2024**: Core engine development, world management
- **Feb-Jun 2024**: Feature expansion, UI development
- **Jun 2024+**: NDL development, constraint programming exploration
- **Jul 2025**: Advanced prompting workflows

**Code Scale**: ~7,500+ lines reported (July 2024)

### 2. Natural Description Language (NDL)

**Purpose**: Convert programmatic game events into natural narrative

**Core Concept**:
```
Backend Events/State → NDL Markup → LLM Processing → Natural Text
```

**Philosophy**:
> "Essentially what I'm doing is that I'm simulating events and state changes on the backend. Those events get passed into a process that converts the events to a markup language, effectively building a prompt, which is then passed to the LLM as a decorator."

**Key Insight**: LLM functions as translator, not decision-maker

**Results Demonstrated** (July 2025):
- Consistent subtext in narrative
- Controlled emotional beats
- Reliable dialogue with implicit intent
- Works across multiple models (Gemma, Llama3, Mistral, Stheno)

### 3. RAG & Memory Systems

**Approach**: Hybrid retrieval combining multiple methods

**Implementation**:
- Exact name matching for direct references
- Tag-based boolean search
- Semantic similarity for descriptions
- Ranking and fusion (RRF-style)
- Context window management with token estimation

**Key Innovation**: Treating semantic search as paragraph comparison, not keyword matching
> "Semantic similarity isn't meant to locate stuff like keywords or key phrases, it's meant to find similar paragraphs of text."

**Solution**: Store examples of experiences rather than

 descriptions

### 4. Constraint Programming Integration

**Discovery** (July 2024): Realized probability/statistics might be wrong approach

**New Paradigm**:
- Scoring expands, probability contracts
- Constraint-based reasoning (deduction)
- Rule engines (induction)
- Utility systems (weighing options)

**Four Component Types**:
1. **Generators** - Create and cache options/choices
2. **Assemblers** - Compose elements from choices, build objects/state
3. **Evaluators** - Parse conditions, score, make choices
4. **Analyzers** - Abstract data, derive metrics from existing data

### 5. Advanced Prompting Techniques

**Late-Stage Development** (July 2025):

**Dialogue System**:
```
Character beliefs → Never state outright
Rules about what can/cannot be said
Information that comes to mind
What they wish to convey vs. what they do convey
```

**Results**: Achieved natural subtext and implicit character motivation without explicit dialogue scripting

**Narrative Generation**:
- Event-based narration
- Subtext through omission and implication
- Controlled emotional beats
- Works with smaller models (7B-9B)

## Technical Expertise

### Backend Development
- Python (Flask, FastAPI patterns)
- PostgreSQL/SQLite
- REST API design
- Background task processing

### Frontend Development
- React/Next.js
- Considered Tauri for desktop deployment
- UI/UX for complex systems

### AI/ML
- RAG architectures
- Vector databases (ChromaDB, Qdrant)
- Prompt engineering
- Multi-model support (Ooba, Kobold, OpenAI, OpenRouter, Mancer)

### System Design
- Modular architecture
- Template systems
- Constraint programming
- Rule engines
- Event-driven systems

## Development Philosophy

### Core Principles

1. **Programmatic Control Over LLM Decision-Making**
   - State changes must be deterministic
   - LLM for narration only
   - Separate logic from presentation

2. **Data-Driven Design**
   - Everything configurable via templates
   - JSON-based entity definitions
   - Modding support from ground up

3. **Just-In-Time Generation**
   - Don't create everything upfront
   - Generate as needed
   - Fix values once created

4. **Modularity & Separation of Concerns**
   - Backend vs. Frontend separation
   - Logic vs. Narrative layers
   - Core vs. Mod systems

5. **Iterative Development**
   - "I'm like on iteration 12 for this thing"
   - Willing to throw out and rebuild
   - Test-driven refinement

## Interaction Style

### Collaborative
- Shares detailed technical implementations
- Provides code examples freely
- Debugs others' issues
- Answers questions comprehensively

### Experimental
- Constantly trying new approaches
- Documents findings
- Pivots when necessary
- "Think tank" mentality

### Pragmatic
- "Better result would be something like: 'I'm not sure, but this is the direction I was thinking...'"
- Focuses on what works over theoretical purity
- Balances ideal vs. practical

### Self-Aware
- Acknowledges when taking over thread
- Apologizes for walls of text
- Transparent about iteration count
- Admits limitations

## Notable Quotes

### On LLM Limitations
> "LLMs are terrible at state management - Keep state programmatic"

### On Architecture
> "Essentially half simulation half story"

### On Development
> "A lot of these ideas make it to partial implementations before throwing them out"

### On Sharing Work
> "I don't typically share projects, since that means people want me to support them. I've been burned by that in the past."

### On LLM Role
> "Sort of like it just narrating what happens in the world rather than the LLM trying to continue the story"

### On Innovation
> "It sort of feels like the industry is catching up to what we were doing in here _last_ year"

## Timeline of Major Developments

### Early 2024 (Jan-Mar)
- Joined thread, began ReallmCraft development
- Established core architecture principles
- Built world generation system
- Implemented RAG retrieval

### Mid 2024 (Apr-Jun)
- Frontend development (React)
- Micromamba integration for cross-platform support
- Workflow systems
- Random table generation
- Extensive playtesting (Zweihander-inspired world)

### Late 2024 (Jul-Dec)
- Character generation systems
- Behavior systems
- NDL initial development
- Constraint programming exploration
- Modding framework

### 2025 (Jan-Jul)
- NDL refinement
- Advanced prompting workflows
- Selection-based UI (reducing typing fatigue)
- Staged/contextual menus (FSM-based)
- Subtext and implicit dialogue systems

## Project Status

**ReallmCraft**:
- Not publicly released on GitHub (as of analysis)
- Went through ~12 iterations
- Current iteration: Modernized expert system with LLM as decorator
- "Very WIP... like 2 weeks in or something like that" (re: latest iteration)

**NDL**:
- Functional and tested across multiple models
- Achieves reliable narrative generation
- Successfully handles subtext and character motivation
- Works with small models (7B-9B parameters)

## Lessons Shared

1. **On Context Management**:
   - Token estimation with padding (1/50 ratio)
   - Priority-based context inclusion
   - Exact matches first, semantic search second

2. **On Content Generation**:
   - LLM-generated templates work well
   - Cache and refine
   - Focus tooling on data creation
   - Consider crowdsourcing content

3. **On Retrieval**:
   - Three approaches: RAG, boolean (keyword), knowledge graphs
   - Hybrid works best
   - Store experiences, not descriptions
   - Generate hypothetical questions for better matching

4. **On Architecture**:
   - Backend work easier than frontend (for them)
   - Security concerns only matter for multi-user
   - Local-first design simplifies considerably
   - Plan for modding from start

## Technical Challenges Overcome

1. **Cross-Platform Support**: Micromamba/conda integration for Windows
2. **Context Window Management**: Dynamic token estimation and pruning
3. **Retrieval Quality**: Hybrid boolean + semantic + exact matching
4. **UI Complexity**: Selection-based menus to reduce typing fatigue
5. **LLM Consistency**: NDL system for controlled generation
6. **State Synchronization**: Event-driven architecture with deterministic outcomes

## Influence on Community

- Dominated technical discussions (sheer volume of contributions)
- Set standards for architectural thinking
- Provided working code examples
- Debugged others' implementations
- Pushed boundaries of what's considered possible with LLMs

## Related Pages

- [[02-ReallmCraft-Project]] - Full project documentation
- [[08-NDL-Natural-Description-Language]] - NDL system details
- [[01-Architecture-and-Design]] - Architectural contributions
- [[05-RAG-and-Memory]] - RAG system implementations
- [[User-50h100a]] - Thread originator, frequent collaborator
- [[User-appl2613]] - Fellow engine developer

---

> [!note] Analysis Note
> veritasr represents the most prolific technical contributor in the transcript, with contributions spanning the entire 18-month period. Their work demonstrates deep systems thinking, willingness to iterate radically, and a pragmatic approach to leveraging LLMs as tools rather than solutions.
