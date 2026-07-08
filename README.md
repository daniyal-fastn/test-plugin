# ucl-marketplace

A one-plugin Claude marketplace you can hand to a customer. Installing the **`ucl-gateway`** plugin does two things at once:

1. **Registers the UCL gateway MCP server** - the agent connects directly to the gateway (which handles auth itself), no manual MCP setup.
2. **Ships the gateway usage skill** - static instructions that persist across sessions, telling the agent how to discover (`list_skills`) and run the org's governed integration/connector skills.

No local tools, no scripts, no runtime dependency. It is a plain plugin: a manifest, an MCP registration, and a skill. This works in Claude Code and in **Claude Cowork** (installed plugins live at the account level and reload every session).

## Layout

```
.claude-plugin/marketplace.json          the marketplace manifest (lists the plugin)
plugins/ucl-gateway/
  .claude-plugin/plugin.json             the plugin manifest
  .mcp.json                              registers the ucl-gateway MCP server
  skills/ucl-gateway/SKILL.md            static usage skill (how to use the gateway)
```

## The one thing to set before shipping: the gateway endpoint

`plugins/ucl-gateway/.mcp.json` points at the production gateway:

```json
{ "ucl-gateway": { "type": "http", "url": "https://connect.fastn.dev/mcp" } }
```

Set `url` to the gateway endpoint the customer should hit (their org's gateway, or the production `ucl.dev` endpoint) before distributing. This is the only required configuration.

## Install (customer steps)

**Claude Code**
```
/plugin marketplace add <owner>/<repo>        # or a local path to this folder
/plugin install ucl-gateway@ucl-marketplace
/reload-plugins                               # skills/MCP register at session start
/mcp                                          # confirm ucl-gateway is connected
```

**Claude Cowork (Claude Desktop)**
Open the **Cowork** tab -> **Customize** -> add this marketplace (from GitHub) or **upload the plugin**, then install `ucl-gateway`. Restart the session so it loads. The gateway connector then appears in the connector list, and the usage skill auto-triggers on integration/connector tasks.

## What the customer gets

On install, with no further setup, the agent can:
- see the gateway's skills via `list_skills`,
- run a saved skill by its slug (composite = one governed call; playbook = step-by-step),
- get a connect link when a needed account isn't connected yet, and connect it,
- file corrections back to the skill owner via `capture_feedback`.

Because the plugin is installed at the account/app level, the skill and the gateway connection are **reused across every session** - the reason this is a plugin, not a one-off skill upload.
