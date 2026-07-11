# Command Reference

Curated from the `maeve-cli` command surface. Verify anything uncertain with `npx maeve-cli --help` or `maeve <command> --help`.

## Contents

- Discovery
- Safe Reads
- Draft And Low-Risk Mutations
- Scheduling And Publishing
- Destructive Or Broad Actions
- Approvals And Client Reviews
- Analytics Reports
- Core Workflow Playbooks
- Common Sequences

## Discovery

Start here before any mutation:

```bash
maeve auth:status
maeve auth:login
maeve auth:whoami
maeve auth:logout
maeve workspaces:list
maeve integrations:list --workspace <workspaceId>
maeve integrations:capabilities --workspace <workspaceId> --integration <integrationId>
maeve integrations:options --workspace <workspaceId> --integration <integrationId> --key <optionKey>
```

Use MCP `get_integration_options` when connected. Use CLI `integrations:options --json <file>` as the fallback when the option key needs input, such as YouTube region, location search, or Instagram catalog/product search.

## Safe Reads

These commands read state and are usually safe to run after auth/workspace discovery:

```bash
maeve content:list --workspace <workspaceId>
maeve content:get --workspace <workspaceId> --id <contentId>
maeve content:failed-count --workspace <workspaceId>
maeve media:list --workspace <workspaceId>
maeve media:list --workspace <workspaceId> --state deleted
maeve media:get --workspace <workspaceId> --id <mediaId>
maeve media:download-url --workspace <workspaceId> --id <mediaId>
maeve media:view-url --workspace <workspaceId> --id <mediaId>
maeve media:folders:list --workspace <workspaceId> --parent-id root
maeve media:folders:get --workspace <workspaceId> --id <folderId>
maeve media:folders:path --workspace <workspaceId> --id <folderId>
maeve media:labels:list --workspace <workspaceId>
maeve taxonomy:labels --workspace <workspaceId>
maeve taxonomy:pillars --workspace <workspaceId>
maeve taxonomy:formats --workspace <workspaceId>
maeve hashtags:list --workspace <workspaceId>
maeve analytics:summary --workspace <workspaceId> --days 30
maeve analytics:health --workspace <workspaceId> --integration <integrationId>
maeve analytics:posts --workspace <workspaceId> --integration <integrationId>
maeve analytics:posts-aggregate --workspace <workspaceId>
maeve analytics:post --workspace <workspaceId> --id <contentId>
maeve analytics:demographics --workspace <workspaceId> --integration <integrationId>
maeve inbox:threads --workspace <workspaceId>
maeve inbox:messages --workspace <workspaceId> --thread <threadId>
maeve inbox:stats --workspace <workspaceId>
maeve inbox:tags --workspace <workspaceId>
maeve grid:list --workspace <workspaceId> --integration <integrationId>
maeve media:usage-history --workspace <workspaceId> --id <mediaId>
maeve media:labels:usage --workspace <workspaceId>
maeve media:labels:health --workspace <workspaceId>
maeve media:label-groups:list --workspace <workspaceId>
maeve campaigns:list --workspace <workspaceId> --include-phases
maeve campaigns:get --workspace <workspaceId> --id <campaignId>
maeve campaigns:content --workspace <workspaceId> --id <campaignId>
maeve campaigns:phases:list --workspace <workspaceId> --id <campaignId>
maeve strategy:foundation --workspace <workspaceId>
maeve strategy:goals --workspace <workspaceId>
maeve strategy:goals:metrics --workspace <workspaceId>
maeve strategy:goal --workspace <workspaceId> --id <goalId>
maeve boosts:list --workspace <workspaceId>
maeve boosts:get --workspace <workspaceId> --id <boostId>
maeve boosts:performance --workspace <workspaceId> --id <boostId>
maeve boosts:ad-accounts --workspace <workspaceId> --integration <integrationId>
maeve boosts:boosted-content-ids --workspace <workspaceId>
maeve boosts:by-content --workspace <workspaceId> --content <contentId>
```

