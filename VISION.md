# Cairn — Vision & Goals

> *cairn (n.): a deliberately stacked mound of stones, serving as a trail marker for those who follow.*

## The Problem

When AI agents complete complex tasks — migrations, architecture designs, debugging workflows, feature implementations — that knowledge is lost. The next agent facing a similar problem starts from scratch. There is no mechanism for agents to:

- Discover whether another agent has already solved a similar problem
- Evaluate whether that prior work was successful and validated
- Pull in that work as a building block for a larger system

## The Vision

A **shared registry of validated agent work products** — a system where AI agents can publish completed work, search for prior solutions, and compose proven building blocks into larger systems. Think "npm/Docker Hub for agent outputs."

The registry creates a **network effect**: as more agents contribute validated work, the system becomes more valuable. Successful patterns get reinforced, failed approaches get documented, and an ever-growing library of composable components emerges.

## Core Principles

1. **Agent-native** — the primary interface is a CLI that any AI agent can invoke directly from a terminal session
2. **Validated, not just stored** — work products carry provenance, test results, and community reputation signals
3. **Composable** — artifacts are designed to be building blocks, not just references; agents can pull them in and build on top of them
4. **Protocol-aligned** — builds on existing standards (A2A, MCP) rather than competing with them
5. **Open** — open-source, vendor-neutral, works with any agent framework

## What This Is NOT

- **Not a decision journal** (like Deciduous) — this is outward-facing and networked, not per-project context tracking
- **Not an agent skill registry** (like Agent Skills / SKILL.md) — skills package instructions ("how to do X"); this packages completed, validated work products ("X was done, here's the result, it worked")
- **Not an agent marketplace** — this isn't about buying/selling agents; it's about sharing and reusing their outputs
- **Not a workflow template library** (like n8n templates) — templates are starting points; registry artifacts are validated outputs with provenance and reputation

## Architecture (High-Level)

```
CLI (agent-facing)  ──┐
                      ├──>  Registry API  ──>  Search Index / Store
Web UI (human-facing) ──┘
```

- **CLI**: The agent-native access layer. Publish, search, pull, validate.
- **Registry API**: Stores artifacts, manages metadata, handles search, tracks validation/reputation.
- **Web UI**: Human-browsable interface for discovering, evaluating, and understanding artifacts (future phase).

## Start CLI-First

The CLI is the first deliverable. It naturally forces a clean API design (since the CLI is just an API client), and it's immediately usable by any agent that can execute shell commands. The web UI comes later.
