# AGENTS.md — radarr-mcp

MCP server exposing Radarr's v3 REST API (OpenAPI 3.0.4) as tools so an LLM can read and manage a Radarr instance: movies, movie files, queue, history, indexers, import lists, custom formats, tags, commands, system status, and more. Full surface — reads and writes. Uses FastMCP, `uv` for deps.

## Testing
- Offline suite: `make test` (or `uv run pytest`)
- Live integration (needs `RADARR_URL`/`RADARR_API_KEY`): `make test-integration`
  - GET endpoints run against the live instance.
  - POST/PUT/DELETE only run when `RADARR_WRITE_TESTS=1` (safe create→update→delete cycles against a scratch tag, then cleanup). Never point write tests at a production library.

## Tool registry and the spec
- `_TOOL_REGISTRY` in `radarr_mcp.py` is generated from the vendored spec at `tests/data/radarr_openapi.json` (pinned to Radarr develop HEAD c6fef4309de3ffdd5e0e9607ad30beb12d791333). It lists every JSON-producing endpoint under `/api/v3` plus `GET /ping`.
- Excluded on purpose: `/login`, `/logout`, `/ping` (kept), static web routes, the `.ics` calendar feed, and binary/text endpoints (media covers, raw log files) — `_req` JSON-decodes every response.
- To add a tool or refresh coverage, regenerate the registry from a newer `openapi.json` (same algorithm as the authoring script) and re-run the tests. Do not hand-edit the registry.
- Tool naming: `radarr_<verb>_<resource>` derived from path + method (e.g. `radarr_list_movie`, `radarr_add_movie`, `radarr_delete_movie`, `radarr_run_command`). Overrides for flagship/action endpoints live in the authoring script.

## Annotations convention
- GET endpoints: `readOnlyHint=True` (`READONLY`).
- POST/PUT: `readOnlyHint=False`, `destructiveHint=False` (`WRITE`).
- DELETE: `readOnlyHint=False`, `destructiveHint=True` (`DESTRUCTIVE`).
- Keep the three `ToolAnnotations` constants; never mark a write read-only.

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
- This server is not yet deployed to the Proxmox host or the christopfarr project copy. When it is, follow the pattern in the other `-mcp` servers: push tags, sync the project copy, then `ssh root@192.168.50.3 -- 'cd /root/radarr-mcp && git fetch origin && git reset --hard origin/main && uv tool install --force .'`.

## Initial state
Version starts at `0.0.0` in the initial commit. No tag on the scaffold commit; releases begin at the first `make bump-*`.
