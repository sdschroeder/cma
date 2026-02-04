# Cognitive Mesh Architecture: Comprehensive Overview & Brightsy Implementation Plan

**Purpose:** Answer *"How to best save and access the content, metadata and strategic intelligence described in our collective intelligence ecosystem (Cognitive Mesh Architecture), with a focus on graph-based systems, content modeling, and optimization for agentic use."*

This doc is a single reference synthesized from your CMA spec. Use it as a planning resource; next step is to review it against what's **possible and practical in Brightsy**.

**Status:** This document describes the **proposed target** model (e.g. `semantic_assets` table). Current implementation uses **records + record_types** as the asset store; see [cma-authoritative-reference.md](cma-authoritative-reference.md) for Path A (records stay) vs Path B (migrate to semantic_assets).

---

## 1. What We're Trying to Do

### 1.1 The Core Question

How do we **persist and retrieve**:

- **Content** – brand stories, audience stories, content strategies, blog posts, whitepapers, social posts, etc.
- **Metadata** – themes, tone, audience, lineage, quality signals.
- **Strategic intelligence** – how pieces relate, how well they align to foundation assets, and how suitable they are for reuse and agentic workflows.

in a way that supports:

- **Agents** that create, edit, and orchestrate content using shared "foundational" assets (e.g. brand story).
- **Semantic and graph-aware retrieval** – by meaning and by explicit relationships.
- **Quality and coherence** – measurable alignment to strategy and to other high-value assets.
- **Reuse and repurposing** – e.g. sections of long-form content as first-class components.

That's the "collective intelligence ecosystem": **content + metadata + strategic intelligence**, stored and queried so both humans and agents can use it effectively.

### 1.2 High-Level Goals

| Goal | Description |
|------|-------------|
| **Unified intelligence storage** | One coherent model for "intelligence assets" (brand story, audience story, strategy, content pieces) with rich, queryable structure. |
| **Graph-aware access** | Explicit relationships between assets (derives_from, references, supports, contradicts) with strength and metadata. |
| **Semantic search** | Find assets by meaning (embeddings) and combine with filters (type, quality, status). |
| **Strategic coherence** | Scores for foundation alignment, cross-asset connectedness, and quality so agents can prefer high-coherence content. |
| **Lightweight context layer** | Memories hold pointers (e.g. `primary_brand_story_id`), thresholds (e.g. `strategic_coherence_minimum`), and preferences; heavy content lives in the DB. |
| **Section-level content** | Long-form = parent asset + ordered sections (each with content, metadata, embedding) for reuse, surgical editing, and section-level semantic search. |
| **Agent-friendly workflows** | Agents initialize from memories + DB; create/update assets; maintain relationships and scores; optionally hand off via events (e.g. Supabase Realtime). |

---

## 2. How We Propose to Accomplish It

### 2.1 Three-Tier Architecture (Conceptual)

```
┌─────────────────────────────────────────────────────────────────┐
│  TIER 1: MEMORY LAYER (Brightsy Memories)                        │
│  Lightweight context: preferences, thresholds, asset IDs        │
└─────────────────────────────────────────────────────────────────┘
                                    ↓
┌─────────────────────────────────────────────────────────────────┐
│  TIER 2: INTELLIGENCE STORAGE (Supabase PostgreSQL + JSONB)       │
│  Full assets: content, semantics, lineage, relationships        │
│  Explicit graph: asset_relationships                            │
│  Optional: section-based content_sections                       │
└─────────────────────────────────────────────────────────────────┘
                                    ↓
┌─────────────────────────────────────────────────────────────────┐
│  TIER 3: SEMANTIC LAYER (Cohere Embed + pgvector)               │
│  Embeddings per asset (and per section if section-based)         │
│  Similarity search + optional quality-aware behavior             │
└─────────────────────────────────────────────────────────────────┘
```

- **Tier 1** – Fast, small: what to load, how to behave, what "good" means.
- **Tier 2** – Authoritative: full content, metadata, and relationship graph.
- **Tier 3** – Retrieval: vector search over embeddings, optionally with coherence/quality filters.

*Schema docs (e.g. Ultra-Intelligent Content semantic framework) may use a two-tier **core vs extended** data pattern within Tier 2 for optimal query performance.*

### 2.2 Core Data Model (Proposed)

**Primary assets**

- **`semantic_assets`** (proposed new table)
  - Identity: `id`, `asset_type`, `status`.
  - Scores: `coherence_score`, `foundation_alignment_score`, `cross_asset_intelligence_score`, `quality_sophistication_score`.
  - JSONB: `content`, `semantics`, `lineage`, `relationships` (each &lt; ~2KB; depth 2–3).
  - Vector: `embedding vector(1024)` (Cohere).
  - Context: `user_id`, `team_id`, timestamps.

