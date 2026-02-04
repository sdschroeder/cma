# Content Overlap and Consolidation Recommendations

**Purpose:** Identify where docs duplicate content and recommend what to combine, trim, or cross-reference for optimal organization.

> **Status:** These recommendations were **applied** (implementation plan slimmed, StoryCycle Genie consolidated and superseded docs archived, cross-references added). Kept for overlap rules ("what not to combine") and for future edits.

---

## 1. Overlap Map (Summary)

| Area | Documents | Overlap type | Recommendation |
|------|-----------|--------------|----------------|
| **CMA goals + three-tier** | authoritative-reference, implementation-plan | **Heavy duplication** | One canonical; other links |
| **Brightsy reality + gaps** | authoritative-reference, implementation-plan | **Partial duplication** | Authoritative reference is source; implementation-plan points to it |
| **Flywheel + Four Pillars + sciences** | technical-spec, whitepaper | **Concept duplication** | Technical spec = canonical; whitepaper = narrative (no merge) |
| **UIC definition + phases** | technical-spec, ultra-intelligent-content/* | **Spec vs narrative** | Technical spec = truth; UIC docs = extended narrative (no merge) |
| **StoryCycle Genie vision** | v2-executable-vision, token-optimization-plan, complete-vision-extension | **Progressive consolidation** | v2 = single reference; slim or archive the other two |

---

## 2. Detailed Overlap and What to Do

### 2.1 Authoritative Reference vs Implementation Plan

**Overlap:** Sections 1 and 2 of both docs are nearly identical.

- **§1 "What We're Trying to Do"** — Same core question (persist/retrieve content + metadata + strategic intelligence), same seven goals table (unified storage, graph-aware, semantic search, coherence, lightweight context, section-level content, agent-friendly).
- **§2 Three-tier architecture** — Same ASCII diagram (Memory → Intelligence Storage → Semantic), same tier bullets, same note on core vs extended in Tier 2.
- **§2.2 Proposed data model** — Same tables: `semantic_assets`, `asset_relationships`, `content_sections`, memory role.

**Authoritative reference unique:** Brightsy/Storycycle "today" (3.2, 3.3), full Gaps table (4), Needs/capabilities (4.1), phased implementation with Path A vs Path B (5), Brightsy capabilities (6), Success metrics (7), Risks (8), One-page summary (9), Path A vs Path B detail (10).

**Implementation plan unique:** More narrative on Memory (2.3), Semantic Search and Coherence (2.4), Agent Patterns (2.5), Section-Based Content/UX (2.6); Phase 1–3 step tables (3); "What Exists in Brightsy Today" (4) — shorter than authoritative reference; Alignment and Decisions (4.3); Success Metrics (5), Risks (6), Summary (7).

**Recommendation — Consolidate by reference:**

1. **Keep** `cma-authoritative-reference.md` as the single place for: "What we're trying to do," three-tier architecture, proposed vs current data model, gaps, Path A vs Path B, Brightsy/Storycycle reality, success metrics, risks.
2. **Slim** `cma+brightsy-implementation-plan.md`:
   - **Remove** §1 (What We're Trying to Do) and §2 (Three-Tier + Proposed Model) in full.
   - **Add** at top: "For goals, architecture, current state, and Path A/B see [cma-authoritative-reference.md](cma-authoritative-reference.md). This doc is the **Brightsy implementation plan** only: phased steps, decisions, and reality check."
   - **Keep** §2.3–2.6 only if they add implementation detail not in authoritative reference (e.g. agent patterns, section UX). Otherwise move that content into authoritative reference and delete from implementation plan.
   - **Keep** §3 (step tables), §4 (reality check — or replace with "See authoritative reference §3.2, §4" and keep only §4.3 Alignment and Decisions), §5–7 (metrics, risks, summary).

**Result:** No duplicated prose; one source of truth for "what and why"; implementation plan is a short planning artifact that links to it.

---

### 2.2 Technical Spec vs Whitepaper

**Overlap:** Same concepts, different audience and depth.

- **Cognitive Amplification Flywheel** — 6 steps (Emerge, Capture, Expand, Leverage, Codify, Amplify). Wording differs slightly; meaning same.
- **Three foundational sciences** — Marines/behavioral/flow research. Technical spec is terse; whitepaper has narrative and attribution.
- **Four Pillars** — Both list Framework Governance, Collective Intelligence, Intelligent Orchestration, Embedded Intelligence. Technical spec has implementation detail; whitepaper has value proposition and story.
- **CMA differentiator** — "Efficiency vs intelligence amplification." Both state it.

**Recommendation — Do not combine.**

- **Technical spec** = canonical definition for implementation and validation. Keep as single source of truth; Flywheel and Pillars are *defined* here.
- **Whitepaper** = thought leadership and business case. Keep for external/executive use; treat as *narrative version* of the same concepts.
- **Optional:** In technical spec, add one line: "A narrative version of Core Purpose, Flywheel, and Pillars for thought leadership is in [cma-whitepaper.md](../assets/cma-whitepaper.md)." In whitepaper, keep existing line that technical spec is authoritative for UIC/schemas. Ensure Flywheel and Pillar *names* and order are identical so they stay in sync.

---

### 2.3 Technical Spec vs Ultra-Intelligent Content Docs

**Overlap:** Technical spec states it contains "all UIC technical definitions"; UIC docs provide "extended narrative, examples, and platform-specific draft schemas."

- **UIC definition** (5 bullets: emerges from assets, semantic coherence, recomposition, cognitive flow, scales expertise) — In `ultra-intelligent-content.md` and echoed in technical spec.
- **UIC phases** (e.g. Phase 1 Foundation, Phase 2 Performance Learning) — In both technical spec and UIC overview.
- **Recomposition agents** — Listed in UIC overview; technical spec may reference or summarize.

**Recommendation — Do not merge UIC into technical spec.**

- **Technical spec** = single source of technical truth (schemas, phases, validation). Keep UIC *definitions* and *requirements* there.
- **ultra-intelligent-content.md** = narrative and evolution story. Keep as-is; ensure it doesn’t contradict the spec.
- **ultra-intelligent-content-tech-implementation.md** = assessment framework + schema JSON. Keep as extended implementation guide; technical spec can point to it for "full JSON and assessment questions."
- **ultra-intelligent-content-semantic-schema-framework.md** = detailed semantic schemas and two-tier patterns. Keep as platform-specific draft; technical spec points here for "detailed semantic schemas."

**Optional:** If tech-implementation and semantic-schema are always used together, merge them into one "UIC technical supplement" to reduce file count. Don’t merge the narrative overview into the supplement.

---

### 2.4 StoryCycle Genie: v2 vs Token Optimization vs Complete Vision Extension

**Overlap:** v2 positions itself as "the single reference" and repeats content from the other two.

- **Executive summary** — Same goal (token reduction, governance-first, full UIC/CMA), same metrics table (76K→8–10K, etc.), same approach bullets.
- **Architecture** — Base framework + MCP skills + tiers; same idea in all three.
- **Principles** — "Governance first," "CMA Four Pillars," "Test via StoryCycle MCP" in v2; same ideas in token-optimization and complete-vision.
- **Phases** — Token-optimization has phase list; complete-vision adds schema/recomposition/memory/lineage; v2 consolidates into one architecture and phase set.

**Recommendation — Treat v2 as the single implementation doc; slim or archive the others.**

1. **Keep** `storycycle-genie-v2-executable-vision.md` as the **single** StoryCycle Genie implementation reference. Ensure it contains any unique content from the other two (e.g. detailed phase tables from complete-vision-extension, specific skill lists from token-optimization).
2. **Slim** `storycycle-genie-token-optimization-plan.md` to a short "Design history / original optimization plan" doc: 1–2 paragraphs + link to v2. Or archive it (move to `docs/archive/` or add an "Superseded by storycycle-genie-v2-executable-vision.md" notice at the top).
3. **Slim** `storycycle-genie-complete-vision-extension.md` similarly: either fold unique sections (e.g. "What Belongs in the Base Framework" table, Phase 11–14 detail) into v2 and then replace this file with a short "Extension design notes" + link to v2, or add "Superseded by v2" and keep as reference only.
4. **Keep** `brightsy-review.md` (MCP tools, implementation notes) and `storycycle-record-types-summary.md` (38 record types) as-is; they’re supplements/reference.
5. **Keep** `storycycle-labs-strategic-framework.md` as strategic/market doc; no merge.

**Result:** One primary StoryCycle Genie doc (v2); no repeated executive summaries or architecture diagrams; history preserved in slim or archived docs.

---

### 2.5 Use Cases (Construction, Healthcare, Legal)

**Overlap:** Each applies "Four Pillars" to the industry. They don’t duplicate each other; they duplicate only the Pillar *framework* (from whitepaper/technical spec).

**Recommendation:** Keep as separate use-case docs. Add a single line at top of each: "Four Pillars are defined in [cma-technical-spec.md](…) and narrated in [cma-whitepaper.md](…)." No merging of use cases into the whitepaper.

---

### 2.6 Research

**Files:** `intelligent-content-modeling-ai.md`, `uic-content-model-alignment-review.md`.

- **Alignment review** explicitly references the research doc and maps it to CMA/UIC. No duplication of the research content.
**Recommendation:** Keep both; no merge.

---

## 3. Implementation Order (Optimal Organization)

1. **Authoritative reference vs implementation plan**  
   Slim implementation plan: remove duplicated §1–§2, add pointer to authoritative reference. Optionally move any unique implementation-plan-only detail (e.g. agent patterns) into authoritative reference so implementation plan is purely "steps + decisions + summary."

2. **StoryCycle Genie**  
   Audit v2 for missing content from token-optimization and complete-vision-extension; add that content to v2. Then add "Superseded by v2" (or archive) to token-optimization and complete-vision-extension.

3. **Cross-references**  
   Add one-line pointers where useful: technical spec → whitepaper (narrative); whitepaper → technical spec (authoritative); use cases → technical spec + whitepaper; implementation plan → authoritative reference.

4. **Optional**  
   Sync Flywheel and Four Pillars wording between technical spec and whitepaper (names and order identical; narrative stays in whitepaper only).

---

## 4. What Not to Combine

- **Whitepaper and technical spec** — Different audiences; combining would blur "spec" vs "narrative."
- **UIC overview and UIC tech/semantic docs** — Overview = narrative; tech/semantic = implementation/schema. Keep split.
- **Use cases** — Keep per-industry; don’t merge into one long doc.
- **Research and alignment review** — Keep separate; alignment review is the bridge.

---

## 5. One-Page Summary

| Action | Doc(s) | Result |
|--------|--------|--------|
| **Slim and link** | cma+brightsy-implementation-plan | Drop §1–§2; point to authoritative reference; keep steps, decisions, risks, summary. |
| **Keep as canonical** | cma-authoritative-reference, cma-technical-spec | No merge; they serve different roles (current state + paths vs full technical definition). |
| **Don’t merge** | technical-spec vs whitepaper, technical-spec vs UIC docs | Different audiences; spec = truth, others = narrative/extended. |
| **Single reference** | storycycle-genie-v2-executable-vision | Add any missing content from token-optimization and complete-vision-extension; supersede or archive those two. |
| **Keep separate** | Use cases, research, brightsy-review, record-types-summary | No consolidation. |

This gives you **optimal organization**: one place for CMA goals/architecture/current state (authoritative reference), one for technical truth (technical spec), one for Brightsy planning (slim implementation plan), one for StoryCycle Genie (v2), and no duplicated blocks of prose.
