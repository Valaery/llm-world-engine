---
tags: [architecture, system-design, state-management, thread]
date: 2026-01-15
participants:
  - "[[User-vali98]]"
  - "[[User-50h100a]]"
  - "[[User-irovos]]"
  - "[[User-veritasr]]"
  - "[[User-giftedgummybee]]"
status: analyzed
---

# Thread: Architecture and Design Patterns

## Summary

This thread captures the foundational architectural discussions about building LLM-powered game engines. The central challenge: how to maintain programmatic control over game state and logic while leveraging LLMs for natural language generation and interaction.

## Key Architectural Questions

### The Core Problem (Jan 9, 2024)

The initial debate centered on a fundamental architectural choice:

**Option A: LLM-First**
```
state → LLM → Parse state updates → Update state
```

**Option B: Program-First**
```
state → Player choice → update state → LLM for text
```

**Primary Issue**: "The processing of state updates from the LLM" - [[User-vali98]]

> [!warning] The Reliability Problem
> "If you need chatgpt to make it work you've already lost" - [[User-50h100a]]
>
> The consensus emerged that relying on the LLM for state management leads to unreliability and inconsistency.

## Proposed Solutions

### 1. Constrain LLM Role (Early Consensus)

**Key Insight from [[User-50h100a]]**:
- Don't let the LLM update state directly
- Have LLM trigger changes instead
- Use function-like keywords with CoT (Chain of Thought)

Example approach:
```
FunctionRoll(Willpower, Ram Girl Lyre)
→ Stop generation
→ Perform roll
→ Inject result
→ Continue generation
```

### 2. Separate Actions from Dialogue

**From [[User-irovos]]**:
> "From the basic get go, i think you need to seperate actions and dialogue"

This insight led to multiple implementation strategies across different projects.

### 3. Processing Pipeline

**[[User-vali98]]'s Flow Proposal**:
```
User Input
→ Parse input to update state
→ Update state
→ LLM Describes state
```

**Compared to**:
```
Have a state
→ User sends actions to LLM
→ LLM returns and now has to be queried for updated state
→ Update state
```

The first approach was generally preferred as it removes the "???" and "Profit" steps where LLM unreliability enters.

#### veritasr's LLM Processing Phases Diagram

**veritasr** [Feb 2024, 10:39]: Formalized the processing pipeline based on community discussions:

![[Media/LLM_Processing_Phases-39904.png]]

*Diagram showing the complete LLM processing lifecycle:*
- **Preprocessing**: Takes user-submitted prompt, prepares context and state
- **Generation**: LLM produces the narrative output
- **Post Processing**: Validates and formats the generated content
- **Post Return Processing**: Handles the response while waiting for next prompt, with feedback loop passing output back to user

**Key Insights**:
- Preprocessing phase prepares deterministic state before LLM sees it
- Generation is isolated - LLM doesn't decide logic, only narrates
- Post-processing validates output meets format/content requirements
- Feedback loop allows continuous refinement while maintaining state consistency

This diagram became a reference point for both ReallmCraft and influenced ChatBot RPG's architecture, demonstrating the consensus around separating programmatic control (pre/post) from generative narration (generation phase).

## State Management Strategies

### Defining "State"

The discussion revealed multiple interpretations:

**Simple Interpretation** ([[User-vali98]]):
> "Your current Stats and debuff values in numeric form"

**Broader Interpretation** ([[User-50h100a]]):
- Statblocks (automatically updated)
- Physical state
- Mental state
- Geographic state
- Item state
- Relationship state

> [!question] The "Sudden Door" Problem
> How do you handle when players or LLMs introduce new elements not in the state?
>
> **[[User-50h100a]]'s insight**: "sudden door" requires exhaustive scene description, which would be 500+ tokens and the model would go off rails.
>
> **Compromise**: Start with constrained state (statblocks, core elements) and expand carefully.

### Handling Environment State

Two competing approaches emerged:

**Top-Down Generation**:
```
World → Continent → Nation → Province → City → Building → Room
```
- Better for story-driven games
- Allows coherent theme management
- More upfront work

**Bottom-Up Generation**:
```
Current Room → Building → City (generate as needed)
```
- More flexible
- Just-In-Time generation
- Can lead to narrative inconsistency

**[[User-veritasr]]'s Hybrid Solution** (Feb 2024):
> "Theoretically if I did it in a deterministic fashion, then it could literally be mapped. But... whether or not it's actually worth doing it in a deterministic fashion or just having loose entries that are related to one another without any sort of overarching heirarchy..."
>
> **Result**: Entry-based system where `entry n` is connected to `entry y` without rigid hierarchy

## The Prolog Experiment

**[[User-irovos]]'s Insight**:
> "I liked the whole idea behind using prolog. Because its already linguistic Ramgirl(has_erection)"
> "you can literally feed it raw prolog and it can interpret it"

This led to explorations of linguistic programming approaches that LLMs could parse naturally while remaining programmatically enforceable.

## Event-Driven Architecture

### Event System Design

**[[User-vali98]]'s Initial Event Structure**:
```typescript
type Event {
   prompt: string,
   statModifier: Function,
   option: Array<Options>,
}

type Option {
  statModifier: Function,
  NextEvent: Dict<Event, chance>
}
```

**Critique**: This is essentially a state machine, which proved too limiting.

**Better Approach** (emerged later):
- Events trigger based on conditions
- Events modify state deterministically
- LLM describes the resolved event
- No user-facing "options" (free-form input instead)

