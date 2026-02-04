---
name: Unified Token Optimization & Content Consolidation
overview: "Complete system optimization: base framework, skills architecture, selective retrieval, and content agent consolidation"
todos: []
isProject: true
---

> **Superseded by:** [storycycle-genie-v2-executable-vision.md](../implementation/storycycle-genie/storycycle-genie-v2-executable-vision.md) — the single reference for StoryCycle Genie implementation (goals, architecture, phases, skills, UIC, recomposition). This doc is retained for **design history** only. *Archived in docs/archive/.*

---

# Unified Token Optimization & Content Consolidation Plan

## Executive Summary

**Goal**: Dramatically reduce token usage, eliminate redundancy, and simplify maintenance across the entire StoryCycle agent system.

**Approach**:

1. Extract optimized base framework from source modules
2. Create reusable skills architecture (tier 1-4)
3. Implement selective field retrieval for strategic assets
4. Consolidate 13 content creation agents into 1 orchestrator + content-type skills

**Test Strategy**: All validation via StoryCycle MCP in Cursor (proxy to Brightsy with full tool access)

**Expected Results**:

| Component | Current | Target | Reduction |

|-----------|---------|--------|-----------|

| Base Framework | 76K tokens | 8-10K | 87% |

| Strategic Assets | 43K tokens | 5K | 88% |

| Content Agents (13 total) | 152K tokens | 15K | 90% |

| Typical Session | 130K+ tokens | 20-25K | **80-85%** |

---

## Architecture

```
┌─────────────────────────────────────────────────────────────────────────┐
│ LAYER 1: BASE FRAMEWORK (Always Loaded)                                  │
├─────────────────────────────────────────────────────────────────────────┤
│ storycycle-base-optimized.md (~8-10K tokens)                            │
│ ├── Pre-response validation, user agency preservation                    │
│ ├── State management, voice system (Park Howell/Professional)           │
│ ├── Brand identification protocol, asset type registry                   │
│ ├── Record management basics, UI component basics                        │
│ ├── Selective field retrieval standards                                  │
│ ├── Streaming UI patterns                                                │
│ └── Progressive loading directives                                       │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│ LAYER 2: ON-DEMAND SKILLS (Load as Workflow Requires)                    │
├─────────────────────────────────────────────────────────────────────────┤
│ skills/                                                                  │
│ ├── tier-1-core/           (~200 tokens each)                           │
│ │   ├── brand-identification.md                                          │
│ │   ├── framework-compliance.md                                          │
│ │   └── navigation-standards.md                                          │
│ │                                                                        │
│ ├── tier-2-specialists/    (~300 tokens each)                           │
│ │   ├── narrative-structure.md      (ABT, Story Cycle)                  │
│ │   ├── archetype-analysis.md       (Brand personality)                 │
│ │   ├── behavioral-enhancement.md   (Persuasion)                        │
│ │   └── search-optimization.md      (SEO/discovery)                     │
│ │                                                                        │
│ ├── tier-3-workflows/      (~200 tokens each)                           │
│ │   ├── strategic-asset-discovery.md                                     │
│ │   ├── selective-field-profiles.md                                      │
│ │   ├── specialist-integration.md                                        │
│ │   ├── dual-process-selection.md                                        │
│ │   ├── checkpoint-enforcement.md                                        │
│ │   └── strategic-intelligence-capture.md                                │
│ │                                                                        │
│ ├── tier-4-utilities/      (~150 tokens each)                           │
│ │   ├── pdf-generation.md                                                │
│ │   └── content-approval.md                                              │
│ │                                                                        │
│ ├── content-types/         ◄── CONTENT SKILLS                           │
│ │   │── Simple (single skill each)                                      │
│ │   ├── blog-post.md                                                    │
│ │   ├── case-study.md                                                   │
│ │   ├── press-release.md                                                │
│ │   ├── one-pager.md                                                    │
│ │   ├── product-description.md                                          │
│ │   ├── sales-email.md                                                  │
│ │   ├── substack.md                                                     │
│ │   └── documentation.md                                                │
│ │   │── Composable (load multiple based on context)                     │
│ │   ├── email/                                                          │
│ │   │   ├── email-core.md                                               │
│ │   │   ├── email-product-led.md                                        │
│ │   │   └── email-relationship.md                                       │
│ │   ├── video/                                                          │
│ │   │   ├── video-core.md                                               │
│ │   │   ├── video-short-form.md                                         │
│ │   │   ├── video-long-form.md                                          │
│ │   │   └── video-professional.md                                       │
│ │   ├── longform/                                                       │
│ │   │   ├── longform-core.md                                            │
│ │   │   ├── longform-whitepaper.md                                      │
│ │   │   ├── longform-ebook.md                                           │
│ │   │   └── longform-research.md                                        │
│ │   ├── presentation/                                                   │
│ │   │   ├── presentation-core.md                                        │
│ │   │   ├── presentation-sales.md                                       │
│ │   │   └── presentation-investor.md                                    │
│ │   └── webinar/                                                        │
│ │       ├── webinar-core.md                                             │
│ │       ├── webinar-leadgen.md                                          │
│ │       └── webinar-educational.md                                      │
│ │                                                                       │
│ ├── brand-story/           ◄── NON-CONTENT COMPOSABLE                   │
│ │   ├── brand-core.md                                                   │
│ │   ├── brand-b2b.md                                                    │
│ │   ├── brand-b2c.md                                                    │
│ │   └── brand-service.md                                                │
│ │                                                                       │
│ ├── voice-discovery/       ◄── NON-CONTENT COMPOSABLE                   │
│ │   ├── voice-core.md                                                   │
│ │   ├── voice-sample-analysis.md                                        │
│ │   └── voice-interview.md                                              │
│ │                                                                       │
│ └── search-optimization/   ◄── NON-CONTENT COMPOSABLE                   │
│     ├── search-core.md                                                  │
│     ├── search-web-seo.md                                               │
│     ├── search-ai-geo.md                                                │
│     ├── search-voice.md                                                 │
│     └── search-visual.md                                                │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│ LAYER 3: SELECTIVE DATA RETRIEVAL                                        │
├─────────────────────────────────────────────────────────────────────────┤
│ Strategic Assets via `select` property:                                  │
│ ├── Brand Story: ~2K tokens (vs 15K full)                               │
│ ├── Content Playbook: ~2K tokens (vs 20K full)                          │
│ └── Audience Story: ~1K tokens (vs 8K full)                             │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## Directory Structure

```
storycycle-assets/
├── framework/
│   ├── storycycle-base-optimized.md      # NEW: Compressed base (~8-10K)
│   ├── progressive-loading-map.md        # NEW: When to load what
│   ├── storycycle-governance.md          # ARCHIVE: Reference only
│   ├── 1-framework-core.md               # SOURCE: Load for edge cases
│   ├── 6-record-management-framework.md  # SOURCE: Load for complex ops
│   ├── 9-ui-guidelines.md                # SOURCE: Load for custom UI
│   └── [other source modules]
│
├── skills/
│   ├── README.md                          # Skills documentation
│   │
│   ├── tier-1-core/                       # Universal (all agents)
│   │   ├── brand-identification.md
│   │   ├── framework-compliance.md
│   │   └── navigation-standards.md
│   │
│   ├── tier-2-specialists/                # Specialist logic
│   │   ├── narrative-structure.md
│   │   ├── archetype-analysis.md
│   │   ├── behavioral-enhancement.md
│   │   └── search-optimization.md
│   │
│   ├── tier-3-workflows/                  # Complex patterns
│   │   ├── strategic-asset-discovery.md
│   │   ├── selective-field-profiles.md
│   │   ├── specialist-integration.md
│   │   ├── dual-process-selection.md
│   │   ├── checkpoint-enforcement.md
│   │   └── strategic-intelligence-capture.md
│   │
│   ├── tier-4-utilities/                  # Helpers
│   │   ├── pdf-generation.md
│   │   └── content-approval.md
│   │
│   ├── content-types/                     # Content-specific skills
│   │   │── Simple (single skill each)
│   │   ├── blog-post.md
│   │   ├── case-study.md
│   │   ├── press-release.md
│   │   ├── one-pager.md
│   │   ├── product-description.md
│   │   ├── sales-email.md
│   │   ├── substack.md
│   │   └── documentation.md
│   │   │
│   │   │── Composable Content Types
│   │   ├── email/
│   │   │   ├── email-core.md
│   │   │   ├── email-product-led.md
│   │   │   └── email-relationship.md
│   │   ├── video/
│   │   │   ├── video-core.md
│   │   │   ├── video-short-form.md
│   │   │   ├── video-long-form.md
│   │   │   └── video-professional.md
│   │   ├── longform/
│   │   │   ├── longform-core.md
│   │   │   ├── longform-whitepaper.md
│   │   │   ├── longform-ebook.md
│   │   │   └── longform-research.md
│   │   ├── presentation/
│   │   │   ├── presentation-core.md
│   │   │   ├── presentation-sales.md
│   │   │   └── presentation-investor.md
│   │   └── webinar/
│   │       ├── webinar-core.md
│   │       ├── webinar-leadgen.md
│   │       └── webinar-educational.md
│   │
│   ├── brand-story/                       # Non-content composable
│   │   ├── brand-core.md
│   │   ├── brand-b2b.md
│   │   ├── brand-b2c.md
│   │   └── brand-service.md
│   │
│   ├── voice-discovery/                   # Non-content composable
│   │   ├── voice-core.md
│   │   ├── voice-sample-analysis.md
│   │   └── voice-interview.md
│   │
│   └── search-optimization/               # Non-content composable
│       ├── search-core.md
│       ├── search-web-seo.md
│       ├── search-ai-geo.md
│       ├── search-voice.md
│       └── search-visual.md
│
├── agents/
│   ├── content-creation-orchestrator.md   # NEW: Replaces 13 agents
│   └── [other optimized agents]
│
├── testing/
│   ├── token-counter.py
│   ├── quality-grader.md
│   ├── test-scenarios.md
│   └── results/
│       ├── baseline/
│       └── optimized/
│
└── agents/content-creation-agents/        # ARCHIVE after migration
    └── [13 original agents - archived]
