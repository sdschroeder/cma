# Knowledge Graph Repo Review: Architecture Considerations for CMA

**Source:** [rahulnyk/knowledge_graph](https://github.com/rahulnyk/knowledge_graph) — "Convert any Corpus of Text into a Graph of Knowledge"

**Purpose:** Evaluate this approach against CMA's collective intelligence architecture. Determine if there's anything to learn or adopt.

---

## What This Repo Does

A notebook-based project that converts a text corpus (PDFs) into a visual knowledge graph using LLM-driven extraction. The pipeline:

1. **Chunk** — Split documents into ~1500-character chunks using LangChain's `RecursiveCharacterTextSplitter` (150-char overlap)
2. **Extract** — For each chunk, prompt a local LLM (Mistral 7B via Ollama) to extract pairs of concepts and their relationships as structured JSON:
   ```json
   {"node_1": "A concept", "node_2": "A related concept", "edge": "relationship in one or two sentences"}
   ```
3. **Contextual proximity** — Concepts appearing in the same chunk are automatically linked with a "contextual proximity" edge, weighted by co-occurrence count across chunks
4. **Merge** — Combine LLM-extracted relationships (weight 4) with contextual proximity edges (weight 1), group by concept pair, sum weights
5. **Graph algorithms** — Community detection (Girvan-Newman) for clustering, degree centrality for node sizing
6. **Visualize** — Interactive HTML graph via pyvis/NetworkX

Key design choice: extracts *concepts* ("Pleasant weather in Bangalore") rather than *entities* ("Bangalore"). The author notes concepts make more meaningful graphs.

---

## How CMA Compares — Dimension by Dimension

| Dimension | This Repo | CMA | Assessment |
|-----------|-----------|-----|------------|
| **Graph nodes** | LLM-extracted concepts from arbitrary text | Expert-validated intelligence assets (Brand Story, Audience Story, Content Asset, etc.) with rich metadata and UIC structure | CMA nodes are first-class business objects, not extracted text fragments. Fundamentally different purpose. |
| **Relationships** | LLM-generated natural language labels ("continues to lag behind") + contextual proximity | Typed, structured relationships (`derives_from`, `references`, `supports`, `contradicts`) with strength scores [0,1] | CMA's relationships are explicit, typed, and weighted — more reliable and queryable than LLM-generated labels. |
| **Relationship creation** | Automatic (LLM extraction + co-occurrence) | Explicit (set at creation via governance contract: `intelligence_assets_referenced`, `derived_from_assets`) + planned `asset_relationships` table | CMA captures *intent and lineage*, not just surface co-occurrence. |
| **Semantic layer** | None — purely structural graph | Full Tier 3: Cohere embeddings + pgvector + similarity search + hybrid retrieval | CMA has vector search; this repo does not. |
| **Quality scoring** | None | Strategic Asset Score (9 weighted components, 0-100 scale) | CMA has quality-aware content discovery; this repo has none. |
| **Agent integration** | None — batch notebook | Full agent-driven architecture: orchestrators, specialists, memory, MCP skills | CMA is built for real-time agent use; this repo is offline processing. |
| **Storage** | Pandas DataFrames + CSV files | PostgreSQL + pgvector (Supabase), planned `asset_relationships` table | CMA has production-grade storage; this repo is research-grade. |
| **Metadata** | Chunk ID only | UIC content record contract: `ultra_intelligent_metadata`, `atomic_components`, lineage, `multi_agent_coordination_data` | CMA captures strategic reasoning and professional intelligence, not just text structure. |
| **Community detection** | Girvan-Newman algorithm for node clustering | Not implemented (noted in semantic graph implementation guide as a graph algorithm requiring custom code or Neo4j) | This repo has it; CMA documents it as a future enhancement. |
| **Contextual proximity** | Automatic co-occurrence signal: concepts in the same chunk are linked | Not implemented — relationships are explicit only | Interesting technique worth noting. See below. |

---

## What's Worth Noting

### 1. Contextual Proximity as an Automatic Relationship Signal (Low Priority)

**What it is:** When two concepts appear in the same text chunk, they're automatically linked with a "contextual proximity" edge. The weight increases with co-occurrence count across chunks. This creates implicit relationships without explicit human or LLM labeling.

**CMA equivalent:** CMA doesn't have an automatic co-occurrence signal. Relationships are explicit — set at creation time through lineage fields or (planned) `asset_relationships`. If two content assets share themes but aren't explicitly linked, there's no mechanism to surface that implicit connection.

**Could this matter for CMA?** In theory, you could analyze content assets post-creation to find shared atomic components, shared semantic meaning markers, or similar embedding proximity, and suggest relationships that weren't captured through explicit lineage. For example: "These two blog posts share 4 atomic components but aren't linked — should they be?"

**Why it's low priority:** CMA already has a better mechanism. The UIC content record contract captures `intelligence_assets_referenced` and `atomic_components` at creation time — this is the *intent* and *strategic reasoning* behind the content, not just surface-level concept overlap. And Tier 3 semantic search already finds similar content by meaning. Automatic relationship suggestion would be a "nice to have" Phase 5+ enhancement, not a gap.

### 2. Community Detection for Content Clustering (Low Priority)

**What it is:** The repo uses Girvan-Newman community detection to identify clusters of related concepts. This groups nodes that are more densely connected to each other than to the rest of the graph.

**CMA equivalent:** The semantic graph implementation guide documents this as a graph algorithm that PostgreSQL doesn't provide natively (would need custom code or Neo4j). Not currently planned.

**Could this matter for CMA?** Community detection could identify thematic clusters of content — "these 5 blog posts, 2 case studies, and 1 whitepaper form a coherent content cluster about Topic X." This could inform content strategy, identify coverage gaps, or help with content organization.

**Why it's low priority:** CMA's content is already organized by type (brand story → audience story → content playbook → content) and linked by explicit lineage. The value of automatic clustering is marginal when explicit relationships and semantic search already serve the discovery need. This is a Phase 4+ graph algorithm enhancement documented in the [semantic graph implementation guide](cma-semantic-graph-implementation-guide.md), contingent on either custom PostgreSQL code or a Neo4j addition.

### 3. Concepts vs. Entities (Already Addressed)

**What it is:** The repo deliberately extracts "concepts" (higher-level ideas like "Pleasant weather in Bangalore") rather than "entities" (proper nouns like "Bangalore"), arguing concepts make more meaningful graphs.

**CMA equivalent:** CMA's `atomic_components` in UIC already capture concept-level abstractions — `key_insight`, `proof_element`, `cta`, etc. These are richer than either concepts or entities because they include `semantic_tags`, `recomposition_constraints`, and `adaptation_contexts`. CMA is already operating at the concept level, with more structure.

---

## Assessment: Our Approach Is Sound

This repo is an educational proof-of-concept for LLM-driven graph construction. It validates the general pattern (chunk text → extract concepts → build graph), but CMA's architecture is significantly more sophisticated in every dimension that matters:

**Where CMA is already ahead:**
- Intelligence assets as nodes (not LLM-extracted fragments)
- Typed, weighted relationships (not LLM-generated natural language)
- Governance-enforced metadata at creation (not post-hoc extraction)
- Embeddings + semantic search (this repo has none)
- Quality scoring and coherence (this repo has none)
- Agent integration and memory (this repo is a batch notebook)
- Production-grade storage (PostgreSQL vs. CSV files)

**Where this repo has a technique CMA doesn't:**
- Automatic contextual proximity edges (low value for CMA — explicit lineage + semantic search already serve this need better)
- Community detection (documented as a future enhancement in the semantic graph guide, not a current gap)

**The fundamental difference:** This repo builds a graph *from* text (information extraction). CMA builds a graph *of* intelligence (strategic content with professional reasoning captured at creation). These serve different purposes. The repo's approach would be like extracting concepts from a brand story to build a graph — but CMA's approach is to make the brand story itself a first-class node in the graph, with its relationships, lineage, and strategic metadata preserved. That's a more valuable graph for agents and professionals.

**No changes to the current CMA spec or implementation plan are recommended.**

---

**Document version:** Review of rahulnyk/knowledge_graph repository
