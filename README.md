# CloudPeek Agent Plugins

The official [Agent Plugins](https://agent-plugins.org/) packages for
[CloudPeek](https://cloudpeek.io), the AI security teammate.

This repository holds two plugins. They do different jobs, connect to
different services, and authenticate differently, so install whichever suits
the agent you are setting up. Each top-level directory is a complete plugin
root.

| Plugin | Directory | Job | Access |
| --- | --- | --- | --- |
| CloudPeek | [`cloudpeek/`](cloudpeek/) | Triage and investigate incidents in your own tenant | Sign in to your CloudPeek tenant |
| CloudPeek Vulnerability Intelligence | [`cloudpeek-vulnerability-intelligence/`](cloudpeek-vulnerability-intelligence/) | Look up CVEs and CWEs from a public indexed corpus | Anonymous, no account |

## CloudPeek

Connects your agent to the hosted CloudPeek MCP gateway and teaches it
CloudPeek's incident triage and investigation workflows.

- **`cloudpeek/mcp.json`** — one remote MCP server (`streamable-http`) at
  `https://app.cloudpeek.ai/mcp`, exposing the full CloudPeek API as MCP
  tools: incidents, investigations, artifacts, runbooks, wiki and more.
- **`cloudpeek/skills/`** — workflow playbooks your agent loads on demand:
  - `triage-incidents` — review the incident queue and surface the small
    fraction of alerts that need a human, with a full audit-comment trail.
  - `investigate-incident` — run a structured investigation, gather evidence,
    propose remediation plans and produce a report.

**Authentication.** The package ships no credentials. On first connection your
client discovers the gateway's OAuth 2.1 metadata, registers itself via
dynamic client registration, and walks you through a browser sign-in to your
CloudPeek tenant. Access is scoped to your CloudPeek user and enforced
server-side; revoke it at any time from your CloudPeek settings.

## CloudPeek Vulnerability Intelligence

Deterministic, source-grounded CVE and CWE lookup from CloudPeek's indexed
Open Knowledge Format corpus. Every answer traces back to the vendor or MITRE
advisory it was compiled from, rather than being recalled. Search is full-text
matching with CVE-to-CWE relationship expansion: no embeddings, no vector
stores, and no model calls anywhere in the lookup path.

- **`cloudpeek-vulnerability-intelligence/mcp.json`** — one remote MCP server
  (`streamable-http`) at `https://mcp.cloudpeek.ai/vulnerability/mcp`.
- **`cloudpeek-vulnerability-intelligence/skills/`**:
  - `research-vulnerability` — explain a specific CVE or CWE with sourced
    detail: severity, how it works, and how to fix it.
  - `review-recent-vulnerabilities` — brief the user on recently indexed or
    modified records, optionally filtered by product, vendor, severity or
    date. Good for catch-up briefings and recurring round-ups.
  - `assess-product-exposure` — work out which indexed CVEs affect a given
    product, vendor, package or version, and turn that into a prioritised
    exposure summary.

**Authentication.** None, and none is needed. This endpoint is anonymous and
read-only, so there is no sign-in step, no account, and no token to manage.
It reads a public corpus and holds nothing about you.

## Installation

Any client that supports the Agent Plugins standard can install either
directory as a plugin root. Point your client's plugin installer at this
repository and pick the plugin you want; no build step is required.

The vulnerability plugin additionally ships `.claude-plugin/` and
`.codex-plugin/` catalogue manifests plus `assets/`, so it presents with a
display name, description, icon and starter prompts in clients that read
those.

## Versioning and sync

Both plugins are mirrored from the CloudPeek monorepo, which is the source of
truth:

| Directory | Source |
| --- | --- |
| `cloudpeek/` | `src/mcp-server/plugin/` |
| `cloudpeek-vulnerability-intelligence/` | `src/plugins/vulnerability-intelligence-mcp/plugin/` |

Each `plugin.json` version tracks the release of the service it was published
from, and the two version lines are independent: the vulnerability MCP is
released on its own cadence and deliberately does not ride the application
release train.

Do not hand-edit these directories. Run the sync from the monorepo, which
mirrors both plugins and fails if a manifest advertises a version its package
never shipped:

```bash
bun scripts/sync-agent-plugins.mjs --check <path-to-this-checkout>   # report drift
bun scripts/sync-agent-plugins.mjs --write <path-to-this-checkout>   # bring in line
```

Contract tests in the monorepo verify the manifests against the spec and check
that every tool referenced by a skill exists in the live catalogue.

## Support

Questions or issues: [support@cloudpeek.io](mailto:support@cloudpeek.io) or
[cloudpeek.io](https://cloudpeek.io).
