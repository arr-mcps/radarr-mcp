# radarr-mcp

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

One tool per Radarr v3 JSON endpoint (plus `GET /ping`). Naming is
`radarr_<verb>_<resource>`. GET endpoints are read-only; POST/PUT are writes;
DELETE endpoints are flagged destructive.

| Tool | Method | Endpoint |
|---|---|---|

**AlternativeTitle**
| `radarr_list_alttitle` | GET | `/api/v3/alttitle` |
| `radarr_get_alttitle` | GET | `/api/v3/alttitle/{id}` |

**AutoTagging**
| `radarr_create_autotagging` | POST | `/api/v3/autotagging` |
| `radarr_list_autotagging` | GET | `/api/v3/autotagging` |
| `radarr_get_autotagging_schema` | GET | `/api/v3/autotagging/schema` |
| `radarr_update_autotagging` | PUT | `/api/v3/autotagging/{id}` |
| `radarr_delete_autotagging` | DELETE | `/api/v3/autotagging/{id}` |
| `radarr_get_autotagging` | GET | `/api/v3/autotagging/{id}` |

**Backup**
| `radarr_list_system_backup` | GET | `/api/v3/system/backup` |
| `radarr_restore_backup_upload` | POST | `/api/v3/system/backup/restore/upload` |
| `radarr_restore_backup` | POST | `/api/v3/system/backup/restore/{id}` |
| `radarr_delete_system_backup` | DELETE | `/api/v3/system/backup/{id}` |

**Blocklist**
| `radarr_list_blocklist` | GET | `/api/v3/blocklist` |
| `radarr_bulk_delete_blocklist` | DELETE | `/api/v3/blocklist/bulk` |
| `radarr_list_blocklist_movie` | GET | `/api/v3/blocklist/movie` |
| `radarr_delete_blocklist` | DELETE | `/api/v3/blocklist/{id}` |

**Calendar**
| `radarr_list_calendar` | GET | `/api/v3/calendar` |

**Collection**
| `radarr_list_collection` | GET | `/api/v3/collection` |
| `radarr_update_collections` | PUT | `/api/v3/collection` |
| `radarr_update_collection` | PUT | `/api/v3/collection/{id}` |
| `radarr_get_collection` | GET | `/api/v3/collection/{id}` |

**Command**
| `radarr_run_command` | POST | `/api/v3/command` |
| `radarr_list_command` | GET | `/api/v3/command` |
| `radarr_delete_command` | DELETE | `/api/v3/command/{id}` |
| `radarr_get_command` | GET | `/api/v3/command/{id}` |

**Credit**
| `radarr_list_credit` | GET | `/api/v3/credit` |
| `radarr_get_credit` | GET | `/api/v3/credit/{id}` |

**CustomFilter**
| `radarr_list_customfilter` | GET | `/api/v3/customfilter` |
| `radarr_create_customfilter` | POST | `/api/v3/customfilter` |
| `radarr_update_customfilter` | PUT | `/api/v3/customfilter/{id}` |
| `radarr_delete_customfilter` | DELETE | `/api/v3/customfilter/{id}` |
| `radarr_get_customfilter` | GET | `/api/v3/customfilter/{id}` |

**CustomFormat**
| `radarr_list_customformat` | GET | `/api/v3/customformat` |
| `radarr_create_customformat` | POST | `/api/v3/customformat` |
| `radarr_bulk_update_customformat` | PUT | `/api/v3/customformat/bulk` |
| `radarr_bulk_delete_customformat` | DELETE | `/api/v3/customformat/bulk` |
| `radarr_get_customformat_schema` | GET | `/api/v3/customformat/schema` |
| `radarr_update_customformat` | PUT | `/api/v3/customformat/{id}` |
| `radarr_delete_customformat` | DELETE | `/api/v3/customformat/{id}` |
| `radarr_get_customformat` | GET | `/api/v3/customformat/{id}` |

**Cutoff**
| `radarr_list_wanted_cutoff` | GET | `/api/v3/wanted/cutoff` |

**DelayProfile**
| `radarr_create_delayprofile` | POST | `/api/v3/delayprofile` |
| `radarr_list_delayprofile` | GET | `/api/v3/delayprofile` |
| `radarr_reorder_delayprofile` | PUT | `/api/v3/delayprofile/reorder/{id}` |
| `radarr_delete_delayprofile` | DELETE | `/api/v3/delayprofile/{id}` |
| `radarr_update_delayprofile` | PUT | `/api/v3/delayprofile/{id}` |
| `radarr_get_delayprofile` | GET | `/api/v3/delayprofile/{id}` |

