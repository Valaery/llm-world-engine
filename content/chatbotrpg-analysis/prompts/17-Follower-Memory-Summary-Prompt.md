---
tags: [prompt, chatbotrpg, memory, context-management]
source: ChatBotRPG/summaries.py
type: System Prompt
status: extracted
date_added: 2026-01-24
---

# Follower Memory Summary Prompt

## Overview

**Purpose**: Summarize shared scenes and experiences for NPC followers to maintain memory coherence
**Source File**: `summaries.py`
**Model Usage**: Context summarization for follower memory system
**Implementation**: ChatBotRPG follower tracking system

## Context

This prompt is used to generate memory summaries for NPC followers when they accompany the player across multiple scenes. It ensures followers retain coherent memories of shared experiences while managing context window limitations.

## Prompt Structure

### Core Objective
Summarize conversation history and shared scenes from the follower's perspective, maintaining:
- Memory coherence across scene transitions
- Follower-specific observations and reactions
- Key events and interactions witnessed
- Emotional state and relationship development

### Expected Inputs
- Previous scene transcripts
- Follower character sheet and personality
- Shared events and interactions
- Current relationship status with player

### Expected Outputs
- Concise summary of shared experiences (follower POV)
- Key emotional moments
- Notable decisions or turning points
- Context for future interactions

## Usage Pattern

```python
# From summaries.py - Follower memory summarization
def summarize_follower_memory(follower, scene_history):
    """
    Generate summary of shared experiences for follower NPC.

    Args:
        follower: NPC follower character data
        scene_history: List of previous scenes with this follower

    Returns:
        str: Summarized memory from follower perspective
    """
    prompt = construct_follower_summary_prompt(
        follower_data=follower,
        scenes=scene_history
    )

    summary = llm_inference(prompt)

    return summary
```

## Related Prompts

- [[16-Context-Summarization-Prompt]] - General conversation history summarization
- [[01-Character-Generation-Prompt]] - Character personality definition
- [[03-Scene-Setting-Prompt]] - Scene context management

## Implementation Notes

### Memory Management
- Summarization triggered on scene transitions
- Maintains follower continuity across locations
- Reduces token usage for long-term followers
- Preserves relationship development arc

### Technical Details
- **Context Window**: Manages follower memory within limits
- **Trigger Condition**: Scene change with active follower
- **Storage**: Persisted in follower character state
- **Update Frequency**: Per scene transition or every N turns

## Related Files

- **[[chatbotrpg-analysis/schemas/character-schema]]** - Character data structure
- **[[chatbotrpg-analysis/analysis/architecture]]** - System architecture overview
- **[[chatbotrpg-analysis/patterns/state-management]]** - State persistence patterns

---

> [!note] Extraction Status
> This prompt was identified in the ChatBotRPG codebase during prompt-forensics-agent analysis. Full source code details available in `summaries.py`.

> [!info] Usage Context
> Part of the follower/companion system that allows NPCs to travel with the player across multiple scenes while maintaining coherent memory and personality.
