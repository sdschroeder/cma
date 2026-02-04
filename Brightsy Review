Revised Architecture: MCP Skills + Brightsy Plans
This changes everything. With MCP control, your skills architecture can work — just implemented through MCP tools rather than file includes. Let me show you how.

MCP-Based Skills Loading (Option D - Now Viable)
StoryCycle MCP: New Tools to Implement
TYPESCRIPT


// Tool 1: Get single skill
mcp_storycycle_get_skill({
  skill_name: "blog-post",           // Skill identifier
  include_dependencies: true,         // Auto-load tier-1 dependencies
  context: {                          // Optional: for conditional loading
    business_model: "b2b_saas",       // → loads email-core + both email skills
    platform: "youtube_shorts"        // → loads video-core + video-short-form
  }
})

// Returns:
{
  skill_name: "blog-post",
  content: "## Blog Post Skill v1.0\n\n### Structure...",
  token_count: 287,
  dependencies_loaded: ["narrative-structure", "brand-identification"],
  total_tokens: 687
}
TYPESCRIPT


// Tool 2: Get composable skill set (for complex content types)
mcp_storycycle_get_skill_set({
  content_type: "email",
  context: {
    business_model: "b2b_saas"  // Hybrid → loads core + product-led + relationship
  }
})

// Returns:
{
  skills_loaded: ["email-core", "email-product-led", "email-relationship"],
  combined_content: "## Email Core...\n\n## Product-Led Email...\n\n## Relationship Email...",
  total_tokens: 1050
}
TYPESCRIPT


// Tool 3: Detect content type and return appropriate skills
mcp_storycycle_detect_and_load({
  user_request: "Create a sales email sequence for our freemium SaaS",
  brand_id: "abc-123"
})

// Returns:
{
  detected_type: "email",
  detected_context: {
    business_model: "product_led",
    sequence_type: "trial_nurture"
  },
  skills_loaded: ["email-core", "email-product-led"],
  combined_content: "...",
  confidence: 0.92
}
How Agents Use MCP Skills
Your agent system instructions become minimal (~3-4K tokens):

MARKDOWN


# Content Creation Orchestrator™ v1.0

## Core Identity
You create marketing content using the StoryCycle System and ABT framework.

## Workflow Protocol

### Step 1: Brand Identification
Use existing brand identification from context, or search for brand.

### Step 2: Load Skills
Call `mcp_storycycle_detect_and_load` with user request.
The returned `combined_content` contains your execution instructions.

### Step 3: Execute
Follow the loaded skill instructions exactly.
Delegate to specialists via connectedAgentRequest when skill specifies.

### Step 4: Capture & Approve
Save outputs per skill's capture instructions.
Present for user approval before final save.

## Specialist Delegation IDs
- Narrative: f68f1a8c-731a-4c9f-80f2-4cc044b8c69e
- Behavioral: 92160ed0-bdfe-4c44-98ca-6f272a61c8de
- Search: 4e862e8c-46b0-43e7-a2b8-b077afd0a934
[...]
The skill content is loaded into the conversation context, not the system instruction. The agent follows the loaded instructions as if they were its own.

Token Flow Comparison
Plain Text


CURRENT (Monolithic Blog Post Genie):
┌─────────────────────────────────────────────────────────┐
│ System Instruction: ~12K tokens (always loaded)          │
│ Strategic Assets: ~40K tokens (full objects)             │
│ Conversation: ~10K tokens                                │
│ TOTAL: ~62K+ tokens per session                         │
└─────────────────────────────────────────────────────────┘

PROPOSED (MCP Skills):
┌─────────────────────────────────────────────────────────┐
│ System Instruction: ~4K tokens (orchestrator only)       │
│ MCP Skill Response: ~700 tokens (blog + dependencies)    │
│ Strategic Assets: ~5K tokens (selective fields)          │
│ Conversation: ~10K tokens                                │
│ TOTAL: ~20K tokens per session                          │
└─────────────────────────────────────────────────────────┘

