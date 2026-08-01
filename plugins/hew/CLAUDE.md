Skills for tracking work in GitHub Issues through the [hew](https://github.com/lumberbarons/hew) CLI.

| Directory | What | When to read |
|-----------|------|--------------|
| `skills/raise-issues/` | Files review findings as deduplicated hew issues | Changing how findings become issues, or the review-key scheme |
| `skills/work-issue/` | Takes a tracked issue from claimed to draft PR, test-first | Changing how issues get implemented, verified, or shipped |
| `README.md` | Plugin overview, the review-key contract, and the unattended pipeline | Understanding what hew skills do |

Requires the `hew` binary on PATH, authenticated via `gh auth login`.

The review-key scheme in `skills/raise-issues/REFERENCE.md` is the load-bearing part of that
skill: it is what lets a review re-run without re-filing. Changing the key format orphans every
issue already filed under the old one, so treat it as a migration, not an edit.

`work-issue` rests on two properties of hew that are easy to undo by accident. Work ends at
`hew pr`, never at `hew close` — close means wontfix or duplicate, so closing a delivered issue
skips review and records the opposite of what happened. And `hew start`'s exit 3 on a claimed
issue is the concurrency lock that makes unattended runs safe; anything that routes around it
with `--force` removes the only thing keeping two agents off the same issue.
