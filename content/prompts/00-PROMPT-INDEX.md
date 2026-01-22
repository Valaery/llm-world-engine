---
tags: [prompt, index, reference, guide]
date: 2026-01-16
status: comprehensive
---

# Prompt Library Index

## Overview

This library catalogs all prompts, prompt techniques, and prompting strategies discussed in the LLM World Engine Discord community. The collection represents nearly two years of experimentation, refinement, and practical implementation across multiple LLM game engine projects (ReallmCraft, ChatBot RPG).

**Total Prompts Documented**: 17 complete
**Techniques Covered**: 9
**Models Tested**: GPT-4, GPT-3.5, Claude, Mixtral, Llama 3, Gemma 2, Mistral, EstopianMaid, Stheno
**Date Range**: January 2024 - December 2025

## Core Philosophy

The community's prompt engineering approach centers on a fundamental insight:

> **LLMs excel at narration, not decision-making.**

Rather than asking LLMs to track state and make game logic decisions (which they do poorly), prompts should provide structured descriptions of predetermined events and ask the LLM to translate them into natural narrative prose.

## Quick Reference by Category

### Narration Prompts
Transform programmatic game events into natural language narrative:
- [[ndl-to-narrative]] - NDL markup to narrative prose (PRIMARY TECHNIQUE)
- [[scene-description]] - Location and environment description
- [[action-narration]] - Player/NPC action narration
- [[dialogue-generation]] - Character speech with subtext
- [[combat-narration]] - Combat sequence description

### Generation Prompts
Create game content (characters, locations, items, quests):
- [[character-generation]] - NPC character sheets
- [[location-generation]] - Places and environments
- [[template-generation]] - **Meta-prompts that generate templates** (NEW)

### Constraint Prompts
Control and limit LLM behavior to prevent hallucinations:
- [[anti-hallucination]] - Core constraint rules ("Impossible actions must fail")
- [[hallucination-prevention]] - Explicit prohibitions with data grounding
- [[format-enforcement]] - **Structured output control** (NEW)
- [[length-limiting]] - **Token budget management** (NEW)

### Retrieval Prompts
RAG and memory system optimization:
- [[query-formulation-hyde]] - **HyDE for improved semantic search** (NEW)

### Reasoning Prompts
Multi-step thinking and decision support:
- [[chain-of-thought]] - Step-by-step reasoning
- [[binary-classification]] - **Yes/No validation for game state** (NEW)

### System Prompts
Base instructions that define LLM behavior:
- [[narration-engine-system]] - Core narration system prompt

## Quick Reference by Technique

### 1. NDL (Natural Description Language)
**Status**: Production-ready, PRIMARY TECHNIQUE
**Creator**: [[User-veritasr]]
**File**: `narration/ndl-to-narrative.md`

Convert programmatic game events into structured markup that LLMs reliably translate into narrative prose. Works on 7B-9B models.

### 2. Chain of Thought (CoT)
**Status**: Proven
**File**: `reasoning/chain-of-thought.md`

Force LLM to think step-by-step before responding. Especially useful for smaller models.

### 3. Few-Shot Prompting
**Status**: Essential for new formats
**File**: `techniques/few-shot-examples.md`

Provide 1-3 examples to teach LLMs custom output formats like NDL.

### 4. Constraint-Based Prompting
**Status**: Critical for reliability
**File**: `constraint/anti-hallucination.md`

Tell LLMs what they CANNOT do to prevent hallucinations.

### 5. Template-Based Generation
**Status**: Production-ready
**Files**: `generation/character-generation.md`, `generation/location-generation.md`

Use structured templates with placeholders for consistent content generation.

### 6. Context Inheritance
**Status**: Proven pattern
**File**: `generation/location-generation.md`

Child elements (locations, NPCs) inherit thematic properties from parent regions.

### 7. HyDE (Hypothetical Document Embeddings)
**Status**: Proven for RAG
**File**: `retrieval/query-formulation-hyde.md`

Generate hypothetical questions content could answer, improving semantic search retrieval quality by 40-80%.

### 8. Meta-Prompting
**Status**: Production-ready
**File**: `generation/template-generation.md`

Prompts that generate other prompts. Use LLMs to create structured templates for procedural content generation.

### 9. Binary Classification
**Status**: Production-ready
**File**: `reasoning/binary-classification.md`

Extract binary decisions from natural language for game logic. Question tree approach with [YES]/[NO] parsing enables deterministic state management.

## Files Included in This Library

### Narration (5 files)
- `narration/ndl-to-narrative.md` - **PRIMARY**: NDL markup → natural prose
- `narration/scene-description.md` - Location and environmental descriptions
- `narration/action-narration.md` - Player/NPC action narration
- `narration/dialogue-generation.md` - Character speech with subtext
- `narration/combat-narration.md` - Turn-based combat sequences

### Generation (3 files)
- `generation/character-generation.md` - Create NPCs with full character sheets
- `generation/location-generation.md` - Generate locations with context inheritance
- `generation/template-generation.md` - **NEW**: Meta-prompts that generate templates

### Constraints (4 files)
- `constraint/anti-hallucination.md` - Core constraint rules approach
- `constraint/hallucination-prevention.md` - Explicit prohibitions with data grounding
- `constraint/format-enforcement.md` - **NEW**: Structured output control with brackets
- `constraint/length-limiting.md` - **NEW**: Token budget management (170-token technique)

**Note**: Both `anti-hallucination.md` and `hallucination-prevention.md` address the same core problem (preventing LLM hallucinations) but use complementary approaches. The former focuses on rule-based constraints, while the latter emphasizes explicit data grounding and comprehensive prohibitions. Use both together for maximum effectiveness.

### Retrieval (1 file)
- `retrieval/query-formulation-hyde.md` - **NEW**: HyDE for improved semantic search

