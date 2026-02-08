# Architectural Smell Catalog

Patterns that signal structural problems. For each smell: what to search for, why it matters, how to confirm it's real.

---

## Smells by Category

### The Frontend Monolith

**What it is**: Computation that belongs on the server running in the browser instead.

**Signal**: Large loops, heavy math, data transformation, or complex sorting/filtering in `.jsx`, `.tsx`, `.svelte`, `.vue` files.

**Risk**: Performance death on low-powered devices. Impossible to optimize without architectural change.

**Verification**: Profile the page. If JavaScript execution dominates load time, confirm by reading the heavy functions.

**Common in**: Internal tools migrated to web, MVPs that "shipped fast", teams without backend engineers.

---

### The Hidden Schema

**What it is**: Business logic that depends on matching specific string values from a database or API.

**Signal**: `if (type === "Agricultural")` or `switch(status)` inside UI components. String literals in conditionals that look like data values.

**Risk**: Database migration breaks application. Adding new values requires code changes. Logic becomes inconsistent across services.

**Verification**: Trace the string to its source. If it comes from external data, this is a hidden schema.

**Common in**: Applications grown organically, rapid prototypes, teams without schema governance.

---

### The Silent Overflow

**What it is**: Code that assumes collections will remain small.

**Signal**: `array.find()`, `.filter()`, `.map()` inside render loops or high-frequency intervals. Nested iterations without size guards.

**Risk**: Linear (or worse) performance degradation as data grows. Sudden production failures at scale thresholds.

**Verification**: Check if the collection has a maximum size. If not, imagine it with 10,000 items. Does the code still work?

**Common in**: Applications before product-market fit, features built for demos, code written against test fixtures.

---

### The Distributed Monolith

**What it is**: Microservices that must deploy together, share databases, or have synchronous call chains.

**Signal**: Multiple services with shared database schemas. Service A calling B calling C in synchronous chains. Deployment scripts that update multiple services atomically.

**Risk**: All the operational complexity of microservices with none of the independence benefits.

**Verification**: Ask: "Can I deploy service A without touching service B?" If no, they're coupled.

**Common in**: Monoliths "broken up" without domain analysis, teams that adopted microservices for résumé reasons.

---

### The Callback Pyramid

**What it is**: Deep nesting from sequential async operations without proper abstraction.

**Signal**: Indentation depth > 5 levels. `then().then().then()` chains. Callbacks inside callbacks.

**Risk**: Impossible error handling. Race conditions. Debugging requires holding entire chain in memory.

**Verification**: Trace the happy path. Can you explain the sequence in one sentence? If not, it's too deep.

**Common in**: Node.js code from 2014-2017, code written before async/await, rushed features.

---

### The God Object

**What it is**: A class or module that knows too much and does too many things.

**Signal**: Files over 1000 lines. Classes with 20+ methods. Imports from every other module. The word "Manager", "Handler", "Service", or "Utils" in a file doing real work.

**Risk**: All changes risk breaking something. Testing requires mocking the world. New developers can't reason about it.

**Verification**: List what the object _is responsible for_. If the list has "and" more than twice, it's a god object.

**Common in**: Every codebase that survived past its first year.

---

### The Leaky Abstraction

**What it is**: An abstraction that requires understanding its implementation to use correctly.

**Signal**: Comments saying "make sure to call X before Y". Error messages that reference internal state. Documentation listing "gotchas" and "edge cases".

**Risk**: Users of the abstraction make mistakes. Bugs are systematic. Refactoring the implementation breaks callers.

**Verification**: Try to use the API knowing only its signature. If you need to read the source, it leaks.

**Common in**: Internal libraries, "helpful" wrappers around external APIs, anything built under time pressure.

---

### The Time Bomb

**What it is**: Code that will break at a specific future point due to hardcoded assumptions.

**Signal**: Hardcoded years (`2024`, `2025`). Date comparisons with fixed values. Certificate paths. License expiration checks.

**Risk**: Production failure at a predictable moment. Often outside business hours (midnight, year boundary).

**Verification**: Search for four-digit numbers starting with `20`. Audit any date logic.

**Common in**: Certificate management, trial/licensing systems, anything with "temporary" workarounds.

---

### The Orphaned Error

**What it is**: Error handling that swallows exceptions or logs without action.

**Signal**: Empty `catch` blocks. `catch (e) { console.log(e) }` with no rethrow. `try/catch` around entire functions.

**Risk**: Failures become silent. System enters inconsistent state. Debugging requires reproducing production conditions.

**Verification**: Trigger the error deliberately. Observe whether the system's behavior is correct.

**Common in**: Defensive code written after production incidents, code without proper error design.

---

### Drifting Duplicates

**What it is**: The same configuration value (weights, thresholds, feature flags) defined independently in multiple pipeline stages instead of being shared from a single source of truth.

**Signal**: Grep for numeric constants like scoring weights, batch sizes, or threshold values. If the same literal appears in multiple files with the same semantic meaning, it's a drift risk.

**Risk**: One stage gets updated, others don't. Pipeline produces subtly wrong results with no error. Often undetectable until someone audits output quality.

**Verification**: Search for the value across all pipeline stages. If found in 2+ places, check whether they reference a shared config or are independent definitions.

**Common in**: Multi-Lambda pipelines, Airflow DAGs with per-task configs, any pipeline where stages were built independently.

---

### Implicit Inter-Stage Contract

**What it is**: Pipeline stage B assumes stage A's output contains specific fields, types, or structures — but this contract is never validated, documented, or enforced by a schema.

