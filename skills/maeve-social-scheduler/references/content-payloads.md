# Content payloads

Use the exact input schema returned by `maeve_details` for each MCP operation. The examples below show content fields, not complete operation inputs; include workspace context and any other required fields from the live schema. Do not send fields that the selected operation does not accept.

## General rules

- Use IDs returned by the connected tools for the selected workspace and integration.
- Create drafts with `content.create_draft`; use `content.schedule` or `content.publish_now` separately when authorized.
- Resolve the date, time and timezone before scheduling. Use an explicit offset or UTC `Z` in the publication timestamp.
- Import supported attachments or public HTTPS links with `media.import`, or ask the user to upload files in Maeve. Attach returned Media Room IDs with `contentMedia[].mediaId`.
- Read [Platform content](platform-content.md) for platform-specific models and clearing behavior.

## Content create

Minimal draft:

```json
{
  "integrationId": "00000000-0000-4000-8000-000000000001",
  "captions": {
    "canonical": "Launch post copy"
  }
}
```

Platform-aware payload with taxonomy:

```json
{
  "integrationId": "00000000-0000-4000-8000-000000000001",
  "internalTitle": "Launch planning card",
  "publishTitle": "Launch title",
  "captions": {
    "canonical": "Launch post copy"
  },
  "contentMedia": [
    {
      "mediaId": "00000000-0000-4000-8000-000000000002",
      "order": 0,
      "cover": {
        "thumbOffsetMs": 2500
      }
    }
  ],
  "postType": "reel",
  "pillarIds": ["00000000-0000-4000-8000-000000000003"],
  "formatIds": ["00000000-0000-4000-8000-000000000004"],
  "labelIds": ["00000000-0000-4000-8000-000000000005"],
  "campaignId": "00000000-0000-4000-8000-000000000006",
  "campaignPhaseId": "00000000-0000-4000-8000-000000000007",
  "priority": "medium"
}
```

Supported content fields:

- `integrationId`: required UUID.
- `internalTitle`: optional internal planning title, max 500 characters. Never published.
- `publishTitle`: optional provider-facing title, max 500 characters. Required before scheduling/publishing when capabilities say `requiresTitle`.
- `notes`: optional rich-text HTML, max 100000 characters. Internal - never published. Use to attach planning context, links, or meeting notes to a content item.
- `captions`: optional object with `canonical` publish text and optional platform overrides.
- `contentMedia`: optional uploaded media relationships, max 10 before stricter platform limits.
- `settings`: optional object for platform fields from capabilities.
- `postType`: optional `post`, `reel`, `story`, `thread`, or `poll` (LinkedIn only; requires the top-level `poll` object).
- `poll`: LinkedIn poll content, only with `postType: "poll"`. Question up to 140 characters, 2 to 4 options up to 30 characters each, `duration` of `ONE_DAY`, `THREE_DAYS`, `SEVEN_DAYS`, or `FOURTEEN_DAYS`, `voteSelectionType: "SINGLE_VOTE"`, `isVoterVisibleToAuthor: true`. Polls cannot carry media, documents, or article content.
- `firstComment`, `shareToFeed`: optional publish behavior fields.
- `contentMedia[].crops`, `contentMedia[].userTags`, `contentMedia[].productTags`, `contentMedia[].cover.coverMediaId`, `contentMedia[].cover.thumbOffsetMs`: optional media metadata where platform capabilities allow it.
- `threadMessages`: optional array for thread-style content, max 20 items.
- `pillarIds`, `formatIds`, `labelIds`: optional UUID arrays.
- `campaignId`, `campaignPhaseId`: optional campaign UUIDs; use `null` to clear campaign links on update.
- `assigneeIds`: optional UUID array.
- `priority`: optional `urgent`, `high`, `medium`, or `low`.

## LinkedIn poll content

```json
{
  "integrationId": "00000000-0000-4000-8000-000000000001",
  "captions": { "canonical": "Which format should we publish next?" },
  "postType": "poll",
  "poll": {
    "question": "Which format should we publish next?",
    "options": [{ "text": "Guide" }, { "text": "Checklist" }],
    "duration": "THREE_DAYS",
    "voteSelectionType": "SINGLE_VOTE",
    "isVoterVisibleToAuthor": true
  }
}
```

## Thread content

Use `postType: "thread"` only when the integration capabilities support threads.

```json
{
  "integrationId": "00000000-0000-4000-8000-000000000001",
  "captions": {
    "canonical": "Thread starter"
  },
  "postType": "thread",
  "threadMessages": [
    {
      "captions": {
        "canonical": "Second post in the thread"
      }
    },
    {
      "captions": {
        "canonical": "Third post with media"
      },
      "contentMedia": [
        {
          "mediaId": "00000000-0000-4000-8000-000000000002"
        }
      ]
    }
  ]
}
```

`threadMessages` accepts up to 20 items; each item uses `captions.canonical` and can include `contentMedia`.

## Schedule or publish existing content

Load details for `content.schedule` or `content.publish_now` and execute through `maeve_write` with the existing content ID and workspace context. Follow the live confirmation contract. Scheduling requires an unambiguous publication instant; a queued response is not publication proof.

## Other workflows

For media changes, review requests, calendar, strategy and other available workflows, discover the operation and use its live schema. Review decisions remain human actions. If a workflow is unavailable through the connected MCP catalog, direct the user to the Maeve app.
