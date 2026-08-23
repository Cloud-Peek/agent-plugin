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

Both plugins are plain manifests. There is no build step and nothing to
compile: your client reads `mcp.json`, connects to the hosted endpoint, and
loads the skills on demand.

Each plugin root carries a spec manifest (`plugin.json`) plus one catalogue
manifest per client, so the same directory installs everywhere:

| File | Read by |
| --- | --- |
| `plugin.json` | Agent Plugins 1.0.0 spec clients |
| `.claude-plugin/plugin.json` | Claude Code |
| `.codex-plugin/plugin.json` | Codex |
| `.cursor-plugin/plugin.json` | Cursor |

All of them point at the one shared `mcp.json`, so the endpoint is defined
exactly once.

### Claude Code

```shell
/plugin marketplace add Cloud-Peek/agent-plugin
/plugin install cloudpeek-vulnerability-intelligence@cloudpeek-plugins
/plugin install cloudpeek@cloudpeek-plugins
```

Keep the `@cloudpeek-plugins` suffix. Claude Code refreshes the catalogue
before a named install, so without it you can install a stale cached version.

To add just the MCP server without the skills:

```shell
claude mcp add --transport http cloudpeek-vulnerability-intelligence https://mcp.cloudpeek.ai/vulnerability/mcp
```

### Cursor

Install the plugin from **Customize** in the sidebar, or add this repository
as a team marketplace under **Dashboard → Plugins → Import from Repo**.

To add only the MCP server, put this in `.cursor/mcp.json` for one project, or
`~/.cursor/mcp.json` for every project:

```json
{
  "mcpServers": {
    "cloudpeek-vulnerability-intelligence": {
      "url": "https://mcp.cloudpeek.ai/vulnerability/mcp"
    }
  }
}
```

### OpenCode

OpenCode plugins are JavaScript modules rather than manifest bundles, so
these plugins install as MCP servers. Add to `opencode.json`:

```json
{
  "$schema": "https://opencode.ai/config.json",
  "mcp": {
    "cloudpeek-vulnerability-intelligence": {
      "type": "remote",
      "url": "https://mcp.cloudpeek.ai/vulnerability/mcp",
      "enabled": true
    }
  }
}
```

Swap the URL for `https://app.cloudpeek.ai/mcp` to add the CloudPeek gateway
instead. Note that the gateway needs an OAuth sign-in, so a client that cannot
run that flow will not connect.

### Any other MCP client

Point it at `https://mcp.cloudpeek.ai/vulnerability/mcp` over streamable HTTP.
No credentials are required.

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
