---
name: hopkin-organic-social
description: Read and compare the public organic (unpaid) posts of social accounts your organization tracks on TikTok, YouTube, Facebook, Instagram and X, using the Hopkin Organic Social MCP. Covers listing posts by account, network, format, date and caption text, ranking posts by views, likes, comments, shares or growth, one post's daily count history, video transcript and frames, account summaries (posting cadence, format mix, median counts, top posts, follower growth), checking an account with a live lookup, tracking and untracking accounts (charged in credits, with a dry run first), and developer feedback. No ad account or connection is needed.
---

# Organic Social Skill

## Introduction

This skill enables Claude to read and analyze the public organic posts that Hopkin collects from social accounts your organization tracks, via the Hopkin Organic Social MCP. Use this skill when you need to:

- See what a competitor or any public account posted on TikTok, YouTube, Facebook, Instagram or X over a period
- Find the posts that got the most views, likes, comments or shares, or that grew fastest
- Find posts whose caption, title or description mentions a word or phrase
- Read what is said in a video (transcript) and look at still frames from it
- Compare posting cadence, format mix and median counts across accounts
- Follow follower growth of tracked accounts
- Check a public account before tracking it, then start or stop tracking it

**No ad account or connection is needed.** The tools read Hopkin's own store of collected posts. Tracking belongs to the organization: any member can track or untrack an account, and every member reads the same tracked accounts.

