# hew

Claude Code skills for tracking work in GitHub Issues through the
[hew](https://github.com/lumberbarons/hew) CLI.

## Overview

- **raise-issues** — turns structured review findings into hew issues, deduplicated against
  what is already tracked so a review can run repeatedly without re-filing the same work.

## Prerequisites

- [`hew`](https://github.com/lumberbarons/hew) on PATH, authenticated via `gh auth login`
- Run from a checkout of the target repository (`--repo owner/name` overrides detection)
- `hew init` once per repo, to create the convention labels

## Skills

| Skill | Description | Model-Invocable |
|-------|-------------|-----------------|
| `/hew:raise-issues` | File review findings as deduplicated GitHub issues | Yes |

### Usage

```
/critique:review-o11y internal/http     # produce findings
/hew:raise-issues                       # file them

/hew:raise-issues --findings out/o11y.json --dry-run
/hew:raise-issues only P1 and P2
```

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

Dedup then follows hew's own read order — `hew search` first, `hew list --json --bodies
--state all` when exhaustiveness matters, `hew show` only for a specific candidate — across
open *and* closed issues. A closed-as-completed match means the pattern regressed and is filed
fresh with `--discovered-from`; a closed-as-declined match is suppressed for good.

## Running unattended

`raise-issues` is built to survive a scheduled loop:

- `--findings <path>` takes a findings file instead of scraping conversation context
- `--non-interactive` removes every question and reports the outcome instead
- writes go through `hew apply`, which checkpoints each creation, so an interrupted run resumes
  without duplicating what already landed
- the final report distinguishes "found nothing" from "received nothing", so a broken pipeline
  cannot masquerade as a clean codebase