```

---

## Testing Strategy: StoryCycle MCP

**All testing happens in Cursor via StoryCycle MCP** - proxy to Brightsy with full tool access.

| Test Type | What It Validates |

|-----------|-------------------|

| Token Counting | Reduction math is correct |

| Skills Loading | Files load on-demand via Brightsy |

| Selective Retrieval | `select` property reduces tokens |

| Quality Comparison | Optimized output matches baseline |

| Streaming UI | `show_buttons` works, no blocking |

| End-to-End | Complete workflows function |

---

## Implementation Phases

### PHASE 1: Infrastructure Setup

#### 1.1 Create Testing Infrastructure

```bash
mkdir -p testing/results/{baseline,optimized}
mkdir -p skills/{tier-1-core,tier-2-specialists,tier-3-workflows,tier-4-utilities,content-types}
```

**Files to create:**

- `testing/token-counter.py` - Multi-model token counting
- `testing/quality-grader.md` - 8-dimension quality rubric
- `testing/test-scenarios.md` - Standardized test cases

#### 1.2 Baseline Measurement

Measure current state:

- Framework tokens (storycycle-framework-optimized-v3.md)
- Sample agent tokens (brand-genius-genie.md, blog-post-genie.md)
- Strategic asset tokens (full retrieval)

Run baseline quality tests via MCP for comparison.

---

### PHASE 2: Base Framework Extraction

#### 2.1 Analyze Source Modules

Read and categorize requirements from:

- `1-framework-core.md` - Execution, voice, UI
- `6-record-management-framework.md` - Records, search
- `9-ui-guidelines.md` - UI components
- `11-compliance.md` - State management

**Categorize as:**

- **Universal** (all agents need) → Extract to base
- **Common** (most agents need) → Extract to tier-3 skills
- **Specialized** (few agents need) → Keep in source modules

#### 2.2 Create storycycle-base-optimized.md

**Target: ~8-10K tokens**

Extract from source modules:

- Pre-response validation (from 1-framework-core.md)
- User agency preservation (from 1-framework-core.md)
- State management structure (from 11-compliance.md)
- Voice system basics (from 1-framework-core.md)
- Brand identification protocol (from 6-record-management-framework.md)
- Asset type registry (from 6-record-management-framework.md)
- UI component basics (from 9-ui-guidelines.md)

Add new sections:

- Selective field retrieval standards
- Streaming UI patterns
- Progressive loading directives
- Inter-agent response protocol (CRAFT-inspired, ~200 tokens)

**Inter-Agent Response Protocol** (optional, for `connectedAgentRequest` responses):

```
When responding to inter-agent API calls, structure responses as:
- Context: Brand ID, confidence level, workflow state
- Role: Agent identity, expertise boundary
- Action: Primary deliverable (structured JSON)
- Format: Output specification
- Tone: Professional (for inter-agent, not Park Howell)
```

This standardizes communication between orchestrators and specialists.

#### 2.3 Create progressive-loading-map.md

Document when to load additional modules/skills:

| Workflow Need | Load | Tokens |

|---------------|------|--------|

| Brand identification | tier-1-core/brand-identification.md | ~200 |

| Asset discovery | tier-3-workflows/strategic-asset-discovery.md | ~200 |

| Archetype work | tier-2-specialists/archetype-analysis.md | ~300 |

| Custom UI | framework/9-ui-guidelines.md (full) | ~15K |

| Complex records | framework/6-record-management-framework.md | ~10K |

---

### PHASE 3: Skills Extraction

#### 3.1 Tier-1 Core Skills

**Extract from existing agents** (blog-post-genie.md is cleanest source):

| Skill | Lines | Content |

|-------|-------|---------|

| `brand-identification.md` | ~100 | Detection patterns, search, confirmation UI |

| `framework-compliance.md` | ~50 | Pre-response validation checklist |

| `navigation-standards.md` | ~50 | No dead-ends, show_buttons patterns |

#### 3.2 Tier-2 Specialist Skills

**Extract from specialist agents:**

| Skill | Source | Content |

|-------|--------|---------|

| `narrative-structure.md` | Narrative Genie | ABT formula, Story Cycle integration |

| `archetype-analysis.md` | Archetype Genie | Selection process, brand infusion |

| `behavioral-enhancement.md` | Behavioral Genie | Silent enhancement, triggers |

| `search-optimization.md` | UniversalSearch Genie | SEO patterns, discovery |

#### 3.3 Tier-3 Workflow Skills

**Extract from agents + framework:**

| Skill | Lines | Content |

|-------|-------|---------|

| `strategic-asset-discovery.md` | ~120 | 5-step search, priority order |

| `selective-field-profiles.md` | ~80 | Field selection by use case |

| `specialist-integration.md` | ~100 | When/how to delegate |

| `dual-process-selection.md` | ~80 | StoryCyclone vs step-by-step + confidence thresholds |

**Confidence Threshold Extension** (for `dual-process-selection.md`):

| Confidence | Behavior |

|------------|----------|

| 0.95+ | Full automation, no user approval needed |

| 0.85-0.94 | Auto-execute, present result for approval |

| 0.70-0.84 | Present options, recommend one |

| <0.70 | Step-by-step guided workflow |

This extends the existing 80% assessment accuracy pattern with granular routing.

| `checkpoint-enforcement.md` | ~30 | Progress validation |

| `strategic-intelligence-capture.md` | ~100 | What/when/how to capture |

#### 3.4 Tier-4 Utility Skills

| Skill | Lines | Content |

|-------|-------|---------|

| `pdf-generation.md` | ~100 | Template selection, generation |

| `content-approval.md` | ~80 | Approval UI, save workflow |

#### 3.5 Cleanup Redundant Files

After extraction, archive:

- `framework/strategic-intelligence-capture-framework.md`
- `framework/strategic-intelligence-helper-functions.md`
- `framework/strategic-intelligence-metadata-implementation.md`
- `framework/strategic-intelligence-*.json` files

---

### PHASE 4: Validation via MCP

#### 4.1 Test Skills Loading

Via StoryCycle MCP:

1. Call agent that references skills
2. Verify skill content loads correctly
3. Confirm progressive loading works

#### 4.2 Test Selective Field Retrieval

```javascript
// Test via MCP
get_records({
  type_id: "60a659ea-f2e3-41d2-83a8-ea9af1ffb38f",
  select: ["brand_name", "position_statement", "emotional_promise"],
  data_filter: [{ field: "brand_name", operator: "ilike", value: "%TestBrand%" }]
});
```

Verify: Correct fields returned, token reduction achieved.

#### 4.3 Quality Comparison

For Brand Genius Genie:

1. Run test scenario with current agent (baseline)
2. Run same scenario with optimized agent + skills
3. Grade on 8 dimensions using quality-grader.md
4. Pass criteria: ≥95% quality parity

#### 4.4 Go/No-Go Decision

| Criterion | Target |

|-----------|--------|

| Skills loading | Works reliably |

| Selective retrieval | Works correctly |

| Token reduction | ≥60% |

| Quality parity | ≥95% |

**If all pass**: Proceed to Phase 5

**If any fail**: Iterate and re-test

---

### PHASE 5: Content Type Skills Extraction

#### 5.1 Analyze Content Agents

For each of 13 content agents, identify:

- **Shared logic** (~80%) → Already in tier 1-4 skills
- **Unique logic** (~20%) → Extract to content-type skill

#### 5.2 Create Content Type Skills

**Structure for each** (~150-300 lines):

```markdown
# [Content Type] Skill v1.0