**Some tools write and some charge credits.** `organic_social_track_account` and `organic_social_untrack_account` change which accounts your organization tracks. `organic_social_lookup_account` runs a live check of a public profile and is charged. `organic_social_developer_feedback` sends feedback to Hopkin. `organic_social_get_billing_status` can create a Stripe checkout link when the organization owner asks to pay. See [Prices and the Dry-Run Habit](#prices-and-the-dry-run-habit).

## Prerequisites

### Check for MCP Availability

1. Check for `organic_social_` prefixed tools in your available tools (e.g. `organic_social_list_tracked_accounts`, `organic_social_list_posts`)
2. If found, the Hopkin Organic Social MCP is properly configured

### If Hopkin Organic Social MCP Is Not Installed

If no `organic_social_` prefixed tools are available:

1. **Inform the user** that the Hopkin Organic Social MCP is required to use this skill
2. **Direct them to sign up** at https://app.hopkin.ai/?utm_source=claude-plugin&utm_medium=skill&utm_content=organic-social
3. **Provide setup instructions:**

> The Hopkin Organic Social MCP is not connected. To use this skill:
>
> 1. Sign up at **https://app.hopkin.ai/?utm_source=claude-plugin&utm_medium=skill&utm_content=organic-social**. Your Hopkin user must belong to an organization (create one or accept an invitation there)
> 2. Make sure the **Hopkin** plugin is installed and enabled. It sets up the hosted `hopkin-organic-social` MCP server (`https://organic.mcp.hopkin.ai/mcp`) for you, so there is nothing to configure by hand
> 3. When Claude asks you to sign in to `hopkin-organic-social`, finish the Hopkin login in your browser. Sign-in uses OAuth, so there is nothing to copy or paste

4. **Pause execution** until the user confirms MCP installation is complete
5. **Re-verify** MCP availability after user confirmation

### Organization

Every tool needs the signed-in user to belong to a Hopkin organization. Without one, tools fail with HTTP 403 and a message to create an organization or accept an invitation at hopkin.ai. There is no platform account to connect and no `connection_id` on any tool.

## Available MCP Tools

### Reading Posts (free)
- `organic_social_list_posts`: posts from tracked accounts. With no filters, the newest 25 posts across every tracked account (there is no default date window). Filters: `accounts` (1 to 25: an account id, profile URL, handle with or without @, or a competitor name linked to tracked accounts), `networks`, `formats` (`short_video`, `long_video`, `image`, `carousel`, `text`, `link`), `posted_since` / `posted_until` (YYYY-MM-DD, inclusive, UTC), `query` (literal, case-insensitive match on the post's own text). `sort`: `newest` (default), `views`, `likes`, `comments`, `shares` or `growth` (with `growth_window_days`, 1 to 90, default 7). `limit` 1 to 100, default 25
- `organic_social_get_post`: one post in full. Provide the post's public URL or the post id from `organic_social_list_posts`. Adds `count_history` (every daily snapshot, oldest first), `transcript` and `transcript_status`, and `frames` (still-frame URLs per mirrored video, capped)

### Accounts
- `organic_social_list_tracked_accounts` (free): every account the organization tracks, with its account id, network, handle, profile URL, followers, linked competitor, when it was tracked, how many posts are stored, the collection state and its price per billing period. Filter by `networks` or `competitor`. Works in every billing state
- `organic_social_get_account_summary` (5 credits): per account and combined, over a period: posts in the period, posts per week, format mix, median views, likes, comments and shares, the five top posts, and follower count and change. `accounts` (1 to 20) or `networks` scope it; `period_days` 1 to 365 (default 30); `until` is the last day (default today, UTC)

### Tracking
- `organic_social_lookup_account` (5 credits when found, 15 on X; free when not found): a live check of one account on one network. Provide `network` and an exact handle or profile URL. Returns the profile (display name, bio, followers, verified) and about the latest 10 posts, whether the organization already tracks it, and the next step. It does not track anything
- `organic_social_track_account` (100 credits per account per billing period, 1,500 on X): start tracking an account. Provide `network` and `account` (a handle, profile URL, the network's own id, or the account id from a lookup). Optional `competitor` links it to a competitor you already track in the Meta or LinkedIn tools; `clear_competitor: true` removes the link. `dry_run: true` previews the plan and price
- `organic_social_untrack_account` (free): stop tracking one account. Provide `account` (handle, profile URL, network id or account id) and `network` when the same handle is tracked on several networks. `dry_run: true` shows which account would be untracked. Works in every billing state

### Housekeeping
- `organic_social_ping`: health check
- `organic_social_get_billing_status`: call after any tool fails with a billing error (HTTP 402). It says why, gives the organization owner a link to pay, and says when to retry
- `organic_social_developer_feedback`: report a missing tool, a needed improvement, a bug or a workflow gap. See [Feedback to Hopkin](#feedback-to-hopkin)

## Core Capabilities

### Pick the Right Tool

| The user asks for | Use |
|---|---|
| What is tracked, whether collection is healthy, what it costs | `organic_social_list_tracked_accounts` |
| The posts themselves: recent, in a date range, by format, by keyword | `organic_social_list_posts` |
| Top posts by views, likes, comments or shares | `organic_social_list_posts` with `sort` |
| What is taking off right now | `organic_social_list_posts` with `sort: "growth"` |
| What is said in a video, how a post grew day by day, frames | `organic_social_get_post` |
| How often an account posts, its format mix, typical counts, follower growth | `organic_social_get_account_summary` |
| Whether a handle is the right account, before tracking | `organic_social_lookup_account` |
| Start or stop collecting an account | `organic_social_track_account` / `organic_social_untrack_account` |

Start with `organic_social_list_tracked_accounts` when you do not know what the organization tracks. It is free.

### Track, Wait, Read

Reads only cover accounts the organization tracks. To add one:

1. **Look up** the account with `organic_social_lookup_account` (network plus exact handle or profile URL). Show the user the display name, verified flag, follower count and a few recent posts, and confirm it is the right account. A lookup is not a brand-name search: if the user gives a brand name, ask for the handle or profile URL
2. **Dry run** `organic_social_track_account` with `dry_run: true`. It shows the price per billing period and whether the real call will need a live check (`planned.verification`). It does not check the network, track or charge
3. **Ask the user** to confirm the price, then **track** without `dry_run`. Provide the `account.id` from the lookup as `account`: the server already knows that account and skips a second live check
4. **Wait** for the first collection. The track answer reports `collection.status`:
   - `in_progress`: the first collection started. Posts appear in `organic_social_list_posts` as it finishes
   - `ready`: the account was collected recently, its stored posts are readable now
   - `queued`: the first collection could not start now; the next daily collection takes this account first
5. **Read** with `organic_social_list_posts`, `organic_social_get_post` and `organic_social_get_account_summary`. Check progress with `organic_social_list_tracked_accounts`: `post_count` and `collection.status` (`pending` until the first collection finishes)

After the first collection, every tracked account is collected once a day. Each collection reads the account's latest posts: 50 (36 on Instagram; up to 50 long-form videos plus 50 Shorts on YouTube) and re-reads their view, like, comment and share counts. History builds up from when the account was tracked; there is no full backfill of older posts. Tell the user this when they ask about a period before tracking started.

### Prices and the Dry-Run Habit

1 credit = $0.01.

| Tool | Price |
|---|---|
| `organic_social_track_account` | 100 credits per account per billing period, 1,500 on X. Charged when the account is first tracked and again at each renewal while it stays tracked. Tracking the same account again in the same billing period is free, including after an untrack. Changing only the competitor link of a tracked account is free |
| `organic_social_lookup_account` | 5 credits when the account is found, 15 on X. Free when it is not found, or when the network cannot be checked. A lookup of an account the organization already tracks is still charged |
| `organic_social_get_account_summary` | 5 credits per call that covers at least one tracked account, even if those accounts posted nothing in the period. Free when no tracked account is in scope |
| Every other tool | Free |

Rules:

- **Before any track, run it with `dry_run: true`, state the price, and get the user's yes.** For X, say 1,500 credits per billing period plainly
- **Mention the cost** when you run a lookup or a summary for the user ("this lookup costs 5 credits")
- **Before a lookup, check `organic_social_list_tracked_accounts`** (free). If the account is already tracked, read its stored posts instead of paying for a lookup
- **Untracking does not refund** the current billing period, and the collected posts are kept. Re-tracking in the same period is free and brings back the full history
- **A trial organization cannot track X accounts** (the charge is larger than a trial's whole credit grant). The call fails with a billing error; call `organic_social_get_billing_status` for the owner's checkout link

### When an Account Is Not Tracked

Any read that names an account the organization does not track fails as not found (HTTP 404), and the message names `organic_social_track_account` and `organic_social_lookup_account`. `organic_social_get_post` answers the same way for a post of an untracked account. Then:

1. Say plainly that the organization does not track this account, so there are no stored posts to read
2. Do not suggest that Hopkin holds the data anyway
3. Offer the path: look it up (5 credits, 15 on X), dry run, then track (100 credits per billing period, 1,500 on X), and explain that posts arrive after the first collection

### Reading `coverage`

`organic_social_list_posts`, `organic_social_get_post`, `organic_social_get_account_summary` and `organic_social_list_tracked_accounts` return a `coverage` array with one entry per network in scope:

- `tracked_accounts`: accounts the organization tracks on that network, within this call's scope
- `returned`: rows of that network in this response. Posts on this page for `list_posts`; accounts for the summary and the tracked-accounts list; 1 for `get_post`
- `empty_reason` (only when `returned` is 0, and only on the first page):
  - `not_tracked`: no tracked account on this network in scope
  - `collection_pending`: the accounts are tracked but not collected yet
  - `no_longer_seen`: the matching posts, or the accounts, are no longer seen on the network
  - `no_match`: collected, but nothing matched the filters
- `collection_paused`: collection is paused for this network
- `stale_accounts`: accounts in scope not collected for 48 hours

Use `empty_reason` to explain an empty answer. "Nothing posted" is only right for `no_match` with a date window the account has been collected over.

### Reading `warnings`

- `organic_collection_paused`: collection is paused for a network where the organization tracks at least one account. Stored posts are readable but not being refreshed. A paused network also refuses new lookups and new tracks (HTTP 503, nothing charged)
- `organic_account_stale`: an account has not been collected for 48 hours, with the last collection date and the failure reason when there is one

Pass these on to the user when they affect the answer, for example "counts for @brand on Instagram are from 3 days ago".

### Counts, Status and Media

- **A count the network does not show is `null`, never zero.** Say "not shown" rather than 0. Medians in the summary ignore missing counts
- `counts.as_of` is when the counts were read. `count_changes` is the change since the previous daily snapshot (null until a second snapshot exists)
- Post `status`: `active`; `out_of_window` (older than the latest posts each collection reads, so its counts are frozen at `counts_frozen_at`); `no_longer_seen` (vanished from the network while still among the latest posts, since `not_seen_since`)
- `media[].url` is Hopkin's permanent copy, null until mirrored. `media[].status` is `pending`, `mirrored`, `skipped_oversize`, `failed` or `unavailable`
- **Transcripts and frames exist only for mirrored video.** `transcript_status`: `ready`, `pending`, `silent` (no speech), `failed`, or `not_applicable` (no video, or no video file kept)
- `query` matches only the post's own text (caption, title, description), not what is said in a video. To read speech, open the post with `organic_social_get_post`

### Per-Network Notes

- **YouTube**: long-form videos and Shorts are collected separately (up to 50 of each). Both are currently stored with a thumbnail only, with no video file, so they have no transcript or frames
- **TikTok**: videos are mirrored, so transcripts and frames become available once processing finishes
- **Facebook**: Reels are `short_video` and their file is mirrored. Other Facebook videos are `long_video` with a thumbnail only. Facebook `views` is the view count shown on the page
- **Instagram**: each collection reads the latest 36 posts, not 50
- **X**: only what the account itself published is stored. Plain reposts and replies to other accounts are dropped; the account's own thread replies are kept. A reply-heavy account can show fewer posts than expected, in a lookup too. X tracking costs 1,500 credits per billing period and a lookup 15 credits; a trial cannot track X

### Pagination

`organic_social_list_posts` and `organic_social_list_tracked_accounts` return a top-level `nextCursor` when more rows remain. Repeat the call with **the same filters** plus `cursor`; only `limit` may change. `count` is the number of rows on this page.

### What This Server Cannot Do

- **No private accounts and no logged-in content.** Only public posts are collected
- **No comment text.** Comment counts only
- **No ads.** Paid ads of competitors are in the Meta and LinkedIn competitor tools (`meta_ads_list_competitor_ads`, `linkedin_ads_list_competitor_ads` and their search tools). A competitor tracked there can be linked to its social accounts with `competitor` on `organic_social_track_account`
- **No search by brand name and no discovery of a brand's accounts.** The user names the account; a lookup checks it
- **No refresh on demand.** Accounts are collected once a day; there is no refresh option

When a user asks for something outside this list, say so plainly and offer to report it with `organic_social_developer_feedback` (call it only if they say yes).

### Feedback to Hopkin

`organic_social_developer_feedback` sends a short description of a missing capability or fault to Hopkin. Call it only when the user asks you to report something, or after you have asked the user whether to send it and they said yes. Never call it automatically or on your own initiative, and keep the description to the gap or fault itself: do not include the conversation, its history or a summary of it.

## Report Workflows

### Check a Competitor's Last Week on One Network

**Primary tools:** `organic_social_list_tracked_accounts` → `organic_social_list_posts` with `accounts`, `networks` and a date window → `organic_social_get_post` for videos worth reading.

**See detailed workflow:** **references/workflows/competitor-last-week.md**

---

### Find What Is Taking Off

Rank posts across tracked accounts by how much their views grew.

**Primary tool:** `organic_social_list_posts` with `sort: "growth"` and `growth_window_days`.

**See detailed workflow:** **references/workflows/whats-taking-off.md**

---

### Compare Posting Cadence and Format Mix

**Primary tool:** `organic_social_get_account_summary` (5 credits).

**See detailed workflow:** **references/workflows/cadence-and-format-mix.md**

---

### Start Tracking a Competitor's Accounts

**Primary tools:** `organic_social_lookup_account` → `organic_social_track_account` with `dry_run: true` → `organic_social_track_account`.

**See detailed workflow:** **references/workflows/start-tracking.md**

---

## Workflow Process

1. **Scope**: find the accounts with `organic_social_list_tracked_accounts`. If an account is not tracked, follow [When an Account Is Not Tracked](#when-an-account-is-not-tracked)
2. **Period**: ask or default sensibly (last 7 days for "recent", 30 days for a summary). Dates are UTC days
3. **Tool**: match the question to [Pick the Right Tool](#pick-the-right-tool)
4. **Pages**: follow `nextCursor` when the answer needs every post
5. **Output**: lead with the answer, then a table. Link each post by its `post_url`. State the networks, the period and when the data was collected (`counts.as_of`, `synced_at`), and pass on any warnings

## Best Practices

### Interpretation

- Compare counts within one network. Each network counts views its own way, and some do not show every count
- Prefer medians (from the summary) over averages: one viral post distorts an average
- Newer posts have had less time to gather views. Use `sort: "growth"` or compare posts of similar age
- A small number of posts is not a trend. Say how many posts a claim rests on
- `partial_period: true` on a summary block means the account was first collected inside the period, so the stored posts do not reach the period start. Do not read it as a drop in posting

### Report Formatting

- Handles with their network, e.g. "@brand on TikTok"
- Counts as whole numbers with thousands separators; "not shown" for null
- Each post with its date (UTC), format, counts and link
- For a summary, a table of accounts with posts per week, the top format and median views

### Error Handling Patterns

1. **Not found (HTTP 404)**: the account or post is not tracked by the organization, or no tracked competitor has that name. See [When an Account Is Not Tracked](#when-an-account-is-not-tracked)
2. **No organization (HTTP 403)**: the user must create or join a Hopkin organization at hopkin.ai
3. **Input error (HTTP 422)**: the message names the fix, e.g. more than 20 accounts in a summary scope, `posted_since` after `posted_until`, a reference that matches several tracked accounts
4. **Billing error (HTTP 402)**: call `organic_social_get_billing_status`
5. **Network paused or check unavailable (HTTP 503)**: nothing was charged. A paused network cannot be looked up or newly tracked; stored posts are still readable. "Could not check" means retry in a minute
6. **Rate limit (HTTP 429)**: 30 not-found lookups in an hour. Check the handles before trying again

---

## Troubleshooting

For detailed troubleshooting guidance, see **references/troubleshooting.md**.

Common quick fixes:

- **"MCP server not found"**: check that the Hopkin plugin is enabled and its `hopkin-organic-social` server is signed in
- **"No tracked account matches"**: the account is not tracked; look it up, dry run, then track
- **A newly tracked account has no posts**: the first collection is still running or queued; check `organic_social_list_tracked_accounts`
- **No transcript**: the video was not mirrored (YouTube, Facebook long-form) or is still processing
- **Fewer X posts than expected**: replies to other accounts and plain reposts are not stored

---

## Additional Resources

- **references/troubleshooting.md**: error solutions and debugging steps
- **references/workflows/competitor-last-week.md**: one competitor's posts over the last week on one network
- **references/workflows/whats-taking-off.md**: fastest-growing posts across tracked accounts
- **references/workflows/cadence-and-format-mix.md**: posting cadence, format mix and medians with the account summary
- **references/workflows/start-tracking.md**: lookup, dry run, then track
- Hopkin dashboard: https://app.hopkin.ai/?utm_source=claude-plugin&utm_medium=skill&utm_content=organic-social

---

**Requires:** Hopkin Organic Social MCP (https://app.hopkin.ai/?utm_source=claude-plugin&utm_medium=skill&utm_content=organic-social)
