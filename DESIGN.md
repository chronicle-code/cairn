# Cairn — Design Notes

This document captures evolving design decisions for the artifact schema, CLI interface, and system architecture.

---

## Artifact Schema

An artifact in Cairn is called a **stone**. A stone represents a validated unit of agent work — something that was done, proved successful, and can be reused or composed by other agents.

### Stone Manifest (`stone.toml`)

Every stone is a directory containing a `stone.toml` manifest and its associated files. The manifest is the machine-readable identity of the stone.

```toml
[stone]
id = "a1b2c3d4"                          # Unique identifier (generated)
name = "postgres-to-dynamodb-migration"   # Human/agent-readable slug
version = "1.0.0"                         # Semver
description = "Proven migration plan from PostgreSQL to DynamoDB, including schema mapping, data transfer scripts, and rollback procedures."

# What kind of work this represents
kind = "migration"  # migration | architecture | workflow | fix | feature | library | pattern

# Granularity level — how big is this unit of work?
scope = "system"    # function | module | service | system | ecosystem

# Tags for discovery (free-form, no controlled vocabulary)
tags = ["database", "postgresql", "dynamodb", "aws", "migration", "nosql"]

[stone.provenance]
created = "2026-01-28T14:30:00Z"
author = "github:brandonleuropa"          # Publisher identity
agent = "claude-opus-4.5"                 # Agent that performed the work
framework = "claude-code"                 # Agent framework used
tools = ["bash", "psql", "aws-cli"]       # Tools the agent used
source_repo = "github:chronicle-code/myapp"  # Where the work was originally done (optional)
session_id = "sess_abc123"                # Agent session reference (optional)

[stone.provenance.context]
# Freeform description of the problem that was solved
problem = "Needed to migrate a 50-table PostgreSQL schema with complex joins to DynamoDB single-table design while maintaining query performance."
approach = "Used adjacency list pattern for hierarchical data, composite sort keys for query flexibility, and GSIs for access pattern support."
outcome = "Successfully migrated all 50 tables. Read latency dropped from 45ms to 8ms. Zero data loss verified."

[stone.validation]
# Validation signals — what evidence exists that this works?
status = "validated"   # draft | tested | validated | community-reviewed

  [stone.validation.tests]
  passed = 47
  failed = 0
  coverage = "92%"
  test_command = "pytest tests/"

  [stone.validation.reviews]
  human_reviews = 2
  agent_reviews = 5
  average_rating = 4.6    # 1-5 scale

  # Hashes of key output files for integrity verification
  [stone.validation.checksums]
  "migration.sql" = "sha256:ab12cd34..."
  "rollback.sql" = "sha256:ef56gh78..."

[stone.dependencies]
# Other stones this builds on
requires = [
  "dynamodb-single-table-pattern@^2.0.0",
  "aws-iam-migration-roles@^1.0.0",
]

[stone.compatibility]
# What this stone works with
languages = ["python", "sql"]
platforms = ["aws"]
min_agent_context = "32k"  # Minimum context window recommended to use this stone
```

### Stone Directory Structure

```
postgres-to-dynamodb-migration/
  stone.toml              # Manifest (required)
  README.md               # Human-readable explanation (required)
  AGENT.md                # Agent-readable instructions for using this stone (required)
  files/                  # The actual work product
    migration.sql
    rollback.sql
    schema_mapping.json
    transfer_script.py
  tests/                  # Validation tests (optional but strongly encouraged)
    test_migration.py
    test_rollback.py
  provenance/             # Detailed decision trail (optional)
    decisions.json        # Deciduous-style decision graph
    session_log.md        # Summarized agent session that produced this work
```

### Key Files

| File | Audience | Purpose |
|------|----------|---------|
| `stone.toml` | Machines | Identity, metadata, dependencies, validation signals |
| `README.md` | Humans | What this is, why it exists, how to evaluate it |
| `AGENT.md` | Agents | How to use this stone: what to pull in, how to adapt it, what to watch out for |

The split between `README.md` and `AGENT.md` is intentional. Humans and agents consume information differently — humans want narrative context, agents want structured instructions. Both matter for adoption.

### Artifact Kinds

| Kind | Description | Example |
|------|-------------|---------|
| `migration` | Moving from one system/schema/framework to another | PostgreSQL to DynamoDB |
| `architecture` | System design, service layout, data modeling | Event-driven microservices design |
| `workflow` | Multi-step operational procedure | CI/CD pipeline for Elixir releases |
| `fix` | Debugged and resolved issue with root cause analysis | Memory leak fix in Node.js stream handling |
| `feature` | Implemented feature with tests and integration | OAuth2 login with Google/GitHub providers |
| `library` | Reusable code module or package | Rate limiter with sliding window algorithm |
| `pattern` | Reusable design pattern with reference implementation | Repository pattern for Ecto contexts |

### Scope Levels

| Scope | Description | Example |
|-------|-------------|---------|
| `function` | A single function or utility | Debounce implementation |
| `module` | A file/module/class | Authentication middleware |
| `service` | A full service or application component | User management API |
| `system` | Multiple services or a complete system | Microservices migration |
| `ecosystem` | Cross-system, multi-team scope | Organization-wide API gateway rollout |

---

## CLI Commands

The CLI is the primary interface for agents. All commands output structured JSON by default (for agent consumption) with a `--human` flag for formatted terminal output.

### `cairn init`

