# scrape_url

Extracts content from a single URL. Consumes credit only when the backend
returns `scrape_status = success`.

## Ask your agent

```
Use the messora scrape_url tool to fetch https://example.com/product/42
as markdown, with render_js enabled.
```

## Tool call

```json
{
  "name": "scrape_url",
  "arguments": {
    "url": "https://example.com/product/42",
    "formats": ["markdown"],
    "render_js": true,
    "only_main_content": true,
    "timeout": 60
  }
}
```

## Parameters

| Parameter | Type | Default | Description |
|---|---|---|---|
| `url` | string (URI) | required | Target URL |
| `formats` | `["markdown"]` | `["markdown"]` | `markdown`, `json`, `raw` |
| `json_schema` | object | — | Required if `formats` includes `json` |
| `render_js` | boolean | `false` | Render JavaScript |
| `only_main_content` | boolean | `false` | Strip nav/footer/ads |
| `timeout` | int (1–180) | `60` | Timeout in seconds |
| `max_pages` | int (1–50) | `1` | Max pages |
| `parse_pdf` | boolean | `true` | Process PDFs |
| `tags` | string[] | — | Up to 10 tags (64 chars each) |

Full reference: <https://docs.messora.dev/en/mcp>