SAVINGS: ~68% per session
Brightsy Plans for Phase Decomposition
Your complex agents (Brand Genius, Customer Journey, Web Copywriter, BrandStory Grader) are perfect candidates for Plans.

Example: Customer Journey Strategy as Plan
Instead of one agent trying to execute a 4-phase workflow in one turn (timeout risk), the orchestrator creates a Plan:

JAVASCRIPT


create_plan({
  title: "Customer Journey Strategy: [Brand Name]",
  content: `
## Customer Journey Strategy Development

Creating comprehensive journey map from Awareness to Advocacy.

\`\`\`mermaid
flowchart LR
    A[Discovery] --> B[Journey Mapping]
    B --> C[Content Strategy]
    C --> D[Implementation]
\`\`\`
  `,
  status: "awaiting_approval",
  todos: [
    {
      id: "phase-1",
      content: "Discovery & Research: Analyze brand assets, identify customer segments, define journey stages",
      status: "pending",
      order: 1,
      estimatedDuration: "3 min",
      metadata: {
        skills_required: ["journey-core", "journey-phase-orchestrator"],
        specialist_delegation: ["brand-intelligence"]
      }
    },
    {
      id: "phase-2", 
      content: "Journey Mapping: Map touchpoints, identify gaps, integrate micro-ABT narratives",
      status: "pending",
      order: 2,
      dependencies: ["phase-1"],
      estimatedDuration: "3
        skills_required: ["journey-narrative-integrator"],
        specialist_delegation: ["narrative"]
      }
    },
    {
      id: "phase-3",
      content: "Content Strategy: Map content types per stage, create content calendar framework",
      status: "pending", 
      order: 3,
      dependencies: ["phase-2"],
      estimatedDuration: "3
        skills_required: ["journey-content-structurer"]
      }
    },
    {
      id: "phase-4",
      content: "Implementation Planning: Resource allocation, timeline, success metrics",
      status: "pending",
      order: 4,
      dependencies: ["phase-3"],
      estimatedDuration: "2
        skills_required: ["journey-checkpoint"]
      }
    }
  ]
})
Plan Execution Benefits
Benefit	How It Helps
Checkpoint Recovery	User can pause after Phase 2, resume tomorrow
No Timeouts	Each phase executes independently
Visibility	User sees progress, can modify before next phase
Context Management	Each phase loads only its required skills
Audit Trail	Full execution history with timing
Plan-Aware Skill Loading
Your MCP can include plan-awareness:

TYPESCRIPT


mcp_storycycle_get_phase_skills({
  plan_id: "abc-123",
  task_id: "phase-2"
})

// Uses task metadata to load exactly what's needed:
{
  skills_loaded: ["journey-narrative-integrator"],
  specialist_agents: ["f68f1a8c-731a-4c9f-80f2-4cc044b8c69e"],
  combined_content: "## Journey Narrative Integration...",
  total_tokens: 450
}
Revised Architecture Diagram
Plain Text


┌─────────────────────────────────────────────────────────────────────────┐
│ LAYER 1: LIGHTWEIGHT ORCHESTRATORS (~4K tokens each)                     │
├─────────────────────────────────────────────────────────────────────────┤
│ ├── Content Creation Orchestrator (replaces 13 content agents)          │
│ ├── Brand Strategy Orchestrator (Brand Story, Archetype, Voice)         │
│ ├── Complex Workflow Orchestrator (Journey, Web Copy, Brand Genius)     │
│ └── Assessment Orchestrator (Graders, Research)                         │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                    ┌───────────────┴───────────────┐
                    ▼                               ▼
