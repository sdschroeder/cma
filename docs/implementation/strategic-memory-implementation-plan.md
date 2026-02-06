# Strategic Memory Utilization: Implementation Plan

**Purpose:** Practical, buildable plan for implementing strategic memory utilization in StoryCycle Genie V2. This document defines what to build, in what order, and exactly what goes into the base framework, orchestrators, and skills.

**Relationship to V2:** This plan provides the specific content for the "Memory conventions" deliverable in V2 Phase 1, the "Add memory conventions" step in V2 Phase 2 (Base Framework), and the "Memory Triggers" test criteria in V2 Phase 9. It also extends memory utilization into V2 Phases 5-7 (orchestrator, skills) and defines post-V2 enhancements.

**Background:** [strategic-memory-utilization-exploration.md](../research/strategic-memory-utilization-exploration.md)

---

## Design Decisions

These decisions resolve the open architecture questions from the exploration and are required before implementation.

### Decision 1: Memory Granularity — Structured Objects per Category

**Decision:** Store CMA memories as **one structured object per category**, not one memory item per individual key.

**Rationale:** Each memory retrieval adds latency and token overhead. Grouping related values into a single object reduces retrieval calls while keeping categories cleanly separated.

**Structure:**

| Memory Name | Level | Scope | Contains |
|-------------|-------|-------|----------|
| `cma.user.context` | User | Global | `primary_brand_id`, `primary_audience_ids`, `active_campaign_id` |
| `cma.user.preferences` | User | Global | `workflow_mode`, `detail_level`, `option_presentation` |
| `cma.team.defaults` | Team | Global | `coherence_minimum`, `confidence_auto_approve`, `content_approval_required`, `required_specialists` — fallback when no brand-specific standards exist |
| `cma.team.brand.{brand_id}` | Team | Per-brand | `coherence_minimum`, `required_specialists`, `voice_standards`, `preferred_content_types`, `content_approval_required` — standards for this specific brand |
| `cma.agent.effectiveness` | Agent | Global | `specialist_sequence_outcomes`, `specialist_value_by_content_type`, `foundation_load_priorities` |
| `cma.agent.learning` | Agent | Global | `common_user_corrections`, `avg_revisions_by_type`, `high_value_creation_sequences`, `anti_patterns` |

**Example — user context:**
```json
{
  "name": "cma.user.context",
  "level": "user",
  "value": {
    "primary_brand_id": "abc-123",
    "primary_audience_ids": ["def-456", "ghi-789"],
    "active_campaign_id": null,
    "last_updated": "2026-02-06T12:00:00Z"
  }
}
```

**Example — brand-specific team standards (agency with multiple clients):**
```json
{
  "name": "cma.team.brand.abc-123",
  "level": "team",
  "value": {
    "coherence_minimum": 8.5,
    "required_specialists": ["narrative", "search"],
    "voice_standards": { "tone": "conversational_authority", "complexity": "technical" },
    "preferred_content_types": ["blog_post", "case_study", "whitepaper"],
    "content_approval_required": true,
    "last_updated": "2026-02-06T12:00:00Z"
  }
}
```

**Impact on retrieval:** At session start, the agent retrieves `cma.user.context`, `cma.user.preferences`, and `cma.team.defaults` (3 calls). After brand is confirmed, one additional retrieval for `cma.team.brand.{brand_id}`. Still 4 retrievals total, but the last one is brand-aware.

### Decision 2: Multi-Brand Scoping — Brand-Specific Team Standards

**Decision:** Team standards are **per-brand** via `cma.team.brand.{brand_id}`, with `cma.team.defaults` as the fallback. User preferences are **global** (they describe how the user works, not how the brand works).

**Rationale:** StoryCycle Genie is used by agencies that manage multiple brands. An agency's quality bar, voice standards, required specialists, and preferred content types differ by client. A flat `cma.team.standards` would force one set of standards across all brands, which doesn't match how agencies work.

**How it works for different account types:**

