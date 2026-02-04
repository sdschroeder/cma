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

Several types use a **`strategic_intelligence`** object (title: "Strategic Intelligence Metadata") with:

- **depth** – `enum`: `brief` \| `standard` \| `deep` \| `comprehensive` ("Collaborative Depth")
- **risks** – array of strings (risk factors)
- **pivots** – array of `{ from, to, why, timestamp? }` (strategic direction changes)
- **decisions** – array of `{ type, what, why, timestamp?, alternatives_rejected? }` (strategic decisions)
- **tradeoffs** – array of `{ factor, resolution }`
- **captured_at** – date-time
- **completeness** – number 0–1 ("Capture Completeness")
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

- **content_sections** – object, "Intelligent Content Sections", "AI-detected content sections (only populated when intelligent_structure.auto_structured = true)".
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

### 2.7 Content Playbook

- **strategic_intelligence** – same shape as above.
- **core_identity** (required) – `brand_name`, `framework_version`.
- **brand_narrative_architecture** – `abt_framework` (and/but/therefore/complete statements), `narrative_integration`, `story_cycle_structure` (act_one, act_two, act_three with heroes, stakes, backstory, disruption, antagonists, mentor, journey, victory, ritual, moral).
- **strategic_content_pillars** – array of `{ pillar_name, description, content_types, content_themes, narrative_structure (abt_framework, story_cycle_elements), related_audience_pain_points }`.
- **channels** – object with channel-specific guidelines: `key_points`, `core_principles`, `example_pattern`, `zero_click_optimization`, `narrative_implementation (abt_adaptation, story_cycle_elements)`.
- **audiences** – array of `{ name, emotional_emphasis, messaging_priorities, narrative_adaptations (abt_adaptation, story_cycle_emphasis) }`.
- **brand_voice** – `abt_framework (and/but/therefore_usage)`, `tone_parameters`, `emotional_triggers`, `language_directives` (prohibited_term → preferred_alternative), `prohibited_elements`, `visual_verbal_alignment`.
- **references** – `brand_story_id`, `audience_story_ids` (links to other records by ID).
- **integrity_checks** – array of `{ type: brand|audience|archetype, description }`.
- **agent_collaboration** – `handoff_protocols`, `conflict_resolution`.
- **measurement_framework** – `success_criteria`, `business_outcomes`, `kpis_by_content_type`, `relationship_development_metrics`.
- **content_approach_preferences** – `default_style` (standard | enhanced_differentiation | full_thought_leadership | per_content_choice), `provocation_level_comfort` (1-4), `evidence_requirement_level`, `competitive_analysis_default`, `content_type_preferences`.
- **relationship_building_framework** – `trust_building_approach`, `audience_journey_mapping`, `emotional_promises_by_touchpoint`.

### 2.8 Author Voice Profile

- **profile_metadata** – `version`, `usage_notes`, `last_updated`, `agent_version`, `creation_date`, `profile_status` (Complete | In Progress | Draft).
- **discovery_pathway** – `pathway_type` (Writing Sample Analysis | Guided Discovery Interview | Hybrid Discovery Process), `discovery_date`, `process_approach` (Step-by-Step | Automated | Hybrid), `samples_provided`, `sample_word_count`.
- **validation_results** – `final_approval`, `accuracy_rating` (1-10), `refinement_notes`, `validation_feedback`, `test_content_generated`.
- **author_context** – `content_types`, `writing_goals`, `target_audience`, `experience_level`, `communication_context`.
- **voice_specification** (comprehensive nested object):
  - `executive_summary`
  - `tonal_architecture`: `perspective_usage` (first/second/third person %), `emotional_expression` (humor_approach, emotional_range, empathy_markers, vulnerability_level), `formality_calibration` (baseline_formality 1-10, context_modulation, formality_indicators)
  - `linguistic_structure`: `sentence_distribution` (avg length, short/medium/long %), `paragraph_patterns` (avg sentences, organization_style, transition_techniques), `punctuation_patterns` (em_dash, questions, parentheticals)
  - `lexical_characteristics`: `vocabulary_level` (complexity_rating, flesch_kincaid_level, specialized_terminology), `word_preferences` (contraction_usage %, adverb_frequency %, adjective_density %, preferred_alternatives)
  - `rhetorical_framework`: `literary_devices` (metaphor_frequency, repetition_patterns, sound_devices), `persuasive_elements` (evidence_types, call_to_action_style)
  - `content_structuring`: `opening_techniques`, `narrative_structure`, `development_patterns`, `conclusion_approaches`
  - `distinctive_elements` (array)