**DiskSpace**
| `radarr_get_diskspace` | GET | `/api/v3/diskspace` |

**DownloadClient**
| `radarr_list_downloadclient` | GET | `/api/v3/downloadclient` |
| `radarr_create_downloadclient` | POST | `/api/v3/downloadclient` |
| `radarr_action_downloadclient` | POST | `/api/v3/downloadclient/action/{name}` |
| `radarr_bulk_update_downloadclient` | PUT | `/api/v3/downloadclient/bulk` |
| `radarr_bulk_delete_downloadclient` | DELETE | `/api/v3/downloadclient/bulk` |
| `radarr_get_downloadclient_schema` | GET | `/api/v3/downloadclient/schema` |
| `radarr_test_downloadclient` | POST | `/api/v3/downloadclient/test` |
| `radarr_test_all_downloadclient` | POST | `/api/v3/downloadclient/testall` |
| `radarr_update_downloadclient` | PUT | `/api/v3/downloadclient/{id}` |
| `radarr_delete_downloadclient` | DELETE | `/api/v3/downloadclient/{id}` |
| `radarr_get_downloadclient` | GET | `/api/v3/downloadclient/{id}` |

**DownloadClientConfig**
| `radarr_get_config_downloadclient` | GET | `/api/v3/config/downloadclient` |
| `radarr_update_config_downloadclient` | PUT | `/api/v3/config/downloadclient/{id}` |
| `radarr_get_config_downloadclient_by_id` | GET | `/api/v3/config/downloadclient/{id}` |

**ExtraFile**
| `radarr_list_extrafile` | GET | `/api/v3/extrafile` |

**FileSystem**
| `radarr_list_filesystem` | GET | `/api/v3/filesystem` |
| `radarr_list_filesystem_mediafiles` | GET | `/api/v3/filesystem/mediafiles` |
| `radarr_list_filesystem_type` | GET | `/api/v3/filesystem/type` |

**Health**
| `radarr_get_health` | GET | `/api/v3/health` |

**History**
| `radarr_list_history` | GET | `/api/v3/history` |
| `radarr_mark_history_item_failed` | POST | `/api/v3/history/failed/{id}` |
| `radarr_list_history_movie` | GET | `/api/v3/history/movie` |
| `radarr_list_history_since` | GET | `/api/v3/history/since` |

**HostConfig**
| `radarr_get_config_host` | GET | `/api/v3/config/host` |
| `radarr_update_config_host` | PUT | `/api/v3/config/host/{id}` |
| `radarr_get_config_host_by_id` | GET | `/api/v3/config/host/{id}` |

**ImportList**
| `radarr_list_importlist` | GET | `/api/v3/importlist` |
| `radarr_create_importlist` | POST | `/api/v3/importlist` |
| `radarr_action_importlist` | POST | `/api/v3/importlist/action/{name}` |
| `radarr_bulk_update_importlist` | PUT | `/api/v3/importlist/bulk` |
| `radarr_bulk_delete_importlist` | DELETE | `/api/v3/importlist/bulk` |
| `radarr_get_importlist_schema` | GET | `/api/v3/importlist/schema` |
| `radarr_test_importlist` | POST | `/api/v3/importlist/test` |
| `radarr_test_all_importlist` | POST | `/api/v3/importlist/testall` |
| `radarr_update_importlist` | PUT | `/api/v3/importlist/{id}` |
| `radarr_delete_importlist` | DELETE | `/api/v3/importlist/{id}` |
| `radarr_get_importlist` | GET | `/api/v3/importlist/{id}` |

**ImportListConfig**
| `radarr_get_config_importlist` | GET | `/api/v3/config/importlist` |
| `radarr_update_config_importlist` | PUT | `/api/v3/config/importlist/{id}` |
| `radarr_get_config_importlist_by_id` | GET | `/api/v3/config/importlist/{id}` |

