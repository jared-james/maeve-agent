# Platform content

Use this reference for content creation, editing and result checks through MCP or the CLI. The CLI examples require maeve-cli 0.12.4 or newer. Inspect live integration capabilities and dynamic options first. Draft acceptance does not prove publishing readiness or native visibility.

## Shared content model

A content edition targets one integration. The shared root title is `organizationalTitle`, edited through MCP operation `content.update_metadata` with `maeve_write` or CLI `content:metadata`. The edition's `internalTitle` is separate; `publishTitle` is provider-facing. Captions use `captions.canonical` and optional `captions.overrides` keyed by platform, including `google-business-profile`. Preserve exact requested whitespace and Unicode. An omitted override map preserves it on update, `{}` clears it, and a platform value of `""` remains an explicitly empty override.

Omit `contentMedia` to preserve attachments, send a complete replacement array to change them, or `[]` to remove them. Media IDs must belong to the selected workspace. The current input contract limits each message to ten attachments even when capabilities advertise a higher provider maximum. Apply the smaller limit.

Creative updates do not change publication time. CLI `content:update` excludes both `intent` and `scheduledAt`; use the scheduling or revert-to-draft operation. Revert-to-draft cancels all scheduled editions of the same root, so inspect its scope before invoking it. Omit `threadMessages` to preserve replies; a supplied array replaces them and `[]` removes them. Replies use canonical captions without nonempty platform overrides. Read the result again to obtain current reply IDs.

For multiple drafts, MCP operation `content.create_drafts` uses the live batch schema through `maeve_write`. CLI runs `content:create` once per item. Retain each result and its stable idempotency key, replay identical requests with the same key, and give a corrected failed input a new key. Inspect per-item results before retrying. Do not schedule or publish a draft batch to validate it. An uncertain publish result needs inspection of existing content and provider state, not a newly created replacement.

## X

For X, inspect `integrations:capabilities` for the selected account first. Root text uses `captions.canonical` with an optional `captions.overrides.x`. Omit `overrides` on an edit to preserve existing overrides; `{}` clears them and `{ "x": "" }` keeps an explicitly empty X caption. Replies use only `captions.canonical`; nonempty reply overrides are rejected. Omit `threadMessages` to preserve replies or send `[]` to remove them. Each supplied reply array replaces the saved reply sequence. Use `content:get` afterward to read the current reply IDs.

X supports up to 20 replies and four attachments per message. Put media in `contentMedia` with unique `order` values and supported per-platform `crops`. Use `media:get` and `media:update` for media-record `altText`, including `""` to clear it. X publishes this metadata for images and GIFs, not videos. Custom covers, tags and first comments are not X composer controls. Standard text uses a 280 weighted-character limit; longer text requires confirmed subscription eligibility. Standard videos are limited to 140 seconds and 512MB; the existing blue-verified path allows up to 10 minutes. A successful draft save proves neither entitlement nor publishing readiness. Maeve blocks URLs in resolved root and reply captions at publishing handoff.

Read `content:get` after publishing to capture the X root and reply `platformPostId` and `permalink` values. Historical publications may have missing reply identities or links. A queued response is not publication proof; verify the native posts before cleanup. Pace draft creation and replay, which both count toward the API request limit, and stop on a rate-limit response before resuming after its cooldown.

The web content header uses the root `organizationalTitle`. Change it with `content:metadata --id <id> --json <file>` and `{"organizationalTitle":null}` to clear it. `content:update` changes the edition's `internalTitle`; it does not rename the shared root or sibling editions. Other metadata fields are optional, and empty assignment arrays clear those assignments.

`settings.xCommunityUrl` targets the root only. Its existing parser extracts digits after `communities/`; a nonempty value without that pattern is ignored and publishing uses the normal account timeline. Set `""` to clear it. Verify the community and the account's access before publishing. Creative edits do not reschedule content; use `content:schedule` or `content:revert-to-draft`. Deleting published X content removes only the selected root or reply from X and retains its sent audit record; deleting the root does not delete its replies. For multiple drafts, run `content:create` once per item with a stable, distinct `--idempotency-key`, retain each successful ID, and read every result. Do not create fresh content to recover an uncertain publication.

