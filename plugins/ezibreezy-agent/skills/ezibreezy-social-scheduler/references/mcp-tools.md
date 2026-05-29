# MCP Tools

Use this reference only when the current agent environment already has the hosted EziBreezy MCP server connected as `ezibreezy`.

Hosted endpoint:

```text
https://api.ezibreezy.com/mcp
```

Authentication:

- Preferred: use your MCP client's **Authenticate** flow. It opens EziBreezy in the browser and asks the user to approve access. This has been validated with Codex and Claude Code.
- Browser-approved MCP access uses 6-hour access tokens with 90-day rotating refresh tokens. Organization admins can revoke active AI tool connections in EziBreezy **Settings -> Developer**.
- Fallback: use `EZIBREEZY_API_KEY` only when the MCP client does not support browser authentication, or for server-side automation.

CLI login is separate. `ezibreezy auth:login` signs in the CLI, not MCP.

Do not paste API keys, OAuth codes, bearer tokens, refresh tokens, presigned URLs, or provider payloads into chat or project files.

## Scope: MCP is the posting surface

MCP exposes only the tools an agent needs to **compose, organize, schedule, and publish content**, plus the reads needed to do that well (posting context and performance). It is a deliberate, curated slice of the public REST API — not the whole product.

Inbox, approvals/client review, boost ads, API keys, integration connect/disconnect, grid/workbench mutations, and PDF reports are intentionally **not** MCP tools. They remain available through the EziBreezy app, the public REST API, and (where supported) the `@ezibreezy/cli`. If a task needs one of those, fall back to the CLI or public API under the normal safety policy. See `Not In MCP` below.

## Operating Order

1. Prefer connected MCP tools for agent-native reads and supported low-risk mutations.
2. Fall back to `@ezibreezy/cli` when MCP is not connected or the workflow is outside the posting surface (inbox, approvals, etc.).
3. Use the public API directly only when both MCP and CLI are insufficient.
4. Keep all normal safety rules: draft-first, explicit timezone for scheduling, capability inspection before platform-specific settings, and confirmation before externally visible, destructive, or broad actions.

## Tool Map

