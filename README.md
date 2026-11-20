# ublock-filter

A [uBlock Origin](https://github.com/gorhill/uBlock) filter list that removes ads (promoted posts) from X (Twitter).

## Install

**One click:** with uBlock Origin installed, click the button on this page on github.com and confirm with **Subscribe** in the tab that uBO opens:

[![Subscribe in uBlock Origin](https://img.shields.io/badge/uBlock%20Origin-Subscribe-800000?style=for-the-badge&logo=ublockorigin&logoColor=white)](https://subscribe.adblockplus.org/?location=https%3A%2F%2Fcdn.jsdelivr.net%2Fgh%2Fandrizan%2Fublock-filter%40main%2Ftwitter-ads.txt&title=X%20%28Twitter%29%20Ads)

**Import by URL (auto-updates):**

1. Open the uBlock Origin dashboard → **Filter lists**.
2. Scroll to the bottom, tick **Import...** and paste:
   ```
   https://cdn.jsdelivr.net/gh/andrizan/ublock-filter@main/twitter-ads.txt
   ```
3. Click **Apply changes**, then reload x.com.

The list is served through the jsDelivr CDN, which caches it for up to 12 hours, so a fix pushed here can take that long to reach subscribers.

**Or paste manually:** copy the contents of [`twitter-ads.txt`](twitter-ads.txt) into the dashboard's **My filters** tab and click **Apply changes**. Manual copies don't auto-update.

### Requirements

- uBlock Origin **1.72.0 or newer** (the list uses uBO-flavored JSONPath, `v2:` syntax).
- The full uBlock Origin extension (Firefox, or a browser that still runs MV2). **uBO Lite (MV3) is not supported.**

## What it blocks

| Where on x.com | Source of the ads | Covered by |
| --- | --- | --- |
| Home → For you | `HomeTimeline` | JSON rule |
| Home → Following | `HomeLatestTimelineQuery` | JSON rule |
| Replies under a post | `TweetDetail` | JSON rule (see limitations) |
| Search, profiles, other timelines | any GraphQL timeline | JSON rule |
| Ad impression logging | `api.x.com/1.1/promoted_content/` | network rule |

## How it works

X sends ads inside its GraphQL timeline responses as entries whose `entry_id` starts with `promoted-` (e.g. `promoted-tweet-<id>-<hash>`) and that carry a `promoted_metadata` object.

1. **JSON rules (primary).** `json-edit-fetch-response` removes those entries from every `/graphql/` response before X renders them:
   ```
   x.com##+js(json-edit-fetch-response, v2:$..entries[?@.entry_id^="promoted-"], propsToMatch, /graphql/)
   x.com##+js(json-edit-fetch-response, v2:$..items[?@.entry_id*="promoted-"], propsToMatch, /graphql/)
   ```
   Removing ads from the data matters on X: the timeline is a virtualized list with absolutely positioned rows that are not re-measured, so hiding an ad with CSS leaves a large blank gap.

2. **Network rule.** Blocks ad impression/engagement logging.

3. **Cosmetic fallback.** If X changes its JSON format and the JSON rules stop matching, a CSS rule hides timeline rows that have no timestamp link in the post header. Regular posts always have one, while ads show "Ad" (or nothing) in its place. It works in any UI language, but it does leave a blank gap.

## Known limitations

- **Reply ads not yet observed.** Ads in replies were reported but did not show up while the list was being written, so the reply coverage relies on the generic `$..entries` / `$..items` rules and is untested.
- **Legacy frontend not covered.** Rules target X's current web frontend (snake_case `entry_id`, Tailwind markup). The older `data-testid`-based frontend is not targeted.
- **Promoted quote posts.** The cosmetic fallback can miss an ad that quotes another post, because the quoted post carries its own timestamp. The JSON rules are not affected.

## Troubleshooting

If ads still appear:

1. Make sure the list is enabled and up to date (dashboard → **Filter lists** → **Update now**).
2. Open the uBO logger on an x.com tab and check that the `json-edit-fetch-response` scriptlets are injected.
3. If X changed its response format, please open an issue with the operation name (e.g. `HomeTimeline`) and the path to the promoted entry.

## Maintaining

After pushing a change to `twitter-ads.txt`, bump `! Last modified:` and purge the CDN cache so subscribers get it right away:

```
https://purge.jsdelivr.net/gh/andrizan/ublock-filter@main/twitter-ads.txt
```
