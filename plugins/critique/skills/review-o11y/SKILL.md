---
name: review-o11y
description: Reviews observability — logging consistency, log level appropriateness, log value, missing logs at I/O boundaries, and error-message quality. Use when asked to review logging, observability, log quality, error messages, or to audit how a codebase logs and reports failures. Also invoke when a user says things like "check the logging", "are our logs any good", "do we log the right things", "are error messages consistent", or asks why operational visibility is poor in a service.
---

# Observability Review

Review a codebase's logging and error messages for consistency, level appropriateness, value, and coverage at I/O boundaries. Language-agnostic. This skill does **not** judge error-handling strategy (where to catch, whether to retry, whether to degrade) — that belongs to `review-code`. This skill judges the *artifacts*: the log statements that get emitted and the error messages that get constructed.

> [!IMPORTANT]
> Consult [REFERENCE.md](REFERENCE.md) for the expected output format and level of detail.

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

## Prerequisites

This review assumes standard tooling is already running:
- **Linters** catch empty `catch` blocks, unused imports, obviously broken format strings
- **Security scanners** catch hardcoded secrets and a few well-known PII patterns

This skill focuses on observability design issues those tools cannot detect: whether logs are useful, whether levels carry meaning, whether context is preserved, whether error messages are consistent and actionable, and whether the codebase logs at the places that matter operationally.

## Philosophy — prescriptive vs descriptive

This skill has two modes of judgement. Understanding the split is critical so you don't flag style choices as bugs or give bad logging a clean pass because it is "consistent".

**Prescriptive** — the skill has a fixed opinion regardless of what the codebase currently does. These rules exist because they reflect a broad operational consensus: break them and real oncall pain follows. Flag violations even if the whole codebase is consistently wrong.

**Descriptive** — the skill detects the codebase's dominant convention and flags outliers. These cover style choices where reasonable shops legitimately differ (logger library, field-name casing, error-message punctuation). A shop's consistent choice is its right; the skill's job is to notice drift, not to impose taste.

**Rule of thumb: semantics are prescriptive, syntax is descriptive.** Whether logs carry correlation context is a semantic question — always enforced. Whether the correlation id is called `request_id`, `requestId`, or `trace_id` is a syntactic question — detect the dominant choice and flag outliers.

## Workflow

### Step 1 — Discover source files

From the script's output, filter to source files, excluding test files, generated code, vendored dependencies, and pure configuration. Record the file list and count.

### Step 2 — Detect the dominant conventions

Before flagging anything on the descriptive axis, determine what "normal" looks like in this codebase. Sample up to 40 source files (or all of them, whichever is smaller). For each of the following axes, tally the distinct patterns and pick the majority:

| Axis | What to record |
|------|---------------|
| Logger library | e.g. `slog`, `zap`, `logrus`, `winston`, `pino`, stdlib `logging`, direct `print`/`console.log` |
| Log call shape | structured (key-value fields) vs string-formatted vs mixed |
| Field-name casing | `snake_case`, `camelCase`, `kebab-case` |
| Correlation field names | e.g. `request_id`, `trace_id`, `user_id` — which names are used |
| Error-message capitalization | leading uppercase vs lowercase |
| Error-message trailing punctuation | period vs none |
| Error-layer separator | `": "`, `" - "`, `", caused by "`, etc. |
| Error-message verb form | `"failed to X"`, `"could not X"`, `"X failed"`, `"error while X"` |
| Error construction pattern | sentinel errors, factory helpers, wrap-at-call-site, exception subclasses |

**Decision rule for flagging outliers:**

- A pattern that represents **≥70%** of the sampled occurrences is the dominant convention. Flag minority-pattern instances as outliers.
- If the top pattern is **<70%** but **≥40%**, the codebase has a legitimate split. Do **not** flag either side as an outlier. Instead raise a single P3 pattern finding: "codebase uses two conventions for <axis>; pick one." List representative files for both sides.
- If no pattern exceeds 40%, the codebase is chaotic on that axis. Raise a single P2 pattern finding naming the chaos and listing the top two or three variants.

Record the detected conventions and use them as the baseline in Step 3. Include a short "Detected conventions" block at the top of the final report so the reader can see what you anchored against (and catch you if you anchored wrong).

### Step 3 — Choose execution strategy

- **1–2 files → Direct mode**: Read the files, evaluate against the Criteria below, then proceed to Pattern Collapsing.
- **3+ files → Parallel mode**: Batch files, spawn subagents, collect results, merge, then proceed to Pattern Collapsing.

### Parallel Review Mode

Use this mode when 3 or more source files are discovered.

#### Batching

Group files into batches based on total file count:

| Total files | Files per batch | ~Subagents |
|-------------|-----------------|------------|
| 3–10        | 1               | 3–10       |
| 11–20       | 2               | 6–10       |
| 21+         | 3               | 7–10       |

#### Spawn subagents

For each batch, use `Agent(subagent_type="general-purpose")`. **Spawn all subagents in a single message** so they run in parallel.