Campaigns and strategy reads are general workspace context. Boost reads require a standard plan; the CLI exposes boost reads only, because boost writes spend ad budget.

Standard-plan reads:

```bash
maeve content:pending-approval-count --workspace <workspaceId>
maeve content:approval-history --workspace <workspaceId>
maeve content:approval-history:client-batch --workspace <workspaceId> --batch <batchId>
maeve content:approval-history:internal --workspace <workspaceId> --record <recordId>
maeve content:history --workspace <workspaceId> --id <contentId>
maeve client-reviews:open --workspace <workspaceId>
maeve client-reviews:get --workspace <workspaceId> --batch <batchId>
```

## Draft And Low-Risk Mutations

Prefer draft creation unless the user explicitly asks to schedule or publish:

```bash
maeve content:create --workspace <workspaceId> --json create-content.json
maeve content:update --workspace <workspaceId> --id <contentId> --json update-content.json
maeve content:notes --workspace <workspaceId> --id <contentId> --notes "<p>Planning notes</p>"
maeve media:upload ./image.png --workspace <workspaceId>
maeve media:update --workspace <workspaceId> --id <mediaId> --json media-update.json
maeve media:restore --workspace <workspaceId> --id <mediaId>
maeve media:bulk-restore --workspace <workspaceId> --json media-ids.json
maeve media:folders:create --workspace <workspaceId> --json media-folder.json
maeve media:folders:update --workspace <workspaceId> --id <folderId> --json media-folder.json
maeve media:folders:move --workspace <workspaceId> --id <folderId> --json media-folder-move.json
maeve media:labels:create --workspace <workspaceId> --json media-label.json
maeve media:labels:update --workspace <workspaceId> --id <labelId> --json media-label.json
maeve media:labels:attach --workspace <workspaceId> --id <mediaId> --json media-label-ids.json
maeve media:labels:detach --workspace <workspaceId> --id <mediaId> --json media-label-ids.json
maeve media:labels:archive --workspace <workspaceId> --id <labelId>
maeve media:labels:restore --workspace <workspaceId> --id <labelId>
maeve media:labels:merge --workspace <workspaceId> --id <sourceLabelId> --json media-label-merge.json
maeve media:label-groups:create --workspace <workspaceId> --json label-group.json
maeve media:label-groups:update --workspace <workspaceId> --id <groupId> --json label-group.json
maeve media:label-groups:reorder --workspace <workspaceId> --json label-group-order.json
maeve media:label-groups:add-labels --workspace <workspaceId> --id <groupId> --json label-ids.json
maeve media:label-groups:ungroup-labels --workspace <workspaceId> --json label-ids.json
maeve campaigns:create --workspace <workspaceId> --json campaign.json
maeve campaigns:update --workspace <workspaceId> --id <campaignId> --json campaign.json
maeve campaigns:phases:create --workspace <workspaceId> --id <campaignId> --json campaign-phase.json
maeve campaigns:phases:replace --workspace <workspaceId> --id <campaignId> --json campaign-phases.json
maeve campaigns:phases:update --workspace <workspaceId> --id <campaignId> --phase-id <phaseId> --json campaign-phase.json
maeve hashtags:create --workspace <workspaceId> --json hashtags.json
maeve hashtags:update --workspace <workspaceId> --id <hashtagGroupId> --json hashtags.json
maeve inbox:note --workspace <workspaceId> --thread <threadId> --json inbox-note.json
maeve inbox:tags:create --workspace <workspaceId> --name "VIP" --color "#2563eb"
maeve inbox:tags:update --workspace <workspaceId> --tag <tagId> --name "New name"
maeve inbox:tags:reorder --workspace <workspaceId> --json inbox-tag-ids.json
maeve inbox:tag-thread --workspace <workspaceId> --thread <threadId> --tag <tagId>
maeve grid:create --workspace <workspaceId> --json grid-item.json
maeve grid:update --workspace <workspaceId> --item <itemId> --json grid-update.json
maeve grid:replace-media --workspace <workspaceId> --item <itemId> --json grid-media.json
maeve grid:set-cover --workspace <workspaceId> --item <itemId> --json grid-cover.json
```

