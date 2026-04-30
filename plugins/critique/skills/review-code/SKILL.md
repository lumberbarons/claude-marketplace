---
name: review-code
description: Reviews code for design issues that static analysis misses — single responsibility, abstraction levels, testability, meaningful naming, API design, and error-handling strategy. Use when asked to review code, audit a module, check architecture or design, find design problems, improve code quality, or look at testability/coupling/SRP. Also invoke when a user says things like "review this code", "is this well-structured", "audit src/X for design problems", "what's wrong with this module", "check the architecture here", "look for SRP violations", "is this testable", or asks for a design-level (not lint-level) read of a file or directory.
---

# Code Review

Review code for design and architecture issues that linters and static analysis tools miss.

> [!IMPORTANT]
> Consult [REFERENCE.md](REFERENCE.md) for the expected output format and level of detail.

## Prerequisites

This review focuses on design issues. Standard tooling handles the mechanical checks and is assumed to run alongside (not before) this review:
- **Linters** (ESLint, golangci-lint, pylint) catch complexity, length, nesting, unused code
- **Formatters** (prettier, gofmt, black) handle style
- **Security scanners** (Semgrep, CodeQL, Bandit) catch injection, XSS, secrets
- **Type checkers** (TypeScript, mypy) catch type errors

If any of these aren't set up, mention it in the report so the team can add them — but proceed with the design review regardless. Skip mechanical findings that those tools would catch.

## Scope

Determine the review scope before discovering files:

- If `$ARGUMENTS` is non-empty, treat it as a path (file or directory) and run:
  ```bash
  ${CLAUDE_PLUGIN_ROOT}/scripts/discover-files.sh "$ARGUMENTS"
  ```
- If `$ARGUMENTS` is empty, scope to files added or modified on the current branch relative to the default branch:
  ```bash
  ${CLAUDE_PLUGIN_ROOT}/scripts/discover-files.sh
  ```

Handle the script's exit codes:
- **0 with output** — use the listed paths as input to the discovery step below.
- **0 with empty output** — branch has no diff vs the default branch. Tell the user and ask which path to review.
- **non-zero** — script prints a message to stderr (path not found, not a git repo, on the default branch with no path, detached HEAD, or default branch indeterminate). Relay the message and ask the user which path to review.

The script returns paths language-blind. The discovery step below filters to source files; if the filter excludes everything but the script's output was non-empty, the language may not be in the pattern list — apply judgment to identify source files in the output.

## Workflow

### Step 1 — Discover source files

From the script's output, filter to source files, excluding test files, vendored/generated code, and files that are purely configuration. Record the full file list and count.

### Step 2 — Choose execution strategy

- **1–4 files → Direct mode**: Read the files, evaluate against the Design Criteria below (skip mechanical checks tools handle), then proceed to Pattern Collapsing.
- **5+ files → Parallel mode**: Batch files, spawn subagents, collect results, merge, then proceed to Pattern Collapsing.

Adjust by file size when the count is on the boundary: if 5–6 files are small (under ~200 lines each), direct mode is fine; if 3–4 files are large (over ~400 lines each), prefer parallel mode. Use judgment — the goal is to avoid serially reading thousands of lines in one context, not to hit an exact threshold.

### Parallel Review Mode

Use this mode when there are enough files (or files are large enough) that reading them serially would meaningfully slow the review.

#### Batching

Group files into batches based on total file count:

| Total files | Files per batch | ~Subagents |
|-------------|-----------------|------------|
| 5–10        | 1               | 5–10       |
| 11–20       | 2               | 6–10       |
| 21+         | 3               | 7–10       |

#### Spawn subagents

For each batch, use `Agent(subagent_type="general-purpose")`. **Spawn all subagents in a single message** so they run in parallel.

Each subagent prompt MUST include:

1. The file paths in its batch (instruct the subagent to read them)
2. The **Design Criteria** section from this skill — copy it verbatim into the prompt
3. The **Severity** section from this skill — copy it verbatim into the prompt
4. The **Prerequisites** note — remind the subagent to skip mechanical checks that linters/formatters/scanners handle
5. The structured output format below
6. The explicit instruction: **"Do NOT use the Bash tool. Do NOT run any shell commands. Use only Read, Grep, and Glob tools. Return findings only."** — the review is static analysis of source files, so shell access adds latency and side-effect risk without enabling anything the read-only tools can't already do.
7. The explicit instruction: **"For every P2 and P3 finding, you MUST state a concrete consequence in the `explanation` field: name a specific extension, change, or maintenance scenario where a caller or maintainer would predictably go wrong because of this design (e.g., 'adding a CLI entry point would force re-implementing the discount logic that's currently embedded in the HTTP handler'). Omit findings that lack this claim. The single exception is P1 findings (security design flaws and tests-cannot-be-written designs carry their consequence implicitly)."**
8. The explicit instruction: **"For the `pattern` field, use a short, reusable label that names the underlying anti-pattern (e.g., 'module-scope side effects', 'mixed abstraction in handlers'). If two findings in your batch stem from the same root cause, they MUST use the same pattern label."**

