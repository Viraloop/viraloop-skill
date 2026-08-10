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

Returns the team's current credit balance plus a paginated ledger of credit movements (top-ups, spending, rewards, refunds). Turbo content generation does not consume credits; influencer media generation does (createInfluencerVideo: 5 credits per second of video, createInfluencer preview: 10). Failed generations are refunded automatically; running out of credits returns HTTP 402 with error type insufficient_credits.

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

### Create a workspace

`POST /workspaces`

Creates a workspace (brand) under the team. Provide the brand's website and the brand profile (identity, positioning, content angles) is generated in the background: poll getWorkspace until brandContext.status is ready. One domain can be claimed by only one workspace across all of Viraloop.

- Scopes: `workspaces:write`
- Credits: none
- Rate limit: 60 per 3600s
- Supports `Idempotency-Key` header
- CLI: `viraloop workspaces create`
- MCP tool: `viraloop_create_workspace`

Body fields:

| Name | Type | Required | Description |
| --- | --- | --- | --- |
| `name` | string | yes |  |
| `websiteUrl` | string | no | The brand's website; seeds the auto-generated brand profile |
| `logoUrl` | string | no |  |

Example:

```bash
curl -s -X POST "https://viraloop.io/api/v1/workspaces" \
  -H "Authorization: Bearer $VIRALOOP_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"name":"Acme","websiteUrl":"https://acme.com"}'
```

Response:

```json
{
  "success": true,
  "data": {
    "id": "665f1b2a9c31a2b3c4d5e6f9",
    "name": "Acme",
    "isDefault": false,
    "website": {
      "url": "https://acme.com",
      "domain": "acme.com"
    },
    "preferences": {
      "timezone": "",
      "contentLanguage": "English"
    }
  }
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

Generates a batch of ready-to-post content suggestions (video decks with captions, hashtags and rationale) using the workspace's brand context and Turbo configuration. Pass prompt to set the subject, and format to pick wall of text, slideshow or green screen meme. Synchronous: the request returns when generation finishes, typically 5 to 60 seconds depending on count. Review the results, then publish the ones you like via POST /posts with suggestionId.

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
| `prompt` | string | no | What the batch should be about, e.g. 'why founders burn out on content'. Omit to let Turbo pick from the workspace's content angles. |
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

Lists generated content suggestions in the workspace queue (automation-owned suggestions are excluded). Filter by status: pending (awaiting review), accepted, scheduled.

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

Returns one suggestion including its full deck (the editable render blueprint), influencer, angle, rationale and remix source. The deck IS the free preview: render its slide images / clips with the caption text boxes overlaid client-side to let a user accept or skip. Previewing and accepting are free; billing only happens when media is downloaded or published.

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

### Accept a generation into the library

`POST /generations/{id}/accept`

Accepts a generated suggestion and saves it to the content library, returning a contentId. That id is what GET /content/{id}/download (fetch the media files) and POST /posts (publish) operate on. Idempotent: re-accepting returns the same contentId. Counts toward the plan's monthly content allowance.

- Scopes: `generations:write`
- Credits: none
- Supports `Idempotency-Key` header
- CLI: `viraloop generations accept <id>`
- MCP tool: `viraloop_accept_generation`

Query parameters:

| Name | Type | Required | Description |
| --- | --- | --- | --- |
| `workspaceId` | string | no | Workspace to operate in. Defaults to the team's default workspace. |

Example:

```bash
curl -s -X POST "https://viraloop.io/api/v1/generations/<id>/accept" \
  -H "Authorization: Bearer $VIRALOOP_API_KEY"
