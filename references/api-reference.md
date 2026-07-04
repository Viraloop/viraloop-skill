# Viraloop API reference

> Generated from src/backend/api/v1/spec/registry.mjs by npm run generate:agents. Do not edit by hand.

Base URL: `https://viraloop.io/api/v1`

Authentication: `Authorization: Bearer vl_live_...` (or `X-API-Key`). Create keys at https://viraloop.io/settings/developers.

Envelope: success responses are `{ "success": true, "data": ..., "pagination"?: { total, page, limit, pages } }`. Errors are `{ "success": false, "error": { "type", "message" } }` with types: `invalid_input`, `unauthorized`, `forbidden_scope`, `not_found`, `conflict`, `insufficient_credits`, `usage_limit_reached`, `rate_limited`, `internal_error`.

## me

### Introspect the API key

`GET /me`

Returns the team behind the credentials: name, credit balance, plan, default workspace and the scopes granted to the key. Call this first to verify auth works.

- Scopes: any valid key
- Credits: none
- CLI: `viraloop whoami`
- MCP tool: `viraloop_get_me`

Example:

```bash
curl -s "https://viraloop.io/api/v1/me" \
  -H "Authorization: Bearer $VIRALOOP_API_KEY"
```

Response:

```json
{
  "success": true,
  "data": {
    "teamId": "665f1b2a9c31a2b3c4d5e6f7",
    "teamName": "Acme",
    "credits": 420,
    "plan": "Growth (monthly)",
    "planValidUntil": "2026-08-01T00:00:00.000Z",
    "workspaceId": "665f1b2a9c31a2b3c4d5e6f8",
    "auth": {
      "via": "api_key",
      "keyPrefix": "vl_live_a1b2c3d4",
      "scopes": [
        "posts:read",
        "posts:write"
      ],
      "legacy": false
    }
  }
}
```

## credits

### Credit balance and ledger

`GET /credits`

Returns the team's current credit balance plus a paginated ledger of credit movements (top-ups, spending, rewards, refunds). Turbo content generation does not consume credits; influencer media generation does (createInfluencerVideo: 20 credits, createInfluencer preview: 10). Failed generations are refunded automatically; running out of credits returns HTTP 402 with error type insufficient_credits.

- Scopes: `credits:read`
- Credits: none
- CLI: `viraloop credits`
- MCP tool: `viraloop_get_credits`

Query parameters:

| Name | Type | Required | Description |
| --- | --- | --- | --- |
| `page` | integer | no | Page number, starting at 1 |
| `limit` | integer | no | Items per page (max 100) |
| `type` | `recurring` \| `topup` \| `spending` \| `trial` \| `spin` \| `reward` \| `refund` | no | Filter by entry type |

Example:

```bash
curl -s "https://viraloop.io/api/v1/credits" \
  -H "Authorization: Bearer $VIRALOOP_API_KEY"
```

Response:

```json
{
  "success": true,
  "data": {
    "balance": 420,
    "entries": [
      {
        "id": "665f1b2a9c31a2b3c4d5e6f9",
        "credits": -10,
        "type": "spending",
        "spendingType": "video_generation",
        "model": "veo3_fast",
        "createdAt": "2026-07-01T12:00:00.000Z"
      }
    ]
  },
  "pagination": {
    "total": 1,
    "page": 1,
    "limit": 20,
    "pages": 1
  }
}
```

## workspaces

### List workspaces

`GET /workspaces`

Lists the team's workspaces (brands). Most endpoints accept an optional workspaceId and default to the team's default workspace.

- Scopes: `workspaces:read`
- Credits: none
- CLI: `viraloop workspaces list`
- MCP tool: `viraloop_list_workspaces`

Example:

```bash
curl -s "https://viraloop.io/api/v1/workspaces" \
  -H "Authorization: Bearer $VIRALOOP_API_KEY"
```

Response:

```json
{
  "success": true,
  "data": [
    {
      "id": "665f1b2a9c31a2b3c4d5e6f8",
      "name": "Acme",
      "isDefault": true,
      "website": {
        "url": "https://acme.com",
        "domain": "acme.com"
      },
      "contentAngles": [
        {
          "title": "Client Revision Hell",
          "description": "..."
        }
      ],
      "preferences": {
        "timezone": "America/New_York",
        "contentLanguage": "English"
      }
    }
  ]
}
```

### Get a workspace

`GET /workspaces/{id}`

Returns one workspace with its brand identity, positioning, tone of voice, content angles and preferences.

- Scopes: `workspaces:read`
- Credits: none
- CLI: `viraloop workspaces get <id>`

Example:

```bash
curl -s "https://viraloop.io/api/v1/workspaces/<id>" \
  -H "Authorization: Bearer $VIRALOOP_API_KEY"
```

Response:

```json
{
  "success": true,
  "data": {
    "id": "665f1b2a9c31a2b3c4d5e6f8",
    "name": "Acme",
    "isDefault": true,
    "identity": {
      "coreIdentity": "...",
      "productOffering": "..."
    },
    "positioning": {
      "mission": "..."
    },
    "toneVoice": {
      "dos": [
        "..."
      ],
      "donts": [
        "..."
      ]
    },
    "contentAngles": [
      {
        "title": "...",
        "description": "..."
      }
    ],
    "preferences": {
      "timezone": "UTC",
      "contentLanguage": "English"
    }
  }
}
```

## accounts

### List connected social accounts

`GET /accounts`

Lists the TikTok, Instagram and YouTube accounts connected to the workspace. Use the returned ids in selectedAccounts when creating posts. Connecting accounts (OAuth) happens in the web app, not through this API.

- Scopes: `accounts:read`
- Credits: none
- CLI: `viraloop accounts list`
- MCP tool: `viraloop_list_accounts`

Query parameters:

| Name | Type | Required | Description |
| --- | --- | --- | --- |
| `platform` | `tiktok` \| `instagram` \| `youtube` | no |  |
| `ownerType` | `brand` \| `influencer` | no |  |
| `workspaceId` | string | no | Workspace to operate in. Defaults to the team's default workspace. |

Example:

```bash
curl -s "https://viraloop.io/api/v1/accounts" \
  -H "Authorization: Bearer $VIRALOOP_API_KEY"
```

Response:

```json
{
  "success": true,
  "data": [
    {
      "id": "665f1b2a9c31a2b3c4d5e700",
      "platform": "tiktok",
      "ownerType": "brand",
      "username": "acme.hq",
      "displayName": "Acme",
      "label": "Main",
      "isDefault": true
    }
  ]
}
```

## generations

### Generate AI post suggestions

`POST /generations`

Generates a batch of ready-to-post content suggestions (video decks with captions, hashtags and rationale) using the workspace's brand context and Turbo configuration. Synchronous: the request returns when generation finishes, typically 5 to 60 seconds depending on count. Review the results, then publish the ones you like via POST /posts with suggestionId.

- Scopes: `generations:write`
- Credits: none
- Rate limit: 10 per 300s
- Supports `Idempotency-Key` header
- CLI: `viraloop generate`
- MCP tool: `viraloop_generate_content`

Body fields:

| Name | Type | Required | Description |
| --- | --- | --- | --- |
| `count` | integer | no | How many suggestions to generate (1-10) |
| `format` | `walloftext` \| `slideshow` \| `greenscreen` | no | Force one content format. Omit to use the workspace's configured mix. |
| `influencerId` | string | no | Feature this AI influencer in every suggestion |
| `workspaceId` | string | no | Workspace to operate in. Defaults to the team's default workspace. |

Example:

```bash
curl -s -X POST "https://viraloop.io/api/v1/generations" \
  -H "Authorization: Bearer $VIRALOOP_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"count":3,"format":"walloftext"}'
```

Response:

```json
{
  "success": true,
  "data": [
    {
      "id": "665f1b2a9c31a2b3c4d5e701",
      "format": "walloftext",
      "caption": "POV: the client asks for one small change",
      "postCaption": "Every designer knows this feeling",
      "hashtags": [
        "design",
        "clientwork"
      ],
      "status": "pending",
      "statusUrl": "/api/v1/generations/665f1b2a9c31a2b3c4d5e701"
    }
  ]
}
```

### List suggestions

`GET /generations`

Lists generated content suggestions in the workspace queue (campaign-owned suggestions are excluded). Filter by status: pending (awaiting review), accepted, scheduled.

- Scopes: `generations:read`
- Credits: none
- CLI: `viraloop generations list`
- MCP tool: `viraloop_list_generations`

Query parameters:

| Name | Type | Required | Description |
| --- | --- | --- | --- |
| `status` | `pending` \| `accepted` \| `scheduled` \| `deleted` | no |  |
| `page` | integer | no | Page number, starting at 1 |
| `limit` | integer | no | Items per page (max 100) |
| `workspaceId` | string | no | Workspace to operate in. Defaults to the team's default workspace. |

Example:

```bash
curl -s "https://viraloop.io/api/v1/generations" \
  -H "Authorization: Bearer $VIRALOOP_API_KEY"
```

Response:

```json
{
  "success": true,
  "data": [
    {
      "id": "665f1b2a9c31a2b3c4d5e701",
      "format": "walloftext",
      "caption": "POV: the client asks for one small change",
      "status": "pending",
      "createdAt": "2026-07-02T10:00:00.000Z"
    }
  ],
  "pagination": {
    "total": 12,
    "page": 1,
    "limit": 20,
    "pages": 1
  }
}
```

### Get a suggestion

`GET /generations/{id}`

Returns one suggestion including its full deck (the editable render blueprint), influencer, angle, rationale and remix source.

- Scopes: `generations:read`
- Credits: none
- CLI: `viraloop generations get <id>`
- MCP tool: `viraloop_get_generation`

Example:

```bash
curl -s "https://viraloop.io/api/v1/generations/<id>" \
  -H "Authorization: Bearer $VIRALOOP_API_KEY"
```

Response:

```json
{
  "success": true,
  "data": {
    "id": "665f1b2a9c31a2b3c4d5e701",
    "format": "walloftext",
    "caption": "POV: the client asks for one small change",
    "postCaption": "Every designer knows this feeling",
    "hashtags": [
      "design",
      "clientwork"
    ],
    "why": [
      "Relatable pain point",
      "Strong hook"
    ],
    "angle": {
      "title": "Client Revision Hell"
    },
    "status": "pending",
    "deck": {
      "ratio": "9:16",
      "textBoxes": []
    }
  }
}
```

## posts

### Create and schedule a post

`POST /posts`

Schedules a post to one or more connected accounts. Three content sources, checked in order: suggestionId (publish a generated suggestion), deck + format (publish a raw deck), or videoUrl / images (publish your own media). Deck-based posts are rendered server-side after this call returns; the response is 202 with a statusUrl to poll. schedule asap posts as soon as the render is ready; schedule scheduled requires a future scheduledTime. Supports the Idempotency-Key header.

- Scopes: `posts:write`
- Credits: none
- Rate limit: 60 per 300s
- Supports `Idempotency-Key` header
- Terminal states: `posted`, `partial`, `completed`, `failed`
- CLI: `viraloop posts create`
- MCP tool: `viraloop_create_post`

Body fields:

| Name | Type | Required | Description |
| --- | --- | --- | --- |
| `suggestionId` | string | no | Publish this generated suggestion (from /generations) |
| `deck` | object | no | Raw deck object (advanced; usually use suggestionId) |
| `format` | `walloftext` \| `slideshow` \| `greenscreen` | no | Deck format, required when deck is provided |
| `videoUrl` | string | no | Publicly reachable video URL to post as-is |
| `thumbnailUrl` | string | no |  |
| `images` | array | no | Image URLs to post as an image/slideshow post |
| `selectedAccounts` | object | yes | Social account ids per platform (from GET /accounts). At least one non-empty platform array is required. |
| `schedule` | `asap` \| `scheduled` | no |  |
| `scheduledTime` | string | no | Required when schedule is scheduled; must be in the future |
| `timezone` | string | no | IANA timezone, default UTC |
| `caption` | string | no | Post caption/description. Defaults to the suggestion's. |
| `hashtags` | array | no |  |
| `ownerType` | `brand` \| `influencer` | no |  |
| `influencerId` | string | no | Required when ownerType is influencer |
| `tiktokSendToInbox` | boolean | no | Send to TikTok inbox as draft instead of publishing |
| `workspaceId` | string | no | Workspace to operate in. Defaults to the team's default workspace. |

Example:

```bash
curl -s -X POST "https://viraloop.io/api/v1/posts" \
  -H "Authorization: Bearer $VIRALOOP_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"suggestionId":"665f1b2a9c31a2b3c4d5e701","selectedAccounts":{"tiktok":["665f1b2a9c31a2b3c4d5e700"]},"schedule":"asap"}'
```

Response:

```json
{
  "success": true,
  "data": {
    "id": "665f1b2a9c31a2b3c4d5e702",
    "kind": "post",
    "status": "scheduled",
    "renderStatus": "pending",
    "scheduledTime": "2026-07-02T10:05:00.000Z",
    "statusUrl": "/api/v1/posts/665f1b2a9c31a2b3c4d5e702"
  }
}
```

### List posts

`GET /posts`

Lists the workspace's posts with optional status, campaign and scheduled-time filters.

- Scopes: `posts:read`
- Credits: none
- CLI: `viraloop posts list`
- MCP tool: `viraloop_list_posts`

Query parameters:

| Name | Type | Required | Description |
| --- | --- | --- | --- |
| `status` | `draft` \| `scheduled` \| `processing` \| `completed` \| `failed` \| `posted` \| `partial` | no |  |
| `campaignId` | string | no | 24 character hex object id |
| `from` | string | no | scheduledTime lower bound |
| `to` | string | no | scheduledTime upper bound |
| `page` | integer | no | Page number, starting at 1 |
| `limit` | integer | no | Items per page (max 100) |
| `workspaceId` | string | no | Workspace to operate in. Defaults to the team's default workspace. |

Example:

```bash
curl -s "https://viraloop.io/api/v1/posts" \
  -H "Authorization: Bearer $VIRALOOP_API_KEY"
```

Response:

```json
{
  "success": true,
  "data": [
    {
      "id": "665f1b2a9c31a2b3c4d5e702",
      "postType": "video",
      "status": "posted",
      "renderStatus": "ready",
      "scheduledTime": "2026-07-02T10:05:00.000Z",
      "postedAt": "2026-07-02T10:06:03.000Z",
      "statusUrl": "/api/v1/posts/665f1b2a9c31a2b3c4d5e702"
    }
  ],
  "pagination": {
    "total": 1,
    "page": 1,
    "limit": 20,
    "pages": 1
  }
}
```

### Get a post with per-platform results

`GET /posts/{id}`

Returns one post including render state and per-platform posting results (status, post URL, analytics snapshot). Poll this after creating a post: terminal statuses are posted, partial, completed and failed; renderStatus failed is also terminal.

- Scopes: `posts:read`
- Credits: none
- Terminal states: `posted`, `partial`, `completed`, `failed`
- CLI: `viraloop posts get <id>`
- MCP tool: `viraloop_get_post`

Example:

```bash
curl -s "https://viraloop.io/api/v1/posts/<id>" \
  -H "Authorization: Bearer $VIRALOOP_API_KEY"
```

Response:

```json
{
  "success": true,
  "data": {
    "id": "665f1b2a9c31a2b3c4d5e702",
    "postType": "video",
    "status": "posted",
    "renderStatus": "ready",
    "videoUrl": "https://cdn.viraloop.io/renders/abc.mp4",
    "platforms": [
      {
        "platform": "tiktok",
        "status": "posted",
        "postUrl": "https://www.tiktok.com/@acme.hq/video/123",
        "analytics": {
          "views": 1200,
          "likes": 80,
          "comments": 4,
          "shares": 2
        }
      }
    ]
  }
}
```

### Cancel a scheduled post

`DELETE /posts/{id}`

Cancels a post that has not been published yet (status scheduled). Published or in-flight posts cannot be cancelled; that returns 409.

- Scopes: `posts:write`
- Credits: none
- CLI: `viraloop posts cancel <id>`
- MCP tool: `viraloop_cancel_post`

Example:

```bash
curl -s -X DELETE "https://viraloop.io/api/v1/posts/<id>" \
  -H "Authorization: Bearer $VIRALOOP_API_KEY"
```

Response:

```json
{
  "success": true,
  "data": {
    "id": "665f1b2a9c31a2b3c4d5e702",
    "status": "cancelled"
  }
}
```

### Posting calendar

`GET /calendar`

Returns the workspace's posts falling in a date window (by scheduled time, posted time, or creation time for drafts). Useful to check what is already queued before scheduling more.

- Scopes: `posts:read`
- Credits: none
- CLI: `viraloop calendar`

Query parameters:

| Name | Type | Required | Description |
| --- | --- | --- | --- |
| `from` | string | yes | ISO 8601 date or datetime, e.g. 2026-07-03T10:00:00Z |
| `to` | string | yes | ISO 8601 date or datetime, e.g. 2026-07-03T10:00:00Z |
| `status` | `draft` \| `scheduled` \| `processing` \| `completed` \| `failed` \| `posted` \| `partial` | no |  |
| `workspaceId` | string | no | Workspace to operate in. Defaults to the team's default workspace. |

Example:

```bash
curl -s "https://viraloop.io/api/v1/calendar" \
  -H "Authorization: Bearer $VIRALOOP_API_KEY"
```

Response:

```json
{
  "success": true,
  "data": [
    {
      "id": "665f1b2a9c31a2b3c4d5e702",
      "status": "scheduled",
      "scheduledTime": "2026-07-03T10:00:00.000Z",
      "postType": "video",
      "platforms": [
        "tiktok"
      ]
    }
  ]
}
```

## campaigns

### Create a campaign

`POST /campaigns`

Creates a draft campaign: a batch of AI posts generated at once and published on a schedule. Set name, cadence, selectedAccounts and settings here or later via PATCH, then call generate, review the posts, and launch. Monthly campaign quota depends on the plan.

