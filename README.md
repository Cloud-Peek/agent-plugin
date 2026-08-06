# CloudPeek Agent Plugin

The official [Agent Plugins](https://agent-plugins.org/) package for
[CloudPeek](https://cloudpeek.io), the AI security teammate. Installing it
connects your coding or desktop agent to the hosted CloudPeek MCP gateway and
teaches it CloudPeek's incident triage and investigation workflows.

## What you get

- **`mcp.json`** — one remote MCP server (`streamable-http`) at
  `https://app.cloudpeek.ai/mcp`, exposing the full CloudPeek API as MCP
  tools: incidents, investigations, artifacts, runbooks, wiki and more.
- **`skills/`** — workflow playbooks your agent loads on demand:
  - `triage-incidents` — review the incident queue and surface the small
    fraction of alerts that need a human, with a full audit-comment trail.
  - `investigate-incident` — run a structured investigation, gather evidence,
    propose remediation plans and produce a report.
- **`plugin.json`** — the plugin manifest (Agent Plugins spec 1.0.0).

## Installation

Any client that supports the Agent Plugins standard can install this
repository as a plugin directory. Point your client's plugin installer at
this repo (or a cloned copy); no build step is required.

## Authentication

The package ships **no credentials**. On first connection your client
discovers the gateway's OAuth 2.1 metadata, registers itself via dynamic
client registration, and walks you through a browser sign-in to your
CloudPeek tenant. Access is scoped to your CloudPeek user and enforced
server-side; revoke it at any time from your CloudPeek settings.

## Versioning

The `version` in `plugin.json` tracks the CloudPeek MCP gateway release it
was published from. This repository is synced from the CloudPeek monorepo
(`src/mcp-server/plugin/`), where a CI contract test verifies the manifests
against the spec and checks that every tool referenced by a skill exists in
the live gateway catalogue.

## Support

Questions or issues: [support@cloudpeek.io](mailto:support@cloudpeek.io) or
[cloudpeek.io](https://cloudpeek.io).
