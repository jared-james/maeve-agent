# Maeve Social for AI agents

Plan campaigns, create content, schedule posts, and understand what works, all by talking to your AI agent.

Connect Maeve Social to Codex, Claude Code, or another compatible agent. Your agent can work inside your existing Maeve workspace to create content, manage your calendar and Media Room, review performance, organize tasks, support approvals, and configure inbox automation.

Maeve supports Instagram, Facebook, TikTok, LinkedIn, LinkedIn Pages, X, Threads, YouTube, Pinterest, and Google Business Profile.

## Things you can ask

- "Show me everything scheduled next week and point out the gaps."
- "Turn these product photos into content for Instagram and LinkedIn."
- "Adapt this announcement for Instagram, X, Threads, and LinkedIn."
- "Schedule the approved campaign for 9:00am Sydney time across the selected accounts."
- "Which content performed best last month, and what patterns should we use next?"
- "Find the campaign images in my Media Room and organize them into a new folder."
- "Create the launch tasks, add checklists, and move them through the task board."
- "Send this content to the team for review."
- "Create an inbox auto-reply for common shipping questions and show me how often it runs."
- "Summarize our current strategy goals and bets."

## What your agent can do

- Create, update, schedule, publish, and organize content across connected social accounts.
- Review upcoming content, calendar notes, and campaign gaps.
- Tailor captions and publishing options for each platform.
- Upload, find, label, move, and organize images and videos in the Media Room.
- Read account and content analytics, including post-level performance and audience demographics.
- Create and manage tasks, comments, and checklists.
- Work with content tables, strategy foundations, goals, bets, and retros.
- Request internal or client review and manage review batches.
- Create and monitor inbox auto-reply rules.
- Use Maeve's CLI for local files and workflows that are not available through the hosted connection.

## Install in Codex

Add this repository as a Codex plugin marketplace:

```bash
codex plugin marketplace add jared-james/maeve-agent
```

Open the plugin browser and install `maeve-agent`:

```text
/plugins
```

Then ask Codex to work with Maeve naturally, or invoke the skill explicitly:

```text
$maeve-social-scheduler show me what is scheduled next week
```

When prompted, authenticate with Maeve in your browser.

## Install in Cursor

Open the Cursor Marketplace, search for `Maeve Social`, and install the plugin.

When prompted, authenticate with Maeve in your browser. Cursor will load the shared Maeve skill and hosted MCP connection from the portable Agent Plugin package.

## Install in Grok Build

Open the Grok Build plugin marketplace, search for `Maeve Social`, and install the plugin.

When prompted, authenticate with Maeve in your browser. Grok Build will load the Maeve skill and hosted MCP connection from the plugin package.

## Install in Gemini CLI

Install the extension directly from GitHub:

```bash
gemini extensions install https://github.com/jared-james/maeve-agent
```

When prompted, authenticate with Maeve in your browser. Gemini CLI will discover the bundled skill and connect to Maeve's hosted MCP server.

## Install in Qwen Code

Install the portable Agent Plugin directly from GitHub:

```bash
qwen extensions install jared-james/maeve-agent
```

When prompted, authenticate with Maeve in your browser. Qwen Code will load the shared skill and Streamable HTTP MCP server from the portable Agent Plugin package.

## Install in Claude Code

Add the marketplace and install the plugin:

```text
/plugin marketplace add jared-james/maeve-agent
/plugin install maeve-agent@maeve-agent
/reload-plugins
```

Open `/mcp`, choose `maeve`, and authenticate in your browser.

You can then ask Claude to work with Maeve naturally, or invoke the namespaced skill:

```text
/maeve-agent:maeve-social-scheduler show me what is scheduled next week
```

## Connect another MCP client

Maeve exposes a hosted Streamable HTTP MCP endpoint:

```text
https://api.maevesocial.com/mcp
```

Add that endpoint to any compatible client and use its **Authenticate** action to sign in to Maeve.

Example Codex configuration in `~/.codex/config.toml`:

```toml
[mcp_servers.maeve]
url = "https://api.maevesocial.com/mcp"
```

Example Claude Code command:

```bash
claude mcp add --transport http maeve https://api.maevesocial.com/mcp
```

Example project `.mcp.json`:

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

## Requirements

- A Maeve Social account with access to at least one workspace.
- An MCP client that supports browser authentication, or an API key for fallback automation.
- Node.js 22 or newer when using the Maeve CLI.

