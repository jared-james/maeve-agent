# MCP operation catalog

Use this reference for hosted MCP setup, operation discovery, execution, upload, batch recovery, and connection recovery. The hosted MCP server runs with the Maeve backend. CLI npm publication and the maintained plugin package are separate releases.

## Connection

- Prefer the MCP client's own browser connection. Claude Code: run `/mcp`, choose `maeve`, and approve Maeve in the browser. Codex: run `codex mcp login maeve` and approve Maeve in the browser. Other clients: use their **Authenticate** action.
- Use `MAEVE_API_KEY` only when the client does not support browser authentication or for server-side automation.
- CLI login does not authenticate MCP. Never paste API keys, OAuth codes, tokens, presigned URLs, or raw provider payloads into chat or files.
- Confirm the API environment, organization, workspace, and integration independently. Different connections can target different environments.

## Four-tool surface

The ordinary `/mcp` surface exposes only the authorized subset of these four names:

| Tool | Purpose |
| --- | --- |
| `maeve_search` | Find a bounded set of authorized operations by intent. It returns canonical operation IDs and the required executor, not schemas. |
| `maeve_details` | Load one authorized operation's input/output schemas, permissions, availability, side effects, confirmation, retry guidance, examples, and next operations. |
| `maeve_read` | Execute one read operation as `{ operationId, input }`. |
| `maeve_write` | Execute one write or dangerous operation as `{ operationId, input }`. The selected operation still enforces its own confirmation and permissions. |

An OAuth grant without read scope can expose only `maeve_write`; a read-only grant exposes `maeve_search`, `maeve_details`, and `maeve_read`. Toolsets still authorize operations even though they no longer create top-level tool names.

Do not guess an operation schema from its canonical ID or from a CLI command. Search, load details for the chosen operation, then use the returned executor and exact input schema. A prior search or details call is guidance, not authorization.

### Discovery pattern

1. Search for workspace discovery and load details for `workspace.list`. Run it through `maeve_read` with an empty operation input.
2. Keep the selected `workspaceId`, then search again with that workspace and the user's actual intent.
3. Load `maeve_details` for the selected operation with the same workspace context.
4. If details says `available: false`, report `unavailableReason` and do not attempt execution. Resolve required plan, platform, or account conditions instead.
5. Execute through the returned `executor`, passing the canonical ID and an `input` object that matches the details schema exactly.
6. Follow returned `nextOperations` only when they advance the user's request. Load their details before execution.

Search is bounded. Refine the query when `hasMore` is true or the desired operation is absent. An operation can remain hidden because of credential scope, toolset grants, organization policy, workspace access, role capability, plan, or platform support. Do not switch credentials to bypass that result.

## Common operation IDs

These stable IDs are useful search targets. Details remains authoritative for their current schemas and executors.

| Workflow | Read operations | Write operations |
| --- | --- | --- |
| Workspace and integration context | `workspace.list`, `workspace.members.list`, `integrations.list`, `integrations.get_capabilities`, `integrations.get_options`, `taxonomy.list`, `hashtag_groups.list`, `campaigns.list`, `grid.items.list` | `integrations.create_pinterest_board` |
| Content | `content.root.list`, `content.root.get`, `content.list`, `content.get`, `content.get_counts` | `content.root.update`, `content.create_draft`, `content.create_drafts`, `content.update_draft`, `content.update_metadata`, `content.update_notes`, `content.set_intended_publish_time`, `content.schedule`, `content.schedule_batch`, `content.revert_to_draft`, `content.publish_now`, `content.retry`, `content.update_published_caption`, `content.resolve_publishing_result`, `content.delete` |
| Media Room | `media.list`, `media.get`, `media.get_usage_history`, `media.list_folders`, `media.get_folder`, `media.get_folder_path`, `media.list_labels` | `media.inspect_audio`, `media.upload`, `media.complete_upload`, `media.upload_batch`, `media.complete_upload_batch`, `media.folder.create`, `media.folder.update`, `media.folder.move`, `media.label.create`, `media.labels.attach`, `media.labels.detach`, `media.update`, `media.organize`, `media.delete`, `media.restore`, `media.folder.delete` |
| Review | `workspace.members.list` | `content.review.request_internal`, `content.review.withdraw`, `client_review.batch.create`, `client_review.batch.remove_content`, `client_review.batch.send` |
| Tasks | `task_board.list`, `task.get`, `task.list_for_post` | `task.create`, `task.update`, `task.move`, `task.create_comment`, `task.create_checklist_item`, `task.update_checklist_item`, `task.move_checklist_item`, `task.delete_checklist_item`, `task.delete` |
| Workbench content tables | `workbench.page.list`, `workbench.content_table.get`, `workbench.content_table.list_rows` | `workbench.page.create`, `workbench.content_table.add_row`, `workbench.content_table.set_row_cells`, `workbench.content_table.remove_row` |
| Calendar | `calendar.list`, `calendar.notes.list`, `calendar.snapshots.list`, `calendar.snapshot.preview` | `calendar.note.create`, `calendar.note.update`, `calendar.note.move`, `calendar.note.delete`, `calendar.snapshot.create`, `calendar.snapshot.regenerate`, `calendar.snapshot.revoke` |
| Analytics | `analytics.get_summary`, `analytics.get_aggregate`, `analytics.get_health`, `analytics.get_demographics`, `analytics.get_posts`, `analytics.get_posts_aggregate`, `analytics.get_post` | None |
| Inbox automation | `inbox_automation.list_rules`, `inbox_automation.get_rule`, `inbox_automation.list_executions` | `inbox_automation.create_rule`, `inbox_automation.update_rule`, `inbox_automation.delete_rule` |

