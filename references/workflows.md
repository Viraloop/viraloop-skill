# Viraloop workflows for agents

Step-by-step recipes using the CLI (always with `--json`). The raw API equivalents are in api-reference.md.

## 1. First contact with a new account

```bash
viraloop whoami --json          # team, credits, plan, granted scopes
viraloop workspaces list --json # brands; note the default workspace
viraloop accounts list --json   # connected socials; empty means the user must connect accounts in the web app first
```

If `accounts list` is empty, stop and tell the user to connect an account at https://viraloop.io/accounts. Posting requires at least one connected account.

## 2. Generate, review, publish

```bash
viraloop generate --count 5 --json
```

Each suggestion has `id`, `caption` (on-screen text), `postCaption` (post body), `hashtags`, `why` (rationale). Present these to the user or pick the strongest yourself.

```bash
viraloop posts create --suggestion <id> --accounts tiktok:<accountId>,youtube:<accountId2> --when asap --wait --json
```

Multiple platforms: repeat `platform:id` pairs comma-separated in `--accounts`. The response is a post with `statusUrl`; `--wait` polls it to a terminal state and prints the final post including per-platform `postUrl`s.

## 3. Fill a week's calendar

```bash
viraloop calendar --from 2026-07-06 --to 2026-07-13 --json   # see what is queued
viraloop generate --count 7 --json
# then one posts create per suggestion with --at spaced across the week
viraloop posts create --suggestion <id> --accounts tiktok:<acc> --at "2026-07-06T15:00:00Z" --json
```

Times are ISO 8601; pass `--timezone` (IANA name) if the user thinks in local time.

## 4. Publish media you already have

```bash
viraloop posts create --video-url https://example.com/final.mp4 \
  --caption "Launch day" --accounts tiktok:<acc> --when asap --json
```

Also `--image-url` (repeatable) for image/slideshow posts. Media URLs must be publicly reachable.

## 5. Run a campaign (batch generate + auto-schedule)

```bash
viraloop campaigns create --name "Launch week" --posts-per-day 2 --length-days 7 \
  --accounts tiktok:<accountId> --json
viraloop campaigns generate <campaignId> --json      # 202; generation is async
viraloop campaigns get <campaignId> --json           # poll until status "review"
viraloop campaigns posts <campaignId> --json         # inspect the generated posts
viraloop campaigns launch <campaignId> --json        # schedules everything at its slot
viraloop campaigns cancel <campaignId> --json        # stops unsent posts anytime
```

Campaign generation typically takes a few minutes for a week of posts. After launch the campaign is hands-off: posts render and publish at their slots.

## 6. Talking-head influencer videos

```bash
viraloop influencers list --json
viraloop videos create --influencer <id> --script "Three things I wish I knew..." --wait --json
# costs 20 credits; --wait polls until completed (2 to 10 minutes)
viraloop posts create --video-url <videoUrl from the result> \
  --caption "..." --accounts tiktok:<acc> --when asap --json
```

No influencer yet? `viraloop influencers create --name Maya --image-url https://... --json` (the image must be a publicly reachable photo of the persona).

## 7. Monitor and clean up

```bash
viraloop posts list --status scheduled --json   # queued
viraloop posts list --status failed --json      # needs attention
viraloop posts cancel <postId> --json           # only works while still scheduled
viraloop posts get <postId> --json              # analytics per platform after posting
```