Instruct each subagent to return findings in this exact delimited format (one block per finding):

```
---FINDING---
priority: P<1|2|3>
location: <file:line>
title: <short title>
category: <Single Responsibility|Abstraction Levels|Meaningful Naming|Testability|API Design|Error Handling Strategy>
pattern: <short label for the underlying anti-pattern, e.g. "module-scope side effects" or "mixed abstraction in handlers" — use the SAME label across findings that share the same root cause>
explanation: <what is wrong and why it matters>
fix: <concrete prescription>
done_when: <verifiable criterion>
---END---
```

If the subagent finds no issues for its batch, it should return `---NO-FINDINGS---`.

#### Collect and merge

After all subagents return:

1. Parse each subagent's structured findings
2. Combine into a single list, sorted by priority (P1 first)
3. Deduplicate: if two findings share the same `location` (file:line) AND the same `category`, keep only the one with the highest priority
4. Group findings by `pattern` label — findings from different subagents that used the same (or very similar) pattern label share a root cause and will be collapsed in the Pattern Collapsing step

#### Error fallback

If a subagent fails or returns unparseable output, review those files directly (as in direct mode) and include a note in the report: `Note: Files [list] were reviewed directly due to subagent failure.`

### Pattern Collapsing

Both direct mode and parallel mode flow into this step before producing the final report.

After merging all findings, look for findings that share the **same root cause** — i.e., the same design pattern repeated across multiple files. Examples:

- Multiple files flagged for "module-level side effect blocks testability" → one pattern: "codebase initialises services at module scope instead of using injection"
- Multiple files flagged for "mixed abstraction levels" where the same kind of mixing recurs → one pattern: "business logic is interleaved with infrastructure calls throughout"

When you identify a shared root cause:

1. **Collapse** the N per-file findings into **one finding** that names the pattern, lists all affected files, and prescribes the codebase-wide fix
2. **Set severity** to the highest severity among the collapsed findings
3. **Keep separate** any findings that happen to share a category but have genuinely different root causes

This is critical: N findings for N instances of the same pattern creates noise. One finding that names the pattern and lists the affected locations is actionable.

---

## Design Criteria

### Single Responsibility

Flag when a unit has multiple unrelated reasons to change:
- Functions that do X AND Y (validate AND save, fetch AND transform AND render)
- Classes with unrelated method groups (UserService with email sending and caching)
- Files with unrelated exports
- Modules mixing infrastructure with business logic

Ask: "If requirement X changes, does this code change? What about unrelated requirement Y?"

### Abstraction Levels

Flag when a function mixes high-level intent with low-level details:
- Business logic interleaved with HTTP/DB/file operations
- Algorithm steps mixed with error handling boilerplate
- Policy decisions embedded in mechanism code

Good: each function operates at one level, delegating details to helpers.

### Meaningful Naming

Linters enforce patterns; this checks *meaning*:
- Does `processData` actually explain what processing occurs?
- Does `handleRequest` distinguish itself from other handlers?
- Do similar names (`userService`, `userManager`, `userHelper`) have clear distinct roles?
- Are abbreviations obvious to someone new to the codebase?

Flag: names that pass linter rules but don't reveal intent.

### Testability

Flag architectural decisions that make testing hard:
- Direct instantiation of external services (no injection points)
- Static/global state that persists between tests
- Business logic that can only run with real I/O
- Hidden side effects (function does more than signature suggests)
- Circular dependencies between modules

Ask: "Can I test this unit in isolation with fake dependencies?"

### API Design

Flag interface issues:
- Leaky abstractions (caller must understand implementation details)
- Inconsistent patterns across similar APIs
- Missing or misleading error information
- Temporal coupling (must call A before B, but nothing enforces it)

### Error Handling Strategy

Linters catch empty catches; this checks *appropriateness*:
- Are errors handled at the right level? (not swallowed too early, not leaked too far)
- Do error messages help the *recipient* (user vs developer vs operator)?
- Is the recovery strategy sensible? (retry? fail fast? degrade gracefully?)
- Are error paths tested?

---

## Severity

Severity is assigned **per-finding by predictable consequence**, not by category. Pattern labels exist only to drive Pattern Collapsing — they do not set severity. Two findings sharing a pattern label can have different severities if their consequences differ in concreteness or stakes.

