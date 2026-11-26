# Check a Competitor's Last Week on One Network

## Overview

Answer "what did <competitor> post on <network> last week, and what did well?". Every call here is free.

---

## Step 1: Find the Account

```json
{
  "tool": "organic_social_list_tracked_accounts",
  "parameters": {
    "networks": ["tiktok"]
  }
}
```

- Find the account by handle, display name or linked `competitor`. Note its `id`
- If the competitor is linked, `competitor: "Acme"` on the next call covers every account linked to it
- Check `collection.status`, `stale` and `post_count`. A `pending` account with 0 posts has not been collected yet: say the posts are still arriving
- **Not in the list?** The organization does not track it. Say so, and offer the steps in `start-tracking.md`

---

## Step 2: List the Week's Posts

```json
{
  "tool": "organic_social_list_posts",
  "parameters": {
    "accounts": ["@acme"],
    "networks": ["tiktok"],
    "posted_since": "2026-11-19",
    "posted_until": "2026-11-25"
  }
}
```

- Dates are UTC days, both inclusive. "Last week" means the last seven full days unless the user says otherwise; state the dates you used
- The default sort is `newest` and the default page is 25 posts. Follow `nextCursor` with the same filters if there are more
- Add `sort: "views"` to see the best performers first. On a network where views are not shown, sort by `likes`

---

## Step 3: Read the Coverage Block

- `returned` above 0: posts were found
- `empty_reason: "no_match"`: the account was collected but posted nothing in the window. This is the only case where "no posts last week" is right
- `empty_reason: "collection_pending"`: not collected yet
- `empty_reason: "no_longer_seen"`: the network no longer shows the posts
- Pass on any `organic_account_stale` or `organic_collection_paused` warning: the counts may be older than they look

---

## Step 4: Open the Posts Worth Reading

For a video the user cares about, read what is said in it and how it grew:

```json
{
  "tool": "organic_social_get_post",
  "parameters": {
    "post": "https://www.tiktok.com/@acme/video/7300000000000000000"
  }
}
```

- `transcript` when `transcript_status` is `ready`
- `count_history`: one row per daily collection, oldest first
- `frames`: still-frame URLs, for describing the visuals

On YouTube, videos are currently stored with a thumbnail only, so there is no transcript.

---

## Step 5: Present

- One line of summary: how many posts, which formats, the best post and its views
- A table: date (UTC), format, text (first line), views, likes, comments, shares, link
- "Not shown" for null counts
- When the counts were read (`counts.as_of`), and any warning
