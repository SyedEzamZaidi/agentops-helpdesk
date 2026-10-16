# Topics

Under generative orchestration the planner selects topics by their description, so each description is written as a routing contract with explicit exclusions.

## Escalate (system topic)

**Trigger:** On Talk to Representative

**Description:**
```text
Use only when the user explicitly asks to speak to a human, a live agent or a person (for example "talk to a person", "connect me to an agent"). Do not use for reporting problems, security incidents, hacked accounts, phishing, outages or any IT request; those must be classified and logged instead.
```

**Why scoped:** with the default description ("Talk to agent, Talk to a person, …") the planner sent security incidents here, which bypassed the logging tool and left no Agent Decision row ([20-case run](../eval/results/2026-10-04-triage-standard-20case.md)). After scoping, all Escalate cases are logged ([rerun](../eval/results/2026-10-04-triage-standard-20case.md#rerun-after-fixes)).

**Next:** replace the placeholder message with a hand-off that logs the request first and returns the IT service desk contact.

## Knowledge Feedback (custom topic)

Records how a Knowledge request ended on the request's Agent Decision row in two stages. First it writes Knowledge / Answered and the cited article as soon as the answer is given, so a conversation abandoned before any feedback is still attributed to the Knowledge agent. Then a Yes/No question upgrades Outcome to Resolved (self-service) or Not found ([D26](../docs/decisions.md#d26-deterministic-closing-for-knowledge-answers-escalation-is-a-path-not-an-agent)).

**Trigger:** The agent chooses. Referenced from the parent instructions ("After ag_Knowledge has answered, always run Knowledge Feedback").

**Description:**
```text
Use immediately after the ag_Knowledge agent has answered an IT how-to question, to ask the user whether the answer solved their problem. Do not use in any other situation.
```

**Input:** `SourceArticle` (String), filled by the orchestrator from the answer just given; never prompted.
```text
The KB article ID cited in the answer just given, for example KB-002. Leave empty if no article was cited.
```

**Nodes:**
1. Action → `wf_Common_UpdateDecision`: ConversationId = `System.Conversation.Id`, HandledBy = `Knowledge`, Outcome = `Answered`, SourceArticle = `Topic.SourceArticle`, IncidentNumber not set.
2. Question "Did this solve your problem?": multiple choice `Yes` / `No`, saved to `Topic.Solved` (topic scope).
3. Condition branches:
   - **Yes** → Action → `wf_Common_UpdateDecision`: HandledBy = `Knowledge`, Outcome = `Resolved (self-service)` → message "Great, glad that sorted it."
   - **No** → Action → `wf_Common_UpdateDecision`: HandledBy = `Knowledge`, Outcome = `Not found` → message "Sorry that didn't help. I've recorded it so the IT team can follow up." (raises a ticket once the ServiceNow agent exists)
   - **All other conditions**: empty; the Answered outcome stands.
   Neither branch passes SourceArticle or IncidentNumber, so the article recorded in step 1 is kept.
4. End current topic.

The flow's `status` output is stored in `Topic.Status` and not used; a failed flow call surfaces as a topic error.

| Value | Source | Deterministic |
|---|---|---|
| Whether the topic runs | Orchestrator, from the description and instruction | No |
| SourceArticle | Orchestrator extracts it from the answer text | No (extraction, not judgement) |
| ConversationId | Platform | Yes |
| HandledBy, Outcome | Fixed in each node; the branch taken decides the Outcome | Yes |

The article is written once, at answer time; later outcome updates leave it unset so the flow keeps the stored value.

**Verified (test pane, 07 Oct 2026):** `How do I set up MFA on a new phone?` → Triage logs Knowledge → `ag_Knowledge` answers from KB-002 → Knowledge Feedback (SourceArticle = KB-002) → row: Handled By = Knowledge, Outcome = Answered, Source Article = KB-002. Negative case: `I cant connect to VPN` → classified Ticket; the topic did not run.

**Verified (test pane, 16 Oct 2026):** `How do I connect to the company Wi-Fi on my phone?` → answer from KB-006 → Knowledge Feedback (SourceArticle = KB-006) → "Did this solve your problem?" → **Yes** → row: Handled By = Knowledge, Outcome = Resolved (self-service), Source Article = KB-006. `How do I set up Microsoft Teams on my laptop?` → **No** → Outcome = Not found, Source Article kept. With the question as the closing message, the parent's separate "request logged" confirmation no longer appeared.

## Request Account Fix (custom topic)

Remediation intake (D29). Submits a Remediation Request; it never changes an account.

**Trigger:** The agent chooses. Referenced from the parent instructions ("If the category is Remediation, after logging run the Request Account Fix topic").

**Description:**
```text
Use after the request has been classified as Remediation and logged, when the employee asks to unlock an account, reset MFA or get VPN access. Do not use for how-to questions, problems that need investigation or anything else.
```

**Inputs** (orchestrator-filled, never prompted):
| Input | Description |
|---|---|
| `ActionCode` | Exactly one of: UNLOCK_ACCOUNT, RESET_MFA, ENABLE_VPN_ACCESS, matching what the user asked for. |
| `TargetUserEmail` | Email of another person whose account the fix is for, only if the user names someone else. Leave empty when the request is about the user's own account. |

**Nodes:**
1. Condition `TargetUserEmail` is blank → Set `TargetUserEmail` = `System.User.Email`. The model never guesses the user's own address; a named other person is kept and rejected later by the self-service rule.
2. Action → `wf_Remediation_SubmitRequest`: ActionCode and TargetUserEmail from the topic, RequesterEmail = `System.User.Email`, ConversationId = `System.Conversation.Id`; output `RequestNumber`.
3. Condition `RequestNumber` is not blank → "I've submitted your request {RequestNumber}. It will be checked against the access rules and, if needed, sent to your manager for approval. You'll get a Teams message with the result." Otherwise → "I couldn't submit that request. Please contact the IT service desk."
4. End current topic.

**Verified (test pane, 16 Oct 2026):** `I need VPN access on my laptop` → Triage logs Remediation → Request Account Fix → REM-00002 submitted (Pending Approval, Enable VPN access, requester = target).
