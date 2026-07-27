---
name: review-docs
description: Review documentation (README.md and CLAUDE.md) for quality, completeness, and consistency. Use when asked to review docs, check documentation, validate README files, or audit CLAUDE.md coverage.
---

# Documentation Review

Review documentation in the specified path (default: entire repository).

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

The script returns paths language-blind. The discovery step below filters to README.md and CLAUDE.md files; this filter is universal across languages so a fallback is rarely needed.

## Workflow

1. Run deterministic validation script for structural issues (always repo-wide — structural drift outside the branch scope still matters)
2. Find README and CLAUDE.md files in the script's output
3. Evaluate against quality criteria
4. Cross-reference enumerations in docs against codebase sources of truth (see Enumeration Completeness)
5. Produce the findings report

### Deterministic Validation

Before manual review, run the validation script for CLAUDE.md structural issues. Pass the repo root so structural problems anywhere in the repo are caught regardless of branch scope:

```bash
python3 ${CLAUDE_PLUGIN_ROOT}/skills/review-docs/scripts/validate-claude-md.py . --json
```

Parse the JSON output and include structural issues (broken references, oversized files) in your findings. The script checks:
- Whether referenced files/directories exist
- CLAUDE.md length against the 200-line target
- `@path` import targets resolve to existing files
- `.claude/rules/` file structure, frontmatter globs, and empty files
- Leaked local preferences (hardcoded user paths, localhost URLs) in shared CLAUDE.md

## Quality Criteria

### Structure (Progressive Disclosure)

README.md files belong at **component boundaries** — the repo root plus the top-level directory of each major component (e.g., `frontend/`, `backend/`, `infra/`, `docs/`). Most subdirectories within a component should not have their own README. If a directory only needs a note about how to work in it, a CLAUDE.md is sufficient.

**Root README.md**:
- Project purpose (why it exists)
- What it does (brief overview)
- Quick start instructions
- Links to component docs
- NO deep architectural details

**Component READMEs** (top-level component dirs only):
- Component's purpose in detail
- Architectural decisions
- Component-specific setup
- Internal APIs/interfaces

Do **not** flag missing READMEs in nested subdirectories (e.g., `src/utils/`, `lib/internal/`). A nested directory that needs anything at all needs a CLAUDE.md, not a README.

### Content Quality

- **Concise**: No unnecessary words or redundancy
- **Clear**: Jargon explained, unambiguous
- **Accurate**: Examples work, paths exist, commands valid, lists reflect what actually exists in code
- **Complete**: Prerequisites listed, all steps included, enumerations cover all items (see Enumeration Completeness below)
- **Consistent**: Uniform terminology and formatting
- **Internally consistent**: Numbers, counts, and facts agree across all mentions within the same document

Content Quality applies primarily to README prose. For CLAUDE.md, focus instead on context cost — see CLAUDE.md Content Value and CLAUDE.md Size below — plus accuracy (references point to real files).

### Enumeration Completeness

When documentation contains a list of things — components, features, agents, services, environment variables, log files, database tables, API endpoints — verify the list is **complete and accurate** by cross-referencing the codebase.

**Workflow**:
1. Identify every enumeration in the docs (any list that claims to describe "all" or "the" items of a type)
2. Find the canonical source in code (e.g., config directory for agents, env template for env vars, schema definitions for DB tables)
3. Diff the two: items in code but missing from docs, items in docs but removed from code
4. Flag gaps as P3 (stale or incomplete)

**Common patterns to cross-reference**:
- Agent/service/component lists → config files or directory listings
- Feature/capability lists → tool, handler, or route files
- Environment variable documentation → `.env.example`, `.env.template`, or equivalent
- Log file lists → logging configuration entries
- Database table/schema documentation → schema definition files or migrations
- CLI command/Makefile target documentation → actual command definitions

**What to skip**: Exhaustive enumeration is not always expected. Use judgement — a "Key features" section need not list every minor capability. But a section titled "Agents" or "Configuration" that presents itself as comprehensive should be complete.

