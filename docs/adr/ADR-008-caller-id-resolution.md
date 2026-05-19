# ADR-008 — caller_id Resolution Architecture

**Status:** Accepted
**Date:** 19-May-2026
**Supersedes:** —
**Superseded by:** —
**Related:** ADR-005 (Client Credentials production target), ADR-006 (Authorization Code tactical), ADR-007 (Maker-provided credentials)

## Context
- Day 10 smoke test exposed a fundamental architectural gap: ServiceNow Agent has no mechanism to resolve the end user's identity into a ServiceNow `sys_id`
- Channel auth (Teams, web embed) provides identity claims (email, Azure AD object ID, UPN) — NOT ServiceNow primary keys
- ServiceNow's `incident` table indexes the caller by `sys_id` (32-char hash from `sys_user` table), not by email or any human-readable identifier
- ADR-007 resolved the connection-identity layer (maker-provided credentials) but does NOT resolve business-identity (caller_id) — these are independent concerns
- Without resolution, the LLM correctly refused to hallucinate sys_id (per `caller_id` input description guardrail) and asked the user — unacceptable UX for production

## Decision
- Build identity resolution as a **dedicated tool call**: add a 5th connector action `GetUserBySysProperty` that hits `/api/now/table/sys_user?sysparm_query=email=<email>`
- Agent instructions updated to mandate: "Resolve caller_id via GetUserBySysProperty before any CRUD operation that requires caller_id (Create, Update)"
- LLM chains tool calls — first resolves identity, then performs the CRUD action

## Why (Pattern 1 over Pattern 2 and Pattern 3)
- **Matches ADR-001 — LLM as primary orchestrator** — chaining tool calls is the LLM's job, not a Power Automate flow's
- **Observable + auditable** — resolution is a discrete tool call surface in agent logs; not buried inside a flow
- **Composable with existing authorization layers** — the resolved sys_id feeds Layer 1 prompt-driven query filtering (per instructions.md three-layer model)
- **Lowest infrastructure cost** — 1 connector action added; no Power Automate, no channel-side resolution wiring, no session bootstrap
- **Portfolio defensible** — interview narrative is "identity translation as an architectural layer," not "Microsoft Flow does the magic"

## Alternatives considered
- **Pattern 2 — Power Automate flow wraps every CRUD action** — rejected: violates ADR-001 (flow becomes orchestrator), adds latency, opaque to agent telemetry
- **Pattern 3 — Channel injects sys_id at session bootstrap** — rejected for now: per-channel wiring complexity (Teams, web, mobile each need their own setup); reconsider Week 4 if Teams-only deployment is the demo target
- **Hybrid — Pattern 1 default, Pattern 3 for performance-critical channels** — deferred: premature optimization; revisit after Teams deployment data exists

## Implementation (Day 11)
1. **ServiceNow prep** — create 3-5 test users in PDI with realistic emails matching Azure AD test accounts; capture sys_ids for verification
2. **Connector — add GetUserBySysProperty action**
   - GET `/api/now/table/sys_user`
   - Parameters: `sysparm_query=email=<email>`, `sysparm_fields` (default: `sys_id,name,email,department`), `sysparm_display_value=true`
   - Returns: `result` array (filter to one user, agent picks first match)
3. **Update connector-spec.yaml in repo**
4. **Agent — add new tool**
   - When may be used: Only when referenced
   - Ask user: No (silent identity resolution)
   - Credentials: Maker-provided (consistent with ADR-007)
   - Inputs: 1 dynamic input (`email`) with description: "Authenticated user's email from channel context. Use System.User.Email or equivalent."
   - Completion: Generative AI (or specific response binding sys_id) — TBD during build
5. **Update instructions.md** — add identity resolution flow at top of operations section: "Before any operation requiring caller_id, call GetUserBySysProperty with the authenticated user's email. Use the returned sys_id as caller_id."
6. **Test in Test pane** — manually inject email via system variable workaround (Power Platform supports test-time variable setting)
7. **Validate via Triage Agent chain** — once Triage → ServiceNow Agent wiring is in place

## Trade-offs accepted
- **Extra API call per session** — first CRUD operation costs an extra GET. Acceptable: <100ms, single-call cost per session if LLM caches sys_id in conversation context
- **LLM must chain correctly** — failure mode if LLM skips identity resolution and tries CRUD with blank caller_id. Mitigation: explicit instruction in instructions.md, deny-list in prompt, and ServiceNow's mandatory-field validation as backstop
- **Test pane limitations** — no native email injection in CPS Test pane; identity resolution can only be fully validated via published channel or manual variable override

## Identity layering (continues from ADR-007)
| Layer | Solved by |
|---|---|
| Connection identity | ADR-007 (maker-provided credentials) |
| Business identity (`caller_id`) | **ADR-008 (this ADR — GetUserBySysProperty)** |
| Operating identity (`opened_by`) | ServiceNow auto-sets to authenticated user |
| Origin identity (`u_agent_name`) | Application layer (hardcoded per agent) |

## Consequences
- Adds one connector action (5th in the set)
- Agent instructions.md requires update — identity-resolution flow must be the first thing the agent does on any caller_id-bearing operation
- Day 11 work item — Workstream B in close-out plan
- Production deployment to Teams becomes meaningful — Teams provides the email automatically, this tool consumes it
- Sets pattern for future agents (Knowledge, Remediation) that may also need identity resolution
