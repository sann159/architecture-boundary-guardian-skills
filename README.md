# Architecture Boundary Guardian Skill

language：English | [简体中文](README_zh.md)

**Architecture Boundary Guardian Skill** is a reusable AI Coding skill designed to help AI coding tools identify and prevent high-risk technical debt when creating, extending, or refactoring software projects.

Its goal is not to make projects complicated from the beginning. Instead, it helps preserve the minimum necessary architecture boundaries during fast development so that the project does not later become difficult to maintain, test, roll back, or evolve.

In one sentence:

```text
Early implementations can be simple, but they must not be boundaryless.
```

---

## What Problem It Solves

When using AI Coding tools to build projects quickly, the following problems often appear:

```text
The feature works, but the code becomes harder and harder to change
One file keeps growing and eventually contains everything
External API timeout causes the core workflow to fail
Reports can only be read from in-memory task objects and cannot be opened after restart
Cache has no cleanup mechanism, so users cannot load fresh data
Configuration sources are scattered, and the frontend display differs from backend-effective behavior
The same business field is mapped separately in UI, reports, exports, and APIs
Frontend state becomes chaotic, and every AI change risks breaking unrelated features
```

This skill helps AI Coding tools check project boundaries before coding, identify risks that may become long-term technical debt, and propose the smallest safe implementation plan.

---

## Suitable Use Cases

This skill is suitable for:

```text
New project initialization
Long-term project iteration
Adding a new core module
Adding async tasks
Adding recent records / history records
Adding report / export / download capabilities
Integrating third-party APIs / paid APIs / AI APIs
Designing cache strategies
Designing configuration systems
Frontend modularization
Backend modularization or refactoring
Rule-library or plugin-like capability development
Planning the next stage of a project
```

It is especially useful for projects that:

```text
Will continue evolving
Will keep integrating external services
Generate reports or export files
Have task states, history records, or recent analysis records
Have a frontend that is growing from a single file into a complex UI
Use AI agents continuously during development
```

---

## Core Principles

### 1. Identify Boundaries Before Coding

AI Coding should not only ask:

```text
Can this feature be implemented?
```

It must also ask:

```text
Which boundaries does this change touch?
Will this boundary continue growing?
Will this change create structural problems that are hard to fix later?
```

Key boundaries to check:

```text
Module boundary
Task boundary
Data boundary
Report / export / presentation output boundary
External service boundary
Configuration boundary
Cache boundary
Frontend state boundary
Test boundary
```

---

### 2. Core Capabilities First, Enhancements Optional

Core capabilities must remain independent and stable. They should not be dragged down by optional enhancements.

Examples:

```text
Core capabilities:
Local analysis, core computation, core report generation, basic data processing

Enhancements:
AI summaries, third-party API lookup, paid APIs, CDN fallback, remote rule sources
```

Default rule:

```text
External service failure must not fail the core task.
```

When an external service fails, prefer degrading it to:

```text
partial
skipped
enhancement_failed
external_failed
```

---

### 3. Long-Lived Results Must Not Depend Only on Memory

If the system supports:

```text
Task history
Recent records
Report viewing
Report download
Result reopening
Task deletion
```

It must answer:

```text
Where is task state stored?
Where are results stored?
Can results still be viewed after service restart?
Can reports be read back from disk, database, or object storage?
```

Do not depend only on:

```text
task.report
in-memory map
runtime objects
```

---

### 4. One Data Contract, Many Output Formats

Different projects may need different outputs:

```text
Web pages
API responses
JSON
Markdown
HTML
PDF
Word
Excel / CSV
Email body
Audit report
Customer-specific template
Third-party callback format
```

The safer approach is:

```text
Define one structured data contract first
Then render different outputs for different audiences and formats
```

Do not let each output format maintain its own independent business-field mapping.

---

### 5. Cache Is a High-Risk Optimization

Before adding cache, answer:

```text
Why is cache needed?
What is cached?
How long does it live?
Who can clear it?
How can it be force-refreshed?
How can it be disabled?
Can it affect data freshness?
Can cached data enter reports or exports?
```

Default rules:

```text
Do not ship cache without a cleanup path.
Do not ship cache without an invalidation strategy.
Disable correctness-affecting cache by default.
```

---

### 6. Configuration Must Have a Resolution Boundary

A project may have multiple configuration sources:

```text
Environment variables
Configuration files
Database settings
Frontend settings
User preferences
Default values
```

But business code should not read configuration from everywhere independently.

Recommended approach:

```text
Business modules should read resolved effective configuration.
```

---

## How to Use

This repository only provides the core skill:

```text
SKILL.md
```

How you use it depends on your AI Coding tool. Different tools support skills, rules, memory, project instructions, or system prompts in different ways. You can adapt this skill to your own agent environment.

General usage:

```text
1. Put SKILL.md into the skill / rule / instruction directory supported by your AI Coding tool.
2. Or merge the content of SKILL.md into your project-level AI instruction file.
3. Before asking AI Coding to perform complex development tasks, require it to read and apply this skill.
4. Whenever you add core capabilities, external services, reports, cache, configuration, or task state, ask AI Coding to perform a boundary scan first.
```

You can add the following to your task prompt:

```text
Please read and apply architecture-boundary-guardian/SKILL.md first.

Before coding, output:
1. Which boundaries this change affects;
2. What high-risk technical debt may be introduced;
3. The smallest safe implementation plan;
4. What is explicitly out of scope for this iteration;
5. What tests or verification steps are required.

Then start modifying the code.
```

---

## Recommended Execution Workflow

When using this skill, AI Coding should follow this workflow:

```text
1. Restate the user goal
2. Identify affected boundaries
3. Mark high-risk technical debt
4. Propose the smallest safe implementation
5. State what is explicitly out of scope
6. Define the file-change scope
7. Implement the code
8. Add tests or verification checks
9. Summarize whether the risk was eliminated
```

---

## Recommended Output Format

When asking AI Coding to create an implementation or iteration plan, you can require this structure:

```markdown
# Architecture Boundary Guardian Assessment

## 1. User Goal

Briefly describe the requested change.

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

Explain which long-term improvements should be handled later, instead of being mixed into this iteration.
```

---

## What This Skill Is Not For

This skill is not:

```text
A framework template
A project scaffold
A code generator
A frontend UI style guide
A mandatory backend layering standard
A tool-specific adapter for a particular agent
```

It does not force a project to use fixed directories, class names, frameworks, or engineering structures.

It focuses on one thing:

```text
Guarding the architecture boundaries that become very expensive to repair once they collapse during fast AI Coding development.
```

---

## Suggested Repository Structure

If you only publish the core skill, keep the repository simple:

```text
architecture-boundary-guardian/
├── README.md
├── SKILL.md
└── LICENSE
```

Where:

```text
README.md  # Project description
SKILL.md   # Core skill
LICENSE    # Open-source license, recommended to keep
```

You do not need to provide separate adapter files for different agents. Users who can apply this skill usually also have the ability to convert it into the rules, instructions, or project context required by their own tools.

---

## Reference

The organization and concise rule-writing style of this project were inspired by:

```text
multica-ai/andrej-karpathy-skills
```

The referenced aspects mainly include:

```text
How project-level AI Coding rules are organized
Concise, action-oriented rule writing
The practice of turning reusable development principles into a skill
```

This project serves a different purpose. It focuses on:

```text
Architecture boundary protection
High-risk technical debt prevention
Long-term project maintainability
Boundary scanning during AI Coding
```

This project is not an official extension of the upstream project and is not endorsed by its maintainers.

---

## License

This project is released under the MIT License.

If you reuse, modify, or distribute this project, please keep the license notice.
