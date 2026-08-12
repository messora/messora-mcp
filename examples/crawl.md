# start_crawl + get_job

Crawl is async: `start_crawl` returns a `job_id`, poll `get_job` until the
state becomes `SUCCESS`, `FAILURE` or `REVOKED`. The MCP adapter does not
poll or retry charged operations for you.

## Ask your agent

```
Use messora start_crawl on https://example.com with max_pages 20, then
poll get_job until it finishes and summarize the pages found.
```

## Tool calls

```json
{
  "name": "start_crawl",
  "arguments": {
    "url": "https://example.com",
    "max_pages": 20,
    "max_depth": 2,
    "only_main_content": true
  }
}
```

```json
{
  "name": "get_job",
  "arguments": {
    "job_id": "<job_id returned above>"
  }
}
```

## Parameters — start_crawl

| Parameter | Type | Default | Description |
|---|---|---|---|
| `url` | string (URI) | required | Seed URL |
| `max_pages` | int (1–50) | required | Page cap |
| `max_depth` | int ≥ 0 | — | Max BFS depth |
| `only_main_content` | boolean | `true` | Main content only |
| `timeout_ms` | int (1000–300000) | `60000` | Per-page timeout |
| `tags` | string[] | `[]` | Usage tags |

## Parameters — get_job

| Parameter | Type | Description |
|---|---|---|
| `job_id` | string | UUID returned by `start_crawl` or `start_search` |

States: `PENDING` → `STARTED` → `SUCCESS` \| `FAILURE` \| `REVOKED`.
Recommended: poll every 2 seconds with backoff.

Full reference: <https://docs.messora.dev/en/mcp>
