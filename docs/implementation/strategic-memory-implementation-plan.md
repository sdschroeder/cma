# Strategic Memory Utilization: Implementation Plan

**Purpose:** Practical, buildable plan for implementing strategic memory utilization in StoryCycle Genie V2. This document defines what to build, in what order, and exactly what goes into the base framework, orchestrators, and skills.

**Relationship to V2:** This plan provides the specific content for the "Memory conventions" deliverable in V2 Phase 1, the "Add memory conventions" step in V2 Phase 2 (Base Framework), and the "Memory Triggers" test criteria in V2 Phase 9. It also extends memory utilization into V2 Phases 5-7 (orchestrator, skills) and defines post-V2 enhancements.

**Background:** [strategic-memory-utilization-exploration.md](../research/strategic-memory-utilization-exploration.md)

---

## Design Decisions

These decisions resolve the open architecture questions from the exploration and are required before implementation.

### Decision 1: Memory Granularity — Structured Objects per Category

**Decision:** Store CMA memories as **one structured object per category**, not one memory item per individual key.

**Rationale:** Each memory retrieval adds latency and token overhead. Grouping related values into a single object reduces retrieval calls from 10+ down to 3-4 while keeping categories cleanly separated.

**Structure:**

| Memory Name | Level | Contains |
|-------------|-------|----------|
| `cma.user.context` | User | `primary_brand_id`, `primary_audience_ids`, `active_campaign_id` |
| `cma.user.preferences` | User | `workflow_mode`, `detail_level`, `option_presentation`, `tone`, `content_length` |
| `cma.team.standards` | Team | `default_brand_story_id`, `default_audience_story_ids`, `coherence_minimum`, `confidence_auto_approve`, `content_approval_required`, `required_specialists` |
| `cma.team.patterns` | Team | `preferred_content_types`, `successful_content_patterns`, `brand_voice_standards` |
| `cma.agent.effectiveness` | Agent | `specialist_sequence_outcomes`, `specialist_value_by_content_type`, `foundation_load_priorities` |
| `cma.agent.learning` | Agent | `common_user_corrections`, `avg_revisions_by_type`, `high_value_creation_sequences`, `anti_patterns` |

**Example memory item:**
```json
{
  "name": "cma.user.context",
  "level": "user",
  "value": {
    "primary_brand_id": "abc-123",
    "primary_audience_ids": ["def-456", "ghi-789"],
    "active_campaign_id": null
  }
}
```

**Impact on retrieval:** At session start, the agent calls retrieve for `cma.user.*` and `cma.team.*` — 4 retrievals total instead of 15+.

### Decision 2: Multi-Brand Scoping — Brand ID in Context, Not in Key Names

**Decision:** Use `cma.user.context.primary_brand_id` to scope other memories, not `cma.brand.{id}.preferences.*`.

**Rationale:** Most users work with one brand at a time. The context memory already captures which brand is active. Agent behavior adjusts based on context, not by loading brand-specific memory trees. If a user switches brands, the context memory updates and the agent adjusts.

**Exception:** If a user works across brands *within a single session* regularly, we revisit this. For v1, single-brand-per-session is the expected pattern.

### Decision 3: Process Intelligence — Memory for Patterns, Records for Logs

**Decision:** Agent execution *patterns* live in memory (lightweight, fast). Raw execution *logs* live in records (structured, queryable).

| Data | Vehicle | Why |
|------|---------|-----|
| "Narrative → Behavioral averages 8.5 for blogs" | Agent memory | Pattern. Agents need this at session start. Fast lookup. |
| "Session X used Narrative → Behavioral, produced score 8.7, user revised 2 times" | Record (optional) | Log entry. Useful for analytics but not needed at session start. |

**For v1:** Implement memory patterns only. Raw execution logging is a future enhancement (record type: "Agent Execution Log").

### Decision 4: Memory Expiration — Category-Based Rules

