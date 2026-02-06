# Strategic Memory Utilization: Exploration

**Purpose:** Consolidate everything we know about strategic memory utilization in CMA, identify what's concrete vs. conceptual, map to Brightsy's actual infrastructure, and define a practical path forward.

---

## Table of Contents

1. [What We've Defined](#1-what-weve-defined)
2. [What Exists in Brightsy Today](#2-what-exists-in-brightsy-today)
3. [The Gap: Conceptual vs. Concrete](#3-the-gap-conceptual-vs-concrete)
4. [Memory Taxonomy: What Goes Where](#4-memory-taxonomy-what-goes-where)
5. [Creation and Retrieval Triggers: From Examples to Rules](#5-creation-and-retrieval-triggers-from-examples-to-rules)
6. [The Evolution Framework: How Intelligence Promotes](#6-the-evolution-framework-how-intelligence-promotes)
7. [Agent Execution Intelligence: The Feedback Loop](#7-agent-execution-intelligence-the-feedback-loop)
8. [Integration Points With Other CMA Components](#8-integration-points-with-other-cma-components)
9. [Practical Path Forward](#9-practical-path-forward)
10. [Open Questions](#10-open-questions)

---

## 1. What We've Defined

Strategic Memory Utilization is referenced across six documents. Here is the consolidated picture.

### The Core Idea

Transform existing memory capabilities from ad hoc storage into **systematic intelligence amplification** through strategic memory governance. Instead of random memory creation, every memory serves a defined purpose in the intelligence ecosystem.

**Source: CMA Technical Spec**
> "Transforms existing memory capabilities into systematic intelligence amplification through strategic memory governance"

### Where It Lives in the Architecture

Strategic Memory Utilization maps to **Tier 1: Memory Layer** in the three-tier architecture:

```
┌─────────────────────────────────────────────────────────────────┐
│  TIER 1: MEMORY LAYER                                            │
│  Pointers, thresholds, preferences, memory conventions          │
│  Fast retrieval: what to load, how to behave                    │
│                                                                   │
│  ← STRATEGIC MEMORY UTILIZATION OPERATES HERE                    │
└─────────────────────────────────────────────────────────────────┘
                                    ↓
┌─────────────────────────────────────────────────────────────────┐
│  TIER 2: INTELLIGENCE STORAGE                                    │
│  Full assets, UIC metadata, lineage, relationships              │
└─────────────────────────────────────────────────────────────────┘
                                    ↓
┌─────────────────────────────────────────────────────────────────┐
│  TIER 3: SEMANTIC LAYER                                          │
│  Embeddings, pgvector, similarity search                        │
└─────────────────────────────────────────────────────────────────┘
```

But the *effects* of strategic memory extend to all three tiers — Tier 1 stores the pointers and rules; Tier 2 stores the assets those pointers reference; Tier 3 can be filtered by thresholds set in Tier 1 (e.g. `min_coherence`).

### Three Memory Levels

| Level | Purpose | Examples |
|-------|---------|----------|
| **User Memories** | Individual patterns and preferences that persist across sessions | "Sean prefers step-by-step for complex frameworks"; "Technical depth required before process selection" |
| **Team Memories** | Organizational intelligence and strategic standards | "StoryCycle team responds best to technical authority positioning"; "Framework-focused organizations need implementation examples" |
| **Agent Memories** | Learned behaviors and process improvements | "Long-form content users need more upfront choice architecture"; "Complex frameworks require enhanced option presentation" |

### Memory Contents (Specified in Spec)

| Category | Examples | Naming Convention |
|----------|----------|-------------------|
| **Pointers** | `primary_brand_story_id`, `primary_audience_story_id` | `cma.primary_brand_story_id` |
| **Thresholds** | `strategic_coherence_minimum`, `confidence_auto_approve_threshold` | `cma.threshold.strategic_coherence_minimum` |
| **Preferences** | Workflow mode, option presentation style, tone | `cma.preference.workflow_mode` |

### Governance Protocol (High-Level)

- **Creation Triggers** — Pattern detection, team intelligence, agent learning, strategic thresholds
- **Retrieval Triggers** — Task initiation, agent handoffs, strategic work, quality assurance
- **Evolution Framework** — Session discovery → User Memory → Team Memory → Agent Memory → Framework Protocol

### Role in Each Pillar

| Pillar | Memory's Role |
|--------|--------------|
| **Framework Governance** | Creates Environmental Regularity through memory-driven consistency |
| **Collective Intelligence Ecosystem** | Extends Learning Opportunity to organizational intelligence accumulation |
| **Intelligent Orchestration** | Maintains cognitive flow through Rapid Feedback with memory continuity |
| **Embedded Intelligence Architecture** | Enables Professional Intelligence Density through memory-accumulated intelligence |

### Implementation Status

The technical spec labels this as: **"Infrastructure available, systematic utilization in development."**

Meaning: Brightsy has the memory infrastructure (user/team/agent memories, remember/retrieve tools, RetrievalEngine). What doesn't exist yet: systematic conventions, triggers, and governance for *how* that infrastructure is used in CMA context.

---

## 2. What Exists in Brightsy Today

### Memory Infrastructure

| Component | Description |
|-----------|-------------|
| **memory_resources** | Top-level containers for memory |
| **memory_items** | Individual memory entries within resources |
| **memory_item_chunks** | Chunks with embeddings for semantic retrieval |
| **RetrievalEngine** | `apps/web/lib/server/memory/retrieval-engine.ts` — Implements category/summary/item tiers for retrieval |
| **Remember tool** | Agent tool to create/update memories |
| **Retrieve tool** | Agent tool to search/recall memories |

### Memory Levels Available

- **User Memories** — Persist across sessions for individual users
- **Team Memories** — Shared across team members
- **Agent Memories** — Learned behaviors per agent

### What's Actually In Memory Today (StoryCycle/Brightsy)

- User/team preferences (general)
- **No CMA asset IDs** — No `primary_brand_story_id` or equivalent
- **No CMA thresholds** — No `strategic_coherence_minimum` or confidence thresholds
- **No naming conventions** — No `cma.*` prefix pattern in use
- **No creation/retrieval triggers** — No systematic rules for when to remember or recall

### How Foundation Assets Are Found Today

Agents load brand story and all foundation assets from **records** (by `brand_name` string or `type + filters`), not from memory pointers. This works but is:
- **Slower** — Requires search every session instead of pointer lookup
- **Ambiguous** — Multiple brand stories could match
- **Not context-aware** — No "this user usually works with brand X" optimization

---

## 3. The Gap: Conceptual vs. Concrete

| Aspect | What's Defined | What's Missing |
|--------|---------------|----------------|
| **Memory levels** | User, Team, Agent | No formal rules for what type of intelligence belongs at each level |
| **Naming conventions** | `cma.primary_brand_story_id`, `cma.preference.workflow_mode` | No complete taxonomy; examples only |
| **Creation triggers** | "Pattern detection, team intelligence, agent learning" | No specific rules — "consider calling remember" vs. "MUST call remember when X" |
| **Retrieval triggers** | "Task initiation, agent handoffs, strategic work, QA" | No specific rules for what to retrieve and when |
| **Evolution framework** | Session → User → Team → Agent → Framework Protocol | No criteria for when/how promotion happens |
| **Adaptive UX** | Detailed example with pattern recognition | No implementation mechanism |
| **Memory taxonomy** | Pointers, thresholds, preferences | No formal categorization of intelligence types across memory levels |
| **Agent trace intelligence** | Identified as high-relevance concept (Glean analysis) | Not yet incorporated into memory framework |
| **Integration with Strategic Asset Score** | Not defined | No mechanism for memory to inform or be informed by coherence scoring |
| **Mapping to Brightsy** | General awareness of infrastructure | No specific mapping of CMA memory concepts to memory_resources/memory_items structure |

**The core gap:** We have a strong *philosophical framework* (ad hoc → strategic, three levels, governance protocol) and a concrete *infrastructure* (Brightsy's memory tools), but the bridge between them — the specific rules, taxonomies, and triggers that turn philosophy into practice — is where the work needs to happen.

---

## 4. Memory Taxonomy: What Goes Where

This section proposes a structured taxonomy for what types of intelligence belong at each memory level, organized by CMA function.

### 4.1 User Memories (Individual Context)

**Purpose:** Personalize the cognitive environment for each knowledge worker. Reduce cognitive load by remembering individual patterns, preferences, and working context.

| Category | Key | Example Value | When Created |
|----------|-----|---------------|--------------|
| **Working context** | `cma.context.primary_brand_id` | `uuid` | User selects or confirms brand |
| **Working context** | `cma.context.primary_audience_ids` | `[uuid, ...]` | User selects audiences |
| **Working context** | `cma.context.active_campaign_id` | `uuid` | User starts campaign work |
| **Workflow preferences** | `cma.preference.workflow_mode` | `step_by_step` \| `automated` | User repeatedly chooses one mode |
| **Workflow preferences** | `cma.preference.detail_level` | `comprehensive` \| `concise` | Observed interaction pattern |
| **Workflow preferences** | `cma.preference.option_presentation` | `comprehensive_options` \| `recommended_only` | User behavior at choice points |
| **Content preferences** | `cma.preference.tone` | `professional_authority` | Repeated user corrections |
| **Content preferences** | `cma.preference.content_length` | `long_form_preferred` | Content selection patterns |
| **Cognitive flow** | `cma.flow.pause_points` | `[before_save, after_draft]` | User frequently pauses at these points |
| **Cognitive flow** | `cma.flow.preferred_specialist_depth` | `detailed_narrative` | User engages more with certain specialists |

### 4.2 Team Memories (Organizational Intelligence)

**Purpose:** Establish organizational standards and shared intelligence that apply across all team members. Create Environmental Regularity at the organizational level.

| Category | Key | Example Value | When Created |
|----------|-----|---------------|--------------|
| **Brand defaults** | `cma.team.default_brand_story_id` | `uuid` | Admin sets or team pattern established |
| **Brand defaults** | `cma.team.default_audience_story_ids` | `[uuid, ...]` | Team's primary audiences identified |
| **Quality standards** | `cma.team.coherence_minimum` | `8.0` | Team establishes quality bar |
| **Quality standards** | `cma.team.confidence_auto_approve` | `0.95` | Admin configures automation threshold |
| **Quality standards** | `cma.team.required_specialists` | `[narrative, search]` | Team policy on specialist review |
| **Workflow standards** | `cma.team.content_approval_required` | `true` | Team policy |
| **Workflow standards** | `cma.team.preferred_content_types` | `[blog_post, case_study, email]` | Usage patterns |
| **Strategic patterns** | `cma.team.successful_content_patterns` | `{high_scoring: [...]}` | Patterns that produce high Strategic Asset Scores |
| **Strategic patterns** | `cma.team.brand_voice_standards` | `{tone: ..., complexity: ...}` | Established voice guidelines |

### 4.3 Agent Memories (Process Intelligence)

**Purpose:** Capture learned behaviors about what works and what doesn't in content creation processes. Enable agents to improve over time.

| Category | Key | Example Value | When Created |
|----------|-----|---------------|--------------|
| **Specialist effectiveness** | `cma.agent.specialist_sequence_outcomes` | `{narrative_then_behavioral: {avg_score: 8.5}}` | After content scoring |
| **Specialist effectiveness** | `cma.agent.specialist_value_by_content_type` | `{blog: [narrative, search], email: [behavioral]}` | Accumulated patterns |
| **Content creation patterns** | `cma.agent.avg_revisions_by_type` | `{blog: 2.3, email: 1.1}` | Accumulated creation data |
| **Content creation patterns** | `cma.agent.common_user_corrections` | `[{type: tone, direction: more_professional}]` | Repeated correction patterns |
| **Process intelligence** | `cma.agent.high_value_creation_sequences` | `[{steps: [...], avg_score: 9.1}]` | Best-performing workflows |
| **Process intelligence** | `cma.agent.anti_patterns` | `[{pattern: skip_audience, impact: -15%_coherence}]` | Learned failure modes |
| **Foundation intelligence** | `cma.agent.foundation_load_priorities` | `{always: [brand_story], conditional: [playbook]}` | Optimized retrieval patterns |
| **Recomposition intelligence** | `cma.agent.recomposition_success_rates` | `{blog_to_social: 0.87, blog_to_email: 0.92}` | Recomposition quality tracking |

---

## 5. Creation and Retrieval Triggers: From Examples to Rules

### 5.1 Creation Triggers (When to Remember)

Moving from "consider calling remember" to defined rules.

#### Mandatory Creation Triggers

These MUST fire — they are part of the governance contract.

| Trigger | Memory Level | What to Store | When |
|---------|-------------|---------------|------|
| **Brand confirmed** | User | `cma.context.primary_brand_id` | User confirms or selects a brand for the session |
| **Foundation loaded** | User | `cma.context.primary_audience_ids` | User confirms audience selection |
| **Quality threshold set** | Team | `cma.team.coherence_minimum` | Admin or lead sets quality bar |
| **Approval policy set** | Team | `cma.team.content_approval_required` | Admin configures team policy |

#### Recommended Creation Triggers

These SHOULD fire when conditions are met — they strengthen the intelligence but aren't blocking.

| Trigger | Memory Level | What to Store | Condition |
|---------|-------------|---------------|-----------|
| **Workflow preference detected** | User | `cma.preference.workflow_mode` | User chooses the same mode 3+ times |
| **Detail preference detected** | User | `cma.preference.detail_level` | User consistently requests more/less detail |
| **User override pattern** | User | `cma.preference.confidence_override` | User overrides auto-approve threshold |
| **High-scoring content** | Team | `cma.team.successful_content_patterns` | Content receives Strategic Asset Score > 90 |
| **Specialist sequence outcome** | Agent | `cma.agent.specialist_sequence_outcomes` | After content receives its Strategic Asset Score |
| **Revision pattern** | Agent | `cma.agent.common_user_corrections` | Same type of correction observed 3+ times |
| **Creation sequence** | Agent | `cma.agent.high_value_creation_sequences` | Content creation workflow produces score > 85 |
| **Anti-pattern detected** | Agent | `cma.agent.anti_patterns` | Skipped step correlates with low score |

#### Optional Creation Triggers (Enhancement)

| Trigger | Memory Level | What to Store | Condition |
|---------|-------------|---------------|-----------|
| **Cognitive flow pattern** | User | `cma.flow.pause_points` | User pauses at specific points consistently |
| **Recomposition outcome** | Agent | `cma.agent.recomposition_success_rates` | After recomposed content is scored |

### 5.2 Retrieval Triggers (When to Recall)

#### At Task Initiation

**Every agent session** begins by retrieving:

1. **User working context** — `cma.context.*` (primary brand, audiences, active campaign)
2. **User preferences** — `cma.preference.*` (workflow mode, detail level, tone)
3. **Team quality standards** — `cma.team.coherence_minimum`, `cma.team.confidence_auto_approve`
4. **Team brand defaults** — `cma.team.default_brand_story_id` (fallback if no user context)

**Implementation:** Base framework includes retrieval instructions: "Before starting content creation, retrieve `cma.*` memories and inject into context."

#### At Agent Handoff

When orchestrator delegates to a specialist or hands off to another agent:

1. **Pass relevant memories** — Working context and preferences
2. **Pass specialist effectiveness data** — `cma.agent.specialist_value_by_content_type` for the current content type

#### Before Content Save (QA Gate)

1. **Retrieve team quality standards** — `cma.team.coherence_minimum`
2. **Retrieve team approval policy** — `cma.team.content_approval_required`
3. **Check against user preferences** — Does this match stored tone/style preferences?

#### Before Recomposition

1. **Retrieve recomposition intelligence** — `cma.agent.recomposition_success_rates` for the target format
2. **Retrieve specialist effectiveness** — Which specialists should be invoked for this transformation?

---

## 6. The Evolution Framework: How Intelligence Promotes

The spec defines an evolution path: **Session discovery → User Memory → Team Memory → Agent Memory → Framework Protocol**. But what triggers promotion?

### Promotion Criteria

```
SESSION DISCOVERY
  │
  │  Condition: Pattern observed 3+ times for same user
  ▼
USER MEMORY
  │
  │  Condition: Same pattern observed across 3+ users on the team
  │  OR: Admin/lead explicitly promotes to team standard
  ▼
TEAM MEMORY
  │
  │  Condition: Pattern validated by outcomes (e.g., content using this
  │  pattern consistently scores 85+)
  │  OR: Agent learning identifies causal relationship
  ▼
AGENT MEMORY
  │
  │  Condition: Pattern is universal enough and validated enough
  │  to become a default behavior in the base framework or skills
  │  Requires: Human review and approval
  ▼
FRAMEWORK PROTOCOL
  (Updated base framework or skill instructions)
```

### Promotion Examples

**Example 1: Workflow Preference**
1. **Session:** User Sean chooses step-by-step mode for complex frameworks
2. **User Memory:** After 3 occurrences → `cma.preference.workflow_mode = step_by_step` for Sean
3. **Team Memory:** 4 of 6 team members show same preference → `cma.team.workflow_mode_for_complex = step_by_step`
4. **Agent Memory:** Agent observes that step-by-step mode produces 12% higher scores for complex content → `cma.agent.complex_content_mode_recommendation = step_by_step`
5. **Framework Protocol:** `dual-process-selection` skill updated to default to step-by-step for complex frameworks

**Example 2: Specialist Sequence**
1. **Session:** Content created with Narrative → Behavioral sequence scores 8.7
2. **Agent Memory:** Over 20 blog posts, Narrative-first sequences average 8.5 vs. 7.8 for Behavioral-first → `cma.agent.specialist_sequence_outcomes`
3. **Team Memory:** Pattern validated as team standard → `cma.team.preferred_specialist_sequence.blog = [narrative, behavioral]`
4. **Framework Protocol:** Content Creation Orchestrator updated to prefer Narrative → Behavioral for blog posts

### Safeguards

- **Promotion requires minimum observations** — No single occurrence creates a pattern
- **Demotion path exists** — If outcomes degrade after promotion, revert
- **Human review at Framework Protocol level** — No automatic changes to base/skills
- **Memory expiration** — User context memories (e.g., active campaign) can expire; strategic patterns persist

---

## 7. Agent Execution Intelligence: The Feedback Loop

This section incorporates the insight from the [Glean context graph analysis](glean-context-graph-architecture-considerations.md): **agent execution patterns should be systematically captured and fed back as compounding intelligence.**

### What to Capture

Every content creation session produces process intelligence that can strengthen the memory system:

| Data Point | Source | Memory Target |
|------------|--------|---------------|
| **Specialist invocation sequence** | Content Creation Orchestrator | Agent memory: `specialist_sequence_outcomes` |
| **Specialists invoked** | `coordinating_agent_roles` from UIC metadata | Agent memory: `specialist_value_by_content_type` |
| **User revision count and type** | Session interaction | Agent memory: `common_user_corrections`, `avg_revisions_by_type` |
| **Time from start to approval** | Session timing | Agent memory: creation efficiency patterns |
| **Final Strategic Asset Score** | Strategic Asset Score record | Agent memory: outcome correlation with process |
| **User acceptance/rejection** | Content approval flow | Agent memory: what produces accepted content |
| **Foundation assets used** | `intelligence_assets_referenced` from UIC metadata | Agent memory: `foundation_load_priorities` |
| **Recomposition preservation score** | `adaptation_history.preservation_score` | Agent memory: `recomposition_success_rates` |

### How It Flows Back

```
CONTENT CREATION SESSION
  │
  ├─ Agent produces content using specialist sequence S
  ├─ User revises N times with corrections of type T
  ├─ User approves/rejects
  ├─ Content scored by Strategic Asset Score agent
  │
  ▼
POST-SESSION ANALYSIS (lightweight)
  │
  ├─ Correlate: specialist sequence S → score X
  ├─ Correlate: user corrections T → pattern P
  ├─ Correlate: foundation assets used → score impact
  │
  ▼
MEMORY UPDATES
  │
  ├─ Agent memory: update specialist effectiveness
  ├─ Agent memory: update revision patterns
  ├─ User memory: update preferences (if new pattern detected)
  │
  ▼
NEXT SESSION BENEFITS
  │
  ├─ Orchestrator prefers specialist sequences with higher historical scores
  ├─ Orchestrator anticipates likely user corrections and pre-adjusts
  ├─ Orchestrator loads foundation assets that correlate with highest scores
```

### The Compound Effect

This is how the Cognitive Amplification Flywheel manifests concretely:

- **Emerge** — User works with agents, and professional intelligence surfaces through their corrections and approvals
- **Capture** — Those patterns are captured in agent and user memories
- **Expand** — Agent memories compound: more sessions → better specialist sequencing → higher scores
- **Leverage** — Future sessions benefit from accumulated process intelligence
- **Codify** — Validated patterns promote to team standards and eventually framework protocols
- **Amplify** — The system becomes measurably better: fewer revisions, higher scores, faster time to approval

### Relationship to the "Validated Example Leverage" Stage

The Intelligence Evolution Framework's "Validated Example Leverage" stage currently means: using previously saved *content assets* as approved examples. With agent execution intelligence, this extends to: using previously successful *creation processes* as templates for how to create new content.

This is analogous to Glean's "natural language playbooks" concept — successful agent execution patterns become implicit playbooks for future work.

---

## 8. Integration Points With Other CMA Components

### 8.1 Strategic Asset Score ↔ Memory

**Forward path (Score → Memory):**
- When a Strategic Asset Score is computed, the creation process metadata (specialist sequence, foundation assets used, revision count) should be correlated with the score
- High scores reinforce the creation patterns; low scores flag anti-patterns
- This correlation updates agent memory (`specialist_sequence_outcomes`, `high_value_creation_sequences`, `anti_patterns`)

**Reverse path (Memory → Score):**
- `reference_network_strength` (20% of total score) could be informed by memory about which foundation assets were actually used and how they correlate with quality
- Memory-informed agent behavior (better specialist sequences, better foundation loading) should produce higher scores over time

### 8.2 Selective Retrieval Profiles ↔ Memory

Memory can make selective retrieval *adaptive*:
- If agent memory shows that content playbook fields X, Y, Z correlate with high scores for blog posts, the selective retrieval profile for blog post creation could prioritize those fields
- v1: Static profiles defined in governance
- Future: Memory-informed profiles that adapt based on what fields agents actually need and use

### 8.3 Lineage/Relationships ↔ Memory

Memory can optimize graph traversal:
- Agent memory tracks which foundation asset combinations produce the best outcomes
- Rather than traversing the full graph, agents can use memory shortcuts: "For this user's brand, the most effective foundation combination is brand story + audience story X + playbook Y"
- This is the practical implementation of "memory can shortcut 'which brand story' per context" (from the authoritative reference)

### 8.4 Recomposition ↔ Memory

- Agent memory tracks recomposition success rates by format pair (e.g., blog → social = 0.87 preservation score)
- Memory informs which specialist combination works best for each recomposition type
- User memory captures format preferences: "This user always wants LinkedIn + Twitter from blog posts"

### 8.5 UIC Content Record ↔ Memory

The UIC `multi_agent_coordination_data` already captures a snapshot of agent coordination. Strategic memory extends this:
- `creation_context.coordinating_agents` is the per-record snapshot
- Agent memory aggregates these snapshots into patterns across many records
- This is the difference between "this blog post used Narrative + Behavioral" (record) and "Narrative + Behavioral produces 15% higher blog post scores than Narrative alone" (memory)

---

## 9. Practical Path Forward

### What to Do Now (V2 Implementation — Phase 1/2)

These items are already planned in the V2 Executable Vision and should be prioritized.

**1. Define the naming convention (documentation)**
- Formalize the `cma.*` prefix pattern
- Publish complete taxonomy (Section 4 above as starting point)
- Deliverable: Memory conventions section in base framework

**2. Implement mandatory retrieval triggers (base framework)**
- At task initiation: retrieve `cma.context.*`, `cma.preference.*`, `cma.team.*`
- At QA gate: retrieve quality standards
- Deliverable: Retrieval instructions in `storycycle-base-optimized.md`

**3. Implement mandatory creation triggers (base framework)**
- Brand confirmed → store `cma.context.primary_brand_id`
- Foundation loaded → store context
- Deliverable: Creation trigger instructions in base framework

### What to Do Next (V2 Implementation — Phase 5+)

**4. Implement recommended creation triggers (orchestrator + skills)**
- Workflow preference detection (3+ occurrences)
- High-scoring content patterns (Strategic Asset Score > 90)
- Deliverable: Trigger logic in Content Creation Orchestrator

**5. Implement agent execution intelligence (lightweight)**
- After content is scored: correlate specialist sequence with score
- Store specialist effectiveness data in agent memory
- Deliverable: Post-scoring memory update logic

### Future Enhancements

**6. Evolution framework automation**
- Build promotion logic: user → team (when N users share a pattern)
- Build validation logic: correlate patterns with outcomes before promoting
- Deliverable: Memory evolution agent or governance logic

**7. Adaptive selective retrieval**
- Memory-informed field selection based on what agents actually need
- Deliverable: Dynamic selective retrieval profiles

**8. Memory analytics and monitoring**
- Track memory usage: which memories are retrieved most? Which produce behavior changes?
- Track memory effectiveness: do memory-informed sessions produce higher scores?
- Deliverable: Memory health dashboard or reporting

---

## 10. Open Questions

### Architecture Questions

1. **Memory granularity** — Should `cma.context.primary_brand_id` be a single memory item, or should all CMA context be one structured memory item with multiple fields? (Single item per key is simpler; structured item reduces retrieval calls.)

2. **Memory retrieval cost** — Each memory retrieval adds latency and potentially tokens to context. What's the practical limit on how many memories an agent should retrieve at session start? Does the RetrievalEngine's tiered approach (category → summary → item) handle this well enough?

3. **Memory vs. records for process intelligence** — Agent execution intelligence (specialist sequences, outcomes) could live in memory OR in a dedicated record type (e.g., "Agent Execution Log"). Memory is lightweight and fast; records are structured and queryable. Which is the right vehicle for different types of process intelligence?

4. **Memory expiration** — Working context memories (e.g., active campaign) should expire. Strategic patterns (e.g., specialist effectiveness) should persist. How to implement expiration in Brightsy's memory infrastructure?

### Governance Questions

5. **Who can promote to team memory?** — Only admins? Any team member after enough observations? The agent automatically?

6. **How to handle conflicting user preferences?** — If 3 users prefer step-by-step and 3 prefer automated, what does the team memory say?

7. **Memory and multi-brand context** — Users who work across multiple brands need brand-scoped memories. Does `cma.context.primary_brand_id` scope everything else, or do we need `cma.brand.{brand_id}.preference.*`?

### Measurement Questions

8. **How do we validate that strategic memory utilization is working?** — What metrics prove that memory-informed sessions are better than memory-less sessions?

9. **Baseline measurement** — Before implementing, we need baseline measurements of: average revision count, average Strategic Asset Score, average time to content approval. Without baselines, we can't measure improvement.

10. **When does memory become noise?** — At what point does accumulated memory create confusion rather than clarity? What's the pruning strategy?

---

## References

**CMA Spec:**
- [cma-technical-spec.md](../spec/cma-technical-spec.md) — §Strategic Memory Utilization Framework, §Memory Conventions, §Strategic Memory Utilization Requirements
- [cma-authoritative-reference.md](../spec/cma-authoritative-reference.md) — §Tier 1 Memory Layer, §Gaps (Memory), §Phase 3 (Optional Memory)

**Implementation:**
- [storycycle-genie-v2-executable-vision.md](../implementation/storycycle-genie/storycycle-genie-v2-executable-vision.md) — Phase 1 (memory conventions), Phase 2 (base framework with memory)
- [storycycle-genie-complete-vision-extension.md](../archive/storycycle-genie-complete-vision-extension.md) — Phase 13 (Strategic Memory Integration) — archived, superseded by V2
- [cma+brightsy-implementation-plan.md](../implementation/cma+brightsy-implementation-plan.md) — Step 2 (Memory integration)

**Research:**
- [glean-context-graph-architecture-considerations.md](glean-context-graph-architecture-considerations.md) — Agent trace feedback loop concept

**Whitepaper:**
- [cma-whitepaper.md](../thought-leadership/cma-whitepaper.md) — Strategic Memory as dynamic intelligence facilitation, three-tier architecture narrative