There is no CLI command to set a content workflow status (idea, drafting, and so on); that is set in the app. The CLI only filters by it, with `content:list --workflow-status` / `--workflow-statuses`.

## Scheduling And Publishing

These need explicit user intent. Scheduling also needs date, time, and timezone:

```bash
maeve content:schedule --workspace <workspaceId> --id <contentId> --scheduled-at "2026-05-01T10:00:00+10:00"
maeve content:intended-time --workspace <workspaceId> --id <contentId> --scheduled-at "2026-05-01T10:00:00+10:00"
maeve content:publish --workspace <workspaceId> --id <contentId> --yes
maeve content:published-caption --workspace <workspaceId> --id <contentId> --json published-caption.json --yes
maeve content:retry --workspace <workspaceId> --id <contentId>
```

`content:create` requires CLI `--yes` when the payload has `intent: "publish_now"`. `content:publish` requires CLI `--yes`. `content:published-caption` requires CLI `--yes` because it edits already-published provider content. `content:retry` does not require `--yes` in the CLI, but agents should still confirm because it can publish externally.

## Destructive Or Broad Actions

Confirm before running any of these:

```bash
maeve content:archive --workspace <workspaceId> --id <contentId>
maeve content:restore --workspace <workspaceId> --id <contentId>
maeve content:delete --workspace <workspaceId> --id <contentId>
maeve media:archive --workspace <workspaceId> --id <mediaId>
maeve media:delete --workspace <workspaceId> --id <mediaId>
maeve media:folders:delete --workspace <workspaceId> --id <folderId>
maeve media:labels:delete --workspace <workspaceId> --id <labelId>
maeve media:label-groups:delete --workspace <workspaceId> --id <groupId> --json label-group-delete.json
maeve media:bulk-archive --workspace <workspaceId> --json media-ids.json
maeve media:bulk-move --workspace <workspaceId> --json media-bulk-move.json
maeve media:bulk-label --workspace <workspaceId> --json media-bulk-labels.json
maeve media:bulk-unlabel --workspace <workspaceId> --json media-bulk-labels.json
maeve campaigns:archive --workspace <workspaceId> --id <campaignId>
maeve campaigns:restore --workspace <workspaceId> --id <campaignId>
maeve hashtags:delete --workspace <workspaceId> --id <hashtagGroupId>
maeve inbox:read --workspace <workspaceId> --thread <threadId>
maeve inbox:unread --workspace <workspaceId> --thread <threadId>
maeve inbox:resolve --workspace <workspaceId> --thread <threadId>
maeve inbox:reopen --workspace <workspaceId> --thread <threadId>
maeve inbox:resolve-message --workspace <workspaceId> --message <messageId>
maeve inbox:reopen-message --workspace <workspaceId> --message <messageId>
maeve inbox:tags:archive --workspace <workspaceId> --tag <tagId>
maeve inbox:tags:restore --workspace <workspaceId> --tag <tagId>
maeve grid:reorder --workspace <workspaceId> --json grid-reorder.json
maeve grid:remove-cover --workspace <workspaceId> --item <itemId>
```

`media:label-groups:delete` takes `{ "mode": "archive-labels" }` or `{ "mode": "ungroup-labels" }` to say what happens to the labels in the group.

CLI-enforced `--yes`:

```bash
maeve content:publish --workspace <workspaceId> --id <contentId> --yes
maeve content:published-caption --workspace <workspaceId> --id <contentId> --json published-caption.json --yes
maeve media:bulk-delete --workspace <workspaceId> --json media-ids.json --yes
maeve media:delete-permanent --workspace <workspaceId> --id <mediaId> --yes
maeve media:bulk-delete-forever --workspace <workspaceId> --json media-ids.json --yes
maeve campaigns:phases:delete --workspace <workspaceId> --id <campaignId> --phase-id <phaseId> --yes
maeve inbox:read-all --workspace <workspaceId> --json inbox-read-all.json --yes
maeve inbox:resolve-all --workspace <workspaceId> --json inbox-resolve-all.json --yes
maeve inbox:reply --workspace <workspaceId> --thread <threadId> --json inbox-reply.json --yes
maeve inbox:moderate --workspace <workspaceId> --message <messageId> --json inbox-moderate.json --yes
maeve inbox:retry-message --workspace <workspaceId> --message <messageId> --yes
maeve inbox:delete-failed --workspace <workspaceId> --message <messageId> --yes
maeve inbox:tags:delete --workspace <workspaceId> --tag <tagId> --yes
maeve content:recurring-occurrence:cancel --workspace <workspaceId> --occurrence <occurrenceId> --yes
maeve content:recurring-series:cancel --workspace <workspaceId> --series <seriesId> --effective-occurrence <occurrenceId> --yes
maeve grid:delete --workspace <workspaceId> --item <itemId> --yes
maeve grid:promote --workspace <workspaceId> --item <itemId> --json grid-promote.json --yes
```

Recurring series and occurrence IDs come from `recurringMetadata` on `content:list`. Cancelling a series removes every not-yet-published occurrence from the effective occurrence forward.

## Approvals And Client Reviews

Standard-plan commands:

```bash
maeve content:comment --workspace <workspaceId> --id <contentId> --json comment.json
maeve content:comment-attachments:init --workspace <workspaceId> --json attachment.json
maeve content:comment-attachments:complete --workspace <workspaceId> --attachment <attachmentId>
maeve content:comment-attachments:abort --workspace <workspaceId> --attachment <attachmentId>
maeve content:activity:react --workspace <workspaceId> --activity <activityId> --json reaction.json
maeve content:activity:unreact --workspace <workspaceId> --activity <activityId> --emoji thumbs-up
maeve content:decision --workspace <workspaceId> --id <contentId> --json decision.json
maeve content:withdraw --workspace <workspaceId> --id <contentId>
maeve content:reopen-client-review --workspace <workspaceId> --id <contentId>
maeve client-reviews:create --workspace <workspaceId> --json client-review.json
maeve client-reviews:add-post --workspace <workspaceId> --batch <batchId> --json post-id.json
maeve client-reviews:remove-post --workspace <workspaceId> --batch <batchId> --json post-id.json
maeve client-reviews:cancel --workspace <workspaceId> --batch <batchId>
maeve client-reviews:override --workspace <workspaceId> --batch <batchId> --json override.json
maeve client-reviews:update-participant --workspace <workspaceId> --batch <batchId> --json participant.json
maeve client-reviews:comment --workspace <workspaceId> --batch <batchId> --json comment.json
```

CLI-enforced `--yes` because people may be notified:

```bash
maeve content:request-approval --workspace <workspaceId> --id <contentId> --json approval.json --yes
maeve content:resubmit --workspace <workspaceId> --id <contentId> --json approval.json --yes
maeve client-reviews:send --workspace <workspaceId> --batch <batchId> --yes
maeve client-reviews:resend --workspace <workspaceId> --batch <batchId> --yes
```

## Analytics Reports

Analytics reads are safe. PDF generation is standard-plan only and requires `--yes`:

```bash
maeve analytics:report --workspace <workspaceId> --provider instagram --json analytics-report.json --output report.pdf --yes
```

## Core Workflow Playbooks

Use these playbooks when a user asks for a full workflow rather than one command.

### Workspace Discovery

Goal: identify the authenticated user and usable workspace.

1. Run `maeve auth:status`.
2. If signed out in a local human session, run `maeve auth:login`; use `MAEVE_API_KEY` for automation.
3. If signed in, run `maeve auth:whoami`.
4. Run `maeve workspaces:list`.
5. If the user did not name a workspace, choose only when there is a single clear workspace; otherwise ask which workspace to use.
6. Return workspace ID, workspace name if present, auth mode if visible, and any missing-access blocker.