| MCP tool                        | CLI fallback                                                                                                     | Notes                                                                                                                                                                                                              |
| ------------------------------- | ---------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `list_workspaces`               | `ezibreezy workspaces:list`                                                                                      | Read-only workspace discovery.                                                                                                                                                                                     |
| `list_integrations`             | `ezibreezy integrations:list --workspace <workspaceId>`                                                          | Read-only connected account list.                                                                                                                                                                                  |
| `get_integration_capabilities`  | `ezibreezy integrations:capabilities --workspace <workspaceId> --integration <integrationId>`                    | Required before platform-specific content settings.                                                                                                                                                                |
| `get_integration_options`       | `ezibreezy integrations:options --workspace <workspaceId> --integration <integrationId> --key <optionKey>`       | Fetches dynamic option keys returned by capabilities, such as boards, creator info, locations, catalogs, products, or YouTube categories.                                                                          |
| `list_content`                  | `ezibreezy content:list --workspace <workspaceId>`                                                               | Read-only public content statuses.                                                                                                                                                                                 |
| `get_content`                   | `ezibreezy content:get --workspace <workspaceId> --id <contentId>`                                               | Read-only sanitized content detail.                                                                                                                                                                                |
| `get_content_counts`            | `ezibreezy content:failed-count --workspace <workspaceId>`, `content:pending-approval-count`                     | Read-only failed content count, plus pending approval count when requested on an Agency or Scale workspace plan.                                                                                                   |
| `list_campaigns`                | Public API fallback only.                                                                                        | Read-only first-class workspace campaigns and optional phases. Campaign status is derived from dates. Assign content to a campaign via `update_content_metadata`.                                                  |
| `list_taxonomy`                 | `ezibreezy taxonomy:labels --workspace <workspaceId>`, `taxonomy:pillars`, `taxonomy:formats`                    | Read-only labels, pillars, and formats for planning.                                                                                                                                                               |
| `list_hashtag_groups`           | `ezibreezy hashtags:list --workspace <workspaceId>`                                                              | Read-only saved hashtag groups.                                                                                                                                                                                    |
| `list_grid_items`               | `ezibreezy grid:list --workspace <workspaceId> --integration <integrationId>`                                    | Read-only grid planner items. Does not promote, reorder, or delete.                                                                                                                                                |
| `list_media`                    | `ezibreezy media:list --workspace <workspaceId>`                                                                 | Read-only media discovery with safe metadata and filters.                                                                                                                                                          |
| `get_media`                     | `ezibreezy media:get --workspace <workspaceId> --id <mediaId>`                                                   | Read-only safe media metadata.                                                                                                                                                                                     |
| `get_media_usage_history`       | `ezibreezy media:usage-history --workspace <workspaceId> --id <mediaId>`                                         | Read-only sanitized usage history for one Media Room asset. Actor emails, raw snapshots, and provider payloads are not exposed.                                                                                    |
| `list_media_folders`            | `ezibreezy media:folders:list --workspace <workspaceId>`                                                         | Read-only media folder discovery.                                                                                                                                                                                  |
| `get_media_folder`              | `ezibreezy media:folders:get --workspace <workspaceId> --id <folderId>`                                          | Read-only safe media folder detail.                                                                                                                                                                                |
| `get_media_folder_path`         | `ezibreezy media:folders:path --workspace <workspaceId> --id <folderId>`                                         | Read-only safe media folder ancestor path.                                                                                                                                                                         |
| `list_media_labels`             | `ezibreezy media:labels:list --workspace <workspaceId>`                                                          | Read-only media label discovery.                                                                                                                                                                                   |
| `get_analytics_summary`         | `ezibreezy analytics:summary --workspace <workspaceId> --days 30`                                                | Read-only sanitized metrics. Does not generate PDF reports.                                                                                                                                                        |
| `get_analytics_aggregate`       | `ezibreezy analytics:summary --workspace <workspaceId> --days 30`                                                | Read-only sanitized aggregate metrics across selected integrations. Does not generate PDF reports.                                                                                                                 |
| `get_analytics_health`          | `ezibreezy analytics:health --workspace <workspaceId> --integration <integrationId>`                             | Read-only sanitized integration analytics health.                                                                                                                                                                  |
| `get_analytics_posts`           | `ezibreezy analytics:posts --workspace <workspaceId> --integration <integrationId>`, `analytics:posts-aggregate` | Read-only sanitized post analytics for one integration or an aggregate set.                                                                                                                                        |
| `get_analytics_posts_aggregate` | `ezibreezy analytics:posts-aggregate --workspace <workspaceId> --days 30`                                        | Read-only sanitized aggregate post analytics across selected integrations.                                                                                                                                         |
| `get_post_analytics`            | `ezibreezy analytics:post --workspace <workspaceId> --id <contentId>`                                            | Read-only sanitized analytics for a single published content item.                                                                                                                                                 |
| `get_analytics_demographics`    | `ezibreezy analytics:demographics --workspace <workspaceId> --integration <integrationId>`                       | Read-only sanitized audience demographics.                                                                                                                                                                         |
| `create_draft_content`          | `ezibreezy content:create --workspace <workspaceId> --json create-content.json`                                  | MCP forces draft behavior and uses canonical content fields such as `captions`, `internalTitle`, `publishTitle`, and `contentMedia`. CLI payload should omit `intent` or set `intent: "draft"`.                    |
| `update_content_metadata`       | Public API fallback only.                                                                                        | Updates planning metadata (pillars, formats, labels, priority, assignees, campaign/phase) on a draft or scheduled item. Does not edit captions or media.                                                           |
| `update_content_notes`          | `ezibreezy content:notes --workspace <workspaceId> --id <contentId> --notes <html>`                              | Replaces the rich-text notes attached to a content item. Notes are internal - never published. Pass an empty string to clear.                                                                                      |
| `set_intended_publish_time`     | `ezibreezy content:intended-time --workspace <workspaceId> --id <contentId> --scheduled-at <isoWithTimezone>`    | Sets a planned publish time for draft or pending approval content. Does not schedule, publish, or create a queue job.                                                                                              |
| `schedule_content`              | `ezibreezy content:schedule --workspace <workspaceId> --id <contentId> --scheduled-at <isoWithTimezone>`         | Requires explicit ISO 8601 timezone such as `2026-05-01T10:00:00+10:00`.                                                                                                                                           |
| `upload_media`                  | `ezibreezy media:upload <file> --workspace <workspaceId>`                                                        | MCP creates an upload session; client uploads bytes, then calls `complete_media_upload`.                                                                                                                           |
| `complete_media_upload`         | `ezibreezy media:upload <file> --workspace <workspaceId>`                                                        | Finalizes an MCP upload session.                                                                                                                                                                                   |
| `update_media`                  | `ezibreezy media:update --workspace <workspaceId> --id <mediaId> --json media-update.json`                       | Updates safe editable metadata for one Media Room asset: filename, alt text, notes, favorite state, or folder assignment.                                                                                          |
| `create_media_folder`           | `ezibreezy media:folders:create --workspace <workspaceId> --json media-folder.json`                              | Creates a Media Room folder. Supports root folders and one level of subfolders.                                                                                                                                    |
| `update_media_folder`           | `ezibreezy media:folders:update --workspace <workspaceId> --id <folderId> --json media-folder.json`              | Renames one Media Room folder.                                                                                                                                                                                     |
| `move_media_folder`             | `ezibreezy media:folders:move --workspace <workspaceId> --id <folderId> --json media-folder-move.json`           | Moves one Media Room folder to root or under another root folder. Deeper nesting is rejected.                                                                                                                      |
| `create_media_label`            | `ezibreezy media:labels:create --workspace <workspaceId> --json media-label.json`                                | Creates a Media Room label. Internal organization only.                                                                                                                                                            |
| `attach_media_labels`           | `ezibreezy media:labels:attach --workspace <workspaceId> --id <mediaId> --json media-label-ids.json`             | Attaches one or more labels to one media asset.                                                                                                                                                                    |
| `detach_media_labels`           | `ezibreezy media:labels:detach --workspace <workspaceId> --id <mediaId> --json media-label-ids.json`             | Detaches one or more labels from one media asset.                                                                                                                                                                  |
| `publish_now`                   | `ezibreezy content:publish --workspace <workspaceId> --id <contentId> --yes`                                     | High-risk external action. Requires `confirmationText: "publish_now:<contentId>"` or server-enabled dangerous auto-confirm.                                                                                        |
| `retry_content`                 | `ezibreezy content:retry --workspace <workspaceId> --id <contentId>`                                             | High-risk content lifecycle action. Requires `confirmationText: "retry_content:<contentId>"` or server-enabled dangerous auto-confirm.                                                                             |
| `archive_content`               | `ezibreezy content:archive --workspace <workspaceId> --id <contentId>`                                           | High-risk content lifecycle action. Requires `confirmationText: "archive_content:<contentId>"` or server-enabled dangerous auto-confirm.                                                                           |
| `restore_content`               | `ezibreezy content:restore --workspace <workspaceId> --id <contentId>`                                           | High-risk content lifecycle action. Requires `confirmationText: "restore_content:<contentId>"` or server-enabled dangerous auto-confirm.                                                                           |
| `delete_content`                | `ezibreezy content:delete --workspace <workspaceId> --id <contentId>`                                            | High-risk content lifecycle action. Requires `confirmationText: "delete_content:<contentId>"` or server-enabled dangerous auto-confirm.                                                                            |
| `organize_media_assets`         | `ezibreezy media:bulk-move`, `media:bulk-label`, `media:bulk-unlabel --workspace <workspaceId>`                  | High-risk preview-first bulk organization: select up to 100 assets, move them to a folder/root, and bulk attach/detach labels. Commit requires `confirmationText: "organize_media_assets:<operationKey>:<count>"`. |

