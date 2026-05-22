name: architecture-boundary-guardian
description: Architecture-boundary guardian skill. Use this skill to help AI Coding identify and avoid high-risk technical debt when creating, extending, or refactoring software projects. It applies to project initialization, feature iteration, external service integration, task-system design, report/export design, cache design, frontend refactoring, backend modularization, and long-term iteration planning. The goal is not to block fast development, but to preserve key boundaries with minimal necessary changes so the project does not drift into structures that are hard to split, test, roll back, or maintain.
license: MIT

## 1. Core Principle

Early implementations can be simple, but they must not be boundaryless.

The core goal of this skill is:

```text
Do not only make the feature work.
Make sure the feature does not damage the project's future ability to evolve.
```

Before performing any coding task, AI Coding must check whether the change could introduce high-risk technical debt such as:

```text
Unbounded single-file growth
Core capabilities mixed with optional enhancements
Task state stored only in memory
Reports or exported results depending on runtime objects
External service failures breaking the main workflow
Configuration read from scattered sources
Cache without invalidation or cleanup strategy
Duplicated field mapping across UI, reports, exports, and APIs
API calls scattered across unrelated modules
Missing architecture guardrail tests
```

If such risks exist, propose a lower-risk implementation approach before coding.

---

## 2. When to Use

Use this skill when the user asks for:

```text
A new project
A new core module
A long-lived feature
A new async task
History or recent-record functionality
Report / export functionality
Cache
External service integration
AI / third-party API integration
New configuration items
Frontend modularization
Backend refactoring
Rule-library or plugin-like capabilities
Recent task / history task support
Report presentation changes
Next-stage iteration planning
```

Also use this skill when the user says something like:

```text
This project will keep evolving
Help me design the architecture
Help me plan the next stage
Help me avoid technical debt
This feature will expand later
This module will keep growing
The code is becoming hard to maintain
main.js is too large
Reports cannot be opened after restart
External API failure affects the main task
Cache cannot be cleared
```

---

## 3. Boundary Scan Before Coding

Before coding, perform an architecture boundary scan.

Check at least these 9 boundary types:

```text
1. Module boundary
2. Task boundary
3. Data boundary
4. Report / export / presentation output boundary
5. External service boundary
6. Configuration boundary
7. Cache boundary
8. Frontend state boundary
9. Test boundary
```

### 3.1 Module Boundary

Check whether the change will keep expanding an already overloaded file or module.

Risk signals:

```text
All frontend logic goes into main.js
All backend logic goes into server.py / app.py
One function handles validation, execution, rendering, persistence, and response
One module handles APIs, business logic, filesystem work, and third-party services
```

Prefer clear ownership for responsibilities such as:

```text
API calls
Task execution
Report or export generation
Configuration resolution
External service calls
Cache behavior
Frontend feature modules
```

Do not force a fixed structure. The goal is to keep responsibilities clear and testable.

### 3.2 Task Boundary

If the feature involves tasks, analysis jobs, processing pipelines, queues, progress, history, or async execution, clarify the task lifecycle.

Answer:

```text
Where is the task created?
Where is task status stored?
Where is the result stored?
Can the result survive process restart?
How is failure reason recorded?
Can the task be deleted?
Can the task be retried or resumed?
```

Avoid making long-lived task state depend only on in-memory objects.

If a simple in-memory implementation is unavoidable for the current iteration, state the limitation and keep a clear boundary so it can be replaced later.

### 3.3 Data Boundary

Separate input data, intermediate data, output data, presentation data, cache data, user configuration, and system configuration.

Do not write user-facing display text directly into raw structured data.

Preferred rule:

```text
Structured data preserves raw values.
Renderers and presentation layers apply user-facing labels, fallbacks, and formatting.
```

Different systems may output web pages, API responses, PDF, Word, Excel, CSV, Markdown, HTML, custom templates, or third-party callback formats. The output format can vary, but the underlying data contract should remain consistent.

### 3.4 Report / Export / Presentation Output Boundary

If the change affects reports, exports, downloads, historical viewing, dashboards, API responses, customer templates, or third-party callbacks, confirm that outputs are modeled as independent resources.

Avoid doing only:

```text
task.report = report
```

Outputs that users can revisit should be:

```text
Savable
Loadable
Downloadable or returnable
Deletable when needed
Openable from history or recent records
```

#### 3.4.1 Do Not Hard-Code Output Formats

A report boundary does not mean only JSON / Markdown / HTML.

Different projects serve different users and may need:

```text
Developer-facing JSON / HTML / Markdown
Internal Excel / CSV / Word documents
Security audit PDF or formal audit templates
Operations dashboards or email bodies
Platform API responses or webhook callbacks
Customer-specific report templates
```

Therefore, AI Coding must not hard-code the output system to a fixed set such as:

