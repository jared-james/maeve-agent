---
name: maeve-social-scheduler
description: Manage content, media, and analytics in a Maeve workspace through the connected MCP tools. Use for workspace operations, not Maeve code development.
---

# Maeve Social

## Use the connected MCP tools

Use the connected Maeve MCP catalog for all Maeve operations. If the connection is missing or needs login, ask the user to connect Maeve through the client's own connect action. Do not request OAuth codes or tokens in chat. If an operation is unavailable, explain the limitation and direct the user to the Maeve app.

Verify the connected environment, organization, workspace and integration before reads or writes. Keep credentials and signed URLs out of output.

## Create and change content

1. Use `maeve_search` to discover `workspace.list`, execute it with `maeve_read`, then search again with the selected workspace. Load `maeve_details` for every chosen operation before execution. Read the connected integration's live capabilities and fetch dynamic options when a capability includes `optionKey`.
2. Read the selected platform in [platform-content.md](references/platform-content.md). Use the live operation schema returned by `maeve_details`.
3. Create a draft unless the user asked to publish or schedule. Execute `content.create_draft` through `maeve_write`. Import supported chat attachments or public HTTPS media links with `media.import` before attaching the returned IDs. If a file cannot be imported through the connected tools, ask the user to upload it in the Maeve app.
4. Resolve the publication date, time and timezone before scheduling. Confirm any missing intent, destination or external effect before publishing, deleting, sending messages or making broad changes. Existing authorization applies to the same action and targets; do not repeatedly ask for it.
5. Read saved values after changes. Preserve exact captions, per-platform overrides and attachment order. Use documented clearing values; omitted fields generally preserve existing values.

For a confirmation-gated MCP operation, put the exact required `confirmationText` inside the `maeve_write` operation input only when the user authorized that action and target. Review decisions remain human actions; requesting a review may notify people. See [safety-policy.md](references/safety-policy.md) for action-specific requirements.

## Results and recovery

Report the affected IDs, workspace, destination and status, plus scheduled time when relevant. Label the returned `appUrl` **Open in Maeve**; never invent app URLs. Return a non-null HTTPS `permalink` from a `content.get` read as **View on platform**.

A queued response, sent status or provider ID is not proof of native visibility. When the task requires live verification, check the provider author, rendered content and resource identity separately. Supported provider deletion can retain the sent audit in Maeve.

For a repeated create, retain successful IDs and reuse the identical payload and idempotency key. Inspect conflicts before changing input. Respect retry-after limits; do not switch connections or create replacement content to recover uncertain publishing outcomes. A lost MCP session requires fresh initialization and a fresh four-tool declaration, not activation or recreation of saved content.

## References

Load only what the task needs:

- [mcp-tools.md](references/mcp-tools.md): connection, catalog discovery, execution, upload, batches, recovery and distribution boundaries.
- [content-payloads.md](references/content-payloads.md): JSON shapes and scheduling.
- [integration-capabilities.md](references/integration-capabilities.md): limits and dynamic options.
- [platform-content.md](references/platform-content.md): platform inputs, clearing and publication evidence.
- [safety-policy.md](references/safety-policy.md): notifications, destructive actions and confirmation fields.
- [troubleshooting.md](references/troubleshooting.md): authentication, validation, permissions and provider errors.
