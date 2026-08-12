# start_search + get_job

Premium search is also async: `start_search` returns a `job_id`, poll
`get_job` the same way as crawl.

## Ask your agent

```
Use messora start_search for "best noise cancelling headphones 2026",
country BR, freshness month, then poll get_job and list the top results.
```

## Tool calls

```json
{
  "name": "start_search",
  "arguments": {
    "query": "best noise cancelling headphones 2026",
    "num_results": 10,
    "country": "BR",
    "freshness": "month"
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

## Parameters — start_search

| Parameter | Type | Default | Description |
|---|---|---|---|
| `query` | string | required | Search query |
| `num_results` | int (10–100) | `10` | Number of results |
| `country` | string (ISO alpha-2) | — | E.g. `BR`, `US` |
| `freshness` | `day\|week\|month\|year\|any` | — | Recency filter |
| `tags` | string[] | — | Up to 10 tags |
| `query_fanout` | boolean | `true` | LLM-expand the query into up to 5 variants. `false` searches only the raw query — faster, one fewer LLM call |
| `use_cache` | boolean | `false` | Reuse a complete result from the last 10 minutes for the same query and filters |

## get_usage

Queries plan, credits used and account balance. No arguments — same
payload as the REST `GET /account/usage`.

Full reference: <https://docs.messora.dev/en/mcp>
