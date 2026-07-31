# Reference

Pattern vocabulary, file shapes, and a worked example for `raise-issues`.

## Pattern vocabulary

The `pattern` component of a review key must come from this list. It is a closed vocabulary for
one reason: a free-form slug drifts between runs — `unstructured-logging` one cycle,
`string-formatted-logs` the next — and a drifted slug is a duplicate issue.

Use `other-<slug>` when nothing fits. Recurring `other-` slugs are the signal to add an entry
here rather than to keep improvising.

**o11y**

```
credential-in-log            pii-in-log                  error-on-user-input
failure-below-alert-level    missing-log-io-boundary     silent-retry
silent-fallback              unstructured-logging        missing-correlation-id
error-wrap-drops-cause       tautological-message        entry-exit-noise
field-name-drift             message-format-drift        mixed-logger-libraries
startup-config-unlogged      double-reported-failure
```

**tests**

```
tautological-assertion       shared-mutable-state        dead-expectation
loose-matching               isolation-hazard            reimplemented-logic
permanently-skipped-test     unclear-test-name           missing-coverage
missing-edge-case
```

**docs**

```
broken-reference             stale-command               failing-quickstart
missing-prerequisite         drifted-enumeration         derivable-content
oversized-claude-md          hardcoded-local-path        missing-expected-output
```

**code**

```
single-responsibility        leaky-abstraction           untestable-seam
unclear-naming               api-design                  error-handling-strategy
duplicated-logic             dead-code
```

## Findings file

What `--findings <path>` expects. `files` drives the mechanically-derived scope, so it must
list every affected path — for a pattern-collapsed finding, all of them.

```json
{
  "skill": "o11y",
  "scope_reviewed": "internal/http",
  "findings": [
    {
      "priority": "P1",
      "pattern": "credential-in-log",
      "title": "Authorization header written to logs on every request",
      "files": ["internal/http/middleware/logging.go"],
      "locations": ["internal/http/middleware/logging.go:13"],
      "explanation": "LogRequests prints the full r.Header map, so every bearer token reaches stdout and the log aggregator.",
      "fix": "Log an allowlist of headers (User-Agent, Content-Type, X-Request-Id) and drop the raw header map.",
      "done_when": "No call site passes r.Header or any http.Header value into a log field."
    }
  ]
}
```

`priority`, `files`, and `fix` are required; a finding missing any of them is skipped and
counted. `pattern` may be omitted, in which case pick one from the vocabulary above.

## Plan file

One JSON object per line, consumed by `hew apply`. `where` carries the locations and the review
key; `done-when` is a list, one checklist item per entry.

```jsonl
{"title":"Logging: Authorization header written to logs on every request","type":"bug","priority":"P1","where":"internal/http/middleware/logging.go:13\n\nreview-key: o11y/credential-in-log/internal/http/middleware/logging.go","problem":"LogRequests prints the full r.Header map, so every bearer token reaches stdout and the log aggregator.","fix":"Log an allowlist of headers (User-Agent, Content-Type, X-Request-Id) and drop the raw header map.","done-when":["No call site passes r.Header or any http.Header value into a log field."]}
{"title":"Logging: error wraps drop the cause chain","type":"bug","priority":"P2","where":"internal/http/repo/orders.go:25, internal/http/repo/orders.go:34, internal/http/stripe/client.go:41\n\nreview-key: o11y/error-wrap-drops-cause/internal/http","problem":"Every wrap site uses fmt.Errorf with %s and err.Error(), so Unwrap returns nil and errors.Is stops working.","fix":"Replace each with fmt.Errorf(\"...: %w\", err).","done-when":["No fmt.Errorf call in internal/http formats an error with %s or .Error()."]}
```

Add `"discovered-from": <n>` when re-filing a regression against a closed issue.

## Deriving scope

The scope component is the deepest directory containing every file in `files`, or the file
itself when there is one:

| `files` | scope |
|---|---|
| `["internal/http/middleware/logging.go"]` | `internal/http/middleware/logging.go` |
| `["internal/http/repo/orders.go", "internal/http/stripe/client.go"]` | `internal/http` |
| `["cmd/api/main.go", "internal/worker/job.go"]` | *(repo root)* — use the skill name alone: `o11y/silent-retry/.` |

A finding spanning the whole repository is usually a sign the collapse went too far; prefer
splitting it per top-level area over anchoring at the root.

## Worked example

A review reports two findings. The first has been filed before and its issue is still open; the
second matches an issue that was closed as completed.

```
$ hew search "review-key: o11y/credential-in-log/internal/http/middleware/logging.go"
#118 P1 bug  Logging: Authorization header written to logs on every request [open]

$ hew search "review-key: o11y/error-wrap-drops-cause/internal/http"
#96  P2 bug  Logging: error wraps drop the cause chain [closed]

$ hew show 96          # closed how?
... closed as completed by #97 ...
```

The first is skipped — already tracked. The second is a regression: the pattern came back after
a fix shipped, so it is filed fresh with `"discovered-from": 96` linking the history, not
silently re-raised as if it were new.

Had #96 been closed as not-planned, the finding would be suppressed instead — permanently. A
declined issue is a decision, and re-filing it every cycle is how a review pipeline gets muted.
