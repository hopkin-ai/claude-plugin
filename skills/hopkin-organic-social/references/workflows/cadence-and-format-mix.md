# Compare Posting Cadence and Format Mix

## Overview

Answer "how often does each competitor post, in what formats, and how do their posts typically do?" with one account summary. It costs 5 credits per call that covers at least one tracked account; say so before running it.

---

## Step 1: Scope

- Up to 20 tracked accounts per summary. With no `accounts` and no `networks`, every tracked account is in scope, and more than 20 is refused with the number
- Check what is tracked with `organic_social_list_tracked_accounts` (free) if the user has not named accounts
- `accounts` takes account ids, profile URLs, handles or a linked competitor's name
- Pick the period: `period_days` 1 to 365 (default 30), `until` the last day (default today, UTC). 30 or 90 days gives enough posts for a cadence

---

## Step 2: Run the Summary

```json
{
  "tool": "organic_social_get_account_summary",
  "parameters": {
    "accounts": ["@acme", "@globex"],
    "networks": ["instagram"],
    "period_days": 90
  }
}
```

The response is JSON with `period`, one block per account in `data` (ordered by network, then handle), and a `combined` block across all of them.

Per account block:

| Field | Meaning |
|---|---|
| `posts_in_period`, `posts_per_week` | Cadence |
| `format_mix` | Posts and share (0 to 1) per format, largest first |
| `median_counts` | Median views, likes, comments and shares; null counts are ignored |
| `top_posts` | The five best posts by views (by likes for posts with no view count) |
| `followers` | `count`, `change` over the period, `change_since`, `as_of` |
| `collection` | `status`, `last_collected_at`, `stale` |
| `partial_period` | The account was first collected inside the period, so its stored posts do not reach the period start |

---

## Step 3: Read Before Comparing

- **`partial_period: true`** understates cadence for that account. Say so, or shorten the period to the time since tracking began
- **Compare within a network.** Views and engagement differ by network; compare TikTok with TikTok
- **Followers `change` is null** until there are two daily snapshots in the period
- Check `coverage` and `warnings` for stale or paused accounts before calling a number current

---

## Step 4: Present

- A table: account (with network), posts per week, top format and its share, median views, median likes, follower change
- The `combined` block as the benchmark row
- Two or three observations, e.g. "Acme posts twice as often but at a third of Globex's median views; 70% of Acme's posts are short videos"
- The top post of each account with its link

To see the posts behind a number, use `organic_social_list_posts` with the same accounts and dates (free).
