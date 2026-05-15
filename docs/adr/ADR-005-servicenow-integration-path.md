# ADR-005: ServiceNow Integration Path

**Status:** Accepted
**Date:** 15-May-2026

## Decision

Custom connector + OAuth 2.0 Client Credentials grant.

## Why

- Preserves service-account design (ADR Day 5-6: trusted subsystem, two-call audit, three-layer auth)
- Standard connector tier — no Premium licensing
- PL-400 exam-relevant skill (15-20% of exam)

## Paths rejected

| Path | Killer reason |
|---|---|
| Microsoft pre-built ServiceNow connector | Premium-licensed + Authorization Code only (user-context) |
| Native ServiceNow MCP (Zurich) | Copilot Studio MCP client only supports Authorization Code, won't carry Client Credentials |
| Self-hosted MCP server (sweetgreen / echelon) | Same Copilot Studio limitation — same wall, different wallpaper |
| Entra ID Certificate auth | Niche, no real-world adoption, no exam relevance |

## Key fact locked

**Copilot Studio's MCP client supports OAuth 2.0 Authorization Code flow only.**
Service-account architectures (Client Credentials) require custom connector, not MCP.

## MCP not abandoned

- Standalone Week 3-4 portfolio piece
- Self-hosted MCP server + Claude Desktop or Cursor as client
- Same ServiceNow PDI, same service account
- Demonstrates protocol skill without compromising AgentOps architecture

## Consequences

- Custom connector work pulled from Week 3 to Day 7 onward
- Week 3 freed for Foundry RAG + custom eval harness as originally scoped
- MCP becomes secondary portfolio asset, not primary integration