### Integration Discovery

Goal: identify a connected social account and its platform requirements.

1. Run `maeve integrations:list --workspace <workspaceId>`.
2. Filter out integrations with status other than `connected` unless the task is troubleshooting.
3. Run `maeve integrations:capabilities --workspace <workspaceId> --integration <integrationId>` for the target account.
4. If any selected settings field has `optionKey`, call MCP `get_integration_options`, or run `maeve integrations:options` as the CLI fallback.
5. Return integration ID, platform, username/name, status, post types, media requirements, title requirement, max media, settings fields, and option values used.

### Media Workflow

Goal: select, upload, inspect, and organize media before content mutation.

1. Discover workspace and integration if media is for a post.
2. List existing media when the user asks to reuse assets:
   `maeve media:list --workspace <workspaceId>`.
3. Upload local files before attaching them:
   `maeve media:upload ./asset.png --workspace <workspaceId>`.
4. Inspect selected/uploaded media:
   `maeve media:get --workspace <workspaceId> --id <mediaId>`.
5. Use temporary URLs only when needed for preview/download:
   `media:view-url` or `media:download-url`.
6. Update metadata with `media:update --json`.
7. Organize with folder and label commands when requested.
8. Confirm before deletes, bulk moves/labels, or archive operations.
9. Return media IDs, filenames, media type, folder/label changes, and whether the media satisfies integration capabilities.

`media:delete` and `media:bulk-delete` move active media to the Bin and return `deleteAfter`. Scheduled-post usage blocks delete/archive operations with `MEDIA_ATTACHED_TO_SCHEDULED_POST`; bulk label toggle is composed from `media:bulk-label` and `media:bulk-unlabel`.

### Content Workflow

Goal: create or edit content safely, defaulting to draft.

1. Discover workspace and integration.
2. Run capabilities and dynamic options before platform-specific settings.
3. Inspect relevant existing content with `content:list` or `content:get` when updating.
4. Upload/select media first if needed.
5. Create new content with `content:create --json` and draft intent unless schedule/publish is explicit.
6. Update content with `content:update --id <contentId> --json <file>`; include only fields that should change.
7. Schedule only when date, time, and timezone are explicit:
   `content:schedule --scheduled-at "2026-05-01T10:00:00+10:00"`.
8. Use `content:intended-time` only when the user wants a planned publish time without scheduling.
9. Workflow status (idea, drafting, and so on) is set in the app, not the CLI; the CLI only filters by it with `content:list --workflow-status`.
10. Return content ID, status, workflow status if known, workspace, integration, scheduled time, media IDs, and next review step.

### Publishing Workflow

Goal: handle publish-now, retry, archive, restore, and delete without accidental external effects.

1. Read the content with `content:get`.
2. Confirm workspace, integration, content ID, current status, and desired action.
3. For publish-now, confirm explicit current-task intent, then run `content:publish --yes`.
4. For published Facebook caption edits, confirm the exact content ID and new caption, then run `content:published-caption --yes`.
5. For retry, confirm external retry intent, then run `content:retry`.
6. For archive/restore/delete, confirm the exact content ID and effect, then run the matching command.
7. Return resulting status or deletion marker, content ID, workspace, integration, and any provider error.

### Approval Workflow

Goal: manage internal approval and client review work in standard-plan workspaces.

1. Confirm the workspace has standard-plan access if commands fail with plan errors.
2. Inspect pending work with `content:pending-approval-count`, `content:approval-history`, `content:history`, or client-review reads.
3. Request or resubmit internal approval with `content:request-approval --yes` or `content:resubmit --yes` only after confirmation because people may be notified.
4. Submit decisions with `content:decision --json`; include `reason` for rejection or override.
5. Add comments or attachments with `content:comment`, `content:comment-attachments:init`, `complete`, or `abort`.
6. Create client review batches with `client-reviews:create`.
7. Add/remove posts, update participants, comment, override, or cancel as requested.
8. Send/resend client reviews with `--yes` only after confirmation.
9. Return content IDs, approval record IDs, batch IDs, reviewer impact, decision state, and next reviewer/client step.