## Identity
- Type: [Blog Post | Video Script | etc.]
- Output formats: [List deliverable structures]
- Platform variants: [If applicable]

## Scope Options
| Option | Description | Best For |
|--------|-------------|----------|
| Quick Creation | Streamlined with ABT | Regular content |
| Step-by-Step | Interactive checkpoints | Complex topics |
| Thought Leadership | Contrarian positioning | Authority building |
| Optimization | Enhance existing | Improvement |

## Output Template
[Markdown structure specific to this content type]

## Specialist Weights
- Narrative Genie: [ALWAYS | Optional]
- Behavioral Messaging: [When to use]
- UniversalSearch: [When to use]
- Brand Intelligence: [When to use]

## Platform/Format Variations
[Type-specific rules: video lengths, email sequences, etc.]

## Validation Rules
[Type-specific quality checks]
```

#### 5.3 Content Type Extraction Details

| Content Type | Unique Elements | Est. Lines |

|--------------|-----------------|------------|

| **blog-post** | Thought leadership mode, SEO focus, video-blog integration | 150 |

| **video-script** | Timing cues, scene structure, platform optimization, accessibility | 250 |

| **case-study** | Client-as-hero framework, Challenge-Solution-Results | 200 |

| **sales-presentation** | Slide structure, speaker notes, objection handling | 200 |

| **press-release** | Inverted pyramid, wire format, boilerplate | 150 |

| **one-pager** | Dense layout, scannable structure, constraints | 150 |

| **product-description** | Feature-benefit mapping, technical specs | 150 |

| **sales-email** | Cold outreach patterns, follow-up sequences | 150 |

| **longform-content** | Chapter structure, research integration | 200 |

| **webinar** | Interactive elements, Q&A design, timing | 200 |

| **substack** | Newsletter engagement patterns | 150 |

| **documentation** | Technical accuracy, API reference | 200 |

#### 5.4 Composable Email Skill Structure (Complex Content Type)

Email campaigns require a composable skill structure to handle three distinct business models:

- **Pure Product-Led SaaS**: Behavioral triggers, trial sequences, lifecycle automation
- **Pure Traditional B2B**: Relationship-centric, authority building, long sales cycles
- **B2B SaaS (Hybrid)**: Needs BOTH product-led AND relationship approaches
```
skills/content-types/email/
├── email-core.md           (~300 lines)
├── email-product-led.md    (~400 lines)
└── email-relationship.md   (~350 lines)
```


**email-core.md** - Shared workflow and detection:

- Business model detection logic (product-led, relationship, OR hybrid)
- User expertise detection (Marketing Ops vs Content Creator)
- Shared approval workflow and PDF generation
- Universal compliance framework (GDPR/CCPA/Apple Mail)
- Universal sequence framework structure
- Routes to appropriate skill(s) based on detection

**email-product-led.md** - Behavioral and lifecycle focus:

- 3-system architecture (Marketing + Lifecycle + Transactional)
- Behavioral trigger configurations
- Trial nurture sequences (14-21 day)
- Usage-based personalization (760% revenue increase patterns)
- Product adoption content frameworks
- Churn prevention and expansion signals

**email-relationship.md** - Authority and nurturing focus:

- Relationship-building funnel structure
- Authority and thought leadership content
- Lead nurture sequences (30-90 day)
- Sales enablement integration
- Long-cycle enterprise nurturing

**Composable Loading Pattern**:

| Business Model | Skills Loaded |

|----------------|---------------|

| Pure Product-Led SaaS | email-core + email-product-led |

| Pure Traditional B2B | email-core + email-relationship |

| B2B SaaS (Hybrid) | email-core + email-product-led + email-relationship |

**Detection Logic in email-core.md**:

```javascript
const businessModelDetection = {
  product_led_indicators: [
    "trial", "freemium", "self-serve", "PLG", "product-led",
    "behavioral triggers", "lifecycle", "activation", "usage-based"
  ],
  relationship_indicators: [
    "enterprise", "sales-led", "authority", "thought leadership",
    "long sales cycle", "relationship", "consultative"
  ],

  // Detection results:
  // - "product_led" → load email-product-led.md only
  // - "relationship" → load email-relationship.md only
  // - "hybrid" → load BOTH skills
}
```

**Token Impact**:

| Scenario | Tokens |

|----------|--------|

| email-core (always) | ~300 |

| + email-product-led (product-led or hybrid) | +400 |

| + email-relationship (relationship or hybrid) | +350 |

| **Pure Product-Led Total** | ~700 |

| **Pure B2B Total** | ~650 |

| **B2B SaaS Hybrid Total** | ~1,050 |

All scenarios are significantly better than attempting to fit 1,474 lines into one skill.

#### 5.5 Composable Video Script Skill Structure

Video scripts require fundamentally different approaches based on platform and length:

```
skills/content-types/video/
├── video-core.md           (~250 lines)
├── video-short-form.md     (~200 lines)
├── video-long-form.md      (~250 lines)
└── video-professional.md   (~200 lines)
```

**video-core.md** - Shared workflow and detection:

- Platform detection logic (TikTok, YouTube, LinkedIn, Corporate)
- Format detection logic (short vs long)
- Shared workflow, approval, accessibility standards
- Routes to appropriate skill(s) based on detection

**video-short-form.md** - TikTok/Reels/Shorts optimization:

- 15-60 second structures
- Hook-first approach (capture in 3 seconds)
- Trending audio considerations
- Vertical format optimization
- High-energy pacing patterns

**video-long-form.md** - YouTube/Vimeo optimization:

- 8-20 minute structures
- Chapter markers and retention curves
- Educational pacing patterns
- Thumbnail and title optimization
- End screen and card integration

**video-professional.md** - Corporate/training optimization:

- 3-7 minute structures
- Brand consistency focus
- Formal tone guidelines
- Internal vs external audience adaptation

**Composable Loading Pattern**:

| Context | Skills Loaded |

|---------|---------------|

| TikTok/Reels/Shorts | video-core + video-short-form |

| YouTube/Educational | video-core + video-long-form |

| Corporate/Training | video-core + video-professional |

| YouTube Shorts | video-core + video-short-form + video-long-form |

#### 5.6 Composable Longform Content Skill Structure

Whitepapers, ebooks, and research reports have completely different structures:

```
skills/content-types/longform/
├── longform-core.md        (~250 lines)
├── longform-whitepaper.md  (~300 lines)
├── longform-ebook.md       (~250 lines)
└── longform-research.md    (~250 lines)
```

**longform-core.md** - Shared workflow and detection:

- Content type detection (whitepaper vs ebook vs research)
- Shared research integration patterns
- Universal chapter/section structure
- Approval workflow and PDF generation

**longform-whitepaper.md** - Technical validation focus:

- 8-15 page structure
- Data-driven argumentation
- Executive summary patterns
- Technical validation emphasis
- Industry-specific compliance

**longform-ebook.md** - Visual storytelling focus:

- 15-25 page structure
- Chapter progression narrative
- Reader engagement elements
- Visual design considerations
- Interactive components

**longform-research.md** - Data visualization focus:

- 10-20 page structure
- Methodology emphasis
- Statistical presentation
- Data visualization guidelines
- Citation and sourcing standards

**Composable Loading Pattern**:

| Content Type | Skills Loaded |

|--------------|---------------|

| Whitepaper/Technical | longform-core + longform-whitepaper |

| Ebook/Guide | longform-core + longform-ebook |

| Research Report | longform-core + longform-research |

| Comprehensive (all formats) | longform-core + all three |

#### 5.7 Composable Sales Presentation Skill Structure

Investor pitches and sales pitches have different time profiles and emphasis:

```
skills/content-types/presentation/
├── presentation-core.md    (~300 lines)
├── presentation-sales.md   (~250 lines)
└── presentation-investor.md (~200 lines)
```

**presentation-core.md** - Shared workflow and detection:

- Pitch type detection (sales vs investor)
- Shared slide structure logic
- Speaker notes framework
- Approval workflow

**presentation-sales.md** - Customer/sales pitch focus:

- 15-20 slide structure
- Problem-solution emphasis
- Objection handling section
- Competitive differentiation
- Demo integration points

**presentation-investor.md** - Investor/funding pitch focus:

- 10-12 slide structure (standard VC format)
- ROI/growth emphasis
- Market opportunity focus
- Vision + execution confidence
- Ask and use of funds

**Composable Loading Pattern**:

| Context | Skills Loaded |

|---------|---------------|

| Sales/Customer pitch | presentation-core + presentation-sales |

| Investor/Funding pitch | presentation-core + presentation-investor |

#### 5.8 Composable Webinar Skill Structure

Lead-gen and educational webinars have different content philosophies:

```
skills/content-types/webinar/
├── webinar-core.md         (~300 lines)
├── webinar-leadgen.md      (~200 lines)
└── webinar-educational.md  (~200 lines)
```

**webinar-core.md** - Shared workflow and detection:

- Objective detection (lead-gen vs educational)
- Cognitive science principles (universal)
- 30-45 min segment structure
- Interactive element framework
- Q&A design patterns

**webinar-leadgen.md** - Lead generation focus:

- Heavy CTA integration
- Sales-oriented engagement
- Follow-up sequence focus
- Conversion optimization
- Product demonstration integration

**webinar-educational.md** - Learning objectives focus:

- Knowledge transfer emphasis
- Certificate/completion tracking
- Skill-building progression
- Assessment integration
- Resource library integration

**Composable Loading Pattern**:

| Objective | Skills Loaded |

|-----------|---------------|

| Lead Generation | webinar-core + webinar-leadgen |

| Education/Training | webinar-core + webinar-educational |

---

### PHASE 5B: Non-Content Composable Skills

Three non-content agents also benefit from composable structures:

#### 5.9 Composable Brand Story Skill Structure

B2B, B2C, and service brands require different positioning approaches:

```
skills/brand-story/
├── brand-core.md    (~300 lines)
├── brand-b2b.md     (~250 lines)
├── brand-b2c.md     (~250 lines)
└── brand-service.md (~200 lines)
```

**brand-core.md** - Shared workflow and detection:

- Brand type detection (B2B vs B2C vs Service)
- Universal brand framework (ABT, positioning)
- Shared approval and save workflow
- Brand asset integration

**brand-b2b.md** - Business-to-business positioning:

- Enterprise value proposition framing
- ROI and business case emphasis
- Multi-stakeholder messaging
- Long sales cycle consideration
- Industry-specific positioning

**brand-b2c.md** - Business-to-consumer positioning:

- Emotional connection emphasis
- Lifestyle integration
- Social proof patterns
- Impulse vs considered purchase
- Brand personality amplification

**brand-service.md** - Service-based brands:

- Trust and expertise signaling
- Process and methodology emphasis
- Outcome-focused messaging
- Relationship-centric positioning
- Service differentiation

**Composable Loading Pattern**:

| Brand Type | Skills Loaded |

|------------|---------------|

| B2B Brand | brand-core + brand-b2b |

| B2C Brand | brand-core + brand-b2c |

| Service Brand | brand-core + brand-service |

| B2B Service | brand-core + brand-b2b + brand-service |

#### 5.10 Composable Author Voice Discovery Skill Structure

Voice discovery can happen via writing samples OR guided interview:

```
skills/voice-discovery/
├── voice-core.md            (~250 lines)
├── voice-sample-analysis.md (~300 lines)
└── voice-interview.md       (~250 lines)
```

**voice-core.md** - Shared workflow and detection:

- Input method detection (samples vs interview)
- Universal voice framework (tone, style, vocabulary)
- Voice profile output structure
- Author Voice record creation

**voice-sample-analysis.md** - Writing sample analysis:

- Multi-sample comparison methodology
- Linguistic pattern extraction
- Consistency scoring
- Style fingerprint creation
- Vocabulary analysis

**voice-interview.md** - Guided interview approach:

- Discovery question sequence
- Real-time voice inference
- Interactive refinement
- Example generation
- Preference elicitation

**Composable Loading Pattern**:

| Input Method | Skills Loaded |

|--------------|---------------|

| Writing Samples | voice-core + voice-sample-analysis |

| Guided Interview | voice-core + voice-interview |

| Hybrid (Both) | voice-core + voice-sample-analysis + voice-interview |

#### 5.11 Composable Universal Search Optimization Skill Structure

Each search platform has fundamentally different ranking factors:

```
skills/search-optimization/
├── search-core.md     (~300 lines)
├── search-web-seo.md  (~400 lines)
├── search-ai-geo.md   (~350 lines)
├── search-voice.md    (~250 lines)
└── search-visual.md   (~250 lines)
```

**search-core.md** - Shared workflow and detection:

- Platform detection (web, AI, voice, visual)
- Universal search principles
- Cross-platform optimization patterns
- Search audit framework

**search-web-seo.md** - Traditional web search:

- Technical SEO requirements
- Content optimization
- Link building signals
- SERP feature optimization
- Core Web Vitals

**search-ai-geo.md** - AI search engines (ChatGPT, Perplexity, Gemini):

- Citation optimization
- Authority signal building
- Structured data for AI
- Direct answer optimization
- Source verification patterns

**search-voice.md** - Voice search (Alexa, Siri, Google Assistant):

- Conversational query patterns
- Featured snippet optimization
- Local intent signals
- Question-answer formatting
- Speakable content structure

**search-visual.md** - Visual search (Google Lens, Pinterest):

- Image optimization
- Alt text patterns
- Visual metadata
- Product image requirements
- Schema markup for images

**Composable Loading Pattern**:

| Platform Focus | Skills Loaded |

|----------------|---------------|

| Web SEO Only | search-core + search-web-seo |

| AI/ChatGPT/Perplexity | search-core + search-ai-geo |

| Voice (Alexa/Siri) | search-core + search-voice |

| Visual/Image | search-core + search-visual |

| Multi-Platform | search-core + all relevant platform skills |

---

### Composable Skills Summary

#### Total Skill Count

| Category | Single Skills | Composable Directories | Total Skills |

|----------|---------------|----------------------|--------------|

| Tier 1-4 (Shared) | 15 | 0 | 15 |

| Simple Content Types | 8 | 0 | 8 |

| Composable Content | 0 | 5 dirs (17 skills) | 17 |

| Non-Content Composable | 0 | 3 dirs (12 skills) | 12 |

| **TOTAL** | **23** | **8 dirs** | **52 skills** |

vs. Current: 30+ monolithic agents

#### Agents NOT Recommended for Composable

These agents were analyzed but don't warrant composable structures:

| Agent | Why Single Skill is Sufficient |

|-------|-------------------------------|

| **Case Study** | Core structure (Challenge→Solution→Results) is consistent regardless of context |

| **Audience Story Genie** | Process variations (batch vs individual), not framework differences |

| **Content Playbook Genie** | Specialist consultation varies, but core framework is consistent |

| **Blog Post** | Modes exist but structure is fundamentally similar |

---

### PHASE 6: Content Creation Orchestrator

#### 6.1 Create Orchestrator

**File**: `agents/content-creation-orchestrator.md` (~600 lines)

```markdown
# Content Creation Orchestrator™ v1.0