The driving question for every P2/P3 finding: **"Name a specific extension, change, or maintenance scenario that would predictably go wrong because of this design."** If you can only say "this could be cleaner" without naming a realistic scenario, the finding is below the reporting bar — omit it. P1 findings carry their consequence implicitly (the code can't be tested at all, or the design is a security vulnerability).

### P1 — the design blocks testing entirely, or it is a security vulnerability
- Code that **cannot** be tested without heroic workarounds — no seam exists even with standard mocking. A module-level singleton that can be mocked with `vi.mock()` or `monkey.Patch` is **not** P1 — that's testable, just inconvenient. P1 means "there is no way to isolate this code for testing."
- Security design flaws: authentication bypass by design, privilege escalation paths, secrets in code that escape into logs/responses, missing authorization on privileged operations.

### P2 — a specific reasonable change predictably goes wrong because of this design
The design imposes a real cost on a real future scenario you can name. Reserved for:
- **SRP violations with named ripple**: "If pricing rules change, this validation function also needs review" — name the actual coupling, not just "function does multiple things."
- **Mixed abstractions with named cost**: "Adding a CLI entry point would require re-implementing the business logic currently interleaved with HTTP parsing" — name the entry point, transport, or extension that the mixing blocks.
- **Leaky APIs with named misuse**: "Callers must remember to call `Init()` before `Run()` and nothing enforces it; this has already caused bug X / will trip up the next implementer of feature Y."
- **Testability friction with named cost**: module-level state or hard-coded dependencies that force mocking at import time, where you can name the test scenario this prevents from being expressed cleanly.
- **Inconsistent patterns across similar APIs**: name the specific divergence and the maintenance cost ("`UserService.Get` returns `(User, error)` but `OrderService.Fetch` returns `(*Order, bool)`; consumers writing generic helpers cannot share code").

### P3 — a smaller, more peripheral consequence
Same shape as P2 but the named consequence is narrower or affects fewer scenarios. Naming and minor API issues live here.
- "A caller would likely call `processOrder` thinking it does X when it actually does Y."
- A divergent name in one place that doesn't follow codebase convention; a maintainer skimming will misclassify the function's role.
- Suboptimal but workable patterns where the consequence affects readability more than future change cost.

### The P2/P3 boundary
Ask whether the named consequence affects a **reasonable extension or change someone is likely to make** (P2) or just a **single reader's first impression** (P3). A design issue that forces a future refactor to land everywhere is P2; a design issue that just reads awkwardly is P3.

If you cannot name *any* specific consequence, the finding is below the reporting bar — omit it entirely. Do not promote "this could be cleaner" to P3 just because it sounds mild.

---

## Reporting Cap

After Pattern Collapsing, cap the report at **10 findings total**. The cap is a ceiling, not a target — if there are only 4 real findings, report 4. **Do not manufacture filler to reach 10.** A short report of high-impact findings beats a padded report of weak ones, and a padded report trains the reader to ignore the long tail.

Selection rules, applied in order:

1. **Include every P1.** Never truncate a P1 — security design flaws and untestable code don't belong in a tail. If P1s alone exceed 10, report all of them and skip P2/P3 entirely.
2. **Fill remaining budget with P2s, then P3s**, ordered by impact within each tier. Impact favors findings whose named consequence affects more scenarios, more callers, or wider blast radius — and pattern-collapsed findings that span more locations.
3. **Footer the tail.** When findings exceed the cap, end the report with: `Note: N additional findings omitted (X P2, Y P3) — re-run after addressing these to surface what remains.` When findings fit under the cap, no footer is needed.

The reasoning: design changes have wider blast radius than test fixes, so churn matters more here, not less. A 30-finding design review is unmergeable as one PR — it will age out, get partially applied, or split attention away from the highest-impact changes. Iterating in batches of 10 is what humans actually do, and re-running after fixes surfaces issues that only become visible once the most pressing ones are out of the way (often a single cross-cutting refactor dissolves several adjacent findings). The cap also creates healthy pressure against the "asked to find things, so finds things" failure mode — if your candidate finding wouldn't make it into the top 10, it probably isn't worth the reader's attention.

---

## Output

Produce a report following the exact structure shown in [REFERENCE.md](REFERENCE.md). The format is fixed so readers can scan a review the same way every time, and so downstream tooling (raise-beads, dashboards) can parse findings without per-run exceptions. When using parallel mode, the lead assembles the unified report from subagent findings — the report format is identical regardless of execution mode.

Each finding MUST include:

- **Priority** (P1/P2/P3) in the H3 header
- **Location** (file:line) on its own line
- **Explanation** of the problem and why it matters
- **Fix** — concrete prescription. For API design issues, specify the exact shape (parameter names, types, signatures), not just the general approach
- **Done when** — a verifiable completion criterion that can be checked by reading the diff. Must reference specific functions, files, or observable behaviours. Example: "Both parseSkillFrontmatter and parseAgentFrontmatter delegate to a shared parseFrontmatterRaw; no duplicated delimiter-scanning code remains." NOT: "The duplication is removed."