### Taxonomy And Hashtags Workflow

Goal: reuse or maintain labels that shape content planning.

1. Read taxonomy before labeling content:
   `taxonomy:labels`, `taxonomy:pillars`, and `taxonomy:formats`.
2. Read hashtag groups with `hashtags:list`.
3. Create or update hashtag groups with JSON only when the user asks to manage reusable groups.
4. Confirm before deleting a hashtag group.
5. Return selected label IDs, pillar IDs, format IDs, campaign IDs, hashtag group ID, and how they map to the content payload.

### Analytics Workflow

Goal: read performance data and generate reports safely.

1. Use `analytics:summary --days <1-90|all>` for workspace or integration overview.
2. Use `analytics:health --integration <id>` for account health.
3. Use `analytics:posts --integration <id>` for a single account's post list.
4. Use `analytics:posts-aggregate --integration-ids <ids>` for cross-account comparisons.
5. Use `analytics:post --id <contentId>` for one published content item.
6. Use `analytics:demographics --integration <id>` for audience data.
7. Generate PDF reports with `analytics:report --yes` only after confirmation; report generation is standard-plan only and can be expensive.
8. Return date range, integrations included, key metric fields requested, report path if generated, and any plan/provider blocker.

### Inbox Workflow

Goal: read inbox work first, then apply replies, notes, read-state, moderation, or retries safely.

1. Read threads with `inbox:threads --workspace <workspaceId>` using filters such as `--read unread`.
2. Read messages for a selected thread with `inbox:messages --thread <threadId>`.
3. Read stats with `inbox:stats` when triaging volume.
4. Mark individual threads read/unread only after the target thread is clear.
5. Add internal notes with `inbox:note --json`.
6. Send public replies with `inbox:reply --yes` only after confirmation.
7. Moderate public content with `inbox:moderate --yes` only after confirmation.
8. Retry or delete failed outgoing messages with `--yes` only after confirmation.
9. Use `inbox:read-all --yes` only after confirming the filter and expected count/scope.
10. Return thread IDs, message IDs, public/private effect, moderation action, and retry/delete state.

### Grid Planner Workflow

Goal: manage visual planning items and promote them to content drafts.

1. List the grid for one integration:
   `maeve grid:list --workspace <workspaceId> --integration <integrationId>`.
2. Create visual-only items with uploaded media IDs and `kind: "visual_only"`.
3. Create linked-post items with `kind: "linked_post"` and `linkedContentId`.
4. Update notes or aspect ratio with `grid:update`.
5. Replace visual media with `grid:replace-media`.
6. Set or remove covers with `grid:set-cover` or `grid:remove-cover`; confirm remove-cover if the result is unclear.
7. Reorder with `grid:reorder` only when the payload contains the full item ID order for the integration.
8. Promote to a content draft with `grid:promote --yes` only after confirmation.
9. Delete with `grid:delete --yes` only after confirmation.
10. Return item IDs, integration ID, media IDs, linked content ID, order changes, promoted draft ID if present, and next planning step.

## Common Sequences

Create a draft:

```bash
maeve auth:status
maeve workspaces:list
maeve integrations:list --workspace <workspaceId>
maeve integrations:capabilities --workspace <workspaceId> --integration <integrationId>
maeve content:create --workspace <workspaceId> --json create-content.json
```

Schedule an existing draft:

```bash
maeve content:get --workspace <workspaceId> --id <contentId>
maeve integrations:capabilities --workspace <workspaceId> --integration <integrationId>
maeve content:schedule --workspace <workspaceId> --id <contentId> --scheduled-at "2026-05-01T10:00:00+10:00"
```

Upload media then create a draft:

```bash
maeve media:upload ./asset.png --workspace <workspaceId>
maeve media:get --workspace <workspaceId> --id <mediaId>
maeve content:create --workspace <workspaceId> --json create-content.json
```