### Reasoning (2 files)
- `reasoning/chain-of-thought.md` - Step-by-step reasoning patterns
- `reasoning/binary-classification.md` - **NEW**: Yes/No validation for game state extraction

### System (1 file)
- `system/narration-engine-system.md` - Base narration engine instructions

### Techniques (1 file)
- `techniques/few-shot-examples.md` - Teaching by example patterns

## Best Practices Summary

### DO:
- ✅ Use NDL for narration (most reliable technique)
- ✅ Separate decision-making from narration
- ✅ Test on target model early and often
- ✅ Apply constraints liberally
- ✅ Use few-shot for new formats
- ✅ Validate and post-process outputs
- ✅ Cache generated content

### DON'T:
- ❌ Let LLM make game logic decisions
- ❌ Assume LLM remembers implicit rules
- ❌ Over-complicate prompts
- ❌ Give unlimited creative freedom
- ❌ Use GPT-4 prompts on 7B models unchanged

## Model Recommendations

### Small Models (7B-9B)
**Best for**: Narration with NDL
**Requirements**: Tight constraints, few-shot examples, simple prompts
**Temperature**: 0.6-0.9 for narration

### Medium Models (13B-30B)
**Best for**: Balanced narration and generation
**Requirements**: Moderate constraints, some examples
**Temperature**: 0.5-0.8

### Large Models (GPT-4, Claude)
**Best for**: Complex generation, reasoning
**Requirements**: Can work with just instructions
**Temperature**: 0.3-0.7 depending on task

## Temperature Guide

```
Structured Output:  0.1 - 0.3  (parsing, validation)
Narration:          0.6 - 0.9  (events, descriptions)
Dialogue:           0.7 - 1.0  (character speech)
Generation:         0.7 - 0.9  (new content)
Reasoning:          0.3 - 0.5  (analysis, planning)
```

## Key Community Insights

> "Turns out that when you take away decision making from the LLM it behaves much better." - [[User-veritasr]]

> "LLMs are super cliche and shallow on their own devices... but so would we if we just one-shot everything" - [[User-appl2613]]

> "{{char}} is a logical and realistic text adventure game. Impossible actions must fail." - [[User-yukidaore]]

> "gpt-4 was very good, its the only model that can kinda-sorta one-shot a good RP with all the rules just added to the context" - [[User-monkeyrithms]]

## Evolution Timeline

**January 2024**: Experimentation with LLM as state manager (failed), CoT patterns
**February-April 2024**: Constraint-based approaches, template prompts, early NDL
**May-June 2024**: NDL formalization, quest pacing, turn counting
**July 2024-2025**: Advanced techniques, character psychology, cross-model testing

## Related Documentation

- [[02-Prompt-Engineering]] - Full theoretical discussion and techniques
- [[08-NDL-Natural-Description-Language]] - Complete NDL specification
- [[03-RAG-and-Memory]] - Context management and retrieval prompts
- [[04-World-Generation]] - Content generation strategies
- [[01-Architecture-and-Design]] - How prompting fits into architecture
- [[User-veritasr]] - Primary contributor (NDL, system design)
- [[User-appl2613]] - Character/location generation, quest pacing
- [[User-50h100a]] - CoT patterns, early architecture
- [[User-yukidaore]] - Constraint patterns, anti-hallucination

## Contributing

When adding new prompts:
1. Use the standard template format (see existing files)
2. Include effectiveness notes from real usage
3. Specify which models were tested
4. Provide complete, working examples
5. Tag with relevant techniques
6. Link to source discussions

## Usage Notes

All prompts are production-tested in real game engines (ReallmCraft, ChatBot RPG). They represent nearly two years of community experimentation and refinement.

**Primary Technique**: NDL (Natural Description Language) is the community's consensus approach for reliable narration. Start here.

**For Small Models**: Use NDL + constraints + few-shot examples. This combination works reliably on 7B-9B parameter models.

**For Content Generation**: Use template-based approaches with validation and post-processing.

**Last Updated**: 2026-01-17 (corrected file count and added hallucination-prevention)
**Source**: LLM World Engine Discord (Jan 2024 - Dec 2025)
**Total Messages Analyzed**: 12,109

## Recent Additions (2026-01-17)

### Latest Update (2026-01-17 Evening)
5. **Binary Classification** (`reasoning/binary-classification.md`)
   - Yes/No validation prompts for extracting game state from natural language
   - Question tree pattern for multi-step validation
   - Grammar-constrained output forcing
   - Production-tested in ChatBot RPG with 95%+ accuracy
   - Complete implementation examples with Python code

### Earlier Additions (2026-01-17 Morning)
1. **Template Generation** (`generation/template-generation.md`)
   - Meta-prompts that generate structured JSON templates
   - Create generators for locations, items, characters, quests
   - Supports hierarchical JITG (Just-In-Time Generation)
   - Production-tested in ReallmCraft

2. **Length Limiting** (`constraint/length-limiting.md`)
   - 170-token sweet spot technique from ChatBot RPG
   - Combines API limits + prompt instructions + regex post-processing
   - Prevents rambling and maintains consistent pacing
   - Based on front-loaded coherence principle

3. **Format Enforcement** (`constraint/format-enforcement.md`)
   - Bracketed output for reliable parsing: [ITEM1],[ITEM2]
   - Structured extraction of items, locations, actions
   - Production-proven in ChatBot RPG inventory system
   - Includes hallucination mitigation strategies

4. **Query Formulation (HyDE)** (`retrieval/query-formulation-hyde.md`)
   - Hypothetical questions for improved semantic search
   - 40-80% better recall vs. standard RAG
   - Hybrid retrieval combining exact, tag, semantic, and graph methods
   - Includes Reciprocal Rank Fusion (RRF) for result combination
