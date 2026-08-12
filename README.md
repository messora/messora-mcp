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

**Licença:** MIT, cobre só o conteúdo deste repositório. A implementação
hospedada do servidor MESSORA é proprietária e não está incluída.