```

Response:

```json
{
  "success": true,
  "data": {
    "id": "665f1b2a9c31a2b3c4d5e711",
    "status": "accepted",
    "contentId": "665f1b2a9c31a2b3c4d5e757"
  }
}
```

## posts

### Create and schedule a post

`POST /posts`

Schedules a post to one or more connected accounts. Four content sources, checked in order: suggestionId (publish a generated suggestion), contentId (publish a saved studio video from any /content operation), deck + format (publish a raw deck), or videoUrl / images (publish your own media). Deck-based posts are rendered server-side after this call returns; the response is 202 with a statusUrl to poll. schedule asap posts as soon as the render is ready; schedule scheduled requires a future scheduledTime. Supports the Idempotency-Key header.

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
| `contentId` | string | no | Publish a saved studio video (from any /content operation). Finished videos post as-is; deck formats render first. |
| `deck` | object | no | Raw deck object (advanced; usually use suggestionId or contentId) |
| `format` | `walloftext` \| `slideshow` \| `greenscreen` \| `grid2x2` \| `listicle` \| `askmeanything` \| `ranking` \| `singlefadein` \| `videohookdemo` \| `talkingheadgreenscreen` | no | Deck format, required when deck is provided |
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

Lists the workspace's posts with optional status, automation and scheduled-time filters.

- Scopes: `posts:read`
- Credits: none
- CLI: `viraloop posts list`
- MCP tool: `viraloop_list_posts`

Query parameters:

| Name | Type | Required | Description |
| --- | --- | --- | --- |
| `status` | `draft` \| `scheduled` \| `processing` \| `completed` \| `failed` \| `posted` \| `partial` | no |  |
| `automationId` | string | no | 24 character hex object id |
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

## automations

### Create an automation

`POST /automations`

Creates a draft automation: a batch of AI posts generated at once and published on a schedule. Set name, cadence, selectedAccounts and settings here or later via PATCH, then call generate, review the posts, and launch. Monthly automation quota depends on the plan.

- Scopes: `automations:write`
- Credits: none
- Supports `Idempotency-Key` header
- CLI: `viraloop automations create`
- MCP tool: `viraloop_create_automation`

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
| `tiktokOptions` | object | no | TikTok publish options (direct posting only) |
| `workspaceId` | string | no | Workspace to operate in. Defaults to the team's default workspace. |

Example:

```bash
curl -s -X POST "https://viraloop.io/api/v1/automations" \
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
    "statusUrl": "/api/v1/automations/665f1b2a9c31a2b3c4d5e710"
  }
}
```

### List automations

`GET /automations`

Lists the workspace's automations plus the monthly quota (limit, used, remaining).

- Scopes: `automations:read`
- Credits: none
- CLI: `viraloop automations list`
- MCP tool: `viraloop_list_automations`

Query parameters:

| Name | Type | Required | Description |
| --- | --- | --- | --- |
| `status` | `draft` \| `generating` \| `review` \| `active` \| `completed` \| `cancelled` | no |  |
| `page` | integer | no | Page number, starting at 1 |
| `limit` | integer | no | Items per page (max 100) |
| `workspaceId` | string | no | Workspace to operate in. Defaults to the team's default workspace. |

Example:

```bash
curl -s "https://viraloop.io/api/v1/automations" \
  -H "Authorization: Bearer $VIRALOOP_API_KEY"
