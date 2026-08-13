# MESSORA MCP

Client configs, examples and the official [Model Context Protocol](https://modelcontextprotocol.io)
manifest for connecting Claude, Cursor, Codex and any MCP client to
[MESSORA](https://messora.dev) — web scraping, crawling and search for AI
agents, using the same account, API key and credit pool as the REST API.

> **This repo is documentation and config only.** The MESSORA MCP server
> implementation is closed-source and hosted at `https://mcp.messora.dev/mcp`.
> Nothing here runs a server — it just tells your MCP client how to connect
> to the one MESSORA already hosts. The MIT license below covers only the
> files in this repository.

Full docs: <https://docs.messora.dev/en/mcp> · Get an API key: <https://messora.dev>

## Quick start

1. Sign up at [messora.dev](https://messora.dev) and grab an API key.
2. Pick your client below, drop in the config, replace `your-api-key-here`.
3. Restart the client. The MESSORA tools show up in the tool picker.

### Claude Code

```bash
claude mcp add --transport http messora https://mcp.messora.dev/mcp \
  --header "Authorization: Bearer your-api-key-here"
```

Or drop [`config/claude-code.json`](config/claude-code.json) into a project-root
`.mcp.json`:

```json
{
  "mcpServers": {
    "messora": {
      "type": "http",
      "url": "https://mcp.messora.dev/mcp",
      "headers": {
        "Authorization": "Bearer your-api-key-here"
      }
    }
  }
}
```

> This is Claude Code (the CLI), not the Claude Desktop app. Claude
> Desktop's remote MCP connectors go through its own OAuth-based UI, not a
> hand-edited config with a static Bearer header.

### Cursor

Edit `.mcp.json` at the project root or `~/.cursor/mcp.json` — see
[`config/cursor-mcp.json`](config/cursor-mcp.json). Same shape as Claude
Code above.

### Codex CLI

Edit `~/.codex/config.toml` — see
[`config/codex-config.toml`](config/codex-config.toml):

```toml
[mcp_servers.messora]
url = "https://mcp.messora.dev/mcp"
bearer_token_env_var = "MESSORA_API_KEY"
```

Then export the key before starting Codex:

```bash
export MESSORA_API_KEY="your-api-key-here"
```

> The MCP server only accepts `Authorization: Bearer <key>`. The
> `X-API-Key` header used by the REST API is **not** accepted here.

## Available tools

The server exposes exactly five tools. Full parameter tables and worked
examples are in [`examples/`](examples):

| Tool | Purpose |
|---|---|
| [`scrape_url`](examples/scrape.md) | Extract content from one URL |
| [`start_crawl`](examples/crawl.md) | Start an async multi-page crawl, returns `job_id` |
| [`start_search`](examples/search.md) | Start an async premium web search, returns `job_id` |
| [`get_job`](examples/crawl.md#parameters--get_job) | Poll the state of a crawl or search job |
| [`get_usage`](examples/search.md#get_usage) | Query plan, credits used and account balance |

`start_crawl` and `start_search` are async: poll `get_job` until the state
is `SUCCESS`, `FAILURE` or `REVOKED`. The MCP adapter does not poll or
retry charged operations for you.

## What a real run looks like

Actual responses from `https://mcp.messora.dev/mcp`, captured 2026-08-13 on a
Free account. Nothing here is mocked or hand-written.

**`get_usage`** — no credit cost, use it to check where you stand:

```json
{
  "plan": "free",
  "monthly_credits": 1000,
  "credits_used": 10,
  "remaining_credits": 990,
  "breakdown": [{ "endpoint": "/crawl", "credits_used": 10 }]
}
```

**`scrape_url`** with `formats: ["markdown"]` — the common case, **1 credit**:

```json
{
  "success": true,
  "scrape_status": "success",
  "markdown": "# Example Domain\n\nThis domain is for use in documentation examples without needing permission. Avoid use in operations.\n\n[Learn more](https://iana.org/domains/example)",
  "credits_used": 1,
  "remaining_credits": 989,
  "diagnostics": {
    "engine": "tls",
    "latency_ms": 74,
    "http_status": 200,
    "pages_fetched": 1,
    "retry_count": 0,
    "from_cache": false
  }
}
```

**`scrape_url`** with `formats: ["json"]` + a `json_schema` — structured
extraction, **10 credits** (it runs an LLM pass, so it is not priced like
markdown):

```json
{
  "success": true,
  "scrape_status": "success",
  "credits_used": 10,
  "remaining_credits": 979,
  "json": {
    "title": "Example Domain",
    "purpose": "This domain is for use in documentation examples without needing permission.",
    "external_links": null
  },
  "diagnostics": { "engine": "tls", "latency_ms": 23, "http_status": 200 }
}
```

Note `external_links` came back `null`: it was **not** in the schema's
`required` list and the page has only one link, so the extractor left it
empty instead of inventing a value. Put a field in `required` when you need
the model to commit to it.

Two things worth knowing before you budget:

- `credits_used` and `remaining_credits` come back on **every** charged call
  — you never have to guess.
- `diagnostics.engine` tells you which path served the request (`tls` here).
  Heavier anti-bot targets escalate to a browser engine and take longer.

## Troubleshooting

| Symptom | Likely cause |
|---|---|
| `401` on every call | The MCP server only accepts `Authorization: Bearer <key>`. `X-API-Key` works on the REST API but **not** here. |
| Tools don't show up in the picker | Client wasn't restarted after editing the config, or the config went in the wrong file. Claude Desktop is not the same as Claude Code — see the note above. |
| `403` with `domain_not_supported_in_beta` | `linkedin.com` and `instagram.com` are refused during the beta. The refusal happens before the fetch, so **no credit is charged**. |
| `402` | Out of credits. Call `get_usage` to confirm, then top up or wait for the monthly reset. |
| Crawl/search "hangs" | They're async by design. `start_crawl`/`start_search` return a `job_id` immediately; you must poll `get_job` until `SUCCESS`, `FAILURE` or `REVOKED`. |
| JSON extraction cost a surprise | `formats: ["json"]` runs an LLM pass and costs more than `markdown`. Check `credits_estimated` in `diagnostics`. |

No universal unblocking is promised. Some targets will fail, and the
`diagnostics` block tells you what was tried.

## server.json

[`server.json`](server.json) is the manifest MESSORA publishes to the
official [MCP Registry](https://registry.modelcontextprotocol.io) under
`dev.messora/messora`. It's included here for reference — verify the live
entry with:

```bash
curl -s "https://registry.modelcontextprotocol.io/v0.1/servers/dev.messora%2Fmessora/versions/latest"
```

## License

MIT — see [LICENSE](LICENSE). Covers only the contents of this repository
(configs, examples, `server.json`, docs). The hosted MESSORA server
implementation is proprietary and not included.

---

## Português (Brasil)

Configs de cliente, exemplos e o manifesto oficial do
[Model Context Protocol](https://modelcontextprotocol.io) pra conectar
Claude, Cursor, Codex e qualquer cliente MCP à [MESSORA](https://messora.dev)
— scraping, crawl e busca web pra agentes de IA, usando a mesma conta,
API key e pool de créditos da API REST.

> **Este repositório é só documentação e config.** A implementação do
> servidor MCP da MESSORA é fechada e hospedada em
> `https://mcp.messora.dev/mcp`. Nada aqui roda servidor — só ensina seu
> cliente MCP a se conectar ao servidor que a MESSORA já hospeda. A
> licença MIT abaixo cobre só os arquivos deste repositório.

Docs completas: <https://docs.messora.dev/pt-BR/mcp> · Criar API key: <https://messora.dev>

**Setup rápido:** crie conta em [messora.dev](https://messora.dev), pegue
uma API key, use os configs em [`config/`](config) (Claude Code, Cursor,
Codex CLI) trocando `your-api-key-here` pela sua chave, reinicie o
cliente. As ferramentas MESSORA aparecem no seletor de tools.

**Ferramentas disponíveis:** `scrape_url`, `start_crawl`, `start_search`,
`get_job`, `get_usage` — parâmetros e exemplos completos em
[`examples/`](examples).

**Execução real:** a seção [What a real run looks like](#what-a-real-run-looks-like)
traz respostas capturadas do servidor em 2026-08-13, sem mock. Resumo dos
custos: `markdown` custa **1 crédito**, extração JSON com schema custa **10**
(roda LLM). Toda chamada cobrada devolve `credits_used` e `remaining_credits`.

**Problemas comuns:** ver [Troubleshooting](#troubleshooting) — `401` costuma
ser uso de `X-API-Key` (o MCP só aceita `Authorization: Bearer`), tools que
não aparecem costumam ser cliente não reiniciado, e crawl/search "travados"
normalmente são só a natureza assíncrona (precisa fazer polling de `get_job`).

**Licença:** MIT, cobre só o conteúdo deste repositório. A implementação
hospedada do servidor MESSORA é proprietária e não está incluída.
