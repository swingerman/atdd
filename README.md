# ATDD — Acceptance Test Driven Development for Claude Code

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Claude Code Plugin](https://img.shields.io/badge/Claude%20Code-Plugin-blueviolet)](https://github.com/swingerman/atdd)
[![Version](https://img.shields.io/badge/version-0.4.0-green)](https://github.com/swingerman/atdd)

A [Claude Code](https://code.claude.com) plugin that enforces the **Acceptance Test Driven Development** (ATDD) methodology when building software with AI. Write human-readable Given/When/Then specs before code, generate project-specific test pipelines, and maintain the discipline of two-stream testing.

**Inspired by [Robert C. Martin's](https://en.wikipedia.org/wiki/Robert_C._Martin) (Uncle Bob) acceptance test approach from [empire-2025](https://github.com/unclebob/empire-2025).** The ideas, methodology, and key insights in this plugin come directly from his work and public writings on Spec Driven Design (SDD) and ATDD.

## Why ATDD with AI?

When using AI to write code, two problems emerge:

1. **AI writes code without constraints** — without acceptance tests anchoring behavior, AI can "willy-nilly plop code around" (Uncle Bob's words) and write unit tests that pass but don't verify the right behavior.
2. **Implementation details leak into specs** — AI naturally tries to fill Given/When/Then statements with class names, API endpoints, and database tables instead of domain language.

This plugin solves both problems by enforcing the ATDD workflow and guarding against implementation leakage. The result: **two test streams** (acceptance + unit) that constrain AI development, producing better-structured, more reliable code.

> "The two different streams of tests cause Claude to think much more deeply about the structure of the code. It can't just willy-nilly plop code around and write a unit test for it. It is also constrained by the structure of the acceptance tests."
> — Robert C. Martin

## How It Works

```
1. Write Given/When/Then specs (natural language, domain-only)
                    ↓
2. Generate test pipeline (parser → IR → test generator)
   Pipeline has DEEP knowledge of your codebase internals
                    ↓
3. Run acceptance tests → they FAIL (red)
                    ↓
4. Implement with TDD (unit tests + code) until BOTH streams pass
                    ↓
5. Review specs for implementation leakage
                    ↓
6. Mutation testing → verify tests actually catch bugs
                    ↓
7. Iterate — next feature, back to step 1
```

The generated pipeline is NOT like Cucumber. It's what Uncle Bob calls "a strange hybrid of Cucumber and the test fixtures" — the parser/generator has **deep knowledge of your system's internals** and produces complete, runnable tests. No manual fixture code needed.

## Installation

Add the marketplace and install the plugin:

```shell
/plugin marketplace add swingerman/atdd
/plugin install atdd@swingerman-atdd
```

Or test locally by cloning:

```bash
git clone https://github.com/swingerman/atdd.git
claude --plugin-dir ./atdd
```

## Getting Started

### 1. Start the ATDD workflow

```
/atdd:atdd Add user authentication with email and password
```

Claude will guide you through understanding the feature, then help you write acceptance specs.

### 2. Write your first spec

Create a file in `specs/` at your project root:

```
;===============================================================
; User can register with email and password.
;===============================================================
GIVEN no registered users.

WHEN a user registers with email "bob@example.com" and password "secret123".

THEN there is 1 registered user.
THEN the user "bob@example.com" can log in.
```

### 3. Generate the test pipeline

Claude analyzes your codebase and generates a project-specific parser, IR format, and test generator tailored to your language and test framework (pytest, Jest, JUnit, Go testing, RSpec, etc.).

### 4. Red → Green → Refactor

Run the acceptance tests (they should fail), then implement with TDD until both acceptance tests AND unit tests pass.

### 5. Check for leakage

```
/atdd:spec-check
```

The spec-guardian agent reviews your specs for implementation details that shouldn't be there.

## Team-Based ATDD Workflow

For larger features, you can orchestrate an agent team that follows the ATDD workflow. The team lead coordinates specialist agents — each with a clear role and strict instructions.

> **Tip:** The plugin includes an `atdd-team` skill that automates team setup and orchestration. Just say "build [feature] with a team" and the skill handles role creation, phase sequencing, and prompt generation.

### Works With Existing Teams

Already running an agent team? The `atdd-team` skill detects active teams and gives you three options:

- **Extend** — Add ATDD roles (spec-writer, implementer, reviewer) to your existing team. Roles that already exist by name are skipped.
- **Replace** — Shut down the current team and create a fresh ATDD team.
- **New team** — Create a separate ATDD team alongside your existing one.

This means you can spin up a team for any purpose, then later say "add ATDD to my team" or "extend my team with ATDD" and the spec-writer, implementer, and reviewer roles join your existing teammates without disrupting ongoing work.

### Team Roles

| Role | Agent Type | Responsibility |
|------|-----------|----------------|
| **Team Lead** | You (or a `general-purpose` agent) | Orchestrates workflow, reviews specs, approves all work, enforces discipline |
| **Spec Writer** | `general-purpose` | Writes Given/When/Then specs from feature requirements using domain language only |
| **Implementer** | `general-purpose` | Builds code using TDD — unit tests first, then implementation — until both test streams pass |
| **Spec Guardian** | `general-purpose` (read-heavy) | Reviews specs for implementation leakage before and after implementation |

### Setting Up the Team

**Prompt to create the team:**

```
Create a team called "atdd-feature" with the following teammates:

1. "spec-writer" (general-purpose) — Writes Given/When/Then acceptance
   test specs. Has the atdd plugin installed. Must follow the /atdd:atdd
   skill strictly.

2. "implementer" (general-purpose) — Implements features using TDD.
   Writes unit tests first, then code, until both acceptance tests and
   unit tests pass.

3. "reviewer" (general-purpose) — Reviews specs for implementation
   leakage and reviews code for quality. Uses /atdd:spec-check.
```

### Step-by-Step: Team Lead Instructions

#### Phase 1 — Spec Writing

Send to **spec-writer**:

```
We're implementing [feature description].

Follow the ATDD workflow from the atdd plugin. Your job:

1. Read the existing codebase to understand the domain language
   (how does the app refer to users, orders, sessions, etc.?)
2. Write Given/When/Then specs in specs/[feature-name].txt
3. Use ONLY external observables — no class names, no API endpoints,
   no database tables, no framework terms
4. Each spec file starts with a semicolon comment block describing
   the behavior
5. Use periods at the end of each statement
6. Send me the specs for review before proceeding

CRITICAL: If you're unsure whether a term is domain language or
implementation language, ask me. Do NOT guess.

Example of what I expect:

;===============================================================
; User can register with email and password.
;===============================================================
GIVEN no registered users.

WHEN a user registers with email "bob@example.com" and password "secret123".

THEN there is 1 registered user.
THEN the user "bob@example.com" can log in.
```

#### Phase 2 — Spec Review

Send to **reviewer**:

```
Review the specs in specs/[feature-name].txt for implementation leakage.

Flag ANY of these:
- Class names, function names, method names
- Database tables, columns, queries
- API endpoints, HTTP methods, status codes
- Framework terms (controller, service, repository, middleware)
- Internal state or data structures
- File paths or module names

For each violation, show the bad line and propose a rewrite using
domain language only.

Also check:
- Is each spec testing ONE behavior?
- Are the GIVEN/WHEN/THEN statements clear to a non-developer?
- Could these specs work for a different implementation language?

Send me your review.
```

#### Phase 3 — Pipeline Generation

As team lead, do this yourself or instruct:

```
Generate the test pipeline for this project. Analyze:
- Language and test framework in use
- Project structure and existing test patterns
- The specs in specs/[feature-name].txt

Create the 3-stage pipeline:
1. Parser — reads specs/*.txt, outputs IR to acceptance-pipeline/ir/
2. Generator — reads IR, produces runnable tests in generated-acceptance-tests/
3. Runner script — run-acceptance-tests.sh

The generator must have DEEP knowledge of the codebase internals.
This is NOT Cucumber. Generated tests should call directly into the
system — no manual fixture glue.

Run the acceptance tests after generation. They MUST fail (red).
If they pass, either the behavior already exists or the generator
isn't testing the right thing.
```

#### Phase 4 — Implementation

Send to **implementer**:

```
The acceptance specs are in specs/[feature-name].txt.
The test pipeline is set up — run ./run-acceptance-tests.sh to execute.

Implement the feature using TDD:

1. Run acceptance tests first — confirm they FAIL
2. Pick the simplest failing acceptance test
3. Write a unit test for the smallest piece needed
4. Write minimal code to make the unit test pass
5. Refactor
6. Repeat 3-5 until that acceptance test passes
7. Move to the next failing acceptance test
8. Continue until ALL acceptance tests AND unit tests pass

RULES:
- Never modify spec files (specs/*.txt) — they are the contract
- Never modify generated test files — only regenerate via the pipeline
- If a spec seems wrong or ambiguous, STOP and ask me
- Run both test streams before reporting done:
  ./run-acceptance-tests.sh  (acceptance tests)
  [project test command]      (unit tests)
- Send me the results when both streams are green
```

#### Phase 5 — Post-Implementation Review

Send to **reviewer**:

```
Implementation is complete. Do two reviews:

1. SPEC REVIEW: Run /atdd:spec-check on specs/[feature-name].txt
   Check if any implementation details leaked into specs during
   development. Propose cleanups if found.

2. CODE REVIEW: Review the implementation for:
   - Test quality (are unit tests testing the right things?)
   - Code structure (does it match what the specs describe?)
   - Missing edge cases (any specs that should be added?)

Send me both reviews.
```

### Tips for Team Leads

- **Never let implementers modify specs.** Specs are YOUR contract. If an implementer says a spec is wrong, review it yourself before authorizing changes.
- **Run spec-check twice** — once before implementation (catch leakage from spec writing) and once after (catch leakage from implementation pressure).
- **Keep specs portable.** Ask yourself: "Could these specs generate the same feature in a different language?" If not, there's leakage.
- **Scope tightly.** Each team cycle should cover one feature. Don't spec the whole system — spec what you're building now.
- **Verify both streams.** Before accepting the implementer's work, run both the acceptance tests and unit tests yourself.

## Mutation Testing

Mutation testing adds a **third validation layer**: after acceptance tests verify WHAT and unit tests verify HOW, mutation testing verifies that your tests **actually catch bugs**.

It works by introducing deliberate bugs (mutants) into your source code and checking whether your tests detect them. A project with 100% code coverage can still have a 60% mutation score — meaning 40% of bugs would go undetected.

```
Three validation layers:
1. Acceptance tests  → verify WHAT (external behavior)
2. Unit tests        → verify HOW  (internal structure)
3. Mutation testing  → verify REAL? (do tests catch bugs?)
```

### Quick Start

```
/atdd:mutate                    # run mutation testing
/atdd:mutate src/auth/          # target specific module
/atdd:kill-mutants              # write tests to kill survivors
```

### Preferred: Custom Mutation Tool

The plugin's preferred approach is to **build a project-specific mutation tool** — a small, TDD-built module that walks the AST, applies one mutation at a time, runs targeted tests, and reports survivors. This follows the approach Uncle Bob developed for [empire-2025](https://github.com/unclebob/empire-2025/blob/master/docs/plans/2026-02-21-mutation-testing.md). It works for any language with no external dependencies.

### Alternative: Existing Frameworks

When rapid setup matters more than tight integration, use an established framework:

| Language | Framework |
|----------|-----------|
| JavaScript/TypeScript | [Stryker](https://stryker-mutator.io/) |
| Python | [mutmut](https://github.com/boxed/mutmut) |
| Java/JVM | [PIT](https://pitest.org/) |
| C# | [Stryker.NET](https://stryker-mutator.io/) |
| Rust | [cargo-mutants](https://github.com/sourcefrog/cargo-mutants) |
| Go | [go-mutesting](https://github.com/zimmski/go-mutesting) |
| Ruby | [mutant](https://github.com/mbj/mutant) |
| Scala | [Stryker4s](https://stryker-mutator.io/) |

### In the Team Workflow

When using the `atdd-team` skill, mutation testing is **Phase 6** — run after post-implementation review, before declaring the feature done.

## GWT Spec Format

Specs use an opinionated, human-readable Given/When/Then format:

```
;===============================================================
; Description of the behavior being specified.
;===============================================================
GIVEN [precondition in domain language].

WHEN [action the user/system takes].

THEN [observable outcome].
```

### The Golden Rule: External Observables Only

Specs must describe what the system does, not how it does it:

| Bad (implementation leakage) | Good (domain language) |
|------------------------------|----------------------|
| `GIVEN the UserService has an empty userRepository.` | `GIVEN there are no registered users.` |
| `WHEN a POST request is sent to /api/users with JSON body.` | `WHEN a new user registers with email "bob@example.com".` |
| `THEN the database contains 1 row in the users table.` | `THEN there is 1 registered user.` |

> "Specs will be co-authored by the humans and the AI, but with final approval, ferociously defended, by the humans."
> — Robert C. Martin

## Plugin Components

| Component | Name | Purpose |
|-----------|------|---------|
| Skill | `atdd` | Core 7-step ATDD workflow: specs → pipeline → red/green → iterate |
| Skill | `atdd-team` | Orchestrates an agent team for ATDD — handles team setup, role assignment, and phase sequencing |
| Skill | `atdd-mutate` | Mutation testing workflow — setup framework, run mutations, analyze and kill surviving mutants |
| Agent | `spec-guardian` | Catches implementation leakage in Given/When/Then statements |
| Agent | `pipeline-builder` | Generates bespoke parser → IR → test generator for your project |
| Command | `/atdd:atdd` | Start the ATDD workflow for a new feature |
| Command | `/atdd:spec-check` | Audit specs for implementation leakage |
| Command | `/atdd:mutate` | Run mutation testing to verify test quality |
| Command | `/atdd:kill-mutants` | Analyze surviving mutants and write tests to kill them |
| Hook | PreToolUse | Soft warning when writing code without acceptance specs |
| Hook | Stop | Reminder to verify both test streams pass |

## Key Principles

These principles from Uncle Bob's writings are encoded in the plugin:

- **"Just enough specs for this sprint"** — Don't write all specs upfront. Spec the current feature, implement it, iterate. Avoid Big Up-Front Design.
- **"Two test streams constrain development"** — Acceptance tests define WHAT (external behavior), unit tests define HOW (internal structure). Both must pass.
- **"Specs describe only external observables"** — No class names, API endpoints, database tables, or framework terms in specs. Domain language only.
- **"Co-authored by humans and AI, ferociously defended by humans"** — Claude proposes specs, you approve them. Always.
- **"Small steps"** — Whether your sprint is a day, an hour, or a microsecond, the same rules apply.

## What This Is NOT

- **Not Cucumber.** The generated pipeline has deep knowledge of your system's internals. No generic fixture layer that requires manual glue code.
- **Not a template library.** Claude analyzes your project fresh every time and generates a bespoke pipeline for your language, framework, and codebase.
- **Not Big Up-Front Design.** Write just enough specs for the current feature. Iterate.

## Works With Any Language

The pipeline-builder agent analyzes your project and generates the parser/generator in your project's language and test framework. Tested approaches include Python/pytest, TypeScript/Jest, Java/JUnit, Go/testing, Ruby/RSpec, Clojure/Speclj, and more.

## Attribution

This plugin is an implementation of Robert C. Martin's (Uncle Bob) Acceptance Test Driven Development and Spec Driven Design methodology for Claude Code. The approach, insights, and principles come from:

- [empire-2025](https://github.com/unclebob/empire-2025) — Uncle Bob's project where this approach was developed and refined. He has since added his own [Clojure-native mutation testing tool](https://github.com/unclebob/empire-2025/blob/master/docs/plans/2026-02-21-mutation-testing.md) and a [spec structure checker](https://github.com/unclebob/empire-2025/blob/master/docs/plans/2026-02-24-spec-structure-check.md) to the project.
- His public writings and tweets on ATDD, SDD, and AI-assisted development

Uncle Bob's mutation testing tool is built specifically for Clojure/Speclj — it walks form trees with `postwalk` and runs targeted specs. This plugin takes a **language-agnostic approach**, using established mutation frameworks (Stryker, mutmut, PIT, etc.) so it works with any language and test framework.

This plugin does not contain any code from empire-2025. It adapts the methodology for use as a Claude Code plugin.

## Contributing

Contributions are welcome! Please open an issue or PR on [GitHub](https://github.com/swingerman/atdd).

## License

[MIT](LICENSE)

