---
title: LLM World Engine Knowledge Base
---

# Welcome to the LLM World Engine Knowledge Base

> **Comprehensive documentation extracted from 24 months of Discord discussions + ChatBotRPG source code analysis**

This knowledge base synthesizes **12,109 Discord messages** from the SillyTavern community's "LLM World Engine" thread (January 2024 - December 2025) into production-ready documentation comprising **139 files** and **63,000+ lines** of content.

## 📚 Quick Navigation

### Core Resources

**[[00-MASTER-INDEX|Master Index]]** - Complete navigation hub for all 139 documents

### Main Topics

1. **[[01-Architecture-and-Design|Architecture & Design]]** - Core philosophy and system design
2. **[[02-Prompt-Engineering|Prompt Engineering]]** - Production-tested prompt templates
3. **[[03-RAG-and-Memory|RAG & Memory]]** - Retrieval systems and context management
4. **[[04-World-Generation|World Generation]]** - Procedural content generation
5. **[[05-State-Management|State Management]]** - Game state and persistence
6. **[[06-UI-and-Frontend|UI & Frontend]]** - User interfaces and interactions
7. **[[07-Models-and-APIs|Models & APIs]]** - LLM integration patterns
8. **[[08-NDL-Natural-Description-Language|NDL Specification]]** - Natural Description Language

## 🎯 Documentation Collections

### [[prompts/00-PROMPT-INDEX|Prompt Library]] (18 Templates)
Production-tested prompts with real performance metrics:
- **Narration** (5): NDL translation, scene/action/dialogue/combat
- **Generation** (3): Character, location, template creation
- **Constraint** (4): Anti-hallucination, format enforcement, length limiting
- **Retrieval, Reasoning, System** (6): HyDE queries, chain-of-thought, few-shot

### [[patterns/00-PATTERN-INDEX|Pattern Library]] (21 Patterns)
Battle-tested architectural patterns:
- **Architectural** (4): Program-first, LLM pipeline, separation of concerns, event-driven
- **Integration** (4): NDL bridge, state injection, API abstraction, multi-model routing
- **State** (3): Three-tier persistence, scene boundaries, conditional saves
- **Generation** (3): JIT generation, hierarchical cascade, template meta-generation
- **Control** (4): Constraint prompting, chain-of-thought, few-shot, temperature switching

### [[ndl/00-NDL-INDEX|NDL Specification]] (27 Documents)
Complete Natural Description Language specification:
- **7 Core Constructs**: `do()`, `search()`, `wait()`, `result()`, sequencing, manner, intention
- **4 Formal Specs**: EBNF grammar, lexical elements, semantics, type system
- **2 Integration Guides**: LLM pipeline, parser implementation

### [[chatbotrpg-analysis/00-CHATBOTRPG-INDEX|ChatBotRPG Analysis]] (41 Documents)
Deep implementation analysis:
- **Discord-based analysis**: Pattern implementation, production lessons
- **Code analysis**: 15 extracted prompts, 100+ code references, 89% validation accuracy
- **Schemas**: 10 core + 8 sub-schemas documented
- **Architecture**: Multi-provider API integration

## 🔍 Key Insights

> **"LLMs as Narrators, Not Decision-Makers"**
>
> The fundamental philosophy: Game logic runs in deterministic code, NDL captures decisions, LLMs translate to natural language.

### Production Metrics
- **170-token sweet spot**: Optimal prompt length
- **40% cost savings**: Structured state vs. verbose context
- **95%+ anti-hallucination accuracy**: Constraint-based prompting
- **8B models viable**: Gemma 2 9B sufficient for narration

### Key Contributors
- **[[User-veritasr]]**: NDL creator, ReallmCraft developer
- **[[User-appl2613]]**: ChatBot RPG developer, token optimization
- **[[User-50h100a]]**: Constraint philosophy, function keywords

## 📊 Project Statistics

- **139 total files**: Comprehensive knowledge base
- **63,563 lines**: Production-ready documentation
- **1,000+ code examples**: Python with type hints
- **41+ Mermaid diagrams**: Architecture visualizations
- **24 months synthesized**: Jan 2024 - Dec 2025
- **15+ contributors**: Community-driven knowledge

## 🚀 Getting Started

New to LLM game engines? Start here:

1. Read [[01-Architecture-and-Design|Architecture & Design]] to understand the core philosophy
2. Explore [[prompts/narration/ndl-to-narrative|NDL to Narrative]] - the foundational prompt
3. Review [[patterns/architectural/program-first-architecture|Program-First Architecture]] pattern
4. See [[chatbotrpg-analysis/analysis/01-Repository-Overview|ChatBotRPG]] for a real implementation

## 📖 How to Use This Knowledge Base

- **Browse by topic**: Use the topic indexes above
- **Search**: Use the search box (top left) to find specific content
- **Follow links**: Click [[wiki-links]] to navigate between related documents
- **View graph**: See the knowledge graph (right sidebar) for visual connections
- **Check backlinks**: Each page shows what other pages reference it

## 🤝 About This Project

This knowledge base was created using automated AI agents that:
1. Analyzed 12,109 Discord messages
2. Extracted 18 prompts, 18 patterns, complete NDL spec
3. Validated claims against ChatBotRPG source code (89% accuracy)
4. Generated Obsidian-compatible markdown with proper wiki-links

**Source**: [SillyTavern Discord](https://discord.gg/sillytavern) - `#llm-tech-chat` channel

---

*Last updated: January 2026 | Built with [Quartz v4](https://quartz.jzhao.xyz/)*