**ImportListExclusion**
| `radarr_list_exclusions` | GET | `/api/v3/exclusions` |
| `radarr_create_exclusions` | POST | `/api/v3/exclusions` |
| `radarr_bulk_create_exclusions` | POST | `/api/v3/exclusions/bulk` |
| `radarr_bulk_delete_exclusions` | DELETE | `/api/v3/exclusions/bulk` |
| `radarr_list_exclusions_paged` | GET | `/api/v3/exclusions/paged` |
| `radarr_update_exclusions` | PUT | `/api/v3/exclusions/{id}` |
| `radarr_delete_exclusions` | DELETE | `/api/v3/exclusions/{id}` |
| `radarr_get_exclusions` | GET | `/api/v3/exclusions/{id}` |

**ImportListMovies**
| `radarr_list_importlist_movie` | GET | `/api/v3/importlist/movie` |
| `radarr_import_list_movie` | POST | `/api/v3/importlist/movie` |

**Indexer**
| `radarr_list_indexer` | GET | `/api/v3/indexer` |
| `radarr_create_indexer` | POST | `/api/v3/indexer` |
| `radarr_action_indexer` | POST | `/api/v3/indexer/action/{name}` |
| `radarr_bulk_update_indexer` | PUT | `/api/v3/indexer/bulk` |
| `radarr_bulk_delete_indexer` | DELETE | `/api/v3/indexer/bulk` |
| `radarr_get_indexer_schema` | GET | `/api/v3/indexer/schema` |
| `radarr_test_indexer` | POST | `/api/v3/indexer/test` |
| `radarr_test_all_indexer` | POST | `/api/v3/indexer/testall` |
| `radarr_update_indexer` | PUT | `/api/v3/indexer/{id}` |
| `radarr_delete_indexer` | DELETE | `/api/v3/indexer/{id}` |
| `radarr_get_indexer` | GET | `/api/v3/indexer/{id}` |

**IndexerConfig**
| `radarr_get_config_indexer` | GET | `/api/v3/config/indexer` |
| `radarr_update_config_indexer` | PUT | `/api/v3/config/indexer/{id}` |
| `radarr_get_config_indexer_by_id` | GET | `/api/v3/config/indexer/{id}` |

**IndexerFlag**
| `radarr_get_indexerflag` | GET | `/api/v3/indexerflag` |

**Language**
| `radarr_list_language` | GET | `/api/v3/language` |
| `radarr_get_language` | GET | `/api/v3/language/{id}` |

**Localization**
| `radarr_list_localization` | GET | `/api/v3/localization` |
| `radarr_get_localization_language` | GET | `/api/v3/localization/language` |

**Log**
| `radarr_list_log` | GET | `/api/v3/log` |

**LogFile**
| `radarr_list_log_file` | GET | `/api/v3/log/file` |

**ManualImport**
| `radarr_list_manualimport` | GET | `/api/v3/manualimport` |
| `radarr_commit_manual_import` | POST | `/api/v3/manualimport` |

**MediaManagementConfig**
| `radarr_get_config_mediamanagement` | GET | `/api/v3/config/mediamanagement` |
| `radarr_update_config_mediamanagement` | PUT | `/api/v3/config/mediamanagement/{id}` |
| `radarr_get_config_mediamanagement_by_id` | GET | `/api/v3/config/mediamanagement/{id}` |

**Metadata**
| `radarr_list_metadata` | GET | `/api/v3/metadata` |
| `radarr_create_metadata` | POST | `/api/v3/metadata` |
| `radarr_action_metadata` | POST | `/api/v3/metadata/action/{name}` |
| `radarr_get_metadata_schema` | GET | `/api/v3/metadata/schema` |
| `radarr_test_metadata` | POST | `/api/v3/metadata/test` |
| `radarr_test_all_metadata` | POST | `/api/v3/metadata/testall` |
| `radarr_update_metadata` | PUT | `/api/v3/metadata/{id}` |
| `radarr_delete_metadata` | DELETE | `/api/v3/metadata/{id}` |
| `radarr_get_metadata` | GET | `/api/v3/metadata/{id}` |

**MetadataConfig**
| `radarr_get_config_metadata` | GET | `/api/v3/config/metadata` |
| `radarr_update_config_metadata` | PUT | `/api/v3/config/metadata/{id}` |
| `radarr_get_config_metadata_by_id` | GET | `/api/v3/config/metadata/{id}` |

**Missing**
| `radarr_list_wanted_missing` | GET | `/api/v3/wanted/missing` |

