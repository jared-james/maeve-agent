# Maeve Agent

Installable agent skills and plugins for safely operating Maeve through the public API, `maeve-cli`, and the hosted MCP endpoint.

This package includes the `maeve-social-scheduler` skill for social scheduling, publishing, content management, media library work, approvals, analytics, inbox, grid planning, hashtags, taxonomy, and automation workflows.

Tested with `maeve-cli >= 0.8.3`.

Hosted MCP compatibility: `https://api.maevesocial.com/mcp` with browser sign-in for OAuth-capable MCP clients. API-key auth is still available as a fallback for automation and older clients.

## What This Installs

- A portable Agent Skills-standard skill at `skills/maeve-social-scheduler`.
- A Codex plugin manifest for installing the skill in Codex.
- A Claude Code plugin manifest and marketplace for installing the skill in Claude Code.

The plugin is CLI-first by default. MCP can be connected separately for clients that support Streamable HTTP MCP servers. This repo does not include secrets and does not need access to the private Maeve application repositories.

## Prerequisites

- Node.js 22 or newer.
- An Maeve account with access to at least one workspace.
- For CLI use: either an Maeve API key or a browser-approved CLI login.
- For MCP use: an MCP client that supports browser authentication, or an API key for fallback setup.

Check the CLI:

```bash
npx maeve-cli auth:status
npx maeve-cli auth:login
npx maeve-cli workspaces:list
```

For automation, prefer environment variables.

macOS / Linux:

```bash
export MAEVE_API_KEY="ezb_live_..."
export MAEVE_API_URL="https://api.maevesocial.com"
```

PowerShell:

```powershell
$env:MAEVE_API_KEY="ezb_live_..."
$env:MAEVE_API_URL="https://api.maevesocial.com"
```

Do not put raw API keys in URLs, prompts, git history, or shared chat.

## Hosted MCP Setup

Maeve exposes a hosted MCP endpoint for agent-native tool calls:

```text
https://api.maevesocial.com/mcp
```

The plugin bundles the current MCP server config in `plugins/maeve-agent/.mcp.json`, so installing the plugin in Claude Code or Codex auto-wires the server. Open your MCP client and choose **Authenticate** for Maeve. Your browser opens Maeve, asks for approval, and returns to the MCP client.

MCP browser authentication has been validated with Codex and Claude Code. Maeve issues 6-hour access tokens and 90-day rotating refresh tokens, so approved clients can stay connected without copying secrets. Organization admins can revoke active AI tool connections from Maeve **Settings -> Developer**.

### Manual Setup

If you are not using the plugin, configure MCP directly.

Codex `~/.codex/config.toml`:

```toml
[mcp_servers.maeve]
url = "https://api.maevesocial.com/mcp"
```

Claude Code:

```bash
claude mcp add --transport http maeve https://api.maevesocial.com/mcp
```

Or a project `.mcp.json`:

```json
{
  "mcpServers": {
    "maeve": {
      "type": "http",
      "url": "https://api.maevesocial.com/mcp"
    }
  }
}
```

### API-Key Fallback

Use this only when your MCP client does not support browser sign-in yet, or for server-side automation.

macOS / Linux:

```bash
export MAEVE_API_KEY="ezb_live_..."
```

PowerShell:

```powershell
$env:MAEVE_API_KEY="ezb_live_..."
```

Codex fallback:

```toml
[mcp_servers.maeve]
url = "https://api.maevesocial.com/mcp"
bearer_token_env_var = "MAEVE_API_KEY"
```

Claude Code fallback:

```bash
claude mcp add --transport http maeve https://api.maevesocial.com/mcp \
  --header "Authorization: Bearer $MAEVE_API_KEY"
```

Do not put raw API keys in URLs, prompts, git history, shared project files, screenshots, or chat.

## Hosted MCP Vs CLI

- Use MCP when your agent client already has `maeve` connected and you want agent-native reads or low-risk actions such as listing workspaces, inspecting integrations, creating drafts, scheduling explicit posts, upload-session media workflows, analytics summaries, or inbox thread reads.
- Use the CLI for the broadest supported surface, local files, full media upload commands, taxonomy, hashtags, grid planner, approvals, reports, and any workflow where MCP is not connected.
- Use the public API directly only when the CLI and MCP do not cover the required workflow.
- CLI login and MCP login are separate. `maeve auth:login` signs in the CLI only. MCP uses browser sign-in through your MCP client, or the API-key fallback above.
- Browser-approved MCP connections can be revoked in Maeve **Settings -> Developer**. Reconnect from the MCP client with **Authenticate** if access is revoked or expires.

## Install In Codex

Add this repository as a Codex plugin marketplace:

```bash
codex plugin marketplace add jared-james/maeve-agent
```

Then open the plugin browser and install `maeve-agent`:

```text
/plugins
```

After installation, invoke the skill explicitly:

```text
$maeve-social-scheduler list my workspaces and connected integrations
```

## Install In Claude Code

From inside Claude Code, add this repository as a plugin marketplace:

```text
/plugin marketplace add jared-james/maeve-agent
/plugin install maeve-agent@maeve-agent
/reload-plugins
```

Plugin skills are namespaced in Claude Code. Invoke it with:

```text
/maeve-agent:maeve-social-scheduler list my workspaces and connected integrations
```

## Manual Local Skill Install

If your agent client does not support plugin marketplaces yet, copy the skill folder into the client-specific user skill directory.

Codex:

```text
~/.agents/skills/maeve-social-scheduler
```

Claude Code:

```text
~/.claude/skills/maeve-social-scheduler
```

## Example JSON Payload

Create `create-content.json`:

```json
{
  "integrationId": "00000000-0000-4000-8000-000000000001",
  "body": "Launch post copy",
  "saveAsDraft": true
}
```

Then run:

```bash
npx maeve-cli content:create --workspace <workspaceId> --json create-content.json
```

## Safety Model

The skill defaults to drafts, inspects workspace and integration capabilities before mutations, and requires explicit confirmation before externally visible, destructive, or broad actions.

## MCP Tool Coverage

See `plugins/maeve-agent/skills/maeve-social-scheduler/references/mcp-tools.md` for the current hosted MCP tool map and CLI/API fallbacks.

## Links

- API docs: https://api.maevesocial.com/docs
- Hosted MCP: https://api.maevesocial.com/mcp
- CLI package: https://www.npmjs.com/package/maeve-cli
- Maeve: https://maevesocial.com