**Graph**

- **`asset_relationships`**
  - `source_id`, `target_id` → `semantic_assets`.
  - `relationship_type`, `strength` [0,1], `metadata` JSONB.
  - Unique on (source_id, target_id, relationship_type).

**Sections (optional but recommended for long-form)**

- **`content_sections`**
  - `parent_asset_id`, `section_order`, `section_type`, `title`.
  - `content` JSONB, `metadata` JSONB, `embedding vector(1024)`.
  - Enables section-level search, reuse tracking, surgical edits.

**Derived / performance**

- Materialized view for strategic coherence; GIN on JSONB; B-tree on scores; vector index (IVFFlat or HNSW) on `embedding`.

### 2.3 Memory System Role

- **Memories** (existing Brightsy memories) store: **pointers** (e.g. `primary_brand_story_id`), **thresholds** (e.g. `strategic_coherence_minimum`), **standards**, **preferences**.
- **Supabase** stores: **full asset payload** and **all relationship rows and embeddings**.

So: **memories = what to load and how to behave; database = content and graph.**

### 2.4 Semantic Search and Coherence

- **Embeddings:** Cohere Embed (e.g. v3/v4), 1024-dim; `search_document` vs `search_query`.
- **Search:** e.g. `semantic_search(query_embedding, min_coherence, limit, asset_type_filter)`.
- **Strategic coherence:** Implemented as **Strategic Asset Score** (per asset): component scores—brand_story_coherence (15%), audience_story_alignment (15%), narrative_structure_quality (10%), reference_network_strength (20%), content_playbook_compliance (10%), professional_intelligence_density (10%), strategic_context_preservation (5%), validation_status_score (10%), implementation_readiness (5%)—aggregated to total_score (0–100); computed on write and/or via materialized view.

### 2.5 Agent Patterns (Proposed)

1. **Initialize:** Read memories → load referenced assets from Supabase.
2. **Create content:** Semantic search for examples → generate → score → embed → insert asset + relationships → optionally update agent memory.
3. **Edit:** Section-based flow: load only affected sections → edit → update and re-embed only changed sections.
4. **Coordinate:** Events (e.g. Supabase Realtime) for handoffs (creation → SEO → QA).

### 2.6 Section-Based Content and UX

- **Storage:** Parent in `semantic_assets`; body in `content_sections`.
- **Presentation:** Progressive loading, TOC, section boundaries in edit mode.
- **Editing:** Click-to-edit or chat ("edit section 5") → load only those sections → apply edits → save and re-embed only changed sections.
- **Reuse:** Section usage tracking; section-level semantic search by theme/format.

---

3. Implementation Plan (Steps Only)
Phase 1: Foundation
Step	Focus	Deliverables
1	Schema	semantic_assets table (columns + JSONB + embedding); asset_relationships; pgvector extension; GIN, B-tree, vector indexes.
2	Memory integration	Naming conventions for CMA-related memories; scripts or docs for setting primary_brand_story_id, coherence_min, etc.; verify memory tool + Supabase lookup.
3	Cohere + search	Embedding generation aligned with existing Cohere usage; semantic_search (or equivalent) RPC with quality/type filters.
4	Validation	Seed data; load test ~1K assets; validate query latency and relevance.
Phase 2: Agent Architecture
Step	Focus	Deliverables
5	Agent bootstrap	Load memories → resolve asset IDs → load from semantic_assets (and relationships as needed); reuse existing auth/context.
6	Content-creation agent	Create content flow: semantic search for examples → generate → score → embed → insert asset + relationships.
7	Specialized agents	SEO/QA/brand-consistency agents (or equivalent); handoff via Realtime or queue.
8	E2E testing	Multi-agent flows; coherence and relationship consistency checks.
Phase 3: Sections and Optimization
Step	Focus	Deliverables
9	Section schema	content_sections (+ optional section_usage); migration path for existing long-form if any.
10	Section APIs and agents	Section CRUD; section-level semantic search; edit-by-section agent behavior.
11	Coherence view and caching	Materialized view for strategic coherence; refresh strategy; optional app-level cache for embeddings/retrievals.
12	Production hardening	RLS for semantic_assets / asset_relationships / sections; monitoring; docs.

---

## 4. What Exists in Brightsy Today (Reality Check)

### 4.1 Already There

