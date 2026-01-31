# Cairn — Landscape Research

Research conducted January 2026.

## Existing Protocol Foundations

### Google A2A (Agent-to-Agent Protocol)
- Announced April 2025, now at v0.3 (July 2025)
- 150+ partner organizations, under the Linux Foundation
- Agents publish **Agent Cards** (JSON at `/.well-known/agent.json`) listing capabilities
- Defines **Artifacts** as immutable results from agent tasks — but these are ephemeral and scoped to a single interaction, not published to a shared registry
- Discovery mechanisms: well-known URI, curated registries, direct config
- **The registry problem is explicitly unsolved** — the spec leaves it open, and the community is actively debating centralized vs. decentralized (see: https://github.com/a2aproject/A2A/discussions/741)
- Key links:
  - https://developers.googleblog.com/en/a2a-a-new-era-of-agent-interoperability/
  - https://a2a-protocol.org/latest/topics/agent-discovery/

### Anthropic MCP (Model Context Protocol)
- Introduced November 2024, standardizes agent-to-tool connections
- 10,000+ active public MCP servers, 97M+ monthly SDK downloads
- Adopted by ChatGPT, Claude, Cursor, Gemini, Copilot, VS Code, and many others
- Official MCP Registry launched September 2025: https://registry.modelcontextprotocol.io/
- GitHub also launched its own MCP Registry: https://github.blog/changelog/2025-09-16-github-mcp-registry-the-fastest-way-to-discover-ai-tools/
- 17+ community registries exist
- MCP is about connecting agents to tools, not sharing agent outputs

### AAIF (Agentic AI Foundation)
- Formed December 2025 under the Linux Foundation
- Co-founded by Anthropic, Block, and OpenAI
- Platinum support from AWS, Bloomberg, Cloudflare, Google, Microsoft
- Founding projects: MCP (Anthropic), goose (Block), AGENTS.md (OpenAI)
- Ambition: "W3C for AI agents"
- Key links:
  - https://www.linuxfoundation.org/press/linux-foundation-announces-the-formation-of-the-agentic-ai-foundation
  - https://openai.com/index/agentic-ai-foundation/

### Other Protocols
- **ANP (Agent Network Protocol)**: Decentralized agent networks using DID (Decentralized Identity)
- **AITP**: Secure economic transactions between agents across trust boundaries
- **Agora**: LLM-based agents negotiate communication autonomously

## Existing Packaging & Reuse Systems

### Agent Skills
- Open standard originated with Claude Code, now cross-platform
- A Skill = folder with `SKILL.md` (YAML frontmatter + Markdown instructions)
- Supported by: Claude Code, GitHub Copilot, Cursor, OpenAI Codex, Gemini CLI, VS Code, Kiro, 11+ others
- CLI tools: `npx add-skill`, `npx skills i`, Tessl CLI
- Vercel released skills packaging 10 years of React/Next.js knowledge
- **Key distinction**: Skills package *instructions* (how to do something), not *completed validated work*
- Key links:
  - https://laurentkempe.com/2026/01/27/Agent-Skills-From-Claude-to-Open-Standard/
  - https://github.com/skillmatic-ai/awesome-agent-skills

### Workflow Template Marketplaces
- **n8n**: 5,288+ community AI workflow templates, open-source
- **Gumloop**: Template marketplace with pre-built AI workflows
- **AI Agent Store** (https://aiagentstore.ai/): Directory for discovering pre-built agents

## Related Projects

### Deciduous (https://github.com/notactuallytreyanastasio/deciduous)
- CLI tool (Rust/SQLite) that creates persistent, queryable graphs of development decisions
- Tracks goals, decisions, options, actions, outcomes as a graph
- Node types: Goals, Decisions, Options, Actions, Outcomes, Observations, Revisit nodes
- Multi-assistant support (Claude Code, OpenCode, Windsurf)
- Web viewer + Terminal UI
- Multi-user sync via patches
- **Key distinction from our project**: Deciduous is inward-facing and per-project (tracks "what did I decide and why?"). Our project is outward-facing and networked (tracks "what has anyone successfully built that others can leverage?")
- Deciduous's graph schema (goals -> decisions -> options -> actions -> outcomes) is useful prior art for representing work product provenance

## Composable Agent Frameworks

| Framework | Reusability Approach | Composability Model |
|-----------|---------------------|-------------------|
| LangChain / LangGraph | Modular chains, graph-based state machines | Component-level |
| CrewAI | Role-based agent teams, YAML config | Team-level |
| AutoGen (Microsoft) | Conversation-centric multi-agent | Conversation-level |
| OpenAI Agents SDK | Lightweight multi-agent with guardrails | Pipeline-level |
| Mastra | TypeScript-native, full-stack | Full-stack TS |
| MetaGPT | Simulates product teams | Role-simulation |
| Shakudo | Wraps multiple frameworks, low-code canvas | Meta-framework |

2026 trend: frameworks building for open protocols (MCP, A2A) are winning over proprietary integrations.

## Validation & Reputation (Nascent)

- **Agent evaluation suites**: Deterministic graders, model-based judges, human evaluators — used during development, not as marketplace reputation
- **Trusta.AI**: Web3 project building "universal credit layer for humans and AI agents" — early-stage, identity-focused
- **Credo AI**: Enterprise AI governance with Model Trust Scores
- **A2A Agent Card Signatures**: v0.3 introduced digitally signed Agent Cards (JWS/RFC 7515)
- **No widely adopted marketplace-level reputation system exists** for scoring agent outputs before reuse

## Identified Gaps (Our Opportunity)

1. **No registry for completed work products** — Skills package instructions, A2A artifacts are ephemeral. Nobody publishes validated outputs for reuse.
2. **No cross-agent knowledge reuse** — successful complex work is lost when an agent session ends.
3. **No reputation/quality system for agent outputs** — no marketplace-grade scoring of artifact reliability.
4. **No semantic search over agent-generated plans** — can discover agents by capability, but not specific completed work by similarity.
5. **No standardized artifact schema** — A2A defines artifacts loosely; no structured format for composable, searchable, reusable work products.
6. **The A2A registry problem remains unsolved** — centralized vs. decentralized debate is open.