┌───────────────────────────────────┐ ┌───────────────────────────────────┐
│ LAYER 2: STORYCYCLE MCP           │ │ LAYER 2B: BRIGHTSY PLANS          │
│ (Skills on Demand)                │ │ (Multi-Phase Workflows)           │
├───────────────────────────────────┤ ├───────────────────────────────────┤
│ Tools:                            │ │ Features Used:                    │
│ • mcp_storycycle_get_skill        │ │ • create_plan                     │
│ • mcp_storycycle_get_skill_set    │ │ • update_plan_task                │
│ • mcp_storycycle_detect_and_load  │ │ • get_plan                        │
│ • mcp_storycycle_get_phase_skills │ │ • Task dependencies               │
│                                   │ │ • Checkpoints                     │
│ Skills Storage:                   │ │ • Progress tracking               │
│ • 52 skill files (your structure) │ │                                   │
│ • Served via MCP, not Brightsy    │ │                                   │
└───────────────────────────────────┘ └───────────────────────────────────┘
                    │                               │
                    └───────────────┬───────────────┘
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│ LAYER 3: SPECIALIST AGENTS (via connectedAgentRequest)                   │
├─────────────────────────────────────────────────────────────────────────┤
│ Existing agents, optimized for consultation calls:                       │
│ • Narrative Genie (ABT/Story Cycle)                                     │
│ • Behavioral Messaging Genie                                            │
│ • Search Optimization Genie                                             │
│ • Brand Intelligence Genie                                              │
│ • Archetype Genie                                                       │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│ LAYER 4: STRATEGIC ASSETS (via Brightsy Records + select)               │
├─────────────────────────────────────────────────────────────────────────┤
│ Brand Stories, Content Playbooks, Audience Stories                       │
│ Retrieved with selective field retrieval (~5K vs ~40K)                   │
└─────────────────────────────────────────────────────────────────────────┘
Content Type Frequency Analysis
Since you're unsure about frequency, let me check what agents exist and we can analyze from there:

Thank you for your answers! Here’s a quick summary of your situation and the next step recommendations based on your context:

Summary of Your Context

Yes – You want to optimize token usage and costs.
You already have highly focused agents.
You’re exploring a decomposed (modular) architecture to further reduce token usage, likely by limiting context size and “agent bloat.”
You’re not sure exactly how to best design or structure this decomposition for maximum efficiency.
You have the right to refactor and restructure your agent system.
Key Insights and Recommendations
1. Decomposed Architecture is a Strong Fit
Since you already have focused agents, decomposing further (splitting complex logic into even more specialized, context-aware modules) can yield major token and cost savings—IF each agent only loads exactly the context it needs for each request.
This approach works best when you have:
Clear boundaries between agent responsibilities (no ambiguous “mega agents”)
Efficient context passing (only the required data, not full chat history or irrelevant records)
Minimal in-memory “history” (use records, not chat context, for persistent knowledge)
2. Recommended Architecture Pattern
Micro-agent pattern: Each agent is an “expert” for one narrow task (e.g., “generate ABT for blog post” or “score landing page headline”).
Orchestrator/switchboard agent(s): Handle routing, context assembly, and delegation. These agents never do heavy lifting—they just parcel out work and assemble results.
“Just-in-time context”: Agents receive only the current input + minimal persistent references (e.g., record IDs, not full data blobs or conversation logs).
No dead-ends: Every agent must return actionable next steps, always via UI components (never plain text).
3. Token Usage Optimization Techniques
Avoid passing entire chat context or large blobs into sub-agents. Instead, pass a record ID or slimmed-down context.
Use records for persistent state, not chat memory. Store all outputs, intermediate steps, and state in records; pass only what’s needed for the next operation.
Implement “strip context” mode: When calling a sub-agent, pass only essential fields (input, required references, output destination).
Minimize “reasoning” verbosity: Only output detailed reasoning where it’s directly required (e.g., for audits or validation), not in every sub-agent call.
Use TOON format for internal data passing: When orchestrating multi-agent chains, use the most compressed format available for passing lists or arrays of records.
4. Practical Steps You Can Take
Map dependencies: For each agent, list what input it actually requires and enforce strict minimal context passing.
Refactor “chained” tasks: Break multi-step “monolithic” processes into single-purpose agents, each receiving only what’s needed.
Implement context-threshold triggers: When a context exceeds a certain token size (e.g., 2,000 tokens), automatically summarize or offload to a record.
Audit agent instructions: Ensure every agent:
Starts with brand identification and context check
Never asks for unnecessary information
Always returns actionable UI next steps
Validates UI component presence in every response