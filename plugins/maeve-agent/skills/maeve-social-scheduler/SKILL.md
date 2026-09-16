---
name: maeve-social-scheduler
description: Use Maeve MCP, CLI or public API to create and schedule social content, manage media, inspect performance, and handle content workflows. Use for tasks in a Maeve workspace.
---

# Maeve Social

## Choose the connection

Prefer the connected Maeve MCP catalog for supported tasks. If the configured connection needs login, use the client's own connect action. Do not request OAuth codes or tokens in chat. Use the CLI for local files, uncovered workflows, or when no MCP connection is configured; use the public API when neither surface covers the task.

Verify the selected API environment, organization, workspace and integration. A coding-session connector, ChatGPT connection and CLI login may point to different environments. CLI login does not authenticate MCP. For CLI work, run `maeve auth:status` against the intended `--api-url` before reads or writes. Treat a command as available only when it appears in root `maeve --help`; do not use subcommand help as an existence probe. Keep credentials and signed URLs out of output.

## Create and change content

1. Use `maeve_search` to discover `workspace.list`, execute it with `maeve_read`, then search again with the selected workspace. Load `maeve_details` for every chosen operation before execution. Read the connected integration's live capabilities and fetch dynamic options when a capability includes `optionKey`.
2. Read the selected platform in [platform-content.md](references/platform-content.md). Use the live tool schema or CLI schema; their arguments are not interchangeable.
3. Create a draft unless the user asked to publish or schedule. MCP executes `content.create_draft` through `maeve_write`; CLI uses `content:create` with omitted or draft `intent`. Upload local media through Maeve before attaching its IDs.
4. Resolve the publication date, time and timezone before scheduling. Confirm any missing intent, destination or external effect before publishing, deleting, sending messages or making broad changes. Existing authorization applies to the same action and targets; do not repeatedly ask for it.
5. Read saved values after changes. Preserve exact captions, per-platform overrides and attachment order. Use documented clearing values; omitted fields generally preserve existing values.

For a confirmation-gated MCP operation, put the exact required `confirmationText` inside the `maeve_write` operation input only when the user authorized that action and target. Add CLI `--yes` only where the command requires it. Review decisions remain human actions; requesting a review may notify people. See [safety-policy.md](references/safety-policy.md) for action-specific requirements.

## Results and recovery

Report the affected IDs, workspace, destination and status, plus scheduled time when relevant. Label the returned `appUrl` **Open in Maeve**; never invent app URLs. Return a non-null HTTPS `permalink` from a `content.get` read as **View on platform**.

A queued response, sent status or provider ID is not proof of native visibility. When the task requires live verification, check the provider author, rendered content and resource identity separately. Supported provider deletion can retain the sent audit in Maeve.

For a repeated create, retain successful IDs and reuse the identical payload and idempotency key. Inspect conflicts before changing input. Respect retry-after limits; do not switch connections or create replacement content to recover uncertain publishing outcomes. A lost MCP session requires fresh initialization and a fresh four-tool declaration, not activation or recreation of saved content.

## References

Load only what the task needs:

- [mcp-tools.md](references/mcp-tools.md): connection, catalog discovery, execution, upload, batches, recovery and distribution boundaries.
- [command-reference.md](references/command-reference.md): CLI command groups and sequences.
- [content-payloads.md](references/content-payloads.md): JSON shapes and scheduling.
- [integration-capabilities.md](references/integration-capabilities.md): limits and dynamic options.
- [platform-content.md](references/platform-content.md): platform inputs, clearing and publication evidence.
- [safety-policy.md](references/safety-policy.md): notifications, destructive actions and confirmation fields.
- [troubleshooting.md](references/troubleshooting.md): authentication, validation, permissions and provider errors.
