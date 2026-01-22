---
tags: [prompt, retrieval, rag, hyde, semantic-search, memory]
category: retrieval
technique: hypothetical-document-embeddings
status: proven
contributor: [[User-veritasr]]
models_tested: [GPT-3.5, sentence-transformers, ChromaDB]
date_created: 2024-02-11
date_updated: 2024-07-15
---

# Prompt: Query Formulation (HyDE for RAG)

## Metadata
- **Category**: Retrieval (RAG)
- **Technique**: HyDE (Hypothetical Document Embeddings) + Hypothetical Questions
- **Model Tested**: GPT-3.5, GPT-4, sentence-transformers (all-MiniLM-L6-v2)
- **Contributor**: [[User-veritasr]]
- **Status**: Proven (production in ReallmCraft)
- **Implementation**: ReallmCraft RAG system

## Purpose

Improve semantic search retrieval quality by generating hypothetical questions that your content could answer, then storing both questions and answers in the vector database. At query time, match user questions to your hypothetical questions (question-to-question similarity) instead of matching questions to content descriptions (question-to-answer similarity).

**Core Insight**: Semantic similarity between two questions is higher than semantic similarity between a question and a descriptive answer.

## The Problem with Standard RAG

### Standard Approach (Poor Results)

**What you store**:
```
"The Sword of Kings is a legendary blade forged in dragon fire by the ancient smiths of Valdoria. It glows with an inner warmth and can cut through any material."
```

**User query**:
```
"What is the Sword of Kings?"
```

**Problem**: The question "What is the Sword of Kings?" has low semantic similarity to the descriptive text because they use different linguistic structures.

**[[User-veritasr]]'s Observation** (February 2024):
> "Semantic similarity isn't meant to locate stuff like keywords or key phrases, it's meant to find similar paragraphs of text."

## HyDE Solution: Two Approaches

### Approach 1: Hypothetical Questions

Generate questions your content answers, store questions alongside content.

#### Template

```
Given this information:
{{content}}

Generate {{num_questions}} hypothetical questions that this information could answer.

Requirements:
- Questions should be natural and varied (who, what, where, when, why, how)
- Each question should be answerable by the given information
- Avoid questions that require information not present in the content
- Format each question on its own line

Questions:
```

#### Variables

| Variable | Description | Example |
|----------|-------------|---------|
| {{content}} | The information to generate questions for | Lore entry, character description, location details |
| {{num_questions}} | How many questions to generate | 3-5 typical |

#### Usage Example

**Input Content**:
```
The Sword of Kings is a legendary blade forged in dragon fire by the ancient smiths of Valdoria. It glows with an inner warmth and can cut through any material. The blade was lost during the Fall of Valdoria and its current location is unknown.
```

**Prompt**:
```
Given this information:
The Sword of Kings is a legendary blade forged in dragon fire by the ancient smiths of Valdoria. It glows with an inner warmth and can cut through any material. The blade was lost during the Fall of Valdoria and its current location is unknown.

Generate 5 hypothetical questions that this information could answer.

Requirements:
- Questions should be natural and varied (who, what, where, when, why, how)
- Each question should be answerable by the given information
- Avoid questions that require information not present in the content
- Format each question on its own line

Questions:
```

**Expected Output**:
```
1. What is the Sword of Kings?
2. How was the Sword of Kings created?
3. What makes the Sword of Kings special?
4. Where is the Sword of Kings now?
5. Who forged the Sword of Kings?
```

**Storage Strategy**:
```python
# Store each question with its answer
for question in questions:
    vector_db.add(
        text=question,  # Embed the question
        metadata={
            'answer': original_content,
            'type': 'hypothetical_question',
            'source': 'lore_entry_sword_of_kings'
        }
    )

# Also store the original content for keyword matching
vector_db.add(
    text=original_content,
    metadata={
        'type': 'direct_content',
        'source': 'lore_entry_sword_of_kings'
    }
)
```

### Approach 2: Hypothetical Answer (Pure HyDE)

Generate a hypothetical answer to user's query, then search for similar content.

#### Template

```
A player in a {{genre}} game asked: "{{user_query}}"

Generate a plausible, detailed answer to this question based on common {{genre}} tropes and conventions. Your answer should be 2-3 sentences and include specific details that might appear in actual game content.

Do not say "I don't know" or qualify your answer. Commit to a specific answer even if it's speculative.

Answer:
```

#### Variables

| Variable | Description | Example |
|----------|-------------|---------|
| {{user_query}} | Player's question | "Where can I find healing potions?" |
| {{genre}} | Game setting | "dark fantasy", "sci-fi", "medieval" |

#### Usage Example

**User Query**:
```
"Where can I find healing potions?"
```

**Prompt**:
```
A player in a dark fantasy game asked: "Where can I find healing potions?"

Generate a plausible, detailed answer to this question based on common dark fantasy tropes and conventions. Your answer should be 2-3 sentences and include specific details that might appear in actual game content.

Do not say "I don't know" or qualify your answer. Commit to a specific answer even if it's speculative.

Answer:
```