**Signal**: Stage B reads fields from stage A's output using dict access (`data["field"]`) or attribute access without try/except or schema validation. The "schema" exists only in the code that writes and the code that reads.

**Risk**: Any change to stage A's output format silently breaks stage B. Errors surface late (often in production) and far from the root cause.

**Verification**: For each stage handoff, compare the fields written by the producer with the fields read by the consumer. If there's no shared schema definition (Pydantic model, JSON Schema, protobuf, etc.) enforced at the boundary, it's implicit.

**Common in**: Every pipeline that grew organically. Especially acute in serverless pipelines where stages are separate Lambda functions.

---

### Path Convention as Schema

**What it is**: Storage paths (S3 keys, file paths, table names) follow a convention like `issues/{id}/topics/{topic}.json` but this convention is encoded independently in every stage rather than defined centrally.

**Signal**: Grep for path construction patterns (f-strings, string concatenation with path separators). If the same path pattern appears in multiple files, the convention is distributed.

**Risk**: Renaming a path segment requires finding and updating every stage. Missing one produces silent failures (file not found, wrong data read).

**Verification**: Extract all S3 key / file path patterns from each stage. If more than 2 stages construct the same path pattern independently, this smell is confirmed.

**Common in**: S3-based pipelines, data lake architectures, any system where storage paths serve as the integration contract between stages.

---

### Assembly Without Validation

**What it is**: The final stage of a pipeline assembles output from multiple upstream sources (files, API results, stage outputs) without verifying that all expected pieces are present and well-formed.

**Signal**: The publisher/aggregator stage reads N files/keys and assembles them. Look for missing checks: does it verify all N exist? Does it validate their schemas? Does it handle partial results?

**Risk**: Partial output published as complete. Readers of the output (frontends, consumers) see incomplete data with no indication of the gap.

**Verification**: Intentionally remove one upstream output and trace what happens. If the final stage proceeds without error, this smell is confirmed.

**Common in**: Newsletter generators, report builders, dashboard data assemblers — any pipeline with fan-in at the end.

---

### Shared Utility SPOF (Single Point of Failure)

**What it is**: A single utility function in a shared library/layer is called by many or all pipeline stages. A bug in this function breaks the entire pipeline simultaneously.

**Signal**: One function imported and called by 50%+ of stages. Often a wrapper around an external API (database client, AI model invocation, HTTP client).

**Risk**: A parsing bug, API change, or edge case in the shared function causes total pipeline failure rather than isolated stage failure. No graceful degradation.

**Verification**: Identify the most-imported function from the shared layer. Count its callers. If a single code change in this function would break N stages, the blast radius equals N.

**Common in**: Lambda layers, shared pip packages in Airflow, common utility modules.

---

## Smells by Codebase Type

### Frontend Applications (React/Vue/Svelte)

- Frontend Monolith (heavy computation in components)
- Silent Overflow (unbound list rendering)
- God Object (2000-line "container" components)
- Time Bomb (hardcoded feature flag dates)

### Backend Services (Node/Python/Go)

- Hidden Schema (string-matching database values)
- Orphaned Error (swallowed exceptions in handlers)
- Leaky Abstraction (ORMs requiring SQL knowledge)
- Distributed Monolith (shared databases)

### Data Pipelines (Airflow/Spark/dbt/Step Functions)

- Time Bomb (hardcoded date ranges in queries)
- Silent Overflow (unbounded joins)
- Hidden Schema (column names as magic strings)
- Orphaned Error (failed tasks without alerts)
- Drifting Duplicates (same config value defined in multiple stages)
- Implicit Inter-Stage Contract (stages assume data shapes without validation)
- Path Convention as Schema (storage key patterns encoded in every stage)
- Assembly Without Validation (final stage assembles output from many sources without checking completeness)
- Shared Utility SPOF (single utility function used by many/all stages)

### CLI Tools

- God Object (one file doing everything)
- Leaky Abstraction (argument parsing that requires reading source)
- Callback Pyramid (nested shell-outs)

### Mobile Applications

- Frontend Monolith (synchronous processing on main thread)
- Silent Overflow (infinite scroll without virtualization)
- Time Bomb (hardcoded API versions)

### Infrastructure as Code (Terraform/CloudFormation)

- Hidden Schema (hardcoded resource names/IDs)
- Time Bomb (certificate expiry, deprecated resource types)
- Distributed Monolith (blast radius spanning environments)

### Machine Learning Systems

- Hidden Schema (feature engineering assuming specific data distributions)
- Time Bomb (model drift without monitoring)
- Leaky Abstraction (preprocessing that must match training exactly)
- God Object (notebooks with training, evaluation, and deployment)

### Serverless Pipelines (Step Functions/Lambda/EventBridge)

- Drifting Duplicates (config values duplicated across Lambda functions)
- Implicit Inter-Stage Contract (Lambda A writes fields that Lambda B assumes exist)
- Path Convention as Schema (S3 key patterns hardcoded in every function)
- Assembly Without Validation (publisher/aggregator assembles from many S3 keys without checking completeness)
- Shared Utility SPOF (single shared layer function used by all Lambdas)
- Hidden Schema (string-matching on enum-like values across Lambda boundaries)
- Orphaned Error (catch-log-continue in Lambda handlers that should fail the state machine)
- Silent Overflow (distributed map without proper tolerated failure thresholds)
- Timeout Mismatch (Lambda timeout vs. Step Functions task timeout vs. API Gateway timeout)

### Monorepos

- Distributed Monolith (packages with circular dependencies)
- God Object (shared "common" package that everything imports)
- Silent Overflow (build times scaling with repo size)