**Decision:**

| Category | Expiration |
|----------|------------|
| `cma.user.context` | **Session-scoped.** Updated at the start of every session when user confirms brand. No automatic expiration, but always overwritten. |
| `cma.user.preferences` | **Persistent.** Only updated when new patterns detected. Never expires — preferences are long-lived. |
| `cma.team.standards` | **Persistent.** Only updated by admin/lead actions. |
| `cma.team.patterns` | **Persistent with review.** Patterns can become stale. Include `last_validated` timestamp. Manual review quarterly. |
| `cma.agent.effectiveness` | **Persistent with decay.** Include observation count. Patterns based on < 5 observations are weighted lower than patterns based on 50+. |
| `cma.agent.learning` | **Persistent with pruning.** Anti-patterns and corrections that haven't been observed in 90 days are candidates for removal. |

**For v1:** Implement as persistent. Add `last_updated` timestamps to all memory objects. Expiration logic is a future enhancement.

### Decision 5: Promotion Authority — Admin for Team, Automated for Agent

**Decision:**

| Promotion | Authority | Mechanism |
|-----------|-----------|-----------|
| Session → User Memory | Automatic (agent) | Agent detects pattern (3+ occurrences) and stores |
| User → Team Memory | Admin/lead | Admin reviews user patterns and promotes to team standard |
| Pattern → Agent Memory | Automatic (post-scoring) | After content scoring, agent correlates process → outcome |
| Any → Framework Protocol | Human review | Requires explicit decision to update base/skills |

**For v1:** Only Session → User and Pattern → Agent are automated. Team promotion is manual. Framework promotion is always manual.

---

## Implementation Phases

### Phase A: Conventions and Base Framework

**When:** Part of V2 Phase 1 (Schema Definition) and Phase 2 (Base Framework Extraction)

**Goal:** Define the memory contract and bake retrieval/creation instructions into the base framework so every agent knows how to use memory from day one.

#### A.1 Memory Conventions Document

This is the "memory conventions doc" deliverable from V2 Phase 1. It defines the contract.

**Deliverable: Section in `framework/uic-content-schema.md` or standalone `framework/memory-conventions.md`**

Content:

```markdown
## Memory Conventions

### Naming
All CMA memories use the `cma.` prefix with dot-separated categories:
- `cma.user.context` — Working context (brand, audiences, campaign)
- `cma.user.preferences` — Workflow and content preferences
- `cma.team.standards` — Quality thresholds and policies
- `cma.team.patterns` — Organizational patterns and voice standards
- `cma.agent.effectiveness` — Specialist and process effectiveness data
- `cma.agent.learning` — Corrections, revision patterns, anti-patterns

### Levels
- **User:** Individual context and preferences. Created/updated by agent.
- **Team:** Organizational standards and patterns. Standards set by admin; 
  patterns promoted from user observations.
- **Agent:** Process intelligence. Updated automatically after content scoring.

### Values
All CMA memories store structured JSON objects, not single values.
Always include `last_updated` timestamp in the value.

Example:
  name: "cma.user.context"
  value: {
    "primary_brand_id": "abc-123",
    "primary_audience_ids": ["def-456"],
    "active_campaign_id": null,
    "last_updated": "2026-02-06T12:00:00Z"
  }
```

#### A.2 Base Framework Memory Section

This goes into `storycycle-base-optimized.md` as a governance section alongside the content record contract, selective retrieval profiles, and validation gate.

**Deliverable: "Strategic Memory Protocol" section in base framework (~300-400 tokens)**

Content:

```markdown
## Strategic Memory Protocol

### At Session Start (Retrieval)
Before starting any content work, retrieve CMA memories:

1. Call retrieve for "cma.user.context" — get primary brand, audiences, campaign
2. Call retrieve for "cma.user.preferences" — get workflow mode, detail level, tone
3. Call retrieve for "cma.team.standards" — get coherence minimum, approval policy
4. Call retrieve for "cma.team.patterns" — get preferred content types, voice standards

If cma.user.context has a primary_brand_id, use it to load the brand story 
directly (skip brand search). If no user context exists, fall through to 
brand identification protocol.

If cma.team.standards has a coherence_minimum, use it as the quality bar for 
this session. Default: 8.0.

If cma.user.preferences has a workflow_mode, use it to set the initial 
interaction mode (step-by-step vs automated). Default: follow 
dual-process-selection confidence logic.

### At Brand Confirmation (Creation)
When user confirms or selects a brand:
- Call remember with name "cma.user.context", updating primary_brand_id
  and primary_audience_ids from the confirmed brand's associated audiences.
- Include last_updated timestamp.

### At Content Approval (Creation)
When user approves final content:
- If this is the 3rd+ time user has chosen the same workflow mode for this 
  content type, call remember with name "cma.user.preferences" to update 
  workflow_mode.
- If user made corrections of a repeated type (same correction 3+ times 
  across sessions), update cma.user.preferences with the implicit preference.

### At QA Gate (Retrieval)
Before content save, re-check:
- cma.team.standards.content_approval_required — enforce if true
- cma.team.standards.coherence_minimum — flag if content likely below bar
```

#### A.3 Deliverables Summary

| Deliverable | Where | Token Budget | V2 Phase |
|-------------|-------|-------------|----------|
| Memory conventions reference | `framework/memory-conventions.md` or section in schema doc | ~200 tokens | Phase 1 |
| Strategic Memory Protocol section | `storycycle-base-optimized.md` | ~350 tokens | Phase 2 |
| Base framework retrieval instructions | Within Strategic Memory Protocol | Included above | Phase 2 |
| Base framework creation triggers | Within Strategic Memory Protocol | Included above | Phase 2 |

---

### Phase B: Orchestrator Integration

**When:** Part of V2 Phase 7 (Content Creation Orchestrator)

**Goal:** The Content Creation Orchestrator uses memory to make smarter decisions about specialist delegation, workflow mode, and content creation.

#### B.1 Orchestrator Memory-Aware Workflow

Update the Content Creation Orchestrator's progressive loading protocol to integrate memory:

```markdown
## Progressive Loading Protocol (Memory-Enhanced)

| Step | Action | Memory Integration |
|------|--------|--------------------|
| 1 | Start | Base framework loaded (includes Strategic Memory Protocol) |
| 2 | Retrieve CMA Memories | Retrieve cma.user.context, cma.user.preferences, cma.team.standards, cma.team.patterns |
| 3 | Brand ID | If cma.user.context.primary_brand_id exists, load brand directly. Otherwise, run brand-identification skill. |
| 4 | Assets | Load foundation assets. If cma.agent.effectiveness.foundation_load_priorities exists, use it to determine which assets to load and in what order. Otherwise, load all foundation assets per strategic-asset-discovery skill. |
| 5 | Type Selection | Present content type options. If cma.team.patterns.preferred_content_types exists, highlight those. |
| 6 | Type Selected | Load content-type skill. If cma.user.preferences.workflow_mode exists, use it. Otherwise, follow dual-process-selection. |
| 7 | Specialist Delegation | If cma.agent.effectiveness.specialist_value_by_content_type has data for this content type, prefer the recommended specialist sequence. Otherwise, follow specialist-integration skill defaults. |
| 8 | Creation | Execute. |
| 9 | UIC Capture | Run strategic-intelligence-capture skill. |
| 10 | QA + Save | Check cma.team.standards. Save with lineage. |
| 11 | Approval | Present for approval per cma.team.standards.content_approval_required. |
| 12 | Post-Approval Memory Update | Update cma.user.context if brand/audience changed. Detect preference patterns. |
```

#### B.2 Specialist Delegation Enhancement

When the orchestrator delegates to specialists, memory informs the decision:

```markdown
## Specialist Delegation (Memory-Informed)

Before delegating to a specialist:
1. Check cma.agent.effectiveness.specialist_value_by_content_type for the 
   current content type.
2. If data exists with 5+ observations:
   - Use the recommended specialist sequence
   - Example: for blog posts, if data shows [narrative, search] produces 
     avg score 8.7 vs [narrative, behavioral] at 8.1, prefer 
     [narrative, search]
3. If no data or < 5 observations:
   - Follow the default specialist-integration skill logic
   - This session's outcome will contribute to building the effectiveness data

When delegating via connectedAgentRequest, include in the context:
- cma.user.preferences (so specialist maintains tone/style consistency)
- cma.team.standards.brand_voice_standards (if set)
```

#### B.3 Deliverables Summary

| Deliverable | Where | V2 Phase |
|-------------|-------|----------|
| Memory-enhanced progressive loading | `agents/content-creation-orchestrator.md` | Phase 7 |
| Memory-informed specialist delegation | Within orchestrator or `specialist-integration.md` skill | Phase 7 |
| Post-approval memory update logic | Within orchestrator | Phase 7 |

---

### Phase C: Agent Execution Intelligence

**When:** After V2 core phases. Can begin as soon as the Content Creation Orchestrator and Strategic Asset Score agent are both operational.

**Goal:** Close the feedback loop — agent execution patterns feed back into memory so the system learns from every content creation session.

#### C.1 What Gets Captured

After a piece of content is created AND scored by the Strategic Asset Score agent, the orchestrator (or a lightweight post-scoring process) captures:

| Data Point | Source | Stored In |
|------------|--------|-----------|
| Specialist sequence used | Orchestrator session log or `coordinating_agent_roles` from UIC metadata | `cma.agent.effectiveness` |
| Final Strategic Asset Score | Strategic Asset Score record | Correlation with sequence |
| Foundation assets referenced | `intelligence_assets_referenced` from UIC metadata | `cma.agent.effectiveness.foundation_load_priorities` |
| User revision count | Session interaction count | `cma.agent.learning.avg_revisions_by_type` |
| User correction types | Session interaction analysis | `cma.agent.learning.common_user_corrections` |

#### C.2 How It Updates Memory

**After content scoring (automatic):**

```
1. Read the Strategic Asset Score for the content just created
2. Read the UIC metadata (coordinating_agent_roles, intelligence_assets_referenced)
3. Update cma.agent.effectiveness:
   - specialist_sequence_outcomes: add this session's sequence → score data point
     {
       "content_type": "blog_post",
       "specialist_sequence": ["narrative", "search"],
       "score": 8.7,
       "observation_count": 15  // increment
     }
   - specialist_value_by_content_type: recalculate if enough observations
   - foundation_load_priorities: track which foundation combinations correlate
     with highest scores
4. Update cma.agent.learning:
   - avg_revisions_by_type: rolling average
   - common_user_corrections: if correction type matches existing pattern,
     increment count; if new, add with count 1
```

**After user approval (automatic):**

```
1. If user made corrections before approval:
   - Categorize correction types (tone, length, structure, messaging, etc.)
   - Update cma.agent.learning.common_user_corrections
2. If user chose a workflow mode:
   - Check if this matches cma.user.preferences.workflow_mode
   - If different for 3+ sessions, update the preference
```

#### C.3 How Memory Informs Future Sessions

With agent execution intelligence accumulated, future sessions benefit:

| Memory | How It's Used | Benefit |
|--------|---------------|---------|
| `specialist_sequence_outcomes` | Orchestrator prefers sequences with highest average scores | Better content quality |
| `specialist_value_by_content_type` | Orchestrator invokes the right specialists for each content type | Fewer unnecessary specialist calls, lower token usage |
| `foundation_load_priorities` | Orchestrator loads the most relevant foundation assets first | Faster context loading, more relevant context |
| `common_user_corrections` | Orchestrator pre-adjusts for known correction patterns | Fewer revision cycles |
| `avg_revisions_by_type` | System can predict revision effort, set expectations | Better workflow planning |
| `anti_patterns` | Orchestrator avoids sequences/patterns that produced low scores | Prevents known failure modes |

