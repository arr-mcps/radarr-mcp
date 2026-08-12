# radarr_openapi.json

Radarr v3 OpenAPI spec, vendored from the Radarr repo so the tool registry and
tests stay reproducible and work offline.

- Source: `src/Radarr.Api.V3/openapi.json`
- Branch: `develop`, pinned to commit
  `c6fef4309de3ffdd5e0e9607ad30beb12d791333`
- The `_TOOL_REGISTRY` in `radarr_mcp.py` is generated from this file. To
  refresh: download a newer `openapi.json`, regenerate the registry with the
  authoring script, update the SHA here and in `radarr_mcp.py`/`AGENTS.md`, and
  re-run the tests.
