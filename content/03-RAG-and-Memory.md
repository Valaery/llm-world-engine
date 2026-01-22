---
tags: [rag, memory, retrieval, vector-db, context-management, thread]
date: 2026-01-15
participants:
  - "[[User-veritasr]]"
  - "[[User-50h100a]]"
  - "[[User-underscore_x]]"
  - "[[User-monkeyrithms]]"
status: analyzed
---

# Thread: RAG and Memory Systems

## Summary

This thread documents the community's exploration of Retrieval-Augmented Generation (RAG) and memory systems for LLM game engines. The conversation evolved from simple vector databases to sophisticated hybrid retrieval systems combining multiple search methods.

## Core Challenge

**The Problem**: Context windows are limited, but game worlds are vast.

**The Question**: How do you give the LLM relevant information without overwhelming the context window?

## Three Main Approaches

### 1. RAG (Vector Database / Semantic Search)
- Store embeddings of text chunks
- Query by semantic similarity
- Return most relevant results

### 2. Boolean/Keyword Search
- Traditional keyword matching
- Tag-based filtering
- Exact name matching

### 3. Knowledge Graphs
- Entity relationships
- Graph traversal
- Connected information discovery

**Consensus**: Hybrid approach works best

## Early RAG Discussions

### Initial Vector Database Issues

**[[User-vali98]]** (January 2024):
> "from what Cohee has told me, vector DB and the likes suck"

**[[User-50h100a]]**:
> "vector db is hard to use"
> "more critically, vector db cannot interface directly with the llm"

**[[User-underscore_x]]**:
> "chroma is good at retrieval. but you have to make it about retrieving information"

### The Fundamental Issue

**[[User-50h100a]]**:
> "vector db can only help you as a search tool... using a vector database for the purpose of, say, letting the LLM write a sentence describing the category of things it wants to recall and then finding relevant 'memory' summaries would be appropriate"

**Key Insight**: Vector DBs are tools, not solutions. They retrieve; you still need to integrate intelligently.

## Vector Database Implementation

### [[User-veritasr]]'s Approach (ReallmCraft)

**Tech Stack**:
- ChromaDB for vector storage
- sentence-transformers (all-MiniLM-L6-v2) for embeddings
- 384-dimensional vectors

**Embedding Model Choice**:
- Small, fast, good enough
- Runs locally without issues
- Reasonable accuracy for game content

### Storage Strategy

**What Gets Embedded**:
- Location descriptions
- Character backstories
- Lore entries
- Item descriptions
- Event histories

**Metadata Stored Alongside**:
```json
{
  "id": "char_001",
  "name": "Vendrick the Cursed",
  "tags": ["player", "warrior", "cursed"],
  "type": "character",
  "last_seen": "2024-07-05",
  "relationships": ["npc_042", "location_15"]
}
```

## The Semantic Similarity Problem

### [[User-veritasr]]'s Critical Insight (February 2024)

> "Semantic similarity isn't meant to locate stuff like keywords or key phrases, it's meant to find similar paragraphs of text."

**Implication**: If you store "Vendrick is a warrior cursed by ancient magic," querying "warrior" won't match well because you're matching a word to a paragraph.

### Solution: Store Experiences, Not Descriptions

**Wrong Approach**:
```
Store: "The Sword of Kings is a legendary blade forged in dragon fire."
```

**Better Approach**:
```
Store: "Vendrick gripped the Sword of Kings, feeling the ancient dragon fire still warm within the blade. Its weight was familiar, comforting, a reminder of battles past."
```

**Why**: Narrative experiences provide richer semantic context than bare facts.

### Hypothetical Questions Approach

**Also Called**: HyDE (Hypothetical Document Embeddings)

**Technique**:
1. Take your content: "The Sword of Kings was forged in dragon fire"
2. Generate hypothetical questions it could answer:
   - "What is the Sword of Kings?"
   - "How was the Sword of Kings created?"
   - "What makes the Sword of Kings special?"
3. Store both questions and answers with embeddings
4. At query time, match user question to hypothetical questions

**[[User-veritasr]]** (February 2024):
> "This is the thing I was imagining the other day. Good to know it has a name"

## Hybrid Retrieval System

### [[User-veritasr]]'s Final Architecture

**Phase 1: Exact Matching**
```python
# Direct name match - highest priority
exact_matches = db.query(name=user_input)
context.add(exact_matches, priority=1)
```

**Phase 2: Tag-Based Boolean Search**
```python
# Extract keywords from input
keywords = extract_keywords(user_input)

# Find entries with matching tags
tag_matches = db.query(tags__contains=keywords)
context.add(tag_matches, priority=2)
```

**Phase 3: Semantic Search**
```python
# Vector similarity search on descriptions
semantic_matches = vector_db.similarity_search(
    query=user_input,
    k=10,
    threshold=1.5
)
context.add(semantic_matches, priority=3)
```

