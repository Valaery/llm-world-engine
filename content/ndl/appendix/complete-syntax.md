---
tags: [ndl, reference, syntax, complete, draft]
type: Reference Documentation
category: NDL Appendix
status: draft
---

# NDL Complete Syntax Reference

## Overview

**Purpose**: Comprehensive reference of all NDL syntax elements
**Status**: DRAFT - Awaiting consolidation from all NDL specification files
**Audience**: Developers and advanced users

## Placeholder Structure

### All Constructs Quick Reference

**Core Constructs**:
- `[scene]` - Scene definitions
- `[character]` - Character elements
- `[dialogue]` - Dialogue blocks
- `[action]` - Actions and events
- `[state]` - State management
- `[entity]` - Generic entities
- `[properties]` - Attribute definitions (see [[ndl/constructs/properties]])

**Advanced Constructs**:
- `[combat]` - Combat sequences
- `[event]` - World events
- `[condition]` - Conditional logic
- `[relationship]` - Entity relationships

### Attributes and Modifiers

**Common Attributes**:
- `id` - Unique identifier
- `type` - Element type
- `mood` - Emotional state
- `intent` - Character intent

### Expected Content

**Complete Syntax Table**:
| Construct | Attributes | Required | Optional | Example |
|-----------|------------|----------|----------|---------|
| [scene] | id, description | id | mood, lighting | ... |
| [character] | id, name | id | mood, state | ... |
| ... | ... | ... | ... | ... |

**All Examples**:
- Basic examples for each construct
- Complex nested examples
- Edge cases and special syntax
- Common patterns

## Related Documentation

- [[ndl/specification/formal-grammar]] - EBNF grammar
- [[ndl/00-NDL-INDEX]] - NDL master index
- All construct documentation files

---

> [!warning] Draft Status
> This file is a stub placeholder. Consolidate syntax from all existing NDL specification files and add examples from Discord discussions.