#### C.4 Minimum Observation Thresholds

Don't act on thin data. Memory-informed decisions require minimum confidence:

| Decision | Minimum Observations | Rationale |
|----------|---------------------|-----------|
| Prefer a specialist sequence | 5 scored content pieces with that sequence | Need enough variation to distinguish signal from noise |
| Flag an anti-pattern | 3 low-scoring instances with the pattern present | Anti-patterns should be caught early but not on single failures |
| Update user preference | 3 consistent choices | People vary; 3 is minimum for a pattern |
| Recommend foundation asset combination | 10 scored content pieces | Foundation impact is subtler; need more data |

#### C.5 Deliverables Summary

| Deliverable | Description | Dependency |
|-------------|-------------|------------|
| Post-scoring memory update logic | Orchestrator or post-scoring process correlates execution → score → memory | Content Creation Orchestrator + Strategic Asset Score agent |
| Post-approval preference detection | Detect correction and preference patterns after user approval | Content Creation Orchestrator |
| Memory-informed specialist delegation | Orchestrator uses `cma.agent.effectiveness` data | Phase B + accumulated data |
| Observation threshold constants | Minimum counts before acting on memory data | Design decision (values above) |

---

### Phase D: Evolution (Manual First)

**When:** After Phase C has been running long enough to accumulate meaningful data (estimate: 50+ content creation sessions across the team).

**Goal:** Establish the promotion path from individual patterns to organizational standards.

#### D.1 Manual Promotion Process

For v1, promotion from user → team memory is a manual, admin-driven process:

**Quarterly memory review:**
1. Admin reviews `cma.agent.effectiveness` data across all users
2. Identifies patterns that are consistent across 3+ users:
   - "All users' specialist sequences converge on Narrative → Search for blog posts"
   - "All users prefer step-by-step for longform content"
3. Admin promotes validated patterns to `cma.team.patterns` or `cma.team.standards`
4. Admin can also update `cma.team.standards` (quality thresholds, approval policies) based on team outcomes

**Promotion criteria (manual checklist):**
- Pattern observed across 3+ users
- Pattern correlates with above-average Strategic Asset Scores
- Pattern has been stable for 30+ days (not a temporary trend)
- Pattern doesn't conflict with existing team standards

#### D.2 Demotion Process

If a promoted pattern stops performing:
1. Agent effectiveness data shows declining scores for the promoted pattern
2. Admin reviews and can demote (remove from team memory or update)
3. Demotion triggers agents to fall back to default behavior for that decision

#### D.3 Framework Protocol Updates

When a pattern is so well-validated and universal that it should become a default:
1. Admin and team review the pattern
2. Human decision to update base framework or skill
3. Update `storycycle-base-optimized.md` or the relevant skill
4. The pattern is no longer in memory — it's in the code

**Example:** If data consistently shows that Narrative → Search is the best specialist sequence for blog posts across all clients, update the `specialist-integration.md` skill to make this the default for blog posts. Remove the pattern from agent memory — it's now governance.

#### D.4 Deliverables Summary

| Deliverable | Description | When |
|-------------|-------------|------|
| Quarterly review checklist | Document for admin to follow when reviewing memory patterns | After 50+ sessions |
| Promotion criteria doc | Thresholds for promoting user patterns to team standards | With Phase D |
| Framework update process | How to move validated patterns from memory into base/skills | With Phase D |

---

## Integration with V2 Phases