### Hidden Knowledge

Good docs surface things newcomers can't discover on their own. Flag when these are missing:

**Prerequisites & Environment**:
- System dependencies (e.g., "requires Docker 20+", "needs libssl-dev on Linux")
- Required tool versions (node, python, go, etc.)
- Required accounts or API keys (with signup links, not the keys themselves)
- Environment variables with example values (use `YOUR_API_KEY` placeholders)

**Platform Differences**:
- OS-specific instructions when behavior differs (Mac vs Linux vs Windows)
- Architecture notes if relevant (ARM vs x86)

**Gotchas & Failure Modes**:
- Common setup errors and fixes ("If you see X, run Y")
- Non-obvious side effects ("This command also resets the database")
- Known limitations or unsupported scenarios

**Magic Values**:
- Default ports, timeouts, limits that aren't obvious from code
- Config file locations that vary by platform
- Implicit ordering dependencies ("Run A before B")

### Quick Start Quality

The quick start should let someone succeed in minutes, not hours. Evaluate:

**Copy-Pasteable Commands**:
- Commands can be copied and run verbatim (no unexplained `<placeholders>`)
- If placeholders are needed, explain how to get the real value
- Shell-specific syntax noted if it matters (bash vs zsh vs fish)

**Expected Output**:
- Show what success looks like ("You should see: ...")
- Include sample output for key commands
- Note how to verify it worked

**Minimal Path**:
- Doesn't require reading other sections first
- Optional features clearly marked as optional
- "Hello world" possible before diving into configuration

**First-Run Troubleshooting**:
- Top 2-3 things that go wrong on first run
- One-liner fixes for each
- Link to more detailed troubleshooting if it exists

### CLAUDE.md Content Value

CLAUDE.md loads into context on every session that touches its directory and competes for attention with the task itself. Content that changes no behaviour is not neutral — it dilutes the content that does, and a bloated file causes the rules that matter to be ignored.

Apply this test to every line: **would removing it cause a mistake?** If not, it is a finding.

| Keep | Cut |
|------|-----|
| Commands that can't be guessed (build, test, lint invocations) | Anything derivable by reading the code |
| Conventions that differ from language or tool defaults | Standard conventions the model already follows |
| Invariants and prohibitions ("never write raw SQL in handlers") | File-by-file descriptions of the directory |
| Gotchas and non-obvious behaviour | Directory layouts, dependency lists, architecture overviews |
| Environment quirks (required env vars, service dependencies) | Long explanations, tutorials, or API reference material |
| Repository etiquette (branch naming, PR conventions) | Anything that changes whenever a file is added or renamed |

**Derivable content (P3)** — Flag rows and prose that restate what a filename, extension, or one-line read already conveys. A `README.md` row described as "Project overview", a `tests/` row described as "Test files", a `src/api/` row described as "API code": each costs context on every load and prevents nothing.

Naming a file is not the same as describing it. An entry earns its place when it resolves something a reader would otherwise get wrong — which of two similar directories is canonical, that a name is misleading, that a path is legacy. Enumeration for its own sake does not.

**Coverage** — A directory needs a CLAUDE.md when something about it would be got wrong without one. Do not flag a directory for lacking a CLAUDE.md; an absent file is not a defect, an empty-calorie file is. Never review generated dirs, vendored deps, stubs (`.gitkeep` only), `.git`, `node_modules`, `__pycache__`, `dist`, `build`, and similar. Also skip `CLAUDE.local.md` and `MEMORY.md` — these are personal/auto-managed and not part of shared project documentation.

### CLAUDE.md Size

Target **under 200 lines** per CLAUDE.md. Longer files consume more context and measurably reduce adherence.

- Over 200 lines — P3. Recommend a specific cut, not "shorten it": task-specific procedures move to a skill, path-specific conventions move to `.claude/rules/` with a `paths:` glob, reference material moves to a file read on demand.
- `@path` imports do **not** reduce context — imported files are expanded and loaded at launch. Splitting an oversized CLAUDE.md into imports is organisation, not a size fix; flag it as such if used that way (P4).

