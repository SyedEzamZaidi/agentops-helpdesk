# Platform notes

Copilot Studio and Power Automate behaviour that affected the build, with the evidence and the workaround used. Checked against the platform as of October 2026.

## 1. Agent flow runs with the caller's connection despite maker credentials

**Symptom:** The tool was set to *Maker-provided credentials*, the flow's run-only setting showed *Use this connection*, and both Dataverse connection references were mapped to the maker. The agent still showed a *Connect to continue* (Dataverse) card, and the flow failed with `FlowActionBadGateway`. The run history showed `401 Unauthorized` on *Add a new row* with an empty bearer token.

**Isolation:**
- Run history inputs were correct, so the data was not the problem; the 401 pointed to authentication.
- With agent authentication set to *No authentication*, the call failed with `AuthenticationNotConfigured`, so the runtime was requesting a user token for this flow regardless of the tool setting.
- The unmanaged solution export showed the flow definition with:
  ```json
  "shared_commondataserviceforapps": {
    "connection": { "connectionReferenceLogicalName": "new_sharedcommondataserviceforapps_7784ebd9" },
    "runtimeSource": "invoker"
  }
  ```
  while the agent tool component had `mode: Maker`.
- A flow rebuilt from scratch in Copilot Studio, with maker credentials set before the first save, was also created as `invoker`.

**Fix:** In the unmanaged export, `"runtimeSource": "invoker"` → `"embedded"` on the flow's Dataverse connection, version bumped, solution re-imported. No consent prompt afterwards and the tool call succeeded. A later edit, save and publish in the Copilot Studio designer did not revert it.

**Contributing factor:** a second agent in the same solution had a tool on the same flow in end-user mode. The credential mode is set per agent tool but stored on the shared flow (see D19).

**Follow-up (07 Oct 2026):** an agent flow built in the classic Power Automate designer (*When an agent calls the flow* trigger, inside the solution) was exported as `embedded` from the start (`wf_Common_UpdateDecision`). The patched `wf_Triage_LogRoutingDecision` was still `embedded` after later edits and publishes in Copilot Studio. Working practice: build agent flows in the classic designer within the solution, then attach them as tools; re-check the export after attaching.

**Guard:** D18. Check `runtimeSource` and the tool `mode` in the unpacked solution before every deployment.

Reference: Microsoft Copilot Studio CAT team, [Combining Agent Flows with Agents: Gotchas, Errors, and Patterns](https://microsoft.github.io/mcscatblog/posts/combining-agent-flows-and-agents-gotchas-errors-and-patterns/).

## 2. Run-only settings panel does not control agent flows

The classic flow details page (`make.powerautomate.com/environments/<env>/flows/<id>/details`) is only reachable by direct URL for agent flows; opening the flow from a list redirects to the Copilot Studio designer. Its *Run only users → Connections used* panel showed *Use this connection* while the definition held `invoker`, and saving a different value did not persist. Use the solution export as the source of truth.

## 3. System variables missing from the tool input picker

On a new tool, the *Choose a variable → System* tab was empty until the tool's required Details (name, description) were saved and the page hard-refreshed. Order that works: fill Details → Save → refresh → bind `System.User.Email` / `System.Conversation.Id` → Save. A stale browser cache produced the same empty list inside topics; a private window confirmed it.

## 4. Flow input keys follow creation order

Trigger inputs get internal keys by type and creation order (`text`, `text_1`, …), independent of display order or name. Expressions must use the key; insert the input from dynamic content inside the expression editor rather than typing the key.

## 5. Deleted tools can keep their name

After deleting a flow, its tool's display name stayed reserved on the agent (`'…' already exists as a 'Model display name'`). The replacement tool is `wf_Triage_LogRoutingDecision_v2`. Open item: remove the orphaned component and restore the original name.
