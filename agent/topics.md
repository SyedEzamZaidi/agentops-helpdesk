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

Records how a Knowledge request ended on the request's Agent Decision row. Step A (built): writes Knowledge / Answered and the cited article as soon as the answer is given, so a conversation abandoned before any feedback is still attributed to the Knowledge agent. Step B (next): Yes/No question that upgrades Outcome to Resolved (self-service) or Not found ([D26](../docs/decisions.md#d26-deterministic-closing-for-knowledge-answers-escalation-is-a-path-not-an-agent)).

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
2. End current topic.

| Value | Source | Deterministic |
|---|---|---|
| Whether the topic runs | Orchestrator, from the description and instruction | No |
| SourceArticle | Orchestrator extracts it from the answer text | No (extraction, not judgement) |
| ConversationId | Platform | Yes |
| HandledBy, Outcome | Fixed in the node | Yes |

The article is written once, at answer time; later outcome updates leave it unset so the flow keeps the stored value.

**Verified (test pane, 07 Oct 2026):** `How do I set up MFA on a new phone?` → Triage logs Knowledge → `ag_Knowledge` answers from KB-002 → Knowledge Feedback (SourceArticle = KB-002) → row: Handled By = Knowledge, Outcome = Answered, Source Article = KB-002. Negative case: `I cant connect to VPN` → classified Ticket; the topic did not run.
