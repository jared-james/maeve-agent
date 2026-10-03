# Safety policy

Use this policy before Maeve mutations. Authorization already given for the same action, targets and external effect remains valid. Ask only when it is missing or those details change.

## Default policy

- Create drafts with `create_draft_content`. Schedule or publish only when the user explicitly requests it.
- Prefer read-only discovery before mutation: auth, workspace, integration, capabilities, options, and relevant current state.
- Never schedule without explicit date, time, and timezone.
- Never publish immediately from ambiguous wording such as "post this" unless the user clearly means publish now.
- Use the connected MCP tools and their confirmation contracts. If the required operation is unavailable, direct the user to the Maeve app.

## Non-negotiable agent rules

- Default ambiguous content creation to `create_draft_content`.
- Never schedule without an explicit date, time, and timezone in the current task or user-provided source of truth.
- Never immediate-publish without explicit publish-now intent in the current task.
- Never send inbox replies, moderation actions, approval notifications, client review emails, or bulk read-all without confirmation.
- Never create or enable an inbox auto-reply rule without confirming the rule, account, audience, reply content, and expected external effect.
- Never add a task comment containing member mentions without confirming the mentioned recipients and notification effect.
- Never delete content, failed inbox messages, grid items, media, media folders/labels, or hashtag groups without confirmation.
- Always inspect integration capabilities before setting platform-specific fields.
- Always inspect dynamic options when a capability field includes `optionKey`.
- Always import supported media through the connected tools or have the user upload it in Maeve before attaching it to posts.
- Always return created/changed IDs and status after mutations.
- Treat API keys, bearer tokens, refresh tokens, presigned URLs, and raw provider payloads as sensitive.

## Confirmation required

Ensure the user has authorized the action and target before publishing, retrying a publication, changing published content, deleting records, changing publication state, sending review notifications, creating public calendar snapshots, or making broad changes. Read each selected tool's declared schema, confirmation requirements and side effects.

- `create_inbox_automation_rule` or `update_inbox_automation_rule` with `enabled: true` needs explicit authorization for the rule, account, audience, reply content and expected external effect. Execute directly with the exact confirmation required by the operation.
- `create_task_comment` that mentions workspace members needs authorization for those recipients and the notification effect.
- Review decisions remain human actions in the Maeve app. If another requested action is unavailable through the connected tools, explain that limitation.

Pass the required `confirmationText` directly as a tool argument only after the user has authorized the action and target. A confirmation value returned by the server is not user permission. Preserve the host's confirmation prompts.

The confirmation prompt should include action, IDs, workspace, integration, scheduled time if any, recipient/reviewer impact if any, and whether the effect is public, destructive, or broad. Do not infer confirmation from older conversation context if the target IDs, integration, or effect changed.

## Credential handling

- Use the MCP client's own browser authentication or reconnect flow.
- Browser-approved MCP grants can be revoked in Maeve **Settings -> Developer**.
- Do not ask the user to paste API keys, OAuth codes, bearer tokens, refresh tokens, presigned URLs, provider tokens, scopes, or raw provider payloads into chat.
- Do not print token values from files or tool output.
- Treat media view/download URLs as sensitive, especially if presigned or temporary.
- If a tool result includes sensitive values, summarize the non-sensitive state instead of echoing the raw payload.

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