### CLAUDE.md Navigation Index

A tabular index (`File` or `Directory`, `What`, `When to read`) is **one valid form** for a CLAUDE.md, not a required one. It earns its place where a directory's structure is genuinely non-obvious and the "When to read" triggers carry routing information a reader could not guess.

Do **not** flag a CLAUDE.md for lacking a table. A file of commands, conventions, and gotchas with no table at all is a good CLAUDE.md, and is the shape most directories need.

> CLAUDE.md conventions inspired by solatis/claude-config (MIT).

Where a table is present, entries should use:
- Backtick-wrapped file/directory names (e.g., `` `src/` ``)
- Noun-based "What" descriptions that add information beyond the name
- Action-verb "When to read" triggers (e.g., "Adding a new route", "Debugging test failures")

**Index completeness is not a goal** — An index is a curated set of pointers, not a directory listing. Do not flag files that exist in the directory but are absent from the index; omission is usually the correct editorial choice. Flag the reverse: an index entry pointing at a file or directory that does not exist.

**Content separation** — Subdirectory CLAUDE.md files should carry that area's commands, conventions, and invariants. Extended architecture narrative, design rationale, and history belong in README.md. Flag multi-paragraph architectural prose (P3); do not flag a few lines stating where things live and what rule applies.

**Root CLAUDE.md operational sections** — The root CLAUDE.md may additionally contain operational sections (Build, Test, Lint commands), agent instruction sections (e.g., beads workflow, tool configuration), and other project-wide guidance. This is expected and should not be flagged.

### Docs Written Under Earlier Guidance

Earlier versions of this skill required a tabular index in every CLAUDE.md and treated any unindexed file as drift. Repositories that followed it — including ones this skill previously reviewed — will have exhaustive `File | What | When to read` tables whose rows largely restate filenames. That is now the anti-pattern, but it was the documented convention when those files were written.

Handle it as a migration, not a pile of defects:

- **Report it once.** When a CLAUDE.md's index is systematically derivable, emit **one** P3 finding for the file naming the pattern and the rows worth keeping. Do not emit a finding per row. Do not emit a finding per file when a whole repo shares the pattern — one finding per CLAUDE.md is the ceiling, and a repo-wide note in the summary is better still.
- **Say why it changed.** The fix text should note that the convention changed, so a maintainer doesn't read it as sloppiness. These files were correct when written.
- **Keep what survives.** Rows that disambiguate rather than enumerate should be preserved in the fix, not swept away with the rest. Name them explicitly.
- **Don't recommend deleting the file** merely because its table is derivable. The fix is to cut the table down to what earns its place, or replace it with the commands and gotchas that directory actually needs.

A review that turns a previously-conformant repository into thirty findings is a worse outcome than one that says "this index predates the current convention, here's the trimmed version."

### CLAUDE.md Progressive Disclosure

**Detail belongs where the code lives.** CLAUDE.md depth should match directory depth. The goal is to minimize context loading at each level — an AI working in `internal/cloud/aws/` should not need to read the entire root CLAUDE.md to understand that directory.

The hierarchy works as follows:
- **Root CLAUDE.md**: repo-wide commands and conventions. Just enough to work anywhere in the tree.
- **Branch CLAUDE.md** (e.g., `internal/cloud/`): the conventions and invariants that govern that area.
- **Leaf CLAUDE.md** (e.g., `internal/cloud/aws/`): implementation specifics that would otherwise be got wrong.

At every level the content is what a reader can't derive — not a description of what the level contains.

**Anti-pattern**: A root CLAUDE.md that contains multi-paragraph explanations of how `internal/cloud/` works, what patterns `internal/broker/` uses, or how `test/` is structured. That detail should live in the respective subdirectory CLAUDE.md files, not the root.

**What root CLAUDE.md should contain**:
- Project overview (1-2 sentences)
- Build/test/lint commands
- Repository-wide conventions that differ from defaults
- Agent workflow instructions (if applicable)
- Common gotchas (brief, not exhaustive)
- Orientation to major areas — only where the layout isn't already obvious from the tree

