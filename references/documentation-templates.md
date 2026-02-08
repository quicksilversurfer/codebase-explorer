# Documentation Templates

Templates for the output artifacts produced during codebase exploration. Optimize for the reader's decision-making, not comprehensiveness.

---

## Full Exploration Report

Use this template for a complete exploration. Sections can be omitted if not relevant to the task at hand.

```markdown
## System: [Name]

### Atmosphere
The environment this code needs to be alive.

- **External services**: [APIs, SaaS dependencies, internal microservices]
- **Data stores**: [databases, caches — connection info, expected schemas]
- **File system**: [paths, permissions, expected contents]
- **Environment variables**: [which are required, what happens if missing]
- **Scheduled jobs**: [what runs when, what breaks if it doesn't]

### I/O Boundary Map
Where the system touches the world.

| Entry Point | First External Call | Type | Notes |
|-------------|-------------------|------|-------|
| [route/handler] | [call] | ●/◐ | [note] |

### Substrate Constraints
Hard limits the code must respect.

| Constraint | Value | Respected? | Evidence |
|-----------|-------|------------|----------|
| [e.g., payload size] | [e.g., 10MB] | Yes/No/Unclear | [where found] |

### Vertical Slices Explored

#### Slice 1: [The observable value traced]
- **Entry**: [file:line] — where data enters the system
- **Decision**: [file:line] — where the value is computed or selected
- **Render**: [file:line] — where the value becomes visible
- **Impure calls**: [list any I/O along the path]
- **Risk**: [what could break this path]

#### Slice 2: [...]
[repeat]

### Shared Fate Groups
Components that fail together, change together, or depend on the same resource.

- **[Group name]**: [files] — fail together when [condition]
- **[Group name]**: [files] — must change together when [trigger]

### Implicit Contracts
Magic values and hidden dependencies.

| Value | Location | Depends On | Risk |
|-------|----------|-----------|------|
| [literal] | [file:line] | [external source] | [what breaks] |

### Activity Heat Map
Where development energy is concentrated.

- **Hot zones** (high recent activity): [files/areas]
- **Frozen zones** (unchanged 12+ months): [files/areas]
- **Instability signals**: [files with fix/hotfix/revert clusters]
- **Knowledge silos**: [single-author files]

### Architectural Smells Detected

| Smell | Location | Severity | Evidence |
|-------|----------|----------|----------|
| [name] | [file/area] | High/Medium/Low | [what was observed] |

### Risk Profile
Specific costs of specific changes.

- [Change X] requires [coordination Y]. Estimated teams involved: [N].
- [File/module Z] has [N] dependents and no tests. High-risk modification.
- [Component] is the single point of failure for [what].

### Unknowns
What was not explored and why.

- [Area] — [reason not explored, e.g., "behind auth I couldn't bypass"]
- [Area] — [reason, e.g., "low relevance to current task"]
- [Area] — [reason, e.g., "appears abandoned, verified no imports"]
```

---

## Quick Assessment (Abbreviated)

Use when time is limited or the scope is narrow. Covers orientation only.

```markdown
## Quick Assessment: [Name]

### Shape
- **Language/Framework**: [what]
- **Rough size**: [files, LOC if easy to measure]
- **Entry points**: [list main ones]

### I/O Surface
[List the external boundaries — what this system talks to]

### Constraints
[List any hard limits found in config]

### Fragilities
[List magic values or implicit contracts]

### Where the Team is Working
[Hot zones from git history]

### First Impression
[2-3 sentences on overall health, main risks, where to dig deeper]
```

---

## Vertical Slice Record (Standalone)

Use when tracing a single value through the system.

```markdown
## Trace: [The observable value]

**Start**: [What you observed — "the green marker on the map"]
**Render site**: [file:line — where it becomes visible]
**Decision point**: [file:line — where the value is determined]
**Data source**: [file:line — where it enters the system]

### Path
1. [file:line] → [what happens here]
2. [file:line] → [what happens here]
3. [file:line] → [what happens here]

### Impure Calls Along Path
- [file:line] — [what external system is touched]

### Fan-in Points
- [location] — converges with [unexplored branch from X]

### Risk
[What could break this specific path]
```

---

## Data Lineage Map (Pipeline Archetypes)

Use this template when the codebase archetype is Data Pipeline, Serverless Pipeline, or ML System. This is the **primary artifact** for pipeline codebases — it replaces the render-site trace that frontend slices produce.

