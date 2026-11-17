# memtomem-memory: B3 static correctness review

Snapshot `465e1abdf5285b2ebdce15e7c8fc6dd9d1a55601` (MANIFEST.json row `memtomem-memory`). No plugin execution or passing-test claims.

## 1. Architecture

`provider` layered over Hermes's existing MCP client. It calls mem_search on configured memtomem MCP connection, with trust/allowlist checks; it does not start its own duplicate MCP client (`sources/memtomem-memory/packages/memtomem-hermes-memory/__init__.py:285`; `sources/memtomem-memory/packages/memtomem-hermes-memory/__init__.py:292`). Engine may run as local MCP subprocess or configured service.

Hook/body inventory (no-op/fallback declarations are not active behavior):

| callable | declaration | first executable body line |
|---|---|---|
| `register` | `sources/memtomem-memory/packages/memtomem-hermes-memory/__init__.py:1511` | `sources/memtomem-memory/packages/memtomem-hermes-memory/__init__.py:1513` |
| `is_available` | `sources/memtomem-memory/packages/memtomem-hermes-memory/__init__.py:1093` | `sources/memtomem-memory/packages/memtomem-hermes-memory/__init__.py:1094` |
| `initialize` | `sources/memtomem-memory/packages/memtomem-hermes-memory/__init__.py:1136` | `sources/memtomem-memory/packages/memtomem-hermes-memory/__init__.py:1137` |
| `shutdown` | `sources/memtomem-memory/packages/memtomem-hermes-memory/__init__.py:1181` | `sources/memtomem-memory/packages/memtomem-hermes-memory/__init__.py:1184` |
| `on_session_switch` | `sources/memtomem-memory/packages/memtomem-hermes-memory/__init__.py:1198` | `sources/memtomem-memory/packages/memtomem-hermes-memory/__init__.py:1207` |
| `on_session_end` | `sources/memtomem-memory/packages/memtomem-hermes-memory/__init__.py:1213` | `sources/memtomem-memory/packages/memtomem-hermes-memory/__init__.py:1214` |
| `prefetch` | `sources/memtomem-memory/packages/memtomem-hermes-memory/__init__.py:1218` | `sources/memtomem-memory/packages/memtomem-hermes-memory/__init__.py:1219` |
| `queue_prefetch` | `sources/memtomem-memory/packages/memtomem-hermes-memory/__init__.py:1318` | `sources/memtomem-memory/packages/memtomem-hermes-memory/__init__.py:1319` |
| `on_memory_write` | `sources/memtomem-memory/packages/memtomem-hermes-memory/__init__.py:1327` | `sources/memtomem-memory/packages/memtomem-hermes-memory/__init__.py:1341` |
| `system_prompt_block` | `sources/memtomem-memory/packages/memtomem-hermes-memory/__init__.py:1468` | `sources/memtomem-memory/packages/memtomem-hermes-memory/__init__.py:1469` |
| `get_tool_schemas` | `sources/memtomem-memory/packages/memtomem-hermes-memory/__init__.py:1471` | `sources/memtomem-memory/packages/memtomem-hermes-memory/__init__.py:1472` |
| `get_config_schema` | `sources/memtomem-memory/packages/memtomem-hermes-memory/__init__.py:1474` | `sources/memtomem-memory/packages/memtomem-hermes-memory/__init__.py:1475` |

## 2. Forget paths and derived copies

`none` in this provider's tool schema; engine `mem_delete` exists on a separately exposed MCP server (`sources/memtomem-memory/packages/memtomem-hermes-memory/__init__.py:1471`; `sources/memtomem-memory/packages/memtomem/src/memtomem/server/tools/memory_crud.py:1641`). Chunk-ID path removes that line span from source then reindexes (`sources/memtomem-memory/packages/memtomem/src/memtomem/server/tools/memory_crud.py:1722`). source_file/namespace paths delete indexed chunks rather than the file itself; project_shared needs confirmation (`sources/memtomem-memory/packages/memtomem/src/memtomem/server/tools/memory_crud.py:1740`; `sources/memtomem-memory/packages/memtomem/src/memtomem/server/tools/memory_crud.py:1818`). `partial`: index-only delete can leave reindexable originals; summaries/vector/FTS/relations have separate storage cascades shown below. Provider optional host-write mirroring is not host history deletion.

Host state.db is independent (`hermes-agent/hermes_state.py:181`). No reviewed adapter forget body performs a general purge of that database or built-in MEMORY.md/USER.md. Prompt/tool-result copies and independent source transcripts remain separate; source operation bodies below establish the scope of each plugin mutation.

<details>
<summary>Deletion, invalidation, archive and reset body ledger</summary>

Public routes, SDK/CLI dispatch and internal storage operations are distinguished by their source. Cache invalidation and management-object deletion are not additional fact-forget tools. A stub is not working deletion. Call-through means the downstream body still determines erasure. HTTP prefixes/selectors must be read with route registration.

**_delete_released_pid_file**, body `sources/memtomem-memory/packages/memtomem/src/memtomem/cli/_liveness.py:604`; implementation/call-through.

- `sources/memtomem-memory/packages/memtomem/src/memtomem/cli/_liveness.py:598` `def _delete_released_pid_file(pid_file: Path) -> StalePidUnlink:`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/cli/_liveness.py:605` `os.unlink(pid_file)`

**version_delete_label_cmd**, body `sources/memtomem-memory/packages/memtomem/src/memtomem/cli/context_cmd.py:1913`; implementation/call-through.

- `sources/memtomem-memory/packages/memtomem/src/memtomem/cli/context_cmd.py:1895` `@version_group.command("delete-label")`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/cli/context_cmd.py:1896` `@click.argument("artifact_type")`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/cli/context_cmd.py:1897` `@click.argument("name")`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/cli/context_cmd.py:1898` `@click.argument("label")`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/cli/context_cmd.py:1899` `@_SCOPE_OPTION`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/cli/context_cmd.py:1900` `def version_delete_label_cmd(`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/cli/context_cmd.py:1927` `versioning.delete_label(artifact_dir, label, lock_timeout=lt)`

**_delete**, body `sources/memtomem-memory/packages/memtomem/src/memtomem/cli/context_cmd.py:1927`; implementation/call-through.

- `sources/memtomem-memory/packages/memtomem/src/memtomem/cli/context_cmd.py:1926` `def _delete(lt: float | None) -> versioning.VersionsManifest:`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/cli/context_cmd.py:1927` `versioning.delete_label(artifact_dir, label, lock_timeout=lt)`

**_archived_line**, body `sources/memtomem-memory/packages/memtomem/src/memtomem/cli/gc_cmd.py:117`; implementation/call-through.

- `sources/memtomem-memory/packages/memtomem/src/memtomem/cli/gc_cmd.py:116` `def _archived_line(count: int) -> str:`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/cli/gc_cmd.py:117` `return f"{count} archived source(s) under retain_missing_dirs are kept."`

**_archived_chunks_line**, body `sources/memtomem-memory/packages/memtomem/src/memtomem/cli/gc_cmd.py:121`; implementation/call-through.

- `sources/memtomem-memory/packages/memtomem/src/memtomem/cli/gc_cmd.py:120` `def _archived_chunks_line(count: int) -> str:`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/cli/gc_cmd.py:121` `return f"{count} archived chunk(s) under retain_missing_dirs are kept."`

**delete_block**, body `sources/memtomem-memory/packages/memtomem/src/memtomem/cli/pinned_cmd.py:193`; implementation/call-through.