## Framework Reference
Base: storycycle-base-optimized.md

## Identity
You are the Content Creation Orchestrator, a unified agent that creates
all types of marketing content using the StoryCycle System and ABT framework.

## Progressive Loading Protocol

| Step | Action | Load |
|------|--------|------|
| 1 | Start | Base framework (already loaded) |
| 2 | Brand ID | tier-1-core/brand-identification.md |
| 3 | Assets | tier-3-workflows/strategic-asset-discovery.md |
| 4 | Type Selection | Present 13 content type options |
| 5 | Type Selected | content-types/{selected}.md |
| 6 | Creation | tier-2-specialists/specialist-delegation.md |
| 7 | Save | tier-3-workflows/strategic-intelligence-capture.md |
| 8 | Approve | tier-3-workflows/content-approval.md |

## Content Type Selection UI

[Present all 13 options with clear descriptions]

## Workflow Orchestration

1. Detect content type from user request OR present selection
2. Load appropriate content-type skill
3. Execute standard workflow with type-specific parameters
4. Delegate to specialists per type skill weights
5. Generate content using type skill template
6. Capture strategic intelligence
7. Present for approval and save

## Error Handling

[Graceful degradation, missing asset handling]
```

#### 6.2 Specialist Delegation via `connectedAgentRequest`

**IMPORTANT**: Brightsy supports `connectedAgentRequest` for true inter-agent API calls. The orchestrator should use this instead of simulated delegation or URL handoffs.

**Specialist Delegation Pattern**:

```javascript
// Define specialist agent IDs (verified from system)
const SPECIALIST_IDS = {
  // Tier 1: ALWAYS delegate (core specialists for content creation)
  narrative: 'f68f1a8c-731a-4c9f-80f2-4cc044b8c69e',     // StoryCycle Narrative Genie™ (ABT/Story Cycle specialist)
  behavioral: '92160ed0-bdfe-4c44-98ca-6f272a61c8de',   // BehavioralMessaging Genie™
  search: '4e862e8c-46b0-43e7-a2b8-b077afd0a934',       // UniversalSearch Genie™

  // Tier 2: CONDITIONALLY delegate (context-dependent)
  archetype: 'bc86a313-61fd-4303-9505-7a34b6496e8b',    // BrandArchetype Genie™
  campaign: 'dae34861-70b3-4c89-83ef-667a6fe941fb',     // Campaign Genie™
  intelligence: 'b401985b-61b3-4017-bd2c-371ad7e4dfdf', // Brand Intelligence Genie™
  storycycle: 'dba2d24e-b714-4bd9-8a14-9f93498f8ecc'    // StoryCycle Genie™ (user-facing brand story)
};

