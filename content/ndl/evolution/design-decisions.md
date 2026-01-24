---
tags: [ndl, evolution, design, rationale, draft]
type: Design Documentation
category: NDL Evolution
status: draft
---

# NDL Design Decisions

## Overview

**Purpose**: Document the rationale behind NDL's design choices
**Status**: DRAFT - Awaiting extraction from Discord discussions
**Key Designer**: [[User-veritasr]]

## Placeholder Structure

### Core Design Principles

**Why Markup Language?**
- Human-readable but machine-processable
- Separates structure from content
- LLM-friendly format
- Easier debugging than pure JSON

**Why Not Just JSON/YAML?**
- More expressive for narrative elements
- Better LLM comprehension
- Hierarchical and contextual
- Natural language compatible

**Why Program-First Architecture?**
> "Turns out that when you take away decision making from the LLM it behaves much better."

- LLMs bad at state management
- Deterministic game logic required
- Prevents hallucination
- Enables small model usage

### Expected Content

**Design Trade-offs**:
- Syntax complexity vs expressiveness
- Parsing performance vs readability
- LLM compatibility vs precision
- Extensibility vs simplicity

**Alternative Approaches Considered**:
- Pure prompt engineering (rejected: inconsistent)
- JSON-based events (rejected: less LLM-friendly)
- Natural language only (rejected: too ambiguous)
- Code generation (rejected: too complex)

**Lessons Learned**:
- Small models work with constraints
- Subtext through structure, not instructions
- Less LLM freedom = better results
- Iteration and testing essential

## Related Documentation

- [[ndl/evolution/version-history]] - How NDL changed over time
- [[ReallmCraft-Project]] - Implementation context
- [[patterns/architectural/program-first-architecture]] - Core pattern

---

> [!warning] Draft Status
> This file is a stub placeholder. Run the ndl-reference-builder agent to extract design rationale from Discord discussions, especially veritasr's philosophy explanations.