- Scopes: `campaigns:write`
- Credits: none
- Supports `Idempotency-Key` header
- CLI: `viraloop campaigns create`
- MCP tool: `viraloop_create_campaign`

Body fields:

| Name | Type | Required | Description |
| --- | --- | --- | --- |
| `name` | string | yes |  |
| `cadence` | object | no | Posting cadence: how many posts per day, for how many days |
| `selectedAccounts` | object | no | Social account ids per platform (from GET /accounts). At least one non-empty platform array is required. |
| `settings` | object | no | Generation settings (defaults are sensible; all optional) |
| `ownerType` | `brand` \| `influencer` | no |  |
| `influencerId` | string | no | Required when ownerType is influencer |
| `tiktokMode` | `direct` \| `inbox` | no | TikTok publish mode; inbox sends drafts to the inbox |
| `workspaceId` | string | no | Workspace to operate in. Defaults to the team's default workspace. |

Example:

```bash
curl -s -X POST "https://viraloop.io/api/v1/campaigns" \
  -H "Authorization: Bearer $VIRALOOP_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"name":"July launch week","cadence":{"postsPerDay":2,"lengthDays":7,"timezone":"America/New_York"},"selectedAccounts":{"tiktok":["665f1b2a9c31a2b3c4d5e700"]}}'
```

Response:

```json
{
  "success": true,
  "data": {
    "id": "665f1b2a9c31a2b3c4d5e710",
    "name": "July launch week",
    "status": "draft",
    "cadence": {
      "postsPerDay": 2,
      "lengthDays": 7,
      "timezone": "America/New_York",
      "timeSlots": []
    },
    "totalPosts": 0,
    "statusUrl": "/api/v1/campaigns/665f1b2a9c31a2b3c4d5e710"
  }
}
```

### List campaigns

`GET /campaigns`

Lists the workspace's campaigns plus the monthly quota (limit, used, remaining).

- Scopes: `campaigns:read`
- Credits: none
- CLI: `viraloop campaigns list`
- MCP tool: `viraloop_list_campaigns`

Query parameters:

| Name | Type | Required | Description |
| --- | --- | --- | --- |
| `status` | `draft` \| `generating` \| `review` \| `active` \| `completed` \| `cancelled` | no |  |
| `page` | integer | no | Page number, starting at 1 |
| `limit` | integer | no | Items per page (max 100) |
| `workspaceId` | string | no | Workspace to operate in. Defaults to the team's default workspace. |

Example:

```bash
curl -s "https://viraloop.io/api/v1/campaigns" \
  -H "Authorization: Bearer $VIRALOOP_API_KEY"
```

Response:

```json
{
  "success": true,
  "data": {
    "campaigns": [
      {
        "id": "665f1b2a9c31a2b3c4d5e710",
        "name": "July launch week",
        "status": "active",
        "totalPosts": 14,
        "postedCount": 6
      }
    ],
    "quota": {
      "monthlyLimit": 8,
      "usedThisMonth": 2,
      "remaining": 6
    }
  },
  "pagination": {
    "total": 2,
    "page": 1,
    "limit": 20,
    "pages": 1
  }
}
```

### Get a campaign

`GET /campaigns/{id}`

Returns one campaign with its live generatedCount. Poll this after calling generate: the campaign leaves the generating status when done (review on success).

- Scopes: `campaigns:read`
- Credits: none
- Terminal states: `draft`, `review`, `active`, `completed`, `cancelled`
- CLI: `viraloop campaigns get <id>`
- MCP tool: `viraloop_get_campaign`

Example:

```bash
curl -s "https://viraloop.io/api/v1/campaigns/<id>" \
  -H "Authorization: Bearer $VIRALOOP_API_KEY"
```

Response:

```json
{
  "success": true,
  "data": {
    "id": "665f1b2a9c31a2b3c4d5e710",
    "name": "July launch week",
    "status": "review",
    "totalPosts": 14,
    "generatedCount": 14,
    "statusUrl": "/api/v1/campaigns/665f1b2a9c31a2b3c4d5e710"
  }
}
```

### Update a draft campaign

`PATCH /campaigns/{id}`

Updates name, cadence, selectedAccounts, settings, ownerType, influencerId or tiktokMode. Only allowed while the campaign is in draft or review; sub-objects are replaced wholesale.

- Scopes: `campaigns:write`
- Credits: none
- CLI: `viraloop campaigns update <id>`

Body fields:

| Name | Type | Required | Description |
| --- | --- | --- | --- |
| `name` | string | no |  |
| `cadence` | object | no | Posting cadence: how many posts per day, for how many days |
| `selectedAccounts` | object | no | Social account ids per platform (from GET /accounts). At least one non-empty platform array is required. |
| `settings` | object | no | Generation settings (defaults are sensible; all optional) |
| `ownerType` | `brand` \| `influencer` | no |  |
| `influencerId` | string | no | 24 character hex object id |
| `tiktokMode` | `direct` \| `inbox` | no |  |

Example:

```bash
curl -s -X PATCH "https://viraloop.io/api/v1/campaigns/<id>" \
  -H "Authorization: Bearer $VIRALOOP_API_KEY"
```

Response:

```json
{
  "success": true,
  "data": {
    "id": "665f1b2a9c31a2b3c4d5e710",
    "name": "July launch week",
    "status": "draft",
    "selectedAccounts": {
      "tiktok": [
        "665f1b2a9c31a2b3c4d5e700"
      ]
    }
  }
}
```

### Generate the campaign's posts

`POST /campaigns/{id}/generate`

Generates cadence.postsPerDay x cadence.lengthDays AI posts for the campaign, each assigned a schedule slot. Asynchronous: returns 202 immediately; poll the campaign until it reaches review (success) or back to draft (nothing generated). Calling it again while generating is a no-op that reports progress. Regenerating a campaign in review replaces its posts.

- Scopes: `campaigns:write`
- Credits: none
- Rate limit: 5 per 3600s
- Terminal states: `draft`, `review`, `active`, `completed`, `cancelled`
- CLI: `viraloop campaigns generate <id>`
- MCP tool: `viraloop_generate_campaign`

Example:

```bash
curl -s -X POST "https://viraloop.io/api/v1/campaigns/<id>/generate" \
  -H "Authorization: Bearer $VIRALOOP_API_KEY"
```

Response:

```json
{
  "success": true,
  "data": {
    "id": "665f1b2a9c31a2b3c4d5e710",
    "kind": "campaign",
    "status": "generating",
    "totalPosts": 14,
    "generatedCount": 0,
    "statusUrl": "/api/v1/campaigns/665f1b2a9c31a2b3c4d5e710"
  }
}
```

### Launch a reviewed campaign

`POST /campaigns/{id}/launch`

Converts every generated post into a scheduled post at its slot and activates the campaign. Requires status review and at least one selected account. Posts render server-side after this returns and publish automatically at their slots.

- Scopes: `campaigns:write`
- Credits: none
- CLI: `viraloop campaigns launch <id>`
- MCP tool: `viraloop_launch_campaign`

Example:

```bash
curl -s -X POST "https://viraloop.io/api/v1/campaigns/<id>/launch" \
  -H "Authorization: Bearer $VIRALOOP_API_KEY"
```

Response:

```json
{
  "success": true,
  "data": {
    "id": "665f1b2a9c31a2b3c4d5e710",
    "kind": "campaign",
    "status": "active",
    "scheduled": 14,
    "failed": 0,
    "statusUrl": "/api/v1/campaigns/665f1b2a9c31a2b3c4d5e710"
  }
}
```

### Cancel a campaign

`POST /campaigns/{id}/cancel`

Cancels the campaign and removes its unsent posts. Content that already went out stays live. Idempotent: cancelling a cancelled or completed campaign returns cancelledPosts 0.

- Scopes: `campaigns:write`
- Credits: none
- CLI: `viraloop campaigns cancel <id>`
- MCP tool: `viraloop_cancel_campaign`

Example:

```bash
curl -s -X POST "https://viraloop.io/api/v1/campaigns/<id>/cancel" \
  -H "Authorization: Bearer $VIRALOOP_API_KEY"
```

Response:

```json
{
  "success": true,
  "data": {
    "id": "665f1b2a9c31a2b3c4d5e710",
    "status": "cancelled",
    "cancelledPosts": 8
  }
}
```

### List a campaign's generated posts

`GET /campaigns/{id}/posts`

Lists the campaign's generated posts (suggestions) in schedule order, for review before launching. Each carries caption, postCaption, hashtags, rationale and its assigned scheduledTime.

- Scopes: `campaigns:read`
- Credits: none
- CLI: `viraloop campaigns posts <id>`
- MCP tool: `viraloop_list_campaign_posts`

Example:

```bash
curl -s "https://viraloop.io/api/v1/campaigns/<id>/posts" \
  -H "Authorization: Bearer $VIRALOOP_API_KEY"
```

Response:

```json
{
  "success": true,
  "data": [
    {
      "id": "665f1b2a9c31a2b3c4d5e701",
      "format": "walloftext",
      "caption": "POV: the client asks for one small change",
      "postCaption": "Every designer knows this feeling",
      "hashtags": [
        "design"
      ],
      "status": "pending",
      "scheduledTime": "2026-07-06T13:00:00.000Z"
    }
  ]
}
```

## influencers

### List AI influencers

`GET /influencers`

Lists the workspace's AI influencers (virtual personas used to front content).

- Scopes: `influencers:read`
- Credits: none
- CLI: `viraloop influencers list`
- MCP tool: `viraloop_list_influencers`

Query parameters:

| Name | Type | Required | Description |
| --- | --- | --- | --- |
| `search` | string | no | Search name/description/tags |
| `niche` | string | no |  |
| `page` | integer | no | Page number, starting at 1 |
| `limit` | integer | no | Items per page (max 100) |
| `workspaceId` | string | no | Workspace to operate in. Defaults to the team's default workspace. |

Example:

```bash
curl -s "https://viraloop.io/api/v1/influencers" \
  -H "Authorization: Bearer $VIRALOOP_API_KEY"
```

Response:

```json
{
  "success": true,
  "data": [
    {
      "id": "665f1b2a9c31a2b3c4d5e720",
      "name": "Maya",
      "niche": "fitness",
      "imageUrl": "https://cdn.viraloop.io/influencers/maya.jpg",
      "turboEnabled": true
    }
  ],
  "pagination": {
    "total": 3,
    "page": 1,
    "limit": 20,
    "pages": 1
  }
}
```

### Create an AI influencer

`POST /influencers`

Creates an influencer from a base image you provide (a publicly reachable photo URL). A short animated preview is generated in the background when the team has credits. To generate a base image from a prompt instead, use the web app.

- Scopes: `influencers:write`
- Credits: 10 credits for the optional animated preview (skipped when out of credits)
- Rate limit: 10 per 3600s
- Supports `Idempotency-Key` header
- CLI: `viraloop influencers create`
- MCP tool: `viraloop_create_influencer`

Body fields:

| Name | Type | Required | Description |
| --- | --- | --- | --- |
| `name` | string | yes |  |
| `imageUrl` | string | yes | Publicly reachable base photo of the persona |
| `description` | string | no |  |
| `niche` | string | no | One of the app's niche slugs, e.g. fitness, tech, food |
| `gender` | `male` \| `female` | no |  |
| `age` | integer | no |  |
| `ethnicity` | string | no |  |
| `tags` | array | no |  |
| `workspaceId` | string | no | Workspace to operate in. Defaults to the team's default workspace. |

Example:

```bash
curl -s -X POST "https://viraloop.io/api/v1/influencers" \
  -H "Authorization: Bearer $VIRALOOP_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"name":"Maya","imageUrl":"https://example.com/maya.jpg","niche":"fitness","gender":"female"}'
```

Response:

```json
{
  "success": true,
  "data": {
    "id": "665f1b2a9c31a2b3c4d5e720",
    "name": "Maya",
    "niche": "fitness",
    "imageUrl": "https://example.com/maya.jpg",
    "videoPreview": {
      "status": "pending",
      "videoUrl": "",
      "thumbnailUrl": ""
    }
  }
}
```

### Get an influencer

`GET /influencers/{id}`

Returns one influencer including its base image, preview video state and Turbo settings.

- Scopes: `influencers:read`
- Credits: none
- CLI: `viraloop influencers get <id>`

Example:

```bash
curl -s "https://viraloop.io/api/v1/influencers/<id>" \
  -H "Authorization: Bearer $VIRALOOP_API_KEY"
```

Response:

```json
{
  "success": true,
  "data": {
    "id": "665f1b2a9c31a2b3c4d5e720",
    "name": "Maya",
    "description": "Energetic fitness coach",
    "niche": "fitness",
    "imageUrl": "https://cdn.viraloop.io/influencers/maya.jpg",
    "turboEnabled": true,
    "allowedAngles": []
  }
}
```

## videos

### Generate a talking-head video

`POST /influencers/{id}/videos`

Generates a talking-head UGC video of the influencer speaking your script (Seedance 2, 9:16). Costs 20 credits, deducted up front and refunded automatically if generation fails. Insufficient credits returns HTTP 402 (insufficient_credits). Asynchronous: returns 202 with the video in processing; poll GET /videos/{videoId} until completed or failed (typically 2 to 10 minutes).

- Scopes: `influencers:write`
- Credits: 20 credits per video
- Rate limit: 10 per 600s
- Supports `Idempotency-Key` header
- Terminal states: `completed`, `failed`
- CLI: `viraloop videos create --influencer <id>`
- MCP tool: `viraloop_create_video`