- **contextual_adaptations** – `audience_modulation`, `platform_adaptations`, `flexibility_boundaries`, `content_type_variations`.
- **implementation_guidance** – `evolution_pathway`, `revision_guidelines`, `consistency_checklist`, `troubleshooting_guide`.
- **style_anchors** – array of style reference points.
- Core: **brand_name**, **author_name**.

---

## 3. CMA gap vs "what's there"

| CMA concept | Storycycle today |
|--------------|-------------------|
| **Unified semantic_assets table** | Many record types; content + metadata + **strategic_intelligence** live in `records.data` by convention. No single "intelligence asset" table. |
| **Explicit asset_relationships (graph)** | No graph table. Relations are embedded (e.g. Brand Story ↔ ABT statements, audiences) or could be has-one/has-many to other *records*; no relationship_type/strength. |
| **Strategic coherence scores** | No columns for foundation_alignment_score, cross_asset_intelligence_score, quality_sophistication_score. **Strategic Asset Score** is a separate record type, not a computed score on assets. |
| **Section-level storage** | **Content Asset** has **content_sections** and **intelligent_structure** inside JSONB; no separate `content_sections` table or section-level embeddings in DB. |
| **Memory conventions** | Not visible from schema; would need to check memory names/values for "primary_brand_story_id", etc. |

So: **metadata and strategic intelligence are already in the schemas** (especially `strategic_intelligence` and section/structure fields on Content Asset). The main gaps are: first-class **graph table**, **coherence score columns/views**, and **section-level table + embeddings** if you want full CMA-style section search and reuse.

---

## 4. What the schemas tell us (no extra data needed)

### 4.1 Foundation referenced by record ID?

**All 38 Storycycle record type schemas were scanned** for Brightsy relationship fields: `recordType`, `has-one`, `has-many`, `foreignKey`. **None of the types use them.** So foundation/related assets are **not** linked via has-one/has-many to other record types. They are either:

- **Embedded** – e.g. Brand Story's `abt_statements`, `primary_audiences` (arrays/objects inside the record), or  
- **By name** – e.g. `brand_name` (string), not a record ID.

So we don't need to "check sample data" for relationship IDs: the schema says there are no relation fields to other record types.

### 4.2 How "Strategic Asset Score" works (from schema)

**Strategic Asset Score** is a **separate record type** whose schema defines:

- **asset_id** (string) – "ID of the scored content asset"
- **asset_type** (string)
- **total_score** (0–100), **calculation_date**, **calculation_trigger** (e.g. manual_request, content_update, batch_processing)
- **score_tier** – e.g. "Strategic Foundation Asset", "High Strategic Value", "Good Strategic Alignment", …
- **Component scores** (percentages in parentheses):  
  **brand_story_coherence** (15%), **audience_story_alignment** (15%), **narrative_structure_quality** (10%), **reference_network_strength** (20%), **content_playbook_compliance** (10%), **professional_intelligence_density** (10%), **strategic_context_preservation** (5%), **validation_status_score** (10%), **implementation_readiness** (5%)
- **foundation_analysis**, **intelligence_analysis**, **overall_assessment**, **quality_analysis**, **top_improvement_area**, **improvement_recommendations**
- **algorithm_version** (default 1.0.0)
- Plus the usual **strategic_intelligence** metadata object.

So coherence/scoring is **already modeled as a separate record** that points at an asset by `asset_id` and stores the breakdown. CMA-style "scores on the asset" could be either columns on the asset record or this pattern (separate score record).

