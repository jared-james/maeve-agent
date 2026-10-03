# Troubleshooting

Use this when connected MCP operations fail or return validation or provider errors.

## Authentication failures

Use the MCP client's own connect or reconnect action. The client handles authentication; do not request credentials or authorization URLs in chat. Stop dependent operations until the connection is restored. If the client cannot connect, report that limitation rather than using another execution surface.

## Missing workspace

Discover `workspace.list`, load its details, and execute it through `maeve_read`. If the workspace is missing, report that the connected account may lack access. Do not switch credentials to bypass the result.

## Missing or unusable integration

Load details for `integrations.list` and `integrations.get_capabilities`, then execute through `maeve_read` for the selected workspace and integration.

- `connected`: proceed subject to capabilities and permissions.
- `requires_reauth`: ask the user to reconnect the social account in Maeve.
- `disabled`: ask the user to enable or choose another integration.
- `auth_incomplete`: ask the user to finish provider authorization in Maeve.

If an option key is unsupported, refetch capabilities for the exact integration ID.

## JSON validation errors

Common fixes:

- Follow the input schema returned by `maeve_details`; fix only the fields identified in `details.validationErrors`.
- Ensure IDs are UUID strings; grid planner IDs must be UUIDv4.
- Include at least one field for update payloads.
- For create content, include `integrationId` and at least one useful draft field such as `captions.canonical`, `internalTitle`, `publishTitle`, or `contentMedia`.
- Keep `captions.canonical` and caption overrides at or below 25000 characters.
- Use `contentMedia` for content attachments. Keep it to 10 or fewer items before applying stricter platform `maxMedia` from capabilities.

## Timezone errors

`scheduledAt` must be ISO 8601 with an explicit timezone:

```json
{ "scheduledAt": "2026-05-01T10:00:00+10:00" }
```

Valid examples include offsets like `+10:00` and UTC `Z`. Do not use vague dates such as "tomorrow morning" in payloads.

## Platform requirement errors

Load details for `integrations.get_capabilities` and execute through `maeve_read` before retrying.

Then check:

- `requiresMedia`: upload/select media first.
- `requiresTitle`: include `publishTitle`.
- `maxMedia`: reduce attached media.
- `postTypes`: choose a supported `postType`.
- `mediaTypes`: choose supported uploaded media.
- `settings.fields`: move platform-specific values into `settings` using supported keys.
- `optionKey`: fetch dynamic options before choosing values.

Known strict cases:

- Pinterest requires media, `publishTitle`, and `settings.boardId`. Video pins also require a cover: pass `contentMedia[].cover.coverMediaId`, or use a video uploaded through the app when it has a stored thumbnail.
- YouTube requires one video and `publishTitle`.
- TikTok requires media and `settings.privacy_level` from `tiktok-creator-info`. `publishTitle` is optional.

Provider behavior confirmed in live testing (2026-08-18):

- X rejects a post whose text matches a recent post with 403 "duplicate content". Text-only threads with identical messages always trip this; vary the text per message.
- Instagram rejects an unresolvable `audio_id` with subcode 2207065 and the misleading message "audio_id is not a valid parameter for the media type, REELS". It usually means the audio ID does not exist; source IDs from the `instagram-audio` option key.
- Pinterest rejects production pins to Sandbox boards with provider code 15, and the boards listing does not identify Sandbox boards. If a pin fails this way, choose a different board.
- Read `platformPostId`, reply identities and `permalink` from MCP operation `content.get` through `maeve_read` after publishing. Link availability differs by provider and can remain null; do not promise that the next analytics sync will populate it. A queued response, `sent` state or saved provider ID alone does not prove native visibility. TikTok `deliveryMode: tiktok_drafts` still requires publication in the TikTok app. Google Offer owner status can say Published before a public card is visible.
- Instagram requires media.

## Plan or role errors

Approval history, client reviews, approval decisions/comments, pending approval counts, boosts, and analytics PDF reports require a standard workspace plan and specific roles.

Report `plan_required`, `forbidden` or `not_found` and do not retry. If an operation is unavailable, explain the limitation and direct the user to Maeve.

## Rate limits and provider errors

If an operation fails with rate limit or provider errors:

- Respect `retryAfterSeconds`; do not immediately loop retries.
- Preserve the operation ID, workspace, integration, content/media/message ID, provider/platform, and error code/message in the response.
- For provider auth errors, recheck integration status.
- Read the affected resource before retrying an uncertain write. Follow the operation retry contract and preserve successful IDs. Use `content.retry` only when available and authorized; never create replacement content to recover an uncertain publication.

## Output and debugging

Summarize structured tool results and errors without exposing credentials or private URLs. Use filters from the live operation schema to bound list results. Never report a queued response as completed publication.
