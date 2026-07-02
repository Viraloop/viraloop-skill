---
name: viraloop
description: Create, schedule and track AI short-form social videos with Viraloop. Use when the user asks to generate AI videos or social content, schedule or publish posts to TikTok, Instagram Reels or YouTube Shorts, manage their Viraloop content queue, or check Viraloop credits and post performance.
---

# Viraloop

Viraloop generates ready-to-post short-form videos (wall-of-text, slideshow, green-screen formats) from a brand's context, renders them server-side, and publishes them to connected TikTok, Instagram and YouTube accounts on a schedule.

## When to use this skill

- "Generate some videos/content for my brand" -> generate suggestions, show them, publish the approved ones
- "Post this to TikTok/Reels/Shorts" or "schedule 5 posts for next week" -> create posts
- "Run a content campaign for two weeks" -> create a campaign, generate, review, launch
- "Make my AI influencer say ..." -> generate a talking-head video, then post it
- "What is queued/posted?" or "how did my posts do?" -> list posts, read analytics
- "How many Viraloop credits do I have?" -> credits

## Setup: install the CLI if missing

1. Check the CLI: `viraloop --version`
2. If not found, install it: `npm install -g viraloop` (or use `npx -y viraloop <command>` without installing)
3. Authenticate, one of:
   - `VIRALOOP_API_KEY` environment variable already set: nothing to do
   - Otherwise ask the user for an API key (created at https://viraloop.io/settings/developers) and run `viraloop login`
4. Verify: `viraloop whoami`

If npm is unavailable, fall back to the raw HTTP API (see the last section).

## Core concepts

- A team owns credits and API keys; a workspace is one brand with its own context, content angles and connected accounts. Commands default to the team's default workspace; pass `--workspace <id>` to target another.
- Content flows: generate suggestions -> review -> publish as posts -> Viraloop renders the video server-side -> the scheduler posts it -> per-platform results and analytics appear on the post.
- Always pass `--json` when running commands from a script or agent and parse the `success` field.

## Command reference

| Command | What it does | Endpoint |
| --- | --- | --- |
| `viraloop whoami` | Introspect the API key | GET /me |
| `viraloop credits` | Credit balance and ledger | GET /credits |
| `viraloop workspaces list` | List workspaces | GET /workspaces |
| `viraloop workspaces get <id>` | Get a workspace | GET /workspaces/{id} |
| `viraloop accounts list` | List connected social accounts | GET /accounts |
| `viraloop generate` | Generate AI post suggestions | POST /generations |
| `viraloop generations list` | List suggestions | GET /generations |
| `viraloop generations get <id>` | Get a suggestion | GET /generations/{id} |
| `viraloop posts create` | Create and schedule a post | POST /posts |
| `viraloop posts list` | List posts | GET /posts |
| `viraloop posts get <id>` | Get a post with per-platform results | GET /posts/{id} |
| `viraloop posts cancel <id>` | Cancel a scheduled post | DELETE /posts/{id} |
| `viraloop calendar` | Posting calendar | GET /calendar |
| `viraloop campaigns create` | Create a campaign | POST /campaigns |
| `viraloop campaigns list` | List campaigns | GET /campaigns |
| `viraloop campaigns get <id>` | Get a campaign | GET /campaigns/{id} |
| `viraloop campaigns update <id>` | Update a draft campaign | PATCH /campaigns/{id} |
| `viraloop campaigns generate <id>` | Generate the campaign's posts | POST /campaigns/{id}/generate |
| `viraloop campaigns launch <id>` | Launch a reviewed campaign | POST /campaigns/{id}/launch |
| `viraloop campaigns cancel <id>` | Cancel a campaign | POST /campaigns/{id}/cancel |
| `viraloop campaigns posts <id>` | List a campaign's generated posts | GET /campaigns/{id}/posts |
| `viraloop influencers list` | List AI influencers | GET /influencers |
| `viraloop influencers create` | Create an AI influencer | POST /influencers |
| `viraloop influencers get <id>` | Get an influencer | GET /influencers/{id} |
| `viraloop videos create --influencer <id>` | Generate a talking-head video | POST /influencers/{id}/videos |
| `viraloop videos list --influencer <id>` | List an influencer's videos | GET /influencers/{id}/videos |
| `viraloop videos get <id>` | Get a video's status | GET /videos/{id} |
| `viraloop assets list` | List media assets | GET /assets |

Full flag-level detail: references/api-reference.md (or https://viraloop.io/llms-full.txt).

## Common workflows

### Generate content and post it end to end

```bash
viraloop accounts list --json                    # find connected account ids
viraloop generate --count 3 --json               # returns suggestions with ids
viraloop generations get <suggestionId> --json   # inspect one (caption, hashtags, why)
viraloop posts create --suggestion <suggestionId> \
  --accounts tiktok:<accountId> --when asap --wait --json
```

`--wait` polls the post until a terminal state (posted, partial, completed, failed). Without `--wait`, poll `viraloop posts get <postId> --json` yourself; `renderStatus: "failed"` is also terminal.

### Schedule for a specific time

```bash
viraloop posts create --suggestion <id> --accounts tiktok:<accountId> \
  --at "2026-07-04T15:00:00Z" --json
```

Check the queue first with `viraloop calendar --from 2026-07-04 --to 2026-07-05 --json`. Do not use `--wait` for far-future posts; it waits until the post actually publishes.

### Check performance

```bash
viraloop posts list --status posted --json
viraloop posts get <postId> --json    # per-platform postUrl + views/likes/comments
```

## Error handling

Errors look like `{ "success": false, "error": { "type", "message" } }`. Exit codes: 2 auth, 3 insufficient credits, 4 rate limited, 1 other.

- `unauthorized`: re-run `viraloop login` (or fix `VIRALOOP_API_KEY`)
- `forbidden_scope`: the key lacks a scope; ask the user to create a key with the needed scopes at https://viraloop.io/settings/developers
- `rate_limited`: respect the Retry-After header / wait and retry
- `insufficient_credits`: tell the user to top up at https://viraloop.io/settings/billing
- `invalid_input`: fix the flags per references/api-reference.md
- `conflict` on posts create with a suggestion: that suggestion is already scheduled; generate or pick another

## Raw HTTP fallback (no CLI)

Base URL `https://viraloop.io/api/v1`, header `Authorization: Bearer $VIRALOOP_API_KEY`.

```bash
curl -s https://viraloop.io/api/v1/me -H "Authorization: Bearer $VIRALOOP_API_KEY"
```

Machine-readable surface: https://viraloop.io/openapi/v1.json and https://viraloop.io/llms-full.txt. A remote MCP server is also available at https://viraloop.io/api/v1/mcp (same Authorization header).