Long-form X Articles have their own commands and operations, not `content:create`: `articles:create` and `articles:update` on the CLI, `articles.create` and `articles.update` on MCP, and `/articles` on the public API. They need an X account whose `content.canPublishArticles` is true. An article has a title of up to 200 characters, an HTML body with up to 100,000 characters of text and up to 20 inline still images named by Media Room ID with `<img data-media-id="...">`, and an optional still-image cover. GIFs and video are not supported. A body replacement needs the `bodyVersion` from the latest read. Schedule, publish, unschedule, retry and delete an article with the content commands and its `contentId`. `articles:send-to-x-drafts` sends it to the account's drafts on X instead of publishing and requires `--yes`, because X's API cannot delete that draft.

## Instagram

Use `post`, `reel` or `story` with uploaded media. Keep crops, user/product tags and cover metadata in `contentMedia`, not settings. Resolve collaborators through `instagram-collaborators` with the exact username; resolve products through `instagram-catalogs` and then `instagram-products`. Only use option keys advertised for the selected integration.

Trial reels require one video and `postType: "reel"`. Set `settings.trialReel` and `trialReelGraduationStrategy` (`manual` or `automatic`); share-to-feed is omitted while trial mode is enabled. Use `trialReel: false` to leave trial mode. Licensed Reel audio requires Facebook Login and a returned `audio_id` from `instagram-audio`. Inspect source audio with MCP operation `media.inspect_audio` through `maeve_write` or CLI `media:inspect-audio` before choosing `audio_volume` and `video_volume`; a null `hasAudio` is inconclusive.

## Facebook

Use top-level `postType` for feed, Reel or Story. Feed accepts up to ten images or one video, without mixing them. Feed link CTA buttons require `facebookLinkUrl` and a supported `facebookCtaType`; they do not apply to media posts, Reels or Stories. Story captions and first comments are not published. Follow live rules for vertical media, dimensions and duration. Discover locations only when the integration advertises that capability. The existing published-caption edit applies to Facebook; do not assume other platforms support it.

## Threads

Use `post` or `thread`, with at most 20 replies. Published root and reply text is limited to 500 characters each. The current input limit is ten ordered images/videos per message, although the provider capability maximum is 20. Root-only settings are `topicTag`, `locationId`, `linkAttachment` and `replyControl`. Topic tags allow 50 characters; a leading hash is removed, while periods, ampersands, tabs and newlines are rejected. Resolve locations through `locations`. A link attachment is used only without media. `settings: {}` clears settings.

Deleting a published root does not delete its replies. Read each reply's `platformPostId` and `permalink` and delete selected replies separately. Older replies without saved provider IDs need native cleanup. Ghost content, first comments and published-caption editing are not supported.

## TikTok

Discover account-specific privacy choices through `tiktok-creator-info`. Use `tiktok_post_to_drafts` only when delivery to TikTok's app is intended. Read `deliveryMode`: a `sent` item with `tiktok_drafts` still requires review and publication in TikTok and is not evidence of a public publication.

For Business direct video music, use `tiktok-cml-tracks` with `countryCode` and optional `genre` and `dateRange`. Select a returned `songClipId` for `tiktok_music`, retaining its account binding and both volume values from the live schema. The chart country is not a licensing guarantee. Selected clips do not support photo content or delivery to TikTok drafts. Set `is_aigc` for AI-generated content. Respect the current ten-attachment input limit even though TikTok capabilities advertise 35 images.

## YouTube

Use one video and `publishTitle`. Top-level `postType: "reel"` selects Shorts and `"post"` selects a normal video. Resolve `categoryId` through `youtube-video-categories`. Set privacy and audience choices deliberately; use media cover metadata for a custom thumbnail where supported. Queued or sent state does not establish finished provider processing or public visibility.

## Pinterest

Use one image or video, `publishTitle`, and a `boardId` from `pinterest-boards`. Video needs an uploaded image cover through `contentMedia[0].cover.coverMediaId` unless the video already has a stored thumbnail. `settings.link` and `settings.altText` are optional.

Create a board only when explicitly requested, with MCP operation `integrations.create_pinterest_board` through `maeve_write` or CLI `integrations:pinterest:create-board`, integration-management permission, and the live confirmation contract. Privacy is `PUBLIC` or `SECRET`; omission defaults to public. Creation is not idempotent, so inspect boards after an uncertain response before retrying. Returned boards do not identify Sandbox status. Provider code 15 requires choosing a production board. Board deletion is not exposed.