```text
to_json()
to_markdown()
to_html()
```

A safer approach is to define:

```text
Audience
Use case
Output format
Structured output contract
Renderer
Sensitive-field policy
```

Recommended model:

```text
One structured result
  ↓
Rendered for different audiences and formats
  ↓
Different target outputs
```

Do not let each output format maintain its own independent business-field mapping.

### 3.5 External Service Boundary

All external services are optional enhancements by default, unless the user explicitly marks them as critical.

External services include:

```text
AI model APIs
Paid lookup APIs
Third-party query APIs
Payment APIs
Email / SMS services
OCR services
Map services
CDNs
Webhooks
Third-party databases
Remote rule sources
```

Each external service must define:

```text
Whether it is enabled by default
Whether it requires network access
Whether it is paid
Whether it requires an API key or secret
Whether caching is allowed
Whether failure blocks the main workflow
Timeout
Retry policy
How failure is shown
Whether results enter outputs
```

Default rule:

```text
External service failure must not fail the core task.
```

Unless the user explicitly says the service is critical, degrade to:

```text
partial
skipped
external_failed
enhancement_failed
```

### 3.6 Configuration Boundary

If the change adds configuration, clarify where the effective value comes from.

Avoid scattered reads from:

```text
Environment variables
JSON files
Frontend state
Database configuration
Default values in multiple modules
Command-line arguments
```

Prefer a single configuration resolution path, even if implemented simply.

All business modules should read resolved effective settings, not interpret configuration sources independently.

### 3.7 Cache Boundary

Cache is a high-risk optimization, not a default capability.

Before adding cache, answer:

```text
Why is cache needed?
What is cached?
How long does it live?
Who can clear it?
How can it be force-refreshed?
How can it be disabled?
Can it affect correctness or freshness?
Can cached data enter reports or exports?
How is a cache hit labeled?
```

Do not add cache without:

```text
Expiration or invalidation strategy
Cleanup path
Disable switch
Force refresh path
Cache-hit visibility
Tests for stale-data behavior
```

If cache can affect correctness, do not enable it by default.

### 3.8 Frontend State Boundary

If the change affects UI state, clarify state ownership.

Avoid:

```text
Many global variables shared across features
Multiple modules writing the same state
DOM state and JavaScript state drifting apart
One event handler modifying unrelated UI regions
API calls scattered directly across UI handlers
```

Prefer:

```text
Feature-owned state
Explicit shared state
Centralized API call handling
Unified toast / dialog / error display
Clear event ownership
```

### 3.9 Test Boundary

High-risk boundaries should have tests or explicit verification checks.

Consider guardrails such as:

```text
External service failure does not fail the main workflow
Reports or outputs can be loaded after restart
Cache does not return stale data unexpectedly
API keys or secrets do not enter reports, exports, logs, or frontend state
Frontend code does not bypass the project API layer
Input directory roots are not analyzed as concrete files unless resolved
Multiple output formats use the same data contract
```

---

## 4. High-Risk Debt Patterns

AI Coding must actively identify these patterns.

If any appear, warn the user and propose a safer approach before coding.

### 4.1 Unbounded Single-File Growth

Risk signals:

```text
main.js grows beyond a maintainable size
server.py handles API, business logic, exports, and external services
One file mixes unrelated business modules
```

Safer approach:

```text
Extract the clearest boundary first
Do not perform a large rewrite unless requested
Extract API calls, utility functions, or isolated feature logic first
Leave complex rendering modules for a later controlled step if necessary
```

### 4.2 Runtime-Memory Dependency

Risk signals:

```text
Tasks disappear after restart
Reports are only reachable through in-memory task objects
History shows a record but cannot open its result
```

Safer approach:

```text
Separate runtime task state from durable result state
Persist or store re-openable outputs
Add a loading path from disk, database, object storage, or the existing project persistence mechanism
```

### 4.3 External Service Breaks the Core Flow

Risk signals:

```text
AI API timeout fails the entire task
Third-party lookup failure prevents report generation
CDN failure makes a report unusable
```

Safer approach:

```text
Treat external services as optional unless declared critical
Limit failure to the corresponding enhancement section
Continue generating core outputs
Show failure reason in the relevant section
```

### 4.4 Cache Without Governance

Risk signals:

```text
Cache cannot be cleared
Fresh data cannot be loaded
Cache state appears in user-facing outputs unexpectedly
Users cannot tell whether data is cached or live
```

Safer approach:

```text
Do not add cache without cleanup, expiration, disable, and force-refresh behavior
Prefer correctness over request reduction
Make paid-API caching explicit to users
```

### 4.5 Scattered Configuration

Risk signals:

```text
Environment variables say one thing
UI settings show another
JSON config says another
Backend effective behavior is different
```

Safer approach:

```text
Use a single effective configuration resolution path
Document priority order
Show the backend-effective configuration in UI when relevant
```

### 4.6 Output Field Duplication

Risk signals:

```text
Frontend maps one field set
HTML report maps another
JSON output maps another
CSV export maps another
Custom template maps another
```

Safer approach:

```text
Define one structured data contract
Render different formats from that contract
Separate raw values from display labels and formatting
```

---

## 5. Required Architecture Rules

These rules apply by default unless the user explicitly asks for a different behavior.

### Rule 1: Core First

Core capability must be independent from optional enhancements.

```text
Core failure → main task may fail
Enhancement failure → main task should not fail; mark partial / skipped / failed enhancement
```

### Rule 2: No Hidden Runtime Dependency

Long-lived results must not depend only on runtime memory objects.

If users can later view, download, delete, or reopen a result, it needs a durable or reloadable boundary.

### Rule 3: No Unbounded Single File

If a file keeps receiving new responsibilities, check whether a boundary should be extracted.

Do not keep adding unrelated logic to an already overloaded file.

### Rule 4: External Services Need Failure Policies

Any external service must define:

```text
enabled / configured state
timeout
cost notice if relevant
retry behavior
failure policy
result contract
```

### Rule 5: No Cache Without Governance

Do not add cache without expiration, cleanup, disable, force-refresh, and stale-data testing.

### Rule 6: One Data Contract, Many Outputs

One business data contract should drive all target outputs, including but not limited to:

```text
Frontend pages
API responses
Structured data
PDF
Word
Excel / CSV
Markdown
HTML
Email body
Audit report
Customer-specific template
Third-party callback format
```

Output formats may differ, but business-field mapping must not drift.

### Rule 7: Configuration Through a Resolution Boundary

Business modules should not independently interpret scattered configuration sources.

Use a clear effective-configuration path.

### Rule 8: Architecture Guardrails Must Be Tested

Critical boundaries should have tests or explicit verification checks.

---

## 6. Implementation Workflow

When executing a development task, AI Coding must follow this workflow:

```text
1. Restate the user goal
2. Identify affected boundaries
3. Mark high-risk technical debt points
4. Propose the smallest safe implementation
5. State what is intentionally out of scope
6. Define the file-change scope
7. Code
8. Add tests or verification checks
9. Summarize whether the risk was eliminated
```

### 6.1 Restate the Goal

Format:

```text
The goal of this change is:
- ...
```

### 6.2 Identify Boundaries

Format:

```text
Affected boundaries:
- Frontend state boundary
- External service boundary
- Output boundary
```

### 6.3 Mark Risks

Format:

```text
Potential high-risk technical debt:
1. ...
2. ...
```

### 6.4 Smallest Safe Implementation

Format:

```text
Smallest safe implementation:
- No large rewrite
- Extract only ...
- Preserve compatibility ...
- Add fallback ...
```

### 6.5 Out of Scope

Format:

```text
This iteration does not:
- ...
```

---

## 7. Stop Conditions

Stop coding and ask for confirmation if the task requires:

```text
Large-scale frontend rewrite
Replacing the task lifecycle model
Deleting historical data tables
Breaking old output formats
Changing API response structures without compatibility
Making external service failure block the main workflow
Adding cache that cannot be cleared or bypassed
Writing API keys or sensitive configuration into reports, exports, logs, or frontend state
Splitting many tightly coupled modules in one pass
```

If the user still wants to proceed, provide:

```text
Risk explanation
Rollback plan
Phased execution plan
Verification checklist
```

---

## 8. Boundary Design Guidelines

This skill does not force a fixed code structure, class name, directory layout, or framework.

Different projects have different stacks, sizes, team habits, and delivery formats. Boundaries should be implemented according to the project context, not mechanically copied from a template.

When designing a boundary, first answer:

```text
Is this responsibility clear?
Will this responsibility keep growing?
Does this responsibility need independent tests?
Does this responsibility need to be independently replaceable?
Does this responsibility need to survive process restart?
Will this responsibility be reused by multiple output formats?
Does this responsibility depend on external services?
Should failure of this responsibility affect the core workflow?
```

Possible implementation forms include, but are not limited to:

```text
Function
Module
Class
Service object
Configuration layer
Adapter
Middleware
Command object
Data contract
Renderer
Plugin
Directory layer
Framework-native mechanism
```

Choose the form that fits the project. Do not treat this skill as a fixed architecture template.

### 8.1 Task Boundary

When a project involves tasks, analysis jobs, queues, progress, history, or async processing, establish a clear task lifecycle boundary.

The boundary should answer:

```text
How is a task created?
How is task status updated?
How is the result associated?
How is failure reason recorded?
How is task history read?
Does the task need deletion, retry, or recovery?
```

