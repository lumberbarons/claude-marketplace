Skills for tracking work in GitHub Issues through the [hew](https://github.com/lumberbarons/hew) CLI.

| Directory | What | When to read |
|-----------|------|--------------|
| `skills/raise-issues/` | Files review findings as deduplicated hew issues | Changing how findings become issues, or the review-key scheme |
| `README.md` | Plugin overview and the review-key contract | Understanding what hew skills do |

Requires the `hew` binary on PATH, authenticated via `gh auth login`.

The review-key scheme in `skills/raise-issues/REFERENCE.md` is the load-bearing part: it is what
lets a review re-run without re-filing. Changing the key format orphans every issue already
filed under the old one, so treat it as a migration, not an edit.
