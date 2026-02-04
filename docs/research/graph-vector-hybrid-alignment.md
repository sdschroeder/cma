# Graph and Vector: Alignment with Hybrid Agent Architecture Thinking

*Alignment of CMA/UIC architecture with the view that vectors and graphs are complementary—graph as structural foundation, vectors as retrieval layer—rather than competing alternatives.*

---

## The Perspective

When the conversation turns to AI agents, people often ask what's special about graph-based approaches compared to vectors. The stronger view:

- **Vectors and graphs are not mutually exclusive**—the strongest agent architectures are hybrid.
- **Vectors** retrieve similar text; **graphs** provide structure.
- **Vectors** carry narrative; **graphs** store facts.
- **Vectors** are subsymbolic pattern recognizers; **graphs** are symbolic world models.
- You can't search a graph using text, and you can't write individual parameters into a vector.

The mental model split: many developers see an app as interface + business logic + relational/NoSQL DB, and add RAG as a fourth layer (usually vectors); if a graph appears, it's another RAG layer. In that setup, vectors and graphs compete for "context formation." In contrast, when **graph is the primary data store**, multi-hop reasoning, semantic hierarchies, constraints, and transparent structure are in the core backend; **adding vectors is cheap and incremental**. The conclusion: **a graph should not be an addition to data storage—it should be the storage** (with vectors as an index layer over it).

---

## How Well We Align

### Strong alignment

| Principle | Our architecture |
|-----------|-------------------|
| **Vectors and graphs complementary** | Tier 2 = structure (lineage, asset_relationships, content_generation_lineage); Tier 3 = vectors (embeddings, similarity search). Both are first-class. |
| **Vectors retrieve similar; graphs provide structure** | Tier 3: "Find assets by meaning." Tier 2: Explicit lineage (`derived_from_assets`, `recomposed_from_content_id`, `adaptation_history`), relationship types (e.g. strategic_extension, strategic_application), semantic_connection_strength. |
| **Vectors carry narrative; graphs store facts** | Atomic components and embeddings carry narrative/semantic content; lineage and relationship records store who-derived-from-whom, preservation_score, adaptation_date. |
| **Vectors subsymbolic; graphs symbolic** | Embeddings are subsymbolic; relationship types, component types, and lineage are symbolic. |
| **Hybrid retrieval** | Spec states: hybrid retrieval (vector + metadata/coherence filters) with optional reranking; Discover step uses "Tier 3 semantic search and/or Tier 2 graph traversal." |

### Explicit alignment (after spec update)

- **Graph as structural foundation:** The spec now states that Tier 2 (Intelligence Storage) is the **structural foundation**: assets as nodes, lineage and asset_relationships as first-class (embedded or dedicated store). "The graph should not be an optional add-on to a document store; the relationship/lineage model is part of the core data model, and vectors augment retrieval over it."
- **Multi-hop and traversal:** Agents can "traverse the graph" (e.g. Brand Story → Audience Stories → Content Assets; content → originating_assets → related content) and "query by similarity"; hybrid retrieval combines both where supported.
- **Agent Interaction Pattern:** Discover step explicitly includes "Tier 2 graph traversal (lineage, asset_relationships, multi-hop from asset to derived/originating content)" alongside Tier 3 semantic search.

### Implementation nuance

- **v1 recommendation:** We still allow "embedded lineage" (lineage fields on records) as recommended for v1, with "separate asset_relationships records" as an option. So we support both entity-centric (embedded) and relationship-centric (separate store) implementations; the **relationship/lineage model** is mandatory, the **storage shape** (embedded vs. dedicated graph store) is implementation-dependent.
- **Target/conceptual model:** `semantic_assets` + `asset_relationships` tables make the graph explicit: nodes (assets) and edges (relationships). That aligns with "graph as storage" for implementations that choose a dedicated relationship store.

---

## Conclusion

We are **well aligned** with the thinking that vectors and graphs are complementary and that the strongest agent architectures are hybrid. We treat **structure (lineage, relationships) as part of the core data model** in Tier 2 and **vectors as an index layer** in Tier 3, with hybrid discovery (graph traversal + vector similarity) and explicit multi-hop reasoning. The spec now states this position clearly; implementations can satisfy it with embedded lineage (entity-centric) or with a dedicated relationship/graph store (graph-centric), as long as the relationship/lineage model is first-class and traversable.