**Phase 4: Graph Traversal**
```python
# For each result, follow relationship links
for entity in current_results:
    related = db.query(id__in=entity.relationships)
    context.add(related, priority=4)
```

**Phase 5: Ranking and Fusion**
```python
# RRF-style ranking
all_results = combine_results(
    exact_matches,
    tag_matches,
    semantic_matches,
    related_entities
)

# Sort by average rank across methods
ranked = reciprocal_rank_fusion(all_results)

# Add to context until token limit reached
context.fill_until_limit(ranked)
```

## Context Window Management

### Token Estimation

**[[User-veritasr]]'s Method** (February 2024):

```python
import tiktoken
import math

class ContextBuilder:
    def __init__(self, prompt, messages, context_strings, max_length):
        self.context = context_strings
        self.prompt = prompt
        self.messages = messages
        # Padding for tokenizer differences (1/50 ratio)
        self.context_padding = math.floor(max_length / 50)
        self.max_length = max_length - self.context_padding

    def get_tokens(self, text, encoder="gpt-3.5-turbo"):
        encoding = tiktoken.encoding_for_model(encoder)
        return len(encoding.encode(text))
```

**Padding Strategy**: Reserve 1/50 tokens as safety buffer for tokenizer differences.

**Later Adjustment**: "1 in 50 might be too high, guess maybe it should be 1 in 25"

### Priority-Based Context Inclusion

**Order of Importance**:
1. System prompt (always included)
2. Current scene description
3. Active characters in scene
4. Recent messages (last N turns)
5. Exact match results (from queries)
6. Related entities
7. Semantic matches (by rank)
8. General world lore

**Implementation**:
```python
def build_context(self, max_tokens):
    remaining = max_tokens
    context = []

    # Add prompt (always fits)
    context.append(self.prompt)
    remaining -= self.get_tokens(self.prompt)

    # Add messages (recent first)
    for msg in reversed(self.messages):
        tokens = self.get_tokens(msg)
        if tokens <= remaining:
            context.insert(1, msg)
            remaining -= tokens
        else:
            break

    # Add context items by priority
    for item in sorted(self.context, key=lambda x: x.priority):
        tokens = self.get_tokens(item.text)
        if tokens <= remaining:
            context.append(item.text)
            remaining -= tokens
        else:
            print(f"Unable to add {item.name}, {remaining} tokens left")
            break

    return context
```

### [[User-veritasr]]'s Results

**Typical Context Size**: Under 3k tokens
**Max Context Tested**: 32k (with Mixtral MoE models)

> "Pretty satisfied with that fact so far. Context window has yet to break 3k"

## Memory Systems

### Iterative Summarization

**Pattern**: Chain summaries together to maintain continuity

**[[User-veritasr]]'s Approach**:
1. Break chat into message blocks (prompt + response pairs)
2. Summarize each block into events
3. Feed previous summary into next summary
4. Store final summary + all intermediate summaries in vector DB
5. Query summaries when needed

**Benefits**:
- Maintains narrative continuity
- Captures fine-grained details
- Doesn't lose information over time

**Trade-offs**:
- Takes longer (more LLM calls)
- Requires background processing
- Complexity in managing summary chains

### Hash-Based Invalidation

**Problem**: User edits or deletes messages, summaries become stale

**Solution** ([[User-veritasr]]):
```python
# Compute hash of message blocks
block_hash = hash(message_content)

# Only summarize if hash doesn't exist
if block_hash not in summary_db:
    summary = summarize(block)
    summary_db.save(block_hash, summary)
```

**Benefit**: Re-summarize only when content actually changes

### SummerWind Approach

**[[User-underscore_x]]**:
> "I'm assuming you're all coming at this from a low-context-local-model territory... chroma is good at retrieval."

**Their System**:
- Define proper template for CYOA continuation
- Create text structure to track character progress
- Pull text into variables and store somewhere
- Create logic to push relevant variables based on scene
- Build prompt dynamically

**Outcome**: "This is how I run summerwind and it's pretty much a video game"

## Retrieval Techniques

### RAKE-NLTK for Keyword Extraction

**[[User-veritasr]]** used RAKE (Rapid Automatic Keyword Extraction):

```python
from rake_nltk import Rake

rake = Rake()
rake.extract_keywords_from_text(description)
keywords = rake.get_ranked_phrases()
```

**Application**: Auto-generate tags for entries

**Process**:
1. Extract keywords from description
2. Run similarity search against all tags in DB
3. Filter results above threshold (1.5)
4. Suggest as tags for new entry

### Threshold Tuning

**Initial Settings**: 1.5 similarity threshold
**Observation**: Too restrictive for keywords, good for paragraphs
**Adjustment**: Different thresholds per content type

## Context Extension Methods

