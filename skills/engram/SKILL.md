---
name: engram
description: "Orchestrates efficient read and write interactions with the engram unified memory store via MCP tools. Provides deterministic workflows for decomposing observations into atomic memories, composing four-dimensional queries (context, similarity, relationship, time) for maximum retrieval accuracy, managing agent-managed focus context, and linking memories into associative webs at write time. Ensures memories are discoverable, precisely ranked, and structurally connected across sessions."
taxonomy: ["memory", "retrieval", "mcp", "persistence", "context-management"]
keywords: ["memory_store", "memory_query", "memory_link", "MCP", "engram", "context filter", "semantic search", "similarity query", "relationship traversal", "temporal query", "focus", "unified memory", "agent memory", "four-dimensional lookup", "active writes", "passive reads", "memory decomposition", "unidirected links"]
---

# engram Memory Interaction Skill

This skill governs how agents interact with the engram unified memory system through its MCP tools: `memory_store`, `memory_query`, and `memory_link`.

engram provides a single `Memory` type with four retrieval dimensions: **context** (exact key-value metadata), **similarity** (vector semantic search), **relationship** (unidirected graph links), and **time** (recency, ordering, decay). Meaning emerges from retrieval, not declaration. This skill enforces the operational principle of **active writes, passive reads**: decomposition and linking happen at write time; reads are simple, deterministic `memory_query` calls.

## 1. Skill Activation Decision Tree

**Activate this skill if and only if:**

```
User request involves ANY of:
├── Storing observations, facts, decisions, or preferences for later retrieval
├── Recalling information from past sessions or tasks
├── Relating previously stored memories to one another
├── Querying a persistent memory backend (engram)
└── Managing the agent's own operational focus context

DO NOT activate for:
├── General knowledge questions with no persistence need ("What is Go's syntax for interfaces?")
├── Direct filesystem operations (use Read/Write/Edit tools, not engram)
├── Code execution or shell commands (use Bash tool)
└── Requests about non-engram memory systems
```

### 1.1 Autonomous Memory Review

This skill also governs **background sub-agents** or **self-review processes** that periodically inspect conversations or execution traces to determine what deserves persistence. In such cases, the agent operates as its own reviewer:

1. Scan recent conversation turns for salient facts, decisions, constraints, or corrections.
2. Apply the same decomposition, context taxonomy, and linking rules as user-initiated writes.
3. Store atoms using `memory_store` and link them immediately — no deferral.
4. Mark contradictory or superseded prior memories with `conflicts` or `supersedes` links.

## 2. Core Principles

### 2.1 Unified Memory Type
There is one `Memory` type. Do not create separate ontologies (Event, Fact, Note, Goal). Store one atomic observation per memory. Its role emerges from how it is retrieved.

### 2.2 Active Writes, Passive Reads
- **Writes are complex**: Decompose, enrich context, and link at store time. This is where intelligence lives.
- **Reads are simple**: A single `memory_query` with the right dimensions returns ranked results. The agent does not need to understand scoring internals.

### 2.3 Agent-Managed Focus
Focus is **your** operational stance, not the store's. You set it on task/project switches and pass it per-query. Focus multiplies relevance (up to +50%), it does **not** filter results. Non-matching memories remain visible but deprioritized.

## 3. Phase 0: Pre-Task Context Bootstrapping

Before beginning substantive work within a known operational context (project, file, library, or team), **always** execute one or more broad `memory_query` calls to load relevant prior guidance. This prevents redundant decisions, honors existing conventions, and surfaces hidden constraints.

### 3.1 Bootstrapping Query Sequence

Execute these queries in order, stopping when diminishing returns are clear:

1. **Project guidance**: `context_filter: { agent: <you>, project: <current> }`, limit: 10, order: "relevance"
2. **File-specific patterns**: `context_filter: { agent: <you>, file: <current-file> }`, limit: 5
3. **Library/framework conventions**: `similar: "<library-name> patterns guidelines"`, focus: { project: <current> }, limit: 5
4. **Team standards and decisions**: `context_filter: { agent: <you>, topic: "standards" }` OR `similar: "team conventions coding standards"`, limit: 5
5. **Recent high-priority items**: `context_filter: { agent: <you>, project: <current>, priority: "high" }`, after: "7d", order: "recency", limit: 5

Load retrieved guidance into short-term working memory. Update focus after bootstrapping. If no relevant memories exist, proceed with defaults and store your assumptions as new memories for future agents.

### 3.2 When to Bootstrap

- **Mandatory**: At the start of every new task, project, or file.
- **Mandatory**: When switching to a different library, framework, or domain.
- **Recommended**: After any long idle period (>30 minutes) within the same project.
- **Skip**: During rapid, tight-loop iteration on the same exact file and task.

## 4. Phase 1: Active Writes — Memory Decomposition Workflow

When you receive a complex observation, user preference, task requirement, or decision, follow this numbered checklist **in exact order**:

### 4.1 Decomposition Checklist

1. **Identify atomic observations**: Break the input into indivisible facts, constraints, decisions, and action items.
2. **Estimate token count**: Target ≤ 500 tokens per memory. If an observation exceeds this, decompose further. Override this guidance only when semantic cohesion would be destroyed by splitting (e.g., a single coherent code block).
3. **Assign context metadata** (minimum required):
   - `agent` — **REQUIRED** (your identity)
   - `project` — **strongly recommended** (the codebase or initiative)
   - Add at least one of: `file`, `topic`, `type`
   - Optional but valuable: `status`, `priority`, `decision`, `user`, `session`
4. **Store each atom sequentially** using `memory_store`. There is no batch API. Each atom is a separate tool call.
5. **Link immediately** using `memory_link`. Do not defer linking to "later" — context is lost.
6. **Track complex chains with planning tools**: When a decomposition yields >3 atoms or >3 links, create a todo/planning checklist to track progress. Mark each store and link step complete as executed. This prevents losing intermediate IDs or skipping links in long chains.

### 4.2 Relationship Vocabulary (Prescribed)

Use **only** these relationship types. Do not invent arbitrary types.

| Type | Use When |
|------|----------|
| `part_of` | The target memory is a component, section, or subset of the source memory. |
| `depends_on` | The source memory cannot be understood or executed without the target memory. |
| `supersedes` | The source memory replaces, overrides, or invalidates the target memory. |
| `derived_from` | The source memory is a conclusion, summary, or inference drawn from the target memory. |
| `conflicts` | The source memory contradicts, contradicts, or is mutually exclusive with the target memory. |
| `relates_to` | Generic association when no semantic type above applies. Use sparingly. |

### 4.3 Write-Time Validation Schema

Before every `memory_store` call, validate:

- [ ] `content` is non-empty and ≤ 500 tokens (override permitted for semantic cohesion)
- [ ] `context` contains at least 3 key-value pairs
- [ ] `context["agent"]` is set and non-empty
- [ ] At least one of `context["project"]` or `context["topic"]` is set
- [ ] `context` keys use snake_case

Before every `memory_link` call, validate:
- [ ] `from` and `to` are valid memory IDs returned from prior `memory_store` calls
- [ ] `type` is from the prescribed vocabulary above
- [ ] The relationship is semantically meaningful, not arbitrary

## 5. Phase 2: Passive Reads — Four-Dimensional Query Composition

To retrieve memories with maximum accuracy, compose queries across the four dimensions using this deterministic decision tree.

### 5.1 Dimension Selection Decision Tree

