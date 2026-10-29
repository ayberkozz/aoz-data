# aoz-data

Scheduled Claude routines write daily JSON files into the folders below. Each new file pushed to `main` triggers a push notification to the app (see [`.github/workflows/notify-aoz.yml`](.github/workflows/notify-aoz.yml)).

## Folders

| Folder | Contents | File name |
| --- | --- | --- |
| [`ai-papers/`](ai-papers) | Curated AI/ML research papers with short summaries | `papers-YYYY-MM-DD.json`, `papers-YYYY-MM-DD-runN.json` |
| [`opportunities/`](opportunities) | Weekly report (Saturday): solo ideas, startup ideas and future-technology signals | `opportunities-YYYY-MM-DD.json`, `opportunities-YYYY-MM-DD-runN.json` |
| [`hn-digests/`](hn-digests) | Hacker News digests: health-related stories and other popular stories | `hn-digest-YYYY-MM-DD.json`, `hn-digest-YYYY-MM-DD-vN.json` |
| [`medical-futurist/`](medical-futurist) | New articles from The Medical Futurist with short summaries | `mf-YYYY-MM-DD.json`, `mf-YYYY-MM-DD-runN.json` |

When a routine runs more than once on the same day, it adds a `-runN` / `-vN` suffix instead of overwriting the earlier file.

## Formats

### `ai-papers/`

```json
{
  "date": "2026-10-04",
  "timestamp": "2026-10-04T05:09:57Z",
  "papers": [
    {
      "title": "LoRA: Low-Rank Adaptation of Large Language Models",
      "pdf_url": "https://arxiv.org/pdf/2106.09685",
      "summary": "…",
      "page_count": 13
    }
  ]
}
```

### `hn-digests/`

```json
{
  "date": "2026-10-03",
  "timestamp": "2026-10-04T10:00:00Z",
  "summary_language": "tr",
  "top_stories": [
    {
      "rank": 1,
      "title": "…",
      "points": 613,
      "num_comments": 315,
      "summary": "…",
      "hn_url": "https://news.ycombinator.com/item?id=…",
      "article_url": "https://…"
    }
  ],
  "health_stories": [ /* same shape as top_stories */ ]
}
```

Older digests use `other_stories` instead of `top_stories`, and may leave out `rank`, `num_comments` and `summary_language`. Clients should treat those as optional.

### `medical-futurist/`

```json
{
  "date": "2026-10-29",
  "timestamp": "2026-10-29T14:50:00Z",
  "summary_language": "tr",
  "articles": [
    {
      "title": "…",
      "summary": "…",
      "url": "https://…"
    }
  ]
}
```

### `opportunities/`

```json
{
  "date": "2026-10-31",
  "timestamp": "2026-10-31T06:00:00Z",
  "summary": "2-3 sentences on what stood out",
  "solo": [
    {
      "title": "…", "problem": "…",
      "evidence": [ { "title": "…", "url": "https://…" } ],
      "competitors": "…", "first_version": "…", "effort": "…",
      "monetization": "…", "risks": "…",
      "score": 7, "score_reason": "…", "confidence": "high | medium | low"
    }
  ],
  "startup": [ /* same shape as solo */ ],
  "future_signals": [
    {
      "technology": "…", "stage": "emerging | rising | mainstream",
      "why_now": "…", "could_enable": "…",
      "signals": [ { "title": "…", "url": "https://…" } ],
      "confidence": "high | medium | low"
    }
  ],
  "limitations": "what could not be checked"
}
```

Only `date`, `timestamp` and the titles are required; the app treats everything else as optional.

## Notifications

On every push to `main` that adds a JSON file under `ai-papers/`, `hn-digests/`, `medical-futurist/` or `opportunities/`, the workflow sends an APNs alert to the AOZ app that includes the folder, the date and the item count. It needs the `APNS_KEY_P8`, `APNS_KEY_ID` and `APNS_DEVICE_TOKEN` secrets. The optional `APNS_HOST` variable defaults to the sandbox host.

## How the app uses these files

In the AOZ app the four folders appear together under **Routines**. There is no refresh button: the app imports new files on launch, when it returns to the foreground, on a background refresh and when the push notification above arrives.

Right after each import the app also syncs the data to the user's Obsidian vault (the **Second Brain** screen), writing `AOZ HN Digest.md`, `AOZ AI Papers.md`, `AOZ Medical Futurist.md` and `AOZ Opportunities.md`. Each file covers the most recent days or reports and is overwritten on every sync.