### 4.3 What schemas don't tell us

- **Memory usage** – Schema doesn't define memory keys (e.g. `primary_brand_story_id`). That's app/agent convention.
- **Section-level product need** – Whether you want section-level *search* and *reuse* (CMA-style) or are fine with Content Asset's JSONB sections is a product decision, not in schema.
- **Agent flows** – Whether Genie "loads brand story from memory then creates content" is behavior, not schema.

---

## 5. Complete Record Type Schemas (All 38)

All schemas retrieved from Supabase on 2026-02-03.

---

### 5.1 Foundation/Strategy Assets

#### Agent Analysis (`agent-analysis`)
Stores pricing and token usage analysis for agents.
- **agent_id**, **agent_name**, **brand_name**, **primary_function**, **analysis_date** (required)
- **token_usage**: avg_input_tokens, avg_output_tokens, total_tokens_per_run, tokens_per_word, words_per_token
- **api_cost**: total_api_cost, input_token_cost, output_token_cost, api_cost_per_credit
- **customer_pricing**: costs by tier (startup, growth, agency, agency_plus, enterprise, beta_pioneers)
- **margin_analysis**: margins by tier
- **word_count_analysis**: avg_word_count, avg_credits_required, credits_per_1000_words
- **professional_comparison**: hourly_rate_min/max, time_required_hours, cost_savings_percentage, time_savings_percentage
- **model_validation**: value_proposition, margin_maintenance, optimization_opportunities

#### Agent Metadata (`agent-metadata`)
Agent configuration and capabilities registry.
- **agent_id**, **name**, **description**, **status** (enabled/disabled), **brand_name** (required)
- **capabilities**: cms, internet, file_tools, web_search, super_agent (booleans)
- **configuration**: model, temperature, token_limit, memory_level, include_reasoning
- **asset_type_id**: links to asset type the agent works with
- **related_agents**: array of `{ agent_id, agent_name, relationship_type }`
- **performance_metrics**: completion_rate, user_satisfaction, average_response_time
- **workflow_position**, **primary_function**, **user_instruction**, **instruction_summary**
- **strategic_intelligence**

#### Brand Intelligence Report (`brand-intelligence-report`)
Competitive and brand positioning analysis.
- **brand_name**, **analysis_type** (single_brand/competitive_analysis/competitive_landscape/thought_leadership_analysis), **analysis_date**, **intelligence_data**, **quality_metrics** (required)
- **analysis_scope**: methodology, analysis_depth, pages_analyzed, competitors_analyzed
- **intelligence_data**:
  - **brand_positioning**: market_position, value_proposition, competitive_advantages, positioning_vulnerabilities
  - **messaging_strategy**: brand_voice, communication_themes, differentiation_approach
  - **audience_intelligence**: target_segments, audience_language, communication_preferences
  - **competitive_landscape**: market_gaps, competitive_threats, positioning_clusters, differentiation_opportunities
  - **competitive_intelligence**: array of competitor analyses
  - **thought_leadership_analysis**: conventional_wisdom, contrarian_opportunities, unique_perspectives
- **quality_metrics**: positioning_clarity, strategic_actionability, differentiation_strength, audience_confidence, overall_quality_score (1-10 scales)
- **strategic_intelligence**

#### Brand-Native Expert (`brand-native-expert`)
Dynamic expert persona profiles for specialized tasks.
- **brand_name**, **persona_name**, **domain_context**, **expertise_profile**, **communication_style**, **methodology_framework** (required)
- **domain_context**: primary_domain, task_complexity (Simple/Moderate/Complex/Highly Complex), industry_context, secondary_domains, cross_domain_requirements
- **expertise_profile**: expertise_level (Intermediate/Senior/Expert/Thought Leader), specializations, technical_depth, key_competencies, tools_and_methods
- **communication_style**: professional_tone, approach_style, delivery_style, technical_language (Non-Technical → Expert Level)
- **methodology_framework**: problem_solving_method, approach_sequence, quality_standards, validation_process
- **brand_integration**: brand_voice_alignment, archetype_integration, audience_considerations
- **specialist_integrations**: mandatory_specialists, conditional_specialists, delegation_sequence
- **task_focus**: primary_objective, success_criteria, deliverable_format, stakeholder_considerations
- **strategic_intelligence**

