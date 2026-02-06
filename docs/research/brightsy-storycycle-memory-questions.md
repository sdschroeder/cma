# Brightsy & StoryCycle Memory Infrastructure: Questions to Answer

**Context:** We're building a strategic memory utilization plan for CMA/StoryCycle Genie V2. The plan depends on understanding what Brightsy's memory infrastructure actually supports. These questions need to be answered by examining the Brightsy and StoryCycle codebases.

**Where answers go:** Answers will update Decision 3 in [strategic-memory-implementation-plan.md](../implementation/strategic-memory-implementation-plan.md).

---

## Brightsy Repo Questions

### 1. Memory Data Model

Look at the database schema (likely Supabase migrations or schema files) for the memory tables.

- **Q1.1:** What columns exist on the `memory_resources` table? Specifically, is there a `level`, `scope`, `type`, or similar column that distinguishes user vs. team vs. agent memories?
- **Q1.2:** What columns exist on `memory_items`? Is there a foreign key to `memory_resources`? Any `level` or `scope` field here?
- **Q1.3:** What columns exist on `memory_item_chunks`? How are they related to `memory_items`?
- **Q1.4:** Is there any RLS (Row Level Security) on these tables? How is memory scoped to accounts/users?

### 2. Memory Levels / Scopes

- **Q2.1:** What distinct memory levels or scopes does the system support? (e.g., user, team, agent, account, global). Look for enums, constants, or type definitions related to memory levels.
- **Q2.2:** Is "agent memory" a supported level? If so, how is it scoped — to a specific agent ID, to the account, or something else?
- **Q2.3:** When a memory is created, how is the level/scope determined? Is it passed as a parameter to the remember tool, or inferred from context?

### 3. RetrievalEngine

Look at `apps/web/lib/server/memory/retrieval-engine.ts` (or wherever the RetrievalEngine lives).

- **Q3.1:** What parameters does the retrieval function accept? Does it filter by memory level/scope?
- **Q3.2:** How does the tiered retrieval work (category → summary → item)? Does level/scope affect which tier is searched?
- **Q3.3:** Can an agent retrieve memories from a specific level (e.g., "give me only team memories" or "give me only agent memories")?

### 4. Remember / Retrieve Tools

Look at the tool handlers for remember and retrieve (likely in `apps/web/lib/server/tools/handlers/` or similar).

- **Q4.1:** What parameters does the `remember` tool accept? Is there a `level`, `scope`, or `type` parameter?
- **Q4.2:** What parameters does the `retrieve` tool accept? Can it filter by level/scope?
- **Q4.3:** When an agent calls `remember`, does it specify whether this is a user memory, team memory, or agent memory? Or is that determined automatically?

### 5. Agent Metadata — memory_level Field

The `agent-metadata` record type has a `configuration` object with a `memory_level` field.

- **Q5.1:** What values does `memory_level` accept? (e.g., "none", "session", "user", "team", "agent", a numeric level, etc.)
- **Q5.2:** What does `memory_level` control? Does it determine what scope of memories the agent can read/write, how much memory context the agent gets, or something else?
- **Q5.3:** Is `memory_level` used in the RetrievalEngine or remember/retrieve tools to filter or scope memory operations?

---

## StoryCycle Repo Questions

### 6. Current Memory Usage

Look at the StoryCycle agent framework files, base framework, and agent configurations.

- **Q6.1:** Do any StoryCycle agents currently call the `remember` tool? If so, which agents and what do they remember?
- **Q6.2:** Do any StoryCycle agents currently call the `retrieve` tool at session start? If so, what do they retrieve?
- **Q6.3:** What `memory_level` values are set on StoryCycle's Agent Metadata records? (Check the agent configurations for Brand Story Genie, Content Creation agents, specialists, etc.)

### 7. Memory Content

- **Q7.1:** Are there any existing memory conventions or naming patterns in use? (e.g., do any memories use prefixes like `cma.*` or any other systematic naming?)
- **Q7.2:** What kinds of information are currently stored in memory? (e.g., user preferences, brand context, workflow state, agent-learned patterns, or something else?)
- **Q7.3:** Is there any distinction between what's stored at the user level vs. team level today?

### 8. Agent-Level Memory in Practice

- **Q8.1:** Do any StoryCycle agents write memories that are scoped to the agent itself (as opposed to the user or team)?
- **Q8.2:** Is there any existing mechanism for agents to learn from past sessions and store those learnings? If so, where does that data live?
- **Q8.3:** Are there any agent framework instructions that reference memory (e.g., "remember the user's preference" or "retrieve previous context")?

---

## Summary: What We Need to Determine

These questions boil down to three things:

1. **Does Brightsy support agent-scoped memory as a distinct level?** (Q1-Q5)
   - If yes: we can store process intelligence (`cma.process.*`) at the agent level for cleaner separation
   - If no: we store process intelligence at the team level (current plan), which works fine

2. **What memory levels are actually available and how do they work?** (Q1-Q5)
   - Specifically: can we write to user level, team level, and (possibly) agent level via the remember tool?
   - Can we retrieve from specific levels?

3. **What's StoryCycle doing with memory today?** (Q6-Q8)
   - Are there existing patterns we should align with or migrate from?
   - Is there anything that conflicts with the `cma.*` naming convention?
   - Are there memories we'd want to preserve vs. replace?

---

## How to Answer

For each question, include:
- The **answer** based on the code
- The **file path(s)** where you found the answer
- Any **relevant code snippets** (keep them short — just the key lines)

If a question can't be answered from the code (e.g., it requires runtime testing), note that.