Body fields:

| Name | Type | Required | Description |
| --- | --- | --- | --- |
| `script` | string | yes | What the influencer says (max 1000 chars) |
| `imageUrl` | string | no | Reference image; defaults to the influencer's base image |
| `language` | string | no | Spoken language, default English |
| `duration` | integer | no |  |
| `mode` | `speaker` \| `scene` | no | speaker: talking head; scene: wider shot |

Example:

```bash
curl -s -X POST "https://viraloop.io/api/v1/influencers/<id>/videos" \
  -H "Authorization: Bearer $VIRALOOP_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"script":"Three things I wish I knew before my first marathon","duration":10}'
```

Response:

```json
{
  "success": true,
  "data": {
    "id": "665f1b2a9c31a2b3c4d5e730",
    "kind": "video",
    "influencerId": "665f1b2a9c31a2b3c4d5e720",
    "status": "processing",
    "model": "seedance_2",
    "creditsUsed": 20,
    "statusUrl": "/api/v1/videos/665f1b2a9c31a2b3c4d5e730"
  }
}
```

### List an influencer's videos

`GET /influencers/{id}/videos`

Lists the influencer's generated videos, newest first, with status and URLs.

- Scopes: `influencers:read`
- Credits: none
- CLI: `viraloop videos list --influencer <id>`

Query parameters:

| Name | Type | Required | Description |
| --- | --- | --- | --- |
| `status` | `pending` \| `processing` \| `completed` \| `failed` | no |  |
| `page` | integer | no | Page number, starting at 1 |
| `limit` | integer | no | Items per page (max 100) |

Example:

```bash
curl -s "https://viraloop.io/api/v1/influencers/<id>/videos" \
  -H "Authorization: Bearer $VIRALOOP_API_KEY"
```

Response:

```json
{
  "success": true,
  "data": [
    {
      "id": "665f1b2a9c31a2b3c4d5e730",
      "status": "completed",
      "videoUrl": "https://cdn.viraloop.io/videos/abc.mp4",
      "thumbnailUrl": "https://cdn.viraloop.io/videos/abc.jpg"
    }
  ],
  "pagination": {
    "total": 1,
    "page": 1,
    "limit": 20,
    "pages": 1
  }
}
```

### Get a video's status

`GET /videos/{id}`

Returns one generated video. Poll this after creating a video: terminal statuses are completed (videoUrl set) and failed (error set). A completed video can be posted via POST /posts with videoUrl.

- Scopes: `influencers:read`
- Credits: none
- Terminal states: `completed`, `failed`
- CLI: `viraloop videos get <id>`
- MCP tool: `viraloop_get_video`

Example:

```bash
curl -s "https://viraloop.io/api/v1/videos/<id>" \
  -H "Authorization: Bearer $VIRALOOP_API_KEY"
```

Response:

```json
{
  "success": true,
  "data": {
    "id": "665f1b2a9c31a2b3c4d5e730",
    "status": "completed",
    "videoUrl": "https://cdn.viraloop.io/videos/abc.mp4",
    "thumbnailUrl": "https://cdn.viraloop.io/videos/abc.jpg",
    "completedAt": "2026-07-02T12:08:00.000Z"
  }
}
```

## assets

### List media assets

`GET /assets`

Lists the workspace's uploaded media library (videos and images used as generation backgrounds and sources). Uploading happens in the web app in v1.

- Scopes: `assets:read`
- Credits: none
- CLI: `viraloop assets list`
- MCP tool: `viraloop_list_assets`

Query parameters:

| Name | Type | Required | Description |
| --- | --- | --- | --- |
| `kind` | `video` \| `image` \| `audio` | no |  |
| `category` | string | no |  |
| `turboEnabled` | boolean | no |  |
| `page` | integer | no | Page number, starting at 1 |
| `limit` | integer | no | Items per page (max 100) |
| `workspaceId` | string | no | Workspace to operate in. Defaults to the team's default workspace. |

Example:

```bash
curl -s "https://viraloop.io/api/v1/assets" \
  -H "Authorization: Bearer $VIRALOOP_API_KEY"
```

Response:

```json
{
  "success": true,
  "data": [
    {
      "id": "665f1b2a9c31a2b3c4d5e740",
      "kind": "video",
      "category": "wall-of-text",
      "name": "gym-broll.mp4",
      "url": "https://cdn.viraloop.io/assets/gym-broll.mp4",
      "turboEnabled": true
    }
  ],
  "pagination": {
    "total": 12,
    "page": 1,
    "limit": 20,
    "pages": 1
  }
}
```