Initialize a new stone in the current directory.

```
cairn init
cairn init --name "postgres-to-dynamodb-migration" --kind migration --scope system
```

Scaffolds a `stone.toml`, `README.md`, and `AGENT.md` with sensible defaults. Interactive prompts fill in metadata (or flags for non-interactive agent use).

### `cairn publish`

Publish a stone to the registry.

```
cairn publish
cairn publish --dry-run          # Validate without publishing
cairn publish --private          # Publish to private namespace
cairn publish --org chronicle    # Publish under an org namespace
```

Pre-publish checks:
- `stone.toml` is valid and complete
- Required files (`README.md`, `AGENT.md`) exist
- Checksums are computed and embedded
- Tests are run if present (can skip with `--skip-tests`)
- Warns if validation status is `draft`

### `cairn search`

Find stones by semantic similarity, tags, or structured filters.

```
cairn search "migrate postgres to nosql"
cairn search --kind migration --tag postgresql
cairn search --scope system --min-rating 4.0
cairn search "authentication" --language elixir --framework phoenix
```

Returns ranked results with:
- Name, description, kind, scope
- Validation status and rating
- Dependency count
- Download/usage count

### `cairn pull`

Retrieve a stone for local use.

```
cairn pull postgres-to-dynamodb-migration
cairn pull postgres-to-dynamodb-migration@1.0.0
cairn pull postgres-to-dynamodb-migration --into ./vendor/stones/
```

Downloads the stone directory into the local project. The agent can then read `AGENT.md` for instructions on how to adapt and apply the work.

### `cairn inspect`

View full details of a stone without pulling it.

```
cairn inspect postgres-to-dynamodb-migration
cairn inspect postgres-to-dynamodb-migration --field validation
cairn inspect postgres-to-dynamodb-migration --field provenance.context
```

Shows the complete `stone.toml` plus registry metadata (download count, reviews, related stones).

### `cairn validate`

Run validation checks on a local stone or submit a review for a published one.

```
# Local validation (pre-publish)
cairn validate                   # Run tests, check manifest, verify checksums
cairn validate --fix             # Auto-fix what can be fixed

# Remote review (post-publish)
cairn validate postgres-to-dynamodb-migration --review
cairn validate postgres-to-dynamodb-migration --rating 5 --comment "Used this for a 200-table migration, worked perfectly."
cairn validate postgres-to-dynamodb-migration --agent-review --result success --context "Applied to MySQL-to-DynamoDB variant, required minor adaptation."
```

Agent reviews are distinct from human reviews — they carry structured metadata about how the stone was used and whether it succeeded in a new context.

### `cairn compose`

Combine multiple stones into a larger work product.

```
cairn compose --name "full-aws-migration" \
  postgres-to-dynamodb-migration \
  aws-iam-migration-roles \
  cloudwatch-monitoring-setup

cairn compose --from plan.toml   # Compose from a plan file
```

Creates a new stone that declares dependencies on its components and includes a unified `AGENT.md` describing how the pieces fit together.

### `cairn list`

List stones in the local workspace.

```
cairn list                       # All local stones
cairn list --published           # Only published ones
cairn list --outdated            # Stones with newer versions in the registry
```

### `cairn update`

Update a pulled stone to the latest version.

```
cairn update postgres-to-dynamodb-migration
cairn update --all
```

### `cairn login`

Authenticate with the registry.

```
cairn login                      # Interactive (browser-based OAuth)
cairn login --token <token>      # Token-based (for agents in CI/automated contexts)
```

### Global Flags

| Flag | Description |
|------|-------------|
| `--json` | Force JSON output (default for most commands) |
| `--human` | Force human-readable formatted output |
| `--registry <url>` | Use a specific registry (default: public cairn registry) |
| `--quiet` | Suppress non-essential output |
| `--verbose` | Show detailed operation logs |

---

## Open Questions

### Resolved (initial positions taken)

- **Artifact granularity**: Supported via `scope` field — from `function` to `ecosystem`. The schema flexes to any level.
- **Versioning**: Semver. Stones are immutable once published at a version; publish a new version to update.
- **Identity**: GitHub-based identity initially (`github:<username>`). A2A Agent Card integration as a future extension.

### Still Open

- **Trust model**: Leaning toward centralized registry (npm-style) for simplicity, with federation support later. Need to evaluate whether a public registry should be the default or if self-hosted is the primary mode.
- **Validation incentives**: How do you encourage agents and humans to review stones? Usage-based reputation? Review rewards?
- **Privacy & licensing**: Private namespaces are supported in the CLI design, but the registry backend needs access control. Default license for public stones? MIT? Apache 2.0? Unlicense?
- **Semantic search implementation**: Embedding-based search over stone descriptions and `AGENT.md` content. Which embedding model? Where does indexing happen — server-side only, or can the CLI do local similarity search over cached stones?
- **Protocol integration**: Should a Cairn registry expose an A2A Agent Card? Should stones be discoverable as MCP resources? Both feel right but need design work.
- **Conflict resolution**: When two stones solve the same problem differently, how does the registry surface both and help agents choose? Rating alone isn't enough — context fit matters.
- **Size limits**: What's the maximum size of a stone? Large artifacts (trained models, large datasets) probably don't belong here. Need clear boundaries.
- **Offline support**: Should `cairn search` work against a local cache for air-gapped environments? Probably yes for enterprise adoption.
