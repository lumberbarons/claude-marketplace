# critique

A Claude Code plugin providing review skills for code, tests, documentation, and observability. Produces structured findings reports that surface design, coverage, doc-structure, and logging/error-message issues that linters and static analysis miss.

## Overview

Four focused review skills, each operating on a path you specify:

- **review-code** — design issues (single responsibility, abstraction levels, testability, meaningful naming, API design, error handling strategy)
- **review-tests** — test completeness, usefulness, coverage gaps, output validation, isolation, readability
- **review-docs** — enumeration completeness (doc lists diffed against the codebase), README quality (quick start, prerequisites, gotchas), CLAUDE.md context cost (derivable content, file size), broken references, `.claude/rules/` validation
- **review-o11y** — observability: logging consistency, log level appropriateness, log value, missing logs at I/O boundaries, and error-message quality and consistency

Each skill produces a structured findings report with P1/P2/P3 (and P4 for docs) severities, specific file:line locations, explanations, and concrete fixes.

## Installation

Install via the lumber-mart marketplace — see the [root README](../../README.md#usage) for the `/plugin marketplace add` and `/plugin install` commands.

## Skills

| Skill | Description | Model-Invocable |
|-------|-------------|-----------------|
| `/critique:review-code` | Review code for design issues | Yes |
| `/critique:review-tests` | Review tests for quality and coverage gaps | Yes |
| `/critique:review-docs` | Review README and CLAUDE.md files | Yes |
| `/critique:review-o11y` | Review logging, log levels, and error messages | Yes |

### Natural Language Triggers

- **review-code**: "review the code in api/", "check this code for design issues", "audit this module"
- **review-tests**: "review the tests", "check test quality", "audit test coverage"
- **review-docs**: "review the docs for this project", "check the documentation", "validate CLAUDE.md files"
- **review-o11y**: "review the logging", "are our logs any good", "check observability", "audit error messages", "do we log the right things"

### Usage Examples

```
/critique:review-code src/auth/
/critique:review-tests tests/unit/
/critique:review-docs
/critique:review-docs backend/
/critique:review-o11y internal/payments/
```

## Output

Each skill produces a report with:

- Header: `N files reviewed, M issues found (severity breakdown)`
- One finding per issue with H3 header, priority tag, location, explanation, and concrete fix

No tables, no passing rows — only actionable findings.

## Using with beads

Critique is intentionally bead-agnostic. To file findings as beads, install the companion plugin **specbeads** and run `/raise-beads` after a review. It reads the review output from conversation context, deduplicates against existing open/closed beads, and creates bug/task beads with structured descriptions.

```
/critique:review-code src/auth/
/specbeads:raise-beads
```

In environments that don't use beads, the review report is the deliverable — read it directly, or ask Claude to act on specific findings.

## Prerequisites

- `python3` (for `review-docs` structural validation script)

No other tooling required. The review skills assume you already run linters, formatters, security scanners, and type checkers separately — critique focuses on issues those tools cannot detect.