| Account Type | How Brand-Scoping Works |
|-------------|-------------------------|
| **Single-brand team** (e.g., in-house marketing) | One `cma.team.brand.{brand_id}` memory for their brand. `cma.team.defaults` may not even be set — the brand-specific memory is the standard. Functionally identical to the old flat model. |
| **Agency with multiple brands** | One `cma.team.brand.{brand_id}` per client brand. `cma.team.defaults` holds the agency's baseline standards. When a user switches brands, the agent loads that brand's standards. Each client gets its own quality bar, voice, and specialist requirements. |
| **New brand (no standards yet)** | No `cma.team.brand.{brand_id}` exists. Agent falls through to `cma.team.defaults`. If that's also empty, falls through to skill defaults. Standards accumulate as the team works with this brand. |

**What's global vs. brand-specific:**

| Memory | Scope | Why |
|--------|-------|-----|
| `cma.user.preferences` (workflow_mode, detail_level, option_presentation) | **Global per user** | These describe how the person works. An agency creative's preference for step-by-step vs. automated doesn't change by client. |
| `cma.user.context` (primary_brand_id, audiences, campaign) | **Session context** | Updates when user switches brands. Points to the active brand. |
| `cma.team.brand.{brand_id}` (quality bar, voice, specialists, content types) | **Per brand** | These describe how work for this brand should be done. An agency has different standards per client. |
| `cma.team.defaults` (baseline quality, approval policy) | **Team-wide fallback** | The agency's baseline that applies when no brand-specific standard is set. |
| `cma.agent.effectiveness`, `cma.agent.learning` | **Global (process intelligence)** | "Narrative → Search is best for blogs" is true regardless of brand. Process intelligence is brand-agnostic. |

**Brand switching flow for an agency user:**
1. User is working on Brand X. `cma.user.context.primary_brand_id` = Brand X.
2. Agent loaded `cma.team.brand.{brand_x_id}` — quality bar 8.5, conversational authority voice.
3. User says "Let's switch to Brand Y."
4. Agent updates `cma.user.context.primary_brand_id` to Brand Y.
5. Agent retrieves `cma.team.brand.{brand_y_id}` — quality bar 7.5, friendly approachable voice.
6. Session continues with Brand Y's standards. No confusion, no bleed between brands.

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

### Decision 5: Promotion Authority — Two Loops, Two Owners

**Decision:** Memory evolution operates in two distinct loops with different authorities:

| Promotion | Authority | Mechanism |
|-----------|-----------|-----------|
| Session → User Memory | Automatic (agent) | Agent detects pattern (3+ occurrences) and stores |
| User → Team Memory | Team admin (customer) | Admin reviews user patterns and promotes to team standard |
| Pattern → Agent Memory | Automatic (post-scoring) | After content scoring, agent correlates process → outcome |
| Agent Memory → Product Skill | CPO (product team) | CPO reviews cross-team aggregated patterns, decides skill updates |

**Within-team loop:** Team admin manages their brand-specific standards. Automated for user patterns, manual for team standards.

**Cross-team loop:** CPO harvests process intelligence from aggregated, anonymized agent effectiveness data across all teams. Validated patterns become skill updates that ship to all teams via MCP. This is a product development decision, not a per-customer action.

**For v1:** Session → User and Pattern → Agent are automated. Team promotion is manual (team admin). Cross-team product updates are manual (CPO-driven, quarterly review cycle).

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
- `cma.user.context` — Working context (active brand, audiences, campaign)
- `cma.user.preferences` — Global workflow and content preferences
- `cma.team.defaults` — Team-wide default standards (fallback)
- `cma.team.brand.{brand_id}` — Brand-specific standards (quality, voice, specialists)
- `cma.agent.effectiveness` — Specialist and process effectiveness data
- `cma.agent.learning` — Corrections, revision patterns, anti-patterns