#### Brand Story Assessment (`brand-story-assessment`)
Full brand story scoring with 14-point framework.
- **brand_name**, **website_url**, **overall_grade**, **final_percentage** (required)
- **framework_scores** (14 criteria, each 1-5): story_structure, abt_messaging, customer_struggle, brand_role, audience_identification, brand_position_uvp, brand_archetype_tone, consistency_cohesion, transformation_clarity, emotional_resonance, language_simplicity, customer_centricity, brand_differentiation, future_vision
- **narrative_breakdown** (component scores): story_cohesion (0-25), emotional_journey (0-25), authenticity (0-25), memorability (0-15), completeness (0-10)
- **performance_tier**: High Performance (80%+) / Moderate Performance (65-79%) / Low Performance (Below 65%)
- **priority_areas**: priority_1/2/3
- **top_strength**, **biggest_opportunity**, **recommended_action**, **recommended_track**
- **business_model**, **business_model_confidence**, **content_completeness**
- **pdf_url**, **analysis_content** (HTML for email/PDF)
- **strategic_intelligence**

#### Brand Story Grade (`brand-story-grade`)
Simplified brand story scoring.
- **brand_name**, **overall_grade**, **score** (required)
- **strengths**, **weaknesses**, **recommendations** (arrays)
- **detailed_analysis**: 14 criteria objects with `{ grade, notes }`
- **competitive_analysis**: array of `{ competitor_name, grade, score, strengths, weaknesses, differentiation_opportunities }`
- **strategic_intelligence**

#### Campaign (`campaign`)
Campaign planning and tracking.
- **campaign_name**, **brand_name** (required)
- **calendar**: start_date, end_date
- **campaign_theme**: core_message, supporting_messages
- **strategic_context**: primary_objective, target_audience, journey_stage, content_pillar_alignment
- **strategic_intelligence**

#### Content Calendar (`content-calendar`)
Content scheduling and planning.
- **brand_name**, **time_period**, **content_plan** (required)
- **content_plan**: array of `{ date, title, content_type (Blog Post/Social Media Post/Email/Video/Podcast/Webinar/Whitepaper/Case Study/Newsletter/Other), platform, audience, objective, content_pillar, status (Planned/In Progress/Published/Delayed/Cancelled), notes }`
- **key_dates**: array of `{ date, event, content_opportunity }`
- **content_pillars**, **audience_focus** (arrays)
- **content_metrics**: kpis, engagement_goals, conversion_goals
- **content_distribution**: channels, promotion_strategy, cross_promotion
- **strategic_intelligence**

#### Content Enhancement (`content-enhancement`)
Behavioral/search/campaign enhancements to content.
- **content_asset_id**, **enhancement_type** (behavioral/search/campaign), **brand_name** (required)
- **context_analysis**: communication_purpose, brand_elements, audience_factors, channel_considerations
- **selected_principles**: array of `{ principle_name, why_selected, how_it_works, ethical_considerations }`
- **enhancements**: original_content, enhanced_version, psychological_mechanism, evidence_approach
- **ethical_verification**: authenticity_assessment, autonomy_respect, vulnerability_protection, evidence_integrity (booleans)
- **metrics**: before, after, improvement
- **implementation_guidance**: testing_recommendations, contextual_adaptations, measurement_approach
- **strategic_intelligence**

#### Content Marketing Strategy (`content-marketing-strategy`)
High-level content marketing approach.
- **brand_name**, **content_marketing_objectives** (required)
- **content_marketing_objectives**: business_objectives, content_performance_metrics, audience_relationship_metrics, emotional_promise_metrics, physical_gift_metrics
- **editorial_implementation**: brand_voice_implementation, abt_implementation_guides, quality_standards_by_format, review_approval_processes
- **implementation_roadmap**: 30_day_plan, 60_day_plan, 90_day_plan, resource_requirements, risk_mitigation
- **strategic_intelligence**

