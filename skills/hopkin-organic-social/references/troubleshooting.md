# Organic Social Skill - Troubleshooting Guide

Solutions to common issues when using the Organic Social skill with the Hopkin Organic Social MCP.

## Table of Contents

1. [MCP Server Issues](#mcp-server-issues)
2. [Organization and Sign-In](#organization-and-sign-in)
3. [Not Found](#not-found)
4. [Empty or Thin Results](#empty-or-thin-results)
5. [Lookups and Tracking](#lookups-and-tracking)
6. [Transcripts, Frames and Media](#transcripts-frames-and-media)
7. [Billing](#billing)
8. [API Error Types](#api-error-types)

---

## MCP Server Issues

### MCP Server Not Found

**Symptoms:**
- No `organic_social_` prefixed tools available

**Solutions:**

#### Solution 1: Install the Hopkin Plugin and Sign In

1. Sign up at **https://app.hopkin.ai/?utm_source=claude-plugin&utm_medium=skill&utm_content=organic-social**
2. Install and enable the **Hopkin** plugin. It registers the hosted `hopkin-organic-social` MCP server (`https://organic.mcp.hopkin.ai/mcp`) for you
3. When Claude asks you to sign in to `hopkin-organic-social`, finish the Hopkin login in your browser (OAuth, nothing to copy or paste)

#### Solution 2: Check the Plugin and Server

If the plugin is installed but tools aren't appearing:
1. Confirm the Hopkin plugin is enabled (`/plugin` in Claude Code, or **Customize → Plugins** in Claude)
2. Confirm the `hopkin-organic-social` server is connected (`/mcp` in Claude Code). If it says it needs authentication, sign in again
3. Update the plugin: an older install has no `hopkin-organic-social` server

#### Solution 3: Restart Claude

After installing or updating the plugin, quit and restart Claude, then check that `organic_social_` tools appear.

---

## Organization and Sign-In

### "This account does not belong to a Hopkin organization"

**Symptoms:** every tool fails with HTTP 403 and this message.

**Solution:** tracking belongs to an organization, so every tool needs one. The user creates an organization or accepts an invitation at https://app.hopkin.ai/?utm_source=claude-plugin&utm_medium=skill&utm_content=organic-social, then retries.

### No Account to Connect

There is no social or ad account to connect for this server, and no `connection_id`. Never ask the user to connect TikTok, YouTube, Facebook, Instagram or X. If another Hopkin skill asks for a connection, that is for its own platform.

---

## Not Found

### "No tracked account matches ..."

**Symptoms:** HTTP 404 from `organic_social_list_posts` or `organic_social_get_account_summary` naming one or more references, or from `organic_social_untrack_account`.

**Causes and solutions:**
1. **The account is not tracked by the organization.** Say so plainly, then offer: lookup, dry run, track (see `start-tracking.md`)
2. **The reference does not match.** References are matched against tracked accounts by account id, profile URL, handle (with or without @, any case), the network's own id, or the name of a linked competitor. Call `organic_social_list_tracked_accounts` and use the `id` it returns
3. **A `networks` filter excludes the account.** A reference is only matched within the networks in scope

### "No post matches that reference among the accounts your organization tracks"

`organic_social_get_post` gives this answer both for a post Hopkin does not hold and for a post of an account the organization does not track. Do not tell the user the post exists. Check:
- The URL is the post's public URL on its network, or the `id` from `organic_social_list_posts`
- The post's account is tracked (`organic_social_list_tracked_accounts`)
- The post is among what the account's collections have read. Collections read the latest posts only, from the day the account was first tracked; older posts were never collected

### "No tracked competitor is named ..."

The `competitor` filter or link takes the name or advertiser id of a competitor tracked in `meta_ads_list_tracked_competitors` or `linkedin_ads_list_tracked_competitors`. Check the name there, or leave the filter out.

---

## Empty or Thin Results

Read the `coverage` block before concluding anything. `empty_reason` appears only when a network returned nothing, and only on the first page.

| `empty_reason` | Meaning | What to tell the user |
|---|---|---|
| `not_tracked` | No tracked account on this network in scope | Offer lookup, dry run, track |
| `collection_pending` | Tracked, not collected yet | Posts arrive after the first collection |
| `no_longer_seen` | The posts or accounts are no longer seen on the network | The network no longer shows these posts or accounts; say so rather than "no posts" |
| `no_match` | Collected, nothing matched the filters | Widen the dates, formats or query |

### A Newly Tracked Account Has No Posts

The first collection runs after tracking. Check `organic_social_list_tracked_accounts`:
- `collection.status` `in_progress`: it is running now
- `collection.status` `pending` with `post_count` 0: not collected yet. If tracking answered `queued`, the next daily collection takes this account first
- `collection.status` `failing`: see `last_failure_reason`

Do not report "no posts" for an account in this state; say the posts are still arriving.

### Nothing Before a Certain Date

History starts when the account was first tracked. Each collection reads the latest posts (50; 36 on Instagram; up to 50 long-form plus 50 Shorts on YouTube), so an account that posts often may have older posts that were never collected. In a summary, `partial_period: true` marks an account whose stored posts do not reach the period start.

### Counts Look Old

- `organic_account_stale` warning, or `stale_accounts` above 0 in `coverage`: the account has not been collected for 48 hours. The warning gives the last collection date and the failure reason when there is one
- `organic_collection_paused` warning, or `collection_paused: true`: collection is paused for that network. Stored posts are readable but not refreshed
- A post with `status: "out_of_window"` is older than what collections read now, so its counts are frozen at `counts_frozen_at`

There is no refresh option. State `counts.as_of` with the numbers.

### Counts Are Null

A count the network does not show is `null`, never zero. Say "not shown". The summary's medians ignore nulls.

### Fewer X Posts Than Expected

Only what the account itself published is stored. Plain reposts and replies to other accounts are dropped; the account's own thread replies are kept. A reply-heavy account stores fewer posts per collection, and a lookup shows fewer posts too.

### "Query" Finds Nothing

`query` is a literal, case-insensitive match on the post's own text (caption, title, description). It does not match words spoken in a video, hashtags written differently, or near spellings. Try a shorter phrase, or read transcripts with `organic_social_get_post`.

---

## Lookups and Tracking

### Lookup Says Not Found

The lookup checks an exact handle or profile URL on one network. It is not a brand-name search, and a network's own numeric id only matches accounts Hopkin already stores. Check the spelling, try the profile URL, or ask the user for the handle. A not-found lookup is free.

### "30 not-found lookups in the last hour"

HTTP 429. Not-found live checks are limited to 30 per organization per hour, and a track of an unknown account counts as a live check too. Stop guessing handles; ask the user for the profile URL.

### "Collection is paused for <network>"

HTTP 503. That network cannot be looked up or newly tracked right now. Nothing was charged. Stored posts can still be read, and accounts already tracked can still have their competitor link changed.

### "Could not check <network> right now"

HTTP 503. The live check failed on the network's side or timed out. Nothing was charged. Retry in a minute.

### The Dry Run Said `verification: "required"`

A dry run never checks the network. `required` means the real track will run a live check first, and it can still fail as not found (nothing charged). To avoid a second check, look the account up first and provide its `account.id` to the track call.

### The Track Answer Says `change: "unchanged"`

The organization already tracks this account. Nothing new was charged; only a competitor link change would have applied. `competitor_link_updated` and `competitor_link_cleared` mean only the link changed.

### "Pass competitor or clear_competitor: true, not both"

HTTP 422. Provide one of them.

### Untrack Matched Several Accounts

HTTP 422 listing the matches. The same handle is tracked on several networks. Provide `network`, or the account `id` from `organic_social_list_tracked_accounts`. Untrack never removes more than one account per call.

### "N tracked accounts are in scope and a summary covers at most 20"

HTTP 422. Narrow `organic_social_get_account_summary` with `accounts` or `networks`, and run several summaries if needed (each costs 5 credits).

---

## Transcripts, Frames and Media

### No Transcript

| `transcript_status` | Meaning |
|---|---|
| `ready` | The transcript is in `transcript` |
| `pending` | The video is still being mirrored or processed. Try later |
| `silent` | The video has no speech |
| `failed` | Extraction failed |
| `not_applicable` | No video, or no video file kept |

YouTube videos (long-form and Shorts) and Facebook long-form videos are currently stored with a thumbnail only, so they have no transcript or frames.

### Media URL Is Null

`media[].url` is Hopkin's permanent copy and is null until mirrored. `media[].status` says why: `pending` (not yet), `skipped_oversize` (the file was too large to keep), `failed` or `unavailable`. In a lookup, `source_url` is the network's own short-lived URL, usable for a quick look before the copy lands.

---

## Billing

### HTTP 402 on Any Tool

Call `organic_social_get_billing_status`. It says why, gives the organization owner a link to pay, and says when to retry. `organic_social_list_tracked_accounts` and `organic_social_untrack_account` work in every billing state, so a blocked organization can still see and stop its tracked accounts.

### A Trial Cannot Track X

Tracking an X account costs 1,500 credits per billing period, more than a trial's whole credit grant. A trial organization's call to track an X account, even with `dry_run: true`, fails with a billing error. The owner chooses a plan through `organic_social_get_billing_status`.

### Tracking Keeps Costing Credits

Each tracked account is charged again at each billing renewal while it stays tracked. Show the user `credits_per_period` from `organic_social_list_tracked_accounts`, and untrack accounts they no longer need (no refund for the current period).

---

## API Error Types

| Status | Meaning | Action |
|---|---|---|
| HTTP 402 | Billing required | `organic_social_get_billing_status` |
| HTTP 403 | No Hopkin organization | Create or join one at hopkin.ai |
| HTTP 404 | Account, post or competitor not tracked or not found | Check `organic_social_list_tracked_accounts`; offer lookup, dry run, track |
| HTTP 422 | Invalid input | Read the message; it names the fix |
| HTTP 429 | 30 not-found lookups in an hour | Stop and confirm handles with the user |
| HTTP 503 | Network paused, live check unavailable, or the store was slow | Nothing charged. Retry later; a store timeout says to retry in 5 seconds |

If a tool returns data that looks wrong, offer to report it with `organic_social_developer_feedback` (`feedback_type: "bug"`), and send it only if the user says yes.

---

## Quick Checklist

- [ ] Hopkin plugin is enabled and its `hopkin-organic-social` server is signed in
- [ ] The user belongs to a Hopkin organization
- [ ] The account is tracked (`organic_social_list_tracked_accounts`)
- [ ] The first collection has finished (`post_count` above 0)
- [ ] `coverage` and `warnings` read before calling a result empty or current
- [ ] Price stated and confirmed before any track; dry run first
