# Cairn — Design Notes

This document captures evolving design decisions for the artifact schema, CLI interface, and system architecture.

## Artifact Schema

> TODO: Define what a "work product" looks like. Key considerations:
> - Must be structured enough to be composable
> - Must be flexible enough to cover diverse agent outputs (migrations, architectures, debugging workflows, feature implementations, etc.)
> - Should capture provenance (how it was built, what agent, what tools)
> - Should capture validation signals (tests passed, human review, community ratings)
> - Deciduous's graph model (goals -> decisions -> options -> actions -> outcomes) is useful prior art for the provenance chain

## CLI Commands

> TODO: Sketch out the command interface. Expected patterns:
> - `publish` — submit a validated work product to the registry
> - `search` — find relevant prior work by semantic similarity
> - `pull` — retrieve an artifact for local use
> - `validate` — run validation checks / submit a review
> - `inspect` — view full details of an artifact
> - `compose` — combine multiple artifacts into a larger system

## Open Questions

- **Artifact granularity**: What's the right unit of work? A single function? A migration plan? An entire system design? Probably need to support multiple levels.
- **Validation model**: Automated tests? Human review? Peer-agent verification? Community voting? Likely a combination.
- **Identity**: How are agents and publishers identified? Tie into existing identity systems (GitHub, A2A Agent Cards)?
- **Versioning**: How do artifacts evolve? Semantic versioning? Immutable snapshots with lineage?
- **Trust model**: Centralized registry (npm-style) vs. federated (git-style) vs. decentralized (blockchain-style)?
- **Privacy**: Some work products may be proprietary. Support private registries alongside public?
- **Protocol integration**: How to align with A2A Artifacts and MCP? Could this registry be discoverable via A2A Agent Cards?
- **Licensing**: How are shared artifacts licensed? Default to open? Per-artifact licensing?