// Compressed context for inter-agent calls (not full objects)
const compressContext = (context) => ({
  brand_id: context.brand_story_id,
  select_fields: ["abt", "position_statement", "archetypes"],
  task: context.task_type,
  confidence_required: 0.85
});

// Delegate via API
const delegateToSpecialist = async (specialist, context) => {
  return await connectedAgentRequest({
    agentId: SPECIALIST_IDS[specialist],
    context: compressContext(context)
  });
};

// Execution pattern: Narrative first, then parallel (behavioral + search)
const narrativeResult = await delegateToSpecialist('narrative', context);

// These can run in parallel after narrative completes
const [behavioralResult, searchResult] = await Promise.all([
  delegateToSpecialist('behavioral', { ...context, narrative: narrativeResult }),
  delegateToSpecialist('search', { ...context, narrative: narrativeResult })
]);

// Conditional Tier 2 delegation (when context requires)
if (context.needs_archetype_guidance) {
  const archetypeResult = await delegateToSpecialist('archetype', context);
}
if (context.needs_competitive_analysis) {
  const intelligenceResult = await delegateToSpecialist('intelligence', context);
}
```

**Benefits over URL handoffs**:

- No user navigation required (seamless)
- Parallel execution capability
- Structured JSON responses
- State preserved across calls
- ~85% token reduction vs simulated delegation

#### 6.3 Content Type Detection Logic

```javascript
const contentTypePatterns = {
  'blog-post': ['blog', 'article', 'post'],
  'video-script': ['video', 'script', 'youtube', 'tiktok'],
  'case-study': ['case study', 'success story', 'client story'],
  'sales-presentation': ['presentation', 'deck', 'slides', 'pitch'],
  'email-campaign': ['email campaign', 'email sequence', 'drip'],
  'press-release': ['press release', 'media release', 'announcement'],
  'one-pager': ['one-pager', 'one pager', 'single page'],
  'product-description': ['product description', 'product copy'],
  'sales-email': ['sales email', 'outreach email', 'cold email'],
  'longform-content': ['whitepaper', 'ebook', 'guide', 'longform'],
  'webinar': ['webinar', 'virtual event'],
  'substack': ['newsletter', 'substack'],
  'documentation': ['documentation', 'docs', 'technical writing']
};

