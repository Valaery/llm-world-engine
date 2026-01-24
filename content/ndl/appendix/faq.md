---
tags: [ndl, faq, questions, guide, draft]
type: FAQ Documentation
category: NDL Appendix
status: draft
---

# NDL Frequently Asked Questions

## Overview

**Purpose**: Answer common questions about NDL usage and implementation
**Status**: DRAFT - Awaiting question extraction from Discord discussions
**Audience**: New NDL users and implementers

## Placeholder Structure

### General Questions

**Q: What is NDL?**
A: Natural Description Language - a markup language for converting programmatic game events into narrative prompts for LLMs.

**Q: Why use NDL instead of direct prompts?**
A: NDL provides structure, prevents LLM hallucination of state, enables small model usage, and separates game logic from narrative.

**Q: Who created NDL?**
A: [[User-veritasr]] developed NDL as part of the [[ReallmCraft-Project]].

### Technical Questions

**Q: What models work with NDL?**
A: Tested on Gemma, Llama3, Mistral, Stheno (7B-9B). Works best with models that follow structural constraints.

**Q: How do I implement NDL in my game?**
A: See [[ndl/integration/game-state-to-ndl]] and [[ndl/integration/llm-integration]] for implementation guides.

**Q: Can I extend NDL with custom constructs?**
A: (To be documented - depends on parser implementation)

### Expected Content

**Implementation Questions**:
- How to parse NDL?
- Performance considerations?
- Error handling best practices?
- Testing strategies?

**Design Questions**:
- When to use NDL vs direct prompts?
- How to handle edge cases?
- Custom construct design?
- Integration with existing systems?

**Troubleshooting**:
- LLM not following NDL structure?
- Parser errors and debugging?
- Performance issues?
- Model compatibility problems?

## Related Documentation

- [[ndl/00-NDL-INDEX]] - NDL master index
- [[08-NDL-Natural-Description-Language]] - NDL overview
- [[ReallmCraft-Project]] - Reference implementation

---

> [!warning] Draft Status
> This file is a stub placeholder. Extract common questions from Discord discussions and create comprehensive FAQ with answers from veritasr's explanations.
