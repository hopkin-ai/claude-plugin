# Find What Is Taking Off

## Overview

Answer "which posts are taking off right now across the accounts we track?" by ranking posts on how much their views grew over a recent window. Free.

---

## Step 1: Rank by Growth

```json
{
  "tool": "organic_social_list_posts",
  "parameters": {
    "sort": "growth",
    "growth_window_days": 7
  }
}
```

- Scope with `networks` or `accounts` when the user names them; omit both for every tracked account
- `growth_window_days` is 1 to 90, default 7. Use a shorter window (1 to 3) for "today" or "this weekend"
- `posted_since` narrows to recent posts if the user wants only new posts

Each post carries a `growth` block:

| Field | Meaning |
|---|---|
| `measure` | `views`, or `likes` where the network shows no view count for the post |
| `gain` | How much the measure grew over the window |
| `window_days` | The window used |
| `baseline_date` | The daily snapshot the gain is measured from |
| `partial_window` | `true` when no snapshot was old enough, so the gain is measured from the earliest one |

---

## Step 2: Read the Gains Carefully

- **A post younger than the window has a baseline of 0**, so its gain is its whole count. A new post can top the list on its first numbers alone. Say "new this week" for these rather than "grew fastest"
- **`partial_window: true`** means the gain covers fewer days than asked. This is common for accounts tracked recently, because snapshots start at the first collection
- **Gains in views and in likes are not comparable.** Group by `measure`, or by network, before ranking across networks
- A post with neither count has no gain and sorts last

---

## Step 3: Confirm the Winners

For the top few posts, open them with `organic_social_get_post`:
- `count_history` shows whether the growth is steady or one jump
- `transcript` and `frames` show what the post is about (videos on YouTube have no transcript)

---

## Step 4: Present

- A ranked table: account (with network), post date, format, `measure`, `gain`, current count, link
- Mark new posts and partial windows
- Group by network when more than one is in scope
- What the leaders have in common: format, topic, length (`duration_seconds`), hook in the first line of text or transcript
