# Storycycle Record Types – Summary (from Supabase)

**Project ID:** `ukccbetnnemxjrqpamld`  
**Source:** Supabase `record_types` joined with `accounts` where account name/slug ILIKE `%storycycle%`.

---

## 1. All Storycycle record types (38)

| Name | Slug | NL search |
|------|------|-----------|
| ABT Statement | abt-statement | ✓ |
| Agent Analysis | agent-analysis | ✓ |
| Agent Metadata | agent-metadata | — |
| Audience Story | audience-story | ✓ |
| Author Voice Profile | author-voice-profile | ✓ |
| Brand Intelligence Report | brand-intelligence-report | ✓ |
| Brand-Native Expert | brand-native-expert | ✓ |
| Brand Story | brand-story | ✓ |
| Brand Story Assessment | brand-story-assessment | ✓ |
| Brand Story Grade | brand-story-grade | ✓ |
| Campaign | campaign | ✓ |
| Content Asset | content-asset | ✓ |
| Content Calendar | content-calendar | ✓ |
| Content Enhancement | content-enhancement | ✓ |
| Content Marketing Strategy | content-marketing-strategy | ✓ |
| Content Playbook | content-playbook | ✓ |
| Customer Journey | customer-journey | ✓ |
| Email Campaigns | email-campaigns | — |
| Email Subscribers | email-subscribers | — |
| Email Templates | email-templates | — |
| Narrative Arc | narrative-arc | ✓ |
| Persona | persona | ✓ |
| Resend Templates | resend-templates | — |
| Sending Domains | sending-domains | — |
| Social Media Post | social-media-post | ✓ |
| Social Media Strategy | social-media-strategy | ✓ |
| Story | story | — |
| Strategic Asset Score | strategic-asset-score | — |
| Testimonial | testimonial | ✓ |
| User Content Preferences | user-content-preferences | ✓ |
| Webhook Events | webhook-events | — |
| Website Content | website-content | ✓ |

---

## 2. Schema patterns (CMA-relevant)

### 2.1 Strategic intelligence (shared pattern)

Several types use a **`strategic_intelligence`** object (title: “Strategic Intelligence Metadata”) with:

- **depth** – `enum`: `brief` \| `standard` \| `deep` \| `comprehensive` (“Collaborative Depth”)
- **risks** – array of strings (risk factors)
- **pivots** – array of `{ from, to, why, timestamp? }` (strategic direction changes)
- **decisions** – array of `{ type, what, why, timestamp?, alternatives_rejected? }` (strategic decisions)
- **tradeoffs** – array of `{ factor, resolution }`
- **captured_at** – date-time
- **completeness** – number 0–1 (“Capture Completeness”)
- **agent_context** – `{ agent_type, brand_name, session_date, agent_version }`
- **user_insights**, **timing_factors**, **user_expertise**, **expected_outcomes**, **market_conditions**, **success_indicators**, **business_priorities**, **competitive_factors**, **learning_objectives**, **resource_constraints**
- **specialist_insights** – `{ search, archetype, narrative, behavioral, intelligence }` (specialist agent contributions)

**Types observed with `strategic_intelligence`:** ABT Statement, Audience Story, Brand Story, Content Asset (and likely others in the full set).

### 2.2 Foundation vs derived (by slug/name)

- **Foundation-like:** Brand Story, Audience Story, ABT Statement, Persona, Content Marketing Strategy, Content Playbook, Social Media Strategy, Customer Journey, Narrative Arc.
- **Derived/content:** Content Asset, Social Media Post, Campaign, Content Enhancement, Website Content, Testimonial, Brand Story Grade, Brand Story Assessment, Content Calendar.
- **Operational:** Email Campaigns, Email Subscribers, Email Templates, Resend Templates, Sending Domains, Webhook Events, Agent Metadata.

### 2.3 Content Asset – sections and structure

**Content Asset** schema includes:

- **content_sections** – object, “Intelligent Content Sections”, “AI-detected content sections (only populated when intelligent_structure.auto_structured = true)”.
- **intelligent_structure** – object with `user_validated`, `auto_structured`, `original_word_count`, `structure_confidence`, `structure_analysis_date`.
- **compilation_metadata** – `auto_compile`, `last_compilation`, `modified_sections`, `section_separator`, `compilation_needed`.
- **abt_structure** – `and_section`, `but_section`, `therefore_section`.

So Storycycle already has **section-aware content** and **AI-driven structure** inside the Content Asset record type (stored in `records.data`), but not as a separate `content_sections` table with per-section embeddings like in the CMA spec.

### 2.4 Brand Story – lineage / related assets

- **abt_statements** – array of ABT objects (audience, reasoning, and/but/therefore statements).
- **primary_audiences** – array of audience objects (name, fears, challenges, aspirations, frustrations).
- **competitive_intelligence_cache** – analysis expiry, last analysis date, competitor content themes, thought leadership opportunities.

No explicit **relationship_type** or **strength** to other records; relations are embedded structures and arrays inside the record.

### 2.5 ABT Statement