Maeve Social accounts are currently offered to customers in Australia, New Zealand, and the United States.

## CLI fallback

The hosted connection handles most agent workflows. The plugin uses the Maeve CLI when it needs local file access or a product area outside the hosted MCP catalog.

Install the current CLI globally:

```bash
npm install -g maeve-cli@latest
```

Or run it without a global installation:

```bash
npx maeve-cli@latest
```

Sign in and check your workspaces:

```bash
maeve auth:status
maeve auth:login
maeve workspaces:list
```

CLI login and MCP login are separate. Signing in to one does not authenticate the other.

### API-key authentication

Use an API key when browser authentication is unavailable or for server-side automation.

macOS and Linux:

```bash
export MAEVE_API_KEY="ezb_live_..."
export MAEVE_API_URL="https://api.maevesocial.com"
```

PowerShell:

```powershell
$env:MAEVE_API_KEY="ezb_live_..."
$env:MAEVE_API_URL="https://api.maevesocial.com"
```

Codex fallback:

```toml
[mcp_servers.maeve]
url = "https://api.maevesocial.com/mcp"
bearer_token_env_var = "MAEVE_API_KEY"
```

Do not put raw API keys in URLs, prompts, project files, screenshots, shared chat, or git history.

## Coverage

The hosted MCP connection supports workspace and integration discovery, content management, scheduling and publishing, Media Room organization, analytics, task boards, workbench content tables, review requests, calendar workflows, strategy, and inbox auto-reply configuration.

The CLI covers local file uploads and additional workflows such as live inbox messaging, approval decisions, client review actions outside MCP, grid planning, taxonomy, hashtags, and report generation. The public API is the final fallback when neither surface covers the workflow.

The current operation map and exact fallbacks are documented in [`mcp-tools.md`](skills/maeve-social-scheduler/references/mcp-tools.md).

## Package details

The stable package ID is `maeve-agent`. The included skill ID is `maeve-social-scheduler`. User-facing surfaces use the product name **Maeve Social**.

This repository includes:

- A portable Agent Plugins v1 package with the Maeve skill at `skills/maeve-social-scheduler`.
- A Codex plugin manifest and repository marketplace entry.
- A Claude Code plugin manifest and repository marketplace.
- A shared Streamable HTTP MCP configuration with browser authentication.

The hosted MCP surface exposes four stable tools:

- `maeve_search` finds the right authorized operation for a request.
- `maeve_details` returns the current requirements and input schema for that operation.
- `maeve_read` runs read operations.
- `maeve_write` runs actions that change Maeve or a connected platform.

The CLI workflows in this package require `maeve-cli >= 0.12.4`. MCP runs in the hosted backend and does not require a CLI installation.

This repository contains no credentials and does not need access to Maeve's private application repositories.

## Manual skill installation

If your client does not support plugin marketplaces, copy `skills/maeve-social-scheduler` into its user skill directory.

Codex:

```text
~/.agents/skills/maeve-social-scheduler
```

Claude Code:

```text
~/.claude/skills/maeve-social-scheduler
```

## Maintaining the plugin

This repository is the maintained public source for the portable, Codex, and Claude plugin packages. Update the bundled skill under `skills/maeve-social-scheduler`, keep the marketplace and plugin manifests aligned when the version changes, then install the repository package locally and test it in a new conversation before publishing. Installed plugin caches are outputs and must not be edited as source.

The hosted backend, this repository, and ChatGPT plugin metadata have separate release lifecycles. After a hosted MCP metadata change, refresh a developer-mode ChatGPT connection, confirm the advertised tool metadata, and start a new conversation. Published ChatGPT plugins use reviewed metadata snapshots, so updates require scanning the server, submitting a new version, and publishing the approved version. See the [official OpenAI connector refresh process](https://developers.openai.com/plugins/deploy/connect-chatgpt#refresh-metadata).

## Support and policies

- Website: https://maevesocial.com
- API documentation: https://api.maevesocial.com/docs
- Hosted MCP: https://api.maevesocial.com/mcp
- CLI package: https://www.npmjs.com/package/maeve-cli
- Support: https://maevesocial.com/contact or `support@maevesocial.com`
- Privacy policy: https://maevesocial.com/privacy
- Terms of service: https://maevesocial.com/terms
- Issues: https://github.com/jared-james/maeve-agent/issues

## License

MIT