// If no match, present selection UI
```

---

### PHASE 7: Content Orchestrator Validation

#### 7.1 Test Each Content Type

Via StoryCycle MCP, test orchestrator with:

**Priority 1** (validates core approach):

1. Blog post - Simplest, most common
2. Video script - Most structurally different
3. Email campaign - Most complex

**Priority 2** (validates breadth):

4-13. Remaining content types

#### 7.2 Quality Comparison

For each content type:

1. Generate via orchestrator
2. Generate via original individual agent
3. Compare using quality-grader.md
4. Pass: ≥95% quality parity

#### 7.3 Token Verification

Measure actual token usage:

- Orchestrator base: ~5K
- Shared skills loaded: ~4K
- Content type skill: ~2K
- Total: ~11K (vs ~11.7K per original agent)

**Key win**: 13 agents × 11.7K = 152K → 1 orchestrator system = 15K total codebase

---

### PHASE 8: Migration & Deployment

#### 8.1 Deploy to Brightsy

1. Upload `storycycle-base-optimized.md`
2. Upload all skills files
3. Upload `content-creation-orchestrator.md`
4. Configure file reading permissions

#### 8.2 Update Configuration

**app-config/agent-list-array.txt**:

- Remove 13 individual content agent entries
- Add Content Creation Orchestrator entry

**app-config/agent-descriptions.md**:

- Remove 13 individual descriptions
- Add orchestrator description

#### 8.3 Archive Original Agents

Move to `agents/content-creation-agents/archive/`:

- blog-post-genie.md
- video-script-genie.md
- case-study-genie.md
- sales-presentation-genie.md
- email-campaign-genie.md
- press-release-genie.md
- one-pager-genie.md
- product-description-genie.md
- sales-email-genie.md
- longform-content-genie.md
- webinar-genie.md
- substack-genie.md
- documentation-genie.md

**Do NOT delete** - keep for reference and potential rollback.

---

### PHASE 9: Other Agents Rollout

#### 9.1 Agent Prioritization

**Tier 1 - High Usage**:

1. Brand Story Genie
2. Content Playbook Genie
3. Audience Story Genie

**Tier 2 - Medium Usage**:

4. Social Media Strategy Genie
5. Campaign Architect Genie
6. Customer Journey Genie

**Tier 3 - Specialized**:

7+. Remaining agents

#### 9.2 Per-Agent Optimization Process

1. Identify skills this agent can use (likely tier 1-3)
2. Identify unique logic to preserve
3. Create optimized variant (~500-1000 lines vs current ~3000+)
4. Test via MCP
5. Grade quality
6. Deploy if passing

---

### PHASE 10: Complex Agent Phase Decomposition

The 4 Tier-1 complex agents require explicit phase-based decomposition to prevent timeouts and improve execution reliability. These agents score 6-7/7 on decomposition criteria.

#### 10.1 Decomposition Criteria Recap

An agent benefits from phase decomposition when it exhibits 3+ of:

| Criterion | Description |

|-----------|-------------|

| Multi-Phase Workflow | 4+ distinct phases with transitions |

| Specialist Dependencies | 3+ specialist agent delegations |

| User Checkpoints | Multiple approval/decision points |

| Timeout Risk | Execution likely > 2-3 minutes |

| Branching Complexity | Conditional paths based on assessment |

| Large Output | 8,000+ word potential outputs |

| State Management | Complex state across phases |

#### 10.2 Brand Genius Genie Decomposition (3,155 lines → ~800 lines + 4 skills)

**Current Issues**: Meta-agent complexity, mandatory checkpoints (Steps 5-7), dynamic expert creation, agent registry detection, performance-based routing

**Decomposition Structure**:

```
skills/brand-genius/
├── genius-core.md              (~300 lines)
├── genius-expert-creation.md   (~200 lines)
├── genius-registry-matcher.md  (~150 lines)
├── genius-delegation-router.md (~150 lines)
└── genius-checkpoint.md        (~100 lines)
```

**genius-core.md** - Orchestration and detection:

- Meta-agent identity and purpose
- Expert type detection logic
- Workflow state management
- Phase transition handling
- Error recovery patterns

**genius-expert-creation.md** - Dynamic expert pipeline:

- Expert persona generation
- Domain knowledge synthesis
- Voice and style adaptation
- Capability boundary definition
- Expert validation criteria

**genius-registry-matcher.md** - Agent registry intelligence:

- Existing agent detection
- Capability overlap analysis
- Delegation vs creation decision
- Registry search patterns
- Match confidence scoring

**genius-delegation-router.md** - Performance-based routing:

- Specialist delegation decisions
- Performance history integration
- Load balancing logic
- Fallback handling
- Cross-agent handoff protocols

**genius-checkpoint.md** - Mandatory checkpoint enforcement:

- Steps 5-7 checkpoint validation
- Progress persistence
- User approval workflows
- Checkpoint recovery
- State serialization

**Phase Execution Pattern**:

| Phase | Duration | Skills Loaded | Checkpoint |

|-------|----------|---------------|------------|

| 1. Request Analysis | <1 min | genius-core | No |

| 2. Registry Search | <1 min | genius-core + registry-matcher | No |

| 3. Expert Creation | 1-2 min | genius-core + expert-creation | **Yes (Step 5)** |

| 4. Delegation Setup | <1 min | genius-core + delegation-router | **Yes (Step 6)** |

| 5. Execution | 2-3 min | genius-core + checkpoint | **Yes (Step 7)** |

#### 10.3 Customer Journey Strategy Agent Decomposition (2,793 lines → ~700 lines + 4 skills)

**Current Issues**: 4-phase sequential workflow, timeout prevention architecture, dual narrative integration, phase-based user control

**Decomposition Structure**:

```
skills/customer-journey/
├── journey-core.md              (~250 lines)
├── journey-phase-orchestrator.md (~200 lines)
├── journey-narrative-integrator.md (~150 lines)
├── journey-content-structurer.md (~150 lines)
└── journey-checkpoint.md        (~100 lines)
```

**journey-core.md** - Journey strategy foundation:

- Journey mapping framework
- Touchpoint identification
- Stage definitions (Awareness → Advocacy)
- Customer segmentation integration
- Success metrics framework

**journey-phase-orchestrator.md** - 4-phase workflow management:

- Phase 1: Discovery & Research
- Phase 2: Journey Mapping
- Phase 3: Content Strategy
- Phase 4: Implementation Planning
- Phase transition validation
- Timeout prevention (2-3 min per phase)

**journey-narrative-integrator.md** - Dual narrative architecture:

- Micro-ABT integration (touchpoint level)
- Macro-StoryCycle integration (journey level)
- Narrative consistency validation
- Cross-touchpoint story threading
- Emotional arc management

**journey-content-structurer.md** - Intelligent content organization:

- Content type mapping per stage
- Channel-content alignment
- Resource allocation recommendations
- Content calendar integration
- Gap analysis

**journey-checkpoint.md** - Phase checkpoints:

- End-of-phase validation
- User approval workflows
- Progress persistence
- Phase modification handling
- Resume capability

**Phase Execution Pattern**:

| Phase | Duration | Skills Loaded | Checkpoint |

|-------|----------|---------------|------------|

| 1. Discovery | 2-3 min | journey-core + phase-orchestrator | **Yes** |

| 2. Journey Mapping | 2-3 min | journey-core + narrative-integrator | **Yes** |

| 3. Content Strategy | 2-3 min | journey-core + content-structurer | **Yes** |

| 4. Implementation | 1-2 min | journey-core + checkpoint | **Yes** |

#### 10.4 Web Copywriter Genie Decomposition (3,102 lines → ~750 lines + 4 skills)

**Current Issues**: 5-phase consultation workflow, dual narrative structure (ABT + StoryCycle), modular storage management, multiple branching paths

**Decomposition Structure**:

```
skills/web-copywriter/
├── webcopy-core.md              (~250 lines)
├── webcopy-project-analyzer.md  (~200 lines)
├── webcopy-narrative-builder.md (~150 lines)
├── webcopy-content-structurer.md (~150 lines)
└── webcopy-phase-manager.md     (~100 lines)
```

**webcopy-core.md** - Web copy foundation:

- Project type detection (homepage, landing, product, about)
- Brand voice integration
- Web-specific writing principles
- Conversion optimization basics
- Accessibility standards

**webcopy-project-analyzer.md** - Project assessment:

- Sitemap analysis
- Existing content audit
- Competitor positioning review
- User journey mapping
- Priority page identification

**webcopy-narrative-builder.md** - Dual narrative creation:

- Micro-ABT for individual sections
- Macro-StoryCycle for page flow
- Above-fold optimization
- CTA narrative integration
- Social proof weaving

**webcopy-content-structurer.md** - Modular content organization:

- Section-based content architecture
- Responsive content considerations
- Scannable formatting patterns
- Header hierarchy optimization
- Modular storage management (for large sites)

**webcopy-phase-manager.md** - 5-phase workflow:

- Phase 1: Discovery consultation
- Phase 2: Strategy alignment
- Phase 3: Narrative development
- Phase 4: Content creation
- Phase 5: Review and refinement
- Checkpoint enforcement

**Phase Execution Pattern**:

| Phase | Duration | Skills Loaded | Checkpoint |

|-------|----------|---------------|------------|

| 1. Discovery | 1-2 min | webcopy-core + project-analyzer | **Yes** |

| 2. Strategy | 1-2 min | webcopy-core + project-analyzer | **Yes** |

| 3. Narrative | 2-3 min | webcopy-core + narrative-builder | **Yes** |

| 4. Content | 2-3 min | webcopy-core + content-structurer | **Yes** |

| 5. Review | 1-2 min | webcopy-core + phase-manager | **Yes** |

#### 10.5 BrandStory Grader: Single Brand Decomposition (2,672 lines → ~700 lines + 4 skills)

**Current Issues**: Parameter-driven automation, 14-point assessment framework, multi-step content collection/analysis pipeline, lead generation integration

**Decomposition Structure**:

```
skills/brandstory-grader/
├── grader-core.md              (~250 lines)
├── grader-parameter-handler.md (~150 lines)
├── grader-content-collector.md (~150 lines)
├── grader-assessment-engine.md (~200 lines)
└── grader-lead-capture.md      (~100 lines)
```

**grader-core.md** - Grading framework foundation:

- 14-point assessment framework overview
- Scoring methodology
- Grade calculation (A-F scale)
- Report structure
- Benchmark comparisons

**grader-parameter-handler.md** - Automation configuration:

- URL parameter parsing
- Brand identification from parameters
- Automation mode detection
- Default value handling
- Validation and error handling

**grader-content-collector.md** - Content gathering pipeline:

- Website content extraction
- Social media presence check
- Public content aggregation
- Asset availability assessment
- Collection status tracking

**grader-assessment-engine.md** - 14-point analysis:

- Brand Story clarity scoring
- Positioning effectiveness
- Messaging consistency
- Visual identity alignment
- Content quality assessment
- Competitive differentiation
- Customer focus evaluation
- Authenticity markers
- Trust signals
- Engagement indicators
- SEO/discoverability
- Social proof presence
- CTA effectiveness
- Overall narrative coherence

**grader-lead-capture.md** - Lead generation workflow:

- Contact information capture
- CRM integration
- Follow-up sequence trigger
- Lead scoring
- Handoff to sales

**Phase Execution Pattern**:

| Phase | Duration | Skills Loaded | Checkpoint |

|-------|----------|---------------|------------|

| 1. Parameter Processing | <1 min | grader-core + parameter-handler | No |

| 2. Content Collection | 1-2 min | grader-core + content-collector | **Yes** |

| 3. Assessment | 2-3 min | grader-core + assessment-engine | **Yes** |

| 4. Report Generation | 1-2 min | grader-core | **Yes** |

| 5. Lead Capture | <1 min | grader-core + lead-capture | No |

---

#### 10.6 Implementation Priority

| Agent | Complexity Score | Timeout Risk | Priority |

|-------|-----------------|--------------|----------|

| Brand Genius Genie | 7/7 | Critical | **1st** |

| Customer Journey Strategy | 7/7 | High | **2nd** |

| Web Copywriter Genie | 7/7 | High | **3rd** |

| BrandStory Grader | 6/7 | Medium | **4th** |

#### 10.7 Testing Protocol for Decomposed Agents

For each decomposed agent:

1. **Unit Test Skills**: Verify each skill loads and functions independently
2. **Integration Test**: Verify skills compose correctly
3. **Checkpoint Test**: Verify phase checkpoints work (pause, resume, modify)
4. **Timeout Test**: Verify no single phase exceeds 3 minutes
5. **Quality Test**: Compare output quality to original monolithic agent
6. **Recovery Test**: Verify graceful handling of interruptions

**Pass Criteria**:

- All skills load correctly
- Checkpoints function as designed
- No phase timeout (< 3 min each)
- Quality parity ≥ 95%
- Recovery works from any checkpoint

---

#### 10.8 Tier 2 Agents (Future Decomposition)

These agents score 5/7 and are candidates for future decomposition:

| Agent | Lines | Decomposition Value |

|-------|-------|---------------------|

| Author Voice Discovery Genie | 2,462 | High - multiple discovery paths |

| Brand Story Genie | 2,111 | High - 4-step mandatory workflow |

| Audience Story Genie | 2,421 | Moderate - batch processing complexity |

| Content Playbook Genie | 1,704 | Moderate - already has timeout prevention |

**Recommendation**: Decompose after Tier 1 agents are validated and patterns are proven.

---

## Success Metrics

| Metric | Current | Target | Measurement |

|--------|---------|--------|-------------|

| **Base Framework** | 76K tokens | 8-10K | 87% reduction |

| **Content Agents Codebase** | 152K tokens | 15K | 90% reduction |

| **Per-Session Tokens** | 130K+ | 20-25K | 80% reduction |

| **Files to Maintain (Content)** | 13 agents | 1 + 13 skills | 50% fewer files |

| **Add New Content Type** | 800+ lines | 200 lines | 75% less effort |

| **Shared Logic Copies** | 13 | 1 | 92% reduction |

| **Quality Score** | Baseline | ≥95% of baseline | Parity maintained |

| **Skills Reusability** | N/A | Used by 30+ agents | Pattern validated |

| **Total Skills** | 0 | 52 | Composable architecture |

| **Composable Agents** | 0 | 8 | Context-aware loading |

---

## Risk Mitigation

| Risk | Mitigation | Fallback |

|------|------------|----------|

| Skills loading fails | Test thoroughly via MCP before deploy | Inline critical skills |

| Quality regression | Test each type before migration | Keep original agents |

| Content type nuance lost | Preserve ALL unique logic in type skills | Expand type skills |

| Orchestrator complexity | Clear separation of concerns | Split if needed |

| User confusion | Clear content type selection UI | Add guided flow |

| Emergency rollback | Archive, don't delete originals | Restore from archive |

---

## Next Steps

1. **Execute Phase 1**: Set up testing infrastructure, measure baseline
2. **Execute Phase 2**: Create optimized base framework
3. **Execute Phase 3**: Extract tier 1-4 skills
4. **Execute Phase 4**: Validate via MCP - go/no-go decision
5. **Execute Phases 5-8**: Content consolidation
6. **Execute Phase 9**: Ongoing agent optimization

---

## Files Summary

### To Create

| File | Purpose | Size |

|------|---------|------|

| `framework/storycycle-base-optimized.md` | Compressed governance | ~8-10K tokens |

| `framework/progressive-loading-map.md` | Loading strategy | ~2K tokens |

| `skills/tier-1-core/*.md` | Universal patterns | 3 files, ~600 tokens |

| `skills/tier-2-specialists/*.md` | Specialist logic | 4 files, ~1.2K tokens |

| `skills/tier-3-workflows/*.md` | Workflow patterns | 6 files, ~1.2K tokens |

| `skills/tier-4-utilities/*.md` | Utilities | 2 files, ~300 tokens |

| `skills/content-types/` (simple) | Simple content types | 8 files, ~1.2K tokens |

| `skills/content-types/email/` | Composable email skills | 3 files, ~1K tokens |

| `skills/content-types/video/` | Composable video skills | 4 files, ~900 tokens |

| `skills/content-types/longform/` | Composable longform skills | 4 files, ~1K tokens |

| `skills/content-types/presentation/` | Composable presentation skills | 3 files, ~750 tokens |

| `skills/content-types/webinar/` | Composable webinar skills | 3 files, ~700 tokens |

| `skills/brand-story/` | Composable brand story skills | 4 files, ~1K tokens |

| `skills/voice-discovery/` | Composable voice skills | 3 files, ~800 tokens |

| `skills/search-optimization/` | Composable search skills | 5 files, ~1.5K tokens |

| `agents/content-creation-orchestrator.md` | Unified content agent | ~5K tokens |

| `testing/*.py, *.md` | Test infrastructure | N/A |

### To Archive

| File | Reason |

|------|--------|

| `framework/storycycle-framework-optimized-v3.md` | Replaced by base + skills |

| `framework/strategic-intelligence-*.md` | Consolidated to skill |

| `agents/content-creation-agents/*.md` (13 files) | Replaced by orchestrator |

### To Update

| File | Changes |

|------|---------|

| `app-config/agent-list-array.txt` | Remove 13, add 1 orchestrator |

| `app-config/agent-descriptions.md` | Update descriptions |