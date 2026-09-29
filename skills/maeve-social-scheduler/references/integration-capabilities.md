# Integration capabilities

Quick-glance reference for the live capability catalog returned by `integrations:capabilities`.

## Contents

- Commands
- Response shape
- Settings and media metadata
- Platform matrix
- Platform settings and options
- Agent rules

## Commands

List integrations:

```bash
maeve integrations:list --workspace <workspaceId>
```

Get safe publishing requirements:

```bash
maeve integrations:capabilities --workspace <workspaceId> --integration <integrationId>
```

Fetch dynamic options returned by capabilities. Prefer MCP operation `integrations.get_options` through `maeve_read` when connected; use the CLI fallback when MCP is unavailable:

```bash
maeve integrations:options --workspace <workspaceId> --integration <integrationId> --key <optionKey>
```

Use `--json <file>` for option bodies:

```json
{ "regionCode": "AU" }
```

Supported option body fields:

- `regionCode`: optional string, max 2 characters.
- `query`: optional string, max 255 characters.
- `catalogId`: optional string, max 255 characters.
- `audioType`: required for `instagram-audio`; use `music` or `original_sound`.
- `countryCode`: required chart country for `tiktok-cml-tracks`.
- `genre`: optional TikTok chart genre, default `ALL`.
- `dateRange`: optional `1DAY`, `7DAY`, `30DAY` or `90DAY`, default `7DAY`.

## Response shape

Capabilities returns:

- `integration`: `id`, `platform`, `username`, `name`, and `status`.
- `content`: `postTypes`, `mediaTypes`, `maxMedia`, `supportsThreads`, `requiresMedia`, and `requiresTitle`.
- `rules`: live plain-text platform publishing rules from the backend. Treat this as the source of truth for platform-specific requirements.
- `settings.fields`: supported platform settings.
- `options`: dynamic option keys available for that integration.
- `capabilities`: curated provider capability data from the connection, when present.

Integration status is one of:

- `connected`
- `requires_reauth`
- `disabled`
- `auth_incomplete`

Do not mutate content through an integration unless its status is `connected`.

## Settings and media metadata

`settings.fields` contains provider publish settings only. Do not place media metadata or editor state in `settings`.

Use `contentMedia` for per-media metadata:

- `contentMedia[].crops`: optional crop data keyed by platform.
- `contentMedia[].userTags`: optional user tags where supported.
- `contentMedia[].productTags`: optional product tags where supported.
- `contentMedia[].cover.coverMediaId`: optional uploaded media ID for a cover/thumbnail where supported.
- `contentMedia[].cover.thumbOffsetMs`: optional thumbnail offset; provider execution maps this to platform API fields.

## Platform matrix

| Platform                  | Post types                | Media types                  | Max media | Threads | Requires media | Requires title |
| ------------------------- | ------------------------- | ---------------------------- | --------: | ------- | -------------- | -------------- |
| `x`                       | `post`, `thread`          | `image`, `gif`, `video`      |         4 | yes     | no             | no             |
| `linkedin`                | `post`, `article`, `poll` | `image`, `video`, `document` |        20 | no      | no             | no             |
| `linkedin-page`           | `post`, `article`, `poll` | `image`, `video`, `document` |        20 | no      | no             | no             |
| `instagram`               | `post`, `reel`, `story`   | `image`, `video`             |        10 | no      | yes            | no             |
| `facebook`                | `post`, `reel`, `story`   | `image`, `video`             |        10 | no      | no             | no             |
| `facebook-page`           | `post`, `reel`, `story`   | `image`, `video`             |        10 | no      | no             | no             |
| `threads`                 | `post`, `thread`          | `image`, `video`             |        20 | yes     | no             | no             |
| `tiktok`                  | `post`                    | `image`, `video`             |        35 | no      | yes            | no             |
| `youtube`                 | `post`, `reel`            | `video`                      |         1 | no      | yes            | yes            |
| `pinterest`               | `post`                    | `image`, `video`             |         1 | no      | yes            | yes            |
| `google-business-profile` | `post`                    | `image`                      |         1 | no      | no             | no             |

X also supports `article`, which only the article operations create (`articles.create`, `articles:create`, `POST /articles`), never content create.

The table describes provider capabilities. Current content inputs cap attachments at ten per message, so apply the smaller input/platform limit. Read [Platform content](platform-content.md) for current model constraints and reverse states.

Unknown platforms fall back to common post types `post`, `reel`, `story`, and `thread`; media types `image`, `video`; max media 10; no required media/title; no thread support.

## Platform settings and options

### X

Settings:

- `xCommunityUrl`: optional string. Community ID is extracted from the URL.

Options: none.

### LinkedIn

Settings: none.

Options: none.

Post types: `post` (text, images, or one video), and `poll` with the top-level `poll` object (see the poll payload in content-payloads). Capabilities also advertise `article` and document media, but neither is creatable through this surface today: media upload accepts only images and videos, and the API does not accept `postType: "article"`. Treat document posts and article shares as app-only.

### Instagram

Settings:

- `collaborators`: optional string array of usernames or IDs, resolved through `instagram-collaborators`.
- `trialReel`: boolean for single-video Reels.
- `trialReelGraduationStrategy`: `manual` or `automatic`; share-to-feed is omitted while trial mode is on.
- `audio_id`: optional Reel audio asset ID from the `instagram-audio` option key. Facebook Login accounts only; single-video Reels only.
- `audio_volume` / `video_volume`: optional 0-100 volumes for the attached audio and the original video audio.

Options:

- `instagram-collaborators`: requires an exact username in `query`.
- `instagram-catalogs`: no input required.
- `instagram-products`: requires `catalogId`; optional `query`.
- `instagram-audio`: requires `audioType` (`music` or `original_sound`); optional `query`, omit for recommended audio. Returns `audio_id`, title, artist, duration, and a preview link per track.

Fetch flow for products:

```bash
# MCP preferred: execute integrations.get_options through maeve_read with optionKey instagram-catalogs, then instagram-products.
maeve integrations:options --workspace <workspaceId> --integration <integrationId> --key instagram-catalogs
maeve integrations:options --workspace <workspaceId> --integration <integrationId> --key instagram-products --json product-options.json
```

Fetch flow for Reel audio:

```bash
# MCP preferred: execute integrations.get_options through maeve_read with optionKey instagram-audio.
maeve integrations:options --workspace <workspaceId> --integration <integrationId> --key instagram-audio --json audio-options.json
# audio-options.json: { "audioType": "music", "query": "upbeat" }
```

### Facebook and Facebook Page

Settings:

- `facebookLinkUrl`: optional feed link URL. Required when `facebookCtaType` is a CTA button.
- `facebookCtaType`: optional CTA button type. Use `NO_BUTTON` or omit for no CTA.
- `facebookLocationId`: optional Facebook Page location ID.

Options: none.

### Threads

Settings:

- `locationId`: optional string, use option key `locations`.
- `topicTag`: optional string.
- `linkAttachment`: optional string URL, used only when the root has no media.
- `replyControl`: `everyone`, `accounts_you_follow`, `mentioned_only`, `parent_post_author_only` or `followers_only`. These settings apply only to the root.

Options:

- `locations`: requires `query`.

### TikTok

Settings:

- `privacy_level`: string, use option key `tiktok-creator-info`.
- `disable_comment`: boolean.
- `disable_duet`: boolean.
- `disable_stitch`: boolean.
- `brand_content_toggle`: boolean.
- `brand_organic_toggle`: boolean.
- `is_aigc`: boolean.
- `photo_cover_index`: number.
- `auto_add_music`: boolean.
- `tiktok_post_to_drafts`: boolean.
- `tiktok_music`: selected Commercial Music Library clip for Business direct video and photo publishing, with the same `songClipId` for both; photo posts ignore the volume values. Use the live schema and `tiktok-cml-tracks` options.

When `privacy_level` is omitted, Maeve publishes `PUBLIC_TO_EVERYONE`. `SELF_ONLY` is forced only while the TikTok app is unaudited.

Options:

- `tiktok-creator-info`: no input required.
- `tiktok-cml-tracks`: required `countryCode`; optional `genre` and `dateRange`. The chart country is not a licensing guarantee. See [Platform content](platform-content.md) for selection and delivery-mode limits.

### YouTube

Settings:

- `privacyStatus`: `public`, `unlisted`, or `private`.
- `tags`: string array.
- `categoryId`: string, use option key `youtube-video-categories`.
- `madeForKids`: boolean.
- `notifySubscribers`: boolean.
- `embeddable`: boolean.
- `license`: `youtube` or `creativeCommon`.

When `privacyStatus` is omitted, Maeve publishes `public`.

Options:

- `youtube-video-categories`: optional `regionCode`, for example `US` or `AU`.

### Pinterest

Settings:

- `boardId`: required string, use option key `pinterest-boards`.
- `link`: optional outbound link string.
- `altText`: optional media alt text string.
- Video pins require an image cover. Set `contentMedia[0].cover.coverMediaId` to an uploaded image media ID unless the uploaded video already has a stored thumbnail.

Options:

- `pinterest-boards`: no input required.
- Pinterest does not identify Sandbox boards in the board-list response. Production pins cannot use Sandbox boards. If Pinterest returns provider code `15`, select a production board and create a newly confirmed publish attempt.

### Google Business Profile

Settings:

- `googlePostType`: `standard`, `event`, or `offer`. Defaults to `standard`.
- `googleCallToActionType`: optional `book`, `order`, `shop`, `learn_more`, `sign_up`, or `call`. Do not use a CTA on offer posts.
- `googleCallToActionUrl`: required for every CTA except `call`, which uses the location phone number.
- `googleEventTitle`: required for event and offer posts.
- `googleEventStartAt`: required for event and offer posts. Use an ISO 8601 local timestamp such as `2026-08-01T09:00`.
- `googleEventEndAt`: required for event and offer posts. It must be after `googleEventStartAt`.
- `googleOfferCouponCode`: optional offer-only coupon code.
- `googleOfferRedeemOnlineUrl`: optional offer-only redemption URL.
- `googleOfferTerms`: optional offer-only terms and conditions.
- `googleLanguageCode`: optional BCP-47 language code override.

Options: none.

Posts may contain text only or one HTTPS image. Follow the live `rules` returned by capabilities for image size, dimensions, caption length, and event or offer requirements. Google Business Profile Local Posts do not support recurrence or a separate provider-side scheduled time through these settings.

## Agent rules

- Call `integrations:capabilities` before platform-specific settings.
- Read and follow the live `rules` string returned by capabilities; do not rely on static prose for platform-specific publishing requirements.
- When a settings field has `optionKey`, execute MCP `integrations.get_options` through `maeve_read` before choosing a value, or use CLI `integrations:options` as the fallback.
- If `requiresMedia` is true, attach uploaded media before scheduling or publishing.
- If `requiresTitle` is true, include `publishTitle` before scheduling or publishing.
- Respect both `maxMedia` and the current input limit when building `contentMedia`.
- Use the top-level `postType` only. Do not send platform-specific post type override settings.
- Do not expose tokens, scopes, stored provider settings, or raw provider payloads; capabilities and options are the safe discovery surface.
