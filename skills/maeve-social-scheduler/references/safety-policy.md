# Safety policy

Use this policy before Maeve mutations. Authorization already given for the same action, targets and external effect remains valid. Ask only when it is missing or those details change.

## Default policy

- Prefer drafts. Omit `intent` or set `intent: "draft"` for new content unless the user explicitly asks to schedule or publish.
- Prefer read-only discovery before mutation: auth, workspace, integration, capabilities, options, and relevant current state.
- Never schedule without explicit date, time, and timezone.
- Never publish immediately from ambiguous wording such as "post this" unless the user clearly means publish now.
- Never use `--yes` on behalf of the user until the current task includes explicit confirmation for the exact action.

## Non-negotiable agent rules

- Default ambiguous content creation to draft intent.
- Never schedule without an explicit date, time, and timezone in the current task or user-provided source of truth.
- Never immediate-publish without explicit publish-now intent in the current task.
- Never send inbox replies, moderation actions, approval notifications, client review emails, or bulk read-all without confirmation.
- Never create or enable an inbox auto-reply rule without confirming the rule, account, audience, reply content, and expected external effect.
- Never add a task comment containing member mentions without confirming the mentioned recipients and notification effect.
- Never delete content, failed inbox messages, grid items, media, media folders/labels, or hashtag groups without confirmation.
- Always inspect integration capabilities before setting platform-specific fields.
- Always inspect dynamic options when a capability field includes `optionKey`.
- Always upload local media through Maeve before attaching it to posts.
- Always return created/changed IDs and status after mutations.
- Treat API keys, CLI login tokens, bearer tokens, refresh tokens, presigned URLs, and raw provider payloads as sensitive.

## Confirmation required

Ensure the user has authorized the action and target before:

- `content:publish`, `content:published-caption`, `content:retry`, `content:resolve-publishing-result`, `content:revert-to-draft`, `content:delete`.
- `content:recurring-occurrence:cancel`, `content:recurring-series:cancel`.
- `content:schedule` if the requested date/time/timezone or target integration is ambiguous.
- `content:request-approval`, `content:resubmit`, `content:withdraw`, `content:reopen-client-review`.
- `client-reviews:send`, `client-reviews:resend`, `client-reviews:cancel`, `client-reviews:override`.
- `inbox:reply`, `inbox:moderate`, `inbox:retry-message`, `inbox:delete-failed`, `inbox:read-all`, `inbox:resolve-all`, `inbox:tags:delete`.
- MCP operation `inbox_automation.create_rule` with `enabled: true` requires `inbox_automation_enable:new:<integrationId>`. MCP operation `inbox_automation.update_rule` with `enabled: true` requires `inbox_automation_enable:<ruleId>`. Execute either through `maeve_write`. Neither action supports dangerous auto-confirm.
- MCP operation `task.create_comment` through `maeve_write` when its content mentions workspace members.
- `tasks:delete` and `tasks:checklist:delete`.
- `media:delete`, `media:delete-permanent`, `media:bulk-delete`, `media:bulk-delete-forever`, `media:bulk-move`, `media:bulk-label`, `media:bulk-unlabel`.
- `media:folders:delete`, `media:labels:delete`, `media:label-groups:delete`, `hashtags:delete`.
- `integrations:pinterest:create-board`, `campaigns:delete`, `calendar:notes:delete`, and public Calendar snapshot creation, regeneration or revocation.
- `strategy:platform:start-version`, `strategy:platform:remove`, `strategy:goal:delete`, `strategy:bet:delete`, and `strategy:bet:resolve`.
- `grid:delete`, `grid:promote`, `grid:reorder`, `grid:remove-cover`.
- `analytics:report`.

The confirmation prompt should include action, IDs, workspace, integration, scheduled time if any, recipient/reviewer impact if any, and whether the effect is public, destructive, or broad. Do not infer confirmation from older conversation context if the target IDs, integration, or effect changed.

## CLI-enforced `--yes`

Only append `--yes` after confirmation for:

- `content:create` when the payload has `intent: "publish_now"`
- `content:request-approval`
- `content:resubmit`
- `content:publish`
- `content:published-caption`
- `content:resolve-publishing-result`
- `content:recurring-occurrence:cancel`
- `content:recurring-series:cancel`
- `client-reviews:send`
- `client-reviews:resend`
- `campaigns:phases:delete`
- `integrations:pinterest:create-board`
- `calendar:notes:delete`
- `calendar:snapshots:revoke`
- `strategy:goal:delete`
- `strategy:bet:delete`
- `strategy:bet:resolve`
- `analytics:report`
- `inbox:read-all`
- `inbox:resolve-all`
- `inbox:reply`
- `inbox:moderate`
- `inbox:retry-message`
- `inbox:delete-failed`
- `inbox:tags:delete`
- `media:bulk-delete`
- `media:delete-permanent`
- `media:bulk-delete-forever`
- `tasks:delete`
- `tasks:checklist:delete`
- `grid:delete`
- `grid:promote`

Some risky commands do not require `--yes`; the agent still must confirm them.

## Credential handling

- Prefer `MAEVE_API_KEY` and `MAEVE_API_URL` environment variables for automation.
- For hosted MCP, prefer the MCP client's browser authentication flow. Browser-authenticated MCP has been validated with Codex and Claude Code.
- Use `MAEVE_API_KEY` for MCP only when browser authentication is not supported or for server-side automation.
- Do not use stored browser CLI tokens as the MCP auth path.
- Browser-approved MCP grants can be revoked in Maeve **Settings -> Developer**.
- Prefer browser login for local human sessions.
- Do not ask the user to paste API keys, CLI login tokens, OAuth codes, bearer tokens, refresh tokens, presigned URLs, provider tokens, scopes, or raw provider payloads into chat.
- Do not print token values from files or command output.
- Treat media view/download URLs as sensitive, especially if presigned or temporary.
- If a command output includes sensitive values, summarize the non-sensitive state instead of echoing the raw payload.

## External visibility

Public or externally visible actions include publishing content, retrying failed publishes, sending inbox replies, enabling rules that can send future replies, moderating comments or messages, notifying mentioned task members, sending or resending approval or client-review notifications, and analytics or report exports that leave the local machine or are shared with clients.

If the user asks for one of these but omits business-critical details, ask a targeted question instead of guessing.

## Generated media

- Create, choose, or upload assets separately before scheduling or publishing.
- Inspect uploaded media IDs and integration requirements before attaching media to content.
- Do not represent generated media as approved client assets unless the user says it is approved.
- For platforms requiring media/title/options, resolve those requirements before scheduling or publishing.

## Bulk actions

- Treat any operation over multiple content, media, inbox, grid, reviewer, or analytics items as broad.
- Confirm the count and explicit ID list or filter before execution.
- Avoid `{}` filters for bulk actions unless the user explicitly confirms the full scope.

## Mutation output

After every mutation, report:

- Created or changed IDs.
- Workspace ID and integration ID when applicable.
- Resulting content/media/inbox/grid/approval status when available.
- Scheduled time with timezone when scheduling.
- External effect, such as public publish, sent reply, reviewer notification, moderation, report file, or deletion.
- Next review or recovery step if the action failed or remains pending.
