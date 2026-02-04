# StoryCycle Genie: Complete Vision Integration Extension

*Extending the Unified Token Optimization & Content Consolidation Plan to realize full Ultra-Intelligent Content and Cognitive Mesh Architecture in this iteration*

---

## Purpose

This document adds **concrete phases and deliverables** to the [storycycle-genie-token-optimization-plan.md](storycycle-genie-token-optimization-plan.md) so that **this iteration** delivers:

1. **Semantic content output schema** — Content saved with Ultra-Intelligent metadata and atomic components for recomposition.
2. **Recomposition workflow** — Multi-agent flow (Context Analysis → Component Selection → Semantic Recombination → QA) and MCP/orchestrator support.
3. **Strategic Memory** — Systematic use of existing memory infrastructure with creation/retrieval triggers and evolution.
4. **Cross-asset intelligence** — Relationship and lineage tracking so intelligence compounds across assets.

Implementation stays within **Brightsy + StoryCycle MCP**: records for storage, existing memory tools, and the same orchestrator/specialist pattern. No separate “Phase 2” deferral.

**Governance-first principle:** Schemas and data architecture are part of the **governance layer** (Framework Governance pillar). They must be defined and baked into the base framework and skills *before* or *as* we implement agents, so every agent enforces the same data contract by design—not as a bolt-on.

---

## Schema & Data Architecture in the Governance Layer

Schemas and data architecture are **governance concerns**. If they live only in separate docs or "Phase 11," agents will be built without a shared contract and we will bolt validation on later. Instead:

1. **Define** the canonical schemas and data rules once (UIC content schema, lineage/relationship, selective retrieval profiles, memory conventions).
2. **Encode** them in the **base framework** and **skills** so that "how to read/write records" and "what every content save must include" are part of the framework every agent loads.
3. **Implement** agents and skills that *follow* that governance; no agent is written without the schema and data architecture already in its governance context.

### What Belongs in the Base Framework (storycycle-base-optimized.md)

The base framework must include (or reference) so that *all* agents operate under the same data contract:

| Governance element | What goes in base |
|--------------------|-------------------|
| **Canonical schema references** | Where to find UIC content schema, atomic component shape, and two-tier (core vs extended) rules. Short spec or pointer; full spec can live in a separate doc linked from base. |
| **Content record contract** | Every content record MUST include: `ultra_intelligent_metadata` (creation_context, strategic_intelligence_preservation, atomic_components), and lineage fields (`intelligence_assets_referenced` or `derived_from_assets`). Pre-response validation checklist includes "output conforms to content record contract." |
| **Selective retrieval profiles** | Which `select` fields to use for Brand Story, Playbook, Audience Story (and any other strategic assets) so retrieval aligns with schema—e.g. only request fields that exist and are needed for creation/recomposition. |
| **Lineage & relationship rules** | On create: set `intelligence_assets_referenced` / `derived_from_assets` from current context. On recompose: set `recomposed_from_content_id` and append `adaptation_history`. No content save without lineage when assets were used. |
| **Memory conventions** | Canonical memory names/levels (e.g. `cma.primary_brand_story_id`, user/team/agent) and when to retrieve at task start / when to consider creating (creation/retrieval triggers). |
| **Validation gate** | Before any content save: "Does this payload satisfy the content record contract?" If not, do not persist; return validation failure and required fields. |

So **Phase 2 (Base Framework)** in the optimization plan should be extended to include these schema and data-architecture elements. Schema definition (Phase 11 below) therefore **feeds Phase 2** and should be done in parallel with or immediately after base framework extraction—not after agents exist.

### What Belongs in Skills

Skills implement the governance contract for their domain:

| Skill type | Schema/data responsibility |
|------------|----------------------------|
| **strategic-intelligence-capture** (tier-3) | How to build `ultra_intelligent_metadata` and `atomic_components` from current context and generated content; required fields and shapes; validation before save. |
| **selective-field-profiles** (tier-3) | Field lists per strategic asset type that align with canonical schema (no ad-hoc `select`; use governance-approved profiles). |
| **Content-type skills** (e.g. blog-post, email-core) | Which atomic component types this type produces (e.g. key_insight, proof_element, cta); type-specific validation rules; any type-specific lineage (e.g. email sequence parent/child). |
| **Recomposition skills** (Phase 12) | How to read `atomic_components` and `strategic_intelligence_preservation`; how to write `recomposed_from_content_id` and `adaptation_history` on output. |