| V2 Phase | Memory Work | Deliverable |
|----------|------------|-------------|
| **Phase 1: Schema Definition** | Memory conventions document | `framework/memory-conventions.md` (or section in schema doc) |
| **Phase 2: Base Framework** | Strategic Memory Protocol section in base | ~350 tokens in `storycycle-base-optimized.md` |
| **Phase 4: Skills Extraction** | Ensure `strategic-asset-discovery` skill references memory shortcuts (cma.user.context.primary_brand_id) | Updated skill |
| **Phase 5: Content-Type Skills** | No memory-specific changes needed — skills conform to base contract | — |
| **Phase 6: Recomposition + MCP** | Recomposition skills retrieve `cma.agent.effectiveness.recomposition_success_rates` if available | Updated recomposition skills |
| **Phase 7: Orchestrator** | Memory-enhanced progressive loading; memory-informed specialist delegation; post-approval memory update | Updated orchestrator |
| **Phase 9: Testing** | Test memory retrieval at session start; test creation triggers on brand confirmation and content approval; test that memory-informed specialist delegation works when data exists | Test scenarios |
| **Phase 10: Deploy** | Deploy base with memory protocol; no separate memory deployment needed | — |
| **Post-V2: Phase C** | Agent execution intelligence (post-scoring feedback loop) | Post-scoring memory update logic |
| **Post-V2: Phase D** | Manual evolution process (quarterly review, promotion, demotion) | Review checklist, promotion criteria |

---

## What Goes Into the Base Framework (Complete Section)

This is the exact section to include in `storycycle-base-optimized.md`. It replaces the "Memory conventions" bullet point in V2 Phase 2 with concrete, buildable instructions.

```markdown
## Strategic Memory Protocol

### Memory Conventions
CMA memories use the `cma.` prefix with structured JSON objects:
- `cma.user.context` (User) — primary_brand_id, primary_audience_ids, active_campaign_id
- `cma.user.preferences` (User) — workflow_mode, detail_level, tone, content_length
- `cma.team.standards` (Team) — coherence_minimum, confidence_auto_approve, content_approval_required, required_specialists
- `cma.team.patterns` (Team) — preferred_content_types, brand_voice_standards
- `cma.agent.effectiveness` (Agent) — specialist_sequence_outcomes, specialist_value_by_content_type, foundation_load_priorities
- `cma.agent.learning` (Agent) — common_user_corrections, avg_revisions_by_type, anti_patterns

All memory values include `last_updated` timestamp.

### Retrieval: At Session Start
Before any content work:
1. Retrieve `cma.user.context` — If primary_brand_id exists, load brand story 
   directly (skip brand search). Otherwise, proceed to brand identification.
2. Retrieve `cma.user.preferences` — Apply workflow_mode if set (overrides 
   dual-process-selection default). Apply tone/detail preferences to generation.
3. Retrieve `cma.team.standards` — Use coherence_minimum as quality bar 
   (default 8.0 if not set). Enforce content_approval_required if true. 
   Note required_specialists for delegation.
4. Retrieve `cma.team.patterns` — Highlight preferred_content_types in type 
   selection. Apply brand_voice_standards to generation.

If any retrieval returns empty, use defaults and proceed.

### Retrieval: At Specialist Delegation
Before connectedAgentRequest to specialists:
- Retrieve `cma.agent.effectiveness` — If specialist_value_by_content_type has 
  5+ observations for this content type, prefer the recommended sequence.
- Pass cma.user.preferences (tone, detail_level) in specialist context.

### Retrieval: At QA Gate
Before content save:
- Re-check cma.team.standards.content_approval_required
- Re-check cma.team.standards.coherence_minimum

### Creation: At Brand Confirmation
When user confirms or selects a brand:
- Remember `cma.user.context` with updated primary_brand_id, 
  primary_audience_ids (from brand's associated audiences), last_updated.

### Creation: At Content Approval
After user approves content:
- If user made 3+ same-type corrections across sessions, remember 
  `cma.user.preferences` with the implicit preference.
- If user chose workflow_mode 3+ times consistently, update 
  cma.user.preferences.workflow_mode.

### Creation: After Content Scoring (Phase C)
After Strategic Asset Score is computed for new content:
- Correlate specialist sequence + foundation assets → score
- Update `cma.agent.effectiveness` with new data point
- Update `cma.agent.learning` with revision count and correction types
- Only act on accumulated data when observation count ≥ 5
```