**What root CLAUDE.md should NOT contain**:
- Multi-paragraph architecture explanations for subdirectories
- Implementation patterns specific to a component
- Detailed testing approaches (belongs in `test/CLAUDE.md`)
- Provider-specific details (belongs in provider directories)
- An exhaustive listing of the repository's files and directories

**Broken references** — Flag when:
- An index entry references a file or directory that does not exist (P2)
- A CLAUDE.md index links to a README that doesn't exist (P2)

Do not flag a file that exists but is absent from the index, and do not flag a README that references a component with no CLAUDE.md — neither is a defect. See *Index completeness is not a goal* above.

**Misplaced detail** — Flag when:
- Root CLAUDE.md contains multi-paragraph explanations about subdirectory internals
- Architectural detail about a component appears in the root instead of that component's CLAUDE.md
- A branch directory lacks a CLAUDE.md but its detail is front-loaded in a parent CLAUDE.md

### CLAUDE.md Imports (`@path` Syntax)

CLAUDE.md files can import other files using `@path/to/file` syntax. The skill validates these references:

- Import target must exist (P2 if missing)
- Relative paths resolve relative to the containing file's directory
- Maximum depth of 4 hops for recursive imports (A imports B imports C...); flag deeper chains as P3
- Circular imports are P2
- Imports inside fenced code blocks and inline code spans are not evaluated — do not flag them
- Exclude email addresses (user@example.com), npm scopes (`@org/pkg`), and social handles (`@username`) from import checking

### `.claude/rules/` Directory

The `.claude/rules/` directory contains per-topic rule files that scope instructions to specific file paths. Do not flag an absent `.claude/rules/` directory — it is optional.

- Each file should cover one topic (P4 if unfocused)
- Filenames should be descriptive — flag generic names like `rules.md`, `misc.md` (P4)
- `paths` frontmatter: P2 for invalid glob syntax, P3 for globs matching no files, P4 if universal rules are unnecessarily scoped to specific paths
- Flag duplication between rules files and CLAUDE.md (P3)

### Memory Location Awareness

**`.claude/CLAUDE.md` alternative** — Both `./CLAUDE.md` and `./.claude/CLAUDE.md` are valid project instruction locations. Do not flag the choice of one over the other. Check both locations when reviewing.