## Google Business Profile

For Google Business Profile, choose the integration for the exact business location with `integrations:list` and inspect `integrations:capabilities`. An account can contain multiple locations. Use `postType: "post"` and `settings.googlePostType: "standard"`, `"event"` or `"offer"`. Text uses `captions.canonical` and optionally `captions.overrides["google-business-profile"]`. Omit overrides on update to preserve them, use `{}` to clear them, or use an explicitly empty platform string to retain an empty override. The published summary is trimmed and limited to 1,500 characters. Existing caption URL restrictions still apply.

Google updates replace the supplied `settings` object. Omit `settings` to preserve it. To remove a button, send the remaining settings without `googleCallToActionType` and `googleCallToActionUrl`; do not send an empty CTA enum. For example, `{"settings":{"googlePostType":"standard"}}` removes the previous CTA and event/offer fields. Use `{"settings":{}}` to reset all Google settings to their defaults; `content:get` returns `settings: null` when no public settings remain. Retain any language override or other wanted fields in a replacement.

The six CTA values are `book`, `order`, `shop`, `learn_more`, `sign_up` and `call`. All except `call` require an HTTPS `googleCallToActionUrl`. CALL uses the business location phone number; omit its URL. Offers cannot have a CTA. Event and Offer require `googleEventTitle`, `googleEventStartAt` and `googleEventEndAt`. The event title is separate from the root `organizationalTitle` and edition `internalTitle`. Use two dates such as `2026-10-01` and `2026-10-02`, or two local wall-clock times such as `2026-10-01T09:15` and `2026-10-01T10:45`. The end must be later. Do not include `Z` or a timezone offset. Use minute precision. These ranges do not schedule publication: use the existing scheduling commands and an explicit publication instant with timezone. Creative updates preserve the existing publication time.

Offer-only fields are `googleOfferCouponCode`, `googleOfferRedeemOnlineUrl` (HTTPS) and `googleOfferTerms`. Remove them when leaving Offer; remove event fields when returning to Standard. `googleLanguageCode` overrides the connected location's language; omission or an empty string falls back to that saved language when available.

Google supports text or one image, not threads, video or multiple images. Attach an owned media record through `contentMedia`, replace it with a new single relationship, or send `[]` to remove it. The current Google provider sends the original HTTPS image URL and does not apply relationship crops or media-record `altText`. Those saved metadata values do not prove a crop or alternative text was published. The image MIME check is not proof of GIF acceptance or compliance with Google's media requirements.

Read `content:get` for the stored Google `platformPostId` (`accounts/.../locations/.../localPosts/...`) and safe `permalink`. A queue response, sent status or resource name alone is not proof of native visibility. Existing selected deletion uses this identity and retains the publication audit record. Google has no product published-caption edit command. For draft matrices use one `content:create` per item with distinct stable idempotency keys, retain successful IDs, replay identical inputs with the same key, and use a new key for a corrected failed item. Never publish or schedule a draft matrix as part of draft validation.

## LinkedIn and LinkedIn Page

Select the exact personal or Page integration. Personal authors use member URNs; Page authors use organization URNs. Page permissions are not required for personal publication. First comments require the separate member-feed or organization-feed capability reported by `canPublishFirstComment`.

The web supports text, URL previews, up to ten ordered photos or one video. Provider discovery also lists 20-media, article, document and poll capabilities; that does not make them reachable web or CLI paths. Existing API formats remain available. CLI create excludes article and its uploader accepts images/videos only. Polls use the top-level payload described in [content-payloads.md](content-payloads.md) and cannot contain media.

Use `captions.canonical` and an optional override keyed by `linkedin` or `linkedin-page`. An empty override stays empty; an empty overrides object clears overrides. Internal titles are planning fields. Use the matching platform crop key on each attachment; legacy personal-key Page crops remain supported. Alt text belongs to the media record. Scheduling uses Maeve's workspace timezone, not LinkedIn-native scheduling.

Read content after publishing for a valid numeric share or ugcPost URN. Missing or fallback identities remain null. A URL or sent status alone does not establish native visibility. Verify the selected author and rendering when required, then use the existing deletion action; the sent audit remains. LinkedIn has no published-caption edit action.
