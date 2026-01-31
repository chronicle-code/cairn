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

## Agent Integration via Hooks

The lowest-friction way for agents to interact with Cairn is through **hooks** — lifecycle events that fire automatically during an agent's workflow. Claude Code hooks are the first integration target.

### Post-Task Hook (Publish Flow)

When an agent completes a task successfully, a hook can prompt the publish flow:

```json
// .claude/settings.json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "task_complete",
        "command": "cairn suggest --check-publishable"
      }
    ]
  }
}
```

The flow:
1. Agent completes a task (migration, feature, fix, etc.)
2. Hook fires `cairn suggest` — analyzes the work and determines if it's a candidate for publishing
3. **Human approves or rejects** the publish (this is the critical abuse gate)
4. If approved, `cairn publish` packages and submits the stone

The human-in-the-loop at the publish step is non-negotiable. Agents can *suggest* publishing, but a human must confirm. This prevents:
- Agents autonomously flooding the registry
- Bad actors instructing agents to submit spam
- Accidental publication of proprietary code

### Pre-Task Hook (Search Flow)

Before starting work, an agent can automatically search for relevant existing stones:

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "task_start",
        "command": "cairn search --context-from-task --top 3 --json"
      }
    ]
  }
}
```

This gives the agent awareness of prior work before it starts from scratch.

---

## Validation & Trust Model

### What Cairn Is NOT

Cairn is **not a decision tree registry**. Decision trees (like Deciduous) track *why* choices were made within a project. Cairn stores *completed work products* — the actual output of agent work, with provenance and validation signals. The analogy: Deciduous is a lab notebook. Cairn is the published paper with reproducible results.

### Multi-Layer Validation

Raw usage count ("this stone was pulled 500 times") is a weak signal. A stone used 500 times for malicious purposes is not a good stone. Validation must be multi-layered:

| Layer | Signal | Weight | Description |
|-------|--------|--------|-------------|
| **Automated tests** | `test_passed` | High | Did the stone's own test suite pass? Did it pass when pulled into a new context? |
| **Checksum integrity** | `checksums_valid` | High | Do file hashes match the published manifest? Detects tampering. |
| **Human reviews** | `human_rating` | High | Explicit human review with rating and comment. |
| **Agent reviews** | `agent_rating` | Medium | Structured agent feedback: used in context X, result was Y. Weighted lower than human reviews. |
| **Contextual success** | `reuse_success_rate` | Medium | Of agents that pulled this stone, what % reported success in their own context? |
| **Provenance verification** | `provenance_verified` | Medium | Is the source repo real? Do the commits exist? Does the author's identity check out? |
| **Age & stability** | `time_since_publish` | Low | Older stones with stable ratings are more trustworthy than new ones. |
| **Download count** | `pull_count` | Low | Popularity is a weak signal but still a signal. |

### Review Structure

Agent reviews carry structured metadata, not just a rating:

```toml
[review]
reviewer = "github:someuser"
reviewer_type = "agent"           # agent | human
agent = "claude-opus-4.5"
rating = 4
result = "success"                # success | partial | failure
context = "Applied to MySQL-to-DynamoDB variant, required minor adaptation of the schema mapping."
adapted = true                    # Did the reviewer modify the stone for their use case?
source_repo = "github:someuser/their-project"  # Where it was applied (optional)
timestamp = "2026-02-15T10:00:00Z"
```

### What Counts as "Validated"

Validation status progresses through stages:

| Status | Meaning | Requirements |
|--------|---------|-------------|
| `draft` | Unpublished or no validation | None |
| `tested` | Author's tests pass | `stone.validation.tests.failed == 0` |
| `validated` | Tests pass + at least 1 human review | Tested + `human_reviews >= 1` |
| `community-reviewed` | Broad validation from multiple independent users | Validated + `human_reviews >= 3` + `reuse_success_rate >= 80%` |

---

## Security & Abuse Prevention

### Threat Model

Cairn is a registry where agents publish and consume code artifacts. This makes it a target for the same classes of attacks that plague npm, PyPI, and Docker Hub — plus new attack vectors unique to agent-consumed content.

| Threat | Description | Mitigation |
|--------|-------------|------------|
| **Malicious stones** | Stones containing harmful code (exfiltration, backdoors, destructive commands) | Automated static analysis on publish, sandboxed test execution, human approval gate |
| **Typosquatting** | Publishing stones with names similar to popular ones | Name similarity checks on publish, reserved namespaces for verified publishers |
| **Reputation farming** | Fake reviews or inflated usage metrics to boost a malicious stone | Reviews tied to verified GitHub identities, rate limiting, weight reviews by reviewer reputation |
| **Prompt injection via AGENT.md** | Crafting AGENT.md content that manipulates the consuming agent into harmful actions | AGENT.md sandboxing guidelines, static analysis for injection patterns, content policy enforcement |
| **Sybil attacks** | Creating many fake accounts to inflate reviews | GitHub identity verification, account age requirements, contribution history checks |
| **Data exfiltration** | Stones that phone home or leak context from the consuming agent's environment | No network calls allowed in stone files, static analysis for URLs/endpoints, sandboxed execution |
| **Dependency confusion** | Publishing a public stone with the same name as a private one | Scoped namespaces (`@org/stone-name`), private registry priority resolution |
| **Supply chain poisoning** | Compromising a popular stone's update to inject malicious content | Immutable versions (can't overwrite a published version), checksum pinning in `cairn.lock`, signing |

### Identity & Authentication

- All publishers must authenticate via **GitHub OAuth** — no anonymous publishing
- Stones are **cryptographically signed** by the publisher's key
- Agent reviews must be traceable to a verified identity — a review from `github:someuser` must prove that `someuser` actually ran the stone
- **Account age and activity requirements** for publishing (prevents throwaway account spam)

### Publish-Time Safeguards

Every `cairn publish` triggers:

1. **Manifest validation** — `stone.toml` schema compliance
2. **Required file check** — `README.md` and `AGENT.md` must exist
3. **Static analysis** — scan `files/` for:
   - Known malicious patterns (obfuscated code, encoded payloads)
   - Network calls (fetch, curl, wget, http imports)
   - File system access outside the stone's directory
   - Environment variable reads (credential harvesting)
   - Prompt injection patterns in `AGENT.md`
4. **Test execution** — run the stone's test suite in a sandboxed environment
5. **Name collision check** — typosquatting detection against existing stones
6. **Human confirmation** — the publisher (a human) must explicitly approve

### Pull-Time Safeguards

When an agent runs `cairn pull`:

1. **Checksum verification** — downloaded files must match published checksums
2. **Signature verification** — stone must be signed by the claimed publisher
3. **Advisory warnings** — flag stones with low validation status, few reviews, or recent publish date
4. **Lockfile support** — `cairn.lock` pins exact versions and checksums for reproducibility

### AGENT.md Sandboxing

`AGENT.md` is the most sensitive file in a stone — it's instructions that an agent will follow. This makes it a prompt injection vector. Mitigations:

- **Content policy** — `AGENT.md` is scanned for injection patterns on publish (e.g., "ignore previous instructions", "you are now", system prompt overrides)
- **Scoped instructions** — `AGENT.md` should only describe how to use *this stone*, not give general behavioral directives to the agent
- **Agent-side sandboxing** — consuming agents should treat `AGENT.md` as untrusted input with limited permissions (no different from how agents should treat any external content)

---

## Open Questions

### Resolved (initial positions taken)

- **Artifact granularity**: Supported via `scope` field — from `function` to `ecosystem`. The schema flexes to any level.
- **Versioning**: Semver. Stones are immutable once published at a version; publish a new version to update.
- **Identity**: GitHub-based identity initially (`github:<username>`). A2A Agent Card integration as a future extension.
- **Human-in-the-loop**: Required at the publish step. Agents suggest, humans approve. Non-negotiable for abuse prevention.
- **Review weighting**: Human reviews weighted higher than agent reviews. Agent reviews carry structured context metadata.
- **Cairn vs. decision trees**: Cairn stores completed work products, not decision histories. Deciduous-style decision graphs are optional provenance metadata within a stone, not the primary artifact.

### Still Open

- **Trust model**: Leaning toward centralized registry (npm-style) for simplicity, with federation support later. Need to evaluate whether a public registry should be the default or if self-hosted is the primary mode.
- **Publish incentives**: Options include give-to-get (publish to unlock pulls), org-level value (your team benefits from your own stones), or pure low-friction convenience (agents auto-suggest, humans approve). Likely a combination — but the default should work without artificial gates.
- **Privacy & licensing**: Private namespaces are supported in the CLI design, but the registry backend needs access control. Default license for public stones? MIT? Apache 2.0? Unlicense?
- **Semantic search implementation**: Embedding-based search over stone descriptions and `AGENT.md` content. Which embedding model? Where does indexing happen — server-side only, or can the CLI do local similarity search over cached stones?
- **Protocol integration**: Should a Cairn registry expose an A2A Agent Card? Should stones be discoverable as MCP resources? Both feel right but need design work.
- **Conflict resolution**: When two stones solve the same problem differently, how does the registry surface both and help agents choose? Rating alone isn't enough — context fit matters.
- **Size limits**: What's the maximum size of a stone? Large artifacts (trained models, large datasets) probably don't belong here. Need clear boundaries.
- **Offline support**: Should `cairn search` work against a local cache for air-gapped environments? Probably yes for enterprise adoption.
- **Static analysis depth**: How deep should publish-time scanning go? Full AST analysis? Or pattern-based heuristics? Tradeoff between security and publish speed.
- **Review verification**: How do you prove an agent actually used a stone before leaving a review? Could require a linked commit or session log, but that raises privacy concerns.