- **strategic_intelligence** – same shape as above.
- **variations** – array of `{ abt_text, variant_name }`.
- **behavioral_principles** – array of strings.
- Core fields: **brand_name**, **abt_statement**, **communication_type**, **audience**, **goal**, **and_component**, **but_component**, **therefore_component**.

### 2.6 Audience Story

- **strategic_intelligence** – same shape.
- **brand_archetypes** – primary/secondary with name, traits, reasoning, description, sub_archetype, application_notes, audience_resonance; **archetype_blend_dynamics**; **archetype_expression_channels** (email, website, social_media, etc.).
- **descriptors** – offering, outcomes, organization (arrays).
- **reasoning** – overall_strategy, archetype_selection, audience_adaptation, emotional_connection, positioning_rationale.
- Core: **brand_name**, **audience_name**, **abt_statement**, **narrative**, **emotional_promise**, **purpose_statement**, **position_statement**, **unique_value_proposition**.

---

## 3. CMA gap vs “what’s there”

| CMA concept | Storycycle today |
|--------------|-------------------|
| **Unified semantic_assets table** | Many record types; content + metadata + **strategic_intelligence** live in `records.data` by convention. No single “intelligence asset” table. |
| **Explicit asset_relationships (graph)** | No graph table. Relations are embedded (e.g. Brand Story ↔ ABT statements, audiences) or could be has-one/has-many to other *records*; no relationship_type/strength. |
| **Strategic coherence scores** | No columns for foundation_alignment_score, cross_asset_intelligence_score, quality_sophistication_score. **Strategic Asset Score** is a separate record type, not a computed score on assets. |
| **Section-level storage** | **Content Asset** has **content_sections** and **intelligent_structure** inside JSONB; no separate `content_sections` table or section-level embeddings in DB. |
| **Memory conventions** | Not visible from schema; would need to check memory names/values for “primary_brand_story_id”, etc. |

So: **metadata and strategic intelligence are already in the schemas** (especially `strategic_intelligence` and section/structure fields on Content Asset). The main gaps are: first-class **graph table**, **coherence score columns/views**, and **section-level table + embeddings** if you want full CMA-style section search and reuse.

---

## 4. What the schemas tell us (no extra data needed)

### 4.1 Foundation referenced by record ID?

**All 38 Storycycle record type schemas were scanned** for Brightsy relationship fields: `recordType`, `has-one`, `has-many`, `foreignKey`. **None of the types use them.** So foundation/related assets are **not** linked via has-one/has-many to other record types. They are either:

- **Embedded** – e.g. Brand Story’s `abt_statements`, `primary_audiences` (arrays/objects inside the record), or  
- **By name** – e.g. `brand_name` (string), not a record ID.

So we don’t need to “check sample data” for relationship IDs: the schema says there are no relation fields to other record types.

### 4.2 How “Strategic Asset Score” works (from schema)

**Strategic Asset Score** is a **separate record type** whose schema defines:

- **asset_id** (string) – “ID of the scored content asset”
- **asset_type** (string)
- **total_score** (0–100), **calculation_date**, **calculation_trigger** (e.g. manual_request, content_update, batch_processing)
- **score_tier** – e.g. “Strategic Foundation Asset”, “High Strategic Value”, “Good Strategic Alignment”, …
- **Component scores** (percentages in parentheses):  
  **brand_story_coherence** (15%), **audience_story_alignment** (15%), **narrative_structure_quality** (10%), **reference_network_strength** (20%), **content_playbook_compliance** (10%), **professional_intelligence_density** (10%), **strategic_context_preservation** (5%), **validation_status_score** (10%), **implementation_readiness** (5%)
- **foundation_analysis**, **intelligence_analysis**, **overall_assessment**, **quality_analysis**, **top_improvement_area**, **improvement_recommendations**
- **algorithm_version** (default 1.0.0)
- Plus the usual **strategic_intelligence** metadata object.

So coherence/scoring is **already modeled as a separate record** that points at an asset by `asset_id` and stores the breakdown. CMA-style “scores on the asset” could be either columns on the asset record or this pattern (separate score record).

### 4.3 What schemas don’t tell us

- **Memory usage** – Schema doesn’t define memory keys (e.g. `primary_brand_story_id`). That’s app/agent convention.
- **Section-level product need** – Whether you want section-level *search* and *reuse* (CMA-style) or are fine with Content Asset’s JSONB sections is a product decision, not in schema.
- **Agent flows** – Whether Genie “loads brand story from memory then creates content” is behavior, not schema.

---

## 5. Full schemas (JSON) for four types

The full `schema` JSON for these types was returned by the Supabase query and is available in the MCP response:

- **ABT Statement** (`abt-statement`)
- **Audience Story** (`audience-story`)
- **Brand Story** (`brand-story`)
- **Content Asset** (`content-asset`)

To get full schema JSON for any other Storycycle record type, run:

```sql
SELECT name, slug, schema
FROM public.record_types rt
JOIN public.accounts a ON a.id = rt.account_id
WHERE (a.name ILIKE '%storycycle%' OR a.slug ILIKE '%storycycle%')
  AND rt.slug = 'your-slug-here';
```
