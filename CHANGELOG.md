# Changelog

## 1.0.0

- Add one portable Agent Plugins v1 package for Cursor, Grok Build, Qwen Code, Codex and compatible clients.
- Add a native Gemini CLI extension manifest and installation guidance for every supported agent.
- Move the shared Maeve skill to the repository root so each package uses one maintained source.
- Describe the current four-tool hosted MCP catalog and Maeve Social workflows across content, media, analytics, tasks, reviews, strategy and inbox automation.

## 0.7.6

- Align the CLI reference with the canonical `maeve-cli` 0.12.4 root command surface, including Calendar, content-root and Strategy workflows.
- Remove retired archive and restore commands, and document the current deletion, Bin and scheduling-reversal semantics.
- Require CLI 0.12.4 for CLI workflows, add the Strategy Retro reads, and document the current confirmation-gated commands.

## 0.7.5

- Publish the scheduler skill rewrite for the four-tool MCP catalog. The 0.7.4 artifact shipped before that rewrite landed, so installed copies still told agents to call the retired `list_workspaces`, `create_draft_content` and `get_integration_options` tool names. The server returns `not_found` for those as tool names and `validation` as operation IDs, so an agent following the old skill failed on its first MCP call.
- The skill now describes the actual surface: four generic tools, `maeve_search`, `maeve_details`, `maeve_read` and `maeve_write`, with read and write executed as `{ operationId, input }`.
- Document catalog discovery as a two-step flow. Searching without a workspace returns almost nothing, because operations whose catalog entry requires workspace context are filtered out. Resolve `workspace.list` first, then search again with the selected workspace.
## 0.7.4

- Refresh content-model guidance through Google Business Profile, including caption overrides, clearing and replacement, thread identities, trial reels, TikTok music and delivery modes.
- Require CLI 0.12.0 for the updated content workflows.

- When the hosted MCP server is configured but not yet connected, point the user at the client's own connect step (`/mcp` in Claude Code, `codex mcp login maeve` in Codex) instead of pasting authorization URLs or falling back to the CLI.
- Only run CLI auth checks when no MCP server is configured.

## 0.7.3

- Correct Codex manifest metadata and align public Maeve Social naming.
- Bring platform capability and installation documentation up to date.
- Add confirmation guidance for inbox automation enablement and task-comment mentions.
- Document the countries where Maeve Social accounts are currently available.
