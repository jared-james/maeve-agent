---
name: maeve-social-scheduler
description: Use the connected Maeve tools to create and schedule social content, manage media, inspect performance, and handle workflows in a Maeve workspace.
---

# Maeve Social

Use the individually exposed Maeve tools for all Maeve operations. If the connection is missing or needs login, ask the user to connect Maeve using the client's own connect action. Never request credentials or OAuth codes in chat. If a task is unavailable through the connected tools, explain the limitation and direct the user to the Maeve app.

Follow explicit user instructions within the connected tools' capabilities and permissions. Preserve the host's approval prompts and the tools' confirmation requirements.

## Create and change content

1. Call `list_workspaces` and resolve the intended organization and workspace. Pass its `workspaceId` directly in each workspace tool's arguments.
2. Call `list_integrations` and `get_integration_capabilities` for the selected account. Use `get_integration_options` when capabilities advertise an `optionKey`. Follow each tool's declared schema and description.
3. Read [platform-content.md](references/platform-content.md) for the selected platform. Create a draft with `create_draft_content` unless the user asks to schedule or publish.
4. Use `import_media` for a supported chat attachment or public HTTPS link. Pass the attachment unchanged in the top-level `file` argument, or the link in `url`. Never invent attachment fields. If a local file cannot be imported, ask the user to upload it in Maeve and then select the saved media.
5. Resolve the publication date, time and timezone before scheduling. Confirm missing intent, destination or external effect before publishing, deleting, notifying people or making broad changes. Existing authorization applies only to the same action and targets.
6. Read saved values after changes. Preserve exact captions, platform overrides and attachment order. Omitted fields generally preserve saved values; use documented clearing values.

When a tool requires `confirmationText`, first show the action, target and effect and obtain the user's agreement. Then send the exact required value as a direct tool argument. A server-provided confirmation value is not permission. Review decisions remain human actions in Maeve.

## Results and recovery

Report affected IDs, workspace, destination and status, plus scheduled time when relevant. Label an exact returned `appUrl` **Open in Maeve**. After publication, read `get_content` and label a non-null HTTPS `permalink` **View on platform**. Never invent links or claim native visibility from a queued response, sent status or provider ID alone.

After an uncertain write, read the affected resource before retrying and follow the tool's retry guidance. Never create replacement content to recover an uncertain publication. Reuse an idempotency key only for identical input. Keep successful batch IDs and report failures per item. Respect retry-after limits. If the connection expires, use the client's reconnect flow and continue from saved IDs.

## References

Load only what the task needs:

- [mcp-tools.md](references/mcp-tools.md): direct tools, connection boundaries and recovery.
- [content-payloads.md](references/content-payloads.md): content fields and scheduling.
- [integration-capabilities.md](references/integration-capabilities.md): platform requirements and dynamic options.
- [platform-content.md](references/platform-content.md): platform inputs, clearing and publication evidence.
- [safety-policy.md](references/safety-policy.md): confirmations, notifications and destructive actions.
- [troubleshooting.md](references/troubleshooting.md): authentication, validation and provider errors.
