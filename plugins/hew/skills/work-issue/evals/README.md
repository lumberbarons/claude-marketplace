# work-issue evals

How to run the eval set in `evals.json`. The fixtures themselves live in
`../../work-issue-workspace/fixtures/`, which is gitignored like every other skill workspace in
this repo — this file is the tracked description of what has to be built there.

## Why a stub instead of a fixture repo

`work-issue` is different from the critique skills to evaluate. Those read a code fixture and
emit a report, so a directory of source files is the whole harness. This one *acts*: it claims
issues, pushes branches, and opens pull requests. Running it against a live repository to score
it would create real issues and real PRs, and every rerun would need them cleaned up first —
which also makes the runs non-deterministic, because the queue is different on the second pass.

So the fixture replaces the CLI rather than the repository.

## The hew stub

`hew-stub/hew` is an executable placed ahead of the real binary on `PATH`. It serves canned
responses for the scenario named by `$HEW_SCENARIO` and appends every invocation to
`$HEW_LOG` — one command line per row.

That log is what most assertions actually read, because the interesting claims about this skill
are claims about *protocol*, and protocol is a sequence of commands:

- did `hew start` come before the first source edit, or after?
- was there a test run between the branch and the implementation, and did it fail?
- did the run end at `hew pr`, or did it call `hew close`?
- when `start` returned exit 3, did the run move on or reach for `--force`?

None of those are visible in a final diff. All of them are visible in an ordered command log,
and none of them require a network.

## Scenarios

Each scenario is a directory under `../../work-issue-workspace/fixtures/` holding the canned
`ready`/`show`/`list` payloads plus a small real project — real enough that TDD has something to
bite on, since a test that cannot fail proves nothing about a skill that is supposed to watch it
fail.

| Scenario | Shape | What it isolates |
|---|---|---|
| `scenario-ready-bug` | One ready `bug` with a three-item Done-when | The happy path: claim → red → green → verify → PR |
| `scenario-epic` | `#40` is an epic; children open, closed, and blocked | Descent, and the filters that pick the right child |
| `scenario-all-ineligible` | Ready queue of one claimed and two untriaged issues | `not_eligible` vs `no_ready_work` — the distinction the loop rests on |
| `scenario-no-done-when` | Issue with Where/Problem/Fix and an empty Done-when | Whether criteria get synthesized or the finish line gets guessed |
| `scenario-gate-fails` | A test in the project fails for a reason outside the issue's scope | Push-but-no-PR, and leaving the claim in place |

`scenario-gate-fails` is the one worth building carefully. The failure has to be genuinely
outside `### Where`, or a capable model will simply fix it and pass the eval for the wrong
reason — the behaviour under test is restraint plus an honest report, not repair.

## Exit codes the stub must reproduce

| Code | Condition | Why it matters here |
|---|---|---|
| 3 | `start` on a claimed issue | The concurrency lock. The skill must treat it as "move on", never as an error to force past. |
| 4 | any command, unauthenticated | Must surface as `error`, not as an empty backlog. |
