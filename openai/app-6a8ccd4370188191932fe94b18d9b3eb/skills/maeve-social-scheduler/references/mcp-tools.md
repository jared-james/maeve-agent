# Maeve tools for OpenAI

This plugin connects to `/mcp/openai`. Each supported operation is an individually declared tool with its own input schema, output schema, description and safety annotations. Use those definitions directly. Tool availability depends on the connected account's scopes and policy; workspace permissions and confirmation are checked on every call.

## Workflow

1. Call `list_workspaces` to select the intended organization and workspace.
2. Pass `workspaceId` directly in each workspace tool's arguments.
3. Use `list_integrations`, `get_integration_capabilities` and `get_integration_options` to resolve the account and supported platform settings.
4. Select the individually exposed tool for the user's task and follow its declared input schema and side effects.
5. Read saved values after mutations. Keep successful IDs and do not invent replacement records after an uncertain result.

If a required tool is unavailable, report the limitation and direct the user to the Maeve app. Do not change credentials to bypass a permission or availability result.

## Confirmation

Obtain the user's agreement to the specific action, target and external effect before sending required `confirmationText`. The server's expected confirmation value is not permission. Preserve the host's approval prompts. Preview operations against the final bounded selection and use only the confirmation returned for that preview. Review decisions remain human actions in Maeve.

## Media

Call `import_media` with `workspaceId` and either `url` for a public HTTPS link or `file` for a supported chat attachment. Pass the supplied attachment object unchanged. Read `get_media` before attaching the saved media ID to content.

For local files unavailable as supported attachments, ask the user to upload them in Maeve and select the resulting media with `list_media` and `get_media`. Do not initialize upload sessions that require byte transfers outside the connected tools. Never report an initialized upload as completed.

## Recovery

- Correct only the fields in `details.validationErrors` and preserve the rest of the user's input.
- Respect `retryAfterSeconds`. Do not switch connections or create replacement content to evade rate limits.
- Read current state before retrying a conflict or uncertain write. Follow each tool's retry guidance.
- Reuse an idempotency key only with the same payload. Keep successful batch IDs and report failures per item.
- Do not retry `plan_required`, `forbidden` or `not_found` errors.
- Use the client's reconnect flow after authentication or session loss. Continue from saved IDs.
- Queued is not published. A sent status or saved provider ID alone does not establish native visibility.

## Results

Report affected IDs, workspace, destination and status, plus scheduled time when relevant. Use exact returned `appUrl` and HTTPS `permalink` values for links. A null permalink is valid; never synthesize one.