- **Supabase + PostgreSQL:** Primary backend; RLS on accounts/agents.
- **pgvector:** Used for `record_chunks`, `chunks`, `memory_item_chunks`, `published_record_chunks`, `record_type_summaries`.
- **Cohere Embed:** Used in `RecordsService`, `MemoryService`, `RetrievalEngine`, `MaintenanceService` (e.g. `embed-v4.0`, `createEmbeddings`).
- **Memories:** `memories` table (name, value, level); enhanced: `memory_resources` → `memory_items` → `memory_item_chunks` (with embeddings). Tool handler exposes `remember` and related tools.
- **Records:** `record_types` (schema) + `records` (data JSONB) + `record_chunks` (embedding); NL search; published records and type summaries with embeddings.
- **Vector dimensions:** Mix of 1024 and 1536 (record_chunks 1536, record_type_summaries 1024, memory_item_chunks 1536). Cohere v2/embed-v4 supports configurable dimensions.

### 4.2 Not There Yet (Gaps vs CMA Spec)

- **No `semantic_assets`** – No unified "intelligence asset" table with content/semantics/lineage/relationships and coherence scores.
- **No `asset_relationships`** – No first-class graph table between content assets.
- **No strategic coherence** – No foundation/cross-asset/quality scoring or materialized view.
- **No section-based content** – Records are whole-document; no `content_sections` or section-level embeddings.
- **No HNSW** – Vector indexes are IVFFlat; spec suggests HNSW for lower latency at scale (optional).
- **No CMA-specific agent flows** – No "load brand story from memory ID → create content aligned to it" pattern; would build on existing tools and services.

### 4.3 Alignment and Decisions

- **Embedding dimensions:** Standardize (e.g. 1024 or 1536) for new CMA tables to match Cohere and existing usage.
- **Memory usage:** Use existing memory tools and levels for CMA pointers and thresholds; only conventions and docs needed.
- **Records vs semantic_assets:** Decide whether CMA is a new domain (new tables) or an extension of records; spec assumes new tables.
- **Realtime:** Use Supabase Realtime for agent handoffs if desired, or existing job/queue patterns.

---

## 5. Success Metrics (From Spec)

| Metric | Target |
|--------|--------|
| Semantic query latency | &lt; 200 ms uncached, &lt; 50 ms cached |
| Embedding generation | &lt; 500 ms per asset |
| Coherence calculation | &lt; 100 ms algorithmic |
| Multi-agent chain (3 agents) | &lt; 2 s |
| Cache hit rate (embeddings) | &gt; 80% (if cache added) |
| Concurrent agents | 20+ |
| Foundation alignment (approved) | &gt; 8.0 |
| Cross-asset connections (strategic) | &gt; 5 |

---

## 6. Risks and Open Points

- **Scope:** Full CMA is a large lift; consider "CMA-lite" (assets + relationships + semantic search only) first.
- **Schema evolution:** Document and version JSONB and relationship_type enums.
- **Dimension consistency:** Align new vector columns with existing Cohere usage.
- **RLS and multi-tenancy:** All new tables must follow Brightsy's account/team model.
- **Cost:** Cohere and LLM usage scale with content and agent activity; section-level edits reduce tokens per edit but add section-level embedding cost.

---

## 7. Summary: One-Page "What and How"

**What we're doing:**  
A **content and intelligence layer** where strategic assets (brand story, audience story, strategies, derived content) are stored with **rich metadata**, **explicit relationships**, and **embeddings**, and **memories** hold lightweight pointers and rules so **agents** can create, edit, and coordinate content in a coherent, graph- and semantic-aware way.

**How we're doing it:**

1. **Tier 1 (Memories):** Use existing Brightsy memories for asset IDs, coherence thresholds, and behavior rules.
2. **Tier 2 (Storage):** Add `semantic_assets` (and optionally `content_sections`) and `asset_relationships`; optional materialized view for coherence.
3. **Tier 3 (Semantic):** Use existing Cohere + pgvector; add semantic search over assets (and sections); optionally coherence/quality filters.
4. **Agents:** Initialize from memories + DB; create/update assets and relationships; use section-level edits for long-form; optionally coordinate via Realtime.
5. **Phasing:** Foundation → agent flows → sections and optimization → production hardening.

**Next step:**  
Review this in the context of **what's possible and practical in Brightsy** (existing records/memories/embeddings, product scope, timeline) and decide Phase 1 scope and whether to start with "CMA-lite" or target the full section-based + coherence view.

---

You can copy everything from the first `# Cognitive Mesh Architecture` heading through "full section-based + coherence view." and save it as a single markdown file (e.g. `cma-overview.md`) wherever you keep planning docs outside the repo.

If you want, the next step can be a **Brightsy-specific pass**: which parts of this are already supported, which need new code/tables, and what a minimal first phase (e.g. "CMA-lite") would look like in this codebase.