### Levels
- **User:** Individual context and global preferences. Created/updated by agent.
- **Team:** Brand-specific standards and team-wide defaults. Set by team admin.
  Brand-scoped via `cma.team.brand.{brand_id}` for agencies with multiple clients.
- **Agent:** Process intelligence (brand-agnostic). Updated automatically after 
  content scoring.

### Brand Scoping
Team standards are per-brand. When a user works on Brand X, the agent loads
`cma.team.brand.{brand_x_id}`. If no brand-specific standards exist, falls
through to `cma.team.defaults`. This supports agencies managing multiple
client brands with different quality bars, voice standards, and specialist
requirements.

### Values
All CMA memories store structured JSON objects, not single values.
Always include `last_updated` timestamp in the value.

Example (user context):
  name: "cma.user.context"
  value: {
    "primary_brand_id": "abc-123",
    "primary_audience_ids": ["def-456"],
    "active_campaign_id": null,
    "last_updated": "2026-02-06T12:00:00Z"
  }

Example (brand-specific team standards):
  name: "cma.team.brand.abc-123"
  value: {
    "coherence_minimum": 8.5,
    "required_specialists": ["narrative", "search"],
    "voice_standards": {"tone": "conversational_authority"},
    "preferred_content_types": ["blog_post", "case_study"],
    "content_approval_required": true,
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
2. Call retrieve for "cma.user.preferences" — get workflow mode, detail level

If cma.user.context has a primary_brand_id, use it to load the brand story 
directly (skip brand search). If no user context exists, fall through to 
brand identification protocol.

If cma.user.preferences has a workflow_mode, use it to set the initial 
interaction mode (step-by-step vs automated). Default: follow 
dual-process-selection confidence logic.

### After Brand Confirmed (Retrieval)
Once primary_brand_id is known (from context or brand identification):

3. Call retrieve for "cma.team.brand.{primary_brand_id}" — brand-specific 
   standards (quality bar, voice, specialists, content types)
4. Call retrieve for "cma.team.defaults" — team-wide fallback standards

For each setting, apply brand-specific value if set, otherwise team default,
otherwise skill default. This supports agencies with per-client standards
and single-brand teams equally.

### At Brand Switch (Retrieval + Creation)
When user switches to a different brand mid-session:
- Update cma.user.context with new primary_brand_id and audiences
- Retrieve cma.team.brand.{new_brand_id} for the new brand's standards
- Apply new standards for the remainder of the session
- No bleed between brands

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
Before content save, re-check active brand standards (already in context):
- content_approval_required — enforce if true
- coherence_minimum — flag if content likely below bar
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

### Phase D: Evolution — Two Loops

**When:** After Phase C has been running long enough to accumulate meaningful data across multiple teams.

**Goal:** Establish two distinct evolution loops that match the SaaS reality: teams manage their own standards, and the CPO harvests cross-team intelligence into product-level improvements.

#### The Two Loops

StoryCycle Genie is a multi-tenant SaaS product. Memory evolution operates at two levels that serve different purposes and are governed by different people:

```
LOOP 1: WITHIN-TEAM (each customer's team manages their own standards)
┌────────────────────────────────────────────────────────┐
│  Session → User Memory → Team Memory                    │
│                                                          │
│  Governed by: Team admin (customer)                      │
│  Scope: Their brand, their voice, their quality bar      │
│  Examples:                                               │
│    - "Our quality threshold is 8.5"                      │
│    - "Our brand voice is conversational authority"        │
│    - "We always need Narrative specialist for our blogs"  │
│    - "Our primary audiences are X and Y"                 │
└────────────────────────────────────────────────────────┘

LOOP 2: CROSS-TEAM → PRODUCT (CPO harvests patterns into skills)
┌────────────────────────────────────────────────────────┐
│  Agent effectiveness data (all teams, aggregated)        │
│       ↓                                                  │
│  CPO reviews cross-team patterns                         │
│       ↓                                                  │
│  Engineering updates skill or base framework             │
│       ↓                                                  │
│  MCP serves updated skill to all teams                   │
│                                                          │
│  Governed by: CPO (product team)                         │
│  Scope: How the creation process works best              │
│  Examples:                                               │
│    - "Narrative → Search produces 12% higher blog        │
│       scores than Narrative → Behavioral across teams"   │
│    - "Skipping Audience Story correlates with -15%       │
│       coherence across all teams"                        │
│    - "Users who see comprehensive options before         │
│       generation request 40% fewer revisions"            │
└────────────────────────────────────────────────────────┘
```

**The key distinction:** Team intelligence is about *their brand and standards*. Product intelligence is about *how the creation process works best*. These are different categories of knowledge with different owners.

| Intelligence Type | Examples | Owner | Stays In |
|-------------------|----------|-------|----------|
| **Brand-specific** | Voice standards, quality bar, primary audiences, preferred content types | Team admin | Team memory (`cma.team.*`) |
| **Process-universal** | Best specialist sequences, foundation loading priorities, anti-patterns, revision predictors | CPO | Skills and base framework (after validation) |

#### D.1 Within-Team Evolution (Team Admin)

Each team's admin manages standards through `cma.team.brand.{brand_id}` (per-brand) and `cma.team.defaults` (team-wide fallback). This supports both single-brand in-house teams and agencies managing multiple client brands.

**What team admins control per brand (`cma.team.brand.{brand_id}`):**
- `coherence_minimum` — quality bar for this brand
- `content_approval_required` — approval policy for this brand
- `required_specialists` — which specialists this brand's content needs
- `voice_standards` — this brand's voice (tone, complexity, style)
- `preferred_content_types` — what they create most for this brand

**What team admins control as defaults (`cma.team.defaults`):**
- Baseline `coherence_minimum`, `content_approval_required`, `required_specialists`
- These apply when a brand has no specific standards yet

**How it works for a single-brand team:**
- One `cma.team.brand.{brand_id}` for their brand. Functionally identical to the previous flat model.

**How it works for an agency with multiple brands:**
1. Admin sets brand-specific standards for each client: "Client A: quality 8.5, requires Narrative + Search, conversational authority voice. Client B: quality 7.5, requires Narrative only, friendly approachable voice."
2. When a user works on Client A, the agent loads Client A's standards. When they switch to Client B, Client B's standards load. No bleed between brands.
3. Agency sets `cma.team.defaults` as the baseline for new clients that don't have specific standards yet.
4. As the team accumulates Phase C data for a new client, the admin can review patterns and set brand-specific standards for that client.

**This loop is self-contained within each account.** No cross-team visibility needed. No product team involvement. The team admin is the authority on their brands' standards.

#### D.2 Cross-Team Product Intelligence (CPO)

This is the loop that turns StoryCycle Genie into a product that gets smarter with every customer. The CPO harvests process intelligence from aggregated, anonymized agent effectiveness data across all teams and converts validated patterns into product-level improvements.

**What kinds of patterns become product intelligence:**

| Pattern | How It's Observed | What It Becomes |
|---------|-------------------|-----------------|
| Specialist sequences that produce higher scores | `cma.agent.effectiveness.specialist_sequence_outcomes` aggregated across teams | Updated `specialist-integration.md` skill with optimized defaults |
| Foundation loading priorities | `cma.agent.effectiveness.foundation_load_priorities` aggregated | Updated `strategic-asset-discovery.md` skill with better load order |
| Anti-patterns | `cma.agent.learning.anti_patterns` consistent across teams | Warnings or guardrails in base framework or content-type skills |
| Revision predictors | `cma.agent.learning.common_user_corrections` aggregated | Pre-adjustments in content-type skills (e.g., "users consistently want more professional tone — default higher") |
| Workflow mode effectiveness | `cma.user.preferences.workflow_mode` correlated with scores across teams | Updated `dual-process-selection.md` skill confidence thresholds |
| Content type best practices | High-scoring content patterns across teams | Enriched content-type skills with validated structural patterns |

**What does NOT become product intelligence:**
- Brand-specific voice standards (unique per team)
- Team quality thresholds (business decision per customer)
- Individual user preferences (privacy, personalization)
- Specific brand/audience asset IDs (team data)

#### D.3 The CPO's Intelligence Pipeline

**Step 1: Data Aggregation**

Agent effectiveness and learning data accumulates per-team in account-scoped memories. To give the CPO cross-team visibility, build a **product analytics view** that aggregates this data across accounts.

For v1 this can be simple — a periodic query or report that:
- Pulls `cma.agent.effectiveness` and `cma.agent.learning` data across all accounts
- Strips team-identifying information (no brand names, no user IDs, no asset content)
- Aggregates into pattern summaries

What the CPO sees (example):

```
SPECIALIST SEQUENCE EFFECTIVENESS (across 45 teams, 1,200 blog posts)
─────────────────────────────────────────────────────────────────────
Narrative → Search:        avg score 8.7  (used by 38 teams, 890 posts)
Narrative → Behavioral:    avg score 8.1  (used by 29 teams, 540 posts)
Search only:               avg score 7.6  (used by 12 teams, 180 posts)
No specialist:             avg score 7.2  (used by 8 teams, 95 posts)

ANTI-PATTERNS (consistent across 10+ teams)
─────────────────────────────────────────────────────────────────────
Skip Audience Story:       -15% coherence  (observed in 22 teams)
Skip Content Playbook:     -8% coherence   (observed in 18 teams)
Automated mode for longform: +40% revisions (observed in 15 teams)

REVISION PREDICTORS (top correction types across all teams)
─────────────────────────────────────────────────────────────────────
Tone too casual:           34% of all corrections
Missing proof elements:    22% of all corrections
Too long:                  18% of all corrections
```

**Step 2: CPO Validation**

The CPO reviews aggregated patterns and applies product judgment:

**Validation criteria:**
- Pattern observed across **10+ teams** (not just a few power users)
- Pattern has **statistical significance** (enough observations to be reliable)
- Pattern has been **stable for 60+ days** (not a temporary trend)
- Pattern represents **process intelligence** (how creation works best), not brand intelligence (team-specific)
- Improvement is **backward compatible** (won't break existing team workflows)

**Not every pattern becomes a product change.** Some patterns are informative but don't warrant a skill update. The CPO decides based on:
- Magnitude of impact (12% score improvement is worth a skill change; 2% may not be)
- Confidence (890 observations vs. 15 observations)
- Risk (changing specialist defaults has lower risk than changing content structure)

**Step 3: Skill Update**

When the CPO validates a pattern, the update path is:

1. **CPO defines the change** — What behavior should change, and what the new default should be
2. **Engineering updates the skill** — e.g., `specialist-integration.md` gets a new default sequence for blog posts
3. **Testing via StoryCycle MCP** — Validate the updated skill doesn't regress quality
4. **Deployment via MCP** — The updated skill is served to all teams on next load. No per-team deployment needed.

**The MCP architecture makes this clean.** Skills are loaded on-demand via `mcp_storycycle_get_skill`. When a skill file is updated, every team gets the new version the next time their orchestrator loads that skill. There's no migration, no per-account update, no version management headache.

**Step 4: Post-Deployment Monitoring**

After a skill update ships:
- Monitor agent effectiveness data across teams for the affected pattern
- Confirm the expected improvement materializes (scores go up, revisions go down)
- If scores degrade, revert the skill change
- The data pipeline from Step 1 provides the monitoring for free — same aggregation, just looking at the trend after the change

#### D.4 Example: Specialist Sequence → Skill Update

End-to-end example of how a cross-team pattern becomes a product improvement:

**Month 1-3:** Phase C accumulates data. Across 30+ teams, `cma.agent.effectiveness.specialist_sequence_outcomes` shows:
- Blog posts with Narrative → Search: avg score 8.7 (600+ observations)
- Blog posts with Narrative → Behavioral: avg score 8.1 (400+ observations)
- Difference is consistent and statistically significant

**Month 4:** CPO reviews quarterly product intelligence report. Sees the specialist sequence data. Validates:
- 30+ teams (not a single-team anomaly)
- 1,000+ observations (high confidence)
- Stable for 90+ days
- This is process intelligence (how blog creation works best), not brand intelligence

**CPO decision:** Update `specialist-integration.md` to default to Narrative → Search for blog posts, while keeping the override path available (teams can still use other sequences via team memory).

**Engineering:** Updates the skill:

```markdown
## Specialist Delegation Defaults

### Blog Posts
Default sequence: Narrative → Search
Rationale: Cross-team data shows 12% higher Strategic Asset Scores 
with this sequence (1,000+ observations across 30+ teams).

Override: If cma.team.standards.required_specialists or 
cma.agent.effectiveness specifies a different sequence with 5+ 
team-specific observations, use the team/agent override.
```

**Deployment:** Skill file updated in StoryCycle MCP. All teams get the new default on their next blog post creation session.

**Monitoring:** Over the next 60 days, blog post scores across all teams trend up ~8%. CPO confirms the change was beneficial. The pattern is now governance — it lives in the skill, not in memory.

**Key detail:** Teams that have their own `cma.team.standards.required_specialists` or enough team-specific agent effectiveness data can override the product default. The product default is the best starting point; team-specific intelligence takes precedence when it exists.

#### D.5 Relationship Between the Two Loops

The loops aren't independent — they interact:

```
TEAM MEMORY (per-team)               PRODUCT SKILLS (all teams)
cma.team.brand.{id}   ←──────────── Default values come from skills
cma.team.defaults      ←──────────── (when no brand-specific standard)
cma.agent.effectiveness ────────────→ Aggregated data informs CPO
                                       ↓
                                  Skill updates ship via MCP
                                       ↓
                                  New defaults ←── Override by brand memory
```

**Precedence order when the agent makes a decision:**
1. **Brand-specific team memory** (`cma.team.brand.{brand_id}`) — if the team has set standards for this brand, use them
2. **Team defaults** (`cma.team.defaults`) — if no brand-specific standard, use team defaults
3. **Agent effectiveness** (`cma.agent.effectiveness`) — if the team has enough local data (5+ observations), use it
4. **Skill defaults** — fall back to product-level defaults (which are informed by cross-team intelligence)

This means:
- A new team gets the best defaults from day one (product intelligence)
- As the team accumulates its own data, the system adapts to their specific patterns
- The team admin can set standards per brand (agencies) or one set of standards (single-brand teams)
- The product keeps getting smarter as more teams contribute to the aggregate

#### D.6 Deliverables Summary

| Deliverable | Description | Owner | When |
|-------------|-------------|-------|------|
| **Within-team admin controls** | UI or agent workflow for team admin to set `cma.team.standards` | Engineering | With Phase A/B |
| **Product analytics aggregation** | Query/report that aggregates agent effectiveness data across accounts (anonymized) | Engineering + CPO | After Phase C is running across multiple teams |
| **CPO review dashboard or report** | Cross-team pattern summaries (specialist effectiveness, anti-patterns, revision predictors) | Engineering + CPO | With product analytics |
| **Validation criteria** | Documented thresholds for when a cross-team pattern warrants a skill update | CPO | With Phase D |
| **Skill update workflow** | Process: CPO decision → engineering update → MCP test → deploy → monitor | CPO + Engineering | With Phase D |
| **Post-deployment monitoring** | Confirm skill changes improve (or don't degrade) agent effectiveness data | CPO + Engineering | Ongoing |
| **Override precedence logic** | Team memory > agent effectiveness > skill defaults, documented in base framework | Engineering | With Phase B |

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
| **Post-V2: Phase D** | Two evolution loops: within-team (admin sets standards) and cross-team (CPO harvests patterns into skills via MCP) | Product analytics aggregation, CPO review process, skill update workflow, override precedence logic |

---

## What Goes Into the Base Framework (Complete Section)

This is the exact section to include in `storycycle-base-optimized.md`. It replaces the "Memory conventions" bullet point in V2 Phase 2 with concrete, buildable instructions.

```markdown
## Strategic Memory Protocol

### Memory Conventions
CMA memories use the `cma.` prefix with structured JSON objects:
- `cma.user.context` (User) — primary_brand_id, primary_audience_ids, active_campaign_id
- `cma.user.preferences` (User) — workflow_mode, detail_level, option_presentation
- `cma.team.defaults` (Team) — coherence_minimum, confidence_auto_approve, content_approval_required, required_specialists
- `cma.team.brand.{brand_id}` (Team) — brand-specific: coherence_minimum, required_specialists, voice_standards, preferred_content_types, content_approval_required
- `cma.agent.effectiveness` (Agent) — specialist_sequence_outcomes, specialist_value_by_content_type, foundation_load_priorities
- `cma.agent.learning` (Agent) — common_user_corrections, avg_revisions_by_type, anti_patterns

All memory values include `last_updated` timestamp.
Team standards are brand-scoped to support agencies with multiple client brands.

### Retrieval: At Session Start
Before any content work:
1. Retrieve `cma.user.context` — If primary_brand_id exists, load brand story 
   directly (skip brand search). Otherwise, proceed to brand identification.
2. Retrieve `cma.user.preferences` — Apply workflow_mode if set (overrides 
   dual-process-selection default). Apply detail/option preferences.

### Retrieval: After Brand Confirmed
Once primary_brand_id is known (from cma.user.context or brand identification):
3. Retrieve `cma.team.brand.{primary_brand_id}` — brand-specific standards.
4. Retrieve `cma.team.defaults` — team-wide fallback standards.
5. Apply brand standards with fallback: for each setting (coherence_minimum, 
   required_specialists, voice_standards, content_approval_required), use 
   brand-specific value if set, otherwise team default, otherwise skill default.

This brand-scoped retrieval supports agencies managing multiple client brands.
Each brand has its own quality bar, voice, and specialist requirements.

### Retrieval: At Specialist Delegation
Before connectedAgentRequest to specialists:
- Retrieve `cma.agent.effectiveness` — If specialist_value_by_content_type has 
  5+ observations for this content type, prefer the recommended sequence.
- Pass cma.user.preferences and active brand voice_standards in specialist context.

### Retrieval: At QA Gate
Before content save, re-check the active brand standards (already in context 
from post-brand retrieval):
- content_approval_required
- coherence_minimum

### Creation: At Brand Confirmation
When user confirms or selects a brand:
- Remember `cma.user.context` with updated primary_brand_id, 
  primary_audience_ids (from brand's associated audiences), last_updated.
- This is the trigger to retrieve brand-specific team standards.

### Creation: At Brand Switch
When user switches to a different brand mid-session:
- Update `cma.user.context` with new primary_brand_id and audiences.
- Retrieve `cma.team.brand.{new_brand_id}` to load the new brand's standards.
- Apply new brand standards for the remainder of the session.
- No bleed between brands — each brand's context is cleanly loaded.

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

### Phase D (Evolution — Within-Team)

| Criterion | How to Verify |
|-----------|--------------|
| Team admin can set standards | Admin successfully updates `cma.team.standards` (quality bar, approval policy) |
| Team standards affect behavior | Team members' sessions reflect the team standards (e.g., coherence minimum enforced) |
| User pattern visible to admin | Admin can see that user patterns exist before deciding to promote |

### Phase D (Evolution — Cross-Team Product)

| Criterion | How to Verify |
|-----------|--------------|
| Product analytics aggregation works | CPO can see anonymized, aggregated agent effectiveness data across teams |
| Cross-team pattern identified | At least one pattern validated across 10+ teams with statistical significance |
| Skill update deployed via MCP | Updated skill serves to all teams on next load |
| Post-deployment monitoring | Agent effectiveness data confirms improvement (or no degradation) after skill update |
| Override precedence works | Team with specific `cma.team.standards` overrides the product default |

---

## What This Enables

When all phases are implemented, strategic memory utilization delivers two compounding intelligence loops:

**Loop 1 — Within a team (agency with multiple brands):**

```
Session 1:  Agency user works on Client A. Agent uses product defaults. 
            User corrects tone. Score: 7.8.
            → Stores: user context = Client A, correction type = tone

Session 5:  User works on Client A again. Agent retrieves Client A 
            directly. Applies tone preference. Fewer corrections. Score: 8.2.
            → Stores: Narrative specialist improved blog score

Session 8:  User switches to Client B. Agent loads Client B's brand story
            and audiences. No brand-specific standards yet for Client B, 
            so team defaults apply. Score: 7.9.
            → Stores: user context = Client B

Session 15: Admin sets Client A standards: quality 8.5, requires Narrative 
            + Search, conversational authority voice. Sets Client B 
            standards: quality 7.5, requires Narrative, friendly voice.
            Each client now has its own quality bar and voice.

Session 25: User works on Client A. Agent loads Client A's standards 
            automatically. Uses preferred specialist sequence. Pre-adjusts
            for conversational authority tone. Zero corrections. Score: 8.9.
            
            Same afternoon, user switches to Client B. Agent loads Client 
            B's standards. Different voice, different quality bar. No bleed
            between brands. Score: 8.1.
```

**Loop 2 — Across all teams into the product:**

```
Month 3:   Phase C is running across 30+ teams. Agent effectiveness data 
           accumulates: Narrative → Search produces 12% higher blog scores 
           than Narrative → Behavioral across 1,000+ observations.

Month 4:   CPO reviews quarterly product intelligence report. Validates 
           the pattern (30+ teams, stable 90+ days, process intelligence).
           Decision: update specialist-integration.md skill.

Month 5:   Updated skill ships via MCP. A brand-new team that signs up 
           today gets the optimized specialist sequence from their first 
           blog post — they benefit from intelligence accumulated across 
           30 other teams without having to learn it themselves.

Month 8:   CPO reviews again. Blog scores across all teams are up 8% 
           since the skill update. Anti-pattern "skip Audience Story" 
           is now flagged in the base framework. Three more skill updates 
           ship based on cross-team patterns.
```

**Three forms of Return on Intelligence:**
- **Per-brand ROI** — Each brand within a team gets smarter with every session. An agency's Client A gets better defaults, better specialist sequences, better quality — independent of Client B.
- **Per-team ROI** — The agency's overall process intelligence accumulates. Their team defaults improve. New clients they onboard benefit from patterns learned across all their brands.
- **Product ROI** — The product gets smarter with every team. Process intelligence compounds across the entire customer base. A new agency signing up today gets optimized specialist sequences from their first session — accumulated from hundreds of teams and thousands of content creation sessions.

The product loop is what makes this a SaaS advantage. Every customer makes the product better for every other customer — without sharing any brand-specific data. And brand-scoped memory means agencies can manage ten clients as cleanly as a single-brand team manages one.

---

## References

- [strategic-memory-utilization-exploration.md](../research/strategic-memory-utilization-exploration.md) — Exploration, taxonomy, open questions
- [storycycle-genie-v2-executable-vision.md](storycycle-genie/storycycle-genie-v2-executable-vision.md) — V2 phases and integration points
- [cma-technical-spec.md](../spec/cma-technical-spec.md) — Strategic Memory Utilization Framework definition
- [cma-authoritative-reference.md](../spec/cma-authoritative-reference.md) — Tier 1 Memory Layer, current gaps
- [glean-context-graph-architecture-considerations.md](../research/glean-context-graph-architecture-considerations.md) — Agent trace feedback loop concept (Phase C source)