Each subagent prompt MUST include:

1. The file paths in its batch (instruct the subagent to read them)
2. The **Criteria** section from this skill — copy it verbatim into the prompt
3. The **Severity** section from this skill — copy it verbatim into the prompt
4. The **detected conventions** from Step 2 — so the subagent knows what counts as an "outlier" on the descriptive axes
5. The structured output format below
6. The explicit instruction: **"Do NOT use the Bash tool. Do NOT run any shell commands. Use only Read, Grep, and Glob tools. Return findings only."**
7. The explicit instruction: **"For every P3 finding, you MUST state a concrete consequence in the `explanation` field: 'a caller/operator would likely \<specific operational mistake\> because of this.' Omit P3 findings that lack this claim."**
8. The explicit instruction: **"For the `pattern` field, use a short, reusable label that names the underlying anti-pattern (e.g., 'unstructured logging', 'error messages drop wrapped cause', 'PII in request logs'). If two findings in your batch stem from the same root cause, they MUST use the same pattern label."**

Instruct each subagent to return findings in this exact delimited format (one block per finding):

```
---FINDING---
priority: P<1|2|3>
location: <file:line>
title: <short title>
category: <Logging Consistency|Log Level|Log Value|Missing Logs|Error Message Quality|Error Message Consistency>
axis: <prescriptive|descriptive>
pattern: <short label for the underlying anti-pattern — use the SAME label across findings that share the same root cause>
explanation: <what is wrong and why it matters operationally>
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

Observability problems tend to be uniform: if the codebase logs unstructured strings, it does so everywhere; if correlation ids are missing from one handler, they are usually missing from all of them. N per-file findings for the same pattern is noise. One finding that names the pattern and lists all affected locations is actionable.

When you identify a shared root cause:

1. **Collapse** the N per-file findings into **one finding** that names the pattern, lists all affected files, and prescribes the codebase-wide fix
2. **Set severity** to the highest severity among the collapsed findings
3. **Keep separate** any findings that share a category but stem from genuinely different root causes

Examples of legitimate collapses:

- 30 files using `logger.info("processing user %s", uid)` → one pattern: "codebase uses string-formatted logging throughout — no structured fields, log aggregators cannot filter or alert"
- 12 inbound handlers with no `request_id` in logger context → one pattern: "inbound handlers do not attach correlation ids; operators cannot stitch together a single request across services"
- 18 sites calling `fmt.Errorf("failed: %s", err.Error())` instead of `fmt.Errorf("failed: %w", err)` → one pattern: "errors are wrapped as strings, dropping the cause chain — `errors.Is`/`errors.As` no longer work"

---

## Criteria

### Logging Consistency *(mostly descriptive — see Step 2 baseline)*

- One logger library per codebase. Mixed libraries or direct `print`/`console.log` alongside structured logging are flagged as outliers against the detected majority.
- Field names for the same concept should be stable. Flag drift like `user_id` in one handler, `userId` in another, `uid` in a third.
- Log call shape should be consistent. Structured and string-formatted calls mixed in the same codebase are flagged.

### Log Level Appropriateness *(prescriptive)*

Log levels must carry operational meaning or they are just decoration. Enforce these definitions regardless of current codebase practice:

- **ERROR** — something the operator must act on. An unexpected failure, a dependency down, a programmer bug. *Not* for expected user-input rejections (bad password, 404 on a missing resource), because paging on those trains operators to ignore the channel.
- **WARN** — a degraded or suspicious condition that was handled, but an operator should know it is happening (cache miss storm, retry succeeded after N attempts, falling back to the secondary provider).
- **INFO** — state changes worth remembering after the fact: startup, shutdown, config load, user action that matters for audit.
- **DEBUG** — the verbose channel for diagnosing a specific problem. Entry/exit traces, full payloads, loop counters belong here if anywhere.

Flag:
- ERROR used on user-caused conditions — this is the most common and most damaging level mistake.
- WARN used as "mild error" without an operational meaning.
- INFO inside hot loops or per-request in high-QPS paths.
- The same failure logged at ERROR *and* propagated — pick one owner for the report. The usual owner is the top of the call stack (handler boundary), and inner layers wrap and return.

### Log Value *(prescriptive)*

- **No tautological messages** — `"an error occurred"`, `"something went wrong"`, `"failed"` with no further context. These are worse than useless because they burn alert volume without giving the operator anywhere to start.
- **Errors are logged with the error chain**, not just `err.Error()` or `str(e)`. The wrapped cause and stack are what the operator needs.
- **No entry/exit noise** outside DEBUG. `logger.info("entering processOrder")` is free cost with no operational value.
- **No sensitive data in logs** — credentials, auth headers, full request/response bodies with user data, query strings containing tokens, PII that was not explicitly allowlisted. This is P1 because it is often a compliance and security incident rolled into one.
- **Correlation context** — at minimum a request id or trace id attached to the logger at inbound handler boundaries, so logs from a single request can be stitched together. Missing correlation context at a handler boundary is prescriptive and P2.

### Missing Logs *(prescriptive, anchored to I/O boundaries)*

The skill is deliberately narrow about where it demands logs. Flagging "this pure function should log" is noise. Flag **only** when the absence of a log at a specific kind of boundary would cause real operational blindness:

- **External calls** (outbound HTTP, gRPC, queue publish, DB query on a separate service) with no log on success, failure, or latency
- **Inbound request handlers** with no access-log equivalent (one log per request, with status and duration)
- **Retries or fallbacks taken silently** — the entire point of a retry is to let operators know a dependency is flaking. A retry that logs nothing is a retry that hides degradation.
- **Degraded-mode branches** (fallback to cache, secondary provider, default value) taken silently
- **Startup configuration** loaded silently — no way to answer "which config did this process load" from the logs alone

If you are tempted to flag a missing log somewhere that is not one of these boundaries, don't.

### Error Message Quality *(prescriptive)*

Judging the *messages themselves*, not the handling strategy.

- **Actionable** — the message should give the recipient (usually an operator reading a log or a developer reading a stack trace) enough to start debugging. `"failed to connect to postgres at postgres-primary:5432: dial tcp: connection refused"` beats `"db error"`.
- **Preserves the cause chain** — when wrapping, the inner error must be retrievable. In Go: `fmt.Errorf("...: %w", err)` not `fmt.Errorf("...: %s", err.Error())`. In Python: `raise X from e` not bare `raise X`. In JS/TS: `new Error("...", { cause: err })`. Flag sites that lose the chain, and flag sites that *duplicate* the inner message on top of the wrap (`"failed to load config: failed to open file: open: no such file"`).
- **Safe** — no secrets, tokens, full payloads, or PII inside error messages. Error messages frequently end up in logs, HTTP responses, or third-party error reporters.
- **Specific enough to locate** — avoid messages so generic they collide with dozens of call sites. `"invalid input"` with no further context is un-greppable; `"invalid input: email missing @"` is.

### Error Message Consistency *(descriptive — flagged against Step 2 baseline)*

- Same format convention across the codebase (capitalization, trailing punctuation, layer separator, verb form). Outliers against the detected majority are findings.
- Same domain concept uses the same phrasing. If `"user not found"` appears in six places, a stray `"no such user"` is a finding — not because it is wrong, but because operators grepping for the known phrase will miss it.
- Error construction goes through the same pattern (sentinel, factory, wrap, exception subclass). Mixing patterns arbitrarily across files is a finding; a principled split (e.g. sentinels for domain errors, wraps for infrastructure errors) is not.

---

## Severity

- **P1**: Secrets, tokens, auth headers, or PII emitted into logs or error messages. Errors silently swallowed with no log and no propagation. Log format strings that always crash at runtime (argument mismatch that throws on every call). These are incidents waiting to happen; they page someone eventually and the blame lands on observability debt.

- **P2**: Log levels misused in ways that cause real operational pain — ERROR on expected user-input failure (pages oncall for nothing), or operationally-critical failure logged at DEBUG (invisible to alerts). Missing logs at I/O boundaries (external calls, inbound handlers, silent retries/fallbacks). Error wrapping that drops the cause chain so `errors.Is`/`except X from e`/`cause` no longer works. Codebase-wide unstructured logging (one collapsed finding, P2) — log aggregators cannot filter or alert on string-formatted logs. Missing correlation ids at inbound handler boundaries.

- **P3**: Field-name drift, formatting inconsistency, entry/exit noise, unactionable messages on rare paths, consistency outliers against a clear dominant convention. P3 findings **must** state a concrete consequence: *"an operator grepping for `failed to` misses half the failures because this site says `could not`"*, not *"this is inconsistent"*. If you cannot state a specific operational mistake that would follow, omit the finding.

---

## Output

You MUST produce a report following the exact structure shown in [REFERENCE.md](REFERENCE.md). When using parallel mode, the lead assembles the unified report from subagent findings. The format is identical regardless of execution mode.

The report begins with a short **Detected conventions** block showing what Step 2 anchored against — logger library, log-call shape, field casing, error-message format. This lets the reader sanity-check the baseline before reading descriptive findings.

Each finding MUST include:

- **Priority** (P1/P2/P3) in the H3 header
- **Location** (file:line, or a list of locations if the finding was pattern-collapsed)
- **Axis** (prescriptive or descriptive) — so the reader can tell whether this is an enforced rule or a consistency call
- **Explanation** of the problem and the operational consequence
- **Fix** — concrete prescription. For consistency findings, reference the detected baseline ("the dominant convention in this codebase is lowercase leading verbs; this site uses capitalized"). For prescriptive findings, reference the rule ("correlation context is required at inbound handler boundaries").
- **Done when** — a verifiable criterion checkable by reading the diff. Example: *"Every handler in `internal/http/` obtains a logger from `ctx` that already has `request_id` attached."* NOT: *"Correlation context is added."*