**Movie**
| `radarr_list_movie` | GET | `/api/v3/movie` |
| `radarr_add_movie` | POST | `/api/v3/movie` |
| `radarr_update_movie` | PUT | `/api/v3/movie/{id}` |
| `radarr_delete_movie` | DELETE | `/api/v3/movie/{id}` |
| `radarr_get_movie` | GET | `/api/v3/movie/{id}` |

**MovieEditor**
| `radarr_bulk_update_movie` | PUT | `/api/v3/movie/editor` |
| `radarr_bulk_delete_movie` | DELETE | `/api/v3/movie/editor` |

**MovieFile**
| `radarr_list_moviefile` | GET | `/api/v3/moviefile` |
| `radarr_bulk_delete_moviefile` | DELETE | `/api/v3/moviefile/bulk` |
| `radarr_bulk_update_moviefile` | PUT | `/api/v3/moviefile/bulk` |
| `radarr_bulk_edit_moviefiles` | PUT | `/api/v3/moviefile/editor` |
| `radarr_update_moviefile` | PUT | `/api/v3/moviefile/{id}` |
| `radarr_delete_moviefile` | DELETE | `/api/v3/moviefile/{id}` |
| `radarr_get_moviefile` | GET | `/api/v3/moviefile/{id}` |

**MovieFolder**
| `radarr_get_movie_folder` | GET | `/api/v3/movie/{id}/folder` |

**MovieImport**
| `radarr_import_movie` | POST | `/api/v3/movie/import` |

**MovieLookup**
| `radarr_lookup_movie` | GET | `/api/v3/movie/lookup` |
| `radarr_lookup_movie_by_imdb_id` | GET | `/api/v3/movie/lookup/imdb` |
| `radarr_lookup_movie_by_tmdb_id` | GET | `/api/v3/movie/lookup/tmdb` |

**NamingConfig**
| `radarr_get_config_naming` | GET | `/api/v3/config/naming` |
| `radarr_list_config_naming_examples` | GET | `/api/v3/config/naming/examples` |
| `radarr_update_config_naming` | PUT | `/api/v3/config/naming/{id}` |
| `radarr_get_config_naming_by_id` | GET | `/api/v3/config/naming/{id}` |

**Notification**
| `radarr_list_notification` | GET | `/api/v3/notification` |
| `radarr_create_notification` | POST | `/api/v3/notification` |
| `radarr_action_notification` | POST | `/api/v3/notification/action/{name}` |
| `radarr_get_notification_schema` | GET | `/api/v3/notification/schema` |
| `radarr_test_notification` | POST | `/api/v3/notification/test` |
| `radarr_test_all_notification` | POST | `/api/v3/notification/testall` |
| `radarr_update_notification` | PUT | `/api/v3/notification/{id}` |
| `radarr_delete_notification` | DELETE | `/api/v3/notification/{id}` |
| `radarr_get_notification` | GET | `/api/v3/notification/{id}` |

**Parse**
| `radarr_get_parse` | GET | `/api/v3/parse` |

**Ping**
| `radarr_ping` | GET | `/ping` |

**QualityDefinition**
| `radarr_list_qualitydefinition` | GET | `/api/v3/qualitydefinition` |
| `radarr_get_qualitydefinition_limits` | GET | `/api/v3/qualitydefinition/limits` |
| `radarr_update_quality_definitions` | PUT | `/api/v3/qualitydefinition/update` |
| `radarr_update_qualitydefinition` | PUT | `/api/v3/qualitydefinition/{id}` |
| `radarr_get_qualitydefinition` | GET | `/api/v3/qualitydefinition/{id}` |

**QualityProfile**
| `radarr_create_qualityprofile` | POST | `/api/v3/qualityprofile` |
| `radarr_list_qualityprofile` | GET | `/api/v3/qualityprofile` |
| `radarr_delete_qualityprofile` | DELETE | `/api/v3/qualityprofile/{id}` |
| `radarr_update_qualityprofile` | PUT | `/api/v3/qualityprofile/{id}` |
| `radarr_get_qualityprofile` | GET | `/api/v3/qualityprofile/{id}` |

**QualityProfileSchema**
| `radarr_get_qualityprofile_schema` | GET | `/api/v3/qualityprofile/schema` |

**Queue**
| `radarr_list_queue` | GET | `/api/v3/queue` |
| `radarr_bulk_delete_queue` | DELETE | `/api/v3/queue/bulk` |
| `radarr_delete_queue` | DELETE | `/api/v3/queue/{id}` |

