# AGENTS.md — radarr-mcp

MCP server exposing Radarr's v3 REST API (OpenAPI 3.0.4) as tools so an LLM can read and manage a Radarr instance: movies, movie files, queue, history, indexers, import lists, custom formats, tags, commands, system status, and more. Full surface — reads and writes. Uses FastMCP, `uv` for deps.

Exposed as **15 resource-scoped portmanteau tools**, not one tool per endpoint — see "Tool registry and the spec" below. A prior version registered all 226 endpoints individually; that blew the MCP context budget (~226 tools × ~250 tokens ≈ 57k tokens just for this one server) and has been retired.

## Testing
- Offline suite: `make test` (or `uv run pytest`)
- Live integration (needs `RADARR_URL`/`RADARR_API_KEY`): `make test-integration`
  - GET endpoints run against the live instance.
  - POST/PUT/DELETE only run when `RADARR_WRITE_TESTS=1` (safe create→update→delete cycles against a scratch tag, then cleanup). Never point write tests at a production library.

## Tool registry and the spec
- `_TOOL_REGISTRY` in `radarr_mcp.py` is generated from the vendored spec at `tests/data/radarr_openapi.json` (pinned to Radarr develop HEAD c6fef4309de3ffdd5e0e9607ad30beb12d791333). It lists every JSON-producing endpoint under `/api/v3` plus `GET /ping`.
- Excluded on purpose: `/login`, `/logout`, `/ping` (kept), static web routes, the `.ics` calendar feed, and binary/text endpoints (media covers, raw log files) — `_req` JSON-decodes every response.
- To add a tool or refresh coverage, regenerate the registry from a newer `openapi.json` (same algorithm as the authoring script) and re-run the tests. Do not hand-edit the registry.
- Endpoint function naming (internal, no longer an MCP tool name): `radarr_<verb>_<resource>` derived from path + method (e.g. `radarr_list_movie`, `radarr_add_movie`, `radarr_delete_movie`, `radarr_run_command`). Overrides for flagship/action endpoints live in the authoring script.

## Portmanteau registration — **do not go back to one tool per endpoint**
- `_GROUPS` buckets every `_TOOL_REGISTRY` name into one of 15 resource groups (`radarr_media_library`, `radarr_queue`, `radarr_config`, ...). `register_tools()` registers exactly one MCP tool per group via `_register_group`, which wraps the group's endpoint functions in a single `dispatch(operation, arguments)` closure. The endpoint functions themselves are unchanged — they're plain callables looked up by name, not separately-registered tools.
- `operation` is typed `Literal[<the group's endpoint names>]`, so FastMCP/pydantic validates it against the real endpoint list before `dispatch` ever runs — an invalid operation never reaches the group tool's body.
- Adding a new endpoint: add its entry to `_TOOL_REGISTRY` as before, then add its name to exactly one group in `_GROUPS`. `tests/test_tools.py::test_all_registry_names_grouped` fails if you forget.
- New resource area big enough to need its own group (rare): add a new `_GROUPS` key. Keep the total group count at or under ~15 — that ceiling is the entire point of this pattern.
- If you're tempted to add a per-endpoint `@mcp.tool` or an extra `mcp.add_tool` call outside `_register_group`, don't — every endpoint must be reachable only via its group's `operation` enum. A 226-tool server (one per endpoint) previously cost ~57k tokens of system-prompt budget on every session start; the 15-tool grouped version costs roughly a tenth of that.

## Annotations convention
- A group tool is `readOnlyHint=True` (`READONLY`) only when *every* operation in it is a GET (e.g. `radarr_wanted`, `radarr_calendar`). Mixed groups carry no hints.
- Per-operation write/destructive notes survive in the group tool's description: each operation line still ends with its original one-line doc, and destructive/write endpoints keep a `WRITE:`/`DESTRUCTIVE:` note in that doc string (see `_TOOL_REGISTRY`'s `doc` field).
- `READONLY`/`WRITE`/`DESTRUCTIVE` constants are kept for reference and for any future per-operation annotation work, but only `READONLY` is actually applied today (to all-GET groups).

## Auth and base path
- Auth: `X-Api-Key` header (generate in Radarr Settings > General > Security). Not bearer.
- `build_client` points at the origin with no path suffix; every registered tool carries its full path (`/api/v3/...` or `/ping`). `_req` raises `ToolError` with the API status and message on `>=400`.

## Release workflow
Always use the `make bump-*` targets to bump the version (`uv version --bump patch|minor|major`), which updates `pyproject.toml` and `uv.lock` together. Do NOT edit the version by hand.

- Bump: `make bump-patch` (or `bump-minor` / `bump-major`)
- Commit message is **just the version**, e.g. `0.1.2` — nothing else.
- Tag it `v<version>` (e.g. `v0.1.2`).
- Push main and the tag:
  ```
  git push origin main
  git push origin v<version>
  ```
- Deploy to the Proxmox host (root SSH key): pull the repo then reinstall the uv tool:
  ```
  ssh root@192.168.50.3 -- 'cd /root/radarr-mcp && git fetch origin && git reset --hard origin/main'
  ssh root@192.168.50.3 -- 'cd /root/radarr-mcp && uv tool install --force .'
  ```
  The host runs it via `uv tool install` → `/root/.local/bin/radarr-mcp` (not from the repo). Locally it is registered in the christopfarr project opencode via `uv run --directory /home/savagecore/Documents/christopfarr/mcp/radarr-mcp`.

## Initial state
Version starts at `0.0.0` in the initial commit. No tag on the scaffold commit; releases begin at the first `make bump-*`.