Implementation can be an in-memory structure, database table, file record, queue system, framework-native task mechanism, or any existing project pattern.

Do not force a fixed class name or directory structure because of an example in this skill.

### 8.2 Result and Report Boundary

When a project involves reports, exports, downloads, historical viewing, or third-party callbacks, establish a result/output boundary.

The boundary should answer:

```text
Is the result independent from runtime objects?
Can the result be saved or persisted?
Can the result be loaded back?
Can the result be deleted?
Does the result support multiple output formats?
Does the result have a unified data contract?
```

Implementation can be a file, database record, object storage entry, framework resource, export service, or existing project report module.

Do not limit reports to fixed JSON / Markdown / HTML, and do not force a fixed interface name.

### 8.3 External Service Boundary

When a project integrates AI APIs, paid APIs, third-party lookups, CDNs, webhooks, email, SMS, OCR, or remote rule sources, establish an external service boundary.

The boundary should answer:

```text
Is the service enabled by default?
Does it require network access?
Is it paid?
Does it require secrets?
Does failure block the core workflow?
Does it have timeout and retry strategy?
Do service results enter outputs?
```

Implementation can be a function wrapper, service module, framework client, adapter layer, gateway layer, or platform SDK.

The important part is the failure policy and capability boundary, not a fixed structure.

### 8.4 Configuration Boundary

When a project adds configuration, establish a configuration resolution boundary.

The boundary should answer:

```text
What are the configuration sources?
What is the priority order?
Does the frontend show the backend-effective configuration?
Where are defaults defined?
How are sensitive settings protected?
```

Implementation can be config files, environment resolution, database config, framework settings, admin UI, or a unified read function.

Do not let multiple business modules independently read and interpret the same configuration.

### 8.5 Cache Boundary

When a project adds cache, establish cache governance.

The boundary should answer:

```text
Why cache?
What is cached?
How long does it live?
How is it cleared?
How is it force-refreshed?
How is it disabled?
Can it affect correctness?
Can it enter reports or exports?
```

Implementation can be memory cache, file cache, database cache, Redis, browser cache, framework cache, or third-party cache service.

Cache without governance should not be added to the core workflow by default.

### 8.6 Frontend API and State Boundary

When frontend features grow, establish API call and state boundaries.

The boundary should answer:

```text
Are API calls handled consistently?
Are errors displayed consistently?
Is state ownership clear?
Do multiple modules share the same state?
Does an event handler affect unrelated areas?
```

Implementation can be frontend modules, a state management library, framework composables, service files, hooks, stores, or an existing UI architecture.

Do not keep piling all logic into one file just because it is convenient.

---

## 9. Output Format

When the user asks for an implementation plan, iteration plan, or code-change plan, use this format.

```markdown
# Architecture Boundary Guardian Assessment

## 1. User Goal

Briefly describe the requested feature or change.

## 2. Affected Boundaries

- Module boundary:
- Task boundary:
- Data boundary:
- Report / export / presentation output boundary:
- External service boundary:
- Configuration boundary:
- Cache boundary:
- Frontend state boundary:
- Test boundary:

## 3. High-Risk Technical Debt

| Risk | Present? | Explanation | Handling Strategy |
|---|---|---|---|

## 4. Smallest Safe Implementation

Explain how to complete the goal with minimal changes while avoiding high-risk technical debt.

## 5. Explicitly Out of Scope

List high-risk large changes that should not be mixed into this iteration.

## 6. File Change Scope

| File | Purpose |
|---|---|

## 7. Tests and Verification

- Functional verification:
- Architecture boundary verification:
- Regression verification:

## 8. Future Evolution

Explain which long-term improvements should be handled later, not mixed into this iteration.
```

---

## 10. Quick Checklist

Before coding, ask:

```text
Will this change keep expanding a single large file?
Does this state exist only in memory?
Can this result be loaded without the runtime task object?
Can an external service failure break the main workflow?
Does this cache have cleanup and invalidation?
Is configuration read from multiple places?
Does this field need to be synchronized across multiple outputs or UI views?
Is there an architecture guardrail test or verification step?
Can this change be rolled back?
Is this the smallest safe implementation for this iteration?
```

If any answer reveals risk, handle the boundary before coding.

---

## 11. Final Reminder

This skill is not meant to make projects complicated early. It is meant to keep projects bounded early.

```text
Simple is fine, but not chaotic.
Fast is fine, but not boundaryless.
In-memory first is fine, but keep a replaceable boundary.
Local-file output is fine, but it must be loadable.
Vanilla JS is fine, but it must be modular.
External services are fine, but they must be optional, failure-tolerant, and degradable.
Cache is fine, but it must be clearable, expirable, and disableable.
```

Final principle:

```text
Do not wait until the project becomes complex to add architecture.
Define boundaries while the project is still simple.
```