## Idempotence and Determinism

**[[User-irovos]]'s Principle**:
> "a system like this should be idempodent... not model bound"

**Meaning**: The same input should always produce the same state change, regardless of which LLM is used. The LLM's job is narration, not decision-making.

## Data-Driven Design Philosophy

### The "Just a Dataset" Realization

**[[User-giftedgummybee]]**:
> "Again, sounds like a dataset problem... train a model to update a state based off a current and parsed update"

**Community Response**: While theoretically possible, building a custom model was deemed impractical. Instead, focus shifted to:

1. **Structured data formats** that any LLM can work with
2. **Prompt engineering** to constrain behavior
3. **Programmatic enforcement** of rules

### Template Systems

The conversation repeatedly returned to templates as a solution:

- **Templates for characters**
- **Templates for locations**
- **Templates for events**
- **Templates for items**

This evolved into full modding frameworks in both major projects ([[ReallmCraft-Project]], [[ChatBotRPG-Project]]).

## Function Calling and Tool Use

### Early Function-Like Keywords

**[[User-50h100a]]'s Proposal**:
```
FunctionRoll(Willpower, Target)
```

This predated widespread function-calling support in LLMs but pointed toward what would become standard practice.

### The Control Problem

> [!important] Core Design Principle
> "Focus on constraining what the LLM 'does' rather than the user" - [[User-50h100a]]
>
> The user doesn't tend to manifest impossible things unless deliberately doing so. The LLM, however, will consistently introduce elements that break game logic.

## Architecture Evolution Over Time

### Phase 1: Pure LLM (Rejected)
Let LLM handle everything → Too unreliable

### Phase 2: Hybrid Parsing
LLM output → Parse for state changes → Update state → Too error-prone

### Phase 3: Constrained LLM (Consensus)
State changes determined programmatically → LLM narrates results → Reliable

### Phase 4: Rule Engines (Later Development)
- Declarative rules
- Event-driven systems
- LLM as "decorator" (narration layer only)

## Constraint Programming (July 2024)

**[[User-veritasr]]'s Late-Stage Insight**:

Shift from procedural logic to constraint-based:
```
Traditional: "Do these steps to reach result"

Constraint-Based:
1. These are possible solutions
2. These constraints make solutions unusable
3. Remove violating solutions
4. Project best solution forward
```

**Function**: Serves as "deduction" - eliminating impossibilities rather than constructing solutions.

**Combined with**:
- **Rule engines** (step-by-step induction)
- **Utility systems** (weighing pros/cons)
- **Constraint solvers** (eliminating impossibilities)

> [!tip] Emergent Behavior
> "When you think of it that way, the three systems combined is actually pretty powerful... Interplay between all the systems basically promotes emergent behavior." - [[User-veritasr]]

## Key Architectural Patterns Identified

### 1. Separation of Concerns
- **Logic Layer**: Game rules, state management
- **Narrative Layer**: LLM text generation
- **Data Layer**: Templates, entities, relationships

### 2. Event-Driven Design
- Events triggered by conditions
- Events processed by rules
- Events narrated by LLM

### 3. Template-Based Generation
- Reusable schemas
- Parameterized content
- Modular expansion

### 4. Just-In-Time Generation
- Don't generate everything upfront
- Create as needed
- Cache once created

### 5. Deterministic State, Probabilistic Narration
- State changes are predictable
- Narration can vary
- Same inputs → same game state
- Different narration each time

## Lessons Learned

1. **LLMs are terrible at state management** - Keep state programmatic
2. **Templates solve the content problem** - Structured data > free generation
3. **Constraint the LLM, not the player** - Users are more predictable than AI
4. **Separate actions from narration** - What happens vs. how it's described
5. **Modular architectures scale better** - Plan for expansion and mods

## Related Threads

- [[02-Prompt-Engineering]] - Techniques for constraining LLMs
- [[05-State-Management]] - State persistence patterns
- [[08-NDL-Natural-Description-Language]] - Solution to narration problem
- [[User-veritasr]] - Primary architect
- [[User-50h100a]] - Early constraint philosophy

## Related Enrichment Outputs

### Pattern Library
- [[patterns/00-PATTERN-INDEX]] - Complete architectural pattern library
- [[patterns/architectural/program-first-architecture]] - Backend-driven decision-making
- [[patterns/architectural/llm-processing-pipeline]] - Pre/Gen/Post processing phases
- [[patterns/architectural/separation-of-concerns]] - Logic/Narrative/Data layer split
- [[patterns/architectural/event-driven-design]] - Trigger-based game events
- [[patterns/integration/ndl-bridge]] - NDL bridge pattern for narration

### Prompt Library
- [[prompts/00-PROMPT-INDEX]] - Complete prompt library
- [[prompts/system/narration-engine-system]] - Base narration system prompt

## Quotes Worth Remembering

> "Just use arbitrary xml tags" - [[User-giftedgummybee]] (the recurring "just" problem)

> "then do so, man" - [[User-50h100a]] (response to "just" suggestions)

> "Turns out that when you take away decision making from the LLM it behaves much better." - [[User-veritasr]]

---

## Action Items That Emerged

- [x] Separate state management from LLM (consensus reached)
- [x] Develop template systems for content (both projects implemented)
- [x] Create rule/event systems (implemented differently in each project)
- [x] Build modding frameworks (both projects achieved this)
- [ ] Industry-wide standard for LLM game state representation (still evolving)
