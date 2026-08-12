# radarr-mcp

Part of the [arr-mcps](https://github.com/SavageCore/arr-mcps) collection.
MCP server exposing [Radarr](https://radarr.video)'s v3 REST API
([OpenAPI 3.0.4](https://radarr.video/docs/api/)) as tools, so an LLM can read
and manage a Radarr instance: movies, movie files, the download queue, history,
indexers, import lists, custom formats, tags, commands, system status, and
more. Full surface — reads **and** writes, with destructive tools flagged.

Built with [FastMCP](https://gofastmcp.com).

## Getting an API key

Generate one in Radarr **Settings > General > Security**. Auth is the
`X-Api-Key` header.

## Install

Download a wheel from the [latest release](https://github.com/SavageCore/radarr-mcp/releases/latest)
and install it as a `uv` tool (no repo checkout needed):

```bash
uv tool install radarr_mcp-*.whl
```

This puts a `radarr-mcp` command on your PATH. Register it with Claude Code:

```bash
claude mcp add radarr \
  --env RADARR_URL=http://your-radarr-host:7878 \
  --env RADARR_API_KEY=<key> \
  -- radarr-mcp
```

### From source

```bash
uv sync
cp .env.example .env   # fill in RADARR_URL and RADARR_API_KEY
```

```bash
claude mcp add radarr \
  --env RADARR_URL=http://your-radarr-host:7878 \
  --env RADARR_API_KEY=<key> \
  -- uv run --directory /path/to/radarr-mcp radarr-mcp
```

## Config

| Env var | Required | Default |
|---|---|---|
| `RADARR_URL` | yes | - |
| `RADARR_API_KEY` | yes* | none (no `X-Api-Key` header sent if unset) |

\* Every API endpoint requires auth; practically you must set it, but the
server still starts without one so errors surface from the API rather than at
startup.

## Tools

**15 resource-scoped tools**, each covering multiple Radarr v3 endpoints (226
total) via an `operation` parameter. Call a tool with `operation` set to one
of its listed operations and an `arguments` dict matching that operation's
parameters — the tool's own description (visible to your MCP client) lists
every operation, its signature, and a one-line doc. This keeps the full REST
surface available while costing a fraction of the context budget of
registering all 226 endpoints as separate tools.

| Tool | Operations | Kind |
|---|---|---|
| `radarr_profiles_formats` | 43 | reads + writes |
| `radarr_media_library` | 32 | reads + writes |
| `radarr_config` | 28 | reads + writes |
| `radarr_import_lists` | 21 | reads + writes |
| `radarr_system_commands` | 20 | reads + writes |
| `radarr_notifications_metadata` | 18 | reads + writes |
| `radarr_download_clients` | 16 | reads + writes |
| `radarr_indexers` | 11 | reads + writes |
| `radarr_history_blocklist` | 8 | reads + writes |
| `radarr_storage` | 8 | reads + writes |
| `radarr_queue` | 7 | reads + writes |
| `radarr_tags` | 7 | reads + writes |
| `radarr_release_search` | 4 | reads + writes |
| `radarr_wanted` | 2 | read-only |
| `radarr_calendar` | 1 | read-only |

Example: `radarr_queue(operation="radarr_delete_queue", arguments={"id": 42})`.
Endpoint-level naming (`radarr_<verb>_<resource>`) is preserved as the
`operation` value, so the full endpoint list is still discoverable from each
group tool's description at runtime.

## Development

```bash
make help  # list all commands
```

| Command | Does |
|---|---|
| `make sync` | `uv sync` |
| `make test` | Offline tests - one per endpoint, mocked HTTP |
| `make test-integration` | Tests against the live instance (needs `RADARR_URL`/`RADARR_API_KEY`) |
| `make build` | Build wheel + sdist into `dist/` |
| `make bump-patch` / `bump-minor` / `bump-major` | Bump the version in `pyproject.toml` + `uv.lock` |
| `make clean` | Remove build artifacts |

The release workflow (`.github/workflows/release.yml`) builds and publishes to
[Releases](https://github.com/SavageCore/radarr-mcp/releases) whenever a `v*`
tag is pushed - so the usual flow is `make bump-patch`, commit, then tag and
push.

The integration suite is read-only by default (GET endpoints only). Set
`RADARR_WRITE_TESTS=1` to also exercise POST/PUT/DELETE against a scratch tag
(created, updated, then deleted). Never run write tests against a production
library.