```markdown
## Data Lineage: [Pipeline Name]

### Entity Traced
**[Entity name]** — e.g., "one arXiv paper", "one customer order", "one training sample"

### Stage-by-Stage Transformation

#### Stage 1: [Stage Name] ([function/file])
- **Input**: [source — API, file, event, upstream stage]
- **Reads fields**: [list fields consumed]
- **Transforms**: [what computation/enrichment happens]
- **Adds fields**: [list new fields added]
- **Output location**: [S3 key pattern / table / file path]
- **Output format**: [JSON, JSONL, Parquet, etc.]
- **External calls**: [APIs, model inference, databases]

→ *Handoff to Stage 2 via [mechanism: S3 key, Step Functions payload, queue message]*

#### Stage 2: [Stage Name] ([function/file])
[repeat structure]

→ *Handoff to Stage 3...*

[continue for all stages]

### Schema Evolution Summary

| Stage | Key Fields Added | Key Fields Consumed | Format |
|-------|-----------------|-------------------|--------|
| [name] | [fields] | [fields] | [format] |

### Inter-Stage Contracts

| Producer → Consumer | Contract Type | Location | Risk |
|-------------------|--------------|----------|------|
| [Stage A → Stage B] | Explicit (Pydantic) / Implicit (dict access) / Validated (JSON Schema) | [where defined] | [what breaks if violated] |

### Storage Path Registry

| Pattern | Written By | Read By | Convention Source |
|---------|-----------|---------|------------------|
| `issues/{id}/raw.jsonl` | [stage] | [stage] | [hardcoded / config / central def] |

### Fan-in / Fan-out Points

- **Fan-out at [stage]**: [entity] splits into [N parallel branches]. Mechanism: [Map state, parallel tasks, etc.]
- **Fan-in at [stage]**: [N upstream outputs] merge into [single output]. Validation: [present / absent]

### Single Points of Failure

- **[shared function/utility]**: Called by [N] stages. A bug here breaks [scope].
- **[external service]**: Used by [stages]. Failure mode: [what happens].
```

---

## Pipeline Exploration Report (Full)

Use this instead of the standard Full Exploration Report when the archetype is a pipeline.

```markdown
## Pipeline: [Name]

### Archetype
[Serverless Pipeline / Data Pipeline / ML Pipeline] — detected via [signals]

### Atmosphere
- **Orchestrator**: [Step Functions / Airflow / manual]
- **Compute**: [Lambda / ECS / Spark / local]
- **Storage**: [S3 / database / filesystem]
- **External APIs**: [list with which stages call them]
- **Schedule**: [cron / event-driven / manual]
- **AI/ML models**: [which models, which stages use them]

### Pipeline Flow Diagram
[ASCII diagram showing stage → stage → stage with parallel branches]

### Data Lineage Map
[Use the Data Lineage Map template above]

### I/O Boundary Map
[Same as standard template]

### Substrate Constraints
[Same as standard template — but emphasize timeouts, memory, concurrency limits, rate limits]

### Shared Fate Groups
[Group by external dependency: "all stages that call Bedrock", "all stages that read/write S3"]

### Implicit Contracts (PROMOTED — Primary Risk for Pipelines)
[This section is the centerpiece for pipeline codebases]

| Contract | Producer | Consumer | Type | Risk |
|----------|----------|----------|------|------|
| [field/path/format] | [stage] | [stage] | Explicit/Implicit | [what breaks] |

### Architectural Smells
[Prioritize pipeline-specific smells from the catalog: Drifting Duplicates, Implicit Inter-Stage Contract, Path Convention as Schema, Assembly Without Validation, Shared Utility SPOF]

### Unknowns
[Same as standard template]
```

---

## Smell Report (Standalone)

Use when conducting a focused health check.

```markdown
## Health Check: [Name]

### Smells Detected

#### [Smell Name]
- **Where**: [file/area]
- **Signal**: [what was observed]
- **Severity**: High/Medium/Low
- **Verification**: [how it was confirmed — or "static analysis only"]
- **Blast radius**: [what breaks if this smell causes a failure]

[repeat per smell]

### Clean Areas
[Areas examined that showed no significant smells — documenting absence is valuable]

### Recommended Priority
1. [Highest risk smell and why]
2. [Second]
3. [Third]
```

---

## Stakeholder Brief (Non-Technical)

Use when the audience is product managers, engineering managers, due diligence reviewers, or anyone who needs to make decisions about the system without reading code. Translate technical findings into business impact.

```markdown
## System Brief: [Name]

### What This System Does
[2-3 sentences in plain language. What business function does it serve? What would users notice if it stopped working?]

### How It Works (Simplified)
[Describe the high-level flow in terms a PM can follow. Use business language, not code language.]

Example: "Every Sunday, the system scans ~250 new research papers, scores them for relevance, picks the best 18, writes analysis and commentary, and publishes a newsletter to the website."

### What's Easy to Change
[List aspects of the system that are well-isolated and low-risk to modify]
- [e.g., "Adding a new scoring criterion — one file, one function, no downstream impact"]
- [e.g., "Changing the publication schedule — single config value"]

### What's Hard to Change
[List aspects that require coordination, carry risk, or have hidden dependencies]
- [e.g., "Changing how papers are scored — the scoring weights are defined in two separate places. Updating one without the other produces silently wrong results."]
- [e.g., "Adding a new newsletter section — requires changes to 3 functions and the final assembly step, with no automated check that all pieces are present."]

### Key Risks
[Translate architectural smells into business consequences]
- **[Risk name]**: [What could go wrong, in business terms]. Likelihood: [Low/Medium/High]. Impact: [What users/customers would experience].

### Dependencies
[External services, APIs, or teams this system relies on]
- **[Service/Team]**: [What it provides]. If unavailable: [what breaks].

### What We Don't Know Yet
[Areas that weren't fully explored and may need further investigation]
- [e.g., "No test suite exists — we can't verify that changes work without manual testing"]
- [e.g., "The frontend that displays the newsletter wasn't examined — it may have its own issues"]

### Recommendations
1. [Highest priority action and why — in business terms]
2. [Second]
3. [Third]
```