```
START memory_query construction
│
├── Dimension 1: Context Filter (fast, exact narrowing)
│   ├── Do I have exact metadata? (e.g., agent=coder, project=engram)
│   │   └── YES → Include context_filter with ALL known pairs (AND semantics)
│   │   └── NO  → Skip context_filter (broaden search)
│   └── ALWAYS prefer context_filter as the first narrowing step
│
├── Dimension 2: Similarity (semantic relevance)
│   ├── Do I need semantic relevance beyond exact metadata match?
│   │   └── YES → Include similar with appropriate threshold:
│   │       ├── Broad exploration: similar_threshold = 0.3
│   │       ├── Standard retrieval: similar_threshold = 0.5
│   │       └── Strict matching: similar_threshold = 0.7
│   │   └── NO  → Skip similar
│   └── NEVER use similar as the sole dimension (no deterministic grounding)
│
├── Dimension 3: Relationship (graph traversal)
│   ├── Am I exploring associations from a known memory?
│   │   └── YES → Include rel_from=<memory_id> + rel_depth:
│   │       ├── Direct connections: rel_depth = 1
│   │       └── Extended exploration: rel_depth = 2
│   │   └── NO  → Skip relationship dimension
│   └── Optionally constrain with rel_type from the prescribed vocabulary
│
├── Dimension 4: Temporal (recency ordering)
│   ├── Does recency significantly affect relevance?
│   │   └── YES → Include after (e.g., "24h", "7d", "30d")
│   │       └── Set order = "recency" to prioritize new memories
│   │   └── NO  → Use default order = "relevance" (implicit temporal decay applies)
│   └── Use before (e.g., "1h") only when excluding very recent memories
│
└── ALWAYS include Focus when operational context is stable
```

### 5.2 Focus Management Protocol

1. **Set focus on task switch**: When you begin a new task, file, or project, construct a Focus map:
   ```json
   {
     "agent": "your-agent-name",
     "project": "current-project",
     "file": "current-file.go",
     "topic": "relevant-topic"
   }
   ```
2. **Pass focus in every query** during stable operational context.
3. **Clear or update focus** when switching tasks, files, or domains.
4. **Understand the multiplier effect**: Focus boosts matching memories by up to +50%. It does **NOT** remove non-matching memories. You still see cross-domain memories; yours float higher.
5. **Minimum viable focus**: `agent` is required. Include `project` whenever known.

### 5.3 Query Parameter Defaults

| Parameter | Default | Override Guidance |
|---|---|---|
| `limit` | 10 | Increase to 50 when exploring. Decrease to 3 when confident. |
| `order` | "relevance" | "recency" for time-sensitive queries. "created" for audit trails. |
| `rel_depth` | 1 | Increase to 2 only when exploring graph neighborhoods. |
| `similar_threshold` | 0.0 (no minimum) | Set to 0.3+ to filter noise. Set to 0.7+ for strict matches. |

## 6. Phase 3: Result Interpretation & Follow-Up Actions

After receiving `memory_query` results, follow this deterministic protocol:

### 6.1 Results Are Non-Empty

1. **Read top 3 results** for relevance. If the top result score is high and has links:
   - Traverse with `rel_from=<top_id>` and `rel_depth=1` to gather related context.
2. **If results partially answer the query**: Formulate a more specific follow-up query (add context_filter, raise similar_threshold).
3. **If new insights emerge**: Store a **derived summary memory** with `derived_from` links to the source memories.

### 6.2 Results Are Empty — Progressive Relaxation Protocol

Execute these steps **in order**, stopping when results are non-empty:

1. **Drop temporal constraints** (remove `after`/`before`)
2. **Increase limit** (10 → 50)
3. **Lower similar_threshold** (0.7 → 0.3, or remove entirely)
4. **Drop to context_filter only** (remove `similar`)
5. **Broaden context_filter** (remove least specific key-value pair)

If all steps yield empty results, the memory likely does not exist. Do not hallucinate. Either:
- Ask the user for clarification, or
- Store the query as a new "information gap" memory for future resolution.

## 7. Tool Call Schemas

### 7.1 memory_store

Store a new atomic memory.

