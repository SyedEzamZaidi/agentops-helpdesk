# ADR-006: Tactical OAuth Grant Deferral + Connector Default Parameters Pattern

**Date:** 17-May-2026
**Status:** Accepted
**Supersedes:** Partially supersedes ADR-005 (OAuth grant choice — see "Relationship to ADR-005")

## Context

Day 8-9 of the AgentOps Helpdesk build attempted to wire the custom Power Platform connector to ServiceNow using Client Credentials grant, as locked in ADR-005. Three days of debugging surfaced a ServiceNow Zurich-release UX issue: the inbound OAuth Client Credentials grant requires an OAuth Entity Profile binding the grant to a service account, but the Zurich UI hides this configuration on default form views. The Entity Profile must either be created via admin-only form personalization or scripted directly via GlideRecord — neither path was available on the developer PDI within the project timeline.

Additionally, during connector definition, an inconsistency emerged in how query parameters were configured across actions. Some actions had `sysparm_fields` and `sysparm_display_value` hardcoded in the URL (no override possible by caller), while others had them as named parameters with defaults (override possible). The hardcoded approach is more restrictive and undermines multi-consumer use of the connector.

## Decision

Two interrelated decisions:

### Decision 1 — Defer Client Credentials, ship Authorization Code with admin user

- Configure the custom connector for OAuth 2.0 Authorization Code grant
- Use the ServiceNow `admin` user for the connection
- Document this as a portfolio-only shortcut, not a production pattern
- Preserve ADR-005's Client Credentials architecture for the production deployment plan

### Decision 2 — All connector query parameters use named-parameter + default-value pattern

- Every reused query parameter (`sysparm_fields`, `sysparm_display_value`, etc.) declared as a named action parameter in the Power Platform definition
- Each parameter gets a sensible default value at the connector level
- Callers (agents) can override defaults by passing values explicitly
- No query parameters hardcoded in URL strings

## Consequences

### Positive

- Day 9 ships a working end-to-end custom connector with 4 actions (GetIncidents, GetIncidentById, CreateIncident, UpdateIncident)
- Architecture surface (connector spec, action signatures, agent design) is unchanged from the ADR-005 plan — only the OAuth grant configuration differs, so the production swap is a configuration change, not a redesign
- Connector becomes a true reusable component: each consuming agent can call actions with default behavior, or override parameters for specific needs
- "Transport concerns in connector, intent in agent" pattern enforced consistently
- ServiceNow API response shape standardized — every action returns the same curated 7-field set by default

### Negative

- Authorization Code grant doesn't match the production multi-user architecture
- End user identity in audit logs will show as `admin`, not the actual end user or service account
- Refresh tokens are stored in Power Platform's connection store per connected user; in production this would create scale issues
- Demo audience must be told the limitation explicitly to avoid misleading expectations
- ServiceNow Zurich Entity Profile UX issue remains unresolved — future projects on this stack must script the binding directly

### Neutral

- Per-parameter default values must be type-coerced explicitly in Power Platform (Boolean inference on `sysparm_display_value` caused a Swagger validation error; resolved by setting type to `string` with default `"true"`)

## Alternatives Considered

### Alt 1 — Stay on Client Credentials, script the Entity Profile binding
Direct GlideRecord script in ServiceNow → Background Scripts → insert binding into `oauth_entity_profile` table. Rejected because: (a) PDI restrictions on Scripts unclear, (b) brittle pattern not portable to production (real ServiceNow admins won't accept undocumented scripted bindings), (c) Day 9 timeline pressure.

### Alt 2 — Use a non-admin user with Authorization Code grant
Create a dedicated user (separate from `agentops_servicenow_svc`) with a password, use it for the OAuth login. Rejected because: it doesn't actually solve the production architecture gap, and it duplicates the service account concept without the benefits.

### Alt 3 — Hardcode all query parameters in URL strings (no named-parameter defaults)
Rejected because it removes caller override capability, making the connector unsuitable for multi-agent reuse. If the Knowledge Agent later needs a different field set in CreateIncident responses, hardcoded URLs force a connector definition change that impacts all consumers.

### Alt 4 — Defer connector defaults entirely (always require callers to pass all params)
Rejected because it pushes transport concerns into agent prompts. Agent prompts should describe intent (when to call, what to do with results), not transport mechanics (which 7 fields to request). Defaults at the connector layer enforce architectural separation.

## Relationship to ADR-005

ADR-005 locked Client Credentials grant + custom connector + service account architecture. That decision **stands as the production target**. ADR-006 is a tactical deferral, not a reversal. The relevant change scope when production-deploying:

- Configuration: Power Platform connector's OAuth section (one-time admin task)
- ServiceNow side: provision Entity Profile binding (via Zurich admin form personalization or script)
- Connector definition: unchanged
- Action signatures: unchanged
- Agent prompts: unchanged
- Authorization model (three-layer): unchanged

## Insights Logged

13. **ServiceNow Zurich Inbound OAuth Client Credentials Trap** — The OAuth app + service account binding requires a separate `oauth_entity_profile` record. Default Zurich forms hide the configuration; production setups script it directly. Authorization Code grant doesn't need this because identity is resolved at user-login time.

14. **OAuth Client Secret Used in Both Grant Types** — Client Credentials uses it as the primary credential; Authorization Code uses it during the server-side token exchange to verify the app's identity (preventing stolen-auth-code attacks). Same field, different security role.

15. **Power Platform Custom Connector Defaults Are Runtime-Only, Not Test-Tab** — The Test tab does not auto-apply parameter defaults; blank fields mean params are omitted from the test request. Runtime engine DOES auto-inject defaults. Test tab behavior differs from runtime behavior — verify both.

16. **Power Apps Connector Query Parameters Must Originate From URL** — There is no UI affordance to add query parameters to an action directly. Parameters are extracted from the URL query string at import time. Defaults can then be edited via the parameter editor. URL-as-input-method is a quirk of the Power Apps custom connector designer.

17. **Power Apps Connector Type Inference Can Be Wrong** — Parameter type is inferred from sample values during import. Names like `sysparm_display_value` plus values like `true` lead Power Apps to infer Boolean type. If the default value is then set as a plain string, Swagger validation fails. Workaround: explicitly set Type=string in the parameter editor.

18. **ServiceNow Priority Is a Calculated Field** — Incident `priority` is derived from `impact` × `urgency` via business rule. POST/PATCH calls sending `priority` directly are silently overridden. To control priority, send `impact` and `urgency`; ServiceNow derives priority.

19. **POST Response Reference Fields Are Resolved Differently Than GET** — `sysparm_display_value=true` resolves reference fields fully on GET responses, but partially on POST responses (returns `{link, value}` object). For full string resolution on POST, use `sysparm_display_value=all`. Trade-off: `all` loses sys_id easy-access for follow-up calls.

20. **Power Platform Custom Connectors Route Through `*.azure-apim.net` Gateway** — The Bearer token on the wire from caller to gateway is the caller's Azure AD token. Gateway substitutes the actual OAuth token to the target API server-side. Explains why connections are per-user even when connector definition is environment-scoped.

## References

- ADR-005 (OAuth grant + custom connector design)
- ServiceNow REST API documentation, Table API
- ServiceNow Priority Lookup Rules table
- Power Platform custom connector OpenAPI extensions (x-ms-*)
- Project Insights #13-20 in project-context.md