## High-Risk Confirmation

High-risk MCP tools require exact confirmation text unless server-side dangerous auto-confirm is enabled for that action.

- `publish_now`: pass `confirmationText` as `publish_now:<contentId>`.
- `retry_content`: pass `confirmationText` as `retry_content:<contentId>`.
- `archive_content`: pass `confirmationText` as `archive_content:<contentId>`.
- `restore_content`: pass `confirmationText` as `restore_content:<contentId>`.
- `delete_content`: pass `confirmationText` as `delete_content:<contentId>`.
- `organize_media_assets`: pass `confirmationText` as `organize_media_assets:<operationKey>:<count>` (the `operationKey` and `count` are returned by the tool's preview mode).
- `dangerouslyAutoConfirm: true` is only honored when server policy allows the action, such as `EZIBREEZY_MCP_DANGEROUS_AUTO_CONFIRM_ACTIONS=archive_content,restore_content,delete_content,retry_content`. `publish_now` and `organize_media_assets` are never auto-confirmable and always require exact confirmation text.
- Tool input alone must not be treated as user permission. If confirmation is missing, ask the user to confirm the exact action and target.

## Not In MCP

These are deliberately **not** MCP tools. Use the CLI or public API, under the safety policy:

- **Inbox** — listing threads/messages/stats, replies, moderation, resolve/reopen, read-state, retry, delete-failed, notes, and bulk read-all. Inbox is a separate community-management surface, not posting.
- **Approvals & client review** — content/approval history reads, requesting/deciding/withdrawing/resubmitting approvals, comments, and the full client-review batch lifecycle including send. Approvals are human sign-off actions.
- **Analytics report generation** — PDF reports (binary output). Use the analytics read tools instead.
- **Boost / paid ads** — spends real ad budget.
- **API keys, integration connect/disconnect, workspace/member/billing administration** — credential and account management.
- **Grid planner / workbench mutations** — visual composition surfaces.
- **Destructive or broad media operations beyond `organize_media_assets`** — permanent deletion, bin, bulk delete/archive.

## Example Sequence

Create a draft through MCP when connected:

1. `list_workspaces`
2. `list_integrations` with `workspaceId`
3. `get_integration_capabilities` with `workspaceId` and `integrationId`
4. `get_integration_options` if capabilities returns a required or useful `optionKey`
5. `list_media`, `list_media_folders`, `get_media_folder`, `get_media_folder_path`, or `list_media_labels` if existing assets need to be selected
6. `create_draft_content` with `workspaceId`, `integrationId`, `captions`, and optional `internalTitle`, `publishTitle`, supported `contentMedia`, and provider-only `settings`

Create and apply a media label through MCP:

1. `list_workspaces`
2. `list_media_labels` with `workspaceId` and `search` to avoid duplicate labels
3. `create_media_label` if the label does not already exist
4. `attach_media_labels` with the target `mediaId` and `labelIds`
5. `get_media` to verify the label is attached

Schedule an existing draft through MCP:

1. `get_content`
2. `get_integration_capabilities`
3. `schedule_content` with `scheduledAt` including explicit timezone