```json
{
  "content": "string (required, ≤500 tokens recommended)",
  "context": {
    "agent": "string (required)",
    "project": "string (strongly recommended)",
    "file": "string (optional)",
    "topic": "string (optional)",
    "type": "string (optional: preference, decision, fact, action_item, gap)",
    "status": "string (optional)",
    "priority": "string (optional: low, medium, high, critical)"
  }
}
```

**Returns:**
```json
{
  "id": "string (assigned by store)",
  "created_at": "string (ISO 8601 timestamp)"
}
```

### 7.2 memory_query

Query across all four dimensions. All fields are optional; omitting a dimension means "do not filter on this dimension."

```json
{
  "context_filter": { "key": "value" },
  "similar": "string (semantic query text)",
  "similar_threshold": 0.5,
  "rel_from": "string (memory ID)",
  "rel_type": "string (from prescribed vocabulary)",
  "rel_depth": 1,
  "after": "string (duration: 24h, 7d, 30d)",
  "before": "string (duration: 1h, 24h)",
  "order": "relevance | recency | created",
  "limit": 10,
  "focus": { "agent": "name", "project": "project-name" }
}
```

**Returns:**
```json
{
  "memories": [
    {
      "id": "string",
      "content": "string",
      "context": { "key": "value" },
      "links": [ { "type": "string", "to": "string" } ],
      "created_at": "string (ISO 8601 timestamp)"
    }
  ]
}
```

### 7.3 memory_link

Create a unidirected relationship between two memories.

```json
{
  "from": "string (source memory ID, required)",
  "to": "string (target memory ID, required)",
  "type": "string (default: relates_to, prefer from vocabulary)"
}
```

**Returns:**
```json
{ "success": true | false }
```

## 8. Error Handling & Circuit Breakers

| Failure Mode | Response |
|--------------|----------|
| `memory_store` fails | Retry once with identical payload. If second failure, log error and continue without storing. |
| `memory_link` fails (missing memory ID) | Do not retry. The target memory may not exist. Query for it, or defer linking. |
| `memory_query` fails | Retry once. If persistent, fall back to broader query (remove constraints). |
| Validation failure (missing `agent` context) | Warn, inject default `agent` context, and proceed with reduced precision. Never crash. |
| Empty results after progressive relaxation | Store an "information gap" memory, or ask the user. Do not hallucinate. |

**Hard limits** (never exceeded):
- Max 2 retries per tool call.
- Never enter an infinite query loop. If results are empty after 3 query variations, stop.

## 9. Few-Shot Examples

### Example A: Complex Observation Decomposition

**User input:** "I prefer dark mode, high contrast, and I need this done by Friday. Also, use the new auth middleware we discussed."

**Step 1: Decompose into atoms:**
1. Preference: dark mode
2. Preference: high contrast
3. Constraint: deadline Friday
4. Decision: use new auth middleware

**Step 2: Store atoms (4 sequential `memory_store` calls):**

```json
// Memory 1: dark mode preference
{ "content": "User prefers dark mode for all UI components", "context": { "agent": "ui-bot", "project": "dashboard", "topic": "preference", "type": "preference" } }
// → Returns ID: pref-1

// Memory 2: high contrast preference
{ "content": "User requires high contrast accessibility settings", "context": { "agent": "ui-bot", "project": "dashboard", "topic": "preference", "type": "preference" } }
// → Returns ID: pref-2

// Memory 3: deadline constraint
{ "content": "Feature delivery deadline is Friday", "context": { "agent": "ui-bot", "project": "dashboard", "topic": "schedule", "type": "constraint", "priority": "high" } }
// → Returns ID: deadline-1

// Memory 4: auth middleware decision
{ "content": "Decision: implement using new auth middleware discussed in prior session", "context": { "agent": "ui-bot", "project": "dashboard", "topic": "auth", "type": "decision" } }
// → Returns ID: decision-1
```

**Step 3: Link atoms (3 `memory_link` calls):**