- `sources/memtomem-memory/packages/memtomem/src/memtomem/cli/pinned_cmd.py:181` `@pinned.command("delete")`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/cli/pinned_cmd.py:182` `@click.argument("block_id")`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/cli/pinned_cmd.py:183` `@click.option("--scope", type=click.Choice(_SCOPES), default="user")`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/cli/pinned_cmd.py:184` `@_AGENT_ID_OPTION`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/cli/pinned_cmd.py:185` `@click.option("--confirm-project-shared", is_flag=True)`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/cli/pinned_cmd.py:186` `def delete_block(`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/cli/pinned_cmd.py:193` `from memtomem.cli._errors import raise_cli_error`

**_delete_block**, body `sources/memtomem-memory/packages/memtomem/src/memtomem/cli/pinned_cmd.py:207`; implementation/call-through.

- `sources/memtomem-memory/packages/memtomem/src/memtomem/cli/pinned_cmd.py:201` `async def _delete_block(`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/cli/pinned_cmd.py:212` `deleted = store.delete(`

**classify_sources_for_purge**, body `sources/memtomem-memory/packages/memtomem/src/memtomem/cli/purge_cmd.py:60`; implementation/call-through.

- `sources/memtomem-memory/packages/memtomem/src/memtomem/cli/purge_cmd.py:47` `def classify_sources_for_purge(`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/cli/purge_cmd.py:60` `from memtomem.indexing.engine import WorktreeMemo, _build_exclude_spec, _path_is_excluded`

**purge**, body `sources/memtomem-memory/packages/memtomem/src/memtomem/cli/purge_cmd.py:156`; implementation/call-through.

- `sources/memtomem-memory/packages/memtomem/src/memtomem/cli/purge_cmd.py:113` `@click.command("purge")`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/cli/purge_cmd.py:114` `@click.option(`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/cli/purge_cmd.py:115` `"--matching-excluded",`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/cli/purge_cmd.py:116` `"matching_excluded",`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/cli/purge_cmd.py:117` `is_flag=True,`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/cli/purge_cmd.py:118` `help=(`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/cli/purge_cmd.py:119` `"Target chunks whose source_path matches built-in denylist, "`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/cli/purge_cmd.py:120` `"indexing.exclude_patterns, a provider index-file convention "`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/cli/purge_cmd.py:121` `"(e.g. claude-memory MEMORY.md/README.md), or a nested git worktree."`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/cli/purge_cmd.py:122` `),`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/cli/purge_cmd.py:123` `)`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/cli/purge_cmd.py:124` `@click.option(`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/cli/purge_cmd.py:125` `"--apply",`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/cli/purge_cmd.py:126` `"apply_",`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/cli/purge_cmd.py:127` `is_flag=True,`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/cli/purge_cmd.py:128` `help="Actually delete. Without this flag, prints what would be deleted (dry-run).",`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/cli/purge_cmd.py:129` `)`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/cli/purge_cmd.py:130` `@click.option(`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/cli/purge_cmd.py:131` `"--sample",`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/cli/purge_cmd.py:132` `"sample_size",`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/cli/purge_cmd.py:133` `default=5,`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/cli/purge_cmd.py:134` `show_default=True,`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/cli/purge_cmd.py:135` `help="Number of sample paths to print in dry-run output.",`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/cli/purge_cmd.py:136` `)`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/cli/purge_cmd.py:137` `@click.option(`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/cli/purge_cmd.py:138` `"--json",`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/cli/purge_cmd.py:139` `"as_json",`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/cli/purge_cmd.py:140` `is_flag=True,`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/cli/purge_cmd.py:141` `help="Emit a machine-readable JSON ack instead of text output.",`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/cli/purge_cmd.py:142` `)`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/cli/purge_cmd.py:143` `def purge(matching_excluded: bool, apply_: bool, sample_size: int, as_json: bool) -> None:`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/cli/purge_cmd.py:156` `if not matching_excluded:`

**reset**, body `sources/memtomem-memory/packages/memtomem/src/memtomem/cli/reset_cmd.py:103`; implementation/call-through.

- `sources/memtomem-memory/packages/memtomem/src/memtomem/cli/reset_cmd.py:76` `@click.command("reset")`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/cli/reset_cmd.py:77` `@click.option("--yes", "-y", is_flag=True, help="Skip confirmation prompt")`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/cli/reset_cmd.py:78` `@click.option(`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/cli/reset_cmd.py:79` `"--backup",`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/cli/reset_cmd.py:80` `is_flag=True,`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/cli/reset_cmd.py:81` `help="Snapshot the DB to <db>.pre-reset-<timestamp>.bak before wiping.",`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/cli/reset_cmd.py:82` `)`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/cli/reset_cmd.py:83` `@click.option(`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/cli/reset_cmd.py:84` `"--force",`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/cli/reset_cmd.py:85` `is_flag=True,`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/cli/reset_cmd.py:86` `help="Bypass the stale-pid/db-lock safety heuristics (use only if you know "`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/cli/reset_cmd.py:87` `"the pid or lock is stale). Does NOT override instance-registry evidence "`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/cli/reset_cmd.py:88` `"of a live server or Web UI, or a held lifecycle barrier , those are positive "`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/cli/reset_cmd.py:89` `"liveness, not heuristics.",`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/cli/reset_cmd.py:90` `)`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/cli/reset_cmd.py:91` `@click.option(`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/cli/reset_cmd.py:92` `"--json",`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/cli/reset_cmd.py:93` `"as_json",`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/cli/reset_cmd.py:94` `is_flag=True,`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/cli/reset_cmd.py:95` `help="Emit a machine-readable JSON ack instead of text output.",`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/cli/reset_cmd.py:96` `)`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/cli/reset_cmd.py:97` `def reset(yes: bool, backup: bool, force: bool, as_json: bool) -> None:`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/cli/reset_cmd.py:103` `asyncio.run(_run(yes, backup=backup, force=force, as_json=as_json))`

**schedule_delete**, body `sources/memtomem-memory/packages/memtomem/src/memtomem/cli/schedule_cmd.py:113`; implementation/call-through.

- `sources/memtomem-memory/packages/memtomem/src/memtomem/cli/schedule_cmd.py:109` `@schedule.command("delete")`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/cli/schedule_cmd.py:110` `@click.argument("sched_id")`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/cli/schedule_cmd.py:111` `def schedule_delete(sched_id: str) -> None:`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/cli/schedule_cmd.py:113` `from memtomem.cli._errors import raise_cli_error`

**_delete**, body `sources/memtomem-memory/packages/memtomem/src/memtomem/cli/schedule_cmd.py:142`; implementation/call-through.

- `sources/memtomem-memory/packages/memtomem/src/memtomem/cli/schedule_cmd.py:141` `async def _delete(sched_id: str) -> bool:`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/cli/schedule_cmd.py:142` `from memtomem.cli._bootstrap import cli_components`

**delete**, body `sources/memtomem-memory/packages/memtomem/src/memtomem/cli/tags_cmd.py:147`; implementation/call-through.

- `sources/memtomem-memory/packages/memtomem/src/memtomem/cli/tags_cmd.py:131` `@tags.command("delete")`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/cli/tags_cmd.py:132` `@click.argument("name")`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/cli/tags_cmd.py:133` `@click.option(`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/cli/tags_cmd.py:134` `"--apply",`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/cli/tags_cmd.py:135` `"apply_",`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/cli/tags_cmd.py:136` `is_flag=True,`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/cli/tags_cmd.py:137` `help="Actually write. Without this flag, prints what would change (dry-run).",`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/cli/tags_cmd.py:138` `)`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/cli/tags_cmd.py:139` `@click.option(`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/cli/tags_cmd.py:140` `"--yes",`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/cli/tags_cmd.py:141` `"assume_yes",`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/cli/tags_cmd.py:142` `is_flag=True,`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/cli/tags_cmd.py:143` `help="Skip the confirmation prompt. Requires --apply.",`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/cli/tags_cmd.py:144` `)`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/cli/tags_cmd.py:145` `def delete(name: str, apply_: bool, assume_yes: bool) -> None:`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/cli/tags_cmd.py:147` `if assume_yes and not apply_:`

**_run_delete**, body `sources/memtomem-memory/packages/memtomem/src/memtomem/cli/tags_cmd.py:153`; implementation/call-through.

- `sources/memtomem-memory/packages/memtomem/src/memtomem/cli/tags_cmd.py:152` `async def _run_delete(name: str, *, apply_: bool, assume_yes: bool) -> None:`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/cli/tags_cmd.py:157` `preview = await tag_svc.delete_tag(comp.storage, name, dry_run=True)`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/cli/tags_cmd.py:178` `result = await tag_svc.delete_tag(`

**_delete_inventory**, body `sources/memtomem-memory/packages/memtomem/src/memtomem/cli/uninstall_cmd.py:1139`; implementation/call-through.

- `sources/memtomem-memory/packages/memtomem/src/memtomem/cli/uninstall_cmd.py:1130` `def _delete_inventory(inv: _Inventory, *, keep_config: bool, keep_data: bool) -> str:`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/cli/uninstall_cmd.py:1139` `staging_roots, completed = _stage_inventory(inv, keep_config=keep_config, keep_data=keep_data)`

**delete_label**, body `sources/memtomem-memory/packages/memtomem/src/memtomem/context/versioning.py:872`; implementation/call-through.

- `sources/memtomem-memory/packages/memtomem/src/memtomem/context/versioning.py:868` `def delete_label(artifact_dir: Path, label: str, *, lock_timeout: float | None = None) -> None:`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/context/versioning.py:872` `if label in RESERVED_LABELS:`

**_delete_missing_source**, body `sources/memtomem-memory/packages/memtomem/src/memtomem/indexing/engine.py:2226`; implementation/call-through.

- `sources/memtomem-memory/packages/memtomem/src/memtomem/indexing/engine.py:2207` `async def _delete_missing_source(`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/indexing/engine.py:2226` `if absent and self._config.retain_missing_dirs:`

**on_deleted**, body `sources/memtomem-memory/packages/memtomem/src/memtomem/indexing/watcher.py:224`; implementation/call-through.

- `sources/memtomem-memory/packages/memtomem/src/memtomem/indexing/watcher.py:220` `def on_deleted(self, event: FileSystemEvent) -> None:`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/indexing/watcher.py:224` `if not event.is_directory:`

**_invalidate_if_mutated**, body `sources/memtomem-memory/packages/memtomem/src/memtomem/indexing/watcher.py:344`; implementation/call-through.

- `sources/memtomem-memory/packages/memtomem/src/memtomem/indexing/watcher.py:335` `def _invalidate_if_mutated(self, stats: IndexingStats) -> None:`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/indexing/watcher.py:345` `self._search_pipeline.invalidate_cache()`

**_forget_probe**, body `sources/memtomem-memory/packages/memtomem/src/memtomem/indexing/watcher.py:647`; implementation/call-through.

- `sources/memtomem-memory/packages/memtomem/src/memtomem/indexing/watcher.py:646` `def _forget_probe(self, probe: asyncio.Future[tuple[str, bool]]) -> None:`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/indexing/watcher.py:647` `self._stalled_probes.discard(probe)`

**delete**, body `sources/memtomem-memory/packages/memtomem/src/memtomem/integrations/langgraph.py:814`; implementation/call-through.

- `sources/memtomem-memory/packages/memtomem/src/memtomem/integrations/langgraph.py:764` `async def delete(self, chunk_id: str, *, confirm_project_shared: bool = False) -> bool:`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/integrations/langgraph.py:884` `action="delete",`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/integrations/langgraph.py:893` `deleted = await comp.storage.delete_chunks([uid])`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/integrations/langgraph.py:896` `# lifetime, so an un-invalidated delete stays visible to`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/integrations/langgraph.py:900` `comp.search_pipeline.invalidate_cache()`

**delete**, body `sources/memtomem-memory/packages/memtomem/src/memtomem/pinned.py:307`; implementation/call-through.

- `sources/memtomem-memory/packages/memtomem/src/memtomem/pinned.py:297` `def delete(`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/pinned.py:324` `action="delete",`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/pinned.py:330` `path.unlink(missing_ok=True)`

**invalidate_cache**, body `sources/memtomem-memory/packages/memtomem/src/memtomem/search/pipeline.py:1036`; implementation/call-through.

- `sources/memtomem-memory/packages/memtomem/src/memtomem/search/pipeline.py:1034` `def invalidate_cache(self) -> None:`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/search/pipeline.py:1036` `self._cache_version += 1`

**_delete**, body `sources/memtomem-memory/packages/memtomem/src/memtomem/server/tools/context.py:2844`; implementation/call-through.

- `sources/memtomem-memory/packages/memtomem/src/memtomem/server/tools/context.py:2843` `def _delete(lt: float | None) -> "VersionsManifest":`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/server/tools/context.py:2844` `versioning.delete_label(artifact_dir, label, lock_timeout=lt)`

**mem_delete**, body `sources/memtomem-memory/packages/memtomem/src/memtomem/server/tools/memory_crud.py:1665`; implementation/call-through.

- `sources/memtomem-memory/packages/memtomem/src/memtomem/server/tools/memory_crud.py:1638` `@mcp.tool()`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/server/tools/memory_crud.py:1639` `@tool_handler`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/server/tools/memory_crud.py:1640` `@register("crud")`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/server/tools/memory_crud.py:1641` `async def mem_delete(`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/server/tools/memory_crud.py:1714` `action="delete",`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/server/tools/memory_crud.py:1729` `op="delete",`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/server/tools/memory_crud.py:1738` `) + _rewrite_notices("delete", stats, meta.source_file)`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/server/tools/memory_crud.py:1834` `action="delete",`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/server/tools/memory_crud.py:1837` `deleted = await app.storage.delete_by_source(stored_key, as_stored=True)`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/server/tools/memory_crud.py:1842` `app.search_pipeline.invalidate_cache()`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/server/tools/memory_crud.py:1894` `action="delete",`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/server/tools/memory_crud.py:1897` `deleted = await app.storage.delete_by_namespace(namespace)`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/server/tools/memory_crud.py:1898` `app.search_pipeline.invalidate_cache()`

**mem_ns_delete**, body `sources/memtomem-memory/packages/memtomem/src/memtomem/server/tools/namespace.py:57`; implementation/call-through.

- `sources/memtomem-memory/packages/memtomem/src/memtomem/server/tools/namespace.py:43` `@mcp.tool()`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/server/tools/namespace.py:44` `@tool_handler`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/server/tools/namespace.py:45` `@register("namespace")`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/server/tools/namespace.py:46` `async def mem_ns_delete(`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/server/tools/namespace.py:62` `deleted = await app.storage.delete_by_namespace(namespace)`

**mem_pinned_delete**, body `sources/memtomem-memory/packages/memtomem/src/memtomem/server/tools/pinned.py:147`; implementation/call-through.

- `sources/memtomem-memory/packages/memtomem/src/memtomem/server/tools/pinned.py:123` `@mcp.tool()`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/server/tools/pinned.py:124` `@tool_handler`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/server/tools/pinned.py:125` `@register("pinned")`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/server/tools/pinned.py:126` `async def mem_pinned_delete(`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/server/tools/pinned.py:150` `"deleted": store.delete(`

**mem_policy_delete**, body `sources/memtomem-memory/packages/memtomem/src/memtomem/server/tools/policy.py:287`; implementation/call-through.

- `sources/memtomem-memory/packages/memtomem/src/memtomem/server/tools/policy.py:275` `@mcp.tool()`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/server/tools/policy.py:276` `@tool_handler`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/server/tools/policy.py:277` `@register("policy")`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/server/tools/policy.py:278` `async def mem_policy_delete(`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/server/tools/policy.py:287` `app = await _get_app_initialized(ctx)`

**mem_schedule_delete**, body `sources/memtomem-memory/packages/memtomem/src/memtomem/server/tools/schedule.py:165`; implementation/call-through.

- `sources/memtomem-memory/packages/memtomem/src/memtomem/server/tools/schedule.py:152` `@mcp.tool()`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/server/tools/schedule.py:153` `@tool_handler`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/server/tools/schedule.py:154` `@register("schedule")`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/server/tools/schedule.py:155` `async def mem_schedule_delete(id: str, ctx: CtxType = None) -> str:`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/server/tools/schedule.py:165` `app = await _get_app_initialized(ctx)`

**_prepare_summary_archive**, body `sources/memtomem-memory/packages/memtomem/src/memtomem/server/tools/session.py:848`; implementation/call-through.

- `sources/memtomem-memory/packages/memtomem/src/memtomem/server/tools/session.py:826` `def _prepare_summary_archive(`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/server/tools/session.py:855` `namespace = f"archive:session:{session_id}"`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/server/tools/session.py:871` `logger.warning("session_summary_archive_declined session_id=%s reason=%s", session_id, exc)`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/server/tools/session.py:884` `return _SummaryArchive(content=content, target=target, namespace=namespace)`

**_write_summary_archive**, body `sources/memtomem-memory/packages/memtomem/src/memtomem/server/tools/session.py:967`; implementation/call-through.

- `sources/memtomem-memory/packages/memtomem/src/memtomem/server/tools/session.py:947` `async def _write_summary_archive(`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/server/tools/session.py:970` `# #2488: an excluded archive path would be written and indexed to zero chunks,`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/server/tools/session.py:987` `app.search_pipeline.invalidate_cache()`

**_count_archived**, body `sources/memtomem-memory/packages/memtomem/src/memtomem/server/tools/status_config.py:416`; implementation/call-through.

- `sources/memtomem-memory/packages/memtomem/src/memtomem/server/tools/status_config.py:415` `def _count_archived() -> int:`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/server/tools/status_config.py:416` `rule = RetainRule.prepare(config.indexing)`

**mem_tag_delete**, body `sources/memtomem-memory/packages/memtomem/src/memtomem/server/tools/tag_management.py:132`; implementation/call-through.

- `sources/memtomem-memory/packages/memtomem/src/memtomem/server/tools/tag_management.py:114` `@mcp.tool()`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/server/tools/tag_management.py:115` `@tool_handler`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/server/tools/tag_management.py:116` `@register("tags")`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/server/tools/tag_management.py:117` `async def mem_tag_delete(`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/server/tools/tag_management.py:140` `result = await tag_svc.delete_tag(`

**_invalidate**, body `sources/memtomem-memory/packages/memtomem/src/memtomem/services/tag_management.py:92`; implementation/call-through.

- `sources/memtomem-memory/packages/memtomem/src/memtomem/services/tag_management.py:91` `def _invalidate(search_pipeline: SearchPipeline | None) -> None:`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/services/tag_management.py:93` `search_pipeline.invalidate_cache()`

**delete_tag**, body `sources/memtomem-memory/packages/memtomem/src/memtomem/services/tag_management.py:150`; implementation/call-through.

- `sources/memtomem-memory/packages/memtomem/src/memtomem/services/tag_management.py:137` `async def delete_tag(`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/services/tag_management.py:160` `affected = await storage.delete_tag(tag)`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/services/tag_management.py:162` `_invalidate(search_pipeline)`

**delete_chunks**, body `sources/memtomem-memory/packages/memtomem/src/memtomem/storage/base.py:223`; stub.

- `sources/memtomem-memory/packages/memtomem/src/memtomem/storage/base.py:221` `async def delete_chunks(`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/storage/base.py:223` `) -> int: ...`

**delete_by_source**, body `sources/memtomem-memory/packages/memtomem/src/memtomem/storage/base.py:228`; stub.

- `sources/memtomem-memory/packages/memtomem/src/memtomem/storage/base.py:228` `async def delete_by_source(self, source_file: Path, *, as_stored: bool = False) -> int: ...`

**delete_by_namespace**, body `sources/memtomem-memory/packages/memtomem/src/memtomem/storage/base.py:450`; stub.

- `sources/memtomem-memory/packages/memtomem/src/memtomem/storage/base.py:450` `async def delete_by_namespace(self, namespace: str) -> int: ...`

**delete_tag**, body `sources/memtomem-memory/packages/memtomem/src/memtomem/storage/base.py:479`; stub.

- `sources/memtomem-memory/packages/memtomem/src/memtomem/storage/base.py:479` `async def delete_tag(self, tag: str) -> int: ...`

**delete_relation**, body `sources/memtomem-memory/packages/memtomem/src/memtomem/storage/base.py:515`; stub.

- `sources/memtomem-memory/packages/memtomem/src/memtomem/storage/base.py:515` `async def delete_relation(self, source_id: UUID, target_id: UUID) -> bool: ...`

**scratch_delete**, body `sources/memtomem-memory/packages/memtomem/src/memtomem/storage/base.py:574`; stub.

- `sources/memtomem-memory/packages/memtomem/src/memtomem/storage/base.py:574` `async def scratch_delete(self, key: str, *, expected_value: str | None = None) -> bool: ...`

**delete_entities_for_chunk**, body `sources/memtomem-memory/packages/memtomem/src/memtomem/storage/mixins/entities.py:127`; implementation/call-through.

- `sources/memtomem-memory/packages/memtomem/src/memtomem/storage/mixins/entities.py:126` `async def delete_entities_for_chunk(self, chunk_id: str) -> int:`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/storage/mixins/entities.py:132` `cur = db.execute("DELETE FROM chunk_entities WHERE chunk_id = ?", (chunk_id,))`

**hermes_mirror_purge**, body `sources/memtomem-memory/packages/memtomem/src/memtomem/storage/mixins/hermes_mirror.py:139`; implementation/call-through.

- `sources/memtomem-memory/packages/memtomem/src/memtomem/storage/mixins/hermes_mirror.py:129` `async def hermes_mirror_purge(self, owner: str, target: str, ts: int) -> dict[str, Any]:`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/storage/mixins/hermes_mirror.py:152` `"DELETE FROM hermes_mirror_entries WHERE owner = ? AND target = ? AND ts <= ?",`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/storage/mixins/hermes_mirror.py:163` `"purged_at = excluded.purged_at, updated_at = excluded.updated_at",`

**_purge_expired**, body `sources/memtomem-memory/packages/memtomem/src/memtomem/storage/mixins/idempotency.py:45`; implementation/call-through.

- `sources/memtomem-memory/packages/memtomem/src/memtomem/storage/mixins/idempotency.py:44` `def _purge_expired(self, db) -> None:`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/storage/mixins/idempotency.py:45` `db.execute("DELETE FROM idempotency_ledger WHERE expires_at < ?", (self._now_iso(),))`

**policy_delete**, body `sources/memtomem-memory/packages/memtomem/src/memtomem/storage/mixins/policies.py:73`; implementation/call-through.

- `sources/memtomem-memory/packages/memtomem/src/memtomem/storage/mixins/policies.py:72` `async def policy_delete(self, name: str) -> bool:`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/storage/mixins/policies.py:75` `cur = db.execute("DELETE FROM memory_policies WHERE name = ?", (name,))`

**delete_relation**, body `sources/memtomem-memory/packages/memtomem/src/memtomem/storage/mixins/relations.py:185`; implementation/call-through.

- `sources/memtomem-memory/packages/memtomem/src/memtomem/storage/mixins/relations.py:184` `async def delete_relation(self, source_id: UUID, target_id: UUID) -> bool:`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/storage/mixins/relations.py:188` `"DELETE FROM chunk_relations WHERE (source_id = ? AND target_id = ?) OR (source_id = ? AND target_id = ?)",`

**delete_tag**, body `sources/memtomem-memory/packages/memtomem/src/memtomem/storage/mixins/relations.py:244`; implementation/call-through.

- `sources/memtomem-memory/packages/memtomem/src/memtomem/storage/mixins/relations.py:241` `async def delete_tag(self, tag: str) -> int:`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/storage/mixins/relations.py:244` `db = self._get_db()`

**schedule_delete**, body `sources/memtomem-memory/packages/memtomem/src/memtomem/storage/mixins/schedules.py:169`; implementation/call-through.

- `sources/memtomem-memory/packages/memtomem/src/memtomem/storage/mixins/schedules.py:168` `async def schedule_delete(self, sched_id: str) -> bool:`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/storage/mixins/schedules.py:171` `cur = db.execute("DELETE FROM schedules WHERE id=?", (sched_id,))`

**scratch_delete**, body `sources/memtomem-memory/packages/memtomem/src/memtomem/storage/mixins/scratch.py:74`; implementation/call-through.

- `sources/memtomem-memory/packages/memtomem/src/memtomem/storage/mixins/scratch.py:67` `async def scratch_delete(self, key: str, *, expected_value: str | None = None) -> bool:`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/storage/mixins/scratch.py:77` `cursor = db.execute("DELETE FROM working_memory WHERE key = ?", (key,))`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/storage/mixins/scratch.py:80` `"DELETE FROM working_memory WHERE key = ? AND value = ?",`

**delete_dangling_chunk_links**, body `sources/memtomem-memory/packages/memtomem/src/memtomem/storage/mixins/share_links.py:112`; implementation/call-through.

- `sources/memtomem-memory/packages/memtomem/src/memtomem/storage/mixins/share_links.py:104` `async def delete_dangling_chunk_links(self) -> int:`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/storage/mixins/share_links.py:114` `cur = db.execute("DELETE FROM chunk_links WHERE source_id IS NULL")`

**delete_chunks**, body `sources/memtomem-memory/packages/memtomem/src/memtomem/storage/sqlite_backend.py:2394`; implementation/call-through.

- `sources/memtomem-memory/packages/memtomem/src/memtomem/storage/sqlite_backend.py:2384` `async def delete_chunks(`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/storage/sqlite_backend.py:2440` `f"DELETE FROM chunks WHERE id IN ({placeholders(len(found_ids))})", found_ids`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/storage/sqlite_backend.py:2443` `f"DELETE FROM chunks_fts WHERE rowid IN ({placeholders(len(rowids))})", rowids`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/storage/sqlite_backend.py:2447` `f"DELETE FROM chunks_vec WHERE rowid IN ({placeholders(len(rowids))})",`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/storage/sqlite_backend.py:2488` `"DELETE FROM _memtomem_meta WHERE key IN (?, ?)",`

**_delete_policy_summaries_for_source**, body `sources/memtomem-memory/packages/memtomem/src/memtomem/storage/sqlite_backend.py:2526`; implementation/call-through.

- `sources/memtomem-memory/packages/memtomem/src/memtomem/storage/sqlite_backend.py:2517` `def _delete_policy_summaries_for_source(self, db: sqlite3.Connection, source: str) -> int:`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/storage/sqlite_backend.py:2536` `db.execute(f"DELETE FROM chunks WHERE id IN ({placeholders(len(ids))})", ids)`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/storage/sqlite_backend.py:2538` `f"DELETE FROM chunks_fts WHERE rowid IN ({placeholders(len(rowids))})", rowids`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/storage/sqlite_backend.py:2542` `f"DELETE FROM chunks_vec WHERE rowid IN ({placeholders(len(rowids))})", rowids`

**delete_by_source**, body `sources/memtomem-memory/packages/memtomem/src/memtomem/storage/sqlite_backend.py:2563`; implementation/call-through.

- `sources/memtomem-memory/packages/memtomem/src/memtomem/storage/sqlite_backend.py:2554` `async def delete_by_source(self, source_file: Path, *, as_stored: bool = False) -> int:`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/storage/sqlite_backend.py:2578` `"DELETE FROM _memtomem_meta WHERE key IN (?, ?)",`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/storage/sqlite_backend.py:2585` `"DELETE FROM source_index_receipts WHERE source_file=?",`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/storage/sqlite_backend.py:2609` `db.execute(f"DELETE FROM chunks WHERE id IN ({placeholders(len(ids))})", ids)`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/storage/sqlite_backend.py:2611` `f"DELETE FROM chunks_fts WHERE rowid IN ({placeholders(len(rowids))})", rowids`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/storage/sqlite_backend.py:2615` `f"DELETE FROM chunks_vec WHERE rowid IN ({placeholders(len(rowids))})",`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/storage/sqlite_backend.py:2619` `"DELETE FROM _memtomem_meta WHERE key IN (?, ?)",`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/storage/sqlite_backend.py:2631` `"DELETE FROM source_index_receipts WHERE source_file=?",`

**delete_ai_summary**, body `sources/memtomem-memory/packages/memtomem/src/memtomem/storage/sqlite_backend.py:3781`; implementation/call-through.

- `sources/memtomem-memory/packages/memtomem/src/memtomem/storage/sqlite_backend.py:3771` `async def delete_ai_summary(self, source_file: Path, *, as_stored: bool = False) -> None:`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/storage/sqlite_backend.py:3784` `"DELETE FROM _memtomem_meta WHERE key=?",`

**delete_by_namespace**, body `sources/memtomem-memory/packages/memtomem/src/memtomem/storage/sqlite_backend.py:3959`; implementation/call-through.

- `sources/memtomem-memory/packages/memtomem/src/memtomem/storage/sqlite_backend.py:3958` `async def delete_by_namespace(self, namespace: str) -> int:`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/storage/sqlite_backend.py:3970` `deleted = await ns.delete_by_namespace(namespace)`

**_delete_and_clear_empty_sources**, body `sources/memtomem-memory/packages/memtomem/src/memtomem/storage/sqlite_backend.py:3963`; implementation/call-through.

- `sources/memtomem-memory/packages/memtomem/src/memtomem/storage/sqlite_backend.py:3962` `async def _delete_and_clear_empty_sources() -> int:`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/storage/sqlite_backend.py:3970` `deleted = await ns.delete_by_namespace(namespace)`

**delete_by_namespace**, body `sources/memtomem-memory/packages/memtomem/src/memtomem/storage/sqlite_namespace.py:294`; implementation/call-through.

- `sources/memtomem-memory/packages/memtomem/src/memtomem/storage/sqlite_namespace.py:293` `async def delete_by_namespace(self, namespace: str) -> int:`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/storage/sqlite_namespace.py:323` `db.execute("DELETE FROM namespace_metadata WHERE namespace=?", (namespace,))`

**_delete_chunk_rows**, body `sources/memtomem-memory/packages/memtomem/src/memtomem/storage/sqlite_namespace.py:710`; implementation/call-through.

- `sources/memtomem-memory/packages/memtomem/src/memtomem/storage/sqlite_namespace.py:690` `def _delete_chunk_rows(`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/storage/sqlite_namespace.py:714` `f"DELETE FROM chunks_fts WHERE rowid IN ({placeholders(len(batch_rowids))})",`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/storage/sqlite_namespace.py:719` `f"DELETE FROM chunks_vec WHERE rowid IN ({placeholders(len(batch_rowids))})",`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/storage/sqlite_namespace.py:723` `f"DELETE FROM chunks WHERE id IN ({placeholders(len(batch_ids))})", batch_ids`

**_resolve_archive_ns**, body `sources/memtomem-memory/packages/memtomem/src/memtomem/tools/policy_engine.py:93`; implementation/call-through.

- `sources/memtomem-memory/packages/memtomem/src/memtomem/tools/policy_engine.py:85` `def _resolve_archive_ns(template: str, tags_json: str | None, fallback: str) -> str:`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/tools/policy_engine.py:93` `if not template:`

**execute_auto_archive**, body `sources/memtomem-memory/packages/memtomem/src/memtomem/tools/policy_engine.py:320`; implementation/call-through.

- `sources/memtomem-memory/packages/memtomem/src/memtomem/tools/policy_engine.py:282` `async def execute_auto_archive(`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/tools/policy_engine.py:321` `archive_ns = config.get("archive_namespace", "archive")`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/tools/policy_engine.py:325` `ns_template = config.get("archive_namespace_template")`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/tools/policy_engine.py:330` `policy_type="auto_archive",`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/tools/policy_engine.py:339` `target_errors = policy_target_errors("auto_archive", config)`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/tools/policy_engine.py:341` `return _refused_target_result("auto_archive", dry_run, target_errors)`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/tools/policy_engine.py:361` `# In the filter's namespace or already archived. Template targets are`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/tools/policy_engine.py:365` `params.extend([namespace, archive_ns])`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/tools/policy_engine.py:382` `target = archive_ns`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/tools/policy_engine.py:385` `_resolve_archive_ns(ns_template, r[3], fallback=archive_ns) for r in rows`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/tools/policy_engine.py:397` `refusal_by_target[target] = _target_refusal("archive target", target)`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/tools/policy_engine.py:400` `logger.warning("auto_archive skipped: %s", refusal_by_target[target])`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/tools/policy_engine.py:428` `verb = "Would archive" if dry_run else "Archived"`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/tools/policy_engine.py:435` `details = f"{verb} {count} chunks older than {max_age} days → '{archive_ns}'"`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/tools/policy_engine.py:439` `# to archive were not all archived. ''affected_count'' still counts what`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/tools/policy_engine.py:447` `f"{skipped} chunk(s) not archived: target namespace a search could not find "`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/tools/policy_engine.py:458` `policy_type="auto_archive",`

**delete_artifact**, body `sources/memtomem-memory/packages/memtomem/src/memtomem/web/routes/_atomic_kind.py:605`; implementation/call-through.

- `sources/memtomem-memory/packages/memtomem/src/memtomem/web/routes/_atomic_kind.py:597` `async def delete_artifact(`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/web/routes/_atomic_kind.py:684` `cur_path.unlink()`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/web/routes/_atomic_kind.py:698` `target.unlink()`

**_delete_locked**, body `sources/memtomem-memory/packages/memtomem/src/memtomem/web/routes/_atomic_kind.py:629`; implementation/call-through.

- `sources/memtomem-memory/packages/memtomem/src/memtomem/web/routes/_atomic_kind.py:628` `def _delete_locked() -> tuple[dict | None, list[str], list[dict[str, str]]]:`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/web/routes/_atomic_kind.py:684` `cur_path.unlink()`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/web/routes/_atomic_kind.py:698` `target.unlink()`

**edit_chunk**, body `sources/memtomem-memory/packages/memtomem/src/memtomem/web/routes/chunks.py:283`; implementation/call-through.

- `sources/memtomem-memory/packages/memtomem/src/memtomem/web/routes/chunks.py:274` `@router.patch("/{chunk_id}", response_model=ChunkOut | ChunkEditNeedsConfirmation)`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/web/routes/chunks.py:275` `async def edit_chunk(`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/web/routes/chunks.py:479` `search_pipeline.invalidate_cache()`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/web/routes/chunks.py:482` `# there is nothing to invalidate.`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/web/routes/chunks.py:492` `# refusal , nothing read, nothing written, nothing to invalidate.`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/web/routes/chunks.py:507` `# Normally a pure refusal. Invalidate conservatively: if this type`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/web/routes/chunks.py:509` `search_pipeline.invalidate_cache()`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/web/routes/chunks.py:518` `# Invalidate for the same reason both branches below do: the`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/web/routes/chunks.py:521` `search_pipeline.invalidate_cache()`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/web/routes/chunks.py:535` `search_pipeline.invalidate_cache()`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/web/routes/chunks.py:550` `# is itself a write (#2141). Invalidate here for the same reason`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/web/routes/chunks.py:552` `# invalidates on its rollback path.`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/web/routes/chunks.py:559` `search_pipeline.invalidate_cache()`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/web/routes/chunks.py:568` `# have moved". Mirrors the MCP twin, which invalidates on the`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/web/routes/chunks.py:570` `search_pipeline.invalidate_cache()`

**delete_chunk**, body `sources/memtomem-memory/packages/memtomem/src/memtomem/web/routes/chunks.py:615`; implementation/call-through.

- `sources/memtomem-memory/packages/memtomem/src/memtomem/web/routes/chunks.py:596` `@router.delete("/{chunk_id}", response_model=DeleteResponse)`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/web/routes/chunks.py:597` `async def delete_chunk(`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/web/routes/chunks.py:692` `await storage.delete_chunks([chunk_id], finalize_emptied=True)`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/web/routes/chunks.py:730` `action="delete",`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/web/routes/chunks.py:795` `# can then fail with a 500, so the invalidation belongs in ''finally''`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/web/routes/chunks.py:798` `# without touching disk or the store, and ''invalidate_cache'' also`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/web/routes/chunks.py:901` `str(source), op="delete", surface="web"`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/web/routes/chunks.py:923` `raise privacy_index_blocked_http(rejection, "delete")`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/web/routes/chunks.py:924` `blocker = preflight.blocker("delete", source.name, needs_valve=False)`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/web/routes/chunks.py:1033` `raise privacy_index_blocked_http(exc, "delete") from exc`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/web/routes/chunks.py:1059` `search_pipeline.invalidate_cache()`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/web/routes/chunks.py:1063` `warning = relinked_after_write_notice("delete", str(source))`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/web/routes/chunks.py:1065` `warning = scope_held_notice("delete")`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/web/routes/chunks.py:1067` `warning = excluded_after_write_notice("delete")`

**update_chunk_tags**, body `sources/memtomem-memory/packages/memtomem/src/memtomem/web/routes/chunks.py:1087`; implementation/call-through.

- `sources/memtomem-memory/packages/memtomem/src/memtomem/web/routes/chunks.py:1071` `@router.patch("/{chunk_id}/tags", response_model=TagsUpdateResponse)`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/web/routes/chunks.py:1072` `async def update_chunk_tags(`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/web/routes/chunks.py:1087` `updated = await tag_svc.replace_chunk_tags(`

**delete_agent**, body `sources/memtomem-memory/packages/memtomem/src/memtomem/web/routes/context_agents.py:261`; implementation/call-through.

- `sources/memtomem-memory/packages/memtomem/src/memtomem/web/routes/context_agents.py:240` `@router.delete("/context/agents/{name}")`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/web/routes/context_agents.py:241` `async def delete_agent(`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/web/routes/context_agents.py:261` `return await _atomic_kind.delete_artifact(`

**delete_command**, body `sources/memtomem-memory/packages/memtomem/src/memtomem/web/routes/context_commands.py:259`; implementation/call-through.

- `sources/memtomem-memory/packages/memtomem/src/memtomem/web/routes/context_commands.py:238` `@router.delete("/context/commands/{name}")`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/web/routes/context_commands.py:239` `async def delete_command(`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/web/routes/context_commands.py:259` `return await _atomic_kind.delete_artifact(`

**delete_skip_entry**, body `sources/memtomem-memory/packages/memtomem/src/memtomem/web/routes/context_gateway.py:199`; implementation/call-through.

- `sources/memtomem-memory/packages/memtomem/src/memtomem/web/routes/context_gateway.py:190` `def delete_skip_entry(path: Path, exc: OSError, project_root: Path) -> dict[str, str]:`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/web/routes/context_gateway.py:199` `try:`

**delete_mcp_server**, body `sources/memtomem-memory/packages/memtomem/src/memtomem/web/routes/context_mcp_servers.py:323`; implementation/call-through.

- `sources/memtomem-memory/packages/memtomem/src/memtomem/web/routes/context_mcp_servers.py:313` `@router.delete("/context/mcp-servers/{name}")`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/web/routes/context_mcp_servers.py:314` `async def delete_mcp_server(`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/web/routes/context_mcp_servers.py:333` `path.unlink()`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/web/routes/context_mcp_servers.py:354` `"reason": "cascade delete for .mcp.json entries is not supported in v1",`

**update_known_project**, body `sources/memtomem-memory/packages/memtomem/src/memtomem/web/routes/context_projects.py:694`; implementation/call-through.

- `sources/memtomem-memory/packages/memtomem/src/memtomem/web/routes/context_projects.py:669` `@router.patch("/context/known-projects/{scope_id}")`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/web/routes/context_projects.py:670` `async def update_known_project(`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/web/routes/context_projects.py:694` `cfg = _gateway_config(request)`

**delete_known_project**, body `sources/memtomem-memory/packages/memtomem/src/memtomem/web/routes/context_projects.py:738`; implementation/call-through.

- `sources/memtomem-memory/packages/memtomem/src/memtomem/web/routes/context_projects.py:730` `@router.delete("/context/known-projects/{scope_id}")`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/web/routes/context_projects.py:731` `async def delete_known_project(scope_id: str, request: Request) -> dict:`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/web/routes/context_projects.py:741` `removed = store.remove_by_scope_id(scope_id)`

**delete_skill**, body `sources/memtomem-memory/packages/memtomem/src/memtomem/web/routes/context_skills.py:577`; implementation/call-through.

- `sources/memtomem-memory/packages/memtomem/src/memtomem/web/routes/context_skills.py:552` `@router.delete("/context/skills/{name}")`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/web/routes/context_skills.py:553` `async def delete_skill(`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/web/routes/context_skills.py:577` `reject_project_local_write(target_scope, "Delete skill")`

**_delete_locked**, body `sources/memtomem-memory/packages/memtomem/src/memtomem/web/routes/context_skills.py:610`; implementation/call-through.

- `sources/memtomem-memory/packages/memtomem/src/memtomem/web/routes/context_skills.py:609` `def _delete_locked() -> tuple[dict | None, list[str], list[dict[str, str]]]:`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/web/routes/context_skills.py:610` `removed: list[str] = []`

**delete_artifact_label**, body `sources/memtomem-memory/packages/memtomem/src/memtomem/web/routes/context_versions.py:666`; implementation/call-through.

- `sources/memtomem-memory/packages/memtomem/src/memtomem/web/routes/context_versions.py:653` `@router.delete("/context/{artifact_type}/{name}/labels/{label}")`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/web/routes/context_versions.py:654` `async def delete_artifact_label(`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/web/routes/context_versions.py:679` `versioning.delete_label(artifact_dir, label, lock_timeout=lt)`

**_delete_and_load**, body `sources/memtomem-memory/packages/memtomem/src/memtomem/web/routes/context_versions.py:679`; implementation/call-through.

- `sources/memtomem-memory/packages/memtomem/src/memtomem/web/routes/context_versions.py:678` `def _delete_and_load(lt: float | None) -> versioning.VersionsManifest:`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/web/routes/context_versions.py:679` `versioning.delete_label(artifact_dir, label, lock_timeout=lt)`

**delete_namespace**, body `sources/memtomem-memory/packages/memtomem/src/memtomem/web/routes/namespaces.py:189`; implementation/call-through.

- `sources/memtomem-memory/packages/memtomem/src/memtomem/web/routes/namespaces.py:186` `@admin_router.delete("/{namespace}", response_model=DeleteResponse)`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/web/routes/namespaces.py:187` `async def delete_namespace(namespace: str, storage=Depends(get_storage)) -> DeleteResponse:`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/web/routes/namespaces.py:189` `deleted = await storage.delete_by_namespace(namespace)`

**delete_scratch**, body `sources/memtomem-memory/packages/memtomem/src/memtomem/web/routes/scratch.py:92`; implementation/call-through.

- `sources/memtomem-memory/packages/memtomem/src/memtomem/web/routes/scratch.py:86` `@router.delete("/{key}", response_model=ScratchDeleteResponse)`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/web/routes/scratch.py:87` `async def delete_scratch(`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/web/routes/scratch.py:92` `deleted = await storage.scratch_delete(key)`

**delete_target_rule**, body `sources/memtomem-memory/packages/memtomem/src/memtomem/web/routes/settings_sync.py:1014`; implementation/call-through.

- `sources/memtomem-memory/packages/memtomem/src/memtomem/web/routes/settings_sync.py:1003` `@router.post("/settings-sync/rules/delete")`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/web/routes/settings_sync.py:1004` `@router.post("/context/settings/rules/delete")`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/web/routes/settings_sync.py:1005` `async def delete_target_rule(`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/web/routes/settings_sync.py:1014` `canonical_path = project_root / CANONICAL_SETTINGS_FILE`

**delete_source**, body `sources/memtomem-memory/packages/memtomem/src/memtomem/web/routes/sources.py:351`; implementation/call-through.

- `sources/memtomem-memory/packages/memtomem/src/memtomem/web/routes/sources.py:326` `@router.delete("")`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/web/routes/sources.py:327` `async def delete_source(`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/web/routes/sources.py:383` `action="delete",`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/web/routes/sources.py:387` `deleted = await storage.delete_by_source(request_path, as_stored=True)`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/web/routes/sources.py:388` `search_pipeline.invalidate_cache()`

**patch_config**, body `sources/memtomem-memory/packages/memtomem/src/memtomem/web/routes/system_config.py:434`; implementation/call-through.

- `sources/memtomem-memory/packages/memtomem/src/memtomem/web/routes/system_config.py:425` `@router.patch("/config", response_model=ConfigPatchResponse)`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/web/routes/system_config.py:426` `async def patch_config(`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/web/routes/system_config.py:686` `search_pipeline.invalidate_cache()`

**delete_tag**, body `sources/memtomem-memory/packages/memtomem/src/memtomem/web/routes/tags.py:151`; implementation/call-through.

- `sources/memtomem-memory/packages/memtomem/src/memtomem/web/routes/tags.py:142` `@router.delete("/{name}", response_model=TagOpResponse)`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/web/routes/tags.py:143` `async def delete_tag(`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/web/routes/tags.py:152` `result = await tag_svc.delete_tag(`

</details>

## 3. What reaches the agent

System block is empty (`sources/memtomem-memory/packages/memtomem-hermes-memory/__init__.py:1469`). Prefetch calls MCP search then shapes results under max_chars default 4000, allowed 500..9000 (`sources/memtomem-memory/packages/memtomem-hermes-memory/__init__.py:313`; `sources/memtomem-memory/packages/memtomem-hermes-memory/__init__.py:325`; `sources/memtomem-memory/packages/memtomem-hermes-memory/__init__.py:1248`). It can return empty for trivial/stale/timeout/missing-trust cases. Tools independently exposed by MCP must not be attributed to provider.get_tool_schemas.

Host assembly: `hermes-agent/agent/memory_manager.py:352` joins provider system blocks. `hermes-agent/agent/turn_context.py:1081` requests prefetch for nontrivial turns; `hermes-agent/agent/turn_context.py:120` fences it; `hermes-agent/agent/turn_context.py:136` appends it to API-bound user content. Tool results are a third channel. Optional host spill is at `hermes-agent/agent/memory_manager.py:419`, not a universal provider cap. Built-in memory remains separately controlled (`hermes-agent/agent/system_prompt.py:542`).

<details>
<summary>Context and tool-result assembly quotes</summary>

`_shape`, body `sources/memtomem-memory/packages/memtomem-hermes-memory/__init__.py:1489`:

- `sources/memtomem-memory/packages/memtomem-hermes-memory/__init__.py:1496` `body = " ".join(str(r.get("content") or "").split())`
- `sources/memtomem-memory/packages/memtomem-hermes-memory/__init__.py:1502` `entry = entry[: max(0, room - 1)] + "…"`
- `sources/memtomem-memory/packages/memtomem-hermes-memory/__init__.py:1503` `lines.append(entry)`
- `sources/memtomem-memory/packages/memtomem-hermes-memory/__init__.py:1504` `ids.append(str(r.get("chunk_id")))`
- `sources/memtomem-memory/packages/memtomem-hermes-memory/__init__.py:1507` `return "", []`
- `sources/memtomem-memory/packages/memtomem-hermes-memory/__init__.py:1508` `return _HEADER + "\n" + "\n".join(lines), ids`

`prefetch`, body `sources/memtomem-memory/packages/memtomem-hermes-memory/__init__.py:1219`:

- `sources/memtomem-memory/packages/memtomem-hermes-memory/__init__.py:1236` `return text`
- `sources/memtomem-memory/packages/memtomem-hermes-memory/__init__.py:1239` `return finish("trivial")`
- `sources/memtomem-memory/packages/memtomem-hermes-memory/__init__.py:1242` `return finish(outcome)`
- `sources/memtomem-memory/packages/memtomem-hermes-memory/__init__.py:1243` `budget = settings["budget_ms"] / 1000.0`
- `sources/memtomem-memory/packages/memtomem-hermes-memory/__init__.py:1244` `if not slot.done.wait(timeout=max(0.0, budget - (time.perf_counter() - started))):`
- `sources/memtomem-memory/packages/memtomem-hermes-memory/__init__.py:1245` `return finish("hook_timeout")  # the call keeps running and keeps the name's slot`
- `sources/memtomem-memory/packages/memtomem-hermes-memory/__init__.py:1247` `return finish(slot.outcome or "no_result")`
- `sources/memtomem-memory/packages/memtomem-hermes-memory/__init__.py:1248` `text, ids = _shape(slot.result["results"], settings["max_chars"])`
- `sources/memtomem-memory/packages/memtomem-hermes-memory/__init__.py:1251` `return finish("stale")`
- `sources/memtomem-memory/packages/memtomem-hermes-memory/__init__.py:1254` `return finish("hit" if text else "no_results", text, ids)`

`queue_prefetch`, body `sources/memtomem-memory/packages/memtomem-hermes-memory/__init__.py:1319`:

- `sources/memtomem-memory/packages/memtomem-hermes-memory/__init__.py:1319` `return None  # policy: the current turn's query, not the previous one's`

`system_prompt_block`, body `sources/memtomem-memory/packages/memtomem-hermes-memory/__init__.py:1469`:

- `sources/memtomem-memory/packages/memtomem-hermes-memory/__init__.py:1469` `return ""`

`get_tool_schemas`, body `sources/memtomem-memory/packages/memtomem-hermes-memory/__init__.py:1472`:

- `sources/memtomem-memory/packages/memtomem-hermes-memory/__init__.py:1472` `return []`

</details>

## 4. Data access boundaries

Direct store/MCP access `yes` using the same local process/files or configured endpoint and credentials. Provider stores no mandatory engine credential itself; MCP command/env/headers live in host MCP configuration (`sources/memtomem-memory/packages/memtomem-hermes-memory/__init__.py:250`). Accountless local mode `yes` for local storage/configured local embeddings, with backend-specific requirements; availability not executed. No provider analytics default proves engine telemetry off.

Boundary model: same OS user, terminal, mounted filesystem and allowed network. Direct reachability means another client can use the same file/endpoint and available credentials, not unauthenticated access or encryption bypass. Hermes setup persists declared secrets through `hermes-agent/hermes_cli/memory_setup.py:321` and `hermes-agent/hermes_cli/config.py:2697`. These are possible configured paths; no live credentials were read.

Host credential-file detail: `get_hermes_home()/.env` is resolved at `hermes-agent/hermes_cli/config.py:473`. New files are secured to 0600 at `hermes-agent/hermes_cli/config.py:506`; the writer preserves an existing mode at `hermes-agent/hermes_cli/config.py:2544` and `:2562`. Therefore 0600 is not an unconditional mode claim for every existing profile.

<details>
<summary>Paths, environment, permissions, egress, telemetry and process-launch evidence</summary>

- `sources/memtomem-memory/packages/memtomem/src/memtomem/config.py:1091` `base_url: str = "http://localhost:11434"`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/config.py:1246` `os.environ.get("LANGFUSE_PUBLIC_KEY") and os.environ.get("LANGFUSE_SECRET_KEY")`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/config.py:1866` `raw = os.environ.get(name)`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/config.py:1939` `''os.environ'' normalises keys. If several spellings of the same name are`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/config.py:1940` `exported at once, the last one in ''os.environ'' order is returned,`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/config.py:1979` `for name in os.environ:`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/config.py:3218` `auto-memory (https://code.claude.com/docs/en/memory). Subdirs without`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/config.py:3224` `(https://learn.chatgpt.com/docs/customization/memories).`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/config.py:3422` `home = Path.home()`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/config.py:3455` `init-wizard write path (string-typed ''state["db_path"]'') and the`

- `sources/memtomem-memory/packages/memtomem/src/memtomem/context/settings.py:1463` `atomic_write_text(path, payload, mode=0o600)`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/context/settings.py:1471` `atomic_write_text(path, payload, mode=0o600)`

- `sources/memtomem-memory/packages/memtomem/src/memtomem/context/settings_doctor.py:481` `home = str(Path.home())`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/context/settings_doctor.py:490` `resolved = str(Path.home().resolve())`

- `sources/memtomem-memory/packages/memtomem/src/memtomem/context/settings_migrate.py:599` `atomic_write_text(path, payload, mode=0o600)`

- `sources/memtomem-memory/packages/memtomem/src/memtomem/embedding/hub_telemetry.py:1` `"""Run memtomem's Hub downloads with huggingface_hub telemetry off (#2550, #2552, #2556)."""`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/embedding/hub_telemetry.py:9` `_hub_telemetry_lock = threading.Lock()`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/embedding/hub_telemetry.py:10` `_hub_telemetry_depth = 0`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/embedding/hub_telemetry.py:11` `_hub_telemetry_saved = False`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/embedding/hub_telemetry.py:15` `def hub_telemetry_off() -> Iterator[None]:`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/embedding/hub_telemetry.py:16` `"""Run memtomem's Hub downloads with huggingface_hub telemetry off.`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/embedding/hub_telemetry.py:21` `huggingface_hub on a cache miss. With telemetry on, huggingface_hub tags the user agent with`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/embedding/hub_telemetry.py:28` `headers while a window is open is also sent without telemetry. Windows may`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/embedding/hub_telemetry.py:34` `global _hub_telemetry_depth, _hub_telemetry_saved`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/embedding/hub_telemetry.py:37` `with _hub_telemetry_lock:`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/embedding/hub_telemetry.py:38` `if _hub_telemetry_depth == 0:`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/embedding/hub_telemetry.py:39` `_hub_telemetry_saved = constants.HF_HUB_DISABLE_TELEMETRY`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/embedding/hub_telemetry.py:40` `constants.HF_HUB_DISABLE_TELEMETRY = True`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/embedding/hub_telemetry.py:41` `_hub_telemetry_depth += 1`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/embedding/hub_telemetry.py:45` `with _hub_telemetry_lock:`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/embedding/hub_telemetry.py:46` `_hub_telemetry_depth -= 1`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/embedding/hub_telemetry.py:47` `if _hub_telemetry_depth == 0 and constants.HF_HUB_DISABLE_TELEMETRY is True:`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/embedding/hub_telemetry.py:48` `constants.HF_HUB_DISABLE_TELEMETRY = _hub_telemetry_saved`

- `sources/memtomem-memory/packages/memtomem/src/memtomem/server/tools/status_config.py:226` `def _collect_concurrent_writers(db_path: Path) -> dict | None:`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/server/tools/status_config.py:241` `digest = _store_digest_for(db_path)`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/server/tools/status_config.py:463` `db_path_resolved = Path(config.storage.sqlite_path).expanduser().resolve()`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/server/tools/status_config.py:594` `concurrent = await asyncio.to_thread(_collect_concurrent_writers, db_path_resolved)`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/server/tools/status_config.py:606` `"db_path": str(db_path_resolved),`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/server/tools/status_config.py:814` `home_text = str(Path.home().resolve() if home is None else home)`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/server/tools/status_config.py:965` `value=_shorten_status_path(cfg["db_path"]),`

- `sources/memtomem-memory/packages/memtomem/src/memtomem/storage/hybrid_store.py:96` `descriptor = os.open(path, os.O_CREAT | os.O_EXCL | os.O_WRONLY, 0o600)`

- `sources/memtomem-memory/packages/memtomem/src/memtomem/storage/mixins/__init__.py:8` `from memtomem.storage.mixins.analytics import AnalyticsMixin`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/storage/mixins/__init__.py:25` `"AnalyticsMixin",`

- `sources/memtomem-memory/packages/memtomem/src/memtomem/storage/mixins/analytics.py:1` `"""Analytics and reporting storage methods , replaces direct _get_db() in tools."""`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/storage/mixins/analytics.py:57` `class AnalyticsMixin:`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/storage/mixins/analytics.py:58` `"""Mixin providing analytics methods. Requires self._get_db() and`

- `sources/memtomem-memory/packages/memtomem/src/memtomem/storage/schema_peers.py:58` `async def refuse_migration_under_live_peers(db: sqlite3.Connection, db_path: Path) -> None:`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/storage/schema_peers.py:67` `digest = store_digest_for(db_path)`

- `sources/memtomem-memory/packages/memtomem/src/memtomem/storage/sqlite_backend.py:95` `AnalyticsMixin,`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/storage/sqlite_backend.py:269` `AnalyticsMixin,`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/storage/sqlite_backend.py:377` `db_path = Path(self._config.sqlite_path).expanduser()`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/storage/sqlite_backend.py:380` `db_path.parent.mkdir(parents=True, exist_ok=True, mode=0o700)`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/storage/sqlite_backend.py:383` `self._db = sqlite3.connect(str(db_path), timeout=10)`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/storage/sqlite_backend.py:392` `db_path.chmod(0o600)`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/storage/sqlite_backend.py:408` `await refuse_migration_under_live_peers(self._db, db_path)`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/storage/sqlite_backend.py:915` `db_path = Path(self._config.sqlite_path).expanduser()`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/storage/sqlite_backend.py:916` `rconn = sqlite3.connect(str(db_path), timeout=10, check_same_thread=False)`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/storage/sqlite_backend.py:1246` `def db_path(self) -> Path:`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/storage/sqlite_backend.py:3017` `db_path = str(Path(self._config.sqlite_path).expanduser())`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/storage/sqlite_backend.py:3020` `conn = sqlite3.connect(db_path, timeout=10)`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/storage/sqlite_backend.py:3216` `BM25-only condition this telemetry exists to surface.`

- `sources/memtomem-memory/packages/memtomem/src/memtomem/web/routes/system_config.py:1177` `On macOS / Linux this uses ''subprocess.run'' with stderr captured`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/web/routes/system_config.py:1180` `instead of a silent "Popen succeeded but Finder never opened"`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/web/routes/system_config.py:1203` `result = subprocess.run(`
- `sources/memtomem-memory/packages/memtomem/src/memtomem/web/routes/system_config.py:1212` `except subprocess.TimeoutExpired as exc:`

- `sources/memtomem-memory/packages/memtomem-hermes-memory/__init__.py:1420` `noninteractive = os.environ.get("HERMES_NONINTERACTIVE", "").strip().lower() in {`

</details>

## 5. Reproducibility

Provider does not pin the existing MCP server version; `unpinned` from this adapter. Background-search/mirror capability checks are runtime contract checks, not exact version locks (`sources/memtomem-memory/packages/memtomem-hermes-memory/__init__.py:611`; `sources/memtomem-memory/packages/memtomem-hermes-memory/__init__.py:1053`).

Retained lock/checksum files: `sources/memtomem-memory/packages/opencode-memtomem/package-lock.json:1`, `sources/memtomem-memory/packages/memtomem/tests-js/package-lock.json:1`. Upstream completeness is unknown: fetch.py:15 excludes .lock files, fetch.py:12 excludes build/vendor directories. Absence here is not absence upstream.

- `sources/memtomem-memory/packages/memtomem-hermes-memory/plugin.yaml:2` `version: "0.2.0"`
- `sources/memtomem-memory/packages/memtomem-hermes-memory/plugin.yaml:4` `requires_hermes: ">=0.21.5"`

## 6. Tests

Hermes provider and mirror tests are distinct from engine mem_delete tests. There is no provider forget tool test; engine storage mutation tests exist; separate route/concurrency tests are listed below. Storage deletion test asserts the chunk disappears. This is engine-storage coverage, not a native provider forget-tool test. (`sources/memtomem-memory/packages/memtomem/tests/test_server_tools_core.py:406`).

Adapter-related declarations: **141**. Python AST test_ functions, fixtures excluded; literal TS/JS it/test, Go Test*, Rust test_* declarations. No parametrization expansion, collection, execution, or coverage measurement. Other rows are engine test candidates selected by mutation names; names alone do not verify forget coverage.

| file | declarations | adapter-related | forget-related declarations |
|---|---:|---|---|
| `sources/memtomem-memory/packages/langchain-memtomem/tests/test_lcmm_protocol.py:1` | 25 | no | `test_delete_interrupted_then_finished_on_open` `sources/memtomem-memory/packages/langchain-memtomem/tests/test_lcmm_protocol.py:121`; `test_delete_marker_is_superseded_by_a_later_put` `sources/memtomem-memory/packages/langchain-memtomem/tests/test_lcmm_protocol.py:139`; `test_two_instances_delete_races_put` `sources/memtomem-memory/packages/langchain-memtomem/tests/test_lcmm_protocol.py:212`; `test_generation_continues_past_a_delete` `sources/memtomem-memory/packages/langchain-memtomem/tests/test_lcmm_protocol.py:277`; `test_interrupted_delete_then_interrupted_put_keeps_the_new_record` `sources/memtomem-memory/packages/langchain-memtomem/tests/test_lcmm_protocol.py:290`; `test_recreated_key_is_not_served_from_a_hit_cached_before_the_delete` `sources/memtomem-memory/packages/langchain-memtomem/tests/test_lcmm_protocol.py:312`; `test_cancelled_delete_cannot_remove_a_record_written_after_it` `sources/memtomem-memory/packages/langchain-memtomem/tests/test_lcmm_protocol.py:405`; `test_repeated_delete_keeps_a_legacy_marker_generation` `sources/memtomem-memory/packages/langchain-memtomem/tests/test_lcmm_protocol.py:440`; `test_the_longest_accepted_key_can_be_written_and_deleted` `sources/memtomem-memory/packages/langchain-memtomem/tests/test_lcmm_protocol.py:481` |
| `sources/memtomem-memory/packages/langchain-memtomem/tests/test_lcmm_store_contract.py:1` | 13 | no | `test_put_replaces_and_delete_removes` `sources/memtomem-memory/packages/langchain-memtomem/tests/test_lcmm_store_contract.py:71`; `test_put_with_none_value_deletes` `sources/memtomem-memory/packages/langchain-memtomem/tests/test_lcmm_store_contract.py:214` |
| `sources/memtomem-memory/packages/memtomem/tests/test_add_namespace_dayfile.py:1` | 56 | no | `test_deleting_one_chunk_leaves_the_survivors_namespace_alone` `sources/memtomem-memory/packages/memtomem/tests/test_add_namespace_dayfile.py:142`; `test_mem_delete_reports_a_namespace_failure_as_retryable` `sources/memtomem-memory/packages/memtomem/tests/test_add_namespace_dayfile.py:1083` |
| `sources/memtomem-memory/packages/memtomem/tests/test_atomic_kind_abandon.py:1` | 4 | no | `test_a_cancelled_delete_removes_nothing` `sources/memtomem-memory/packages/memtomem/tests/test_atomic_kind_abandon.py:198` |
| `sources/memtomem-memory/packages/memtomem/tests/test_chunk_frontmatter_route.py:1` | 13 | no | `test_a_symlinked_or_deleted_source_is_false` `sources/memtomem-memory/packages/memtomem/tests/test_chunk_frontmatter_route.py:154` |
| `sources/memtomem-memory/packages/memtomem/tests/test_chunk_links_schema.py:1` | 10 | no | `test_delete_source_sets_link_source_id_to_null` `sources/memtomem-memory/packages/memtomem/tests/test_chunk_links_schema.py:104`; `test_delete_target_cascades_link_row` `sources/memtomem-memory/packages/memtomem/tests/test_chunk_links_schema.py:136` |
| `sources/memtomem-memory/packages/memtomem/tests/test_chunk_links_writer.py:1` | 7 | no | `test_source_delete_nulls_link_source_id` `sources/memtomem-memory/packages/memtomem/tests/test_chunk_links_writer.py:328` |
| `sources/memtomem-memory/packages/memtomem/tests/test_chunkers_extended.py:1` | 46 | no | `test_deleted_chunks_detected` `sources/memtomem-memory/packages/memtomem/tests/test_chunkers_extended.py:559`; `test_mixed_upsert_delete_unchanged` `sources/memtomem-memory/packages/memtomem/tests/test_chunkers_extended.py:567` |
| `sources/memtomem-memory/packages/memtomem/tests/test_claude_mcp_preflight.py:1` | 58 | no | `test_review_unrelated_error_does_not_invalidate_plugin_parsing` `sources/memtomem-memory/packages/memtomem/tests/test_claude_mcp_preflight.py:757` |
| `sources/memtomem-memory/packages/memtomem/tests/test_cli.py:1` | 148 | no | `test_unset_deletes_empty_config_file` `sources/memtomem-memory/packages/memtomem/tests/test_cli.py:1762` |
| `sources/memtomem-memory/packages/memtomem/tests/test_cli_lazy_subcommands.py:1` | 13 | no | `test_deletion_removes_both_lazy_and_loaded_entries` `sources/memtomem-memory/packages/memtomem/tests/test_cli_lazy_subcommands.py:144` |
| `sources/memtomem-memory/packages/memtomem/tests/test_config_load_diagnostics.py:1` | 26 | no | `test_a_deleted_config_json_clears_its_entry` `sources/memtomem-memory/packages/memtomem/tests/test_config_load_diagnostics.py:396` |
| `sources/memtomem-memory/packages/memtomem/tests/test_consolidation_already_consolidated.py:1` | 19 | no | `test_deleting_the_summary_lists_its_originals_again` `sources/memtomem-memory/packages/memtomem/tests/test_consolidation_already_consolidated.py:130` |
| `sources/memtomem-memory/packages/memtomem/tests/test_consolidation_summary_edit.py:1` | 14 | no | `test_deleting_the_summary_lists_its_originals_again` `sources/memtomem-memory/packages/memtomem/tests/test_consolidation_summary_edit.py:207`; `test_deleting_the_summary_beside_another_entry_carries_nothing` `sources/memtomem-memory/packages/memtomem/tests/test_consolidation_summary_edit.py:217` |
| `sources/memtomem-memory/packages/memtomem/tests/test_context_bundle.py:1` | 92 | no | `test_orphan_snapshot_refuses_and_never_says_delete` `sources/memtomem-memory/packages/memtomem/tests/test_context_bundle.py:382`; `test_replaced_staging_is_neither_promoted_nor_deleted` `sources/memtomem-memory/packages/memtomem/tests/test_context_bundle.py:834` |
| `sources/memtomem-memory/packages/memtomem/tests/test_context_canonical_txn.py:1` | 18 | no | `test_skill_delete_acquires_canonical_lock_even_when_absent` `sources/memtomem-memory/packages/memtomem/tests/test_context_canonical_txn.py:453` |
| `sources/memtomem-memory/packages/memtomem/tests/test_context_cli_subprocess_e2e.py:1` | 11 | no | `test_version_delete_label_roundtrip` `sources/memtomem-memory/packages/memtomem/tests/test_context_cli_subprocess_e2e.py:312`; `test_version_delete_label_flat_layout_refuses` `sources/memtomem-memory/packages/memtomem/tests/test_context_cli_subprocess_e2e.py:358` |
| `sources/memtomem-memory/packages/memtomem/tests/test_context_install.py:1` | 37 | no | `test_install_refuses_worktree_deletion` `sources/memtomem-memory/packages/memtomem/tests/test_context_install.py:721` |
| `sources/memtomem-memory/packages/memtomem/tests/test_context_migrate.py:1` | 150 | no | `test_fanout_dry_run_previews_deletion_half` `sources/memtomem-memory/packages/memtomem/tests/test_context_migrate.py:3647`; `test_a_failed_tree_fill_does_not_delete_a_replacement` `sources/memtomem-memory/packages/memtomem/tests/test_context_migrate.py:1085`; `test_a_failed_file_fill_does_not_delete_a_replacement` `sources/memtomem-memory/packages/memtomem/tests/test_context_migrate.py:1110` |
| `sources/memtomem-memory/packages/memtomem/tests/test_context_settings.py:1` | 109 | no | `test_delete_target_rule_uses_index_and_hash_for_duplicate_matchers` `sources/memtomem-memory/packages/memtomem/tests/test_context_settings.py:2085`; `test_delete_target_rule_ignores_stale_canonical_mtime` `sources/memtomem-memory/packages/memtomem/tests/test_context_settings.py:2117` |
| `sources/memtomem-memory/packages/memtomem/tests/test_context_status.py:1` | 61 | no | `test_missing_state_dest_deleted` `sources/memtomem-memory/packages/memtomem/tests/test_context_status.py:217` |
| `sources/memtomem-memory/packages/memtomem/tests/test_context_swap_prelude_fanout.py:1` | 77 | no | `test_row_4_refuses_and_deletes_nothing` `sources/memtomem-memory/packages/memtomem/tests/test_context_swap_prelude_fanout.py:292`; `test_unconfirmed_user_delete_of_a_mid_swap_skill_changes_nothing` `sources/memtomem-memory/packages/memtomem/tests/test_context_swap_prelude_fanout.py:601`; `test_confirmed_user_delete_recovers_then_deletes` `sources/memtomem-memory/packages/memtomem/tests/test_context_swap_prelude_fanout.py:620`; `test_delete_maps_row_4_to_409_swap_recovery_pending` `sources/memtomem-memory/packages/memtomem/tests/test_context_swap_prelude_fanout.py:694`; `test_delete_converges_a_row_2_then_deletes` `sources/memtomem-memory/packages/memtomem/tests/test_context_swap_prelude_fanout.py:709`; `test_row_4_refuses_and_deletes_nothing` `sources/memtomem-memory/packages/memtomem/tests/test_context_swap_prelude_fanout.py:773`; `test_row_4_refuses_the_update_and_deletes_nothing` `sources/memtomem-memory/packages/memtomem/tests/test_context_swap_prelude_fanout.py:816` |
| `sources/memtomem-memory/packages/memtomem/tests/test_context_transfer.py:1` | 64 | no | `test_a_replaced_parked_source_is_not_recommended_for_deletion` `sources/memtomem-memory/packages/memtomem/tests/test_context_transfer.py:1411` |
| `sources/memtomem-memory/packages/memtomem/tests/test_context_update.py:1` | 104 | no | `test_update_removes_upstream_deleted_file` `sources/memtomem-memory/packages/memtomem/tests/test_context_update.py:2069`; `test_update_force_removes_deleted_and_edited_file_with_bak` `sources/memtomem-memory/packages/memtomem/tests/test_context_update.py:2096`; `test_update_noop_with_user_deleted_file_does_not_resurrect` `sources/memtomem-memory/packages/memtomem/tests/test_context_update.py:2163`; `test_update_refuses_on_user_deleted_file_after_wiki_advance` `sources/memtomem-memory/packages/memtomem/tests/test_context_update.py:2182`; `test_update_refuses_worktree_deletion` `sources/memtomem-memory/packages/memtomem/tests/test_context_update.py:2871` |
| `sources/memtomem-memory/packages/memtomem/tests/test_context_versioning.py:1` | 92 | no | `test_delete_label_preserves_unknown_fields` `sources/memtomem-memory/packages/memtomem/tests/test_context_versioning.py:337`; `test_non_conforming_staging_name_is_not_deleted` `sources/memtomem-memory/packages/memtomem/tests/test_context_versioning.py:940` |
| `sources/memtomem-memory/packages/memtomem/tests/test_context_window.py:1` | 48 | no | `test_deleted_chunk_graceful` `sources/memtomem-memory/packages/memtomem/tests/test_context_window.py:208` |
| `sources/memtomem-memory/packages/memtomem/tests/test_count_chunks_by_sources.py:1` | 8 | no | `test_the_preview_number_matches_what_apply_deletes` `sources/memtomem-memory/packages/memtomem/tests/test_count_chunks_by_sources.py:123` |
| `sources/memtomem-memory/packages/memtomem/tests/test_cpu_embedding_profiles.py:1` | 24 | no | `test_receipt_skips_chunker_but_changed_source_invalidates` `sources/memtomem-memory/packages/memtomem/tests/test_cpu_embedding_profiles.py:223`; `test_receipt_invalidates_when_stored_metadata_changes` `sources/memtomem-memory/packages/memtomem/tests/test_cpu_embedding_profiles.py:278` |
| `sources/memtomem-memory/packages/memtomem/tests/test_cpu_profile_review_fixes.py:1` | 37 | no | `test_upgrading_memtomem_invalidates_a_receipt` `sources/memtomem-memory/packages/memtomem/tests/test_cpu_profile_review_fixes.py:356`; `test_manifest_span_change_invalidates_a_receipt` `sources/memtomem-memory/packages/memtomem/tests/test_cpu_profile_review_fixes.py:400` |
| `sources/memtomem-memory/packages/memtomem/tests/test_dedup.py:1` | 31 | no | `test_dry_run_returns_count_without_deleting` `sources/memtomem-memory/packages/memtomem/tests/test_dedup.py:505`; `test_apply_deletes_and_merges_tags` `sources/memtomem-memory/packages/memtomem/tests/test_dedup.py:542`; `test_failed_apply_invalidates` `sources/memtomem-memory/packages/memtomem/tests/test_dedup.py:649`; `test_failed_dry_run_does_not_invalidate` `sources/memtomem-memory/packages/memtomem/tests/test_dedup.py:664`; `test_keeper_in_delete_ids_is_an_error_string` `sources/memtomem-memory/packages/memtomem/tests/test_dedup.py:678`; `test_keeper_in_delete_ids_is_refused_before_any_read` `sources/memtomem-memory/packages/memtomem/tests/test_dedup.py:699` |
| `sources/memtomem-memory/packages/memtomem/tests/test_derived_target_project_tier_refusal.py:1` | 11 | no | `test_pinned_delete_user_scope_refuses_a_project_tier_base` `sources/memtomem-memory/packages/memtomem/tests/test_derived_target_project_tier_refusal.py:124` |
| `sources/memtomem-memory/packages/memtomem/tests/test_differ.py:1` | 25 | no | `test_deletions_when_new_chunks_empty` `sources/memtomem-memory/packages/memtomem/tests/test_differ.py:60`; `test_collapsing_duplicate_hash_deletes_unused_ids` `sources/memtomem-memory/packages/memtomem/tests/test_differ.py:342`; `test_collapsing_duplicate_hash_without_hierarchy_deletes_the_surplus` `sources/memtomem-memory/packages/memtomem/tests/test_differ.py:362` |
| `sources/memtomem-memory/packages/memtomem/tests/test_dirty.py:1` | 19 | no | `test_user_deleted_file_reports_missing` `sources/memtomem-memory/packages/memtomem/tests/test_dirty.py:415` |
| `sources/memtomem-memory/packages/memtomem/tests/test_dirty_digest.py:1` | 33 | no | `test_deletion_detected_via_digest_keys_without_manifest` `sources/memtomem-memory/packages/memtomem/tests/test_dirty_digest.py:259`; `test_reconcile_deletes_untouched_wiki_dropped_file_with_fresh_mtime` `sources/memtomem-memory/packages/memtomem/tests/test_dirty_digest.py:306`; `test_recorded_untouched_bytes_deleted_despite_fresh_mtime` `sources/memtomem-memory/packages/memtomem/tests/test_dirty_digest.py:707`; `test_recorded_diverged_but_baked_deleted` `sources/memtomem-memory/packages/memtomem/tests/test_dirty_digest.py:739` |
| `sources/memtomem-memory/packages/memtomem/tests/test_docs_guards.py:1` | 107 | no | `test_guide_states_the_never_delete_while_running_rule` `sources/memtomem-memory/packages/memtomem/tests/test_docs_guards.py:421` |
| `sources/memtomem-memory/packages/memtomem/tests/test_entities.py:1` | 23 | no | `test_delete_entities` `sources/memtomem-memory/packages/memtomem/tests/test_entities.py:107` |
| `sources/memtomem-memory/packages/memtomem/tests/test_entity_extraction.py:1` | 26 | no | `test_delete_entities` `sources/memtomem-memory/packages/memtomem/tests/test_entity_extraction.py:238` |
| `sources/memtomem-memory/packages/memtomem/tests/test_entity_scan_overwrite.py:1` | 11 | no | `test_overwrite_with_no_entities_deletes_old_rows` `sources/memtomem-memory/packages/memtomem/tests/test_entity_scan_overwrite.py:36`; `test_dry_run_reports_but_does_not_delete` `sources/memtomem-memory/packages/memtomem/tests/test_entity_scan_overwrite.py:51`; `test_write_invalidates` `sources/memtomem-memory/packages/memtomem/tests/test_entity_scan_overwrite.py:119`; `test_dry_run_does_not_invalidate` `sources/memtomem-memory/packages/memtomem/tests/test_entity_scan_overwrite.py:130`; `test_failure_after_a_write_still_invalidates` `sources/memtomem-memory/packages/memtomem/tests/test_entity_scan_overwrite.py:140` |
| `sources/memtomem-memory/packages/memtomem/tests/test_excluded_source_mutation.py:1` | 4 | no | `test_a_rollback_after_the_purge_does_not_bring_the_chunks_back` `sources/memtomem-memory/packages/memtomem/tests/test_excluded_source_mutation.py:253` |
| `sources/memtomem-memory/packages/memtomem/tests/test_health_watchdog.py:1` | 40 | no | `test_cleanup_orphans_skips_suspected_mass_delete` `sources/memtomem-memory/packages/memtomem/tests/test_health_watchdog.py:424` |
| `sources/memtomem-memory/packages/memtomem/tests/test_hermes_memory_contract.py:1` | 5 | yes | none |
| `sources/memtomem-memory/packages/memtomem/tests/test_hermes_memory_provider.py:1` | 92 | yes | none |
| `sources/memtomem-memory/packages/memtomem/tests/test_hermes_mirror_storage.py:1` | 17 | yes | `test_purge_deletes_rows_and_fences_older_writes` `sources/memtomem-memory/packages/memtomem/tests/test_hermes_mirror_storage.py:125`; `test_an_older_purge_is_fenced` `sources/memtomem-memory/packages/memtomem/tests/test_hermes_mirror_storage.py:138`; `test_a_purge_keeps_rows_newer_than_itself_in_either_order` `sources/memtomem-memory/packages/memtomem/tests/test_hermes_mirror_storage.py:173`; `test_a_purge_keeps_a_newer_tombstone_so_an_older_add_stays_dead` `sources/memtomem-memory/packages/memtomem/tests/test_hermes_mirror_storage.py:184`; `test_every_order_of_writes_and_a_purge_converges` `sources/memtomem-memory/packages/memtomem/tests/test_hermes_mirror_storage.py:192` |
| `sources/memtomem-memory/packages/memtomem/tests/test_home_guard_engine.py:1` | 71 | no | `test_deletion_is_a_violation` `sources/memtomem-memory/packages/memtomem/tests/test_home_guard_engine.py:176`; `test_witness_sees_a_deletion` `sources/memtomem-memory/packages/memtomem/tests/test_home_guard_engine.py:896` |
| `sources/memtomem-memory/packages/memtomem/tests/test_id_access_bypass_pins.py:1` | 7 | no | `test_dedup_merge_deletes_an_out_of_boundary_id` `sources/memtomem-memory/packages/memtomem/tests/test_id_access_bypass_pins.py:175` |
| `sources/memtomem-memory/packages/memtomem/tests/test_idempotency.py:1` | 27 | no | `test_claim_purges_expired_rows` `sources/memtomem-memory/packages/memtomem/tests/test_idempotency.py:106` |
| `sources/memtomem-memory/packages/memtomem/tests/test_import_exclusion_after_write.py:1` | 11 | no | `test_reimport_that_only_purges_still_flags_the_session` `sources/memtomem-memory/packages/memtomem/tests/test_import_exclusion_after_write.py:86`; `test_purge_if_excluded_deletes_the_chunks_and_clears_hold_and_pending` `sources/memtomem-memory/packages/memtomem/tests/test_import_exclusion_after_write.py:138`; `test_purge_if_excluded_indexes_a_file_that_is_not_excluded` `sources/memtomem-memory/packages/memtomem/tests/test_import_exclusion_after_write.py:160`; `test_the_purge_waits_for_another_task_write_transaction` `sources/memtomem-memory/packages/memtomem/tests/test_import_exclusion_after_write.py:171`; `test_purge_if_excluded_deletes_a_symlinked_path_own_rows_not_its_target` `sources/memtomem-memory/packages/memtomem/tests/test_import_exclusion_after_write.py:214`; `test_the_purge_key_never_resolves_the_leaf` `sources/memtomem-memory/packages/memtomem/tests/test_import_exclusion_after_write.py:267`; `test_a_leaf_spelled_in_another_case_purges_nothing_on_windows` `sources/memtomem-memory/packages/memtomem/tests/test_import_exclusion_after_write.py:292` |
| `sources/memtomem-memory/packages/memtomem/tests/test_import_relinked_after_write.py:1` | 16 | no | `test_a_relinked_and_excluded_leaf_is_held_not_purged` `sources/memtomem-memory/packages/memtomem/tests/test_import_relinked_after_write.py:259` |
| `sources/memtomem-memory/packages/memtomem/tests/test_index_invalidates_search_cache.py:1` | 14 | no | `test_changed_file_invalidates` `sources/memtomem-memory/packages/memtomem/tests/test_index_invalidates_search_cache.py:177`; `test_unchanged_file_does_not_invalidate` `sources/memtomem-memory/packages/memtomem/tests/test_index_invalidates_search_cache.py:187`; `test_deleted_file_invalidates` `sources/memtomem-memory/packages/memtomem/tests/test_index_invalidates_search_cache.py:198`; `test_steady_state_reindex_does_not_invalidate` `sources/memtomem-memory/packages/memtomem/tests/test_index_invalidates_search_cache.py:274` |
| `sources/memtomem-memory/packages/memtomem/tests/test_index_path_waits_for_writer.py:1` | 24 | no | `test_a_deleted_file_is_held_once_the_owner_finishes` `sources/memtomem-memory/packages/memtomem/tests/test_index_path_waits_for_writer.py:153`; `test_a_non_regular_file_relinked_during_the_wait_deletes_only_its_own_rows` `sources/memtomem-memory/packages/memtomem/tests/test_index_path_waits_for_writer.py:424`; `test_an_emptied_file_relinked_during_the_scope_wait_stops_and_deletes_nothing` `sources/memtomem-memory/packages/memtomem/tests/test_index_path_waits_for_writer.py:491` |
| `sources/memtomem-memory/packages/memtomem/tests/test_indexing_engine.py:1` | 269 | no | `test_bulk_unchanged_and_delete_only_namespaces_are_preview_only` `sources/memtomem-memory/packages/memtomem/tests/test_indexing_engine.py:1841`; `test_force_reindex_deletes_vanished_chunks` `sources/memtomem-memory/packages/memtomem/tests/test_indexing_engine.py:3370`; `test_collapsing_identical_sections_deletes_orphan_rows` `sources/memtomem-memory/packages/memtomem/tests/test_indexing_engine.py:3467`; `test_delete_section_removes_chunks` `sources/memtomem-memory/packages/memtomem/tests/test_indexing_engine.py:4232` |
| `sources/memtomem-memory/packages/memtomem/tests/test_indexing_summarizer.py:1` | 19 | no | `test_llm_failure_no_cache_does_not_call_delete` `sources/memtomem-memory/packages/memtomem/tests/test_indexing_summarizer.py:323` |
| `sources/memtomem-memory/packages/memtomem/tests/test_langgraph.py:1` | 93 | no | `test_delete_reports_whether_rows_were_deleted` `sources/memtomem-memory/packages/memtomem/tests/test_langgraph.py:904`; `test_delete_refuses_an_out_of_boundary_id` `sources/memtomem-memory/packages/memtomem/tests/test_langgraph.py:937`; `test_shared_chunk_with_confirm_deletes_and_records_one_consent` `sources/memtomem-memory/packages/memtomem/tests/test_langgraph.py:2080`; `test_other_tiers_delete_without_asking_and_record_no_consent` `sources/memtomem-memory/packages/memtomem/tests/test_langgraph.py:2106`; `test_a_vanished_source_directory_still_deletes_and_stays_gone` `sources/memtomem-memory/packages/memtomem/tests/test_langgraph.py:2133` |
| `sources/memtomem-memory/packages/memtomem/tests/test_langgraph_hybrid_privacy.py:1` | 13 | no | `test_legacy_identifiers_can_be_read_and_deleted` `sources/memtomem-memory/packages/memtomem/tests/test_langgraph_hybrid_privacy.py:233` |
| `sources/memtomem-memory/packages/memtomem/tests/test_lockfile.py:1` | 25 | no | `test_remove_entry_deletes_and_returns_true` `sources/memtomem-memory/packages/memtomem/tests/test_lockfile.py:248` |
| `sources/memtomem-memory/packages/memtomem/tests/test_locking_contention.py:1` | 54 | no | `test_holds_the_lifecycle_barrier_while_deleting` `sources/memtomem-memory/packages/memtomem/tests/test_locking_contention.py:1291` |
| `sources/memtomem-memory/packages/memtomem/tests/test_maintenance_runs.py:1` | 16 | no | `test_deleted_policy_history_stays_reachable` `sources/memtomem-memory/packages/memtomem/tests/test_maintenance_runs.py:184` |
| `sources/memtomem-memory/packages/memtomem/tests/test_mcp_servers_copy.py:1` | 21 | no | `test_replaced_staging_is_neither_promoted_nor_deleted` `sources/memtomem-memory/packages/memtomem/tests/test_mcp_servers_copy.py:324` |
| `sources/memtomem-memory/packages/memtomem/tests/test_mcp_tag_management_parity.py:1` | 18 | no | `test_mcp_rename_routes_through_service_invalidates_cache` `sources/memtomem-memory/packages/memtomem/tests/test_mcp_tag_management_parity.py:49`; `test_mcp_rename_dry_run_does_not_invalidate_cache` `sources/memtomem-memory/packages/memtomem/tests/test_mcp_tag_management_parity.py:68`; `test_mcp_delete_routes_through_service` `sources/memtomem-memory/packages/memtomem/tests/test_mcp_tag_management_parity.py:85`; `test_mcp_delete_dry_run_no_invalidate` `sources/memtomem-memory/packages/memtomem/tests/test_mcp_tag_management_parity.py:100`; `test_mcp_merge_dry_run_no_invalidate` `sources/memtomem-memory/packages/memtomem/tests/test_mcp_tag_management_parity.py:131`; `test_mcp_delete_defaults_to_dry_run` `sources/memtomem-memory/packages/memtomem/tests/test_mcp_tag_management_parity.py:166`; `test_mem_do_delete_refuses_malformed_dry_run` `sources/memtomem-memory/packages/memtomem/tests/test_mcp_tag_management_parity.py:226` |
| `sources/memtomem-memory/packages/memtomem/tests/test_memory_crud_concurrency.py:1` | 24 | no | `test_chunk_deleted_while_waiting_for_lock` `sources/memtomem-memory/packages/memtomem/tests/test_memory_crud_concurrency.py:171`; `test_bulk_source_delete_not_resurrected_by_inflight_edit` `sources/memtomem-memory/packages/memtomem/tests/test_memory_crud_concurrency.py:211`; `test_bulk_namespace_delete_not_resurrected_by_inflight_add` `sources/memtomem-memory/packages/memtomem/tests/test_memory_crud_concurrency.py:256`; `test_delete_shares_the_removal_contract` `sources/memtomem-memory/packages/memtomem/tests/test_memory_crud_concurrency.py:655`; `test_delete_does_not_recreate_a_source_removed_before_the_write` `sources/memtomem-memory/packages/memtomem/tests/test_memory_crud_concurrency.py:772`; `test_delete_leaves_a_source_replaced_before_the_write_as_found` `sources/memtomem-memory/packages/memtomem/tests/test_memory_crud_concurrency.py:791` |
| `sources/memtomem-memory/packages/memtomem/tests/test_memory_crud_cross_process.py:1` | 25 | no | `test_mem_delete_by_source_does_not_resurrect_a_removed_subdir` `sources/memtomem-memory/packages/memtomem/tests/test_memory_crud_cross_process.py:894`; `test_a_removed_subdir_delete_still_serializes_in_process` `sources/memtomem-memory/packages/memtomem/tests/test_memory_crud_cross_process.py:973` |
| `sources/memtomem-memory/packages/memtomem/tests/test_memory_write_scope.py:1` | 38 | no | `test_mem_delete_by_namespace_with_project_shared_chunk_rejects_without_confirm` `sources/memtomem-memory/packages/memtomem/tests/test_memory_write_scope.py:807`; `test_user_tier_chunk_delete_records_no_consent` `sources/memtomem-memory/packages/memtomem/tests/test_memory_write_scope.py:1442`; `test_user_only_namespace_delete_records_no_consent` `sources/memtomem-memory/packages/memtomem/tests/test_memory_write_scope.py:1460`; `test_confirmed_namespace_delete_records_one_consent` `sources/memtomem-memory/packages/memtomem/tests/test_memory_write_scope.py:1487`; `test_user_only_source_delete_records_no_consent` `sources/memtomem-memory/packages/memtomem/tests/test_memory_write_scope.py:1505`; `test_confirmed_source_delete_records_one_consent` `sources/memtomem-memory/packages/memtomem/tests/test_memory_write_scope.py:1539` |
| `sources/memtomem-memory/packages/memtomem/tests/test_mutation_reindex_gate.py:1` | 29 | no | `test_mem_delete` `sources/memtomem-memory/packages/memtomem/tests/test_mutation_reindex_gate.py:69` |
| `sources/memtomem-memory/packages/memtomem/tests/test_mutation_reindex_gate_web.py:1` | 6 | no | `test_chunk_delete_refuses_untouched` `sources/memtomem-memory/packages/memtomem/tests/test_mutation_reindex_gate_web.py:91` |
| `sources/memtomem-memory/packages/memtomem/tests/test_namespace_writer_transactions.py:1` | 10 | no | `test_outer_transaction_commits_successful_delete` `sources/memtomem-memory/packages/memtomem/tests/test_namespace_writer_transactions.py:12`; `test_outer_transaction_abort_restores_successful_delete` `sources/memtomem-memory/packages/memtomem/tests/test_namespace_writer_transactions.py:22`; `test_failed_delete_restores_rows_and_preserves_callers_write` `sources/memtomem-memory/packages/memtomem/tests/test_namespace_writer_transactions.py:34` |
| `sources/memtomem-memory/packages/memtomem/tests/test_nested_worktree_exclusion.py:1` | 66 | no | `test_purge_claims_an_unowned_worktree_row_and_nothing_beside_it` `sources/memtomem-memory/packages/memtomem/tests/test_nested_worktree_exclusion.py:656`; `test_purge_classifies_past_an_unresolvable_row_and_reports_it` `sources/memtomem-memory/packages/memtomem/tests/test_nested_worktree_exclusion.py:1039` |
| `sources/memtomem-memory/packages/memtomem/tests/test_norm_path_symlink_loop.py:1` | 3 | no | `test_delete_by_source_removes_the_rows_of_a_loop` `sources/memtomem-memory/packages/memtomem/tests/test_norm_path_symlink_loop.py:72` |
| `sources/memtomem-memory/packages/memtomem/tests/test_orphan_detect.py:1` | 22 | no | `test_permission_error_is_unavailable_not_purge_candidate` `sources/memtomem-memory/packages/memtomem/tests/test_orphan_detect.py:55`; `test_many_orphans_still_deleted` `sources/memtomem-memory/packages/memtomem/tests/test_orphan_detect.py:150`; `test_under_limits_the_purge` `sources/memtomem-memory/packages/memtomem/tests/test_orphan_detect.py:351` |
| `sources/memtomem-memory/packages/memtomem/tests/test_orphan_gc.py:1` | 30 | no | `test_dry_run_default_does_not_delete` `sources/memtomem-memory/packages/memtomem/tests/test_orphan_gc.py:419`; `test_apply_yes_deletes_non_interactively` `sources/memtomem-memory/packages/memtomem/tests/test_orphan_gc.py:457`; `test_dry_run_lists_without_deleting` `sources/memtomem-memory/packages/memtomem/tests/test_orphan_gc.py:507`; `test_abort_after_an_accepted_group_still_invalidates` `sources/memtomem-memory/packages/memtomem/tests/test_orphan_gc.py:596`; `test_apply_yes_deletes_and_invalidates` `sources/memtomem-memory/packages/memtomem/tests/test_orphan_gc.py:641`; `test_under_matching_nothing_deletes_nothing` `sources/memtomem-memory/packages/memtomem/tests/test_orphan_gc.py:681`; `test_a_retained_source_is_kept_out_of_the_purge` `sources/memtomem-memory/packages/memtomem/tests/test_orphan_gc.py:699` |
| `sources/memtomem-memory/packages/memtomem/tests/test_pinned_context.py:1` | 30 | no | `test_delete_is_exact_and_confirmed_for_shared` `sources/memtomem-memory/packages/memtomem/tests/test_pinned_context.py:395`; `test_mcp_pinned_delete_consent_names_the_tool` `sources/memtomem-memory/packages/memtomem/tests/test_pinned_context.py:586` |
| `sources/memtomem-memory/packages/memtomem/tests/test_pipeline_replay_mode.py:1` | 5 | no | `test_a_visibility_change_still_invalidates_under_record_false` `sources/memtomem-memory/packages/memtomem/tests/test_pipeline_replay_mode.py:94` |
| `sources/memtomem-memory/packages/memtomem/tests/test_policies.py:1` | 8 | no | `test_delete` `sources/memtomem-memory/packages/memtomem/tests/test_policies.py:34`; `test_delete_nonexistent` `sources/memtomem-memory/packages/memtomem/tests/test_policies.py:43` |
| `sources/memtomem-memory/packages/memtomem/tests/test_policy_scheduler.py:1` | 16 | no | `test_cache_invalidated_when_affected` `sources/memtomem-memory/packages/memtomem/tests/test_policy_scheduler.py:87`; `test_cache_not_invalidated_when_no_changes` `sources/memtomem-memory/packages/memtomem/tests/test_policy_scheduler.py:101`; `test_cache_invalidated_when_mutated_without_affected_count` `sources/memtomem-memory/packages/memtomem/tests/test_policy_scheduler.py:115`; `test_cache_invalidated_when_run_raises` `sources/memtomem-memory/packages/memtomem/tests/test_policy_scheduler.py:132`; `test_cache_invalidated_when_run_is_cancelled` `sources/memtomem-memory/packages/memtomem/tests/test_policy_scheduler.py:149`; `test_named_run_invalidates_before_bookkeeping_write` `sources/memtomem-memory/packages/memtomem/tests/test_policy_scheduler.py:257`; `test_all_enabled_invalidates_when_run_raises` `sources/memtomem-memory/packages/memtomem/tests/test_policy_scheduler.py:291`; `test_dry_run_failure_does_not_invalidate` `sources/memtomem-memory/packages/memtomem/tests/test_policy_scheduler.py:307`; `test_cancellation_invalidates` `sources/memtomem-memory/packages/memtomem/tests/test_policy_scheduler.py:323` |
| `sources/memtomem-memory/packages/memtomem/tests/test_project_bind.py:1` | 28 | no | `test_edit_and_delete_refuse_a_row_whose_bound_tier_was_unregistered` `sources/memtomem-memory/packages/memtomem/tests/test_project_bind.py:486` |
| `sources/memtomem-memory/packages/memtomem/tests/test_purge_cli.py:1` | 19 | no | `test_dry_run_does_not_delete` `sources/memtomem-memory/packages/memtomem/tests/test_purge_cli.py:126`; `test_apply_deletes` `sources/memtomem-memory/packages/memtomem/tests/test_purge_cli.py:139` |
| `sources/memtomem-memory/packages/memtomem/tests/test_qa_audit_pins.py:1` | 29 | no | `test_delete_namespace_returns_404_in_prod` `sources/memtomem-memory/packages/memtomem/tests/test_qa_audit_pins.py:86`; `test_delete_namespace_reachable_in_dev` `sources/memtomem-memory/packages/memtomem/tests/test_qa_audit_pins.py:96` |
| `sources/memtomem-memory/packages/memtomem/tests/test_quality_fixes.py:1` | 38 | no | `test_delete_tag_batch` `sources/memtomem-memory/packages/memtomem/tests/test_quality_fixes.py:189`; `test_delete_relation` `sources/memtomem-memory/packages/memtomem/tests/test_quality_fixes.py:385` |
| `sources/memtomem-memory/packages/memtomem/tests/test_read_only_root_write_paths.py:1` | 33 | no | `test_pinned_set_and_delete_refuse_a_protected_destination` `sources/memtomem-memory/packages/memtomem/tests/test_read_only_root_write_paths.py:573` |
| `sources/memtomem-memory/packages/memtomem/tests/test_redaction_held_id_access.py:1` | 21 | no | `test_mem_delete_and_mem_edit_leave_file_and_hold_in_place` `sources/memtomem-memory/packages/memtomem/tests/test_redaction_held_id_access.py:329` |
| `sources/memtomem-memory/packages/memtomem/tests/test_relinked_keys_at_start.py:1` | 20 | no | `test_a_purge_still_keeps_every_retained_key_including_a_dangling_link` `sources/memtomem-memory/packages/memtomem/tests/test_relinked_keys_at_start.py:524`; `test_a_retained_key_that_flickers_present_is_never_a_purge_candidate` `sources/memtomem-memory/packages/memtomem/tests/test_relinked_keys_at_start.py:541` |
| `sources/memtomem-memory/packages/memtomem/tests/test_retain_missing_dirs.py:1` | 24 | no | `test_a_deleted_file_under_a_retain_dir_stays_searchable` `sources/memtomem-memory/packages/memtomem/tests/test_retain_missing_dirs.py:161`; `test_a_deleted_file_outside_every_retain_dir_is_held` `sources/memtomem-memory/packages/memtomem/tests/test_retain_missing_dirs.py:174`; `test_purge_neither_lists_nor_deletes_a_retained_source` `sources/memtomem-memory/packages/memtomem/tests/test_retain_missing_dirs.py:290`; `test_purge_of_a_relinked_source_keeps_the_archive_it_points_at` `sources/memtomem-memory/packages/memtomem/tests/test_retain_missing_dirs.py:354`; `test_a_confirmed_purge_also_reports_what_it_kept` `sources/memtomem-memory/packages/memtomem/tests/test_retain_missing_dirs.py:393` |
| `sources/memtomem-memory/packages/memtomem/tests/test_schedule_cmd.py:1` | 9 | no | `test_add_list_delete_roundtrip` `sources/memtomem-memory/packages/memtomem/tests/test_schedule_cmd.py:138` |
| `sources/memtomem-memory/packages/memtomem/tests/test_scheduler_jobs.py:1` | 13 | no | `test_transient_absence_not_deleted` `sources/memtomem-memory/packages/memtomem/tests/test_scheduler_jobs.py:177`; `test_mass_orphan_event_skips_delete` `sources/memtomem-memory/packages/memtomem/tests/test_scheduler_jobs.py:193` |
| `sources/memtomem-memory/packages/memtomem/tests/test_schedules_store.py:1` | 17 | no | `test_delete` `sources/memtomem-memory/packages/memtomem/tests/test_schedules_store.py:40`; `test_delete_missing_returns_false` `sources/memtomem-memory/packages/memtomem/tests/test_schedules_store.py:46` |
| `sources/memtomem-memory/packages/memtomem/tests/test_scope_loss_reindex.py:1` | 14 | no | `test_an_emptied_file_still_deletes_and_escalation_is_not_judged` `sources/memtomem-memory/packages/memtomem/tests/test_scope_loss_reindex.py:224`; `test_web_edit_and_delete_refuse_after_the_tier_is_unregistered` `sources/memtomem-memory/packages/memtomem/tests/test_scope_loss_reindex.py:331` |
| `sources/memtomem-memory/packages/memtomem/tests/test_scratch.py:1` | 12 | no | `test_delete` `sources/memtomem-memory/packages/memtomem/tests/test_scratch.py:28`; `test_delete_expected_value_takes_only_the_value_read` `sources/memtomem-memory/packages/memtomem/tests/test_scratch.py:35` |
| `sources/memtomem-memory/packages/memtomem/tests/test_search_history.py:1` | 15 | no | `test_a_failed_save_keeps_the_rows_its_prune_deleted` `sources/memtomem-memory/packages/memtomem/tests/test_search_history.py:245` |
| `sources/memtomem-memory/packages/memtomem/tests/test_search_observation.py:1` | 23 | no | `test_legacy_backend_keeps_fire_and_forget_and_skips_cache_hit_history` `sources/memtomem-memory/packages/memtomem/tests/test_search_observation.py:215` |
| `sources/memtomem-memory/packages/memtomem/tests/test_server_tools_advanced.py:1` | 78 | no | `test_delete_relation` `sources/memtomem-memory/packages/memtomem/tests/test_server_tools_advanced.py:84`; `test_delete_nonexistent_relation` `sources/memtomem-memory/packages/memtomem/tests/test_server_tools_advanced.py:96`; `test_delete_policy` `sources/memtomem-memory/packages/memtomem/tests/test_server_tools_advanced.py:159`; `test_delete_nonexistent_policy` `sources/memtomem-memory/packages/memtomem/tests/test_server_tools_advanced.py:167`; `test_delete_entities_for_chunk` `sources/memtomem-memory/packages/memtomem/tests/test_server_tools_advanced.py:287`; `test_delete_by_source` `sources/memtomem-memory/packages/memtomem/tests/test_server_tools_advanced.py:701`; `test_auto_tag_storage_invalidates_when_it_writes` `sources/memtomem-memory/packages/memtomem/tests/test_server_tools_advanced.py:799`; `test_auto_tag_storage_does_not_invalidate_without_a_write` `sources/memtomem-memory/packages/memtomem/tests/test_server_tools_advanced.py:815`; `test_auto_tag_storage_invalidates_after_a_partial_failure` `sources/memtomem-memory/packages/memtomem/tests/test_server_tools_advanced.py:831` |
| `sources/memtomem-memory/packages/memtomem/tests/test_server_tools_context_version.py:1` | 52 | no | `test_delete_label` `sources/memtomem-memory/packages/memtomem/tests/test_server_tools_context_version.py:345`; `test_delete_absent_label_is_noop` `sources/memtomem-memory/packages/memtomem/tests/test_server_tools_context_version.py:355`; `test_delete_with_version_is_contradiction` `sources/memtomem-memory/packages/memtomem/tests/test_server_tools_context_version.py:395`; `test_delete_label_refuses_without_mutating` `sources/memtomem-memory/packages/memtomem/tests/test_server_tools_context_version.py:638` |
| `sources/memtomem-memory/packages/memtomem/tests/test_server_tools_core.py:1` | 72 | no | `test_upsert_then_delete_by_chunk_id` `sources/memtomem-memory/packages/memtomem/tests/test_server_tools_core.py:406`; `test_delete_by_source` `sources/memtomem-memory/packages/memtomem/tests/test_server_tools_core.py:415`; `test_delete_by_namespace` `sources/memtomem-memory/packages/memtomem/tests/test_server_tools_core.py:428`; `test_delete_nonexistent_chunk` `sources/memtomem-memory/packages/memtomem/tests/test_server_tools_core.py:439` |
| `sources/memtomem-memory/packages/memtomem/tests/test_server_tools_hermes_mirror.py:1` | 27 | yes | `test_purge_empties_the_file_and_fences_older_operations` `sources/memtomem-memory/packages/memtomem/tests/test_server_tools_hermes_mirror.py:138`; `test_purge_needs_a_ts_from_the_provider_clock` `sources/memtomem-memory/packages/memtomem/tests/test_server_tools_hermes_mirror.py:150`; `test_reconcile_repairs_a_deleted_or_edited_file` `sources/memtomem-memory/packages/memtomem/tests/test_server_tools_hermes_mirror.py:161` |
| `sources/memtomem-memory/packages/memtomem/tests/test_server_tools_indexing.py:1` | 18 | no | `test_mutated_run_invalidates` `sources/memtomem-memory/packages/memtomem/tests/test_server_tools_indexing.py:350`; `test_metadata_only_run_invalidates_despite_silent_counters` `sources/memtomem-memory/packages/memtomem/tests/test_server_tools_indexing.py:367`; `test_unmutated_run_does_not_invalidate_even_with_counters_set` `sources/memtomem-memory/packages/memtomem/tests/test_server_tools_indexing.py:388`; `test_invalidates_before_provenance_so_cancellation_cannot_skip_it` `sources/memtomem-memory/packages/memtomem/tests/test_server_tools_indexing.py:408`; `test_steady_state_run_does_not_invalidate` `sources/memtomem-memory/packages/memtomem/tests/test_server_tools_indexing.py:464` |
| `sources/memtomem-memory/packages/memtomem/tests/test_server_tools_ns_assign.py:1` | 24 | no | `test_merge_remaps_references_and_deletes_loser_sidecars` `sources/memtomem-memory/packages/memtomem/tests/test_server_tools_ns_assign.py:159` |
| `sources/memtomem-memory/packages/memtomem/tests/test_server_tools_org.py:1` | 101 | no | `test_delete_by_namespace` `sources/memtomem-memory/packages/memtomem/tests/test_server_tools_org.py:58`; `test_delete_namespace_empty` `sources/memtomem-memory/packages/memtomem/tests/test_server_tools_org.py:71`; `test_duplicate_deletes_are_split_into_batches` `sources/memtomem-memory/packages/memtomem/tests/test_server_tools_org.py:678`; `test_multi_batch_delete_leaves_one_copy_of_each` `sources/memtomem-memory/packages/memtomem/tests/test_server_tools_org.py:705`; `test_delete_tag` `sources/memtomem-memory/packages/memtomem/tests/test_server_tools_org.py:836`; `test_delete_tag_preserves_chunks` `sources/memtomem-memory/packages/memtomem/tests/test_server_tools_org.py:846`; `test_delete_tag_nonexistent` `sources/memtomem-memory/packages/memtomem/tests/test_server_tools_org.py:855`; `test_delete` `sources/memtomem-memory/packages/memtomem/tests/test_server_tools_org.py:982`; `test_delete_nonexistent` `sources/memtomem-memory/packages/memtomem/tests/test_server_tools_org.py:988` |
| `sources/memtomem-memory/packages/memtomem/tests/test_services_tag_management.py:1` | 33 | no | `test_rename_apply_mutates_and_invalidates_cache` `sources/memtomem-memory/packages/memtomem/tests/test_services_tag_management.py:57`; `test_rename_apply_no_match_skips_invalidate` `sources/memtomem-memory/packages/memtomem/tests/test_services_tag_management.py:74`; `test_delete_dry_run_returns_count_without_writing` `sources/memtomem-memory/packages/memtomem/tests/test_services_tag_management.py:150`; `test_delete_apply_mutates_and_invalidates` `sources/memtomem-memory/packages/memtomem/tests/test_services_tag_management.py:166`; `test_delete_rejects_empty_name` `sources/memtomem-memory/packages/memtomem/tests/test_services_tag_management.py:181`; `test_delete_rejects_whitespace_only_name` `sources/memtomem-memory/packages/memtomem/tests/test_services_tag_management.py:187`; `test_merge_apply_mutates_and_invalidates` `sources/memtomem-memory/packages/memtomem/tests/test_services_tag_management.py:224`; `test_replace_chunk_tags_mutates_and_invalidates` `sources/memtomem-memory/packages/memtomem/tests/test_services_tag_management.py:387`; `test_replace_chunk_tags_does_not_resurrect_chunk_deleted_after_prefetch` `sources/memtomem-memory/packages/memtomem/tests/test_services_tag_management.py:420`; `test_delete_preserves_content_embedding_hash_created_at` `sources/memtomem-memory/packages/memtomem/tests/test_services_tag_management.py:562` |
| `sources/memtomem-memory/packages/memtomem/tests/test_sessions.py:1` | 107 | no | `test_deleted_provenance_id_shortfall_falls_back` `sources/memtomem-memory/packages/memtomem/tests/test_sessions.py:1472` |
| `sources/memtomem-memory/packages/memtomem/tests/test_settings_copy.py:1` | 37 | no | `test_canonical_delete_race_aborts_and_does_not_recreate` `sources/memtomem-memory/packages/memtomem/tests/test_settings_copy.py:498`; `test_tier_delete_race_keeps_canonical_and_does_not_recreate` `sources/memtomem-memory/packages/memtomem/tests/test_settings_copy.py:523` |
| `sources/memtomem-memory/packages/memtomem/tests/test_settings_migrate.py:1` | 72 | no | `test_all_exact_batch_aborts_on_target_deletion` `sources/memtomem-memory/packages/memtomem/tests/test_settings_migrate.py:1800` |
| `sources/memtomem-memory/packages/memtomem/tests/test_skills_routes_abandon.py:1` | 5 | no | `test_a_cancelled_delete_removes_nothing` `sources/memtomem-memory/packages/memtomem/tests/test_skills_routes_abandon.py:253` |
| `sources/memtomem-memory/packages/memtomem/tests/test_source_holds.py:1` | 32 | no | `test_pending_delete_hidden_and_replayed_after_restart` `sources/memtomem-memory/packages/memtomem/tests/test_source_holds.py:34`; `test_external_hold_invalidates_warm_search_cache` `sources/memtomem-memory/packages/memtomem/tests/test_source_holds.py:60`; `test_delete_journal_keeps_nfd_path_on_nonfolding_platform` `sources/memtomem-memory/packages/memtomem/tests/test_source_holds.py:81`; `test_policy_summary_tracks_source_hold_and_explicit_purge` `sources/memtomem-memory/packages/memtomem/tests/test_source_holds.py:186`; `test_last_parent_chunk_delete_keeps_only_a_marked_summary` `sources/memtomem-memory/packages/memtomem/tests/test_source_holds.py:292`; `test_explicit_purge_still_removes_a_held_summary` `sources/memtomem-memory/packages/memtomem/tests/test_source_holds.py:318`; `test_last_chunk_delete_clears_hold_before_direct_upsert` `sources/memtomem-memory/packages/memtomem/tests/test_source_holds.py:347`; `test_explicit_source_delete_clears_hold_and_pending` `sources/memtomem-memory/packages/memtomem/tests/test_source_holds.py:426`; `test_new_delete_journal_cannot_be_released_by_older_index_pass` `sources/memtomem-memory/packages/memtomem/tests/test_source_holds.py:485` |
| `sources/memtomem-memory/packages/memtomem/tests/test_sqlite_concurrency_phase2.py:1` | 34 | no | `test_delete_by_namespace_is_refused_while_a_grant_is_pending` `sources/memtomem-memory/packages/memtomem/tests/test_sqlite_concurrency_phase2.py:396` |
| `sources/memtomem-memory/packages/memtomem/tests/test_storage.py:1` | 51 | no | `test_delete_chunks` `sources/memtomem-memory/packages/memtomem/tests/test_storage.py:31`; `test_delete_tag` `sources/memtomem-memory/packages/memtomem/tests/test_storage.py:813`; `test_delete_tag_bumps_updated_at_symmetric` `sources/memtomem-memory/packages/memtomem/tests/test_storage.py:970`; `test_delete_relation` `sources/memtomem-memory/packages/memtomem/tests/test_storage.py:1053` |
| `sources/memtomem-memory/packages/memtomem/tests/test_storage_extended.py:1` | 103 | no | `test_reset_all_deletes_all_chunks` `sources/memtomem-memory/packages/memtomem/tests/test_storage_extended.py:619`; `test_delete_ai_summary_clears_existing_row` `sources/memtomem-memory/packages/memtomem/tests/test_storage_extended.py:890`; `test_delete_ai_summary_missing_is_noop` `sources/memtomem-memory/packages/memtomem/tests/test_storage_extended.py:905`; `test_delete_by_source_clears_ai_summary` `sources/memtomem-memory/packages/memtomem/tests/test_storage_extended.py:1042`; `test_delete_by_source_clears_orphan_summary` `sources/memtomem-memory/packages/memtomem/tests/test_storage_extended.py:1074`; `test_delete_chunks_clears_summary_when_source_emptied` `sources/memtomem-memory/packages/memtomem/tests/test_storage_extended.py:1087`; `test_delete_chunks_preserves_summary_on_partial_delete` `sources/memtomem-memory/packages/memtomem/tests/test_storage_extended.py:1118`; `test_delete_chunks_in_transaction_preserves_summary_for_rewrite` `sources/memtomem-memory/packages/memtomem/tests/test_storage_extended.py:1152`; `test_delete_chunks_in_transaction_skips_cleanup` `sources/memtomem-memory/packages/memtomem/tests/test_storage_extended.py:1194`; `test_delete_chunks_handles_mixed_sources_atomically` `sources/memtomem-memory/packages/memtomem/tests/test_storage_extended.py:1225`; `test_deletes_metadata_only_namespace` `sources/memtomem-memory/packages/memtomem/tests/test_storage_extended.py:1326`; `test_deletes_chunks_and_metadata` `sources/memtomem-memory/packages/memtomem/tests/test_storage_extended.py:1346` |
| `sources/memtomem-memory/packages/memtomem/tests/test_storage_noop.py:1` | 3 | no | `test_delete_paths_noop_mode` `sources/memtomem-memory/packages/memtomem/tests/test_storage_noop.py:84` |
| `sources/memtomem-memory/packages/memtomem/tests/test_storage_param_batching.py:1` | 23 | no | `test_deletes_a_source_past_the_bound_variable_ceiling` `sources/memtomem-memory/packages/memtomem/tests/test_storage_param_batching.py:159`; `test_deletes_an_id_list_past_the_bound_variable_ceiling` `sources/memtomem-memory/packages/memtomem/tests/test_storage_param_batching.py:206`; `test_the_lookup_and_the_delete_are_both_split` `sources/memtomem-memory/packages/memtomem/tests/test_storage_param_batching.py:217`; `test_deletes_a_namespace_past_the_bound_variable_ceiling` `sources/memtomem-memory/packages/memtomem/tests/test_storage_param_batching.py:347`; `test_splits_at_the_namespace_delete_batch` `sources/memtomem-memory/packages/memtomem/tests/test_storage_param_batching.py:358` |
| `sources/memtomem-memory/packages/memtomem/tests/test_storage_transactions.py:1` | 44 | no | `test_idempotency_claim_rolls_back_its_purge` `sources/memtomem-memory/packages/memtomem/tests/test_storage_transactions.py:684`; `test_cleanup_old_sessions_rolls_back_a_failed_delete` `sources/memtomem-memory/packages/memtomem/tests/test_storage_transactions.py:709`; `test_delete_by_source_empty_branch_defers_to_owner` `sources/memtomem-memory/packages/memtomem/tests/test_storage_transactions.py:843`; `test_delete_by_source_empty_branch_failure_closes_its_transaction` `sources/memtomem-memory/packages/memtomem/tests/test_storage_transactions.py:1015` |
| `sources/memtomem-memory/packages/memtomem/tests/test_stored_key_dir_remove.py:1` | 5 | no | `test_removing_the_link_target_root_deletes_only_its_own_rows` `sources/memtomem-memory/packages/memtomem/tests/test_stored_key_dir_remove.py:109`; `test_removing_the_link_root_deletes_the_link_key_not_its_target` `sources/memtomem-memory/packages/memtomem/tests/test_stored_key_dir_remove.py:122` |
| `sources/memtomem-memory/packages/memtomem/tests/test_stored_source_keys.py:1` | 33 | no | `test_dangling_relink_deletes_each_source_own_rows` `sources/memtomem-memory/packages/memtomem/tests/test_stored_source_keys.py:78`; `test_live_relink_deletes_the_link_rows_not_the_target` `sources/memtomem-memory/packages/memtomem/tests/test_stored_source_keys.py:87`; `test_delete_clears_the_stored_key_side_rows_only` `sources/memtomem-memory/packages/memtomem/tests/test_stored_source_keys.py:104`; `test_gc_orphan_sources_deletes_both_sides_of_a_dangling_relink` `sources/memtomem-memory/packages/memtomem/tests/test_stored_source_keys.py:305`; `test_purge_skips_a_relinked_key_and_deletes_matches_by_their_key` `sources/memtomem-memory/packages/memtomem/tests/test_stored_source_keys.py:318`; `test_mem_delete_refuses_a_stored_key_that_now_reaches_another_source` `sources/memtomem-memory/packages/memtomem/tests/test_stored_source_keys.py:350`; `test_mem_delete_refusal_creates_no_sidecar_beside_an_outside_target` `sources/memtomem-memory/packages/memtomem/tests/test_stored_source_keys.py:361`; `test_mem_delete_refuses_an_outside_root_key_that_links_into_a_root` `sources/memtomem-memory/packages/memtomem/tests/test_stored_source_keys.py:431`; `test_mem_delete_removes_a_legacy_nfd_key_by_that_key` `sources/memtomem-memory/packages/memtomem/tests/test_stored_source_keys.py:444`; `test_mem_delete_refuses_a_relative_name_for_a_relinked_key` `sources/memtomem-memory/packages/memtomem/tests/test_stored_source_keys.py:459`; `test_mem_delete_refuses_another_unicode_spelling_of_a_relinked_key` `sources/memtomem-memory/packages/memtomem/tests/test_stored_source_keys.py:471`; `test_mem_delete_refuses_a_case_variant_of_a_relinked_key` `sources/memtomem-memory/packages/memtomem/tests/test_stored_source_keys.py:489`; `test_mem_delete_of_an_unindexed_path_is_an_error` `sources/memtomem-memory/packages/memtomem/tests/test_stored_source_keys.py:504`; `test_mem_delete_refuses_a_relative_name_under_a_root_that_became_a_link` `sources/memtomem-memory/packages/memtomem/tests/test_stored_source_keys.py:512` |
| `sources/memtomem-memory/packages/memtomem/tests/test_tools_logic.py:1` | 142 | no | `test_auto_expire_row_records_deleted_ids_and_namespaces` `sources/memtomem-memory/packages/memtomem/tests/test_tools_logic.py:296`; `test_regeneration_run_records_deleted_and_new_summary_ids` `sources/memtomem-memory/packages/memtomem/tests/test_tools_logic.py:1376` |
| `sources/memtomem-memory/packages/memtomem/tests/test_uninstall_cmd.py:1` | 154 | no | `test_custom_storage_path_in_inventory_and_deleted` `sources/memtomem-memory/packages/memtomem/tests/test_uninstall_cmd.py:221`; `test_user_managed_memory_dirs_never_deleted` `sources/memtomem-memory/packages/memtomem/tests/test_uninstall_cmd.py:253`; `test_runtime_pid_deleted_and_lock_infrastructure_retained` `sources/memtomem-memory/packages/memtomem/tests/test_uninstall_cmd.py:996`; `test_stale_sentinels_deleted_and_dir_pruned` `sources/memtomem-memory/packages/memtomem/tests/test_uninstall_cmd.py:1940`; `test_registry_going_live_between_probe_and_delete_refuses` `sources/memtomem-memory/packages/memtomem/tests/test_uninstall_cmd.py:2124`; `test_server_going_live_between_probe_and_delete_refuses` `sources/memtomem-memory/packages/memtomem/tests/test_uninstall_cmd.py:2243`; `test_db_lock_going_live_between_probe_and_delete_refuses` `sources/memtomem-memory/packages/memtomem/tests/test_uninstall_cmd.py:2323`; `test_only_a_dangling_link_is_not_nothing_to_delete` `sources/memtomem-memory/packages/memtomem/tests/test_uninstall_cmd.py:2564`; `test_live_link_to_empty_dir_is_not_nothing_to_delete` `sources/memtomem-memory/packages/memtomem/tests/test_uninstall_cmd.py:2624`; `test_empty_owned_dir_alone_is_deleted_not_nothing` `sources/memtomem-memory/packages/memtomem/tests/test_uninstall_cmd.py:2647`; `test_shared_holder_refuses_deletion` `sources/memtomem-memory/packages/memtomem/tests/test_uninstall_cmd.py:2906`; `test_released_after_successful_delete` `sources/memtomem-memory/packages/memtomem/tests/test_uninstall_cmd.py:3105`; `test_dangling_state_dir_link_reports_nothing_to_delete` `sources/memtomem-memory/packages/memtomem/tests/test_uninstall_cmd.py:3342`; `test_squatter_alone_is_not_nothing_to_delete` `sources/memtomem-memory/packages/memtomem/tests/test_uninstall_cmd.py:3428` |
| `sources/memtomem-memory/packages/memtomem/tests/test_upgrade_cmd.py:1` | 64 | no | `test_pid_file_gone_before_the_delete_is_still_reported_removed` `sources/memtomem-memory/packages/memtomem/tests/test_upgrade_cmd.py:1433`; `test_pid_file_delete_failure_is_reported` `sources/memtomem-memory/packages/memtomem/tests/test_upgrade_cmd.py:1463` |
| `sources/memtomem-memory/packages/memtomem/tests/test_watcher_deleted_files.py:1` | 23 | no | `test_on_deleted_enqueues_md` `sources/memtomem-memory/packages/memtomem/tests/test_watcher_deleted_files.py:84`; `test_on_deleted_filters_unsupported_suffix` `sources/memtomem-memory/packages/memtomem/tests/test_watcher_deleted_files.py:90`; `test_on_deleted_ignores_directory` `sources/memtomem-memory/packages/memtomem/tests/test_watcher_deleted_files.py:96`; `test_deleted_file_holds_its_chunks` `sources/memtomem-memory/packages/memtomem/tests/test_watcher_deleted_files.py:166`; `test_transient_oserror_does_not_delete` `sources/memtomem-memory/packages/memtomem/tests/test_watcher_deleted_files.py:179`; `test_unmounted_root_does_not_delete` `sources/memtomem-memory/packages/memtomem/tests/test_watcher_deleted_files.py:198`; `test_subdir_deletion_holds_chunks_without_resurrecting_dir` `sources/memtomem-memory/packages/memtomem/tests/test_watcher_deleted_files.py:211`; `test_deleted_excluded_path_is_still_held` `sources/memtomem-memory/packages/memtomem/tests/test_watcher_deleted_files.py:248`; `test_present_excluded_path_is_not_indexed_or_purged` `sources/memtomem-memory/packages/memtomem/tests/test_watcher_deleted_files.py:264`; `test_file_replaced_by_directory_purges_chunks` `sources/memtomem-memory/packages/memtomem/tests/test_watcher_deleted_files.py:280`; `test_excluded_file_replaced_by_directory_still_purges` `sources/memtomem-memory/packages/memtomem/tests/test_watcher_deleted_files.py:294` |
| `sources/memtomem-memory/packages/memtomem/tests/test_watcher_overflow.py:1` | 18 | no | `test_delete_dropped_while_the_loop_is_held_off_is_hidden` `sources/memtomem-memory/packages/memtomem/tests/test_watcher_overflow.py:427`; `test_stop_replays_a_dropped_delete_before_exiting` `sources/memtomem-memory/packages/memtomem/tests/test_watcher_overflow.py:543` |
| `sources/memtomem-memory/packages/memtomem/tests/test_web_csrf_middleware.py:1` | 26 | no | `test_delete_without_token_returns_403` `sources/memtomem-memory/packages/memtomem/tests/test_web_csrf_middleware.py:257` |
| `sources/memtomem-memory/packages/memtomem/tests/test_web_routes.py:1` | 360 | no | `test_delete_chunk` `sources/memtomem-memory/packages/memtomem/tests/test_web_routes.py:2748`; `test_delete_preflights_the_namespace_without_pinning_it` `sources/memtomem-memory/packages/memtomem/tests/test_web_routes.py:2784`; `test_delete_refuses_on_a_mixed_namespace_source` `sources/memtomem-memory/packages/memtomem/tests/test_web_routes.py:2812`; `test_delete_refuses_when_the_namespace_lookup_fails` `sources/memtomem-memory/packages/memtomem/tests/test_web_routes.py:2842`; `test_delete_refuses_unusable_source_provenance_without_index_fallback` `sources/memtomem-memory/packages/memtomem/tests/test_web_routes.py:2868`; `test_delete_refuses_source_stat_error_without_index_fallback` `sources/memtomem-memory/packages/memtomem/tests/test_web_routes.py:2892`; `test_delete_refuses_source_write_error_without_index_fallback` `sources/memtomem-memory/packages/memtomem/tests/test_web_routes.py:2913`; `test_delete_verifies_and_cleans_the_row_after_reindex_failure` `sources/memtomem-memory/packages/memtomem/tests/test_web_routes.py:2931`; `test_delete_reports_partial_failure_when_index_cleanup_does_not_finish` `sources/memtomem-memory/packages/memtomem/tests/test_web_routes.py:2953`; `test_delete_chunk_not_found` `sources/memtomem-memory/packages/memtomem/tests/test_web_routes.py:2967`; `test_delete_project_shared_chunk_requires_confirm` `sources/memtomem-memory/packages/memtomem/tests/test_web_routes.py:2973`; `test_delete_project_shared_chunk_with_confirm_proceeds` `sources/memtomem-memory/packages/memtomem/tests/test_web_routes.py:3024`; `test_tag_update_invalidates_search_cache` `sources/memtomem-memory/packages/memtomem/tests/test_web_routes.py:3704`; `test_tag_update_no_op_skips_write_and_invalidate` `sources/memtomem-memory/packages/memtomem/tests/test_web_routes.py:3735`; `test_delete_source_matches_nfd_indexed_with_nfc_query` `sources/memtomem-memory/packages/memtomem/tests/test_web_routes.py:5007`; `test_default_does_not_delete_chunks` `sources/memtomem-memory/packages/memtomem/tests/test_web_routes.py:6192`; `test_delete_chunks_true_removes_matching_source_files` `sources/memtomem-memory/packages/memtomem/tests/test_web_routes.py:6211`; `test_sweep_deletes_what_the_status_count_showed` `sources/memtomem-memory/packages/memtomem/tests/test_web_routes.py:6366`; `test_delete_chunk_does_not_resurrect_a_removed_parent_dir` `sources/memtomem-memory/packages/memtomem/tests/test_web_routes.py:7804`; `test_delete_chunk_takes_the_index_only_branch_when_degraded` `sources/memtomem-memory/packages/memtomem/tests/test_web_routes.py:7884`; `test_delete_chunk_returns_503_when_sidecar_held` `sources/memtomem-memory/packages/memtomem/tests/test_web_routes.py:7923`; `test_delete_chunk_finishes_index_only_when_the_source_is_removed_mid_span` `sources/memtomem-memory/packages/memtomem/tests/test_web_routes.py:8033`; `test_delete_chunk_answers_409_when_the_source_is_replaced_mid_span` `sources/memtomem-memory/packages/memtomem/tests/test_web_routes.py:8064`; `test_delete_invalidates_search_cache` `sources/memtomem-memory/packages/memtomem/tests/test_web_routes.py:8193`; `test_user_scope_source_deletes_without_confirmation` `sources/memtomem-memory/packages/memtomem/tests/test_web_routes.py:8207`; `test_project_shared_with_flag_deletes` `sources/memtomem-memory/packages/memtomem/tests/test_web_routes.py:8238`; `test_scope_probe_runs_before_the_delete` `sources/memtomem-memory/packages/memtomem/tests/test_web_routes.py:8269`; `test_a_relinked_stored_key_is_probed_and_deleted_as_stored` `sources/memtomem-memory/packages/memtomem/tests/test_web_routes.py:8284`; `test_trigger_index_invalidates` `sources/memtomem-memory/packages/memtomem/tests/test_web_routes.py:8349`; `test_trigger_index_steady_state_does_not_invalidate` `sources/memtomem-memory/packages/memtomem/tests/test_web_routes.py:8357`; `test_reindex_all_invalidates_per_root` `sources/memtomem-memory/packages/memtomem/tests/test_web_routes.py:8366`; `test_index_stream_invalidates_even_without_a_complete_event` `sources/memtomem-memory/packages/memtomem/tests/test_web_routes.py:8375`; `test_add_memory_dir_auto_index_invalidates` `sources/memtomem-memory/packages/memtomem/tests/test_web_routes.py:8394`; `test_edit_chunk_invalidates_on_the_namespace_503_branch` `sources/memtomem-memory/packages/memtomem/tests/test_web_routes.py:8409`; `test_delete_chunk_invalidates` `sources/memtomem-memory/packages/memtomem/tests/test_web_routes.py:8431`; `test_delete_chunk_invalidates_when_verification_fails` `sources/memtomem-memory/packages/memtomem/tests/test_web_routes.py:8441`; `test_delete_chunk_refusal_inside_the_guarded_region_does_not_invalidate` `sources/memtomem-memory/packages/memtomem/tests/test_web_routes.py:8462`; `test_reindex_all_keeps_earlier_roots_invalidation_when_a_later_root_raises` `sources/memtomem-memory/packages/memtomem/tests/test_web_routes.py:8485`; `test_index_stream_invalidates_when_the_generator_is_closed_mid_run` `sources/memtomem-memory/packages/memtomem/tests/test_web_routes.py:8517` |
| `sources/memtomem-memory/packages/memtomem/tests/test_web_routes_context.py:1` | 135 | no | `test_delete_canonical_only` `sources/memtomem-memory/packages/memtomem/tests/test_web_routes_context.py:1039`; `test_delete_cascade` `sources/memtomem-memory/packages/memtomem/tests/test_web_routes_context.py:1046`; `test_delete_missing_is_idempotent` `sources/memtomem-memory/packages/memtomem/tests/test_web_routes_context.py:1060`; `test_create_update_delete` `sources/memtomem-memory/packages/memtomem/tests/test_web_routes_context.py:1684`; `test_cascade_deletes_runtime_only_command` `sources/memtomem-memory/packages/memtomem/tests/test_web_routes_context.py:1852`; `test_create_update_delete` `sources/memtomem-memory/packages/memtomem/tests/test_web_routes_context.py:2113`; `test_delete_rejects_non_shared` `sources/memtomem-memory/packages/memtomem/tests/test_web_routes_context.py:2484` |
| `sources/memtomem-memory/packages/memtomem/tests/test_web_routes_context_mcp_servers.py:1` | 17 | no | `test_delete_removed_entry_is_posix` `sources/memtomem-memory/packages/memtomem/tests/test_web_routes_context_mcp_servers.py:402`; `test_delete_skip_reason_is_path_free` `sources/memtomem-memory/packages/memtomem/tests/test_web_routes_context_mcp_servers.py:415` |
| `sources/memtomem-memory/packages/memtomem/tests/test_web_routes_context_mutations.py:1` | 30 | no | `test_install_project_root_deleted_is_404` `sources/memtomem-memory/packages/memtomem/tests/test_web_routes_context_mutations.py:193`; `test_update_project_root_deleted_is_404` `sources/memtomem-memory/packages/memtomem/tests/test_web_routes_context_mutations.py:226` |
| `sources/memtomem-memory/packages/memtomem/tests/test_web_routes_context_mutators.py:1` | 28 | no | `test_DELETE_holds_gateway_lock` `sources/memtomem-memory/packages/memtomem/tests/test_web_routes_context_mutators.py:420`; `test_DELETE_missing_is_idempotent` `sources/memtomem-memory/packages/memtomem/tests/test_web_routes_context_mutators.py:604`; `test_DELETE_existing_removes_canonical` `sources/memtomem-memory/packages/memtomem/tests/test_web_routes_context_mutators.py:617`; `test_dir_layout_delete_removes_dir_file_only` `sources/memtomem-memory/packages/memtomem/tests/test_web_routes_context_mutators.py:733` |
| `sources/memtomem-memory/packages/memtomem/tests/test_web_routes_context_projects.py:1` | 61 | no | `test_delete_round_trip` `sources/memtomem-memory/packages/memtomem/tests/test_web_routes_context_projects.py:848`; `test_delete_unknown_scope_id_404` `sources/memtomem-memory/packages/memtomem/tests/test_web_routes_context_projects.py:864`; `test_delete_removes_stale_entry` `sources/memtomem-memory/packages/memtomem/tests/test_web_routes_context_projects.py:870`; `test_delete_over_corrupt_store_500_preserves_bytes` `sources/memtomem-memory/packages/memtomem/tests/test_web_routes_context_projects.py:1111` |
| `sources/memtomem-memory/packages/memtomem/tests/test_web_routes_context_versions.py:1` | 68 | no | `test_delete_removes_pointer` `sources/memtomem-memory/packages/memtomem/tests/test_web_routes_context_versions.py:526`; `test_delete_absent_label_is_noop_200` `sources/memtomem-memory/packages/memtomem/tests/test_web_routes_context_versions.py:537`; `test_delete_reserved_label_latest_400` `sources/memtomem-memory/packages/memtomem/tests/test_web_routes_context_versions.py:545`; `test_delete_non_shared_tier_400` `sources/memtomem-memory/packages/memtomem/tests/test_web_routes_context_versions.py:551`; `test_delete_works_on_commands` `sources/memtomem-memory/packages/memtomem/tests/test_web_routes_context_versions.py:559`; `test_delete_threads_lock_budget_kwarg` `sources/memtomem-memory/packages/memtomem/tests/test_web_routes_context_versions.py:649`; `test_delete_label_409_and_writes_nothing` `sources/memtomem-memory/packages/memtomem/tests/test_web_routes_context_versions.py:894`; `test_delete_label_reports_read_only_not_tier` `sources/memtomem-memory/packages/memtomem/tests/test_web_routes_context_versions.py:972` |
| `sources/memtomem-memory/packages/memtomem/tests/test_web_routes_extended.py:1` | 128 | no | `test_rename_apply_returns_count_and_invalidates` `sources/memtomem-memory/packages/memtomem/tests/test_web_routes_extended.py:366`; `test_delete_apply_invalidates_cache` `sources/memtomem-memory/packages/memtomem/tests/test_web_routes_extended.py:402`; `test_delete_dry_run_skips_apply` `sources/memtomem-memory/packages/memtomem/tests/test_web_routes_extended.py:412`; `test_merge_apply_invalidates_cache` `sources/memtomem-memory/packages/memtomem/tests/test_web_routes_extended.py:420`; `test_merge_only_target_in_sources_returns_zero_no_invalidate` `sources/memtomem-memory/packages/memtomem/tests/test_web_routes_extended.py:444`; `test_dedup_merge_invalidates_cache` `sources/memtomem-memory/packages/memtomem/tests/test_web_routes_extended.py:665`; `test_dedup_merge_empty_delete_ids_does_not_invalidate` `sources/memtomem-memory/packages/memtomem/tests/test_web_routes_extended.py:677`; `test_dedup_merge_invalid_uuid_does_not_invalidate` `sources/memtomem-memory/packages/memtomem/tests/test_web_routes_extended.py:691`; `test_dedup_merge_keeper_in_delete_ids_is_400` `sources/memtomem-memory/packages/memtomem/tests/test_web_routes_extended.py:700`; `test_dedup_merge_failure_still_invalidates` `sources/memtomem-memory/packages/memtomem/tests/test_web_routes_extended.py:718`; `test_decay_expire_apply_invalidates_cache` `sources/memtomem-memory/packages/memtomem/tests/test_web_routes_extended.py:762`; `test_decay_expire_no_candidates_does_not_invalidate` `sources/memtomem-memory/packages/memtomem/tests/test_web_routes_extended.py:778`; `test_decay_expire_dry_run_does_not_invalidate` `sources/memtomem-memory/packages/memtomem/tests/test_web_routes_extended.py:793`; `test_decay_expire_failure_does_not_invalidate` `sources/memtomem-memory/packages/memtomem/tests/test_web_routes_extended.py:807`; `test_delete_namespace` `sources/memtomem-memory/packages/memtomem/tests/test_web_routes_extended.py:2364`; `test_delete_blocked_in_prod` `sources/memtomem-memory/packages/memtomem/tests/test_web_routes_extended.py:2434`; `test_delete_scratch` `sources/memtomem-memory/packages/memtomem/tests/test_web_routes_extended.py:2467`; `test_reset_returns_deleted_counts` `sources/memtomem-memory/packages/memtomem/tests/test_web_routes_extended.py:2622` |
| `sources/memtomem-memory/packages/memtomem/tests/test_web_sync_write_guard.py:1` | 15 | no | `test_cascade_delete_on_paused_rejected` `sources/memtomem-memory/packages/memtomem/tests/test_web_sync_write_guard.py:395`; `test_plain_delete_on_paused_not_gated` `sources/memtomem-memory/packages/memtomem/tests/test_web_sync_write_guard.py:413`; `test_mcp_cascade_delete_on_paused_not_gated` `sources/memtomem-memory/packages/memtomem/tests/test_web_sync_write_guard.py:427` |
| `sources/memtomem-memory/packages/memtomem/tests/test_web_writes_wait_for_writer.py:1` | 22 | no | `test_a_timed_out_sweep_deletes_nothing_and_still_reconfigures` `sources/memtomem-memory/packages/memtomem/tests/test_web_writes_wait_for_writer.py:442`; `test_a_sweep_that_fails_partway_deletes_nothing` `sources/memtomem-memory/packages/memtomem/tests/test_web_writes_wait_for_writer.py:562`; `test_index_only_delete_waits_and_finalizes_the_emptied_source` `sources/memtomem-memory/packages/memtomem/tests/test_web_writes_wait_for_writer.py:620`; `test_a_timed_out_index_only_delete_keeps_its_failure_detail` `sources/memtomem-memory/packages/memtomem/tests/test_web_writes_wait_for_writer.py:644` |
| `sources/memtomem-memory/packages/memtomem/tests/test_wiki_store.py:1` | 81 | no | `test_deleted_worktree_file` `sources/memtomem-memory/packages/memtomem/tests/test_wiki_store.py:347` |
| `sources/memtomem-memory/packages/memtomem/tests/test_write_provenance.py:1` | 50 | no | `test_a_bulk_namespace_delete_marks_the_session_incomplete` `sources/memtomem-memory/packages/memtomem/tests/test_write_provenance.py:1238`; `test_a_bulk_delete_that_removed_nothing_flags_nothing` `sources/memtomem-memory/packages/memtomem/tests/test_write_provenance.py:1263`; `test_a_namespace_tool_delete_marks_the_session_incomplete` `sources/memtomem-memory/packages/memtomem/tests/test_write_provenance.py:1285`; `test_a_source_file_delete_marks_the_session_incomplete` `sources/memtomem-memory/packages/memtomem/tests/test_write_provenance.py:1309` |
| `sources/memtomem-memory/packages/memtomem/tests/web/test_context_gateway_409_errors.py:1` | 6 | no | `test_cascade_delete_409_sync_paused` `sources/memtomem-memory/packages/memtomem/tests/web/test_context_gateway_409_errors.py:126`; `test_cascade_delete_409_sync_not_enrolled` `sources/memtomem-memory/packages/memtomem/tests/web/test_context_gateway_409_errors.py:152`; `test_cascade_delete_409_unknown_reason_falls_back_to_message` `sources/memtomem-memory/packages/memtomem/tests/web/test_context_gateway_409_errors.py:175`; `test_cascade_delete_409_sync_paused_ko` `sources/memtomem-memory/packages/memtomem/tests/web/test_context_gateway_409_errors.py:202`; `test_cascade_delete_paused_scope_offers_canonical_only_delete` `sources/memtomem-memory/packages/memtomem/tests/web/test_context_gateway_409_errors.py:302`; `test_cascade_delete_not_enrolled_scope_offers_canonical_only_delete` `sources/memtomem-memory/packages/memtomem/tests/web/test_context_gateway_409_errors.py:308` |
| `sources/memtomem-memory/packages/memtomem/tests/web/test_context_gateway_versions.py:1` | 11 | no | `test_remove_label_button_deletes_pointer` `sources/memtomem-memory/packages/memtomem/tests/web/test_context_gateway_versions.py:185` |
| `sources/memtomem-memory/packages/memtomem/tests/web/test_context_gateway_write_blocked.py:1` | 8 | no | `test_per_item_edit_and_delete_buttons_follow_tier_gate` `sources/memtomem-memory/packages/memtomem/tests/web/test_context_gateway_write_blocked.py:325` |
| `sources/memtomem-memory/packages/memtomem/tests/web/test_context_portal.py:1` | 4 | no | `test_portal_unregister_fires_delete_after_confirm` `sources/memtomem-memory/packages/memtomem/tests/web/test_context_portal.py:169` |
| `sources/memtomem-memory/packages/memtomem/tests/web/test_csrf_recovery.py:1` | 4 | no | `test_single_promote_preserves_partial_success_after_delete_refusal` `sources/memtomem-memory/packages/memtomem/tests/web/test_csrf_recovery.py:156` |
| `sources/memtomem-memory/packages/memtomem/tests/web/test_memory_dir_remove_confirm.py:1` | 5 | no | `test_confirm_options_follow_delete_chunk_count` `sources/memtomem-memory/packages/memtomem/tests/web/test_memory_dir_remove_confirm.py:40`; `test_md_remove_of_a_covered_dir_warns_and_deletes_nothing` `sources/memtomem-memory/packages/memtomem/tests/web/test_memory_dir_remove_confirm.py:112`; `test_md_remove_when_status_refresh_fails_offers_no_chunk_delete` `sources/memtomem-memory/packages/memtomem/tests/web/test_memory_dir_remove_confirm.py:141` |
| `sources/memtomem-memory/packages/memtomem/tests/web/test_settings_hooks_no_source.py:1` | 11 | no | `test_target_hook_detail_renders_delete_and_promote_actions` `sources/memtomem-memory/packages/memtomem/tests/web/test_settings_hooks_no_source.py:329`; `test_delete_target_hook_posts_exact_rule_identity` `sources/memtomem-memory/packages/memtomem/tests/web/test_settings_hooks_no_source.py:347`; `test_promote_with_remove_original_posts_delete` `sources/memtomem-memory/packages/memtomem/tests/web/test_settings_hooks_no_source.py:399` |
| `sources/memtomem-memory/packages/memtomem/tests/web/test_sources_partial_list.py:1` | 21 | no | `test_delete_keeps_the_count_left_out` `sources/memtomem-memory/packages/memtomem/tests/web/test_sources_partial_list.py:229` |

## 7. Corrections to B2

Old values below are copied from the triage row (TRIAGE.md), not reconstructed from memory. A refinement narrows an over-broad field; it need not reverse its original value. Counts are distinct field paths, not the number of factual errors.

- `forget_tool.scope`: `"Provider advertises no tools; separate MCP tools and mirrored builtin writes are distinct surfaces."` → Provider has none, but host MCP connection can independently expose mem_delete; chunk-ID edits source, source/namespace delete only index. Evidence: `sources/memtomem-memory/packages/memtomem/src/memtomem/server/tools/memory_crud.py:1722`, `sources/memtomem-memory/packages/memtomem/src/memtomem/server/tools/memory_crud.py:1740`.
- `engine_delete.scope`: `"Named deletion/retraction endpoint or method, not a guarantee of physical erasure. Its reach may overlap an agent tool; lack of a native tool is counted separately."` → `none` in this provider's tool schema; engine `mem_delete` exists on a separately exposed MCP server (`sources/memtomem-memory/packages/memtomem-hermes-memory/__init__.py:1471`; `sources/memtomem-memory/packages/memtomem/src/memtomem/server/tools/memory_crud.py:1641`). Chunk-ID path removes that line span from source then reindexes (`sources/memtomem-memory/packages/memtomem/src/memtomem/server/tools/memory_crud.py:1722`). source_file/namespace paths delete indexed chunks rather than the file itself; project_shared needs confirmation (`sources/memtomem-memory/packages/memtomem/src/memtomem/server/tools/memory_crud.py:1740`; `sources/memtomem-memory/packages/memtomem/src/memtomem/server/tools/memory_crud.py:1818`). `partial`: index-only delete can leave reindexable originals; summaries/vector/FTS/relations have separate storage cascades shown below. Provider optional host-write mirroring is not host history deletion. Evidence: `sources/memtomem-memory/packages/memtomem-hermes-memory/__init__.py:1471`, `sources/memtomem-memory/packages/memtomem/src/memtomem/server/tools/memory_crud.py:1641`, `sources/memtomem-memory/packages/memtomem/src/memtomem/server/tools/memory_crud.py:1722`, `sources/memtomem-memory/packages/memtomem/src/memtomem/server/tools/memory_crud.py:1740`, `sources/memtomem-memory/packages/memtomem/src/memtomem/server/tools/memory_crud.py:1818`.
- `context.mechanism.v`: `"Source expressions below identify retrieval, cached/static returns, lifecycle capture, and limits. A hook definition is not a guarantee it emits nonempty context each turn; branches and configuration apply."` → System block is empty (`sources/memtomem-memory/packages/memtomem-hermes-memory/__init__.py:1469`). Prefetch calls MCP search then shapes results under max_chars default 4000, allowed 500..9000 (`sources/memtomem-memory/packages/memtomem-hermes-memory/__init__.py:313`; `sources/memtomem-memory/packages/memtomem-hermes-memory/__init__.py:325`; `sources/memtomem-memory/packages/memtomem-hermes-memory/__init__.py:1248`). It can return empty for trivial/stale/timeout/missing-trust cases. Tools independently exposed by MCP must not be attributed to provider.get_tool_schemas. Evidence: `sources/memtomem-memory/packages/memtomem-hermes-memory/__init__.py:1469`, `sources/memtomem-memory/packages/memtomem-hermes-memory/__init__.py:313`, `sources/memtomem-memory/packages/memtomem-hermes-memory/__init__.py:325`, `sources/memtomem-memory/packages/memtomem-hermes-memory/__init__.py:1248`.
- `sec.telemetry.v`: `"unknown: no emitter identified in the selected adapter files; external dependencies not cleared"` → Direct store/MCP access `yes` using the same local process/files or configured endpoint and credentials. Provider stores no mandatory engine credential itself; MCP command/env/headers live in host MCP configuration (`sources/memtomem-memory/packages/memtomem-hermes-memory/__init__.py:250`). Accountless local mode `yes` for local storage/configured local embeddings, with backend-specific requirements; availability not executed. No provider analytics default proves engine telemetry off. Evidence: `sources/memtomem-memory/packages/memtomem-hermes-memory/__init__.py:250`.
- `repro.decls`: `["pyproject.toml:3", "pyproject.toml:5", "pyproject.toml:6", "pyproject.toml:16", "pyproject.toml:37", "pyproject.toml:50", "pyproject.toml:51", "pyproject.toml:52", "pyproject.toml:53", "pyproject.toml:54", "pyproject.toml:58", "pyproject.toml:59", "pyproject.toml:60", "pyproject.toml:61"]` → Provider does not pin the existing MCP server version; `unpinned` from this adapter. Background-search/mirror capability checks are runtime contract checks, not exact version locks (`sources/memtomem-memory/packages/memtomem-hermes-memory/__init__.py:611`; `sources/memtomem-memory/packages/memtomem-hermes-memory/__init__.py:1053`). Evidence: `sources/memtomem-memory/packages/memtomem-hermes-memory/__init__.py:611`, `sources/memtomem-memory/packages/memtomem-hermes-memory/__init__.py:1053`.
- `tests.count`: `15543` → 141 adapter-related declarations using the stated parser/scope; B2 counted a different, repository-wide inventory. This is a scope refinement, not a count of executed or passing tests. Hermes provider and mirror tests are distinct from engine mem_delete tests. There is no provider forget tool test; engine storage mutation tests exist; separate route/concurrency tests are listed below. Storage deletion test asserts the chunk disappears. This is engine-storage coverage, not a native provider forget-tool test. (`sources/memtomem-memory/packages/memtomem/tests/test_server_tools_core.py:406`). Evidence: `sources/memtomem-memory/packages/memtomem/tests/test_server_tools_core.py:406`.

MATRIX counts these 6 explicit field corrections/refinements. Sections 1-6 provide the body and test inventories. Test-name candidates are not passing tests or measured coverage.
