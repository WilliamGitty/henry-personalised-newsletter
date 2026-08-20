# Henry's Personalised Newsletter

A daily briefing built entirely from live RSS feeds — same architecture as
`manoj-daily-briefing`, `william-daily-briefing`, and `bc-daily-briefing`,
just with Henry's own topics.

**No AI-generated content.** `build.py` never asks an LLM to "write" or
"find" news — it fetches real RSS/Atom feeds, filters by each feed's own
publish timestamp, and renders headlines/summaries/links verbatim from the
feed itself. This avoids the hallucinated-story failure mode of the
original email-briefing attempt this pattern replaced.

## Status: scaffold ready, topics pending

Henry is sending over a PDF describing what he wants covered. Once it
arrives, the only file that needs editing is the `SECTIONS` list near the
top of `build.py` — swap the placeholder "General News" section for his
real topics and their RSS feeds. Everything else (fetching, filtering,
dedup, rendering, scheduling, hosting) is already wired up and works
today with the placeholder.

## How it runs

- `build.py` runs via GitHub Actions (`.github/workflows/daily.yml`) —
  no server, no local machine needing to be on.
- Scheduled three times daily (staggered) as redundancy against GitHub's
  occasionally-flaky internal cron scheduler; `workflow_dispatch` allows
  manual triggering too.
- Publishes to GitHub Pages automatically after each successful build:
  **https://williamgitty.github.io/henry-personalised-newsletter/**
- Public repo, so no Cloudflare/Access setup needed (unlike the private
  Agilisys/EMV briefings) — this is a personal newsletter, not
  confidential.

## Adding or changing topics

Edit the `SECTIONS` list in `build.py`:

```python
SECTIONS = [
    {"key": "example", "title": "Section Title", "feeds": [FEED_URL_1, FEED_URL_2]},
]
```

Each section needs at least one working RSS/Atom feed URL. Test a
candidate feed before adding it:

```bash
curl -s -A "Mozilla/5.0" --max-time 10 -L "<feed-url>" | grep -oE "<\?xml|<rss|<feed"
```

Optional per-section fields:
- `keywords`: only include items whose title/summary contains one of
  these (case-insensitive) — use for a feed that's broader than the topic.
- `always_feeds`: feeds that are already inherently on-topic and
  shouldn't be keyword-filtered (e.g. a source's own dedicated feed for
  that exact subject).
