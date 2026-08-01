# hew

Claude Code skills for tracking work in GitHub Issues through the
[hew](https://github.com/lumberbarons/hew) CLI.

## Overview

- **raise-issues** — turns structured review findings into hew issues, deduplicated against
  what is already tracked so a review can run repeatedly without re-filing the same work.
- **work-issue** — takes a tracked issue from `hew ready` to a draft PR, test-first, with the
  issue's own "Done when" checklist driving the tests.

Together they close the loop: a review files what is wrong, and the next run picks it up and
fixes it.

## Prerequisites

- [`hew`](https://github.com/lumberbarons/hew) on PATH, authenticated via `gh auth login`
- Run from a checkout of the target repository (`--repo owner/name` overrides detection)
- `hew init` once per repo, to create the convention labels

## Skills

| Skill | Description | Model-Invocable |
|-------|-------------|-----------------|
| `/hew:raise-issues` | File review findings as deduplicated GitHub issues | Yes |
| `/hew:work-issue` | Take a tracked issue from claimed to draft PR, test-first | Yes |

### Usage

```
/critique:review-o11y internal/http     # produce findings
/hew:raise-issues                       # file them
/hew:work-issue                         # work the top of the queue

/hew:raise-issues --findings out/o11y.json --dry-run
/hew:raise-issues only P1 and P2

/hew:work-issue 42                      # a specific issue, or an epic to descend into
/hew:work-issue --all                   # drain the ready queue
/hew:work-issue --dry-run
```

## Working an issue

`work-issue` follows the workflow hew prescribes rather than inventing one: `hew ready` →
`hew start` → branch → tests → push → `hew pr`. Two properties of that workflow do the heavy
lifting.

**Work ends at a PR.** `hew pr` composes the PR from tracker state with exactly one `Fixes #n`,
so the merge closes the issue and the change goes through review. `hew close` stays what it is
in hew — a human deciding no work was needed.

**`hew start` is a lock.** It refuses a claimed issue with exit 3, so two agents cannot take the
same issue and the loser simply moves down the queue. That is what makes an unattended run safe
to point at a shared backlog.

The `### Done when` checklist is what makes the test-first part more than ceremony: the
acceptance criteria already exist, written by whoever filed the issue and checked by whoever
reviews the PR. Behavioural items become failing tests before any implementation; structural
ones ("no call site still passes the raw header") become checks run against the diff, because a
unit test asserting the absence of a pattern is theatre.

Epics are trees, not work — given one, the skill descends to the next ready child, works exactly
that, and leaves the epic open for the next run.

## The review key

Every issue this skill files carries a key in its `### Where` section:

```
review-key: <skill>/<pattern>/<scope>
```

The key is the identity of a *finding*, not of a line of code, which is what makes it survive
the two things that change constantly — line numbers and the model's phrasing of a title.

- `skill` — which review produced it (`o11y`, `tests`, `docs`, `code`)
- `pattern` — the root-cause slug the review emitted, from the vocabulary each critique skill
  owns in its own `REFERENCE.md`, passed through unchanged
- `scope` — the deepest directory containing every affected file, derived mechanically from the
  file list rather than chosen, so two runs cannot anchor the same finding differently

Severity is deliberately excluded: it moves between runs, and identity must not.

Dedup then follows hew's own read order — `hew search` first, then `hew list --json --bodies`
plus `hew list --json --bodies --closed` when exhaustiveness matters, and `hew show` only for a
specific candidate — across open *and* closed issues. A closed-as-completed match means the
pattern regressed and is filed fresh with `--discovered-from`; a closed-as-declined match is
suppressed for good.

## Running unattended

Both skills are built to survive a scheduled loop.

`raise-issues`:

- `--findings <path>` takes a findings file instead of scraping conversation context
- `--non-interactive` removes every question and reports the outcome instead
- writes go through `hew apply`, which checkpoints each creation, so an interrupted run resumes
  without duplicating what already landed
- the final report distinguishes "found nothing" from "received nothing", so a broken pipeline
  cannot masquerade as a clean codebase

`work-issue`:

- `--non-interactive` removes every question; `--json <path>` writes a machine-readable outcome
- one issue per invocation by default, so each tick costs one reviewable PR rather than ten
- `hew start`'s exit 3 keeps parallel runs off each other's work without any coordination
- a run that cannot verify its change pushes the branch, opens no PR, and leaves the issue
  claimed — so the next tick moves on instead of retrying a broken change forever
- `status` separates the four ways to finish with zero PRs: `no_ready_work` (drained),
  `not_eligible` (something else holds the queue), `failed`, and `error`

The whole pipeline runs unattended end to end:

```
/critique:review-tests --json out/tests.json --non-interactive
/hew:raise-issues --findings out/tests.json --non-interactive
/hew:work-issue --non-interactive --json out/work.json
```
