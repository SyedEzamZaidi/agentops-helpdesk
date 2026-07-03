# AgentOps Helpdesk

Multi-agent IT helpdesk on Microsoft Copilot Studio. Work in progress.

## What it does

Routes IT support requests through specialized agents — triage, knowledge lookup, ServiceNow ticket actions, and gated remediation — instead of one monolithic chatbot.

## Why I'm building it

I've spent 7 years building RPA solutions, mostly in Automation Anywhere. Most enterprise IT helpdesk work isn't a narrow automation problem — it's a coordination problem across systems. Multi-agent systems on Copilot Studio look like the right fit for that. Building this to find out if I'm right.

Also prepping for Microsoft AB-620.

## Stack

- Microsoft Copilot Studio (agents)
- Microsoft Foundry (orchestration, RAG, evaluation)
- ServiceNow (ITSM integration via MCP)
- Azure AI Search (knowledge base)
- Azure Key Vault, Application Insights
- GitHub Actions for ALM

## Status

Oauth connection setup with ServiceNow. Triage agent is working.

