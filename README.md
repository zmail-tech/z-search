# z-search

An optimized search tool that combines SearXNG (simple) and Vane (deep) search engines with automatic mode selection based on query complexity. Injects current date/time for temporal queries.

**Author:** Zmail-Tech
**Version:** 1.2.0
**License:** MIT

Based on the original Optimized Search tool by [@cooooookiecrisp](https://github.com/cooooookiecrisp) (v1.5, MIT).

## Overview

z-search intelligently routes queries between two search backends:

- **Simple Mode** – Fast results via a SearXNG instance, with optional full-page content extraction.
- **Deep Mode** – AI-synthesized answers via a Vane instance, pulling from multiple sources.

In **auto** mode, the tool analyzes query complexity (token count, keywords, interrogative patterns, reasoning indicators) and selects the best backend automatically.

## Features

- Automatic mode selection based on query complexity heuristics
- Temporal awareness — injects current date/time for queries like "what's the weather today"
- Configurable search sources: `web`, `academia`, `social`
- Full-page content extraction with snippet consistency analysis
- Three optimization modes for deep search: `speed`, `balanced`, `quality`
- Auto optimization mode — lets the LLM (or heuristics) select the best optimization per query
- Dual configuration layers: admin valves (system-wide) and user valves (per-chat overrides)

## Requirements

- Python 3.10+
- `requests`
- `beautifulsoup4`
- `pydantic`

A running SearXNG instance and optionally a Vane instance.

## Configuration

### Admin Valves (System-wide)

Configure via Admin Panel → Tools → Gear Icon.

| Setting | Default | Description |
|---|---|---|
| `SEARXNG_URL` | `http://searxng:8080` | URL to your SearXNG instance |
| `VANE_URL` | `http://localhost:3000` | URL to your Vane instance |
| `VANE_CHAT_MODEL_PROVIDER_ID` | *(required for deep)* | Vane chat model provider UUID |
| `VANE_CHAT_MODEL_KEY` | `gpt-4o-mini` | Chat model key |
| `VANE_EMBEDDING_MODEL_PROVIDER_ID` | *(required for deep)* | Vane embedding model provider UUID |
| `VANE_EMBEDDING_MODEL_KEY` | `text-embedding-3-small` | Embedding model key |
| `VANE_TIMEOUT` | `30` | Timeout in seconds for Vane (10–120) |
| `INJECT_DATETIME` | `true` | Inject date/time into temporal queries |
| `DATETIME_FORMAT` | `%Y-%m-%d %A %B %d` | strftime format for injected date |
| `TIMEZONE` | `America/Los_Angeles` | IANA timezone for date injection |
| `DEFAULT_MODE` | `auto` | Default search mode |
| `DEFAULT_SOURCES` | `web` | Default sources (comma-separated) |
| `ENABLE_FULL_FETCH` | `true` | Enable full-page extraction in simple mode |
| `MAX_FULL_FETCH_RESULTS` | `3` | Top results to fully fetch (1–10) |
| `SHOW_SELECTION_REASONING` | `true` | Show mode selection reasoning |
| `DEFAULT_OPTIMIZATION` | `speed` | Deep search optimization: `speed`, `balanced`, `quality`, `auto` |
| `DEFAULT_MAX_RESULTS` | `5` | Max results to return (1–20) |

### User Valves (Per-chat overrides)

Available in the chat interface when using the tool. These override admin settings and persist across chats.

| Setting | Default | Description |
|---|---|---|
| `mode` | `auto` | `auto`, `simple`, or `deep` |
| `sources` | `web` | Comma-separated: `web`, `academia`, `social` |
| `full_fetch` | `true` | Enable full-page extraction |
| `full_fetch_results` | `3` | Number of results to fully fetch (1–10) |
| `show_reasoning` | `true` | Display mode selection reasoning |
| `max_results` | `5` | Maximum results to return (1–20) |
| `optimization` | `speed` | Deep search optimization: `speed`, `balanced`, `quality`, `auto` |

## Usage

```python
from z_search import Tools

search = Tools()
result = await search.optimized_search("what's the weather today in Tokyo")
```

The `optimized_search` method accepts a query string and an optional event emitter for streaming status updates.

## Search Modes

### Simple (SearXNG)
Fast, lightweight search. Returns result titles, URLs, snippets, and optionally full page content. Includes a snippet analysis engine that detects contradictions, uncertainty, and missing information to decide whether full-page fetches are needed.

### Deep (Vane)
AI-synthesized answers with source citations. Supports configurable LLM backends and optimization modes (`speed`, `balanced`, `quality`).

### Auto Optimization (Deep only)
When set to `auto`, optimization is determined per-query rather than being fixed. The tool first attempts an LLM-based classification via Vane's `/api/chat` endpoint. On any failure (timeout, unreachable endpoint, malformed response), it falls back to a heuristic scoring engine.

**Heuristic signals:**

| Signal | Direction |
|---|---|
| Query < 6 tokens | +2 → speed |
| Query > 15 tokens | +1 → quality |
| 2+ question marks | +1 → quality |
| Comparative keywords (`vs`, `compare`, `better`, etc.) | clamp to balanced |
| 2+ reasoning keywords (`explain`, `why`, `recommend`, `pros`, `cons`, etc.) | +2 → quality |
| Simple lookup patterns (`what is`, `who is`, `define`, etc.) | +2 → speed |
| Multi-part indicators (`and`, `also`, `plus`, etc.) | each +1 → quality |

Score mapping: `>= 2` → speed, `0–1` → balanced, `<= -1` → quality.

### Auto
Analyzes the query using heuristics:
- Token count threshold (>15 triggers deep)
- Comparative keywords (`vs`, `compare`, `better`, `pros/cons`, etc.)
- Complex interrogatives (`how does…`, `why is…`, `what are the best…`)
- Reasoning indicators (`best for`, `top N`, `should I`, `guide`, `how to`)

If any trigger matches, deep mode is selected; otherwise simple mode is used.

## Temporal Queries

Queries containing temporal language (`today`, `latest`, `weather`, `news`, etc.) are automatically enriched with the current date and time in multiple formats, ensuring search engines and LLMs return current results.

## License

MIT — see [LICENSE](LICENSE) for details.