**`CLAUDE.local.md` (personal, not reviewed)** — `CLAUDE.local.md` contains personal preferences and is gitignored. Do not flag it as missing. Do not review its content. Flag personal/local preferences leaked into shared CLAUDE.md as P3 (hardcoded user home paths like `/Users/<name>/`, `/home/<name>/`, `C:\Users\<name>\`; personal sandbox URLs; machine-specific values).

**`MEMORY.md` / auto memory (excluded)** — Files in `~/.claude/projects/<project>/memory/` including `MEMORY.md` are managed by Claude Code directly. The skill must:
- Not suggest changes to MEMORY.md
- Not flag it for any criteria
- Not recommend creating or restructuring it
- Skip it entirely during discovery

### Instruction Specificity

Instructions in CLAUDE.md and `.claude/rules/` should be specific and actionable. Flag vague directives as P4:
- "follow best practices"
- "write clean code"
- "use appropriate naming"
- "ensure quality"

Do not flag intentionally high-level guidance that communicates a genuine design preference (e.g., "prefer composition over inheritance", "use dependency injection for external services").

### Formatting Best Practices

Instructions should be structured for fast scanning:
- Instructions should use bullet points, not prose paragraphs (P4)
- Related bullets should be grouped under descriptive headings — flag 10+ ungrouped consecutive items (P4)

Applies to CLAUDE.md and `.claude/rules/` instruction content only. Does not apply to tabular indexes, README prose, or code block sections.

## Checklist

### Structure & Navigation
- [ ] Root README has: purpose, overview, quick start
- [ ] Root README links to component docs
- [ ] Each major component root has a README (not nested subdirs)
- [ ] Progressive disclosure maintained
- [ ] Subdirectory CLAUDE.md files carry commands and rules, not architecture narrative
- [ ] Root CLAUDE.md contains commands + conventions, not subdirectory architecture details
- [ ] Detail lives where the code lives (no front-loading in parent CLAUDE.md files)

### CLAUDE.md Content
- [ ] Every line would cause a mistake if removed
- [ ] Nothing derivable from filenames, code, or standard conventions
- [ ] No file-by-file descriptions or exhaustive directory layouts
- [ ] Each CLAUDE.md under 200 lines
- [ ] Index entries, where present, resolve to existing files
- [ ] Imports not used to work around length
- [ ] Indexes predating the current convention reported as one migration finding, not per row

### Content Quality
- [ ] No outdated references (paths, commands)
- [ ] Code examples syntactically correct
- [ ] Referenced files/commands exist
- [ ] Consistent terminology
- [ ] No duplicate information
- [ ] Numbers and facts consistent across all mentions within the same document

### Codebase Consistency
- [ ] Enumerations (agents, features, env vars, etc.) cross-referenced against code
- [ ] No undocumented components in sections that present themselves as comprehensive
- [ ] No documented items that no longer exist in code

### Memory Ecosystem
- [ ] `@path` imports resolve to existing files
- [ ] No import chains exceed 4 hops
- [ ] `.claude/rules/` files focused and descriptively named
- [ ] `paths` frontmatter valid and matching real files
- [ ] No duplication between rules files and CLAUDE.md
- [ ] No personal preferences leaked into shared CLAUDE.md
- [ ] MEMORY.md excluded from review scope
- [ ] Instructions specific and actionable
- [ ] Instructions use bullets grouped under headings

### Hidden Knowledge
- [ ] Prerequisites documented (system deps, tool versions)
- [ ] Required env vars listed with example values
- [ ] Platform-specific instructions where behavior differs
- [ ] Common errors and fixes documented
- [ ] Magic values (ports, timeouts, defaults) explained

### Quick Start
- [ ] Commands are copy-pasteable (no unexplained placeholders)
- [ ] Expected output shown for key steps
- [ ] Minimal path to first success (no detours)
- [ ] First-run troubleshooting for common failures

## Output

You MUST produce a report following the exact structure shown in [REFERENCE.md](REFERENCE.md).

**Severity guide**:
- **P1** — Security-relevant: docs omit auth steps, expose secrets in examples, or give dangerous command examples
- **P2** — Broken: code examples that error, paths/commands that don't exist, quick start fails on copy-paste, index entries pointing to missing files, broken `@path` imports, circular imports, invalid glob syntax in rules frontmatter
- **P3** — Stale, incomplete, or context-wasting: outdated references, missing prerequisites, missing env var docs, no expected output shown, misplaced detail (architecture in root instead of subdirectory CLAUDE.md), enumeration gaps (components/features/env vars in code but missing from docs), internal fact contradictions within the same document, leaked local preferences in shared CLAUDE.md, orphan globs in rules frontmatter, duplication between rules files and CLAUDE.md, import chains exceeding 4 hops, **derivable CLAUDE.md content** (file-by-file descriptions, directory layouts, restated conventions), **CLAUDE.md over 200 lines**
- **P4** — Polish: formatting inconsistencies, verbose wording, missing "When to read" triggers, missing platform-specific notes, vague instructions ("follow best practices"), generic rule filenames, ungrouped instruction lists, unnecessarily scoped universal rules, imports used to work around CLAUDE.md length

Each finding MUST include:

- **Priority** (P1/P2/P3/P4) in the H3 header
- **Location** (file:line, or just filename if no specific line)
- **Explanation** of the problem or missing content and why it matters
- **Fix** — concrete prescription of exactly what to change or add
- **Done when** — a verifiable criterion checkable by reading the file. For errors: "The link at README.md:45 resolves to an existing file." For missing content: "The Quick Start section lists the minimum required Docker version with a link to the install page." NOT: "The docs are accurate."
