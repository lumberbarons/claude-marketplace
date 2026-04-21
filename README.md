# 🪵🛒 Lumber Mart

A plugin marketplace for Claude Code that lets you discover and install plugins for enhanced development workflows.

Learn more about Claude Code marketplaces in the [official documentation](https://code.claude.com/docs/en/plugin-marketplaces).

## Prerequisites

- [Claude Code](https://docs.claude.com/en/docs/claude-code) with plugin marketplace support
- `git` (only required to add a local marketplace from a clone)

## Available Plugins

| Plugin | Category | Description |
|--------|----------|-------------|
| [circuits](plugins/circuits/) | Hardware | Skills and agents for microcontroller and hardware projects -- timing diagrams, PCB design, and datasheet extraction |
| [specbeads](plugins/specbeads/) | Workflow | Specification-driven development with spec-kit and beads -- task tracking and spec conformance |
| [runbooks](plugins/runbooks/) | Workflow | Operational runbook creation with consistent structure and built-in maintenance feedback loops |
| [critique](plugins/critique/) | Workflow | Review skills for code, tests, documentation, and observability -- design, coverage, doc-structure, and logging issues that linters miss |
| [decisions](plugins/decisions/) | Workflow | Architectural Decision Record (ADR) authoring in MADR format with relationship tracking and CLAUDE.md registration |

### [circuits](plugins/circuits/)

Skills and agents to help with microcontroller and other hardware projects.

| Component | Type | Description |
|-----------|------|-------------|
| `circuits:wavejson` | Skill | Generate WaveJSON timing diagrams and HTML viewers using WaveDrom |
| `circuits:circuit-synth` | Skill | Design PCB circuits in Python with KiCad output and component search |
| `circuits:datasheet` | Skill | Extract structured information from IC/component datasheets into markdown |
| `circuits:datasheet-agent` | Agent | Autonomous agent for datasheet reading and extraction |

### [specbeads](plugins/specbeads/)

Integrates [spec-kit](https://github.com/spec-kit/specify) with [beads](https://github.com/beads-project/beads) for streamlined specification-driven development. Pairs with the [critique](plugins/critique/) plugin -- pipe its review output through `/raise-beads` to file findings as trackable beads.

| Component | Type | Description |
|-----------|------|-------------|
| `specbeads:init` | Command | Initialize a repository with spec-kit and beads |
| `specbeads:beadify` | Command | Convert tasks.md into beads (epics for phases, tasks as children) |
| `specbeads:implement` | Command | Implement a spec-kit feature phase, one task at a time with per-task commits |
| `specbeads:fix` | Command | Implement standalone bug/task beads (e.g. from review findings) |
| `specbeads:raise-beads` | Command | File review findings from conversation context as beads, with deduplication |
| `specbeads:review-spec` | Command | Validate implementation against spec-kit artifacts |

### [runbooks](plugins/runbooks/)

Operational runbook creation with consistent structure and built-in maintenance feedback loops.

| Component | Type | Description |
|-----------|------|-------------|
| `runbooks:create-runbook` | Skill | Create a new operational runbook from a standard template |

### [critique](plugins/critique/)

Review skills that surface design, coverage, doc-structure, and logging issues that linters and static analysis miss. Each skill produces a structured findings report with P1/P2/P3 (and P4 for docs) severities, file:line locations, and concrete fixes.

| Component | Type | Description |
|-----------|------|-------------|
| `critique:review-code` | Skill | Review code for design issues -- single responsibility, abstraction levels, testability, naming |
| `critique:review-tests` | Skill | Review tests for completeness, coverage gaps, output validation, isolation, readability |
| `critique:review-docs` | Skill | Review README and CLAUDE.md files for progressive disclosure, enumeration completeness, index drift |
| `critique:review-o11y` | Skill | Review observability -- log levels, log value, missing logs at I/O boundaries, error-message quality |

### [decisions](plugins/decisions/)

Architectural Decision Record (ADR) authoring, maintenance, and enforcement. Produces MADR-format ADRs as standalone, append-only policy with ADR-to-ADR relationship tracking (`supersedes`, `superseded-by`, `related`). Specs cite the ADRs that constrained them; ADRs do not track their downstream consumers. Includes a review skill that audits a code change against the accepted ADRs and flags violations, erosions, drift, and driver-shift candidates.

| Component | Type | Description |
|-----------|------|-------------|
| `decisions:create-adr` | Skill | Create a new ADR from a MADR template, research context from the codebase, and register it in the project's CLAUDE.md |
| `decisions:review-policy` | Skill | Review a code change (working diff, PR, or path) against the accepted ADRs, flagging violations, invariant erosions, drift toward rejected options, and driver-shift candidates for ADR revisit |

## Usage

### Adding This Marketplace

```bash
# If hosted on GitHub
/plugin marketplace add lumberbarons/lumber-mart

# For local testing
/plugin marketplace add /path/to/lumber-mart
```

### Installing Plugins

```bash
# List available plugins
/plugin list

# Install a plugin
/plugin install circuits@lumber-mart
```

## License

MIT