No skill should invent its own record shape; all must conform to the schema and data architecture defined in the base framework.

### Implementation Order (Governance Before Agents)

1. **Define** schemas and data architecture (UIC content schema, lineage, relationships, memory conventions)—this is the "schema definition" deliverable from Phase 11.
2. **Bake into base framework** (Phase 2): Add the sections above to `storycycle-base-optimized.md` so the governance layer includes schema and data rules.
3. **Bake into skills** (Phase 3): strategic-intelligence-capture, selective-field-profiles, and content-type skills reference and enforce the contract.
4. **Implement orchestrators and agents** (Phases 5–6+): They load base + skills and therefore already "know" the schema and data architecture; no separate "add schema later" step.

So in the **optimization plan**, Phase 2 (Base Framework) and Phase 3 (Skills) should explicitly include "schema and data architecture as governance." The **complete vision extension** phases (11–14) then align as: Phase 11 (schema definition) feeds Phase 2; Phase 13 (memory) feeds Phase 2; Phase 14 (lineage) feeds Phase 2 and capture logic; Phase 12 (recomposition) builds on governance that is already in place.

---

## Architecture Additions (Complete Vision)

```
┌─────────────────────────────────────────────────────────────────────────┐
│ EXISTING: LAYER 1–4 (Base, MCP Skills, Specialists, Strategic Assets)   │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│ NEW: CAPTURE AT CREATION                                                 │
├─────────────────────────────────────────────────────────────────────────┤
│ Every content save writes ultra_intelligent_metadata + atomic_components│
│ to record (Core in record.data; Extended refs or nested per Brightsy)    │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│ NEW: RECOMPOSITION ORCHESTRATOR + MCP                                    │
├─────────────────────────────────────────────────────────────────────────┤
│ • Recomposition flow: Context Analysis → Component Selection →          │
│   Semantic Recombination → QA (via specialists + skills)                │
│ • MCP: get_atomic_components, recompose_content (or equivalent)          │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│ NEW: STRATEGIC MEMORY LAYER                                              │
├─────────────────────────────────────────────────────────────────────────┤
│ • Creation/retrieval triggers in base + orchestrators                    │
│ • Conventions: primary_brand_story_id, coherence_min, user preferences   │
│ • Evolution: Session → User → Team → Agent → Framework Protocol          │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│ NEW: CROSS-ASSET INTELLIGENCE                                            │
├─────────────────────────────────────────────────────────────────────────┤
│ • Asset relationship records (or fields): derives_from, supports, etc.  │
│ • Content lineage + adaptation_history on content records                │
│ • Optional MCP: list_asset_relationships, get_content_lineage            │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## Phase 11: Semantic Content Output Schema & Capture at Creation

**Goal:** Every piece of content created by the Content Creation Orchestrator is stored with Ultra-Intelligent metadata and atomic components so it can be recomposed later without re-running the full creation pipeline.

### 11.1 Schema Definition (Brightsy-Optimized)

Use **two-tier** where needed (per [ultra-intelligent-content-semantic-schema-framework.md](ultra-intelligent-content-semantic-schema-framework.md)):

**Tier 1 — Core (in `record.data` or Content Asset type):**

| Field | Purpose |
|-------|---------|
| `ultra_intelligent_metadata.creation_context` | originating_agent, coordinating_agents, intelligence_assets_referenced |
| `ultra_intelligent_metadata.strategic_intelligence_preservation` | brand_positioning_elements, content_strategy_alignment, semantic_meaning_markers |
| `ultra_intelligent_metadata.atomic_components` | Array of { component_type, content, semantic_tags, recomposition_constraints, adaptation_contexts } — keep array small (e.g. top 5–15 per content) |
| `multi_agent_coordination_data.coordinating_agent_roles` | Snapshot of which specialists participated |

**Tier 2 — Extended (optional separate record or nested):**

- Detailed adaptation rules per component.
- Full relationship map (derives_from, supports_components); or reference a separate Asset Relationship record.

### 11.2 Capture-at-Creation in Content Creation Orchestrator

- **Before save:** Orchestrator (or a dedicated “Semantic Capture” step) produces:
  - `creation_context` from current session (orchestrator + specialists called).
  - `strategic_intelligence_preservation` from Brand Story / Playbook / Audience references (already in context).
  - `atomic_components`: extract key insights, proof elements, and CTAs from the generated content; tag and add constraints (from [ultra-intelligent-content-tech-implementation.md](ultra-intelligent-content-tech-implementation.md) Core Content Intelligence Schema).
- **Write:** Same `create_record` / update flow used today; add the above to the content record’s `data` (or linked UIC metadata record if you split).
- **Skill:** Add or extend `strategic-intelligence-capture.md` (tier-3-workflows) with “UIC metadata and atomic component extraction” so every content type skill expects this structure on save.

### 11.3 Deliverables

- [ ] **UIC content schema doc** — One-page spec: field names, types, and two-tier split for Brightsy (aligned with existing Content Asset / record type).
- [ ] **Updated strategic-intelligence-capture skill** — Instructions for generating and attaching `ultra_intelligent_metadata` and `atomic_components` on save.
- [ ] **Orchestrator change** — Call capture logic before every content save; persist to record.
- [ ] **Validation** — New content records include `ultra_intelligent_metadata` and non-empty `atomic_components` where applicable.

---

## Phase 12: Recomposition Workflow & MCP

**Goal:** Support “recompose this content for [new format/context]” using stored atomic components and strategic constraints, via a defined multi-agent flow and MCP.

### 12.1 Recomposition Flow (Aligned with [ultra-intelligent-content.md](ultra-intelligent-content.md))

1. **Context Analysis** — New context (platform, audience, objective); assess constraints and cognitive flow.
2. **Component Selection** — From stored `atomic_components`, select and rank by relevance and preservation requirements.
3. **Semantic Recombination** — Rebuild content for target format (e.g. blog → social series, email → slides) preserving strategic meaning; use Narrative/Behavioral/Search specialists as needed.
4. **Quality Assurance** — Validate against Framework Governance and strategic_intelligence_preservation (brand voice, message hierarchy, proof elements).

### 12.2 Implementation Options

**Option A — Recomposition Orchestrator (new agent):**

- New agent: `content-recomposition-orchestrator.md` (~4K tokens).
- Loads skills: `recomposition-context-analysis.md`, `recomposition-component-selection.md`, `recomposition-semantic-recombination.md`, `recomposition-qa.md` (tier-3-workflows or new tier).
- Uses `connectedAgentRequest` for Narrative/Behavioral/Search where needed; reads content record + `atomic_components` via existing record MCP/tools.

**Option B — Mode of Content Creation Orchestrator:**

- Add “Recomposition” path: user says “Turn this [content] into [format].”
- Orchestrator loads recomposition skills (same as above), fetches content + UIC metadata via MCP, runs the same four steps, then saves new content with its own UIC metadata and lineage pointing to source.

### 12.3 MCP Additions (StoryCycle MCP)

- **`get_atomic_components(record_id)`** — Returns `ultra_intelligent_metadata.atomic_components` (and optional strategic_intelligence_preservation) for a content record.
- **`recompose_content(record_id, target_format, context)`** — Can either:
  - Return atomic components + constraints for the orchestrator to run the flow, or
  - Trigger the full recomposition flow and return new content + new record id.

Implement whichever fits your MCP design (tool returns data vs. side-effect).

### 12.4 Deliverables

- [ ] **Recomposition skills** — At least: context-analysis, component-selection, semantic-recombination, qa (each ~150–250 lines).
- [ ] **Recomposition orchestrator or mode** — Single agent or Content Creation Orchestrator path; uses specialists and writes UIC metadata on output.
- [ ] **MCP tools** — `get_atomic_components`; `recompose_content` (or equivalent).
- [ ] **Test scenarios** — e.g. blog → 3 social posts; email sequence → 5 slides; validate strategic preservation (manual or rubric).

---

## Phase 13: Strategic Memory Integration

**Goal:** Use Brightsy’s existing memory infrastructure (user/team/agent) with clear creation/retrieval triggers and evolution so that cognitive flow and preferences compound (per [cma-technical-spec.md](cma-technical-spec.md) Strategic Memory Utilization).

### 13.1 Conventions (Document in Base Framework)

- **Pointers:** e.g. `primary_brand_story_id`, `primary_audience_story_id` (memory name/value or record IDs in memory).
- **Thresholds:** e.g. `strategic_coherence_minimum`, `confidence_auto_approve_threshold` (from dual-process-selection).
- **Preferences:** e.g. “step-by-step vs automated for complex frameworks,” “comprehensive vs minimal options” (from CMA examples).

Store in existing memory tools with a **naming convention** (e.g. `cma.primary_brand_story_id`, `cma.preference.workflow_mode`).

### 13.2 Creation Triggers (When to Remember)

- Pattern detection: user repeatedly chooses step-by-step for complex tasks → create/update user memory.
- After handoffs: optional “what context was preserved” summary for agent memory.
- After strategic work: e.g. “brand story approved” → update team or user memory for default brand.
- Thresholds: when user overrides auto-approve, consider updating confidence threshold memory.

Implement in **base framework** or orchestrator instructions: “After [X], consider calling remember with [convention].”

### 13.3 Retrieval Triggers (When to Recall)

- Task initiation: load primary_brand_story_id, primary_audience_story_id, preferences.
- Agent handoffs: pass relevant memory keys so next agent has context.
- Strategic work: ensure coherence_min and approval thresholds are in context.
- QA: “Does this match stored preferences for tone/options?”

Implement in **base framework**: “Before starting content creation, retrieve cma.* memories and inject into context.”

### 13.4 Evolution Framework (Document Only for This Iteration)

- **Session discovery** → **User memory** (individual patterns).
- **User memory** → **Team memory** (shared standards) when validated (e.g. admin or repeated pattern).
- **Agent memory** (learned behaviors) updated when agents succeed/fail patterns.
- **Framework protocol** — Base/skills updated when patterns become standard.

No need to automate evolution in v1; document the intended flow and manually promote a few examples.

### 13.5 Deliverables

- [ ] **Memory conventions doc** — Names, levels (user/team/agent), and when to create/retrieve (one-page).
- [ ] **Base framework update** — Retrieval at task start; “consider remember” at creation trigger points.
- [ ] **Orchestrator update** — Pass memory-derived context (brand id, preferences) into specialist calls.
- [ ] **Evolution framework** — Short section in cma-technical-spec or governance doc describing Session → User → Team → Agent → Framework.

---

## Phase 14: Cross-Asset Intelligence & Lineage

**Goal:** Track relationships and lineage so “intelligence compounds” and recomposition/QA can use “which assets this derives from” and “what was adapted from this.”

### 14.1 Asset Relationships

**Option A — New record type “Asset Relationship”:**

- Fields: `source_asset_id`, `target_asset_id`, `relationship_type` (e.g. derives_from, supports, references), optional `strength` or metadata.
- Created when: content is created from Brand Story + Playbook (derives_from); or when recomposition creates new content from existing (derives_from).

**Option B — Embedded in existing records:**

- On Content Asset: `derived_from_assets: [brand_story_id, audience_story_id, playbook_id]`.
- On content created by recomposition: `recomposed_from_content_id`, `adaptation_history: [{ target_format, date, preservation_score }]`.

Choose based on Brightsy query needs; Option B is minimal and often enough for v1.

### 14.2 Lineage and Adaptation History

- **Creation:** When saving content, set `intelligence_assets_referenced` (or `derived_from_assets`) from current context (brand, audience, playbook).
- **Recomposition:** When saving recomposed content, set `recomposed_from_content_id` and append to `adaptation_history` (target_format, date, optional preservation_score).

This satisfies “content generation lineage” and “adaptive recomposition history” from [ultra-intelligent-content-tech-implementation.md](ultra-intelligent-content-tech-implementation.md).

### 14.3 Optional MCP

- **`list_asset_relationships(record_id)`** — Returns derived_from, supports, etc. (from relationship records or embedded fields).
- **`get_content_lineage(record_id)`** — Returns chain: source assets → this content → any recomposed children.

Useful for “show me everything derived from this brand story” or “what was adapted from this blog post.”

### 14.4 Deliverables

- [ ] **Schema decision** — Relationship record type vs embedded fields; document in UIC schema doc.
- [ ] **Orchestrator + Recomposition** — Write `derived_from_assets` / `intelligence_assets_referenced` on create; write `recomposed_from_content_id` + `adaptation_history` on recompose.
- [ ] **Optional MCP** — `list_asset_relationships`, `get_content_lineage` if useful for UI or agents.
- [ ] **Cross-asset section** — One-page “Cross-Asset Intelligence” in governance or schema doc (what we store, how we use it).

---

## Integration with Existing Plan

| Existing phase | Integration point |
|----------------|--------------------|
| **Phase 2 (Base framework)** | **Governance layer must include schema and data architecture:** canonical schema references, content record contract, selective retrieval profiles, lineage/relationship rules, memory conventions, validation gate (see "Schema & Data Architecture in the Governance Layer" above). Add retrieval/creation triggers and memory conventions (Phase 13). Schema definition (Phase 11) feeds Phase 2—do it in parallel with or immediately after base extraction so agents are never built without the contract. |
| **Phase 3 (Skills)** | **Skills must enforce the governance contract:** strategic-intelligence-capture builds and validates UIC metadata + atomic components (Phase 11); selective-field-profiles use governance-approved `select` lists; content-type skills specify atomic component types and validation. Add recomposition skills (Phase 12). No skill invents its own record shape. |
| Phase 5 (Content type skills) | Each content type skill specifies which atomic component types to emit (e.g. key_insight, proof_element, cta) and conforms to content record contract from base. |
| Phase 6 (Content Creation Orchestrator) | Loads base (which already includes schema contract); before save: run UIC capture per strategic-intelligence-capture skill; write lineage/derived_from (Phases 11, 14). Optional: add Recomposition mode (Phase 12). |
| Phase 8 (Deploy) | Deploy schema changes, base framework with governance, new skills, new orchestrator/mode, MCP tools, memory conventions. |
| MCP (Brightsy Review) | Add get_atomic_components, recompose_content; optional list_asset_relationships, get_content_lineage. |

---

## Success Criteria (Complete Vision)

- **UIC schema:** All new content (and recomposed content) has `ultra_intelligent_metadata` and `atomic_components` (or equivalent) per schema.
- **Recomposition:** At least one path (e.g. blog → social) works end-to-end with strategic preservation; QA step runs.
- **Memory:** At least two creation triggers and two retrieval triggers implemented; conventions doc exists.
- **Lineage:** New content has `derived_from_assets` or equivalent; recomposed content has `recomposed_from_content_id` and adaptation_history.
- **CMA:** All four pillars explicitly used: Governance (base + skills), Collective Intelligence (assets + UIC + lineage), Orchestration (orchestrator + recomposition + specialists), Embedded Intelligence (specialists + skills). Strategic Memory documented and partially implemented.

---

## Order of Execution (Suggested)

**Governance first:** Schema and data architecture must be in the base framework and skills *before* we implement orchestrators and agents, so every agent enforces the same data contract by design.

1. **Schema definition (Phase 11)** — Define UIC content schema, lineage/relationship fields, two-tier split for Brightsy. This is the canonical spec that governance will encode.
2. **Phase 2 (Base framework)** — Extract base as in optimization plan *and* add governance sections: content record contract, selective retrieval profiles, lineage rules, memory conventions, validation gate. Reference or embed the schema from step 1. Phase 13 (memory conventions) feeds this.
3. **Phase 14 (lineage)** — Decide relationship record type vs embedded fields; document in schema doc. Lineage/relationship rules are already in base (step 2); this phase implements the write rules and optional MCP.
4. **Phase 3 (Skills)** — Extract skills as in optimization plan *and* ensure strategic-intelligence-capture, selective-field-profiles, and content-type skills enforce the contract from base. No skill invents its own record shape.
5. **Phase 12 (Recomposition)** — Recomposition skills + orchestrator/mode + MCP (depends on schema and base being in place).
6. **Phases 5–6+ (Orchestrators/agents)** — Implement Content Creation Orchestrator (and others) so they load base + skills and therefore already "know" the schema and data architecture.

This order keeps token optimization intact while ensuring schema and data architecture are accounted for in the governance layer when we implement the agents.