```json
{ "from": "pref-1", "to": "pref-2", "type": "relates_to" }
{ "from": "decision-1", "to": "deadline-1", "type": "depends_on" }
{ "from": "decision-1", "to": "pref-1", "type": "relates_to" }
```

### Example B: Composite Query for Precision Retrieval

**Goal:** Find recent decisions about the dashboard's auth implementation.

**Query:**

```json
{
  "context_filter": { "agent": "ui-bot", "project": "dashboard", "topic": "auth", "type": "decision" },
  "similar": "auth middleware implementation",
  "after": "7d",
  "order": "recency",
  "limit": 5,
  "focus": { "agent": "ui-bot", "project": "dashboard" }
}
```

### Example C: Progressive Relaxation

**Initial query (too strict, 0 results):**

```json
{ "context_filter": { "agent": "ui-bot", "project": "dashboard", "type": "decision" }, "similar": "auth middleware", "similar_threshold": 0.8, "after": "24h", "limit": 3 }
```

**Step 1: Remove temporal:**

```json
{ "context_filter": { "agent": "ui-bot", "project": "dashboard", "type": "decision" }, "similar": "auth middleware", "similar_threshold": 0.8, "limit": 3 }
```

**Step 2: Lower threshold + increase limit:**

```json
{ "context_filter": { "agent": "ui-bot", "project": "dashboard", "type": "decision" }, "similar": "auth middleware", "similar_threshold": 0.5, "limit": 20 }
```

**Step 3: Drop to context only:**

```json
{ "context_filter": { "agent": "ui-bot", "project": "dashboard" }, "limit": 50 }
```

## 10. Anti-Patterns (Explicitly Forbidden)

| Anti-Pattern | Why It Fails | Correct Alternative |
|--------------|--------------|---------------------|
| Storing without `agent` context | Memory becomes unfindable across sessions | Always include `agent` in context |
| One giant memory dump (>500 tokens) | Loses precision, pollutes similarity space | Decompose into atomic memories |
| Treating `focus` as `context_filter` | Excludes cross-domain memories, loses serendipity | Use `focus` for ranking, `context_filter` for exclusion |
| `similar`-only queries | Slow, noisy, no deterministic grounding | Always pair `similar` with `context_filter` or `focus` |
| Deferring linking to "later" | Context is lost, links are never created | Link immediately after storing related atoms |
| Using only `relates_to` for all links | Loses semantic graph structure, weakens traversal | Use precise types from the prescribed vocabulary |
| Attempting batch store/link | MCP tools are atomic; no batch API exists | Sequential calls, link after IDs are returned |
| Ignoring progressive relaxation on empty results | Assumes memory doesn't exist when it might be slightly mis-tagged | Systematically relax constraints |
| Storing contradictory facts without `conflicts` links | Creates inconsistent memory graph | Use `conflicts` to mark contradictory memories |

## 11. Evaluation Criteria (EDD)

This skill is validated against the following layered metrics:

### Reasoning Layer
- **PlanAdherence**: Does the agent follow Bootstrap → Decomposition → Store → Link → Query in the correct order?
- **FocusManagement**: Does the agent set, maintain, and pass focus appropriately across the session?

### Action Layer
- **ToolCorrectness**: Is `memory_store` used for writes, `memory_query` for reads, and `memory_link` for associations?
- **ArgumentCorrectness**: Are tool arguments syntactically valid? Does every `memory_store` include `agent` context?
- **VocabularyCompliance**: Are relationship types from the prescribed list?

### Execution Layer
- **TaskCompletion**: Does the agent successfully retrieve relevant memories? Does it store decomposed, linkable atoms?
- **StepEfficiency**: Does the agent use `context_filter` before falling back to `similar`? Does it minimize redundant queries?
- **ErrorRecovery**: Does the agent follow the retry-once-then-degrade protocol on failures?

### Negative Controls (Must NOT Trigger This Skill)
- General knowledge questions with no persistence need
- Direct filesystem operations
- Code execution or shell commands
- Non-engram memory system requests
