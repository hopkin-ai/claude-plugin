# Start Tracking a Competitor's Accounts

## Overview

Add a competitor's public accounts so their posts are collected every day: look each one up, dry run the track, get the user's yes on the price, then track. Tracking charges credits, so never skip the confirmation.

| Step | Tool | Price |
|---|---|---|
| Check what is already tracked | `organic_social_list_tracked_accounts` | Free |
| Look up | `organic_social_lookup_account` | 5 credits when found, 15 on X; free when not found |
| Dry run | `organic_social_track_account` with `dry_run: true` | Free |
| Track | `organic_social_track_account` | 100 credits per account per billing period, 1,500 on X |

---

## Step 1: Get the Handles

The server does not search by brand name or find a brand's accounts. Ask the user for the handle or profile URL on each network, or have them confirm the ones they know. Then check `organic_social_list_tracked_accounts`: an account already tracked needs no lookup and no new charge.

---

## Step 2: Look Up Each Account

Tell the user the lookup costs 5 credits (15 on X) when the account is found.

```json
{
  "tool": "organic_social_lookup_account",
  "parameters": {
    "network": "instagram",
    "account": "https://www.instagram.com/acme/"
  }
}
```

- `found: true`: show the display name, bio, follower count, verified flag and a few of the latest posts, and ask the user to confirm it is the right account. Note `account.id`
- `found: false`: free. Check the spelling or ask for the profile URL. After 30 not-found lookups in an hour, lookups are refused
- `tracked: true`: the organization already tracks it; read its posts instead
- Post media in a lookup carries the network's short-lived `source_url`; counts and dates can be missing when the check was cut short

---

## Step 3: Dry Run

```json
{
  "tool": "organic_social_track_account",
  "parameters": {
    "network": "instagram",
    "account": "<account.id from the lookup>",
    "competitor": "Acme",
    "dry_run": true
  }
}
```

- `price.credits_per_period`: 100, or 1,500 on X
- `planned.verification`: `not_required` when the server already knows the account (true after a lookup); `required` means the real track will check the network first and can still fail as not found
- `competitor` (optional) links the account to a competitor already tracked in the Meta or LinkedIn tools, by its name or advertiser id. The link lets the user filter posts and summaries by the competitor's name
- A dry run tracks, charges and collects nothing

---

## Step 4: Confirm the Price, Then Track

Say, for example: "Tracking @acme on Instagram costs 100 credits now and again at each billing renewal while it stays tracked. Untracking later keeps the collected posts but does not refund the period. Go ahead?" For X, the price is 1,500 credits per period, and a trial organization cannot track X.

On a yes, repeat the call without `dry_run`. For several accounts, confirm the total first.

The answer reports:
- `change`: `tracked` (new), `unchanged` (already tracked), or a competitor link change
- `tracked`: the account id and handle
- `collection.status`: `in_progress` (first collection running), `ready` (already collected recently) or `queued` (the next daily collection takes it first)

---

## Step 5: Wait, Then Read

Posts appear in `organic_social_list_posts` as the first collection finishes. Check progress with `organic_social_list_tracked_accounts` (`collection.status`, `post_count`). Until then, tell the user the posts are still arriving rather than reporting none.

History starts now. Each collection reads the account's latest posts (50; 36 on Instagram; up to 50 long-form plus 50 Shorts on YouTube), so older posts from before tracking may not be available.

---

## Stopping

`organic_social_untrack_account` is free. Run it with `dry_run: true` first to show which account would be untracked. The organization can no longer read the account's posts; the collected posts are kept, and the current period is not refunded. Tracking it again in the same billing period is free and brings back the full history.
