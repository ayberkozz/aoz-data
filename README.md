# aoz-data

Scheduled Claude routines write daily JSON files into the folders below. Each new file pushed to `main` triggers a push notification to the app (see [`.github/workflows/notify-aoz.yml`](.github/workflows/notify-aoz.yml)).

## Folders

| Folder | Contents | File name |
| --- | --- | --- |
| [`ai-papers/`](ai-papers) | Curated AI/ML research papers with short summaries | `papers-YYYY-MM-DD.json`, `papers-YYYY-MM-DD-runN.json` |
| [`hn-digests/`](hn-digests) | Hacker News digests: health-related stories and other popular stories | `hn-digest-YYYY-MM-DD.json`, `hn-digest-YYYY-MM-DD-vN.json` |

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

## Notifications

On every push to `main` that adds a JSON file under `ai-papers/` or `hn-digests/`, the workflow sends an APNs alert to the AOZ app that includes the folder, the date and the item count. It needs the `APNS_KEY_P8`, `APNS_KEY_ID` and `APNS_DEVICE_TOKEN` secrets. The optional `APNS_HOST` variable defaults to the sandbox host.