```

Response:

```json
{
  "success": true,
  "data": {
    "automations": [
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

### Get an automation

`GET /automations/{id}`

Returns one automation with its live generatedCount. Poll this after calling generate: the automation leaves the generating status when done (review on success).

- Scopes: `automations:read`
- Credits: none
- Terminal states: `draft`, `review`, `active`, `completed`, `cancelled`
- CLI: `viraloop automations get <id>`
- MCP tool: `viraloop_get_automation`

Example:

```bash
curl -s "https://viraloop.io/api/v1/automations/<id>" \
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
    "statusUrl": "/api/v1/automations/665f1b2a9c31a2b3c4d5e710"
  }
}
```

### Update a draft automation

`PATCH /automations/{id}`

Updates name, cadence, selectedAccounts, settings, ownerType, influencerId or tiktokMode. Only allowed while the automation is in draft or review; sub-objects are replaced wholesale.

- Scopes: `automations:write`
- Credits: none
- CLI: `viraloop automations update <id>`

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
| `tiktokOptions` | object | no | TikTok publish options (direct posting only) |

Example:

```bash
curl -s -X PATCH "https://viraloop.io/api/v1/automations/<id>" \
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

### Generate the automation's posts

`POST /automations/{id}/generate`

Generates cadence.postsPerDay x cadence.lengthDays AI posts for the automation, each assigned a schedule slot. Asynchronous: returns 202 immediately; poll the automation until it reaches review (success) or back to draft (nothing generated). Calling it again while generating is a no-op that reports progress. Regenerating an automation in review replaces its posts.

- Scopes: `automations:write`
- Credits: none
- Rate limit: 5 per 3600s
- Terminal states: `draft`, `review`, `active`, `completed`, `cancelled`
- CLI: `viraloop automations generate <id>`
- MCP tool: `viraloop_generate_automation`

Example:

```bash
curl -s -X POST "https://viraloop.io/api/v1/automations/<id>/generate" \
  -H "Authorization: Bearer $VIRALOOP_API_KEY"
```

Response:

```json
{
  "success": true,
  "data": {
    "id": "665f1b2a9c31a2b3c4d5e710",
    "kind": "automation",
    "status": "generating",
    "totalPosts": 14,
    "generatedCount": 0,
    "statusUrl": "/api/v1/automations/665f1b2a9c31a2b3c4d5e710"
  }
}
```

### Extend an automation by more days

`POST /automations/{id}/extend`

Adds `days` more days to an active or completed automation: generates cadence.postsPerDay x days new posts, schedules them in the days after the current window and publishes them to the automation's accounts. Posts already scheduled or published are never touched. Extending a completed automation revives it to active. Asynchronous: returns 202 immediately; the automation carries extending=true until the new posts land, so poll GET /automations/{id} until it flips false.

- Scopes: `automations:write`
- Credits: none
- Rate limit: 5 per 3600s
- CLI: `viraloop automations extend <id> --days <n>`
- MCP tool: `viraloop_extend_automation`

Body fields:

| Name | Type | Required | Description |
| --- | --- | --- | --- |
| `days` | integer | yes | How many days to add to the automation window |
| `workspaceId` | string | no | Workspace to operate in. Defaults to the team's default workspace. |

Example:

```bash
curl -s -X POST "https://viraloop.io/api/v1/automations/<id>/extend" \
  -H "Authorization: Bearer $VIRALOOP_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"days":7}'
```

Response:

```json
{
  "success": true,
  "data": {
    "id": "665f1b2a9c31a2b3c4d5e710",
    "kind": "automation",
    "status": "active",
    "extending": true,
    "adding": 7,
    "statusUrl": "/api/v1/automations/665f1b2a9c31a2b3c4d5e710"
  }
}
```

### Launch a reviewed automation

`POST /automations/{id}/launch`

Converts every generated post into a scheduled post at its slot and activates the automation. Requires status review and at least one selected account. Posts render server-side after this returns and publish automatically at their slots.

- Scopes: `automations:write`
- Credits: none
- CLI: `viraloop automations launch <id>`
- MCP tool: `viraloop_launch_automation`

Example:

```bash
curl -s -X POST "https://viraloop.io/api/v1/automations/<id>/launch" \
  -H "Authorization: Bearer $VIRALOOP_API_KEY"
```

Response:

```json
{
  "success": true,
  "data": {
    "id": "665f1b2a9c31a2b3c4d5e710",
    "kind": "automation",
    "status": "active",
    "scheduled": 14,
    "failed": 0,
    "statusUrl": "/api/v1/automations/665f1b2a9c31a2b3c4d5e710"
  }
}
```

### Cancel an automation

`POST /automations/{id}/cancel`

Cancels the automation and removes its unsent posts. Content that already went out stays live. Idempotent: cancelling a cancelled or completed automation returns cancelledPosts 0.

- Scopes: `automations:write`
- Credits: none
- CLI: `viraloop automations cancel <id>`
- MCP tool: `viraloop_cancel_automation`

Example:

```bash
curl -s -X POST "https://viraloop.io/api/v1/automations/<id>/cancel" \
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

### List an automation's generated posts

`GET /automations/{id}/posts`

Lists the automation's generated posts (suggestions) in schedule order, for review before launching. Each carries caption, postCaption, hashtags, rationale and its assigned scheduledTime.

- Scopes: `automations:read`
- Credits: none
- CLI: `viraloop automations posts <id>`
- MCP tool: `viraloop_list_automation_posts`

Example:

```bash
curl -s "https://viraloop.io/api/v1/automations/<id>/posts" \
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

Generates a talking-head UGC video of the influencer speaking your script (Seedance 2, 9:16). Costs 5 credits per second of video (default 10s = 50 credits), deducted up front and refunded automatically if generation fails. Insufficient credits returns HTTP 402 (insufficient_credits). Asynchronous: returns 202 with the video in processing; poll GET /videos/{videoId} until completed or failed (typically 2 to 10 minutes).

- Scopes: `influencers:write`
- Credits: 5 credits per second (duration 5-30s; default 10s = 50 credits)
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
    "creditsUsed": 50,
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

### Upload media into the workspace

`POST /assets`

Ingests a publicly reachable file into the workspace's media bank and returns a stable hosted asset. Use this to get a source video, a character photo or an app screenshot into Viraloop before generating a studio format that needs one. The source URL is fetched server-side, so it must be publicly readable (no auth headers, no signed-cookie hosts) and stay up until the request returns. Synchronous.

- Scopes: `assets:write`
- Credits: none
- Rate limit: 30 per 300s
- Supports `Idempotency-Key` header
- CLI: `viraloop assets upload --url <url>`
- MCP tool: `viraloop_upload_asset`

Body fields:

| Name | Type | Required | Description |
| --- | --- | --- | --- |
| `url` | string | yes | Publicly reachable https URL of the file to ingest |
| `kind` | `video` \| `image` \| `audio` | no | video, image or audio. Inferred from the response content-type when omitted. |
| `name` | string | no | Display name in the media bank |
| `category` | string | no | Media Bank category |
| `workspaceId` | string | no | Workspace to operate in. Defaults to the team's default workspace. |

Example:

```bash
curl -s -X POST "https://viraloop.io/api/v1/assets" \
  -H "Authorization: Bearer $VIRALOOP_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"url":"https://example.com/clips/reaction.mp4","kind":"video","name":"reaction.mp4"}'
```

Response:

```json
{
  "success": true,
  "data": {
    "id": "665f1b2a9c31a2b3c4d5e741",
    "kind": "video",
    "name": "reaction.mp4",
    "url": "https://cdn.viraloop.io/assets/reaction.mp4"
  }
}
```

## content

### Swap the character in a video

`POST /content/character-swap`

Replaces the person in a source clip with your character, driven by the clip's motion (Kling motion control, 9:16). Both media URLs must be hosted; upload them with POST /assets first. Costs 2 credits per second of source video, plus 3 credits when scene is 'recreate' (an extra image edit that composites your character into the clip's opening frame). Credits are deducted up front and refunded automatically if generation fails; insufficient credits returns HTTP 402. Asynchronous: returns 202, then poll GET /content/{id} until ready or failed (typically 3 to 20 minutes).

- Scopes: `generations:write`
- Credits: 2 credits per second of source video, +3 when scene is 'recreate'
- Rate limit: 10 per 300s
- Supports `Idempotency-Key` header
- Terminal states: `ready`, `failed`
- CLI: `viraloop content character-swap --video <url> --character <url>`
- MCP tool: `viraloop_create_character_swap`

Body fields:

| Name | Type | Required | Description |
| --- | --- | --- | --- |
| `videoUrl` | string | yes | Hosted source clip, 3-30 seconds. Its length sets the output length and price. |
| `characterImageUrl` | string | yes | Hosted photo of the character to swap in; should clearly show one face |
| `scene` | `image` \| `recreate` | no | image (default): keep the character photo's own background, borrow only the clip's motion. recreate: composite the character into the clip's first frame instead, keeping the clip's scene but a still background. |
| `keepSourceAudio` | boolean | no | Lay the clip's original audio over the result. Default true. |
| `influencerId` | string | no | Link the result to this influencer's gallery |
| `name` | string | no |  |
| `workspaceId` | string | no | Workspace to operate in. Defaults to the team's default workspace. |

Example:

```bash
curl -s -X POST "https://viraloop.io/api/v1/content/character-swap" \
  -H "Authorization: Bearer $VIRALOOP_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"videoUrl":"https://cdn.viraloop.io/assets/reaction.mp4","characterImageUrl":"https://cdn.viraloop.io/assets/maya.jpg","scene":"image"}'
```

Response:

```json
{
  "success": true,
  "data": {
    "id": "665f1b2a9c31a2b3c4d5e750",
    "format": "Character Swap",
    "status": "processing",
    "creditsUsed": 20
  }
}
```

### Generate a presenter holding your app

`POST /content/green-screen-mobile`

Generates a UGC video of a presenter holding a phone with your app on screen, speaking your script (9:16). Runs two steps server-side: one image edit puts the phone in the presenter's hand, then Seedance animates it. Both media URLs must be hosted; upload them with POST /assets first. Costs 3 credits for the composite plus 5 credits per second of video (default 8s = 43 credits), deducted up front and refunded automatically on failure; insufficient credits returns HTTP 402. Asynchronous: returns 202, then poll GET /content/{id} until ready or failed (typically 2 to 10 minutes).

- Scopes: `generations:write`
- Credits: 3 credits + 5 credits per second (duration 4-15s; default 8s = 43 credits)
- Rate limit: 10 per 300s
- Supports `Idempotency-Key` header
- Terminal states: `ready`, `failed`
- CLI: `viraloop content green-screen-mobile --screenshot <url> --presenter <url>`
- MCP tool: `viraloop_create_green_screen_mobile`

Body fields:

| Name | Type | Required | Description |
| --- | --- | --- | --- |
| `appScreenshot` | string | yes | Hosted screenshot of your app; it is composited onto the phone screen as-is |
| `presenterImage` | string | yes | Hosted photo of the presenter; should clearly show their face |
| `script` | string | yes | What the presenter says about your app (max 1000 chars) |
| `duration` | integer | no | Seconds, default 8 |
| `language` | string | no | Spoken language, default English |
| `captionOverlay` | boolean | no | Transcribe the speech into a styled caption track. Default true. |
| `influencerId` | string | no | Link the result to this influencer |
| `name` | string | no |  |
| `workspaceId` | string | no | Workspace to operate in. Defaults to the team's default workspace. |

Example:

```bash
curl -s -X POST "https://viraloop.io/api/v1/content/green-screen-mobile" \
  -H "Authorization: Bearer $VIRALOOP_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"appScreenshot":"https://cdn.viraloop.io/assets/app-home.png","presenterImage":"https://cdn.viraloop.io/assets/maya.jpg","script":"This is the fastest way to ship short-form video.","duration":8}'
```

Response:

```json
{
  "success": true,
  "data": {
    "id": "665f1b2a9c31a2b3c4d5e751",
    "format": "green-screen-mobile",
    "status": "processing",
    "creditsUsed": 43
  }
}
```

### Clone a video's motion onto your character

`POST /content/clone-video`

Drives your character with the motion of a reference clip (Kling motion control, 9:16). Unlike character swap this keeps your character's own scene throughout; it borrows only the movement. Both media URLs must be hosted; upload them with POST /assets first. Costs 2 credits per second of reference video, deducted up front and refunded automatically on failure; insufficient credits returns HTTP 402. Asynchronous: returns 202, then poll GET /content/{id} until ready or failed (typically 3 to 20 minutes).

- Scopes: `generations:write`
- Credits: 2 credits per second of reference video
- Rate limit: 10 per 300s
- Supports `Idempotency-Key` header
- Terminal states: `ready`, `failed`
- CLI: `viraloop content clone-video --image <url> --video <url>`
- MCP tool: `viraloop_create_clone_video`

Body fields:

| Name | Type | Required | Description |
| --- | --- | --- | --- |
| `imageUrl` | string | yes | Hosted photo of the character to animate |
| `videoUrl` | string | yes | Hosted reference clip whose motion is copied. Its length sets the price. |
| `videoDuration` | number | no | Reference clip length in seconds. Probed from the file when omitted. |
| `influencerId` | string | no | Also save the result to this influencer's gallery |
| `name` | string | no |  |
| `workspaceId` | string | no | Workspace to operate in. Defaults to the team's default workspace. |

Example:

```bash
curl -s -X POST "https://viraloop.io/api/v1/content/clone-video" \
  -H "Authorization: Bearer $VIRALOOP_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"imageUrl":"https://cdn.viraloop.io/assets/maya.jpg","videoUrl":"https://cdn.viraloop.io/assets/dance.mp4"}'
```

Response:

```json
{
  "success": true,
  "data": {
    "id": "665f1b2a9c31a2b3c4d5e751",
    "format": "Clone Video",
    "status": "processing",
    "statusUrl": "/api/v1/content/665f1b2a9c31a2b3c4d5e751"
  }
}
```

### Generate a talking-head UGC video

`POST /content/talking-head`

Generates a UGC video of a person speaking your script to camera (Seedance, 9:16, with voice). Describe the person with gender/age/ethnicity/appearance, or pin their exact likeness with avatarImageUrl (a hosted photo; upload one with POST /assets). Costs 5 credits per second (default 10s = 50 credits), deducted up front and refunded automatically on failure; insufficient credits returns HTTP 402. Asynchronous: returns 202, then poll GET /content/{id} until ready or failed (typically 2 to 10 minutes).

- Scopes: `generations:write`
- Credits: 5 credits per second (duration 4-15s; default 10s = 50 credits)
- Rate limit: 20 per 300s
- Supports `Idempotency-Key` header
- Terminal states: `ready`, `failed`
- CLI: `viraloop content talking-head --script <text>`
- MCP tool: `viraloop_create_talking_head`

Body fields:

| Name | Type | Required | Description |
| --- | --- | --- | --- |
| `script` | string | yes | What the person says (max 1000 chars) |
| `avatarImageUrl` | string | no | Hosted photo pinning the speaker's likeness. Overrides the persona fields. |
| `duration` | integer | no | Seconds, default 10 |
| `language` | string | no | Spoken language, default English |
| `mode` | `speaker` \| `scene` | no | speaker (default): head-and-shoulders to camera. scene: a wider shot. |
| `gender` | string | no | Persona hint, ignored when avatarImageUrl is set |
| `age` | string | no | Persona hint, ignored when avatarImageUrl is set |
| `ethnicity` | string | no | Persona hint, ignored when avatarImageUrl is set |
| `appearance` | string | no | Free-text look, ignored when avatarImageUrl is set |
| `referenceImages` | array | no | Extra hosted reference images (props, setting) |
| `captionOverlay` | boolean | no | Transcribe the speech into a styled caption track. Default true. |
| `influencerId` | string | no | Link the result to this influencer |
| `name` | string | no |  |
| `workspaceId` | string | no | Workspace to operate in. Defaults to the team's default workspace. |

Example:

```bash
curl -s -X POST "https://viraloop.io/api/v1/content/talking-head" \
  -H "Authorization: Bearer $VIRALOOP_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"script":"I stopped writing captions by hand and my reach doubled.","duration":10,"gender":"woman","age":"20s"}'
```

Response:

```json
{
  "success": true,
  "data": {
    "id": "665f1b2a9c31a2b3c4d5e752",
    "format": "Talking Head UGC",
    "status": "processing",
    "statusUrl": "/api/v1/content/665f1b2a9c31a2b3c4d5e752"
  }
}
```

### Generate a presenter over your demo video

`POST /content/talking-head-green-screen`

Generates a presenter speaking your script on a green screen, then composites them into the corner of your demo video at render time (9:16). Both media URLs must be hosted; upload them with POST /assets first. Costs 5 credits per second (default 10s = 50 credits), deducted up front and refunded automatically on failure; insufficient credits returns HTTP 402. Asynchronous: returns 202, then poll GET /content/{id} until ready or failed (typically 2 to 15 minutes).

- Scopes: `generations:write`
- Credits: 5 credits per second (duration 4-15s; default 10s = 50 credits)
- Rate limit: 20 per 300s
- Supports `Idempotency-Key` header
- Terminal states: `ready`, `failed`
- CLI: `viraloop content talking-head-green-screen --script <text> --avatar <url> --demo <url>`
- MCP tool: `viraloop_create_talking_head_green_screen`

Body fields:

| Name | Type | Required | Description |
| --- | --- | --- | --- |
| `script` | string | yes | What the presenter says (max 1000 chars) |
| `avatarImageUrl` | string | yes | Hosted photo of the presenter |
| `demoVideoUrl` | string | yes | Hosted demo/screen-recording that plays behind |
| `demoThumb` | string | no | Poster image for the demo video |
| `avatarPosition` | `bottom-left` \| `bottom-right` | no | Which corner the presenter sits in. Default bottom-right. |
| `duration` | integer | no | Seconds, default 10 |
| `language` | string | no | Spoken language, default English |
| `captionOverlay` | boolean | no | Transcribe the speech into a styled caption track. Default false. |
| `influencerId` | string | no | Link the result to this influencer |
| `name` | string | no |  |
| `workspaceId` | string | no | Workspace to operate in. Defaults to the team's default workspace. |

Example:

```bash
curl -s -X POST "https://viraloop.io/api/v1/content/talking-head-green-screen" \
  -H "Authorization: Bearer $VIRALOOP_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"script":"Here is how the scheduler actually works.","avatarImageUrl":"https://cdn.viraloop.io/assets/maya.jpg","demoVideoUrl":"https://cdn.viraloop.io/assets/product-demo.mp4"}'
```

Response:

```json
{
  "success": true,
  "data": {
    "id": "665f1b2a9c31a2b3c4d5e753",
    "format": "Talking Head Green Screen",
    "status": "processing",
    "statusUrl": "/api/v1/content/665f1b2a9c31a2b3c4d5e753"
  }
}
```

### Generate a spokesperson holding your product

`POST /content/product-spokesperson`

Generates a UGC video of a person holding your product and talking about it (Seedance, 9:16, with voice). Give either spokespersonImage (a shot of someone already holding it) or avatarImage plus productImage, which are composed into one first. All media URLs must be hosted; upload them with POST /assets first. Costs 5 credits per second (default 8s = 40 credits) plus 3 credits when we compose the shot, deducted up front and refunded automatically on failure; insufficient credits returns HTTP 402. Asynchronous: returns 202, then poll GET /content/{id} until ready or failed (typically 2 to 10 minutes).

- Scopes: `generations:write`
- Credits: 5 credits per second (duration 4-15s; default 8s = 40 credits), +3 when composing the shot
- Rate limit: 20 per 300s
- Supports `Idempotency-Key` header
- Terminal states: `ready`, `failed`
- CLI: `viraloop content product-spokesperson --script <text> --product <url>`
- MCP tool: `viraloop_create_product_spokesperson`

Body fields:

| Name | Type | Required | Description |
| --- | --- | --- | --- |
| `script` | string | yes | What the spokesperson says (max 1000 chars) |
| `spokespersonImage` | string | no | Hosted photo of a person already holding the product |
| `avatarImage` | string | no | Hosted photo of the person, composed with productImage |
| `productImage` | string | no | Hosted photo of the product, composed with avatarImage |
| `instruction` | string | no | Extra direction for the composed shot, e.g. 'outdoors, morning light' |
| `duration` | integer | no | Seconds, default 8 |
| `language` | string | no | Spoken language, default English |
| `captionOverlay` | boolean | no | Transcribe the speech into a styled caption track. Default true. |
| `influencerId` | string | no | Link the result to this influencer |
| `name` | string | no |  |
| `workspaceId` | string | no | Workspace to operate in. Defaults to the team's default workspace. |

Example:

```bash
curl -s -X POST "https://viraloop.io/api/v1/content/product-spokesperson" \
  -H "Authorization: Bearer $VIRALOOP_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"script":"This is the only bottle I take to the gym now.","avatarImage":"https://cdn.viraloop.io/assets/maya.jpg","productImage":"https://cdn.viraloop.io/assets/bottle.png"}'
```

Response:

```json
{
  "success": true,
  "data": {
    "id": "665f1b2a9c31a2b3c4d5e754",
    "format": "Product Spokesperson",
    "status": "processing",
    "statusUrl": "/api/v1/content/665f1b2a9c31a2b3c4d5e754"
  }
}
```

### Create a 2x2 grid video

`POST /content/grid-video`

Writes a listicle heading plus four labelled cells from your brand and prompt, finds a stock photo for each cell, and saves the result to your library (9:16). Costs no credits. Synchronous: the request returns when the deck is built, typically 10 to 40 seconds. Publish it with POST /posts using the deck from GET /content/{id}, or open it in the studio to edit first.

- Scopes: `generations:write`
- Credits: none
- Rate limit: 20 per 300s
- Supports `Idempotency-Key` header
- CLI: `viraloop content grid-video --prompt <text>`
- MCP tool: `viraloop_create_grid_video`

Body fields:

| Name | Type | Required | Description |
| --- | --- | --- | --- |
| `prompt` | string | no | What the grid is about. Omit to let the model pick from your brand. |
| `mentionBusiness` | boolean | no | Make one of the four cells your brand. Default true. |
| `portrait` | boolean | no | Use portrait cell photos instead of landscape. Default false. |
| `name` | string | no |  |
| `workspaceId` | string | no | Workspace to operate in. Defaults to the team's default workspace. |

Example:

```bash
curl -s -X POST "https://viraloop.io/api/v1/content/grid-video" \
  -H "Authorization: Bearer $VIRALOOP_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"prompt":"tools every solo founder should be using"}'
```

Response:

```json
{
  "success": true,
  "data": {
    "id": "665f1b2a9c31a2b3c4d5e755",
    "format": "2x2 Grid Video",
    "status": "ready",
    "title": "tools every solo founder should be using:"
  }
}
```

### Create a listicle video

`POST /content/listicle`

Writes a numbered-list title plus 4 to 6 items from your brand and prompt, lays them over a stock UGC background clip that reveals one item per beat, and saves the result to your library (9:16). Costs no credits. Synchronous: the request returns when the deck is built, typically 5 to 20 seconds. Publish it with POST /posts using the deck from GET /content/{id}, or open it in the studio to edit first.

- Scopes: `generations:write`
- Credits: none
- Rate limit: 20 per 300s
- Supports `Idempotency-Key` header
- CLI: `viraloop content listicle --prompt <text>`
- MCP tool: `viraloop_create_listicle`

Body fields:

| Name | Type | Required | Description |
| --- | --- | --- | --- |
| `prompt` | string | no | What the list is about. Omit to let the model pick from your brand. |
| `mentionBusiness` | boolean | no | Make one of the items your brand. Default true. |
| `layout` | `list` \| `pyramid` \| `checklist` \| `countdown` | no | Display style: numbered list (default), tier pyramid, checklist, or countdown to #1. |
| `name` | string | no |  |
| `workspaceId` | string | no | Workspace to operate in. Defaults to the team's default workspace. |

Example:

```bash
curl -s -X POST "https://viraloop.io/api/v1/content/listicle" \
  -H "Authorization: Bearer $VIRALOOP_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"prompt":"habits of founders who ship every week"}'
```

Response:

```json
{
  "success": true,
  "data": {
    "id": "665f1b2a9c31a2b3c4d5e757",
    "format": "Listicle",
    "status": "ready",
    "title": "5 habits of founders who ship every week:"
  }
}
```

### Create an Ask Me Anything video

`POST /content/ask-me-anything`

Writes the question your audience actually asks plus the on-video answer from your brand and prompt, lays them over a stock UGC background clip as an IG-style question sticker, and saves the result to your library (9:16). Costs no credits. Synchronous: the request returns when the deck is built, typically 5 to 20 seconds. Publish it with POST /posts using the deck from GET /content/{id}, or open it in the studio to edit first.

- Scopes: `generations:write`
- Credits: none
- Rate limit: 20 per 300s
- Supports `Idempotency-Key` header
- CLI: `viraloop content ask-me-anything --prompt <text>`
- MCP tool: `viraloop_create_ask_me_anything`

Body fields:

| Name | Type | Required | Description |
| --- | --- | --- | --- |
| `prompt` | string | no | What the question should be about. Omit to let the model pick from your brand. |
| `mentionBusiness` | boolean | no | Name the brand in the answer. Default true. |
| `name` | string | no |  |
| `workspaceId` | string | no | Workspace to operate in. Defaults to the team's default workspace. |

Example:

```bash
curl -s -X POST "https://viraloop.io/api/v1/content/ask-me-anything" \
  -H "Authorization: Bearer $VIRALOOP_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"prompt":"how I stay consistent posting every day"}'
```

Response:

```json
{
  "success": true,
  "data": {
    "id": "665f1b2a9c31a2b3c4d5e758",
    "format": "Ask Me Anything",
    "status": "ready",
    "title": "how do you post every single day?"
  }
}
```

### Create a ranking (tier list) video

`POST /content/ranking`

Writes a tier-list title plus 5 to 7 tiered items (S/A/B/C) from your brand and prompt, finds a stock photo for each item, and saves an image tier-board video to your library (9:16, items reveal one per beat). Costs no credits. Synchronous: the request returns when the deck is built, typically 10 to 40 seconds. Publish it with POST /posts using the deck from GET /content/{id}, or open it in the studio to edit first.

- Scopes: `generations:write`
- Credits: none
- Rate limit: 20 per 300s
- Supports `Idempotency-Key` header
- CLI: `viraloop content ranking --prompt <text>`
- MCP tool: `viraloop_create_ranking`

Body fields:

| Name | Type | Required | Description |
| --- | --- | --- | --- |
| `prompt` | string | no | What the ranking is about. Omit to let the model pick from your brand. |
| `mentionBusiness` | boolean | no | Make one of the items your brand (tier S, revealed last). Default true. |
| `name` | string | no |  |
| `workspaceId` | string | no | Workspace to operate in. Defaults to the team's default workspace. |

Example:

```bash
curl -s -X POST "https://viraloop.io/api/v1/content/ranking" \
  -H "Authorization: Bearer $VIRALOOP_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"prompt":"ranking the ways to grow on tiktok"}'
```

Response:

```json
{
  "success": true,
  "data": {
    "id": "665f1b2a9c31a2b3c4d5e759",
    "format": "Ranking",
    "status": "ready",
    "title": "ranking the ways to grow on tiktok:"
  }
}
```

### Create a single fade-in video

`POST /content/fade-in-video`

Writes one bold caption from your brand and prompt, pairs it with a stock photo that fades in, and saves the result to your library (9:16). Pass imageUrl to use your own picture instead. Costs no credits. Synchronous: the request returns when the deck is built, typically 10 to 30 seconds.

- Scopes: `generations:write`
- Credits: none
- Rate limit: 20 per 300s
- Supports `Idempotency-Key` header
- CLI: `viraloop content fade-in-video --prompt <text>`
- MCP tool: `viraloop_create_fade_in_video`

Body fields:

| Name | Type | Required | Description |
| --- | --- | --- | --- |
| `prompt` | string | no | What the video is about. Omit to let the model pick from your brand. |
| `imageUrl` | string | no | Hosted image to fade in. A stock photo is found when omitted. |
| `mentionBusiness` | boolean | no | Name the brand in the copy. Default true. |
| `name` | string | no |  |
| `workspaceId` | string | no | Workspace to operate in. Defaults to the team's default workspace. |

Example:

```bash
curl -s -X POST "https://viraloop.io/api/v1/content/fade-in-video" \
  -H "Authorization: Bearer $VIRALOOP_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"prompt":"the real reason your reach dropped"}'
```

Response:

```json
{
  "success": true,
  "data": {
    "id": "665f1b2a9c31a2b3c4d5e756",
    "format": "Single Fade-in Video",
    "status": "ready",
    "title": "the real reason your reach dropped"
  }
}
```

### Stitch a hook clip onto your demo

`POST /content/hook-demo`

Puts a captioned hook clip in front of your product demo and saves the pair to your library (9:16). Both media URLs must be hosted; upload them with POST /assets first. The on-screen line is written from your brand unless you pass caption. Costs no credits. Synchronous: returns when the deck is built, typically 5 to 30 seconds.

- Scopes: `generations:write`
- Credits: none
- Rate limit: 20 per 300s
- Supports `Idempotency-Key` header
- CLI: `viraloop content hook-demo --hook <url> --demo <url>`
- MCP tool: `viraloop_create_hook_demo`

Body fields:

| Name | Type | Required | Description |
| --- | --- | --- | --- |
| `hookVideoUrl` | string | yes | Hosted attention-grabbing clip that plays first |
| `demoVideoUrl` | string | yes | Hosted product demo that plays after the hook |
| `caption` | string | no | The line burned over the hook. Written from your brand when omitted. |
| `prompt` | string | no | What the caption should be about, when caption is omitted |
| `hookThumb` | string | no | Poster image for the hook clip |
| `mentionBusiness` | boolean | no | Name the brand in the copy. Default true. |
| `name` | string | no |  |
| `workspaceId` | string | no | Workspace to operate in. Defaults to the team's default workspace. |

Example:

```bash
curl -s -X POST "https://viraloop.io/api/v1/content/hook-demo" \
  -H "Authorization: Bearer $VIRALOOP_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"hookVideoUrl":"https://cdn.viraloop.io/assets/hook.mp4","demoVideoUrl":"https://cdn.viraloop.io/assets/product-demo.mp4","prompt":"scheduling a week of posts in one sitting"}'
```

Response:

```json
{
  "success": true,
  "data": {
    "id": "665f1b2a9c31a2b3c4d5e757",
    "format": "Video Hook & Demo",
    "status": "ready",
    "title": "i scheduled a whole week in one sitting"
  }
}
```

### Get a studio render's status

`GET /content/{id}`

Polls a piece created by the /content operations or accepted from a generation. status is processing, ready or failed. Video formats carry videoUrl when ready; DECK formats (slideshow, wall of text, green screen, ...) have no flat media until rendered - publishing renders them automatically, or GET /content/{id}/download renders on demand and returns the files. A failed render has already refunded its credits.

- Scopes: `generations:read`
- Credits: undefined
- Terminal states: `ready`, `failed`
- CLI: `viraloop content get <id>`
- MCP tool: `viraloop_get_content`

Query parameters:

| Name | Type | Required | Description |
| --- | --- | --- | --- |
| `workspaceId` | string | no | Workspace to operate in. Defaults to the team's default workspace. |

Example:

```bash
curl -s "https://viraloop.io/api/v1/content/<id>" \
  -H "Authorization: Bearer $VIRALOOP_API_KEY"
```

Response:

```json
{
  "success": true,
  "data": {
    "id": "665f1b2a9c31a2b3c4d5e750",
    "format": "Character Swap",
    "status": "ready",
    "videoUrl": "https://cdn.viraloop.io/renders/character_swap.mp4",
    "thumbnailUrl": "https://cdn.viraloop.io/renders/character_swap.jpg",
    "durationSec": 8
  }
}
```

### Get a content piece's media files

`GET /content/{id}/download`

Returns the finished media files (mp4 for video formats, the slide images for carousels) for a content piece. For deck formats (slideshow, wall of text, green screen and the other studio decks) the FIRST call starts the server-side render and returns 409; poll this endpoint until it returns 200 with the files (typically under a minute). Generations must be accepted first (POST /generations/{id}/accept) to get a content id. Fetching files here marks the piece accepted for metered (enterprise) billing, exactly like publishing it does.

- Scopes: `generations:read`
- Credits: none
- CLI: `viraloop content download <id>`
- MCP tool: `viraloop_download_content`

Query parameters:

| Name | Type | Required | Description |
| --- | --- | --- | --- |
| `workspaceId` | string | no | Workspace to operate in. Defaults to the team's default workspace. |

Example:

```bash
curl -s "https://viraloop.io/api/v1/content/<id>/download" \
  -H "Authorization: Bearer $VIRALOOP_API_KEY"
```

Response:

```json
{
  "success": true,
  "data": {
    "id": "665f1b2a9c31a2b3c4d5e757",
    "format": "Slideshow",
    "files": [
      {
        "type": "image",
        "url": "https://cdn.viraloop.io/renders/slide-1.jpg"
      },
      {
        "type": "image",
        "url": "https://cdn.viraloop.io/renders/slide-2.jpg"
      }
    ]
  }
}
```
