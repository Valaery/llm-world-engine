---
tags: [ndl, glossary, terminology, reference, draft]
type: Reference Documentation
category: NDL Appendix
status: draft
---

# NDL Glossary

## Overview

**Purpose**: Definitions of all NDL-related terms and concepts
**Status**: DRAFT - Awaiting term extraction from NDL documentation
**Audience**: All NDL users

## Placeholder Structure

### Core Terms

**NDL (Natural Description Language)**
- Markup language for converting programmatic game events into LLM-compatible narrative prompts
- Created by [[User-veritasr]] for [[ReallmCraft-Project]]
- Key innovation: separates state (program) from narration (LLM)

**Construct**
- Structural element of NDL syntax (e.g., [scene], [character], [action])
- Defines semantic meaning and nesting rules

**Decorator Pattern**
- LLM as decorator: AI adds narrative "decoration" to programmatic events
- LLM makes no decisions, only generates text

**Program-First Architecture**
- Design pattern where backend logic determines all state
- LLM relegated to narrative generation only
- Core philosophy of NDL-based engines

### Expected Content

**Technical Terms**:
- AST (Abstract Syntax Tree)
- Lexer / Parser / Tokenizer
- Semantic validation
- Context window management
- Token estimation

**NDL-Specific Terms**:
- Scene construct
- Character mood
- Dialogue intent
- Entity reference
- State fixation

**Related Concepts**:
- RAG (Retrieval-Augmented Generation)
- Constraint programming
- Template system
- Just-In-Time generation
- Context injection

## Related Documentation

- [[ndl/00-NDL-INDEX]] - NDL master index
- [[08-NDL-Natural-Description-Language]] - NDL overview
- [[patterns/architectural/program-first-architecture]] - Core pattern

---

> [!warning] Draft Status
> This file is a stub placeholder. Extract terms from all NDL documentation files and Discord discussions for comprehensive glossary.