### [[User-50h100a]]'s Vision

> "ultimately, if external state tracking is reliable, it could be used more broadly as a context extension method"

**Concept**: Instead of cramming everything into context:
1. Store world state externally
2. Use function calling or keywords to query state
3. Inject query results mid-generation
4. LLM continues with new information

**Status**: "In lieu of realtime backprop" (not yet possible with current models)

### World Info Territory

**[[User-50h100a]]**:
> "in some sense this is approaching WorldInfo territory"
> "as large worlds or maps or character memories are impractical to cram into the prompt every time"

**SillyTavern's World Info**: Keyword-triggered lore injection
**This Project's Evolution**: Semantic + keyword hybrid

## Specific Implementation Details

### ChromaDB Configuration

**[[User-veritasr]]'s Setup**:
```python
import chromadb
from chromadb.config import Settings

client = chromadb.Client(Settings(
    chroma_db_impl="duckdb+parquet",
    persist_directory="./qdrant_data"
))

collection = client.create_collection(
    name="discord_chat",
    metadata={"hnsw:space": "cosine"}
)
```

**Storage Path**: `./qdrant_data` (local persistence)
**Distance Metric**: Cosine similarity
**Embedding Dimension**: 384 (all-MiniLM-L6-v2)

### Ranking and Fusion

**Reciprocal Rank Fusion (RRF)**:
```python
def reciprocal_rank_fusion(results_lists, k=60):
    """
    Combine multiple ranked result lists
    k is a constant (typically 60)
    """
    scores = {}
    for results in results_lists:
        for rank, item in enumerate(results, 1):
            if item.id not in scores:
                scores[item.id] = 0
            scores[item.id] += 1 / (k + rank)

    return sorted(scores.items(), key=lambda x: x[1], reverse=True)
```

**Application**: Combine exact matches, tag matches, and semantic matches into unified ranking

### Tag vs Description Weighting

**[[User-veritasr]]** (February 2024):
> "Technically this means that there's a higher probability of things with tags getting higher rank than those without, but I'll see how it performs."

**Observation**: Tags + descriptions work better than description alone

**Reason**: Tags provide keyword signal, descriptions provide semantic signal

## RAG Article Reference

**[[User-veritasr]]** shared (February 2024):
https://pub.towardsai.net/advanced-rag-techniques-an-illustrated-overview-04d193d8fec6

**Key Techniques Referenced**:

1. **Hypothetical Questions**: Generate questions for each chunk
2. **HyDE**: Generate hypothetical response and use for search
3. **Chunk Optimization**: Overlap chunks for better context
4. **Metadata Filtering**: Pre-filter by metadata before similarity search
5. **Query Rewriting**: Rephrase user query for better matches
6. **Re-ranking**: Two-stage retrieval (broad then narrow)

## Memory Extension

**[[User-veritasr]]'s Memory Extension** (mentioned, not fully detailed):
- Compared current text with stored summaries
- Retrieved relevant "memories"
- Pretty good results

**Use Case**: Long-running campaigns where events from weeks ago matter

## Scene-Based Context Management

### Current Scene Priority

**Pattern**: Prioritize information about current location and present characters

```python
def gather_context(self, user_input):
    context = []

    # Current scene (always include)
    context.append(f"Location: {self.current_location.description}")

    # Characters in scene
    for char in self.current_location.characters:
        context.append(f"Present: {char.summary}")

    # Now do retrieval for additional context
    retrieved = self.retrieval_system.search(user_input)
    context.extend(retrieved)

    return context
```

### Dynamic Context Updates

**On Scene Change**:
1. Clear low-priority context
2. Load new scene description
3. Load characters in new scene
4. Keep recent messages
5. Re-run retrieval with new location context

## Chat Context Management

**[[User-monkeyrithms]]** (February 2024):
> "context window management for long chats... since mine is splintered into a bunch of different 'rooms', I haven't actually stayed in one spot and just chatted the context all the way to max"

**Their Approach**: Scene-based context splitting
- Each location is a separate context space
- Moving between locations resets context
- Summaries carry over key information

**Issue**: "if that does happen, I think my app will crash"
**Solution**: Needed to implement context pruning

## Background Task Processing

**[[User-veritasr]]'s Design**:

```python
# Use separate LLM instance for background tasks
background_llm = LLM(endpoint="secondary_api")

# Generate summaries in background
async def summarize_in_background(messages):
    summary = await background_llm.summarize(messages)
    summary_db.save(summary)

# Doesn't block main gameplay
```

**Benefits**:
- Doesn't interrupt player
- Can use different/cheaper model
- Processes during idle time

**Preferred Model**: Flan-T5 or similar text2text model (better at summarization)

## Comparison: Vector vs World Info

### Traditional World Info (SillyTavern)
- Keyword triggers
- Exact match required
- Manual curation
- Simple, reliable
- No AI needed