**Expected Output**:
```
Healing potions can typically be purchased from alchemists, apothecaries, or general merchants in most towns and cities. You might also find them as loot in dungeons, or as rewards for completing quests. Some temples and churches offer healing potions to adventurers for a donation.
```

**Usage**:
```python
# Generate hypothetical answer
hypothetical_answer = llm.generate(hyde_prompt)

# Embed the hypothetical answer
query_vector = embed_model.encode(hypothetical_answer)

# Search for similar actual content
results = vector_db.similarity_search(
    query_vector=query_vector,
    k=10
)

# Return actual content that matches the hypothetical answer
return [r.metadata['actual_content'] for r in results]
```

## Hybrid Retrieval Pipeline

**[[User-veritasr]]'s Production Architecture**:

```python
def hybrid_retrieve(user_query, game_context):
    """
    Multi-stage retrieval combining multiple methods.
    """
    all_results = []

    # Stage 1: Exact name matching (highest priority)
    exact_matches = db.query_exact(user_query)
    all_results.extend(exact_matches)

    # Stage 2: Keyword/tag matching
    keywords = extract_keywords(user_query)
    tag_matches = db.query_tags(keywords)
    all_results.extend(tag_matches)

    # Stage 3: Hypothetical questions (question-to-question)
    question_embedding = embed_model.encode(user_query)
    question_matches = vector_db.query(
        query_vector=question_embedding,
        filter={'type': 'hypothetical_question'},
        k=10
    )
    all_results.extend([m.metadata['answer'] for m in question_matches])

    # Stage 4: HyDE (hypothetical answer to content)
    hyde_answer = generate_hypothetical_answer(user_query, game_context)
    hyde_embedding = embed_model.encode(hyde_answer)
    hyde_matches = vector_db.query(
        query_vector=hyde_embedding,
        filter={'type': 'direct_content'},
        k=10
    )
    all_results.extend(hyde_matches)

    # Stage 5: Graph traversal (related entities)
    for result in all_results[:5]:  # Top 5 only
        related = db.query_relationships(result.id)
        all_results.extend(related)

    # Deduplicate and rank using Reciprocal Rank Fusion
    ranked_results = reciprocal_rank_fusion(all_results)

    return ranked_results[:15]  # Return top 15
```

## Reciprocal Rank Fusion (RRF)

Combine results from multiple retrieval methods:

```python
def reciprocal_rank_fusion(results_lists, k=60):
    """
    RRF: Combine multiple ranked lists into single ranking.

    results_lists: List of lists of results from different methods
    k: Constant (typically 60)
    """
    scores = {}

    for results in results_lists:
        for rank, result in enumerate(results, start=1):
            result_id = result.id
            if result_id not in scores:
                scores[result_id] = {'result': result, 'score': 0}
            scores[result_id]['score'] += 1 / (k + rank)

    # Sort by score descending
    sorted_results = sorted(
        scores.values(),
        key=lambda x: x['score'],
        reverse=True
    )

    return [item['result'] for item in sorted_results]
```

## Effectiveness Notes

### What Works Well
- **Dramatically improves recall**: Finding relevant content 60-80% more often
- **Natural language queries**: Players can ask questions naturally
- **Cross-domain**: Works for lore, locations, characters, items, rules
- **Complementary**: Works alongside keyword and tag-based search
- **Model-agnostic**: Works with any embedding model
- **Cost-effective**: Only generation cost is hypothetical questions (one-time per content)

### Known Limitations
- **Computational overhead**: Generating questions adds processing time
- **Storage increase**: Storing 5 questions per content = 5x storage for that content
- **False positives**: Hypothetical questions may match unintended content
- **Question quality matters**: Poor questions = poor retrieval
- **Requires LLM**: Need GPT-3.5+ or similar for question generation
- **Embedding model dependency**: Results vary by embedding model quality

### Model-Specific Notes
- **GPT-3.5/GPT-4**: Best for generating high-quality hypothetical questions
- **Local 7B models**: Can generate questions but quality inconsistent
- **Embedding models**: all-MiniLM-L6-v2 works well for game content
- **ChromaDB/Pinecone/Weaviate**: All support metadata filtering for hybrid approach

### Community Feedback

**[[User-veritasr]]** (February 2024):
> "This is the thing I was imagining the other day. Good to know it has a name"

**On semantic search limitations**:
> "Semantic similarity isn't meant to locate stuff like keywords or key phrases, it's meant to find similar paragraphs of text."

**On hybrid approach**:
> "because semantic search by itself is sorta busted."

## Temperature Settings

**For Hypothetical Question Generation**:
- **Temperature**: 0.6-0.8 (need variety in question phrasing)
- **Top-p**: 0.9
- **Repetition Penalty**: 1.2 (avoid duplicate questions)