---

## Success Criteria

### Phase A (Conventions + Base Framework)

| Criterion | How to Verify |
|-----------|--------------|
| Memory conventions documented | Document exists and is referenced from base framework |
| Base framework includes Strategic Memory Protocol | Section present in `storycycle-base-optimized.md` |
| Retrieval triggers defined | At least 4 retrieval points specified (session start, specialist delegation, QA gate, brand confirmation) |
| Creation triggers defined | At least 2 creation points specified (brand confirmation, content approval) |

### Phase B (Orchestrator Integration)

| Criterion | How to Verify |
|-----------|--------------|
| Orchestrator retrieves memories at session start | Test: agent calls retrieve for cma.user.* and cma.team.* before content work |
| Brand shortcut works | Test: with cma.user.context.primary_brand_id set, agent skips brand search |
| Preferences applied | Test: with cma.user.preferences.workflow_mode set, agent uses that mode |
| Memory creation on brand confirmation | Test: after brand confirmation, cma.user.context is updated |

### Phase C (Agent Execution Intelligence)

| Criterion | How to Verify |
|-----------|--------------|
| Post-scoring memory update fires | After content scoring, cma.agent.effectiveness is updated |
| Specialist sequence recorded | Sequence + score pair stored in agent memory |
| Memory-informed delegation works | With 5+ observations, orchestrator selects the recommended sequence |
| Minimum observation thresholds enforced | With < 5 observations, orchestrator falls back to default behavior |

### Phase D (Evolution)

| Criterion | How to Verify |
|-----------|--------------|
| Review process executed | Admin completes quarterly review checklist |
| Pattern promoted successfully | At least one user pattern promoted to team standard |
| Promoted pattern affects behavior | Team members see promoted pattern reflected in their sessions |

---

## What This Enables

When all phases are implemented, strategic memory utilization delivers the concrete mechanisms behind the Cognitive Amplification Flywheel:

```
Session 1: Agent uses defaults. User corrects tone. Content scores 7.8.
           → Stores: user worked with Brand X, correction type = tone

Session 5: Agent retrieves Brand X directly (no search). Applies tone 
           preference. User makes fewer corrections. Content scores 8.2.
           → Stores: Narrative specialist improved blog score

Session 15: Agent uses preferred specialist sequence for blogs. 
            Loads optimal foundation assets. Pre-adjusts for tone.
            User approves with 0 corrections. Content scores 8.9.
            → Stores: high-value creation sequence identified

Session 50: Admin reviews patterns. Promotes specialist sequence to team 
            standard. All team members benefit from accumulated intelligence.
            Average blog score across team: 8.6 (up from 7.9 baseline).
```

**The system doesn't just get faster — it gets smarter.** Every session contributes to the intelligence that makes the next session better. That's Return on Intelligence.

---

## References

- [strategic-memory-utilization-exploration.md](../research/strategic-memory-utilization-exploration.md) — Exploration, taxonomy, open questions
- [storycycle-genie-v2-executable-vision.md](storycycle-genie/storycycle-genie-v2-executable-vision.md) — V2 phases and integration points
- [cma-technical-spec.md](../spec/cma-technical-spec.md) — Strategic Memory Utilization Framework definition
- [cma-authoritative-reference.md](../spec/cma-authoritative-reference.md) — Tier 1 Memory Layer, current gaps
- [glean-context-graph-architecture-considerations.md](../research/glean-context-graph-architecture-considerations.md) — Agent trace feedback loop concept (Phase C source)