### Vector/Semantic Approach
- Semantic similarity
- Fuzzy matching
- Can be automated
- More complex
- Requires embeddings

### Hybrid (This Project)
- Best of both worlds
- Exact matches get priority
- Semantic fills gaps
- Graph traversal for relationships

## Performance Considerations

### Embedding Generation Speed

**Local Models**: Fast enough for real-time
- sentence-transformers: ~100ms per chunk
- Can batch process for better throughput

### Vector Search Speed

**ChromaDB Performance**:
- Search 10k entries: ~50-100ms
- Acceptable for gameplay
- Can index/cache for speed

### Context Building Speed

**Bottleneck**: Token counting
- tiktoken is reasonably fast
- Can cache token counts
- Pre-compute when possible

## Lessons Learned

### What Worked

1. **Hybrid retrieval** better than pure vector
2. **Exact name matching** essential
3. **Priority-based context** prevents important info from being cut
4. **Token padding** (1/25 to 1/50) prevents overflow
5. **Store experiences not descriptions** for better semantic matches

### What Didn't Work

1. **Pure vector search** too unreliable
2. **Keyword-only** too rigid
3. **No thresholds** returned irrelevant results
4. **Same embedding for all content types** (descriptions vs experiences)
5. **Not considering relationships** missed obvious connections

### Ongoing Challenges

1. **Relevance ranking** still imperfect
2. **Query formulation** affects results
3. **Threshold tuning** requires experimentation
4. **Model-specific embeddings** don't transfer well
5. **Balancing recall vs precision**

## Integration with Game Loop

### Typical Flow

```
1. User inputs action
2. Extract entities mentioned (NPC names, locations, items)
3. Query RAG system with entities + action text
4. Retrieve relevant context
5. Build prompt with:
   - System instructions
   - Current scene
   - Retrieved context
   - Recent messages
   - User input
6. Generate response
7. Update world state
8. (Background) Summarize turn and update memory
```

### [[User-veritasr]]'s Context Ranking Implementation

```python
def gather_context(self):
    # Check all entity types
    location_tags = check_for_tags("location")
    location_desc = check_for_description("location")
    character_tags = check_for_tags("character")
    character_desc = check_for_description("character")
    # ... etc for events, items, lore

    # Rank each type
    location_rankings = rank_results(location_tags, location_desc, "location")
    character_rankings = rank_results(character_tags, character_desc, "character")
    # ... etc

    # Combine and sort
    overall = location_rankings + character_rankings + ...
    overall.sort(reverse=True)  # Highest scores first

    # Add to context until full
    for rank, item in overall:
        if item not in self.context:
            self.context.append(format_entry(item))
```

**Result**: "More relevant stuff will be added in before less relevant stuff"

## Related Threads

- [[01-Architecture-and-Design]] - How RAG fits in architecture
- [[02-Prompt-Engineering]] - Using retrieved context in prompts
- [[04-World-Generation]] - Generating content to populate RAG
- [[User-veritasr]] - Primary RAG implementer
- [[User-50h100a]] - Early RAG discussions

## Future Directions

- Attention mechanism integration
- Long-term memory consolidation
- Automatic relevance feedback
- Query expansion and refinement
- Multi-modal embeddings (text + metadata)
- Graph neural networks for relationships

## Key Quotes

> "Semantic similarity isn't meant to locate stuff like keywords or key phrases, it's meant to find similar paragraphs of text." - [[User-veritasr]]

> "vector db can only help you as a search tool" - [[User-50h100a]]

> "chroma is good at retrieval. but you have to make it about retrieving information." - [[User-underscore_x]]

> "It sort of feels like the industry is catching up to what we were doing in here _last_ year" - [[User-veritasr]] (July 2025)

## Related Threads

- [[01-Architecture-and-Design]] - How RAG fits into architecture
- [[02-Prompt-Engineering]] - RAG for prompt context
- [[05-State-Management]] - Retrieving persistent state
- [[User-veritasr]] - ChromaDB implementation
- [[User-underscore_x]] - Hybrid retrieval patterns

## Related Enrichment Outputs

### Prompt Library
- [[prompts/00-PROMPT-INDEX]] - Complete prompt library
- [[prompts/retrieval/query-formulation-hyde]] - HyDE for improved semantic search (40-80% better recall)

### Pattern Library
- [[patterns/00-PATTERN-INDEX]] - Complete pattern library

---

## Technical Resources Referenced

- **ChromaDB**: Vector database used by veritasr
- **Qdrant**: Alternative vector DB mentioned
- **sentence-transformers**: Embedding models
- **all-MiniLM-L6-v2**: Specific embedding model (384dim)
- **tiktoken**: Token counting library
- **RAKE-NLTK**: Keyword extraction
- **Advanced RAG Techniques**: TowardsAI article shared by veritasr