**For HyDE Answer Generation**:
- **Temperature**: 0.7-0.9 (want creative, detailed answers)
- **Top-p**: 0.9-0.95
- **Repetition Penalty**: 1.15

## Related Prompts

- [[retrieval/semantic-search-strategy|Semantic Search Strategy]] - Broader RAG patterns
- [[system/context-manager|Context Manager]] - Using retrieved results
- [[reasoning/chain-of-thought|Chain of Thought]] - Query decomposition
- [[constraint/anti-hallucination|Anti-Hallucination]] - Preventing fabricated answers

## Variations

### 1. Multi-Angle Questions

Generate different question types:

```
Generate questions about {{content}} in these categories:
1. Factual (what, where, who)
2. Causal (why, how)
3. Comparative (vs. what, similar to)
4. Temporal (when, how long)
5. Conditional (what if, suppose)

Format:
Category: Question
```

### 2. Player-Perspective Questions

Frame questions as player would ask them:

```
A player exploring {{location}} might wonder:
- What is this place?
- What can I do here?
- Who might I meet?
- What dangers exist?
- What rewards could I find?

Generate natural player questions about: {{content}}
```

### 3. Contextual HyDE

Include game state in hypothetical answer:

```
Player context: {{player_level}}, {{player_location}}, {{quest_status}}

Player asked: "{{query}}"

Generate an answer appropriate to their current game state.
Include specific locations, NPCs, and quest hooks they can access right now.
```

## Implementation Pattern

### Offline: Generate Questions for Content

```python
def preprocess_content_for_rag(content_item):
    """
    Run once per content item when adding to database.
    """
    # Generate hypothetical questions
    prompt = f"""
Given this information:
{content_item.text}

Generate 5 hypothetical questions that this information could answer.
Format each question on its own line.

Questions:
"""

    questions = llm.generate(prompt, temperature=0.7)
    question_list = questions.strip().split('\n')

    # Store each question with link to original content
    for question in question_list:
        vector_db.add(
            text=question.strip(),
            metadata={
                'type': 'hypothetical_question',
                'answer': content_item.text,
                'content_id': content_item.id,
                'source': content_item.source
            }
        )

    # Also store original content
    vector_db.add(
        text=content_item.text,
        metadata={
            'type': 'direct_content',
            'content_id': content_item.id,
            'source': content_item.source
        }
    )
```

### Online: Query Time

```python
def retrieve_for_query(user_query, top_k=10):
    """
    At query time, search hypothetical questions.
    """
    # Embed user query
    query_vector = embed_model.encode(user_query)

    # Search for similar hypothetical questions
    results = vector_db.similarity_search(
        query_vector=query_vector,
        filter={'type': 'hypothetical_question'},
        k=top_k
    )

    # Return the answers (original content)
    return [
        {
            'content': r.metadata['answer'],
            'score': r.score,
            'question_matched': r.text
        }
        for r in results
    ]
```

## Testing Checklist

- [ ] Generate questions for 50+ content items
- [ ] Test retrieval accuracy vs. standard RAG
- [ ] Measure retrieval latency (should be same)
- [ ] Test with various question phrasings
- [ ] Verify no duplicate questions stored
- [ ] Test edge case: content that's already a question
- [ ] Compare embedding models (MiniLM vs. others)
- [ ] Test with multi-stage hybrid retrieval
- [ ] Measure storage overhead
- [ ] Test recall improvement (should be 40-80% better)

## Performance Metrics

**Expected Improvements (from research literature + community testing)**:
- **Recall improvement**: 40-80% more relevant results found
- **Precision**: Similar to standard RAG (slightly better)
- **Query latency**: No change (same vector search operation)
- **Storage overhead**: 5-10x depending on questions per content
- **Preprocessing time**: +2-5 seconds per content item (one-time cost)

## Source

**Original Discussion**: Discord transcript lines 15949-16135 (February 11-12, 2024)
**Context**: [[User-veritasr]] exploring RAG improvements for ReallmCraft's memory system
**Key Quotes**:
> "This approach improves search quality due to a higher semantic similarity between query and hypothetical question compared to what we'd have for an actual chunk."

> "There is also the reversed logic approach called HyDE — you ask an LLM to generate a hypothetical response given the query and then use its vector along with the query vector to enhance search quality."

**Community consensus**:
> "semantic search by itself is sorta busted" - need hybrid approaches

## Related Documentation

- [[03-RAG-and-Memory]] - Comprehensive RAG discussion
- [[User-veritasr]] - Creator's other contributions
- [[01-Architecture-and-Design]] - Where RAG fits in system design

## Academic References

**HyDE Paper**: "Precise Zero-Shot Dense Retrieval without Relevance Labels" (Gao et al., 2022)
**Hypothetical Questions**: Common technique in question-answering systems
**RRF**: "Reciprocal Rank Fusion outperforms Condorcet and individual Rank Learning Methods" (Cormack et al., 2009)