Strategy operations use the `strategy.*` namespace. Search for the requested Foundation, platform, Goal, Bet, prediction, progress, or Retro workflow instead of loading the whole namespace.

`content.create_draft` can place new content directly on a table with `workbenchPageId` and optional `sortOrder`. If placement fails after creation, keep the returned `contentId` and finish with `workbench.content_table.add_row`; do not create replacement content. Row cell replacement and removal require the latest row `version`. Re-read after a stale-version conflict. Removing a row requires exact confirmation and leaves the content intact.

Live inbox messaging, approval decisions, ads, billing, credentials, integration connect/disconnect, recurring cancellation, and PDF reports remain outside the MCP catalog. Use the CLI or public API only where they support the task and the user has authorized the effect.

## Confirmation

`maeve_write` is a generic executor, not a blanket confirmation. Inspect `maeve_details.confirmation` and `sideEffects` for the selected operation.

- Ask for authorization when intent, destination, audience, or external effect is missing. Existing authorization for the same action and targets remains valid.
- When confirmation is required, pass `confirmationText` inside the operation's `input`. Use the exact value described by details or returned by a `confirmation_required` error. Never construct a broader token or substitute another resource ID.
- A preview-first operation such as `media.organize` must be previewed against the final bounded selection. Commit only with the exact confirmation returned for that preview.
- Never assume dangerous auto-confirm is available. If details says the action never auto-confirms, exact text is mandatory. A server policy cannot make an irreversible action safe to replay.
- Review decisions remain human actions. Creating or sending a review request can notify people and still needs the user's intended recipients and content scope.

## Upload sequence

MCP uploads require both operation calls and an external byte transfer:

1. Load details for `media.upload`, then call it through `maeve_write` with file name, MIME type, size, and workspace context.
2. Keep the returned upload session fields private. PUT the exact local file bytes to the returned URL before expiry, using the required headers.
3. Load details for `media.complete_upload`, then execute it through `maeve_write` with the returned session identity.
4. Read `media.get` before attaching the new media ID to content.

If the current client cannot read the local file or make the PUT request, use `maeve media:upload` instead. The four MCP tools do not add file-system or arbitrary HTTP capability. Never report an initialized upload as completed. Replaying initialization does not renew an expired URL; inspect the current session and create a new upload session only when the old transfer can no longer complete.

## Batch operations

`content.create_drafts` and `content.schedule_batch` accept up to 100 items. `media.upload_batch` and `media.complete_upload_batch` accept up to 50. Load details for the exact operation before constructing a batch.

- Keep the batch `idempotencyKey`, item order, and payload unchanged for an identical replay.
- Treat results independently by `index`. Retain every successful resource ID and per-item replay key even when neighbours fail.
- Replaying the same key and manifest returns the recorded successes and failures. It does not make a failed item succeed or renew upload URLs.
- Corrected input needs a new item or batch key. Reusing a key with changed input is a conflict.
- Media upload batches still require one byte PUT per initialized item before completion.
- Do not schedule or publish a draft batch merely to verify draft creation.

## Recovery

- For validation errors, use `details.validationErrors`, correct only the rejected field, and preserve successful IDs.
- For `rate_limited`, wait for `retryAfterSeconds` when present. Do not switch connections or create replacement content.
- For conflict or uncertain mutation results, follow the operation's details retry contract. Read the affected resource before retrying an idempotent or unsafe write.
- A queued publish is not a native publication. Do not retry or create replacement content while the provider outcome is uncertain.
- If an MCP session is lost or expired, initialize a fresh session, reload the four-tool declaration, and continue from persisted Maeve IDs. There is no activation state to rebuild, and session loss does not erase saved content or media.
- Authentication loss uses the client's reconnect flow. CLI login cannot repair MCP OAuth.

## Returned links

Report the affected IDs, workspace, integration or destination, status, and scheduled time when relevant.

- Label an exact returned `appUrl` as **Open in Maeve**. Do not construct one from slugs or IDs.
- After publication, read `content.get`. Label a non-null HTTPS `permalink` from that read as **View on platform**.
- A null permalink is a valid result. Never synthesize a provider URL from `platformPostId`, a resource name, or prior URL patterns.
- A queued response, sent status, or provider ID is not proof of native visibility. Verify the selected author, rendered content, and resource identity separately when the task requires it.

## Distribution boundary

The maintained public plugin source is the separate `jared-james/maeve-agent` repository, under `skills/maeve-social-scheduler/`. The backend's mirrored skill is the application-maintained source for this repository only. Installed Codex, Claude, or ChatGPT plugin caches are deployment artifacts and must not be edited as source.

Updating the backend server, this mirrored skill, the public plugin repository, and a ChatGPT connection are separate release steps. After a hosted MCP metadata change, refresh a developer-mode ChatGPT connection, confirm the advertised metadata, and start a new conversation. A published ChatGPT plugin uses a reviewed metadata snapshot and needs a scanned, submitted, approved, and published new version. See the [official OpenAI connector refresh process](https://developers.openai.com/plugins/deploy/connect-chatgpt#refresh-metadata).
