# ATDD — Acceptance Test Driven Development for Claude Code

A Claude Code plugin that enforces the Acceptance Test Driven Development methodology when building software with Claude.

**Inspired by [Robert C. Martin's](https://en.wikipedia.org/wiki/Robert_C._Martin) (Uncle Bob) acceptance test approach from [empire-2025](https://github.com/unclebob/empire-2025).** The ideas, methodology, and key insights in this plugin come directly from his work and public writings on Spec Driven Design and ATDD.

## What This Plugin Does

When you start building a feature, this plugin ensures you follow the ATDD discipline:

1. **Write specs first** — Human-readable Given/When/Then acceptance tests before any code
2. **Generate a test pipeline** — Claude builds a project-specific parser/generator that understands your codebase internals (not a generic Cucumber-style fixture layer)
3. **Two test streams** — Acceptance tests define WHAT, unit tests define HOW. Both constrain Claude's implementation
4. **Guard against leakage** — Automatically detects when implementation details creep into your specs
5. **Iterate** — "Just enough specs for this sprint." Small steps, always

## Key Principles (from Uncle Bob)

> "The two different streams of tests cause Claude to think much more deeply about the structure of the code. It can't just willy-nilly plop code around and write a unit test for it. It is also constrained by the structure of the acceptance tests."

> "Specs will be co-authored by the humans and the AI, but with final approval, ferociously defended, by the humans."

> "All the principles of software engineering are not really about code per se. They are about how to organize the highly detailed specification of a product."

## Installation

```bash
claude plugin add miklos/atdd
```

## Usage

### Start ATDD workflow for a new feature

```
/atdd Add user authentication with email and password
```

### Audit specs for implementation leakage

```
/spec-check
/spec-check specs/authentication.txt
```

## GWT Spec Format

Specs live in `specs/` at your project root and use a simple, opinionated format:

```
;===============================================================
; User can register with email and password.
;===============================================================
GIVEN no registered users.

WHEN a user registers with email "bob@example.com" and password "secret123".

THEN there is 1 registered user.
THEN the user "bob@example.com" can log in.
```

Rules:
- Semicolon comments with `===` separators mark test boundaries
- GIVEN sets up state, WHEN performs actions, THEN asserts outcomes
- **Natural language only** — no class names, API endpoints, database tables, or framework terms
- Describe external observables, not implementation details

## Components

| Component | Name | Purpose |
|-----------|------|---------|
| Skill | `atdd` | Core workflow: specs → pipeline → red/green → iterate |
| Agent | `spec-guardian` | Catches implementation leakage in GWT statements |
| Agent | `pipeline-builder` | Generates bespoke parser→IR→generator for the project |
| Command | `/atdd` | Start ATDD workflow for a feature |
| Command | `/spec-check` | Audit specs for implementation leakage |
| Hook | PreToolUse | Soft warning when writing code without specs |
| Hook | Stop | Reminder to verify both test streams |

## What This Is NOT

- **Not Cucumber.** The generated pipeline has deep knowledge of your system's internals — it's a "strange hybrid of Cucumber and test fixtures" (Uncle Bob's words). No generic fixture layer.
- **Not a template library.** Claude analyzes your project fresh every time and generates a bespoke pipeline.
- **Not Big Up-Front Design.** Write just enough specs for the current feature. Iterate.

## Attribution

This plugin is an implementation of Robert C. Martin's (Uncle Bob) Acceptance Test Driven Development and Spec Driven Design methodology for Claude Code. The approach, insights, and principles come from:

- [empire-2025](https://github.com/unclebob/empire-2025) — Uncle Bob's project where this approach was developed and refined
- His public writings and tweets on ATDD, SDD, and AI-assisted development

This plugin does not contain any code from empire-2025. It adapts the methodology for use as a Claude Code plugin.

## License

MIT