**QueueAction**
| `radarr_grab_queue_bulk` | POST | `/api/v3/queue/grab/bulk` |
| `radarr_grab_queue_item` | POST | `/api/v3/queue/grab/{id}` |

**QueueDetails**
| `radarr_get_queue_details` | GET | `/api/v3/queue/details` |

**QueueStatus**
| `radarr_get_queue_status` | GET | `/api/v3/queue/status` |

**Release**
| `radarr_search_releases` | POST | `/api/v3/release` |
| `radarr_list_release` | GET | `/api/v3/release` |

**ReleaseProfile**
| `radarr_create_releaseprofile` | POST | `/api/v3/releaseprofile` |
| `radarr_list_releaseprofile` | GET | `/api/v3/releaseprofile` |
| `radarr_delete_releaseprofile` | DELETE | `/api/v3/releaseprofile/{id}` |
| `radarr_update_releaseprofile` | PUT | `/api/v3/releaseprofile/{id}` |
| `radarr_get_releaseprofile` | GET | `/api/v3/releaseprofile/{id}` |

**ReleasePush**
| `radarr_push_release` | POST | `/api/v3/release/push` |

**RemotePathMapping**
| `radarr_create_remotepathmapping` | POST | `/api/v3/remotepathmapping` |
| `radarr_list_remotepathmapping` | GET | `/api/v3/remotepathmapping` |
| `radarr_delete_remotepathmapping` | DELETE | `/api/v3/remotepathmapping/{id}` |
| `radarr_update_remotepathmapping` | PUT | `/api/v3/remotepathmapping/{id}` |
| `radarr_get_remotepathmapping` | GET | `/api/v3/remotepathmapping/{id}` |

**RenameMovie**
| `radarr_get_rename` | GET | `/api/v3/rename` |

**RootFolder**
| `radarr_create_rootfolder` | POST | `/api/v3/rootfolder` |
| `radarr_list_rootfolder` | GET | `/api/v3/rootfolder` |
| `radarr_delete_rootfolder` | DELETE | `/api/v3/rootfolder/{id}` |
| `radarr_get_rootfolder` | GET | `/api/v3/rootfolder/{id}` |

**System**
| `radarr_restart_radarr` | POST | `/api/v3/system/restart` |
| `radarr_get_system_routes` | GET | `/api/v3/system/routes` |
| `radarr_get_system_routes_duplicate` | GET | `/api/v3/system/routes/duplicate` |
| `radarr_shutdown_radarr` | POST | `/api/v3/system/shutdown` |
| `radarr_get_system_status` | GET | `/api/v3/system/status` |

**Tag**
| `radarr_list_tag` | GET | `/api/v3/tag` |
| `radarr_create_tag` | POST | `/api/v3/tag` |
| `radarr_update_tag` | PUT | `/api/v3/tag/{id}` |
| `radarr_delete_tag` | DELETE | `/api/v3/tag/{id}` |
| `radarr_get_tag` | GET | `/api/v3/tag/{id}` |

**TagDetails**
| `radarr_list_tag_detail` | GET | `/api/v3/tag/detail` |
| `radarr_get_tag_detail` | GET | `/api/v3/tag/detail/{id}` |

**Task**
| `radarr_get_system_task` | GET | `/api/v3/system/task` |
| `radarr_get_system_task_by_id` | GET | `/api/v3/system/task/{id}` |

**UiConfig**
| `radarr_get_config_ui` | GET | `/api/v3/config/ui` |
| `radarr_update_config_ui` | PUT | `/api/v3/config/ui/{id}` |
| `radarr_get_config_ui_by_id` | GET | `/api/v3/config/ui/{id}` |

**Update**
| `radarr_list_update` | GET | `/api/v3/update` |

**UpdateLogFile**
| `radarr_list_log_file_update` | GET | `/api/v3/log/file/update` |

Endpoints excluded on purpose: `/login`, `/logout`, `/api` (swagger metadata),
static web routes, the `.ics` calendar feed, and binary/text content endpoints
(media covers, raw log file contents) — `_req` JSON-decodes every response.

For POST/PUT tools, the request body is passed as a single `body` object (or
`body` list for array-bodied endpoints); query/path params are explicit
arguments. Optional params are omitted when unset so the API's defaults apply.

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
