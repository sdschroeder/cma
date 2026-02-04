# CMA + Brightsy Implementation Plan

**Purpose:** Brightsy-specific implementation plan: phased steps, decisions, reality check, and summary. Use this doc for planning and scoping; it does not repeat goals, architecture, or current-state detail.

**For goals, architecture, current state, and Path A/B:** See [cma-authoritative-reference.md](../spec/cma-authoritative-reference.md) (§1 What We're Trying to Do, §2 Three-Tier Architecture, §3 Data Model & Brightsy/Storycycle Today, §4 Gaps, §5–10 Implementation Approach & Path A vs Path B).

**Status:** This document describes the **proposed target** model (e.g. `semantic_assets` table). Current implementation uses **records + record_types** as the asset store; see [cma-authoritative-reference.md](../spec/cma-authoritative-reference.md) for Path A (records stay) vs Path B (migrate to semantic_assets).

---

## 1. Implementation Plan (Steps Only)

**Phase 1: Foundation**

| Step | Focus | Deliverables |
|------|-------|--------------|
| 1 | Schema | semantic_assets table (columns + JSONB + embedding); asset_relationships; pgvector extension; GIN, B-tree, vector indexes. |
| 2 | Memory integration | Naming conventions for CMA-related memories; scripts or docs for setting primary_brand_story_id, coherence_min, etc.; verify memory tool + Supabase lookup. |
| 3 | Cohere + search | Embedding generation aligned with existing Cohere usage; semantic_search (or equivalent) RPC with quality/type filters. |
| 4 | Validation | Seed data; load test ~1K assets; validate query latency and relevance. |

**Phase 2: Agent Architecture**

| Step | Focus | Deliverables |
|------|-------|--------------|
| 5 | Agent bootstrap | Load memories → resolve asset IDs → load from semantic_assets (and relationships as needed); reuse existing auth/context. |
| 6 | Content-creation agent | Create content flow: semantic search for examples → generate → score → embed → insert asset + relationships. |
| 7 | Specialized agents | SEO/QA/brand-consistency agents (or equivalent); handoff via Realtime or queue. |
| 8 | E2E testing | Multi-agent flows; coherence and relationship consistency checks. |

**Phase 3: Sections and Optimization**

| Step | Focus | Deliverables |
|------|-------|--------------|
| 9 | Section schema | content_sections (+ optional section_usage); migration path for existing long-form if any. |
| 10 | Section APIs and agents | Section CRUD; section-level semantic search; edit-by-section agent behavior. |
| 11 | Coherence view and caching | Materialized view for strategic coherence; refresh strategy; optional app-level cache for embeddings/retrievals. |
| 12 | Production hardening | RLS for semantic_assets / asset_relationships / sections; monitoring; docs. |

---

## 2. What Exists in Brightsy Today (Reality Check)

For full **Brightsy today** (records, record_chunks, memories, Cohere) and **Storycycle today** (38 record types, strategic_intelligence, gaps), see [cma-authoritative-reference.md](../spec/cma-authoritative-reference.md) §3.2, §3.3, and §4.

### Alignment and Decisions

- **Embedding dimensions:** Standardize (e.g. 1024 or 1536) for new CMA tables to match Cohere and existing usage.
- **Memory usage:** Use existing memory tools and levels for CMA pointers and thresholds; only conventions and docs needed.
- **Records vs semantic_assets:** Decide whether CMA is a new domain (new tables) or an extension of records; spec assumes new tables.
- **Realtime:** Use Supabase Realtime for agent handoffs if desired, or existing job/queue patterns.

---

## 3. Success Metrics (From Spec)

| Metric | Target |
|--------|--------|
| Semantic query latency | < 200 ms uncached, < 50 ms cached |
| Embedding generation | < 500 ms per asset |
| Coherence calculation | < 100 ms algorithmic |
| Multi-agent chain (3 agents) | < 2 s |
| Cache hit rate (embeddings) | > 80% (if cache added) |
| Concurrent agents | 20+ |
| Foundation alignment (approved) | > 8.0 |
| Cross-asset connections (strategic) | > 5 |

---

## 4. Risks and Open Points

- **Scope:** Full CMA is a large lift; consider "CMA-lite" (assets + relationships + semantic search only) first.
- **Schema evolution:** Document and version JSONB and relationship_type enums.
- **Dimension consistency:** Align new vector columns with existing Cohere usage.
- **RLS and multi-tenancy:** All new tables must follow Brightsy's account/team model.
- **Cost:** Cohere and LLM usage scale with content and agent activity; section-level edits reduce tokens per edit but add section-level embedding cost.

---

## 5. Summary: One-Page "What and How"

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

You can copy the full narrative (goals, architecture, current state) from [cma-authoritative-reference.md](../spec/cma-authoritative-reference.md) plus this plan and save as a single markdown file (e.g. `cma-overview.md`) wherever you keep planning docs outside the repo.