#### Customer Journey (`customer-journey`)
End-to-end customer journey mapping.
- **brand_name**, **journey_name**, **journey_stages** (required)
- **journey_stages**: array of `{ stage_name, stage_description, customer_mindset, key_questions, barriers, metrics, touchpoints[] }`
- **touchpoints**: `{ channel, content_type, content_purpose, key_messages[], narrative_elements (and/but/therefore_component) }`
- **primary_audience**, **journey_objective**
- **brand_story_alignment**: story_integration, archetype_expression, emotional_progression
- **personalization_strategy**: segmentation_approach, behavioral_triggers, content_variations
- **optimization_insights**: friction_points, drop_off_points, opportunity_areas
- **performance_metrics**: conversion_rate, journey_completion_rate, average_journey_time, cost_per_acquisition, stage_metrics
- **integration_points**: crm_integration, marketing_automation, analytics_platform, data_collection_points
- **strategic_intelligence**

#### Narrative Arc (`narrative-arc`)
Story arc planning (markdown-based).
- **title**, **brand** (required)
- **description**, **key_themes**, **archetypes**
- **core_narrative** (markdown): ABT elements, story positioning, key messages
- **strategic_context** (markdown): audience insights, brand positioning, business objectives
- **behavioral_design** (markdown): behavioral science principles, tactics
- **implementation** (markdown): phases, timeline, touchpoints, channels
- **recommendations** (markdown): optimization suggestions, strategic notes
- **metadata**: version, status (draft/in_review/approved/active/archived), created_for, tags, last_updated

#### Persona (`persona`)
Detailed buyer/audience persona.
- **brand_name**, **persona_name**, **core_identity** (required)
- **core_identity**: role, challenges, success_metrics
- **jtbd_statement** (Jobs To Be Done)
- **buying_insights**: sources, barriers, concerns, criteria
- **content_preferences**: topics, formats, channels
- **strategic_intelligence**

#### Social Media Strategy (`social-media-strategy`)
Comprehensive social media planning.
- **brand_name**, **strategic_foundation**, **platforms**, **content_pillars**, **competitive_positioning** (required)
- **strategic_foundation**: brand_voice_parameters, strategic_differentiation, thought_leadership_level (standard/enhanced_differentiation/full_thought_leadership), language_directives, content_playbook_id
- **platforms**: array of `{ name, strategic_focus, content_approach, primary_audience, posting_frequency, content_mix, engagement_tactics, zero_click_formats, quality_metrics, response_strategy, algorithm_visibility_tactics }`
- **content_pillars**: array of `{ theme, core_message, formats, platforms, content_themes, zero_click_application, platform_adaptations }`
- **competitive_positioning**: primary_differentiation, key_messages, proof_points, competitor_analysis
- **audience_targeting**: primary_audiences, cross_audience_strategy
- **platform_abt_frameworks**: per-platform ABT adaptations
- **cross_platform_integration**: consistency_guidelines, cross_promotion_strategy, content_theme_adaptations
- **thought_leadership_framework**: quality_standards (originality/evidence_strength/provocation_level/implementation_focus targets 1-4), validation_checkpoints, content_enhancement_protocols
- **measurement**: visibility_metrics, engagement_quality_metrics, community_metrics, brand_impact_metrics, business_impact_metrics, thought_leadership_impact, influence_metrics
- **implementation_roadmap**: phases, overall_timeline, resource_requirements
- **strategic_intelligence**

