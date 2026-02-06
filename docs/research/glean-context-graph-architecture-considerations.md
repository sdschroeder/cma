# Glean Context Graph: Architecture Considerations for CMA

**Source:** [Arvind Jain (CEO, Glean) — "How do you build a context graph?"](https://x.com/jainarvind/status/2019553277571190821) (February 5, 2026)

**Purpose:** Analyze Glean's context graph approach and identify what is relevant, complementary, or worth considering for our Cognitive Mesh Architecture and collective intelligence ecosystem.

---

## Article Summary

Arvind Jain describes Glean's approach to building **context graphs** — models that connect enterprise entities (people, documents, tickets, systems) with **temporal traces of actions and events** between them. The core argument: AI models can now use tools, but they still lack the **process knowledge** needed to reliably automate work. Systems of record capture decisions, but real work happens in meetings, chats, emails, and docs. Without a structured view of how work actually gets done, AI cannot automate it reliably.

### Key Concepts from the Article

1. **From "what" to "how"** — Traditional knowledge graphs model *things* (customers, tickets, docs). Context graphs model *behavior*: who did what, in which apps, in what order, and with what effect.

2. **Actions as first-class graph entities** — Nodes include actions ("created," "viewed," "approved," "escalated") with timestamps and rich metadata. Edges represent causality and correlation ("Message A triggered Update B at probability P").

3. **Probabilistic path modeling** — Rather than hard-coding workflows, Glean builds distributions of likely paths through processes, giving agents autonomy to pick the most probable path for a given scenario.

4. **Derived "why" layer** — On top of process paths, derived insights capture reasoning behind why "Path A" differed from "Path B," encoding not just the "how" but a probable "why" that can be fed to agents at runtime.

5. **Agent trace feedback loop** — After agents run, their actions become new traces for the context graph. Reinforcement learning evaluates whether chosen paths were optimal, identifies alternatives, and feeds learnings back into the graph.

6. **Three-layer graph architecture:**
   - **Knowledge graph** — factual entities and relationships (people, teams, projects, documents, systems)
   - **Process graph** — temporal, behavioral, and operational layers on top of the knowledge graph
   - **Abstracted traces** — anonymized, stripped of raw text and user identifiers, retaining entity references, process tags, timing features, and outcomes

7. **Hybrid storage model** — Pure graph structures are rigid; raw text is flexible but hard to navigate. Glean uses a hybrid: free-form text broken into smaller chunks with embedded entity IDs, allowing step-by-step traversal.

8. **Graph + orchestration co-ownership** — The context graph must be owned jointly by the data layer and the orchestration layer. Separating them creates drift: the graph evolves one way, agent execution evolves another, producing two divergent versions of reality.

9. **Value-based process prioritization** — High-value processes are identified by analyzing time spent (processes that consume the most time across users are likely highest value). These become the priority targets for agent automation.

---

## Where CMA Already Aligns

These are areas where CMA's existing architecture addresses similar concepts, though sometimes from a different angle.

### Hybrid graph + vector architecture

Glean's conclusion — pure graph is rigid, pure text is hard to navigate, so use a hybrid — directly mirrors CMA's three-tier architecture and the position stated in our [graph-vector-hybrid-alignment](graph-vector-hybrid-alignment.md) research:

- **CMA Tier 2** (Intelligence Storage) = structural foundation (assets, lineage, asset_relationships)
- **CMA Tier 3** (Semantic Layer) = vector index for similarity-based discovery
- Our spec explicitly states: "The graph should not be an optional add-on to a document store; the relationship/lineage model is part of the core data model, and vectors augment retrieval over it."

**Assessment: Strong alignment. No changes needed.**

### Knowledge graph of entities and relationships

Glean's knowledge graph layer (people, teams, projects, documents, systems with typed relationships) corresponds to CMA's Tier 2 intelligence storage with records, record_types, and the planned `asset_relationships` table (or embedded lineage fields). Our existing 38 record types with `strategic_intelligence` metadata, plus the planned graph edges (derives_from, references, supports, contradicts), serve the same structural purpose.

**Assessment: Strong alignment. Our Phase 1 linkage work (relationship fields + optional asset_relationships table) directly addresses this.**

### Agent-driven architecture

Glean emphasizes that agent execution traces feed back into the graph. CMA's architecture is explicitly agent-driven (as documented in the [semantic graph implementation guide](cma-semantic-graph-implementation-guide.md)), where agents make complex strategic decisions rather than relying on event-driven triggers. Both architectures recognize that agents need to be grounded in a live, evolving model.

**Assessment: Aligned in philosophy. The feedback mechanism is where we differ — see considerations below.**

---

## What is New or Different — Considerations for CMA

These are the genuinely novel ideas from the article that CMA does not currently address and that warrant consideration.

### 1. Process/Temporal Layer (High Relevance)

**What Glean does:** Models *how* work happens over time — the sequence of actions, their timing, causality, and outcomes. Actions (created, reviewed, approved, revised, published) are first-class nodes in the graph with timestamps, durations, and causal edges between them.

**What CMA does today:** Models *what* exists — strategic intelligence assets, their content, metadata, and relationships. The `adaptation_history` field and `content_generation_lineage` capture lineage (what was derived from what), but not the full temporal process (how long did review take? what sequence of agent actions led to the final version? where did the process deviate from the typical path?).

**Consideration:** CMA's content creation workflows have natural process patterns:
- Brand Story Discovery → Audience Story → Content Playbook → Content Creation → Enhancement
- Each stage involves specific agent actions, user validations, revisions, and approvals
- The timing and sequence of these steps contains intelligence about workflow effectiveness

**What this could mean for CMA:**
- **Process traces as a data layer** on top of existing intelligence storage — not replacing our content-focused graph, but augmenting it with temporal/behavioral data about how content intelligence is created, validated, and applied
- Could inform the Intelligence Evolution Framework (the 7-stage progression) with empirical data about how intelligence actually evolves in practice vs. the theoretical model
- Could identify bottlenecks or friction points in the Cognitive Amplification Flywheel (Emerge → Capture → Expand → Leverage → Codify → Amplify)

**Relevance to CMA:** Medium-High. Not a gap in our current vision (we're focused on content intelligence, not process automation), but a potential enhancement that could strengthen the feedback loop between the Intelligent Orchestration pillar and the Collective Intelligence Ecosystem.

**Action:** Consider as a future enhancement (Phase 5+). If implemented, it would be a temporal/behavioral layer within Tier 2, complementing the existing asset-and-relationship model. No changes to current spec or implementation plan needed.

---

### 2. Actions as First-Class Graph Entities (Medium Relevance)

**What Glean does:** Makes actions (created, viewed, approved, escalated, commented, resolved) into nodes in the graph, not just properties on edges. Each action has timestamps, metadata about what changed, and causal links to other actions.

**What CMA does today:** Actions are implicit. We track `created_at`, `updated_at` on records. Agent coordination is captured in `multi_agent_coordination_data` (coordinating_agent_roles, handoff_intelligence). But we don't model the sequence of agent actions that produced a content asset as traversable graph data.

**Consideration:** For CMA, the relevant actions would be:
- Agent invocations (which specialist was called, in what order, with what context)
- User validations (where did the user approve, reject, or modify AI-generated content?)
- Content state transitions (draft → reviewed → approved → published)
- Fidelity validations (the critical human step where professional intelligence is captured)

**What this could mean for CMA:**
- Could strengthen the **Professional Intelligence Capture** mechanism — tracking where in the workflow users provide the most valuable corrections or additions
- Could identify which agent specialist sequences produce the highest Strategic Asset Scores
- Could improve the **Embedded Intelligence Architecture** pillar by revealing which agent knowledge produces the best outcomes

**Relevance to CMA:** Medium. Valuable for optimizing agent behavior and understanding professional intelligence emergence patterns, but not essential for the core content intelligence mission. The `multi_agent_coordination_data` in UIC already captures a snapshot; full action-level tracking would be an extension of this.

**Action:** No immediate changes. Consider extending `multi_agent_coordination_data` in a future UIC schema revision to include ordered action sequences and timing data (lightweight version of action-as-entity).

---

### 3. Probabilistic Path Modeling and "Why" Layer (Medium Relevance)

**What Glean does:** Builds distributions of likely paths through processes, allowing agents to pick the most probable path for a scenario. On top of that, derives reasoning about why certain paths differ.

**What CMA does today:** The Intelligence Evolution Framework defines a theoretical progression (Intelligence Discovery → Core Intelligence → ... → Domain Expertise Emergence). The StoryCycle Genie V2 architecture defines fixed phases and specialist invocation patterns. But we don't build probabilistic models of how content creation workflows actually unfold in practice.

**Consideration:** In CMA's context, this would look like:
- Learning that "for B2B SaaS companies, the most effective brand story creation sequence is X → Y → Z" based on Strategic Asset Score outcomes
- Understanding that "when users skip the Audience Story step, content coherence scores drop by 15%"
- Predicting that "this content type typically requires 3 revision cycles before approval"

**What this could mean for CMA:**
- Could make the **Intelligent Orchestration** pillar adaptive rather than prescribed — agents suggest next steps based on learned patterns, not just predefined workflows
- Could feed into **Strategic Memory Utilization** — team-level memories about optimal content creation sequences
- Aligns with the **Cognitive Amplification Flywheel** feedback mechanism

**Relevance to CMA:** Medium. Interesting for making orchestration smarter over time, but the current prescribed workflow model (governance-first, orchestrator + skills) is appropriate for our stage. Probabilistic path modeling is a maturity-stage enhancement.

**Action:** Consider as input to the Strategic Memory Utilization framework (currently in development). Team memories could capture observed workflow patterns and their outcomes without building a full probabilistic model.

---

### 4. Agent Trace Feedback Loop (High Relevance)

**What Glean does:** Every agent execution produces traces that feed back into the context graph. Successful runs reinforce preferred patterns (becoming "natural language playbooks"). Failed or intervention-needed runs highlight anti-patterns. Over time, the graph becomes a joint model of human and agent behavior.

**What CMA does today:** Agent learning is captured through Strategic Memory Governance (agent memories) and the Strategic Asset Score validates content quality. But agent execution traces — which tools were called, in what order, with what inputs/outputs, whether the user accepted or rejected the output — are not systematically captured as intelligence that compounds.

**Consideration:** This is perhaps the most directly relevant concept from the article for CMA. Our architecture already has the philosophical foundation:
- The Cognitive Amplification Flywheel describes a cycle from emergence to codification
- The Intelligence Evolution Framework describes progression from discovery to domain expertise emergence
- Strategic Memory Governance includes agent learning documentation

But **the mechanism for closing the loop** — systematically capturing agent execution patterns and feeding them back as compounding intelligence — is not yet specified as a concrete data model.

**What this could mean for CMA:**
- **Agent execution traces as a new data type** within the Collective Intelligence Ecosystem — not just content assets, but records of how those assets were created, what worked, and what didn't
- **Reinforcement of the "Validated Example Leverage" stage** — currently, validated examples are content assets; they could also include validated agent *processes* (the sequence of actions that produced high-quality content)
- **Empirical foundation for Embedded Intelligence Architecture** — agent knowledge could be updated based on observed outcomes, not just initial research and training
- **Analytics-Enhanced Feedback Loop strengthened** — currently focused on content performance analytics; could include agent execution analytics

**Relevance to CMA:** High. This is an area where CMA's conceptual framework (flywheel, memory governance, intelligence evolution) is strong, but the concrete mechanism for capturing and leveraging agent execution patterns could be more specified.

**Action:** 
1. Consider adding "Agent Execution Intelligence" as a concept within the Collective Intelligence Ecosystem — a lightweight mechanism for capturing which agent sequences and specialist combinations produce the highest-quality outcomes
2. This fits naturally within the Strategic Memory Utilization framework (agent memories that capture "this specialist sequence with this context produced a score of X")
3. No schema changes needed now; the existing memory infrastructure (agent memories) can serve as the initial vehicle

---

### 5. Graph + Orchestration Co-ownership (Medium-High Relevance)

**What Glean does:** Argues that the context graph and orchestration layer must be unified. Separating them creates drift — the graph evolves one way, agent execution evolves another, producing divergent versions of reality.

**What CMA does today:** The four pillars are designed as an integrated system (all four required for valid CMA). The Collective Intelligence Ecosystem (Tier 2 data) and Intelligent Orchestration (agent coordination) are separate pillars but work together. However, the spec doesn't explicitly address the risk of drift between the intelligence model and the orchestration patterns.

**Consideration:** In CMA's context, this risk manifests as:
- Agent orchestration patterns evolving (new skills, changed specialist sequences) without the intelligence graph being updated to reflect those patterns
- Content intelligence accumulating in Tier 2 while agent behavior stays static
- The Strategic Asset Score measuring content quality without feeding back into how agents create content

**What this could mean for CMA:**
- Reinforces the importance of the Analytics-Enhanced Feedback Loop
- Suggests that the Intelligent Orchestration pillar should have explicit read/write access to the Collective Intelligence Ecosystem, not just read access
- Agent orchestration decisions should be *informed by* the accumulated intelligence, and orchestration outcomes should *feed back into* the intelligence

**Relevance to CMA:** Medium-High. The four-pillar integration principle already addresses this conceptually. The practical implementation (especially the feedback path from orchestration outcomes back to intelligence) could be made more explicit.

**Action:** No spec changes needed. This reinforces existing principles. Worth emphasizing in implementation planning that the feedback path from agent execution to intelligence storage is as important as the forward path from intelligence to agent behavior.

---

### 6. Value-Based Process Prioritization (Low-Medium Relevance)

**What Glean does:** Identifies high-value processes by analyzing time spent across users — processes consuming the most collective time are prioritized for agent automation.

**What CMA does today:** Strategic Asset Score measures content value. The Intelligence Evolution Framework prioritizes by intelligence stage. But we don't systematically measure which content creation *processes* consume the most time or produce the most value.

**Consideration:** Less directly applicable to CMA's content intelligence focus. Our "high-value" targets are defined by strategic importance (brand story, audience story, content playbook) rather than by time-spent analysis. The StoryCycle Genie phases already represent a curated, expert-designed prioritization of intelligence building.

**Relevance to CMA:** Low-Medium. Interesting for future optimization (e.g., which enhancement agents save the most time?), but not a gap in the current architecture.

**Action:** No changes needed. Could inform future analytics work.

---

## Summary: Considerations Matrix

| Concept | CMA Alignment | Relevance | Action |
|---------|--------------|-----------|--------|
| **Hybrid graph + vector** | Strong (already in spec) | Validated | None — confirms our approach |
| **Knowledge graph of entities** | Strong (Phase 1 plan) | Validated | None — confirms Phase 1 |
| **Agent-driven architecture** | Strong (existing position) | Validated | None — confirms our approach |
| **Process/temporal layer** | Gap (we model "what," not "how") | Medium-High | Future enhancement (Phase 5+); no current changes |
| **Actions as first-class entities** | Partial (coordination_data exists) | Medium | Consider extending UIC coordination data in future |
| **Probabilistic path modeling** | Gap (we use prescribed workflows) | Medium | Input to Strategic Memory Utilization |
| **Agent trace feedback loop** | Conceptual (flywheel) but not concrete | High | Define "Agent Execution Intelligence" concept; leverage existing memory infrastructure |
| **Graph + orchestration co-ownership** | Strong conceptually (four pillars) | Medium-High | Emphasize feedback path in implementation planning |
| **Value-based process prioritization** | Not applicable (different focus) | Low-Medium | Future analytics consideration |

---

## Key Takeaway

**Glean's context graph operates in a fundamentally different domain than CMA** — Glean models enterprise *process intelligence* (how work gets done across systems), while CMA models *content and strategic intelligence* (how professional expertise is captured, structured, and amplified). The architectures are complementary rather than competing.

**The most valuable insight for CMA is the agent trace feedback loop** — the idea that agent execution patterns should be systematically captured and fed back into the intelligence model so the system learns not just what good content looks like, but what good content *creation processes* look like. This strengthens CMA's existing Cognitive Amplification Flywheel and Analytics-Enhanced Feedback Loop concepts with a concrete mechanism.

**Our hybrid graph + vector approach is independently validated** by Glean's conclusion that pure graph is too rigid and pure text too unstructured — confirming our Tier 2/Tier 3 architecture and the position in our graph-vector-hybrid-alignment research.

**No changes to the current CMA spec or implementation plan are recommended.** The most relevant concept (agent trace feedback) fits naturally within the Strategic Memory Utilization framework already under development, and can be pursued as a maturity-stage enhancement rather than a foundational change.

---

**Document version:** Initial analysis based on Arvind Jain's "How do you build a context graph?" (Feb 5, 2026)