#### Story (`story`)
Reusable brand stories with Story Cycle System mapping.
- **brand_name**, **story_title**, **full_story**, **story_category**, **five_primal_elements** (required)
- **story_category**: Framework Validation / Celebrity/Authority Validation / Personal Vulnerability / Client Success / Framework Demonstration / Teaching/Educational
- **five_primal_elements**: character, timestamp, location_stamp, action, aha, primal_elements_strength (1-5)
- **summary**, **themes**, **quotable_moments**, **best_formats**
- **abt_framework**: and/but/therefore_component, teaching_application
- **story_cycle_mapping**: primary_elements (10 Story Cycle elements), element_demonstrations (heroes, stakes, backstory, disruption, antagonists, mentor, journey, victory, ritual, moral), teaching_value
- **oooh_exercise_examples**: offering_traits, outcomes_traits, organization_traits, oooh_application_notes
- **credibility_score** (1-10), **vulnerability_level** (1-10), **celebrity_factor** (None/Low/Medium/High/Very High)
- **measurable_results**
- **deployment_count**, **deployment_history**: array of `{ date, content_type, content_title, content_id, effectiveness_rating }`
- **strategic_intelligence**

#### Testimonial (`testimonial`)
Customer testimonials with strategic analysis.
- **brand_name**, **testimonial_title**, **original_content**, **customer_name**, **context_details**, **outcome_achieved**, **permission_status** (required)
- **permission_status**: Full permission with attribution / Permission with anonymization / Internal use only / Permission pending / Unknown
- **customer_title**, **customer_company**, **industry_vertical**, **company_profile** (size)
- **story_arc**: challenge, solution, outcome
- **abt_framework**: and/but/therefore_element
- **emotional_journey**: before_state, after_state, transformation_moment
- **primary_emotions**, **pain_point_addressed**, **transformation_type**, **impact_level**
- **quantifiable_results**, **time_to_value**, **credibility_markers**, **credibility_score** (High/Medium/Low)
- **audience_alignment**, **optimal_formats**, **channel_recommendations**, **campaign_integration**, **intended_usage**
- **case_study_potential**, **quality_score** (1-10)
- **testimonial_source**: Email correspondence / Review platform / Interview/call / Survey response / Social media / Case study interview / Other
- **strategic_intelligence**

#### User Content Preferences (`user-content-preferences`)
Tracks user content creation patterns.
- **brand_name**, **content_creation_history** (required)
- **content_creation_history**: array of `{ date, content_type, content_id, approach_chosen, enhancements_added[], user_feedback }`
- **detected_patterns**: preferred_approach, high_use_features, low_use_features, last_pattern_analysis
- **strategic_intelligence**

#### Website Content (`website-content`)
Website structure and content.
- **brand_name**, **site_structure** (required)
- **site_structure**: primary_navigation, secondary_navigation, footer_links
- **page_content**: array of `{ page_name, page_type (Home/About/Product/Service/Blog/Contact/Landing/Other), url_path, sections[] }`
- **sections**: `{ heading, content, call_to_action }`
- **strategic_intelligence**

---

### 5.2 Content Output Assets

#### Social Media Post (`social-media-post`)
Individual social media content.
- **brand_name**, **platform**, **copy** (required)
- **visual**, **hashtags**, **cta**
- **objective**, **content_pillar**, **journey_stage**, **audience_segment**
- **zero_click_category**, **zero_click_tactics**
- **engagement_approach**, **engagement_metrics**, **visibility_metrics**
- **strategic_intelligence**

---

### 5.3 Operational/Infrastructure

#### Email Campaigns (`email-campaigns`)
Email campaign management.
- **brand_name**, **name**, **template_id** (required)
- **status**: draft / scheduled / sending / completed / cancelled
- **subject_line**, **from_name**, **from_email**
- **scheduled_date**, **sent_date**, **recipient_count**, **batch_size**
- **audience_filter** (object for recipient criteria)
- **global_variables** (object for template variables)
- **metrics**: sent, delivered, opened, clicked, bounced, complained, unsubscribed
- **tags**, **description**

#### Email Subscribers (`email-subscribers`)
Email subscriber management.
- **brand_name**, **email**, **first_name** (required)
- **last_name**, **organization**
- **subscription_status**: subscribed / unsubscribed / bounced / complained
- **subscription_date**, **last_email_sent**
- **consent**: consent_date, consent_source, consent_ip
- **email_sequence_position**
- **email_history**: array of `{ email_id, template_id, subject_line, sent_date, opened, opened_date, clicked, clicked_date, sequence_position, is_resend, original_email_id }`
- **engagement_metrics**: opens, clicks, last_opened, last_clicked
- **resend_contact_id**, **resend_audience_id**

#### Email Templates (`email-templates`)
Email template storage.
- **brand_name**, **email_name**, **subject_line**, **email_body** (required)
- **preheader**, **call_to_action**, **target_audience**
- **sequence_position** (1 = first email), **send_delay_days**

#### Resend Templates (`resend-templates`)
Resend email service templates.
- **brand_name**, **name**, **resend_template_id** (required)
- **subject_line**, **description**, **preview_link**
- **variables** (array of template variable names)
- **sequence_position**, **send_delay_days**
- **usage_count**
- **created_at**, **updated_at**

#### Sending Domains (`sending-domains`)
Email sending domain configuration.
- **brand_name**, **domain**, **resend_domain_id** (required)
- **status**: pending / verified / failed
- **region**, **is_default** (boolean)
- **dns_records**: array of `{ record_type, name, value, status }`
- **verified_at**, **bounce_domain**, **tracking_domain**
- **created_at**

#### Webhook Events (`webhook-events`)
Email event tracking.
- **brand_name**, **event_type**, **email_id**, **recipient** (required)
- **event_type**: email.sent / email.delivered / email.opened / email.clicked / email.bounced / email.complained
- **timestamp**, **processed** (boolean)
- **raw_data** (object)

---

## 6. Schema Summary by Category

| Category | Record Types | Has strategic_intelligence |
|----------|--------------|---------------------------|
| **Foundation Strategy** | Brand Story, Audience Story, ABT Statement, Content Playbook, Author Voice Profile, Brand Intelligence Report, Social Media Strategy, Content Marketing Strategy, Customer Journey, Narrative Arc, Persona, Story | ✓ (all) |
| **Quality/Assessment** | Strategic Asset Score, Brand Story Assessment, Brand Story Grade | ✓ |
| **Content Output** | Content Asset, Social Media Post, Website Content, Testimonial | ✓ |
| **Planning** | Campaign, Content Calendar, Content Enhancement | ✓ |
| **Agent System** | Agent Metadata, Agent Analysis, Brand-Native Expert | ✓ |
| **User Data** | User Content Preferences | ✓ |
| **Email Operations** | Email Campaigns, Email Subscribers, Email Templates, Resend Templates, Sending Domains | — |
| **Infrastructure** | Webhook Events | — |

---

## 7. Key Observations for CMA Implementation

### 7.1 What's already there
- **strategic_intelligence** is ubiquitous across 30+ record types - provides consistent metadata pattern
- **ABT framework** is deeply integrated (and/but/therefore components across multiple schemas)
- **Story Cycle System** is native to Content Playbook and Story schemas
- **Section-aware content** exists in Content Asset (`content_sections`, `intelligent_structure`)
- **Scoring framework** exists via Strategic Asset Score with defined component weights
- **Cross-references** exist but via embedded objects/arrays, not foreign keys

### 7.2 What's missing for full CMA
- **Graph relationships**: No `asset_relationships` table with `relationship_type` and `strength`
- **Section-level embeddings**: Sections are in JSONB, not searchable/reusable independently
- **Coherence scores on assets**: Scores are separate records, not columns on assets
- **Lineage tracking**: No `derived_from_assets` or `recomposed_from_content_id` fields
- **Memory conventions**: Not in schema (app-level)

### 7.3 Selective retrieval opportunities

**Essential fields** (always load):
- brand_name, strategic_intelligence.depth, strategic_intelligence.decisions

**Optional fields** (load when needed):
- Full strategic_intelligence object (for deep context)
- competitive_intelligence_cache (for competitive analysis tasks)
- detailed sub-objects (abt_framework, brand_archetypes, etc.)

**Excludable fields** (rarely needed for content creation):
- deployment_history, deployment_count
- email_history, engagement_metrics (operational)
- pdf_url, analysis_content (derived outputs)
