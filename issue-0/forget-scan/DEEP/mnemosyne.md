# mnemosyne: B3 static correctness review

Snapshot `7106311a95683cbb908b1f4f0f9f4df034aad50b` (MANIFEST.json row `mnemosyne`). No plugin execution or passing-test claims.

## 1. Architecture

`provider`: catalogue wrapper installs mnemosyne-hermes; active implementation is integrations/hermes/src/mnemosyne_hermes, not merely the legacy hermes_memory_provider module. It imports local BeamMemory/SQLite, vector and canonical stores; optional sync/model paths are separate (`sources/mnemosyne/integrations/hermes-catalog/pyproject.toml:10`; `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:4934`).

Hook/body inventory (no-op/fallback declarations are not active behavior):

| callable | declaration | first executable body line |
|---|---|---|
| `register` | `sources/mnemosyne/hermes_memory_provider/__init__.py:4863` | `sources/mnemosyne/hermes_memory_provider/__init__.py:4865` |
| `is_available` | `sources/mnemosyne/hermes_memory_provider/__init__.py:1870` | `sources/mnemosyne/hermes_memory_provider/__init__.py:1872` |
| `get_config_schema` | `sources/mnemosyne/hermes_memory_provider/__init__.py:2161` | `sources/mnemosyne/hermes_memory_provider/__init__.py:2162` |
| `save_config` | `sources/mnemosyne/hermes_memory_provider/__init__.py:2178` | `sources/mnemosyne/hermes_memory_provider/__init__.py:2180` |
| `initialize` | `sources/mnemosyne/hermes_memory_provider/__init__.py:2260` | `sources/mnemosyne/hermes_memory_provider/__init__.py:2262` |
| `system_prompt_block` | `sources/mnemosyne/hermes_memory_provider/__init__.py:2409` | `sources/mnemosyne/hermes_memory_provider/__init__.py:2410` |
| `prefetch` | `sources/mnemosyne/hermes_memory_provider/__init__.py:2455` | `sources/mnemosyne/hermes_memory_provider/__init__.py:2459` |
| `queue_prefetch` | `sources/mnemosyne/hermes_memory_provider/__init__.py:2559` | `sources/mnemosyne/hermes_memory_provider/__init__.py:2560` |
| `sync_turn` | `sources/mnemosyne/hermes_memory_provider/__init__.py:2665` | `sources/mnemosyne/hermes_memory_provider/__init__.py:2667` |
| `on_pre_compress` | `sources/mnemosyne/hermes_memory_provider/__init__.py:2765` | `sources/mnemosyne/hermes_memory_provider/__init__.py:2771` |
| `on_session_switch` | `sources/mnemosyne/hermes_memory_provider/__init__.py:2777` | `sources/mnemosyne/hermes_memory_provider/__init__.py:2792` |
| `get_tool_schemas` | `sources/mnemosyne/hermes_memory_provider/__init__.py:2923` | `sources/mnemosyne/hermes_memory_provider/__init__.py:2925` |
| `handle_tool_call` | `sources/mnemosyne/hermes_memory_provider/__init__.py:2927` | `sources/mnemosyne/hermes_memory_provider/__init__.py:2930` |
| `on_turn_start` | `sources/mnemosyne/hermes_memory_provider/__init__.py:4714` | `sources/mnemosyne/hermes_memory_provider/__init__.py:4715` |
| `on_session_end` | `sources/mnemosyne/hermes_memory_provider/__init__.py:4717` | `sources/mnemosyne/hermes_memory_provider/__init__.py:4723` |
| `on_memory_write` | `sources/mnemosyne/hermes_memory_provider/__init__.py:4770` | `sources/mnemosyne/hermes_memory_provider/__init__.py:4771` |
| `shutdown` | `sources/mnemosyne/hermes_memory_provider/__init__.py:4803` | `sources/mnemosyne/hermes_memory_provider/__init__.py:4809` |
| `handle_tool_call` | `sources/mnemosyne/hermes_memory_provider/persona_adapter.py:47` | `sources/mnemosyne/hermes_memory_provider/persona_adapter.py:48` |
| `shutdown` | `sources/mnemosyne/hermes_memory_provider/sync_adapter.py:250` | `sources/mnemosyne/hermes_memory_provider/sync_adapter.py:252` |
| `handle_tool_call` | `sources/mnemosyne/hermes_memory_provider/sync_adapter.py:267` | `sources/mnemosyne/hermes_memory_provider/sync_adapter.py:268` |
| `register` | `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:5556` | `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:5559` |
| `is_available` | `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:1858` | `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:1860` |
| `get_config_schema` | `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:2156` | `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:2157` |
| `save_config` | `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:2173` | `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:2175` |
| `initialize` | `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:2486` | `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:2488` |
| `system_prompt_block` | `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:2777` | `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:2778` |
| `prefetch` | `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:2836` | `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:2842` |
| `queue_prefetch` | `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:2960` | `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:2961` |
| `sync_turn` | `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:3131` | `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:3133` |
| `get_tool_schemas` | `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:3387` | `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:3389` |
| `handle_tool_call` | `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:3391` | `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:3392` |
| `on_turn_start` | `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:5260` | `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:5261` |
| `on_pre_compress` | `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:5264` | `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:5270` |
| `on_session_switch` | `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:5276` | `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:5288` |
| `on_session_end` | `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:5330` | `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:5336` |
| `on_memory_write` | `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:5395` | `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:5396` |
| `shutdown` | `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:5427` | `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:5433` |
| `handle_tool_call` | `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/persona_adapter.py:47` | `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/persona_adapter.py:48` |
| `shutdown` | `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/sync_adapter.py:250` | `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/sync_adapter.py:252` |
| `handle_tool_call` | `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/sync_adapter.py:267` | `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/sync_adapter.py:268` |

## 2. Forget paths and derived copies

`yes`: mnemosyne_forget calls `_forget_with_episodic_fallback` and reports deleted/not_found (`sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:4934`). mnemosyne_forget_canonical retires the owner/category/name slot (`sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:4578`). mnemosyne_invalidate marks invalid; mnemosyne_validate(action=delete) and batch/replay paths must not be conflated with plain update (`sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:4237`; `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:4745`). Engine `forget_working` removes session-or-global authorized row and vector, conditionally cascades children only without a competing episodic parent, then invalidates query cache (`sources/mnemosyne/mnemosyne/core/beam.py:8001`; `sources/mnemosyne/mnemosyne/core/beam.py:8008`; `sources/mnemosyne/mnemosyne/core/beam.py:8014`). forget_episodic has analogous ownership guards (`sources/mnemosyne/mnemosyne/core/beam.py:8048`). Derived reach `partial`: ambiguity deliberately preserves shared children, and tier/canonical/sync copies need separate handling.

Host state.db is independent (`hermes-agent/hermes_state.py:181`). No reviewed adapter forget body performs a general purge of that database or built-in MEMORY.md/USER.md. Prompt/tool-result copies and independent source transcripts remain separate; source operation bodies below establish the scope of each plugin mutation.

<details>
<summary>Deletion, invalidation, archive and reset body ledger</summary>

Public routes, SDK/CLI dispatch and internal storage operations are distinguished by their source. Cache invalidation and management-object deletion are not additional fact-forget tools. A stub is not working deletion. Call-through means the downstream body still determines erasure. HTTP prefixes/selectors must be read with route registration.

**_forget_with_episodic_fallback**, body `sources/mnemosyne/hermes_memory_provider/__init__.py:333`; implementation/call-through.

- `sources/mnemosyne/hermes_memory_provider/__init__.py:331` `def _forget_with_episodic_fallback(beam: Any, memory_id: str) -> bool:`
- `sources/mnemosyne/hermes_memory_provider/__init__.py:333` `ok = beam.forget_working(memory_id)`

**_invalidate_surface_locked**, body `sources/mnemosyne/hermes_memory_provider/__init__.py:1825`; implementation/call-through.

- `sources/mnemosyne/hermes_memory_provider/__init__.py:1823` `def _invalidate_surface_locked(self) -> None:`
- `sources/mnemosyne/hermes_memory_provider/__init__.py:1825` `self._clear_sync_adapter()`

**_handle_shared_forget**, body `sources/mnemosyne/hermes_memory_provider/__init__.py:3476`; implementation/call-through.

- `sources/mnemosyne/hermes_memory_provider/__init__.py:3475` `def _handle_shared_forget(self, args: Dict[str, Any]) -> str:`
- `sources/mnemosyne/hermes_memory_provider/__init__.py:3482` `ok = self._surface_beam.forget_working(memory_id)`
- `sources/mnemosyne/hermes_memory_provider/__init__.py:3488` `return json.dumps({"status": "deleted" if ok else "not_found", "memory_id": memory_id, "shared_db": str(self._shared_surface_path or "")})`

**_invalidate_target_active**, body `sources/mnemosyne/hermes_memory_provider/__init__.py:3577`; implementation/call-through.

- `sources/mnemosyne/hermes_memory_provider/__init__.py:3564` `def _invalidate_target_active(self, beam: Any, memory_id: str) -> bool:`
- `sources/mnemosyne/hermes_memory_provider/__init__.py:3577` `try:`

**_handle_invalidate**, body `sources/mnemosyne/hermes_memory_provider/__init__.py:3606`; implementation/call-through.

- `sources/mnemosyne/hermes_memory_provider/__init__.py:3605` `def _handle_invalidate(self, args: Dict[str, Any]) -> str:`
- `sources/mnemosyne/hermes_memory_provider/__init__.py:3623` `# invalidation could never land on the surface (#1050 work-order (a)).`
- `sources/mnemosyne/hermes_memory_provider/__init__.py:3635` `ok = target_beam.invalidate(`
- `sources/mnemosyne/hermes_memory_provider/__init__.py:3639` `"invalidate", memory_id=memory_id, bank=bank,`
- `sources/mnemosyne/hermes_memory_provider/__init__.py:3640` `source_tool="mnemosyne_invalidate",`
- `sources/mnemosyne/hermes_memory_provider/__init__.py:3641` `metadata={"replacement_id": replacement_id, "invalidated": ok} if replacement_id else {"invalidated": ok},`
- `sources/mnemosyne/hermes_memory_provider/__init__.py:3644` `if replacement_id and self._invalidate_target_active(target_beam, memory_id):`
- `sources/mnemosyne/hermes_memory_provider/__init__.py:3646` `# remaining reason the invalidation can fail is that the`
- `sources/mnemosyne/hermes_memory_provider/__init__.py:3658` `return json.dumps({"status": "invalidated", "memory_id": memory_id, "bank": bank})`

**_handle_forget_canonical**, body `sources/mnemosyne/hermes_memory_provider/__init__.py:4014`; implementation/call-through.

- `sources/mnemosyne/hermes_memory_provider/__init__.py:4013` `def _handle_forget_canonical(self, args: Dict[str, Any]) -> str:`
- `sources/mnemosyne/hermes_memory_provider/__init__.py:4024` `retired = store.forget(owner_id, category, name)`

**_handle_forget**, body `sources/mnemosyne/hermes_memory_provider/__init__.py:4522`; implementation/call-through.

- `sources/mnemosyne/hermes_memory_provider/__init__.py:4521` `def _handle_forget(self, args: Dict[str, Any]) -> str:`
- `sources/mnemosyne/hermes_memory_provider/__init__.py:4532` `"status": "deleted" if ok else "not_found",`

**_forget_with_episodic_fallback**, body `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:319`; implementation/call-through.

- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:317` `def _forget_with_episodic_fallback(beam: Any, memory_id: str) -> bool:`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:319` `ok = beam.forget_working(memory_id)`

**_invalidate_surface_locked**, body `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:1730`; implementation/call-through.

- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:1728` `def _invalidate_surface_locked(self) -> None:`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:1730` `self._clear_provider_adapters()`

**_handle_shared_forget**, body `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:3982`; implementation/call-through.

- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:3981` `def _handle_shared_forget(self, args: Dict[str, Any]) -> str:`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:3988` `ok = self._surface_beam.forget_working(memory_id)`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:3994` `return json.dumps({"status": "deleted" if ok else "not_found", "memory_id": memory_id, "shared_db": str(self._shared_surface_path or "")})`

**_invalidate_target_active**, body `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:4083`; implementation/call-through.

- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:4070` `def _invalidate_target_active(self, beam: Any, memory_id: str) -> bool:`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:4083` `try:`

**_handle_invalidate**, body `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:4112`; implementation/call-through.

- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:4111` `def _handle_invalidate(self, args: Dict[str, Any]) -> str:`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:4129` `# invalidation could never land on the surface (#1050 work-order (a)).`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:4141` `ok = target_beam.invalidate(`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:4145` `"invalidate", memory_id=memory_id, bank=bank,`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:4146` `source_tool="mnemosyne_invalidate",`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:4147` `metadata={"replacement_id": replacement_id, "invalidated": ok} if replacement_id else {"invalidated": ok},`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:4150` `if replacement_id and self._invalidate_target_active(target_beam, memory_id):`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:4152` `# remaining reason the invalidation can fail is that the`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:4164` `return json.dumps({"status": "invalidated", "memory_id": memory_id, "bank": bank})`

**_handle_forget_canonical**, body `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:4568`; implementation/call-through.

- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:4567` `def _handle_forget_canonical(self, args: Dict[str, Any]) -> str:`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:4578` `retired = store.forget(owner_id, category, name)`

**_handle_forget**, body `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:4931`; implementation/call-through.

- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:4930` `def _handle_forget(self, args: Dict[str, Any]) -> str:`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:4941` `"status": "deleted" if ok else "not_found",`

**cmd_delete**, body `sources/mnemosyne/mnemosyne/cli.py:320`; implementation/call-through.

- `sources/mnemosyne/mnemosyne/cli.py:318` `def cmd_delete(args):`
- `sources/mnemosyne/mnemosyne/cli.py:325` `success = mem.forget(memory_id)`

**delete_bank**, body `sources/mnemosyne/mnemosyne/core/banks.py:253`; implementation/call-through.

- `sources/mnemosyne/mnemosyne/core/banks.py:251` `def delete_bank(name: str, data_dir: Path = None, force: bool = False) -> bool:`
- `sources/mnemosyne/mnemosyne/core/banks.py:253` `return BankManager(data_dir).delete_bank(name, force=force)`

**delete_bank**, body `sources/mnemosyne/mnemosyne/core/banks.py:99`; implementation/call-through.

- `sources/mnemosyne/mnemosyne/core/banks.py:85` `def delete_bank(self, name: str, force: bool = False) -> bool:`
- `sources/mnemosyne/mnemosyne/core/banks.py:99` `self._validate_name(name)`

**_wm_vec_delete**, body `sources/mnemosyne/mnemosyne/core/beam.py:4296`; implementation/call-through.

- `sources/mnemosyne/mnemosyne/core/beam.py:4294` `def _wm_vec_delete(conn: sqlite3.Connection, memory_id: str) -> None:`
- `sources/mnemosyne/mnemosyne/core/beam.py:4301` `conn.execute("DELETE FROM vec_working WHERE rowid = ?", (rowid,))`

**_invalidate_working_embedding**, body `sources/mnemosyne/mnemosyne/core/beam.py:4314`; implementation/call-through.

- `sources/mnemosyne/mnemosyne/core/beam.py:4304` `def _invalidate_working_embedding(conn: sqlite3.Connection, memory_id: str) -> None:`
- `sources/mnemosyne/mnemosyne/core/beam.py:4314` `conn.execute("DELETE FROM memory_embeddings WHERE memory_id = ?", (memory_id,))`

**_invalidate_query_cache_for_conn**, body `sources/mnemosyne/mnemosyne/core/beam.py:4519`; implementation/call-through.

- `sources/mnemosyne/mnemosyne/core/beam.py:4515` `def _invalidate_query_cache_for_conn(conn, operation: str) -> None:`
- `sources/mnemosyne/mnemosyne/core/beam.py:4531` `cache.invalidate()`
- `sources/mnemosyne/mnemosyne/core/beam.py:4536` `"%s: query-cache invalidation failed (%s): %s",`

**_invalidate_query_cache**, body `sources/mnemosyne/mnemosyne/core/beam.py:7268`; implementation/call-through.

- `sources/mnemosyne/mnemosyne/core/beam.py:7266` `def _invalidate_query_cache(self) -> None:`
- `sources/mnemosyne/mnemosyne/core/beam.py:7270` `cache.invalidate()`
- `sources/mnemosyne/mnemosyne/core/beam.py:7272` `_invalidate_query_cache_for_conn(self.conn, "beam")`

**_invalidate_query_cache_after_remember_commit**, body `sources/mnemosyne/mnemosyne/core/beam.py:7276`; implementation/call-through.

- `sources/mnemosyne/mnemosyne/core/beam.py:7274` `def _invalidate_query_cache_after_remember_commit(self) -> None:`
- `sources/mnemosyne/mnemosyne/core/beam.py:7277` `self._invalidate_query_cache()`
- `sources/mnemosyne/mnemosyne/core/beam.py:7280` `"remember: query-cache invalidation failed after commit (%s): %s",`

**_invalidate_query_cache_after_commit**, body `sources/mnemosyne/mnemosyne/core/beam.py:7293`; implementation/call-through.

- `sources/mnemosyne/mnemosyne/core/beam.py:7285` `def _invalidate_query_cache_after_commit(self, operation: str) -> None:`
- `sources/mnemosyne/mnemosyne/core/beam.py:7294` `self._invalidate_query_cache()`
- `sources/mnemosyne/mnemosyne/core/beam.py:7297` `"%s: query-cache invalidation failed after commit (%s): %s",`

**invalidate**, body `sources/mnemosyne/mnemosyne/core/beam.py:7319`; implementation/call-through.

- `sources/mnemosyne/mnemosyne/core/beam.py:7301` `def invalidate(`
- `sources/mnemosyne/mnemosyne/core/beam.py:7335` `def validate_and_invalidate() -> bool:`
- `sources/mnemosyne/mnemosyne/core/beam.py:7392` `invalidated = validate_and_invalidate()`
- `sources/mnemosyne/mnemosyne/core/beam.py:7394` `invalidated = validate_and_invalidate()`
- `sources/mnemosyne/mnemosyne/core/beam.py:7395` `if invalidated and not defer_cache_invalidation:`
- `sources/mnemosyne/mnemosyne/core/beam.py:7397` `self._invalidate_query_cache_after_commit("invalidate")`
- `sources/mnemosyne/mnemosyne/core/beam.py:7399` `self._invalidate_query_cache()`
- `sources/mnemosyne/mnemosyne/core/beam.py:7400` `return invalidated`
- `sources/mnemosyne/mnemosyne/core/beam.py:7410` `invalidated = False`
- `sources/mnemosyne/mnemosyne/core/beam.py:7420` `invalidated = True`
- `sources/mnemosyne/mnemosyne/core/beam.py:7424` `if invalidated and not defer_cache_invalidation:`
- `sources/mnemosyne/mnemosyne/core/beam.py:7426` `self._invalidate_query_cache_after_commit("invalidate")`
- `sources/mnemosyne/mnemosyne/core/beam.py:7428` `self._invalidate_query_cache()`
- `sources/mnemosyne/mnemosyne/core/beam.py:7429` `return invalidated`

**_delete_orphaned_legacy_mirror**, body `sources/mnemosyne/mnemosyne/core/beam.py:7919`; implementation/call-through.

- `sources/mnemosyne/mnemosyne/core/beam.py:7910` `@staticmethod`
- `sources/mnemosyne/mnemosyne/core/beam.py:7911` `def _delete_orphaned_legacy_mirror(cursor, memory_id: str) -> None:`
- `sources/mnemosyne/mnemosyne/core/beam.py:7924` `cursor.execute("DELETE FROM memories WHERE id = ?", (memory_id,))`

**_delete_derived_memory_rows**, body `sources/mnemosyne/mnemosyne/core/beam.py:7935`; implementation/call-through.

- `sources/mnemosyne/mnemosyne/core/beam.py:7926` `@staticmethod`
- `sources/mnemosyne/mnemosyne/core/beam.py:7927` `def _delete_derived_memory_rows(cursor, memory_id: str) -> None:`
- `sources/mnemosyne/mnemosyne/core/beam.py:7951` `f"DELETE FROM {table} WHERE source_memory_id = ?", (memory_id,)`
- `sources/mnemosyne/mnemosyne/core/beam.py:7954` `cursor.execute("DELETE FROM facts WHERE source_msg_id = ?", (memory_id,))`

**_delete_unambiguous_memory_children**, body `sources/mnemosyne/mnemosyne/core/beam.py:7966`; implementation/call-through.

- `sources/mnemosyne/mnemosyne/core/beam.py:7956` `@staticmethod`
- `sources/mnemosyne/mnemosyne/core/beam.py:7957` `def _delete_unambiguous_memory_children(cursor, memory_id: str) -> None:`
- `sources/mnemosyne/mnemosyne/core/beam.py:7966` `cursor.execute("DELETE FROM annotations WHERE memory_id = ?", (memory_id,))`
- `sources/mnemosyne/mnemosyne/core/beam.py:7967` `cursor.execute("DELETE FROM memory_embeddings WHERE memory_id = ?", (memory_id,))`
- `sources/mnemosyne/mnemosyne/core/beam.py:7972` `cursor.execute("DELETE FROM gists WHERE memory_id = ?", (memory_id,))`

**forget_working**, body `sources/mnemosyne/mnemosyne/core/beam.py:7985`; implementation/call-through.

- `sources/mnemosyne/mnemosyne/core/beam.py:7975` `def forget_working(self, memory_id: str) -> bool:`
- `sources/mnemosyne/mnemosyne/core/beam.py:8001` `cursor.execute("DELETE FROM vec_working WHERE rowid = ?", (int(authorized_row["rowid"]),))`
- `sources/mnemosyne/mnemosyne/core/beam.py:8003` `"DELETE FROM working_memory WHERE id = ? AND (session_id = ? OR scope = 'global')",`
- `sources/mnemosyne/mnemosyne/core/beam.py:8014` `self._invalidate_query_cache_after_commit("forget_working")`
- `sources/mnemosyne/mnemosyne/core/beam.py:8016` `self._invalidate_query_cache()`

**forget_episodic**, body `sources/mnemosyne/mnemosyne/core/beam.py:8028`; implementation/call-through.

- `sources/mnemosyne/mnemosyne/core/beam.py:8019` `def forget_episodic(self, memory_id: str) -> bool:`
- `sources/mnemosyne/mnemosyne/core/beam.py:8045` `"DELETE FROM vec_episodes WHERE rowid = ?",`
- `sources/mnemosyne/mnemosyne/core/beam.py:8049` `"DELETE FROM episodic_memory "`
- `sources/mnemosyne/mnemosyne/core/beam.py:8061` `self._invalidate_query_cache_after_commit("forget_episodic")`
- `sources/mnemosyne/mnemosyne/core/beam.py:8064` `# Invalidate now; rollback can safely leave a cache miss.`
- `sources/mnemosyne/mnemosyne/core/beam.py:8065` `self._invalidate_query_cache()`

**validate_and_invalidate**, body `sources/mnemosyne/mnemosyne/core/beam.py:7336`; implementation/call-through.

- `sources/mnemosyne/mnemosyne/core/beam.py:7335` `def validate_and_invalidate() -> bool:`
- `sources/mnemosyne/mnemosyne/core/beam.py:7336` `now = datetime.now(timezone.utc).isoformat()`

**delete_vector**, body `sources/mnemosyne/mnemosyne/core/binary_vectors.py:231`; implementation/call-through.

- `sources/mnemosyne/mnemosyne/core/binary_vectors.py:229` `def delete_vector(self, memory_id: str):`
- `sources/mnemosyne/mnemosyne/core/binary_vectors.py:233` `DELETE FROM {self.table_name} WHERE memory_id = ?`

**forget_canonical**, body `sources/mnemosyne/mnemosyne/core/canonical.py:718`; implementation/call-through.

- `sources/mnemosyne/mnemosyne/core/canonical.py:706` `def forget_canonical(`
- `sources/mnemosyne/mnemosyne/core/canonical.py:719` `return store.forget(owner_id, category, name)`

**forget**, body `sources/mnemosyne/mnemosyne/core/canonical.py:350`; implementation/call-through.

- `sources/mnemosyne/mnemosyne/core/canonical.py:342` `def forget(self, owner_id: str, category: str, name: str) -> bool:`
- `sources/mnemosyne/mnemosyne/core/canonical.py:350` `cursor = self.conn.cursor()`

**_hygiene_delete_cascade**, body `sources/mnemosyne/mnemosyne/core/hygiene.py:134`; implementation/call-through.

- `sources/mnemosyne/mnemosyne/core/hygiene.py:116` `def _hygiene_delete_cascade(`
- `sources/mnemosyne/mnemosyne/core/hygiene.py:140` `cursor.execute(f"DELETE FROM {table_name} WHERE id = ?", (memory_id,))`
- `sources/mnemosyne/mnemosyne/core/hygiene.py:141` `cursor.execute("DELETE FROM annotations WHERE memory_id = ?", (memory_id,))`
- `sources/mnemosyne/mnemosyne/core/hygiene.py:142` `cursor.execute("DELETE FROM memory_embeddings WHERE memory_id = ?", (memory_id,))`
- `sources/mnemosyne/mnemosyne/core/hygiene.py:147` `cursor.execute("DELETE FROM gists WHERE memory_id = ?", (memory_id,))`
- `sources/mnemosyne/mnemosyne/core/hygiene.py:167` `cursor.execute(f"DELETE FROM {vec_table} WHERE rowid = ?", (int(rowid),))`

**restore_archived**, body `sources/mnemosyne/mnemosyne/core/hygiene.py:963`; implementation/call-through.

- `sources/mnemosyne/mnemosyne/core/hygiene.py:943` `def restore_archived(`
- `sources/mnemosyne/mnemosyne/core/hygiene.py:973` `f"SELECT * FROM hygiene_audit_log WHERE id IN ({placeholders}) AND action = 'archived'",`
- `sources/mnemosyne/mnemosyne/core/hygiene.py:978` `"SELECT * FROM hygiene_audit_log WHERE action = 'archived' ORDER BY id DESC LIMIT ?",`
- `sources/mnemosyne/mnemosyne/core/hygiene.py:989` `# stored during archive), not the audit log's original_metadata`
- `sources/mnemosyne/mnemosyne/core/hygiene.py:990` `# snapshot (which is pre-archive and lacks the saved importance).`
- `sources/mnemosyne/mnemosyne/core/hygiene.py:1001` `# Idempotency guard: only restore rows that are still archived.`
- `sources/mnemosyne/mnemosyne/core/hygiene.py:1002` `# The audit log is append-only, so a second restore_archived()`
- `sources/mnemosyne/mnemosyne/core/hygiene.py:1006` `# Skipping rows whose _archived flag is gone makes repeat calls a`
- `sources/mnemosyne/mnemosyne/core/hygiene.py:1008` `if not meta.get("_archived"):`
- `sources/mnemosyne/mnemosyne/core/hygiene.py:1012` `# available; fall back to 0.5 for entries archived before`
- `sources/mnemosyne/mnemosyne/core/hygiene.py:1015` `meta.pop("_archived", None)`
- `sources/mnemosyne/mnemosyne/core/hygiene.py:1016` `meta.pop("_archived_at", None)`
- `sources/mnemosyne/mnemosyne/core/hygiene.py:1017` `meta.pop("_archive_reason", None)`

**forget**, body `sources/mnemosyne/mnemosyne/core/memory.py:1536`; implementation/call-through.

- `sources/mnemosyne/mnemosyne/core/memory.py:1534` `def forget(memory_id: str, bank: str = None) -> bool:`
- `sources/mnemosyne/mnemosyne/core/memory.py:1536` `return _get_default(bank).forget(memory_id)`

**forget**, body `sources/mnemosyne/mnemosyne/core/memory.py:898`; implementation/call-through.

- `sources/mnemosyne/mnemosyne/core/memory.py:896` `def forget(self, memory_id: str) -> bool:`
- `sources/mnemosyne/mnemosyne/core/memory.py:898` `emit_invalidation = True`
- `sources/mnemosyne/mnemosyne/core/memory.py:920` `result = self.beam.forget_episodic(memory_id)`
- `sources/mnemosyne/mnemosyne/core/memory.py:921` `emit_invalidation = result`
- `sources/mnemosyne/mnemosyne/core/memory.py:924` `"DELETE FROM memories WHERE id = ? AND session_id = ?",`
- `sources/mnemosyne/mnemosyne/core/memory.py:928` `result = self.beam.forget_episodic(memory_id)`
- `sources/mnemosyne/mnemosyne/core/memory.py:929` `emit_invalidation = result`
- `sources/mnemosyne/mnemosyne/core/memory.py:932` `"DELETE FROM memories WHERE id = ? AND session_id = ?",`
- `sources/mnemosyne/mnemosyne/core/memory.py:936` `result = self.beam.forget_working(memory_id)`
- `sources/mnemosyne/mnemosyne/core/memory.py:939` `# not publish a successful invalidation.`
- `sources/mnemosyne/mnemosyne/core/memory.py:940` `if emit_invalidation:`
- `sources/mnemosyne/mnemosyne/core/memory.py:941` `self._emit_after_commit("MEMORY_INVALIDATED", memory_id)`

**invalidate**, body `sources/mnemosyne/mnemosyne/core/memory.py:1024`; implementation/call-through.

- `sources/mnemosyne/mnemosyne/core/memory.py:1022` `def invalidate(self, memory_id: str, replacement_id: str = None) -> bool:`
- `sources/mnemosyne/mnemosyne/core/memory.py:1024` `result = self.beam.invalidate(memory_id, replacement_id=replacement_id)`
- `sources/mnemosyne/mnemosyne/core/memory.py:1026` `self._emit_wrapper("MEMORY_INVALIDATED", memory_id, replacement_id=replacement_id)`

**on_invalidate**, body `sources/mnemosyne/mnemosyne/core/plugins.py:77`; stub.

- `sources/mnemosyne/mnemosyne/core/plugins.py:74` `@abc.abstractmethod`
- `sources/mnemosyne/mnemosyne/core/plugins.py:75` `def on_invalidate(self, memory_id: str) -> None:`
- `sources/mnemosyne/mnemosyne/core/plugins.py:77` `...`

**on_invalidate**, body `sources/mnemosyne/mnemosyne/core/plugins.py:138`; implementation/call-through.

- `sources/mnemosyne/mnemosyne/core/plugins.py:137` `def on_invalidate(self, memory_id: str) -> None:`
- `sources/mnemosyne/mnemosyne/core/plugins.py:139` `"event": "invalidate",`
- `sources/mnemosyne/mnemosyne/core/plugins.py:145` `"[LoggingPlugin] invalidate: %s", entry)`

**on_invalidate**, body `sources/mnemosyne/mnemosyne/core/plugins.py:200`; implementation/call-through.

- `sources/mnemosyne/mnemosyne/core/plugins.py:199` `def on_invalidate(self, memory_id: str) -> None:`
- `sources/mnemosyne/mnemosyne/core/plugins.py:200` `self._counters["invalidate"] += 1`

**reset**, body `sources/mnemosyne/mnemosyne/core/plugins.py:227`; implementation/call-through.

- `sources/mnemosyne/mnemosyne/core/plugins.py:225` `def reset(self) -> None:`
- `sources/mnemosyne/mnemosyne/core/plugins.py:227` `for key in self._counters:`

**on_invalidate**, body `sources/mnemosyne/mnemosyne/core/plugins.py:289`; stub.

- `sources/mnemosyne/mnemosyne/core/plugins.py:288` `def on_invalidate(self, memory_id: str) -> None:`
- `sources/mnemosyne/mnemosyne/core/plugins.py:289` `pass`

**on_invalidate**, body `sources/mnemosyne/mnemosyne/core/plugins.py:424`; stub.

- `sources/mnemosyne/mnemosyne/core/plugins.py:423` `def on_invalidate(self, memory_id: str) -> None:`
- `sources/mnemosyne/mnemosyne/core/plugins.py:424` `pass`

**notify_invalidate**, body `sources/mnemosyne/mnemosyne/core/plugins.py:682`; implementation/call-through.

- `sources/mnemosyne/mnemosyne/core/plugins.py:680` `def notify_invalidate(self, memory_id: str) -> None:`
- `sources/mnemosyne/mnemosyne/core/plugins.py:685` `instance.on_invalidate(memory_id)`
- `sources/mnemosyne/mnemosyne/core/plugins.py:687` `logger.error("Plugin %s on_invalidate error: %s", instance.name, exc)`

**invalidate**, body `sources/mnemosyne/mnemosyne/core/query_cache.py:167`; implementation/call-through.

- `sources/mnemosyne/mnemosyne/core/query_cache.py:165` `def invalidate(self):`
- `sources/mnemosyne/mnemosyne/core/query_cache.py:176` `self._conn.execute("DELETE FROM query_cache")`

**_delete_persisted**, body `sources/mnemosyne/mnemosyne/core/query_cache.py:419`; implementation/call-through.

- `sources/mnemosyne/mnemosyne/core/query_cache.py:417` `def _delete_persisted(self, keys: List[str]):`
- `sources/mnemosyne/mnemosyne/core/query_cache.py:428` `"DELETE FROM query_cache WHERE normalized = ? "`

**reset**, body `sources/mnemosyne/mnemosyne/core/recall_diagnostics.py:327`; implementation/call-through.

- `sources/mnemosyne/mnemosyne/core/recall_diagnostics.py:324` `def reset(self) -> None:`
- `sources/mnemosyne/mnemosyne/core/recall_diagnostics.py:327` `with self._lock:`

**reset**, body `sources/mnemosyne/mnemosyne/extraction/diagnostics.py:266`; implementation/call-through.

- `sources/mnemosyne/mnemosyne/extraction/diagnostics.py:263` `def reset(self) -> None:`
- `sources/mnemosyne/mnemosyne/extraction/diagnostics.py:266` `with self._lock:`

**delete**, body `sources/mnemosyne/mnemosyne/integrations/openclaw.py:263`; implementation/call-through.

- `sources/mnemosyne/mnemosyne/integrations/openclaw.py:252` `def delete(self, key: str, **kwargs: Any) -> bool:`
- `sources/mnemosyne/mnemosyne/integrations/openclaw.py:269` `self._memory.forget(memory_id)`

**_delete_key_index**, body `sources/mnemosyne/mnemosyne/integrations/openclaw.py:321`; implementation/call-through.

- `sources/mnemosyne/mnemosyne/integrations/openclaw.py:319` `def _delete_key_index(self, key: str) -> None:`
- `sources/mnemosyne/mnemosyne/integrations/openclaw.py:321` `if hasattr(self, "_key_index") and key in self._key_index:`

**mnemosyne_forget**, body `sources/mnemosyne/mnemosyne/integrations/openwebui_tool.py:170`; implementation/call-through.

- `sources/mnemosyne/mnemosyne/integrations/openwebui_tool.py:165` `async def mnemosyne_forget(self, memory_id: str) -> str:`
- `sources/mnemosyne/mnemosyne/integrations/openwebui_tool.py:172` `mem.forget(memory_id)`

**_handle_shared_forget**, body `sources/mnemosyne/mnemosyne/mcp_tools.py:515`; implementation/call-through.

- `sources/mnemosyne/mnemosyne/mcp_tools.py:513` `def _handle_shared_forget(arguments: Dict[str, Any]) -> Dict[str, Any]:`
- `sources/mnemosyne/mnemosyne/mcp_tools.py:519` `ok = surface_beam.forget_working(memory_id)`
- `sources/mnemosyne/mnemosyne/mcp_tools.py:520` `return {"status": "deleted" if ok else "not_found", "memory_id": memory_id, "shared_db": str(_shared_db_path())}`

**_handle_invalidate**, body `sources/mnemosyne/mnemosyne/mcp_tools.py:569`; implementation/call-through.

- `sources/mnemosyne/mnemosyne/mcp_tools.py:567` `def _handle_invalidate(arguments: Dict[str, Any]) -> Dict[str, Any]:`
- `sources/mnemosyne/mnemosyne/mcp_tools.py:575` `if not mem.invalidate(memory_id, replacement_id=replacement_id):`
- `sources/mnemosyne/mnemosyne/mcp_tools.py:577` `return {"status": "invalidated", "memory_id": memory_id}`

**_handle_forget_canonical**, body `sources/mnemosyne/mnemosyne/mcp_tools.py:997`; implementation/call-through.

- `sources/mnemosyne/mnemosyne/mcp_tools.py:995` `def _handle_forget_canonical(arguments: Dict[str, Any]) -> Dict[str, Any]:`
- `sources/mnemosyne/mnemosyne/mcp_tools.py:1016` `retired = store.forget(owner_id, category, name)`

**_handle_forget**, body `sources/mnemosyne/mnemosyne/mcp_tools.py:1078`; implementation/call-through.

- `sources/mnemosyne/mnemosyne/mcp_tools.py:1076` `def _handle_forget(arguments: Dict[str, Any]) -> Dict[str, Any]:`
- `sources/mnemosyne/mnemosyne/mcp_tools.py:1083` `ok = mem.forget(memory_id)`
- `sources/mnemosyne/mnemosyne/mcp_tools.py:1084` `return {"status": "deleted" if ok else "not_found", "memory_id": memory_id}`

**forget_working**, body `sources/mnemosyne/mnemosyne/mcp_tools.py:292`; implementation/call-through.

- `sources/mnemosyne/mnemosyne/mcp_tools.py:291` `def forget_working(self, memory_id: str):`
- `sources/mnemosyne/mnemosyne/mcp_tools.py:292` `return self._call_wrapper("forget", memory_id)`

**invalidate**, body `sources/mnemosyne/mnemosyne/mcp_tools.py:295`; implementation/call-through.

- `sources/mnemosyne/mnemosyne/mcp_tools.py:294` `def invalidate(self, memory_id: str, *, replacement_id=None):`
- `sources/mnemosyne/mnemosyne/mcp_tools.py:295` `return self._call_wrapper("invalidate", memory_id, replacement_id=replacement_id)`

</details>

## 3. What reaches the agent

System prompt changes with active backend/persona mode (`sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:2777`). Prefetch filters low-quality or non-distinctive hits, semantically deduplicates, takes _PREFETCH_TOP_K, optionally caps each formatted content and joins timestamp/importance/provenance lines (`sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:2929`; `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:2954`). It can return empty by context/query/backend conditions, not guaranteed recall each turn. Tool JSON assembly is distinct. Five hits is the default top-k; per-memory content truncation defaults to 0 (disabled), so there is no fixed default aggregate character bound (`sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:351`; `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:384`).

Host assembly: `hermes-agent/agent/memory_manager.py:352` joins provider system blocks. `hermes-agent/agent/turn_context.py:1081` requests prefetch for nontrivial turns; `hermes-agent/agent/turn_context.py:120` fences it; `hermes-agent/agent/turn_context.py:136` appends it to API-bound user content. Tool results are a third channel. Optional host spill is at `hermes-agent/agent/memory_manager.py:419`, not a universal provider cap. Built-in memory remains separately controlled (`hermes-agent/agent/system_prompt.py:542`).

<details>
<summary>Context and tool-result assembly quotes</summary>

`_sanitize_prefetch_query`, body `sources/mnemosyne/hermes_memory_provider/__init__.py:652`:

- `sources/mnemosyne/hermes_memory_provider/__init__.py:654` `return sanitize_prefetch_query(query)`

`_configured_tool_schemas`, body `sources/mnemosyne/hermes_memory_provider/__init__.py:2110`:

- `sources/mnemosyne/hermes_memory_provider/__init__.py:2112` `return list(ALL_TOOL_SCHEMAS)`
- `sources/mnemosyne/hermes_memory_provider/__init__.py:2114` `return list(ALL_TOOL_SCHEMAS)`
- `sources/mnemosyne/hermes_memory_provider/__init__.py:2123` `known = ", ".join(sorted(available))`
- `sources/mnemosyne/hermes_memory_provider/__init__.py:2124` `bad = ", ".join(str(name) for name in unknown)`
- `sources/mnemosyne/hermes_memory_provider/__init__.py:2126` `return [available[name] for name in configured]`

`_configured_tool_names`, body `sources/mnemosyne/hermes_memory_provider/__init__.py:2129`:

- `sources/mnemosyne/hermes_memory_provider/__init__.py:2129` `return {schema["name"] for schema in self._configured_tool_schemas()}`

`system_prompt_block`, body `sources/mnemosyne/hermes_memory_provider/__init__.py:2410`:

- `sources/mnemosyne/hermes_memory_provider/__init__.py:2431` `return self._with_persona_block(base)`
- `sources/mnemosyne/hermes_memory_provider/__init__.py:2439` `return (`
- `sources/mnemosyne/hermes_memory_provider/__init__.py:2445` `return ""`

`register_prefetch_source`, body `sources/mnemosyne/hermes_memory_provider/__init__.py:2452`:



`prefetch`, body `sources/mnemosyne/hermes_memory_provider/__init__.py:2459`:

- `sources/mnemosyne/hermes_memory_provider/__init__.py:2460` `return self._prefetch_locked(query, session_id=session_id)`

`_prefetch_locked`, body `sources/mnemosyne/hermes_memory_provider/__init__.py:2468`:

- `sources/mnemosyne/hermes_memory_provider/__init__.py:2469` `return ""`
- `sources/mnemosyne/hermes_memory_provider/__init__.py:2488` `blocks.append(block)`
- `sources/mnemosyne/hermes_memory_provider/__init__.py:2505` `return "\n\n".join(b for b in blocks if b)`

`_prefetch_identity`, body `sources/mnemosyne/hermes_memory_provider/__init__.py:2517`:

- `sources/mnemosyne/hermes_memory_provider/__init__.py:2517` `return render_identity(self._identity_fichas(), existing_blocks, profile)`

`_prefetch_model_slots`, body `sources/mnemosyne/hermes_memory_provider/__init__.py:2527`:

- `sources/mnemosyne/hermes_memory_provider/__init__.py:2527` `return render_model_slots(`

`_prefetch_bank`, body `sources/mnemosyne/hermes_memory_provider/__init__.py:2548`:

- `sources/mnemosyne/hermes_memory_provider/__init__.py:2549` `return render_bank_source(`
- `sources/mnemosyne/hermes_memory_provider/__init__.py:2557` `return ""`

`queue_prefetch`, body `sources/mnemosyne/hermes_memory_provider/__init__.py:2560`:



`get_tool_schemas`, body `sources/mnemosyne/hermes_memory_provider/__init__.py:2925`:

- `sources/mnemosyne/hermes_memory_provider/__init__.py:2925` `return self._configured_tool_schemas()`

`handle_tool_call`, body `sources/mnemosyne/hermes_memory_provider/__init__.py:2930`:

- `sources/mnemosyne/hermes_memory_provider/__init__.py:2931` `return self._handle_tool_call_locked(tool_name, args, **kwargs)`

`_handle_tool_call_locked`, body `sources/mnemosyne/hermes_memory_provider/__init__.py:2935`:

- `sources/mnemosyne/hermes_memory_provider/__init__.py:2948` `return self._dispatch_tool_call(tool_name, args, **kwargs)`

`_dispatch_tool_call`, body `sources/mnemosyne/hermes_memory_provider/__init__.py:2951`:

- `sources/mnemosyne/hermes_memory_provider/__init__.py:2953` `return json.dumps({"error": f"Unknown Mnemosyne tool: {tool_name}"})`
- `sources/mnemosyne/hermes_memory_provider/__init__.py:2955` `return json.dumps({"error": str(exc)})`
- `sources/mnemosyne/hermes_memory_provider/__init__.py:2957` `return json.dumps(self._reflection_skip_response("reflect_disabled_for_cron", "tool"))`
- `sources/mnemosyne/hermes_memory_provider/__init__.py:2967` `return json.dumps({`
- `sources/mnemosyne/hermes_memory_provider/__init__.py:2975` `return self._handle_remember(args)`
- `sources/mnemosyne/hermes_memory_provider/__init__.py:2977` `return self._handle_batch(args)`
- `sources/mnemosyne/hermes_memory_provider/__init__.py:2979` `return self._handle_recall(args)`
- `sources/mnemosyne/hermes_memory_provider/__init__.py:2981` `return self._handle_shared_remember(args)`
- `sources/mnemosyne/hermes_memory_provider/__init__.py:2983` `return self._handle_shared_recall(args)`
- `sources/mnemosyne/hermes_memory_provider/__init__.py:2985` `return self._handle_shared_forget(args)`
- `sources/mnemosyne/hermes_memory_provider/__init__.py:2987` `return self._handle_shared_stats(args)`
- `sources/mnemosyne/hermes_memory_provider/__init__.py:2989` `return self._handle_sleep(args)`
- `sources/mnemosyne/hermes_memory_provider/__init__.py:2991` `return self._handle_resolve_conflicts(args)`
- `sources/mnemosyne/hermes_memory_provider/__init__.py:2993` `return self._handle_stats(args)`
- `sources/mnemosyne/hermes_memory_provider/__init__.py:2995` `return self._handle_invalidate(args)`
- `sources/mnemosyne/hermes_memory_provider/__init__.py:2997` `return self._handle_validate(args)`
- `sources/mnemosyne/hermes_memory_provider/__init__.py:2999` `return self._handle_get(args)`
- `sources/mnemosyne/hermes_memory_provider/__init__.py:3001` `return self._handle_remember_media(args)`
- `sources/mnemosyne/hermes_memory_provider/__init__.py:3003` `return self._handle_triple_add(args)`
- `sources/mnemosyne/hermes_memory_provider/__init__.py:3005` `return self._handle_triple_end(args)`
- `sources/mnemosyne/hermes_memory_provider/__init__.py:3007` `return self._handle_triple_query(args)`
- `sources/mnemosyne/hermes_memory_provider/__init__.py:3009` `return self._handle_remember_canonical(args)`
- `sources/mnemosyne/hermes_memory_provider/__init__.py:3011` `return self._handle_recall_canonical(args)`
- `sources/mnemosyne/hermes_memory_provider/__init__.py:3013` `return self._handle_forget_canonical(args)`
- `sources/mnemosyne/hermes_memory_provider/__init__.py:3015` `return self._handle_apply_pending(args)`
- `sources/mnemosyne/hermes_memory_provider/__init__.py:3017` `return self._handle_model_card(args)`
- `sources/mnemosyne/hermes_memory_provider/__init__.py:3019` `return self._handle_model_refresh(args)`
- `sources/mnemosyne/hermes_memory_provider/__init__.py:3021` `return self._handle_scratchpad_write(args)`
- `sources/mnemosyne/hermes_memory_provider/__init__.py:3023` `return self._handle_scratchpad_read(args)`
- `sources/mnemosyne/hermes_memory_provider/__init__.py:3025` `return self._handle_scratchpad_clear(args)`
- `sources/mnemosyne/hermes_memory_provider/__init__.py:3027` `return self._handle_export(args)`
- `sources/mnemosyne/hermes_memory_provider/__init__.py:3029` `return self._handle_update(args)`
- `sources/mnemosyne/hermes_memory_provider/__init__.py:3031` `return self._handle_forget(args)`
- `sources/mnemosyne/hermes_memory_provider/__init__.py:3033` `return self._handle_import(args)`
- `sources/mnemosyne/hermes_memory_provider/__init__.py:3035` `return self._handle_diagnose(args)`
- `sources/mnemosyne/hermes_memory_provider/__init__.py:3037` `return self._handle_recall_diagnostics(args)`
- `sources/mnemosyne/hermes_memory_provider/__init__.py:3039` `return self._handle_task_progress(args)`
- `sources/mnemosyne/hermes_memory_provider/__init__.py:3041` `return self._handle_graph_query(args)`
- `sources/mnemosyne/hermes_memory_provider/__init__.py:3043` `return self._handle_graph_link(args)`
- `sources/mnemosyne/hermes_memory_provider/__init__.py:3045` `return self._handle_sync_tool(tool_name, args)`
- `sources/mnemosyne/hermes_memory_provider/__init__.py:3047` `return self._handle_persona_tool(tool_name, args)`
- `sources/mnemosyne/hermes_memory_provider/__init__.py:3049` `return json.dumps({"error": f"Unknown Mnemosyne tool: {tool_name}"})`
- `sources/mnemosyne/hermes_memory_provider/__init__.py:3052` `return json.dumps({"error": f"Mnemosyne tool '{tool_name}' failed: {e}"})`

`handle_tool_call`, body `sources/mnemosyne/hermes_memory_provider/persona_adapter.py:48`:

- `sources/mnemosyne/hermes_memory_provider/persona_adapter.py:49` `return json.dumps({`
- `sources/mnemosyne/hermes_memory_provider/persona_adapter.py:55` `return self._promote(**args)`
- `sources/mnemosyne/hermes_memory_provider/persona_adapter.py:57` `return self._demote(**args)`
- `sources/mnemosyne/hermes_memory_provider/persona_adapter.py:59` `return self._list(**args)`
- `sources/mnemosyne/hermes_memory_provider/persona_adapter.py:61` `return self._reinforce(**args)`
- `sources/mnemosyne/hermes_memory_provider/persona_adapter.py:64` `return json.dumps({"status": "error", "error": str(exc)})`
- `sources/mnemosyne/hermes_memory_provider/persona_adapter.py:65` `return json.dumps({"status": "error", "error": f"Unknown tool: {tool_name}"})`

`handle_tool_call`, body `sources/mnemosyne/hermes_memory_provider/sync_adapter.py:268`:

- `sources/mnemosyne/hermes_memory_provider/sync_adapter.py:269` `return json.dumps({`
- `sources/mnemosyne/hermes_memory_provider/sync_adapter.py:276` `return self._handle_push()`
- `sources/mnemosyne/hermes_memory_provider/sync_adapter.py:278` `return self._handle_pull()`
- `sources/mnemosyne/hermes_memory_provider/sync_adapter.py:280` `return self._handle_status()`
- `sources/mnemosyne/hermes_memory_provider/sync_adapter.py:282` `return json.dumps({"status": "error", "error": f"Unknown tool: {tool_name}"})`
- `sources/mnemosyne/hermes_memory_provider/sync_adapter.py:285` `return json.dumps({"status": "error", "error": str(exc)})`

`_prefetch_content_char_limit`, body `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:351`:

- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:353` `return max(0, int(raw))`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:359` `return 0`

`_format_prefetch_content`, body `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:368`:

- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:368` `if limit <= 0 or len(content) <= limit:`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:369` `return content`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:371` `cut = content[:limit].rstrip()`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:372` `# Prefer a word boundary when one exists reasonably close to the limit.`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:374` `if boundary >= max(1, limit // 2):`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:375` `cut = cut[:boundary].rstrip()`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:376` `return f"{cut}..."`

`_prefetch_min_distinctive_tokens`, body `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:431`:

- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:431` `return _parse_bounded_int_env("MNEMOSYNE_PREFETCH_MIN_DISTINCTIVE_TOKENS", 2, 1)`

`_prefetch_min_query_coverage`, body `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:435`:

- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:435` `return _parse_unit_float_env("MNEMOSYNE_PREFETCH_MIN_QUERY_COVERAGE", 0.30)`

`_prefetch_canonical_rare_token_max_frequency`, body `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:439`:

- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:439` `return _parse_bounded_int_env("MNEMOSYNE_PREFETCH_CANONICAL_RARE_TOKEN_MAX_FREQUENCY", 1, 0)`

`_prefetch_canonical_generic_tokens`, body `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:485`:

- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:493` `return configured | extras`

`_is_low_quality_prefetch`, body `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:505`:

- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:507` `return True`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:511` `return True`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:512` `return False`

`_strip_prefetch_prefix`, body `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:516`:

- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:520` `return c[len(prefix):].strip()`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:521` `return c`

`_is_prefetch_cjk_char`, body `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:526`:

- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:526` `return (`

`_prefetch_tokens`, body `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:539`:

- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:552` `return tokens`

`_canonical_prefetch_rows`, body `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:877`:

- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:880` `return []`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:884` `return []`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:897` `tokenized_rows.append((row, body, row_tokens))`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:905` `rare_document_frequency = _prefetch_canonical_rare_token_max_frequency()`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:941` `candidates.append({`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:967` `return candidates[:limit]`

`_prefetch_is_polyphonic`, body `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:979`:

- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:981` `return False`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:982` `return bool(set(vs) & {"vector", "graph", "fact", "temporal"})`

`_prefetch_topic_signal`, body `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:986`:

- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:999` `return signal`

`_prefetch_source_quality`, body `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:1003`:

- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:1007` `return 0.0`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:1019` `return quality`

`_prefetch_is_raw`, body `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:1023`:

- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:1025` `return source in _PREFETCH_RAW_SOURCES or content.startswith("[USER]") or content.startswith("[IDENTITY]")`

`_prefetch_adjusted_score`, body `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:1029`:

- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:1032` `return (score * 0.65 + signal * 0.35 + importance * 0.05) * _prefetch_source_quality(row)`

`_prefetch_has_distinctive_lexical_evidence`, body `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:1037`:

- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:1040` `return (`

`_sanitize_prefetch_query`, body `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:1048`:

- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:1050` `return sanitize_prefetch_query(query)`

`_semantic_dedup_prefetch`, body `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:1054`:

- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:1072` `kept.append(row)`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:1073` `kept_tokens.append(tokens)`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:1074` `return kept`

`_configured_tool_schemas`, body `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:2100`:

- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:2102` `return list(ALL_TOOL_SCHEMAS)`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:2104` `return list(ALL_TOOL_SCHEMAS)`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:2113` `known = ", ".join(sorted(available))`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:2114` `bad = ", ".join(str(name) for name in unknown)`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:2116` `return [available[name] for name in configured]`

`_configured_tool_names`, body `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:2119`:

- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:2119` `return {schema["name"] for schema in self._configured_tool_schemas()}`

`system_prompt_block`, body `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:2778`:

- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:2807` `return self._with_persona_block(base)`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:2815` `return (`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:2829` `return (`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:2834` `return ""`

`prefetch`, body `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:2842`:

- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:2844` `return ""`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:2848` `return ""`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:2851` `return ""`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:2893` `return ""`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:2925` `filtered.append(r)`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:2930` `filtered = _semantic_dedup_prefetch(filtered)[:_PREFETCH_TOP_K]`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:2932` `return ""`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:2934` `content_limit = _prefetch_content_char_limit()`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:2938` `content_limit,`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:2940` `content = " ".join(content.split())`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:2941` `ts = r.get("timestamp", "")[:16] if r.get("timestamp") else ""`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:2954` `lines.append(f"  [{ts}] (importance {imp:.2f}{source_tag}){trust_tag}{prov} {content}")`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:2955` `return "\n".join(lines)`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:2958` `return ""`

`queue_prefetch`, body `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:2961`:



`get_tool_schemas`, body `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:3389`:

- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:3389` `return self._configured_tool_schemas()`

`handle_tool_call`, body `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:3392`:

- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:3394` `return json.dumps({"error": f"Unknown Mnemosyne tool: {tool_name}"})`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:3396` `return json.dumps({"error": str(exc)})`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:3398` `return json.dumps(self._reflection_skip_response("reflect_disabled_for_cron", "tool"))`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:3430` `return _guard_err`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:3444` `return json.dumps({`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:3453` `return self._dispatch_tool_call_locked(tool_name, args)`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:3456` `return json.dumps({"error": f"Mnemosyne tool '{tool_name}' failed: {e}"})`

`_dispatch_tool_call_locked`, body `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:3460`:

- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:3461` `return self._handle_remember(args)`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:3463` `return self._handle_batch(args)`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:3465` `return self._handle_recall(args)`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:3467` `return self._handle_shared_remember(args)`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:3469` `return self._handle_shared_recall(args)`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:3471` `return self._handle_shared_forget(args)`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:3473` `return self._handle_shared_stats(args)`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:3475` `return self._handle_sleep(args)`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:3477` `return self._handle_resolve_conflicts(args)`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:3479` `return self._handle_stats(args)`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:3481` `return self._handle_invalidate(args)`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:3483` `return self._handle_validate(args)`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:3485` `return self._handle_get(args)`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:3487` `return self._handle_remember_media(args)`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:3489` `return self._handle_triple_add(args)`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:3491` `return self._handle_triple_query(args)`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:3493` `return self._handle_triple_end(args)`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:3495` `return self._handle_remember_canonical(args)`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:3497` `return self._handle_recall_canonical(args)`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:3499` `return self._handle_forget_canonical(args)`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:3501` `return self._handle_apply_pending(args)`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:3503` `return self._handle_model_card(args)`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:3505` `return self._handle_model_refresh(args)`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:3507` `return self._handle_scratchpad_write(args)`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:3509` `return self._handle_scratchpad_read(args)`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:3511` `return self._handle_scratchpad_clear(args)`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:3513` `return self._handle_export(args)`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:3515` `return self._handle_update(args)`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:3517` `return self._handle_forget(args)`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:3519` `return self._handle_import(args)`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:3521` `return self._handle_diagnose(args)`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:3523` `return self._handle_recall_diagnostics(args)`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:3525` `return self._handle_task_progress(args)`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:3527` `return self._handle_graph_query(args)`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:3529` `return self._handle_graph_link(args)`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:3531` `return self._handle_sync_tool(tool_name, args)`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:3533` `return self._handle_persona_tool(tool_name, args)`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:3535` `return json.dumps({"error": f"Unknown Mnemosyne tool: {tool_name}"})`

`handle_tool_call`, body `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/persona_adapter.py:48`:

- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/persona_adapter.py:49` `return json.dumps({`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/persona_adapter.py:55` `return self._promote(**args)`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/persona_adapter.py:57` `return self._demote(**args)`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/persona_adapter.py:59` `return self._list(**args)`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/persona_adapter.py:61` `return self._reinforce(**args)`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/persona_adapter.py:64` `return json.dumps({"status": "error", "error": str(exc)})`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/persona_adapter.py:65` `return json.dumps({"status": "error", "error": f"Unknown tool: {tool_name}"})`

`handle_tool_call`, body `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/sync_adapter.py:268`:

- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/sync_adapter.py:269` `return json.dumps({`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/sync_adapter.py:276` `return self._handle_push()`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/sync_adapter.py:278` `return self._handle_pull()`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/sync_adapter.py:280` `return self._handle_status()`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/sync_adapter.py:282` `return json.dumps({"status": "error", "error": f"Unknown tool: {tool_name}"})`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/sync_adapter.py:285` `return json.dumps({"status": "error", "error": str(exc)})`

</details>

## 4. Data access boundaries

Local SQLite/bank files are reachable independently by the same OS user. Accountless local mode `yes`; optional embedding/model downloads and sync endpoints are not implied to be offline. Credential/env/model/sync paths and file modes are enumerated below. A provider ownership predicate is not an OS access boundary (`sources/mnemosyne/mnemosyne/core/beam.py:8003`). Telemetry defaults across optional dependencies `unknown`.

Boundary model: same OS user, terminal, mounted filesystem and allowed network. Direct reachability means another client can use the same file/endpoint and available credentials, not unauthenticated access or encryption bypass. Hermes setup persists declared secrets through `hermes-agent/hermes_cli/memory_setup.py:321` and `hermes-agent/hermes_cli/config.py:2697`. These are possible configured paths; no live credentials were read.

Host credential-file detail: `get_hermes_home()/.env` is resolved at `hermes-agent/hermes_cli/config.py:473`. New files are secured to 0600 at `hermes-agent/hermes_cli/config.py:506`; the writer preserves an existing mode at `hermes-agent/hermes_cli/config.py:2544` and `:2562`. Therefore 0600 is not an unconditional mode claim for every existing profile.

<details>
<summary>Paths, environment, permissions, egress, telemetry and process-launch evidence</summary>

- `sources/mnemosyne/hermes_memory_provider/__init__.py:353` `raw = os.environ.get("MNEMOSYNE_SYNC_TURN_USER_LIMIT", "500").strip()`
- `sources/mnemosyne/hermes_memory_provider/__init__.py:370` `raw = os.environ.get("MNEMOSYNE_SYNC_TURN_ASSISTANT_LIMIT", "800").strip()`
- `sources/mnemosyne/hermes_memory_provider/__init__.py:937` `"Pass an https:// URL, a data: URI, a blob:// reference, or an absolute local "`
- `sources/mnemosyne/hermes_memory_provider/__init__.py:947` `"description": "https:// URL, data: URI, blob://sha256/... reference, or an absolute local path inside MNEMOSYNE_MEDIA_ALLOWED_PATHS.",`
- `sources/mnemosyne/hermes_memory_provider/__init__.py:1532` `val = os.environ.get(key)`
- `sources/mnemosyne/hermes_memory_provider/__init__.py:1559` `return _coerce_bool(os.environ.get(key), default)`
- `sources/mnemosyne/hermes_memory_provider/__init__.py:1575` `return _coerce_optional_int(os.environ.get(key), default)`
- `sources/mnemosyne/hermes_memory_provider/__init__.py:1667` `self._sync_turn_telemetry: Dict[str, Any] = {`
- `sources/mnemosyne/hermes_memory_provider/__init__.py:1698` `_sync_env = os.environ.get("MNEMOSYNE_SYNC_ROLES")`
- `sources/mnemosyne/hermes_memory_provider/__init__.py:1706` `_skip_env = os.environ.get("MNEMOSYNE_SKIP_CONTEXTS")`
- `sources/mnemosyne/hermes_memory_provider/__init__.py:1712` `os.environ.get("MNEMOSYNE_PREFETCH_PROFILE", "general").strip() or "general"`
- `sources/mnemosyne/hermes_memory_provider/__init__.py:1796` `db_path = getattr(self._beam, "db_path", None)`
- `sources/mnemosyne/hermes_memory_provider/__init__.py:1797` `if db_path:`
- `sources/mnemosyne/hermes_memory_provider/__init__.py:1798` `self._audit = AuditLog(Path(db_path))`
- `sources/mnemosyne/hermes_memory_provider/__init__.py:1799` `logger.debug("Audit log initialized: %s", db_path)`
- `sources/mnemosyne/hermes_memory_provider/__init__.py:2364` `self._session_id, bank_name, mem.db_path,`
- `sources/mnemosyne/hermes_memory_provider/__init__.py:2562` `def _ensure_sync_turn_telemetry(self) -> None:`
- `sources/mnemosyne/hermes_memory_provider/__init__.py:2563` `"""Initialize sync_turn telemetry for tests that construct via __new__."""`
- `sources/mnemosyne/hermes_memory_provider/__init__.py:2567` `if not hasattr(self, "_sync_turn_telemetry"):`
- `sources/mnemosyne/hermes_memory_provider/__init__.py:2568` `self._sync_turn_telemetry = {`
- `sources/mnemosyne/hermes_memory_provider/__init__.py:2655` `"""Return a PII-safe snapshot of sync_turn telemetry."""`
- `sources/mnemosyne/hermes_memory_provider/__init__.py:2656` `self._ensure_sync_turn_telemetry()`
- `sources/mnemosyne/hermes_memory_provider/__init__.py:2658` `return dict(self._sync_turn_telemetry)`
- `sources/mnemosyne/hermes_memory_provider/__init__.py:2675` `self._ensure_sync_turn_telemetry()`
- `sources/mnemosyne/hermes_memory_provider/__init__.py:2677` `self._sync_turn_telemetry["in_flight"] += 1`
- `sources/mnemosyne/hermes_memory_provider/__init__.py:2678` `in_flight = int(self._sync_turn_telemetry["in_flight"])`
- `sources/mnemosyne/hermes_memory_provider/__init__.py:2682` `self._sync_turn_telemetry["pending_queue_length"] = in_flight`
- `sources/mnemosyne/hermes_memory_provider/__init__.py:2683` `self._sync_turn_telemetry["max_queue_length"] = max(`
- `sources/mnemosyne/hermes_memory_provider/__init__.py:2684` `int(self._sync_turn_telemetry.get("max_queue_length") or 0),`
- `sources/mnemosyne/hermes_memory_provider/__init__.py:2735` `self._sync_turn_telemetry["completed"] += 1`
- `sources/mnemosyne/hermes_memory_provider/__init__.py:2736` `self._sync_turn_telemetry["last_error"] = None`
- `sources/mnemosyne/hermes_memory_provider/__init__.py:2739` `self._sync_turn_telemetry["failed"] += 1`
- `sources/mnemosyne/hermes_memory_provider/__init__.py:2740` `self._sync_turn_telemetry["last_error"] = self._sanitize_sync_turn_error(e)`
- `sources/mnemosyne/hermes_memory_provider/__init__.py:2746` `self._sync_turn_telemetry["in_flight"] = max(0, self._sync_turn_telemetry["in_flight"] - 1)`
- `sources/mnemosyne/hermes_memory_provider/__init__.py:2747` `self._sync_turn_telemetry["pending_queue_length"] = int(self._sync_turn_telemetry["in_flight"])`
- `sources/mnemosyne/hermes_memory_provider/__init__.py:2748` `self._sync_turn_telemetry["last_duration_ms"] = duration_ms`
- `sources/mnemosyne/hermes_memory_provider/__init__.py:2749` `self._sync_turn_telemetry["max_duration_ms"] = max(`
- `sources/mnemosyne/hermes_memory_provider/__init__.py:2750` `float(self._sync_turn_telemetry.get("max_duration_ms") or 0.0),`
- `sources/mnemosyne/hermes_memory_provider/__init__.py:2754` `self._sync_turn_telemetry["slow_sync_count"] += 1`
- `sources/mnemosyne/hermes_memory_provider/__init__.py:2755` `snapshot = dict(self._sync_turn_telemetry)`
- `sources/mnemosyne/hermes_memory_provider/__init__.py:2886` `"db_path": beam.db_path,`
- `sources/mnemosyne/hermes_memory_provider/__init__.py:3397` `self._surface_beam = BeamMemory(session_id="hermes_shared_surface", db_path=shared_path)`
- `sources/mnemosyne/hermes_memory_provider/__init__.py:3908` `db_path=self._beam.db_path)`
- `sources/mnemosyne/hermes_memory_provider/__init__.py:3920` `db_path=self._beam.db_path)`
- `sources/mnemosyne/hermes_memory_provider/__init__.py:3930` `as_of=as_of, db_path=self._beam.db_path)`
- `sources/mnemosyne/hermes_memory_provider/__init__.py:4022` `store = CanonicalStore(db_path=self._beam.db_path, conn=self._beam.conn)`
- `sources/mnemosyne/hermes_memory_provider/__init__.py:4042` `store = CanonicalStore(db_path=self._beam.db_path, conn=self._beam.conn)`
- `sources/mnemosyne/hermes_memory_provider/__init__.py:4367` `if _os.environ.get("MNEMOSYNE_RECALL_DIAGNOSTICS", "0") != "1":`
- `sources/mnemosyne/hermes_memory_provider/__init__.py:4406` `store = CanonicalStore(db_path=self._beam.db_path, conn=self._beam.conn)`
- `sources/mnemosyne/hermes_memory_provider/__init__.py:4493` `mem = Mnemosyne(session_id=self._session_id, db_path=self._beam.db_path)`
- `sources/mnemosyne/hermes_memory_provider/__init__.py:4543` `mem = Mnemosyne(session_id=self._session_id, db_path=self._beam.db_path)`
- `sources/mnemosyne/hermes_memory_provider/__init__.py:4555` `api_key = os.environ.get(env_key, "")`
- `sources/mnemosyne/hermes_memory_provider/__init__.py:4610` `active_db = getattr(self._beam, "db_path", None)`
- `sources/mnemosyne/hermes_memory_provider/__init__.py:4615` `result["active_provider_db_path"] = str(active_db)`
- `sources/mnemosyne/hermes_memory_provider/__init__.py:4736` `"db_path": beam.db_path,`

- `sources/mnemosyne/hermes_memory_provider/audit.py:51` `def __init__(self, db_path: Path):`
- `sources/mnemosyne/hermes_memory_provider/audit.py:52` `self._db_path = db_path`
- `sources/mnemosyne/hermes_memory_provider/audit.py:61` `str(self._db_path), timeout=5, check_same_thread=False`

- `sources/mnemosyne/hermes_memory_provider/cli.py:361` `db_path = getattr(args, "db_path", None)`
- `sources/mnemosyne/hermes_memory_provider/cli.py:371` `"db_path": db_path,`
- `sources/mnemosyne/hermes_memory_provider/cli.py:396` `api_key = os.environ.get(pk)`

- `sources/mnemosyne/hermes_memory_provider/sync_adapter.py:19` `MNEMOSYNE_SYNC_REMOTE     , remote sync server URL (https://host:port)`
- `sources/mnemosyne/hermes_memory_provider/sync_adapter.py:135` `host = os.environ.get("MNEMOSYNE_SYNC_HOST", "").strip()`
- `sources/mnemosyne/hermes_memory_provider/sync_adapter.py:136` `port = os.environ.get("MNEMOSYNE_SYNC_PORT", "").strip()`
- `sources/mnemosyne/hermes_memory_provider/sync_adapter.py:138` `return f"https://{host}:{port}"`
- `sources/mnemosyne/hermes_memory_provider/sync_adapter.py:153` `return os.environ.get("MNEMOSYNE_SYNC_KEY", "").strip()`
- `sources/mnemosyne/hermes_memory_provider/sync_adapter.py:174` `env_val = os.environ.get(env_key, "").strip()`

- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:255` `PERSONA_FILE = Path.home() / ".hermes" / "memory" / "persona.md"`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:278` `def _release_thread_connection(db_path=None) -> None:`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:297` `close_thread_connection(db_path)`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:351` `raw = os.environ.get("MNEMOSYNE_PREFETCH_CONTENT_CHARS", "0").strip()`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:406` `raw = os.environ.get(key, "").strip()`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:417` `raw = os.environ.get(key, "").strip()`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:449` `raw = os.environ.get(key, "").strip()`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:1083` `raw = os.environ.get("MNEMOSYNE_SYNC_TURN_USER_LIMIT", "500").strip()`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:1100` `raw = os.environ.get("MNEMOSYNE_SYNC_TURN_ASSISTANT_LIMIT", "800").strip()`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:1395` `val = os.environ.get(key)`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:1422` `return _coerce_bool(os.environ.get(key), default)`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:1438` `return _coerce_optional_int(os.environ.get(key), default)`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:1554` `self._sync_turn_telemetry: Dict[str, Any] = {`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:1588` `_skip_env = os.environ.get("MNEMOSYNE_SKIP_CONTEXTS")`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:1682` `db_path = getattr(self._beam, "db_path", None)`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:1683` `if db_path:`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:1684` `self._audit = AuditLog(Path(db_path))`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:1685` `logger.debug("Audit log initialized: %s", db_path)`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:1850` `hermes_home=self._hermes_home or os.environ.get("HERMES_HOME", ""),`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:1967` `_sync_raw = os.environ.get("MNEMOSYNE_SYNC_ROLES", "user")`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:2685` `private_db_path = None`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:2692` `private_db_path = (`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:2700` `db_path=private_db_path,`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:2709` `self._session_id, bank_name, mem.db_path,`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:2713` `db_path = (`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:2718` `beam_kwargs = {"session_id": self._session_id, "db_path": db_path}`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:2725` `self._session_id, db_path or "default",`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:2886` `store = CanonicalStore(db_path=beam.db_path, conn=beam.conn)`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:2952` `_home = str(getattr(self._beam, "db_path", "") or "?")`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:2963` `def _ensure_sync_turn_telemetry(self) -> None:`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:2964` `"""Initialize sync_turn telemetry for tests that construct via __new__."""`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:2968` `if not hasattr(self, "_sync_turn_telemetry"):`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:2969` `self._sync_turn_telemetry = {`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:3121` `"""Return a PII-safe snapshot of sync_turn telemetry."""`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:3122` `self._ensure_sync_turn_telemetry()`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:3124` `return dict(self._sync_turn_telemetry)`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:3142` `self._ensure_sync_turn_telemetry()`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:3144` `self._sync_turn_telemetry["in_flight"] += 1`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:3145` `in_flight = int(self._sync_turn_telemetry["in_flight"])`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:3149` `self._sync_turn_telemetry["pending_queue_length"] = in_flight`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:3150` `self._sync_turn_telemetry["max_queue_length"] = max(`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:3151` `int(self._sync_turn_telemetry.get("max_queue_length") or 0),`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:3217` `self._sync_turn_telemetry["completed"] += 1`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:3218` `self._sync_turn_telemetry["last_error"] = None`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:3221` `self._sync_turn_telemetry["failed"] += 1`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:3222` `self._sync_turn_telemetry["last_error"] = self._sanitize_sync_turn_error(e)`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:3228` `self._sync_turn_telemetry["in_flight"] = max(0, self._sync_turn_telemetry["in_flight"] - 1)`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:3229` `self._sync_turn_telemetry["pending_queue_length"] = int(self._sync_turn_telemetry["in_flight"])`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:3230` `self._sync_turn_telemetry["last_duration_ms"] = duration_ms`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:3231` `self._sync_turn_telemetry["max_duration_ms"] = max(`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:3232` `float(self._sync_turn_telemetry.get("max_duration_ms") or 0.0),`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:3236` `self._sync_turn_telemetry["slow_sync_count"] += 1`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:3237` `snapshot = dict(self._sync_turn_telemetry)`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:3310` `"db_path": beam_ref.db_path,`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:3368` `_release_thread_connection(sleep_args.get("db_path"))`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:3813` `store = CanonicalStore(db_path=self._beam.db_path, conn=self._beam.conn)`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:3891` `shared_path = self._shared_surface_path or (Path.home() / ".mnemosyne" / "data" / "shared" / "mnemosyne.db")`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:3894` `self._surface_beam = BeamMemory(session_id="hermes_shared_surface", db_path=shared_path)`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:4414` `db_path=self._beam.db_path)`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:4426` `db_path=self._beam.db_path)`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:4437` `as_of=as_of, db_path=self._beam.db_path)`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:4500` `store = CanonicalStore(db_path=self._beam.db_path, conn=self._beam.conn)`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:4544` `store = CanonicalStore(db_path=self._beam.db_path, conn=self._beam.conn)`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:4576` `store = CanonicalStore(db_path=self._beam.db_path, conn=self._beam.conn)`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:4854` `store = CanonicalStore(db_path=self._beam.db_path, conn=self._beam.conn)`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:4902` `mem = Mnemosyne(session_id=self._session_id, db_path=self._beam.db_path)`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:4952` `mem = Mnemosyne(session_id=self._session_id, db_path=self._beam.db_path)`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:4964` `api_key = os.environ.get(env_key, "")`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:5017` `active_db = getattr(self._beam, "db_path", None)`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:5022` `result["active_provider_db_path"] = str(active_db)`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:5073` `if _os.environ.get("MNEMOSYNE_RECALL_DIAGNOSTICS", "0") != "1":`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:5117` `store = CanonicalStore(db_path=self._beam.db_path, conn=self._beam.conn)`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:5351` `"db_path": beam.db_path,`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:5380` `_release_thread_connection(sleep_args.get("db_path"))`

- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/audit.py:26` `timeout_ms = int(os.environ.get("MNEMOSYNE_BUSY_TIMEOUT_MS", "5000"))`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/audit.py:63` `def __init__(self, db_path: Path):`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/audit.py:64` `self._db_path = db_path`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/audit.py:73` `str(self._db_path), timeout=5, check_same_thread=False`

- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/cli.py:152` `def _export_schema_is_complete_read_only(db_path: Path) -> bool:`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/cli.py:154` `db_uri = f"{db_path.resolve().as_uri()}?mode=ro"`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/cli.py:299` `hermes_home = os.environ.get("HERMES_HOME", "")`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/cli.py:354` `from mnemosyne.core.banks import get_bank_db_path_read_only`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/cli.py:355` `db_path = get_bank_db_path_read_only(bank)`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/cli.py:356` `if not db_path.is_file():`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/cli.py:359` `if not _export_schema_is_complete_read_only(db_path):`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/cli.py:601` `db_path = getattr(args, "db_path", None)`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/cli.py:611` `"db_path": db_path,`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/cli.py:636` `api_key = os.environ.get(pk)`

- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:131` `return subprocess.list2cmdline(`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:138` `return Path(os.environ.get("HERMES_HOME") or "~/.hermes").expanduser()`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:475` `subprocess.list2cmdline(retry_args) if os.name == "nt" else shlex.join(retry_args)`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:486` `result = subprocess.run(`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:492` `except subprocess.TimeoutExpired as exc:`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:544` `result = subprocess.run(`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:555` `for key, value in os.environ.items()`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:562` `except subprocess.TimeoutExpired:`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:1159` `Path.home() / "hermes-agent",`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:1190` `ve = os.environ.get("VIRTUAL_ENV")`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:1208` `result = subprocess.run(cmd, capture_output=True, text=True, timeout=120)`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:1237` `result = subprocess.run(`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:2315` `result = subprocess.run(`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:2333` `except (OSError, subprocess.TimeoutExpired) as exc:`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:2770` `environment = {k: v for k, v in os.environ.items()`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:2773` `result = subprocess.run([str(python), "-I", "-B", "-c", code, str(root), str(candidate)],`

- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/sync_adapter.py:19` `MNEMOSYNE_SYNC_REMOTE     , remote sync server URL (https://host:port)`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/sync_adapter.py:135` `host = os.environ.get("MNEMOSYNE_SYNC_HOST", "").strip()`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/sync_adapter.py:136` `port = os.environ.get("MNEMOSYNE_SYNC_PORT", "").strip()`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/sync_adapter.py:138` `return f"https://{host}:{port}"`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/sync_adapter.py:153` `return os.environ.get("MNEMOSYNE_SYNC_KEY", "").strip()`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/sync_adapter.py:174` `env_val = os.environ.get(env_key, "").strip()`

- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/tools.py:298` `"Pass an https:// URL, a data: URI, a blob:// reference, or an absolute local "`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/tools.py:308` `"description": "https:// URL, data: URI, blob://sha256/... reference, or an absolute local path inside MNEMOSYNE_MEDIA_ALLOWED_PATHS.",`

- `sources/mnemosyne/mnemosyne/core/config.py:38` `"db_path",`
- `sources/mnemosyne/mnemosyne/core/config.py:40` `"shared_db_path",`
- `sources/mnemosyne/mnemosyne/core/config.py:71` `"db_path": "MNEMOSYNE_DB_PATH",`
- `sources/mnemosyne/mnemosyne/core/config.py:74` `"shared_db_path": "MNEMOSYNE_SHARED_DB_PATH",`
- `sources/mnemosyne/mnemosyne/core/config.py:217` `"db_path": "",`
- `sources/mnemosyne/mnemosyne/core/config.py:220` `"shared_db_path": "",`
- `sources/mnemosyne/mnemosyne/core/config.py:359` `data_dir = os.environ.get("MNEMOSYNE_DATA_DIR")`
- `sources/mnemosyne/mnemosyne/core/config.py:362` `hermes_home = os.environ.get("HERMES_HOME")`
- `sources/mnemosyne/mnemosyne/core/config.py:365` `return Path.home() / ".hermes" / "mnemosyne" / "config.yaml"`
- `sources/mnemosyne/mnemosyne/core/config.py:444` `if env_var and env_var in os.environ:`
- `sources/mnemosyne/mnemosyne/core/config.py:445` `env_val = os.environ[env_var]`
- `sources/mnemosyne/mnemosyne/core/config.py:472` `env_count = sum(1 for k in seed_data if ENV_VAR_MAP.get(k, "") in os.environ)`
- `sources/mnemosyne/mnemosyne/core/config.py:635` `val = os.environ.get(env_var)`
- `sources/mnemosyne/mnemosyne/core/config.py:668` `env_value = os.environ.get(env_var) if env_var else None`
- `sources/mnemosyne/mnemosyne/core/config.py:791` `val = os.environ.get(env_var)`
- `sources/mnemosyne/mnemosyne/core/config.py:858` `val = os.environ.get(env_var)`

- `sources/mnemosyne/mnemosyne/core/sqlite_config.py:15` `os.environ.get("MNEMOSYNE_BUSY_TIMEOUT_MS", str(DEFAULT_BUSY_TIMEOUT_MS))`

- `sources/mnemosyne/mnemosyne/hermes_config.py:73` `resolved_home = hermes_home or os.environ.get("HERMES_HOME")`

</details>

## 5. Reproducibility

`range`: mnemosyne-hermes>=0.7.7,<0.8 and mnemosyne-memory[embeddings]>=4.0.0 (`sources/mnemosyne/integrations/hermes-catalog/pyproject.toml:10`). Engine has no upper bound. Legacy adapter source in the repo is not proof that it is the installed entrypoint.

Retained lock/checksum files: none in snapshot. Upstream completeness is unknown: fetch.py:15 excludes .lock files, fetch.py:12 excludes build/vendor directories. Absence here is not absence upstream.

- `sources/mnemosyne/hermes_memory_provider/plugin.yaml:2` `version: 4.0.0`
- `sources/mnemosyne/hermes_memory_provider/plugin.yaml:45` `dependencies: []`

- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:1` `"""Installer CLI for the Mnemosyne Hermes memory provider."""`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:7` `# This installer can also be loaded directly by the core-free console script.`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:9` `sys.platform == "linux" and hasattr(sys, "getandroidapilevel")`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:11` `_platform_label = "android" if sys.platform == "linux" else sys.platform`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:55` `"""Detailed installation state for the Hermes plugin directory."""`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:58` `installed: bool`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:90` `"""Detailed installation state for the bundled Hermes skill."""`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:93` `installed: bool`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:99` `class SkillInstallResult:`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:100` `"""Result of installing, skipping, or planning the bundled skill."""`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:110` `"""Installer-owned plugin, profile-link, and skill paths in one Hermes home."""`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:126` `return getattr(error, "winerror", None) == 1314`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:132` `["mnemosyne-hermes", "install", "--mode", "wrapper", "--python", str(hermes_python)]`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:137` `"""Return the Hermes home directory used for user-installed plugins."""`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:142` `"""Return the installed mnemosyne_hermes package directory.`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:145` `package ''__init__'' before this module, and install/status commands should`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:146` `remain useful even when the Mnemosyne core dependency is unavailable or`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:166` `"""Return the deterministic install target for the bundled Hermes skill.`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:184` `"""Return the persistent path boundary owned by the Mnemosyne installer."""`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:220` `"""Return state for the bundled Hermes skill install target."""`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:224` `status="installed",`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:225` `installed=True,`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:227` `message="Bundled memory override skill is installed.",`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:232` `installed=False,`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:238` `installed=False,`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:250` `"""Return the sidecar path used to track installer-managed skill content."""`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:260` `"""Return whether target still matches the installer-managed sidecar hash."""`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:268` `return expected == _sha256_text(target.read_text(encoding="utf-8"))`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:274` `"""Record the digest for installer-managed skill content."""`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:278` `def install_bundled_skill(`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:283` `) -> SkillInstallResult:`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:284` `"""Install the bundled memory override skill into Hermes' skills directory."""`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:290` `up_to_date = target.is_file() and target.read_text(encoding="utf-8") == content`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:295` `return SkillInstallResult(`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:301` `return SkillInstallResult(`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:308` `action = "refresh" if exists and managed_copy else ("overwrite" if exists else "install")`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:311` `return SkillInstallResult(`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:330` `verb = {"install": "Installed", "overwrite": "Overwrote", "refresh": "Refreshed"}[action]`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:331` `return SkillInstallResult(`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:349` `"""Return whether a plugin has installer-owned Mnemosyne identity markers."""`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:419` `or data.get("schema_version") != 1`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:439` `if not math.isfinite(timeout) or timeout <= 0:`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:461` `"installer validation."`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:466` `"install",`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:475` `subprocess.list2cmdline(retry_args) if os.name == "nt" else shlex.join(retry_args)`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:529` `# installer's CLI preflight in the selected interpreter.`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:537` `+ "print(getattr(mnemosyne_hermes, '__version__', 'unknown'))\n"`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:542` `# installer's current directory to satisfy the selected runtime.`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:564` `if result.returncode == 0:`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:591` `installed=False,`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:598` `"or package reinstall)."`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:605` `installed=False,`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:614` `installed=False,`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:624` `installed=False,`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:635` `installed=False,`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:675` `installed=False,`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:684` `if status == "invalid_wrapper"`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:690` `status="installed",`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:691` `installed=True,`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:699` `message="Plugin is installed and discoverable.",`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:710` `if not (name[0].isalpha() or name[0] == "_"):`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:712` `if not all(char.isalnum() or char == "_" for char in name):`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:728` `if token == "--":`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:751` `pipx layout the launcher branch exists to serve.`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:753` `return any(token.split(maxsplit=1)[:1] == ["exec"] for token in tokens if token)`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:830` `if rest[0] == "env":`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:869` `for pipx / package entry points.`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:927` `"""Return whether the installer is running on native Windows."""`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:928` `return os.name == "nt"`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:931` `DEFAULT_INSTALL_MODE_POSIX = "symlink"`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:932` `DEFAULT_INSTALL_MODE_WINDOWS = "wrapper"`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:935` `def default_install_mode() -> str:`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:936` `"""Return the install mode to use when the caller did not choose one.`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:942` `one of those, ''os.symlink'' raises ''WinError 1314'' and the install fails.`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:944` `That is why native Windows installs have been reported as working for some`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:950` `return DEFAULT_INSTALL_MODE_WINDOWS if _is_windows_platform() else DEFAULT_INSTALL_MODE_POSIX`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:983` `so the launcher, the known install roots, ''sys.prefix'' and ''VIRTUAL_ENV''`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:989` `runs it as ''<python> -m pip install ...'', so a file that cannot be`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:1004` `"interpreter, or omit --python to let the installer find it."`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:1013` `def _hermes_install_key(project_root: Path) -> str:`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:1014` `"""Return Hermes' PM state key for a checkout (''pm.environments.install_key'')."""`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:1024` `''<home>/installs/<key>/environments/<generation>/venv'', which Hermes' PM`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:1025` `selects through ''installs/<key>/facts.json'' (''packages.venv.environment'').`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:1026` `''<key>'' is derived from the checkout path, so the active install is found`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:1027` `by construction: ''installs/'' is never scanned and no directory is chosen`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:1028` `among several. This mirrors ''pm.environments.selected_venv''; the installer`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:1035` `used (unreadable, outside this install's ''environments/'', or without an`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:1041` `facts = hermes_home_path / "installs" / _hermes_install_key(project_root) / "facts.json"`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:1083` `"""Try to find Hermes' Python executable for dependency validation.`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:1089` `provider runs from a staged generation under ''<home>/installs/''. For a`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:1132` `# selected plugin home to a launcher, active venv, or global install that`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:1136` `# A pip/pipx-installed Hermes puts its console script next to the`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:1202` `"""Install mnemosyne-hermes into Hermes' Python venv."""`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:1203` `from . import __version__`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:1204` `pkg_name = f"mnemosyne-hermes[all]=={__version__}"`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:1205` `cmd = [str(hermes_python), "-m", "pip", "install", "--upgrade", pkg_name]`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:1206` `print(f"  Installing {pkg_name} into {hermes_python.parent.parent.name}...")`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:1213` `print("  ✓ mnemosyne-hermes installed into Hermes' venv")`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:1221` `"""Verify mnemosyne-memory core library is installed."""`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:1225` `print(f"  mnemosyne-memory {mnemosyne.__version__} installed")`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:1232` `"""Check if Hermes' Python can import Mnemosyne's operational dependencies.`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:1234` `Returns the version string if importable, None otherwise.`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:1239` `"import mnemosyne; print(mnemosyne.__version__); "`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:1243` `if result.returncode == 0:`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:1271` `return memory.get("provider") == "mnemosyne"`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:1279` `''memory.provider == "mnemosyne"'' (see ''_config_selects_mnemosyne'').`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:1280` `Symlinked profile entries are skipped (the installer must not follow a`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:1283` `directory exists (the default, no-profile install).`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:1308` `"""Return whether Hermes runs in a different environment from the installer.`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:1338` `already = target.resolve() == source.resolve()`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:1390` `"""Return the installer-managed profile-link preference for a Hermes home."""`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:1432` `New installs persist the explicit selection, which lets upgrades distinguish`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:1433` `the default enabled behavior from an explicit root-only installation even`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:1434` `when no opted-in child profile exists yet. Existing installs without the`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:1458` `if profile_target.resolve() == expected:`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:1483` `ok = target.resolve() == source.resolve()`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:1520` `left.link_target == right.link_target`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:1521` `and left.device == right.device`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:1522` `and left.inode == right.inode`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:1702` `and unrecognized installation layouts are outside its scope.`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:1709` `same = lambda a, b: a.casefold() == b.casefold()`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:1713` `same = lambda a, b: a == b`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:1714` `prefix = (*root.parts, "installs")`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:1735` `"For persistence, install Mnemosyne in a side venv outside installs/ "`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:1746` `"""Validate the selected wrapper runtime before touching an installed plugin."""`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:1754` `"interpreter, or omit --python to let the installer find it."`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:1835` `"schema_version": 1,`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:1854` `sys.platform == "linux" and hasattr(sys, "getandroidapilevel")`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:1856` `_platform_label = "android" if sys.platform == "linux" else sys.platform`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:1864` `\"\"\"Reject a selected virtualenv that targets another Python minor version.\"\"\"`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:1873` `selected_version = None`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:1880` `if separator and name.strip().lower() in {"version_info", "version"}:`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:1883` `selected_version = (int(match.group(1)), int(match.group(2)))`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:1885` `runtime_version = sys.version_info[:2]`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:1886` `if selected_version is None or selected_version != runtime_version:`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:1888` `f"{selected_version[0]}.{selected_version[1]}"`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:1889` `if selected_version is not None`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:1894` `f"runtime Python {runtime_version[0]}.{runtime_version[1]}; "`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:1897` `"then reinstall mnemosyne-hermes."`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:1911` `if not isinstance(manifest, dict) or manifest.get("schema_version") != 1:`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:1937` `# Setuptools' PEP 660 editable installs use a .pth-installed finder, so a`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:1960` `if name == package_name or name.startswith(package_name + ".")`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:1979` `sys.platform == "linux" and hasattr(sys, "getandroidapilevel")`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:1981` `_platform_label = "android" if sys.platform == "linux" else sys.platform`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:1998` `sys.platform == "linux" and hasattr(sys, "getandroidapilevel")`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:2000` `_platform_label = "android" if sys.platform == "linux" else sys.platform`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:2033` `def install_plugin(`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:2043` `"""Install the Mnemosyne provider into Hermes' user plugin directory.`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:2048` `install ''.pth'' files) and imports ''mnemosyne_hermes'' from there.`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:2050` `opted-in profile fan-out by default; set it False to install only at the`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:2054` `mode = default_install_mode()`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:2067` `mode == "symlink"`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:2107` `if mode == "symlink":`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:2183` `def uninstall_plugin(*, hermes_home_path: str | Path | None = None) -> Path:`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:2272` `def is_installed(*, hermes_home_path: str | Path | None = None) -> bool:`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:2273` `"""Return whether the Mnemosyne provider is installed for Hermes discovery."""`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:2274` `return plugin_state(hermes_home_path=hermes_home_path).installed`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:2277` `def _distribution_version(distribution: str) -> str:`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:2278` `"""Return an installed distribution version without importing package globals."""`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:2280` `return importlib.metadata.version(distribution)`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:2295` `"""Print the selected Hermes interpreter and version as JSON."""`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:2325` `"'version': platform.python_version()}))"`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:2340` `version = payload.get("version") if isinstance(payload, dict) else None`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:2345` `or not isinstance(version, str)`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:2346` `or not version`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:2349` `if result.returncode == 0 and not result.stderr.strip():`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:2352` `f"Could not read Python version from {python}: {detail}"`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:2355` `print(json.dumps({"ok": True, "python": str(python), "version": version}))`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:2368` `result = {"ownership": "unknown", "plugin": "unknown", "requirements": "unknown",`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:2440` `version = metadata.get("Version", "")`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:2441` `if metadata.get_all("Name") != [name] or len(metadata.get_all("Version", [])) != 1 or not re.fullmatch(r"\d+(?:\.\d+)*(?:(?:a|b|rc)\d+)?", version):`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:2442` `raise ValueError("unsupported distribution identity/version: " + name)`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:2448` `rows = [r for r in records if len(r) == 3 and r[0] == relative]`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:2457` `found[name] = (version, set(extras))`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:2483` `result["skill"] = "installed (metadata only)"`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:2508` `facts_path = base / "installs" / key / "facts.json"`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:2512` `if data.get("schema_version") != 1 or data.get("package") != "mnemosyne_hermes" or not all(isinstance(data.get(k), str) and Path(data[k]).is_absolute() for k in ("python", "site_packages")):`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:2518` `if binding and destination != candidate and destination.name == "mnemosyne_hermes":`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:2543` `if type(locked.get("version")) is not int or locked["version"] != 1 or not isinstance(locked.get("package"), list):`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:2546` `if not all(isinstance(p, dict) and isinstance(p.get("name"), str) and isinstance(p.get("version"), str) for p in packages):`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:2560` `requirements = recorded["dependencies"]`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:2561` `if not isinstance(requirements, list) or not all(isinstance(r, str) for r in requirements) or len(requirements) != len(set(requirements)):`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:2562` `raise ValueError("invalid/duplicate recorded dependencies")`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:2563` `result["requirements"] = repr(requirements) + "; Hermes extras=" + repr(fact["extras"])`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:2581` `installed = pair(site_for(environment))`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:2583` `for requirement in requirements:`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:2584` `m = re.fullmatch(r"(mnemosyne-memory|mnemosyne-hermes)(?:\[([a-z0-9_,.-]+)\])?==(\d+(?:\.\d+)*(?:(?:a|b|rc)\d+)?)", requirement)`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:2586` `raise ValueError("unsupported/ambiguous recorded exact requirements")`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:2589` `raise ValueError("duplicate requirement extras")`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:2590` `if set(expected) != set(installed):`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:2592` `matches = all(installed[n][0] == v and extras <= installed[n][1] for n, (v, extras) in expected.items())`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:2593` `for name, (version, _) in expected.items():`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:2594` `rows = [p for p in packages if p["name"].lower().replace("_", "-") == name]`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:2595` `if len(rows) != 1 or rows[0]["version"] != version:`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:2599` `elif declared.get("python_dependencies") != requirements or declared.get("version") != expected["mnemosyne-hermes"][0] or declared.get("name") != "hermes-mnemosyne":`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:2600` `result.update(ownership="incomplete", consistency="candidate requirements mismatch", reason="candidate differs from recorded PM inputs")`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:2602` `result.update(ownership="incomplete", consistency="selected package metadata mismatch", reason="selected versions/extras differ from recorded requirements")`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:2621` `print("Status for mnemosyne-hermes plugin (installation metadata only)")`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:2623` `for key, label in (("path", "Plugin path"), ("plugin", "State"), ("mode", "Mode"), ("skill_path", "Skill path"), ("skill", "Skill"), ("skill_reason", "Skill reason"), ("ownership", "Ownership"), ("requirements", "Recorded requirements"), ("selected", "Selected environment"), ("consistency", "Package metadata consistency"), ("stamp", "PM stamp"), ("reason", "Reason"), ("warning", "Warning")):`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:2635` `"""Read and verify an index-installed artifact without executing its code."""`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:2638` `raise ValueError(f"Ambiguous or missing installed artifact: {name}")`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:2643` `version = meta.get("Version", "")`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:2645` `r"\d+(?:\.\d+)*(?:(?:a|b|rc)\d+)?(?:\.post\d+)?(?:\.dev\d+)?", version`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:2647` `raise ValueError(f"Unsupported installed metadata: {name}")`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:2649` `raise ValueError(f"Not a wheel-installed artifact: {name}")`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:2652` `raise ValueError(f"Invalid installed package origin: {name}")`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:2661` `if path == info / "RECORD":`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:2663` `if path.suffix == ".pyc" and not recorded_hash:`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:2670` `raise ValueError(f"Modified/unverified installed artifact: {relative}")`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:2676` `raise ValueError(f"Incomplete installed RECORD: {name}")`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:2677` `return version, set(meta.get_all("Provides-Extra", [])), digest.hexdigest()`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:2683` `if value == "none":`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:2705` `requirements = []`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:2706` `for name, version, extras in [`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:2710` `requirements.append(name + ("[" + ",".join(extras) + "]" if extras else "") + "==" + version)`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:2725` `manifest = ("".join(fields.values()) + "version: " + provider + "\npython_dependencies: "`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:2726` `+ json.dumps(requirements) + "\n")`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:2739` `return names == set(payload) and all(`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:2740` `not (candidate / name).is_symlink() and (candidate / name).read_text(encoding="utf-8") == text`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:2750` `destination == candidate.resolve() and _adoption_candidate_matches(candidate, payload)`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:2752` `raise ValueError("Plugin binding is not the verified prior installed artifact")`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:2758` `"""Use the existing Hermes public PM in its own checkout/runtime, never pip."""`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:2771` `if k not in {"PYTHONPATH", "PYTHONHOME", "HERMES_INSTALL_ROOT"}}`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:2796` `if not profile.is_symlink() and link.is_symlink() and link.resolve() == prior.resolve():`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:2806` `print("PM adoption requirements: " + payload["plugin.yaml"].split("python_dependencies: ")[1].strip())`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:2808` `print("Read-only plan; no PM, pip, product imports, skills or profile changes.")`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:2848` `description="Install the Mnemosyne memory provider for Hermes Agent.",`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:2854` `parser.add_argument("--version", action="store_true", help="Show installed package versions and exit.")`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:2858` `install = subparsers.add_parser(`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:2859` `"install",`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:2860` `help="Install Mnemosyne into Hermes' memory provider plugin directory.",`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:2862` `install.add_argument("--adopt-pm", action="store_true", help="Explicitly admit a verified legacy pair to Hermes PM; --python names its PRIOR environment.")`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:2863` `install.add_argument("--core-extras", help="With --adopt-pm, explicitly preserve chosen core extras, comma-separated or none.")`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:2864` `install.add_argument("--provider-extras", help="With --adopt-pm, explicitly preserve chosen provider extras, comma-separated or none; installed metadata cannot infer selection.")`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:2865` `install.add_argument(`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:2873` `install.add_argument(`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:2878` `install.add_argument(`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:2881` `help="Skip auto-installing mnemosyne-hermes into Hermes' venv.",`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:2883` `install.add_argument(`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:2888` `"Install mode. Defaults to symlink, except on native Windows where "`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:2893` `install.add_argument(`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:2898` `"and install-root discovery. Also selects the site-packages a wrapper "`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:2899` `"install imports from."`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:2902` `install.add_argument(`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:2912` `install.add_argument(`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:2917` `"wrapper with the default symlink install."`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:2920` `install.add_argument(`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:2925` `help="Install only at the selected Hermes home; do not link opted-in child profiles.",`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:2928` `"uninstall",`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:2933` `help="Show whether Mnemosyne is installed for Hermes memory discovery.",`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:2935` `subparsers.add_parser("version", help="Show installed package versions.")`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:2944` `help="Emit the selected interpreter and version as JSON.",`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:2961` `help="Upgrade mnemosyne-hermes via pipx and re-install the plugin symlink.",`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:2971` `def run_install(`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:2982` `"""Core install logic , check deps, bootstrap Hermes venv if needed, create symlink.`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:2985` `Can be called from the CLI ''install'' subcommand or programmatically`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:2986` `(e.g., from ''upgrade.py'' after upgrading the pip package).`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:2989` `mode = default_install_mode()`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:2991` `# Validate supplied input before the symlink-mode installer preflight. A`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:2997` `# interpreter discovered when --python is omitted. Only symlink installs`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:2998` `# retain the installer-Python core preflight and bootstrap path.`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:2999` `if mode == "symlink" and not check_mnemosyne_core():`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:3001` `"  mnemosyne-memory NOT found in this Python. Install it first:\n"`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:3002` `"    pip install mnemosyne-hermes[all]",`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:3007` `# Both install modes need a Hermes interpreter. Symlink installs bootstrap`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:3008` `# it when needed; wrapper installs validate it and record its metadata. An`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:3014` `if mode == "wrapper" and hermes_python is None:`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:3022` `if mode == "symlink" and hermes_python is None:`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:3029` `"     and no Hermes install root contains one.\n\n"`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:3030` `"  Point the installer at it directly:\n"`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:3031` `"    mnemosyne-hermes install --python /path/to/hermes/venv/bin/python\n\n"`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:3032` `"  'mnemosyne-hermes install --dry-run' shows what discovery found.",`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:3036` `# no wrong-interpreter install to prevent and the run continues without`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:3037` `# dependency validation, as it did before. Failing here instead would`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:3039` `# install, turning a data-safety message into a discovery message.`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:3042` `print("     Continuing without dependency validation (--no-bootstrap).", file=sys.stderr)`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:3045` `# base install as the same runtime and skips the check that bootstraps`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:3047` `if mode == "symlink" and hermes_python and hermes_python != Path(sys.executable):`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:3051` `print(f"     mnemosyne-hermes is installed in YOUR Python ({sys.executable}),")`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:3056` `print("     ✓ Hermes venv now has mnemosyne-hermes installed.\n")`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:3058` `print("\n  Install it manually:\n"`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:3059` `f"    uv pip install --python {hermes_python} -U 'mnemosyne-hermes[all]'\n"`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:3060` `"  Then re-run: mnemosyne-hermes install")`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:3064` `"    Install manually:\n"`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:3065` `f"      uv pip install --python {hermes_python} -U 'mnemosyne-hermes[all]'\n"`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:3066` `"    Then re-run: mnemosyne-hermes install")`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:3072` `target = install_plugin(`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:3076` `python=hermes_python if mode == "wrapper" else python,`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:3084` `"     Symlink mode was requested explicitly, so the installer did not\n"`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:3086` `"     with an account granted the symbolic-link privilege, or install in\n"`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:3106` `skill_result = install_bundled_skill(`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:3110` `if mode == "wrapper":`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:3112` `print(f"Installed. Wrapper directory at {target}")`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:3118` `print(f"Installed. Symlink at {target}")`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:3128` `"""Run the mnemosyne-hermes installer CLI."""`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:3131` `if args.version or args.command == "version":`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:3132` `print(f"Mnemosyne {_distribution_version('mnemosyne-memory')}")`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:3133` `print(f"Mnemosyne Hermes {_distribution_version('mnemosyne-hermes')}")`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:3135` `command = args.command or "install"`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:3138` `if command == "install":`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:3147` `args.mode = default_install_mode()`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:3148` `if args.mode == DEFAULT_INSTALL_MODE_WINDOWS and _is_windows_platform():`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:3150` `"  Native Windows detected: installing in persistent wrapper mode.\n"`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:3160` `if args.mode == "wrapper" and hermes_python is None:`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:3178` `getattr(args, "mode", "symlink") == "symlink"`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:3184` `skill_plan = install_bundled_skill(`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:3191` `print(f"  Currently installed: {'yes' if is_installed(hermes_home_path=args.hermes_home) else 'no'}")`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:3192` `print(f"  Install mode: {getattr(args, 'mode', 'symlink')}")`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:3197` `if getattr(args, "mode", "symlink") == "wrapper":`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:3198` `# Match run_install(): wrapper metadata and validation use`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:3226` `# Only a symlink install bootstraps Hermes' environment;`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:3227` `# run_install() does not even look for an interpreter in wrapper`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:3230` `# installs specifically.`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:3231` `if hermes_python and getattr(args, "mode", "symlink") == "symlink":`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:3235` `return run_install(`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:3246` `if command == "uninstall":`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:3247` `target = uninstall_plugin(hermes_home_path=args.hermes_home)`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:3251` `if command == "status":`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:3254` `if command == "runtime-python":`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:3260` `if command == "cleanup":`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:3274` `if command == "upgrade":`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/install.py:3286` `if __name__ == "__main__":`

- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/plugin.yaml:2` `version: 0.7.7`
- `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/plugin.yaml:30` `dependencies: []`

- `sources/mnemosyne/integrations/hermes-catalog/plugin.yaml:3` `version: 0.7.7`
- `sources/mnemosyne/integrations/hermes-catalog/plugin.yaml:4` `description: "Native local memory for Hermes - SQLite with vector search, FTS5 hybrid ranking, episodic consolidation, and temporal triples. Zero cloud. Installed from the Hermes plugin catalog; the implementation is the mnemosyne-hermes package declared in pyproject.toml."`
- `sources/mnemosyne/integrations/hermes-catalog/plugin.yaml:6` `# A floor, not a pin: Hermes refuses to load this plugin when the running version fails this`
- `sources/mnemosyne/integrations/hermes-catalog/plugin.yaml:8` `# path this wrapper depends on: declared Python dependencies installed and re-applied after`
- `sources/mnemosyne/integrations/hermes-catalog/plugin.yaml:9` `# 'hermes update' (hermes-agent#113851, commit 96e8a23222) and automatic catalog install of`
- `sources/mnemosyne/integrations/hermes-catalog/plugin.yaml:12` `# does not exist at that tag, so 0.21.3 loads this plugin without the install path it needs.`
- `sources/mnemosyne/integrations/hermes-catalog/plugin.yaml:15` `requires_hermes: ">=0.21.4"`

- `sources/mnemosyne/integrations/hermes-catalog/pyproject.toml:3` `# [project].dependencies from this file and installs them into its venv.`
- `sources/mnemosyne/integrations/hermes-catalog/pyproject.toml:6` `version = "0.7.7"`
- `sources/mnemosyne/integrations/hermes-catalog/pyproject.toml:8` `requires-python = ">=3.11"`
- `sources/mnemosyne/integrations/hermes-catalog/pyproject.toml:9` `dependencies = [`
- `sources/mnemosyne/integrations/hermes-catalog/pyproject.toml:10` `"mnemosyne-hermes>=0.7.7,<0.8; sys_platform != 'android' and sys_platform != 'emscripten'",`
- `sources/mnemosyne/integrations/hermes-catalog/pyproject.toml:11` `"mnemosyne-memory[embeddings]>=4.0.0; sys_platform != 'android' and sys_platform != 'emscripten'",`

## 6. Tests

Native integration and legacy provider test families are listed separately by path. Engine forget_episodic/working tests include derived-row/ownership behavior; these are not proof of backup or host-history erasure. Engine test asserts owned row, unambiguous child rows and episodic vector are removed. (`sources/mnemosyne/tests/test_forget_episodic.py:128`).

Adapter-related declarations: **771**. Python AST test_ functions, fixtures excluded; literal TS/JS it/test, Go Test*, Rust test_* declarations. No parametrization expansion, collection, execution, or coverage measurement. Other rows are engine test candidates selected by mutation names; names alone do not verify forget coverage.

| file | declarations | adapter-related | forget-related declarations |
|---|---:|---|---|
| `sources/mnemosyne/integrations/hermes/tests/test_calltime_bindings.py:1` | 20 | yes | `test_invalidate_routes_by_id_namespace_and_explicit_bank` `sources/mnemosyne/integrations/hermes/tests/test_calltime_bindings.py:493`; `test_invalidate_self_replacement_rejected_before_lookup` `sources/mnemosyne/integrations/hermes/tests/test_calltime_bindings.py:521`; `test_invalidate_reports_missing_replacement` `sources/mnemosyne/integrations/hermes/tests/test_calltime_bindings.py:531`; `test_invalidate_inactive_target_stays_memory_not_found` `sources/mnemosyne/integrations/hermes/tests/test_calltime_bindings.py:554`; `test_invalidate_active_helper_uses_sql_when_beam_has_connection` `sources/mnemosyne/integrations/hermes/tests/test_calltime_bindings.py:570`; `test_invalidate_failed_activity_probe_with_real_get_shape_falls_closed` `sources/mnemosyne/integrations/hermes/tests/test_calltime_bindings.py:608`; `test_invalidate_parity_on_west_of_utc_host` `sources/mnemosyne/integrations/hermes/tests/test_calltime_bindings.py:712` |
| `sources/mnemosyne/integrations/hermes/tests/test_canonical_contract_edges.py:1` | 9 | yes | none |
| `sources/mnemosyne/integrations/hermes/tests/test_canonical_prefetch_parity.py:1` | 10 | yes | none |
| `sources/mnemosyne/integrations/hermes/tests/test_canonical_single_topic_unit.py:1` | 13 | yes | none |
| `sources/mnemosyne/integrations/hermes/tests/test_canonical_tools.py:1` | 6 | yes | `test_invalidate_reports_not_found_when_no_row_matched` `sources/mnemosyne/integrations/hermes/tests/test_canonical_tools.py:180` |
| `sources/mnemosyne/integrations/hermes/tests/test_cli_bank.py:1` | 27 | yes | none |
| `sources/mnemosyne/integrations/hermes/tests/test_diagnose_connection_provenance.py:1` | 4 | yes | none |
| `sources/mnemosyne/integrations/hermes/tests/test_find_hermes_python.py:1` | 46 | yes | none |
| `sources/mnemosyne/integrations/hermes/tests/test_find_hermes_python_staged_runtime.py:1` | 17 | yes | none |
| `sources/mnemosyne/integrations/hermes/tests/test_hermes_machine_notice_filter.py:1` | 23 | yes | none |
| `sources/mnemosyne/integrations/hermes/tests/test_init_ordering_1177.py:1` | 1 | yes | none |
| `sources/mnemosyne/integrations/hermes/tests/test_init_retry.py:1` | 9 | yes | none |
| `sources/mnemosyne/integrations/hermes/tests/test_install_cli_entrypoint.py:1` | 5 | yes | none |
| `sources/mnemosyne/integrations/hermes/tests/test_install_hermes.py:1` | 67 | yes | none |
| `sources/mnemosyne/integrations/hermes/tests/test_install_status.py:1` | 36 | yes | none |
| `sources/mnemosyne/integrations/hermes/tests/test_path_contract.py:1` | 3 | yes | none |
| `sources/mnemosyne/integrations/hermes/tests/test_persona_plugin_registration.py:1` | 3 | yes | none |
| `sources/mnemosyne/integrations/hermes/tests/test_platform_boundary.py:1` | 4 | yes | none |
| `sources/mnemosyne/integrations/hermes/tests/test_plugin_tool_initialization.py:1` | 5 | yes | none |
| `sources/mnemosyne/integrations/hermes/tests/test_pm_generation_wrapper_guard.py:1` | 5 | yes | none |
| `sources/mnemosyne/integrations/hermes/tests/test_pm_ownership_status.py:1` | 20 | yes | none |
| `sources/mnemosyne/integrations/hermes/tests/test_prefetch_hardening.py:1` | 38 | yes | none |
| `sources/mnemosyne/integrations/hermes/tests/test_recall_diagnostics_tool.py:1` | 3 | yes | none |
| `sources/mnemosyne/integrations/hermes/tests/test_runtime_python_cli.py:1` | 6 | yes | none |
| `sources/mnemosyne/integrations/hermes/tests/test_same_home_profiles.py:1` | 4 | yes | none |
| `sources/mnemosyne/integrations/hermes/tests/test_setup_status_contract.py:1` | 11 | yes | none |
| `sources/mnemosyne/integrations/hermes/tests/test_supported_path_a_b_a.py:1` | 4 | yes | none |
| `sources/mnemosyne/integrations/hermes/tests/test_sync_roles_config.py:1` | 10 | yes | none |
| `sources/mnemosyne/integrations/hermes/tests/test_system_prompt.py:1` | 5 | yes | none |
| `sources/mnemosyne/integrations/hermes/tests/test_task_progress_tool.py:1` | 5 | yes | none |
| `sources/mnemosyne/integrations/hermes/tests/test_thread_connection_release.py:1` | 1 | yes | none |
| `sources/mnemosyne/integrations/hermes/tests/test_upgrade_wrapper.py:1` | 5 | yes | none |
| `sources/mnemosyne/integrations/hermes/tests/test_upstream_hermes_contract.py:1` | 1 | yes | none |
| `sources/mnemosyne/integrations/hermes/tests/test_windows_default_install_mode.py:1` | 8 | yes | none |
| `sources/mnemosyne/integrations/hermes/tests/test_windows_symlink_privilege.py:1` | 3 | yes | none |
| `sources/mnemosyne/integrations/hermes/tests/test_wrapper_bootstrap.py:1` | 6 | yes | none |
| `sources/mnemosyne/integrations/hermes/tests/test_wrapper_import_timeout.py:1` | 9 | yes | none |
| `sources/mnemosyne/tests/test_936_scope_parity_audit.py:1` | 23 | no | `test_redirected_forget_targets_the_staging_session` `sources/mnemosyne/tests/test_936_scope_parity_audit.py:357`; `test_redirected_invalidate_targets_the_staging_session` `sources/mnemosyne/tests/test_936_scope_parity_audit.py:405`; `test_replayed_forget_emits_audit_event` `sources/mnemosyne/tests/test_936_scope_parity_audit.py:772`; `test_replayed_invalidate_emits_audit_event_with_replacement_metadata` `sources/mnemosyne/tests/test_936_scope_parity_audit.py:802`; `test_replayed_forget_and_invalidate_audit_the_staging_session` `sources/mnemosyne/tests/test_936_scope_parity_audit.py:837` |
| `sources/mnemosyne/tests/test_annotations.py:1` | 17 | no | `test_no_auto_invalidation_columns` `sources/mnemosyne/tests/test_annotations.py:88`; `test_tripstore_silently_invalidates_sibling_annotations` `sources/mnemosyne/tests/test_annotations.py:379`; `test_tripstore_query_with_as_of_drops_silently_invalidated` `sources/mnemosyne/tests/test_annotations.py:401` |
| `sources/mnemosyne/tests/test_apply_pending_replay_926.py:1` | 20 | no | `test_apply_pending_forget_is_content_less` `sources/mnemosyne/tests/test_apply_pending_replay_926.py:286`; `test_apply_pending_invalidate_is_content_less` `sources/mnemosyne/tests/test_apply_pending_replay_926.py:324`; `test_apply_pending_invalidate_replacement_chains` `sources/mnemosyne/tests/test_apply_pending_replay_926.py:366`; `test_apply_pending_record_deleted_only_after_success` `sources/mnemosyne/tests/test_apply_pending_replay_926.py:423` |
| `sources/mnemosyne/tests/test_batch_transaction_owner_726.py:1` | 9 | no | `test_forget_cascade_failure_after_guard_preserves_caller_transaction` `sources/mnemosyne/tests/test_batch_transaction_owner_726.py:101` |
| `sources/mnemosyne/tests/test_beam.py:1` | 135 | no | `test_remember_update_and_forget_maintain_vec_working` `sources/mnemosyne/tests/test_beam.py:389`; `test_invalidate_stamps_aware_utc_valid_until` `sources/mnemosyne/tests/test_beam.py:1120`; `test_invalidated_memory_excluded_across_utc_surfaces` `sources/mnemosyne/tests/test_beam.py:1140`; `test_invalidate_rejects_offset_bearing_expiry` `sources/mnemosyne/tests/test_beam.py:1261`; `test_invalidate_rejects_equal_instant` `sources/mnemosyne/tests/test_beam.py:1295`; `test_invalidate_accepts_future_offset_bearing_target` `sources/mnemosyne/tests/test_beam.py:1331`; `test_forget_removes_both_layers` `sources/mnemosyne/tests/test_beam.py:1976` |
| `sources/mnemosyne/tests/test_beam_event_timestamps.py:1` | 90 | no | `test_trim_never_deletes_pinned_rows` `sources/mnemosyne/tests/test_beam_event_timestamps.py:714`; `test_update_working_invalidates_after_commit` `sources/mnemosyne/tests/test_beam_event_timestamps.py:1417`; `test_update_working_noop_does_not_invalidate` `sources/mnemosyne/tests/test_beam_event_timestamps.py:1429`; `test_consolidate_caller_txn_defers_invalidation` `sources/mnemosyne/tests/test_beam_event_timestamps.py:1441` |
| `sources/mnemosyne/tests/test_canonical.py:1` | 30 | no | `test_forget_retires_current_keeps_history` `sources/mnemosyne/tests/test_canonical.py:121`; `test_forget_empty_slot_returns_false` `sources/mnemosyne/tests/test_canonical.py:128`; `test_remember_after_forget_starts_fresh_current` `sources/mnemosyne/tests/test_canonical.py:131` |
| `sources/mnemosyne/tests/test_canonical_valid_until_utc.py:1` | 7 | no | `test_forget_stamps_valid_until_in_created_at_clock` `sources/mnemosyne/tests/test_canonical_valid_until_utc.py:68`; `test_history_order_survives_forget_and_reremember` `sources/mnemosyne/tests/test_canonical_valid_until_utc.py:111` |
| `sources/mnemosyne/tests/test_cli_errors.py:1` | 11 | no | `test_bank_cli_list_create_delete_uses_configured_data_dir` `sources/mnemosyne/tests/test_cli_errors.py:250` |
| `sources/mnemosyne/tests/test_cli_operation_failures.py:1` | 1 | no | `test_update_delete_missing_memory_report_operation_failure` `sources/mnemosyne/tests/test_cli_operation_failures.py:21` |
| `sources/mnemosyne/tests/test_conflict_provenance_atomicity.py:1` | 6 | no | `test_invalidate_cache_deferral_preserves_transaction_ownership` `sources/mnemosyne/tests/test_conflict_provenance_atomicity.py:166` |
| `sources/mnemosyne/tests/test_cross_session_conflict_resolution.py:1` | 23 | no | `test_invalidate_respects_bank` `sources/mnemosyne/tests/test_cross_session_conflict_resolution.py:448`; `test_invalidate_preserves_existing_successor` `sources/mnemosyne/tests/test_cross_session_conflict_resolution.py:519` |
| `sources/mnemosyne/tests/test_degrade_vector.py:1` | 8 | no | `test_tier_2_to_tier_3_invalidates_when_provider_unavailable` `sources/mnemosyne/tests/test_degrade_vector.py:510` |
| `sources/mnemosyne/tests/test_deletion_cascade.py:1` | 7 | no | `test_forget_working_deletes_derived_rows` `sources/mnemosyne/tests/test_deletion_cascade.py:94`; `test_forget_working_ignores_the_legacy_mirror` `sources/mnemosyne/tests/test_deletion_cascade.py:107`; `test_forget_working_keeps_children_for_a_same_id_episodic_parent` `sources/mnemosyne/tests/test_deletion_cascade.py:138`; `test_forget_episodic_deletes_derived_rows` `sources/mnemosyne/tests/test_deletion_cascade.py:174`; `test_trim_deletes_derived_rows_for_trimmed_ids` `sources/mnemosyne/tests/test_deletion_cascade.py:189`; `test_delayed_entity_extraction_after_forget_writes_nothing` `sources/mnemosyne/tests/test_deletion_cascade.py:212`; `test_delayed_graph_enrichment_after_forget_writes_nothing` `sources/mnemosyne/tests/test_deletion_cascade.py:227` |
| `sources/mnemosyne/tests/test_direct_wrapper_atomicity_726.py:1` | 14 | no | `test_direct_invalidate_is_beam_only_and_leaves_legacy_row_unchanged` `sources/mnemosyne/tests/test_direct_wrapper_atomicity_726.py:72`; `test_direct_wrapper_caller_decides_update_and_forget_rows` `sources/mnemosyne/tests/test_direct_wrapper_atomicity_726.py:196`; `test_direct_wrapper_cross_session_forget_removes_global_dual_rows` `sources/mnemosyne/tests/test_direct_wrapper_atomicity_726.py:227`; `test_direct_wrapper_cross_session_forget_preserves_private_rows` `sources/mnemosyne/tests/test_direct_wrapper_atomicity_726.py:238`; `test_owner_forget_removes_legacy_only_row_but_foreign_caller_cannot` `sources/mnemosyne/tests/test_direct_wrapper_atomicity_726.py:330`; `test_owner_legacy_only_forget_keeps_caller_transaction_rollbackable` `sources/mnemosyne/tests/test_direct_wrapper_atomicity_726.py:344` |
| `sources/mnemosyne/tests/test_e6a_followup_gaps.py:1` | 7 | no | `test_forget_deletes_annotations_for_memory_id` `sources/mnemosyne/tests/test_e6a_followup_gaps.py:47`; `test_forget_doesnt_touch_other_memories_annotations` `sources/mnemosyne/tests/test_e6a_followup_gaps.py:66`; `test_beam_forget_working_directly_cascades` `sources/mnemosyne/tests/test_e6a_followup_gaps.py:82`; `test_forget_after_export_leaves_no_leaked_annotations` `sources/mnemosyne/tests/test_e6a_followup_gaps.py:94`; `test_wrong_session_forget_does_not_touch_annotations` `sources/mnemosyne/tests/test_e6a_followup_gaps.py:131`; `test_correct_session_forget_still_works` `sources/mnemosyne/tests/test_e6a_followup_gaps.py:148`; `test_failed_cascade_rolls_back_working_memory_delete` `sources/mnemosyne/tests/test_e6a_followup_gaps.py:178` |
| `sources/mnemosyne/tests/test_embedding_api_retry.py:1` | 42 | no | `test_update_working_invalidates_stale_vector_when_embedding_raises` `sources/mnemosyne/tests/test_embedding_api_retry.py:824`; `test_update_working_invalidates_stale_vector_when_embed_returns_none` `sources/mnemosyne/tests/test_embedding_api_retry.py:842`; `test_update_working_invalidates_stale_vector_when_embeddings_unavailable` `sources/mnemosyne/tests/test_embedding_api_retry.py:855` |
| `sources/mnemosyne/tests/test_enhanced_recall_cache.py:1` | 42 | no | `test_cache_version_bump_invalidates_prior_candidate_and_cap_entries` `sources/mnemosyne/tests/test_enhanced_recall_cache.py:134`; `test_successful_invalidate_clears_persisted_enhanced_recall_cache` `sources/mnemosyne/tests/test_enhanced_recall_cache.py:174`; `test_successful_forget_working_clears_persisted_enhanced_recall_cache` `sources/mnemosyne/tests/test_enhanced_recall_cache.py:214`; `test_failed_cross_session_forget_keeps_cache_and_memory` `sources/mnemosyne/tests/test_enhanced_recall_cache.py:267`; `test_invalidate_without_local_query_cache_clears_persisted_enhanced_recall_cache` `sources/mnemosyne/tests/test_enhanced_recall_cache.py:304`; `test_remember_invalidates_persisted_cache_when_post_commit_trim_raises` `sources/mnemosyne/tests/test_enhanced_recall_cache.py:440`; `test_dedup_invalidates_before_enrichment` `sources/mnemosyne/tests/test_enhanced_recall_cache.py:488`; `test_remember_finally_invalidates_cache_refilled_during_enrichment` `sources/mnemosyne/tests/test_enhanced_recall_cache.py:513`; `test_new_remember_finally_invalidates_cache_refilled_during_enrichment` `sources/mnemosyne/tests/test_enhanced_recall_cache.py:547`; `test_new_remember_finally_invalidates_cache_refilled_during_enrichment_exception` `sources/mnemosyne/tests/test_enhanced_recall_cache.py:580`; `test_dedup_remember_finally_invalidates_cache_refilled_during_enrichment_exception` `sources/mnemosyne/tests/test_enhanced_recall_cache.py:643`; `test_new_remember_write_survives_post_commit_cache_invalidation_failure` `sources/mnemosyne/tests/test_enhanced_recall_cache.py:704`; `test_dedup_remember_write_survives_post_commit_cache_invalidation_failure` `sources/mnemosyne/tests/test_enhanced_recall_cache.py:735`; `test_post_commit_mutations_survive_query_cache_invalidation_failure` `sources/mnemosyne/tests/test_enhanced_recall_cache.py:768`; `test_mutations_keep_cache_invalidation_errors_before_caller_commit` `sources/mnemosyne/tests/test_enhanced_recall_cache.py:807`; `test_invalidate_without_replacement_keeps_cache_failure_in_caller_transaction` `sources/mnemosyne/tests/test_enhanced_recall_cache.py:849`; `test_invalidate_keeps_cache_invalidation_error_during_deferred_commit` `sources/mnemosyne/tests/test_enhanced_recall_cache.py:891`; `test_successful_invalidate_episodic_memory_invalidates_enhanced_recall_cache` `sources/mnemosyne/tests/test_enhanced_recall_cache.py:919`; `test_enhanced_cache_key_version_bump_invalidates_literal_flag_ranking_change` `sources/mnemosyne/tests/test_enhanced_recall_cache.py:1096`; `test_consolidate_to_episodic_invalidates_warmed_v3_cache` `sources/mnemosyne/tests/test_enhanced_recall_cache.py:1536`; `test_consolidate_enrichment_failure_still_invalidates_warmed_v3_cache` `sources/mnemosyne/tests/test_enhanced_recall_cache.py:1584`; `test_reclaim_orphans_invalidates_warmed_v3_cache` `sources/mnemosyne/tests/test_enhanced_recall_cache.py:1637`; `test_degrade_episodic_invalidates_warmed_v3_cache` `sources/mnemosyne/tests/test_enhanced_recall_cache.py:1687`; `test_sleep_invalidates_warmed_v3_cache` `sources/mnemosyne/tests/test_enhanced_recall_cache.py:1745` |
| `sources/mnemosyne/tests/test_entities.py:1` | 34 | no | `test_single_deletion` `sources/mnemosyne/tests/test_entities.py:37` |
| `sources/mnemosyne/tests/test_forget_episodic.py:1` | 20 | no | `test_forget_working_ignores_a_legacy_mirror_with_the_same_id` `sources/mnemosyne/tests/test_forget_episodic.py:93`; `test_forget_episodic_ignores_a_legacy_mirror_with_the_same_id` `sources/mnemosyne/tests/test_forget_episodic.py:113`; `test_forget_deletes_owned_episodic_row_and_unambiguous_children` `sources/mnemosyne/tests/test_forget_episodic.py:128`; `test_forget_episodic_refuses_foreign_private_row_and_keeps_children` `sources/mnemosyne/tests/test_forget_episodic.py:148`; `test_forget_episodic_allows_global_row_cross_session` `sources/mnemosyne/tests/test_forget_episodic.py:161`; `test_forget_episodic_preserves_children_when_working_parent_shares_id` `sources/mnemosyne/tests/test_forget_episodic.py:170`; `test_forget_working_preserves_children_when_episodic_parent_shares_id` `sources/mnemosyne/tests/test_forget_episodic.py:192`; `test_forget_working_preserves_foreign_episodic_parent_and_children` `sources/mnemosyne/tests/test_forget_episodic.py:209`; `test_forget_shared_children_after_final_same_id_parent_deleted` `sources/mnemosyne/tests/test_forget_episodic.py:236`; `test_forget_locks_before_a_competing_parent_can_arrive` `sources/mnemosyne/tests/test_forget_episodic.py:254`; `test_mnemosyne_forget_locks_before_episodic_fallback_reads` `sources/mnemosyne/tests/test_forget_episodic.py:313`; `test_mnemosyne_forget_falls_back_to_episodic_and_emits_only_on_success` `sources/mnemosyne/tests/test_forget_episodic.py:359`; `test_mnemosyne_forget_removes_episodic_after_legacy_cleanup` `sources/mnemosyne/tests/test_forget_episodic.py:381`; `test_batch_forget_falls_back_to_episodic` `sources/mnemosyne/tests/test_forget_episodic.py:397`; `test_batch_forget_locks_before_episodic_fallback_reads` `sources/mnemosyne/tests/test_forget_episodic.py:414`; `test_batch_forget_preserves_legacy_beam_without_episodic_method` `sources/mnemosyne/tests/test_forget_episodic.py:464`; `test_forget_episodic_failure_rolls_back_full_cascade` `sources/mnemosyne/tests/test_forget_episodic.py:492`; `test_forget_episodic_defers_event_to_outer_commit` `sources/mnemosyne/tests/test_forget_episodic.py:511`; `test_forget_episodic_rollback_discards_event_and_restores_row` `sources/mnemosyne/tests/test_forget_episodic.py:528`; `test_forget_episodic_invalidates_cache_inside_outer_transaction` `sources/mnemosyne/tests/test_forget_episodic.py:555` |
| `sources/mnemosyne/tests/test_forget_gist_cascade.py:1` | 5 | no | `test_direct_forget_deletes_only_authorized_target_gists` `sources/mnemosyne/tests/test_forget_gist_cascade.py:29`; `test_batch_forget_uses_the_same_gist_cascade` `sources/mnemosyne/tests/test_forget_gist_cascade.py:47`; `test_forget_succeeds_when_optional_gists_table_is_absent` `sources/mnemosyne/tests/test_forget_gist_cascade.py:64`; `test_foreign_session_forget_keeps_private_memory_and_gist` `sources/mnemosyne/tests/test_forget_gist_cascade.py:76`; `test_gist_delete_failure_rolls_back_the_entire_forget_cascade` `sources/mnemosyne/tests/test_forget_gist_cascade.py:91` |
| `sources/mnemosyne/tests/test_hermes_catalog_plugin.py:1` | 8 | yes | none |
| `sources/mnemosyne/tests/test_hermes_config.py:1` | 8 | yes | `test_atomic_same_size_replacement_invalidates_cached_config` `sources/mnemosyne/tests/test_hermes_config.py:46`; `test_cached_config_fails_closed_after_malformed_or_deleted_file` `sources/mnemosyne/tests/test_hermes_config.py:85` |
| `sources/mnemosyne/tests/test_hermes_core_release_contract.py:1` | 4 | yes | none |
| `sources/mnemosyne/tests/test_hermes_doctor_bank_routing.py:1` | 15 | yes | none |
| `sources/mnemosyne/tests/test_hermes_hindsight_cli.py:1` | 7 | yes | none |
| `sources/mnemosyne/tests/test_hermes_legacy_shadowing_cwd.py:1` | 2 | yes | none |
| `sources/mnemosyne/tests/test_hermes_llm_adapter.py:1` | 14 | yes | none |
| `sources/mnemosyne/tests/test_hermes_machine_notice_filter.py:1` | 23 | yes | none |
| `sources/mnemosyne/tests/test_hermes_memory_provider.py:1` | 65 | yes | none |
| `sources/mnemosyne/tests/test_hermes_memory_provider_audit.py:1` | 15 | yes | `test_forget_creates_audit_event` `sources/mnemosyne/tests/test_hermes_memory_provider_audit.py:70`; `test_forget_not_found_no_audit` `sources/mnemosyne/tests/test_hermes_memory_provider_audit.py:82`; `test_invalidate_creates_audit_event` `sources/mnemosyne/tests/test_hermes_memory_provider_audit.py:88`; `test_invalidate_not_found_returns_status` `sources/mnemosyne/tests/test_hermes_memory_provider_audit.py:100`; `test_invalidate_success_returns_status` `sources/mnemosyne/tests/test_hermes_memory_provider_audit.py:109`; `test_invalidate_with_replacement_persists_link_and_audit_metadata` `sources/mnemosyne/tests/test_hermes_memory_provider_audit.py:132`; `test_shared_forget_creates_audit_event` `sources/mnemosyne/tests/test_hermes_memory_provider_audit.py:206` |
| `sources/mnemosyne/tests/test_hermes_memory_provider_diagnose_active_db.py:1` | 5 | yes | none |
| `sources/mnemosyne/tests/test_hermes_memory_provider_shared_crud.py:1` | 17 | yes | `test_shared_forget_deletes_then_reports_not_found` `sources/mnemosyne/tests/test_hermes_memory_provider_shared_crud.py:98`; `test_invalidate_missing_replacement_reports_its_own_status` `sources/mnemosyne/tests/test_hermes_memory_provider_shared_crud.py:143`; `test_invalidate_missing_target_stays_memory_not_found` `sources/mnemosyne/tests/test_hermes_memory_provider_shared_crud.py:169`; `test_invalidate_inactive_target_stays_memory_not_found` `sources/mnemosyne/tests/test_hermes_memory_provider_shared_crud.py:186`; `test_invalidate_expired_target_stays_memory_not_found` `sources/mnemosyne/tests/test_hermes_memory_provider_shared_crud.py:211`; `test_invalidate_failed_activity_probe_with_real_get_shape_falls_closed` `sources/mnemosyne/tests/test_hermes_memory_provider_shared_crud.py:242`; `test_invalidate_rejects_self_replacement` `sources/mnemosyne/tests/test_hermes_memory_provider_shared_crud.py:300`; `test_invalidate_replacement_must_be_visible_in_the_routed_bank` `sources/mnemosyne/tests/test_hermes_memory_provider_shared_crud.py:319`; `test_invalidate_explicit_surface_selector_beats_prefix_inference` `sources/mnemosyne/tests/test_hermes_memory_provider_shared_crud.py:351`; `test_invalidate_parity_on_west_of_utc_host` `sources/mnemosyne/tests/test_hermes_memory_provider_shared_crud.py:442` |
| `sources/mnemosyne/tests/test_hermes_memory_provider_shared_multi_agent.py:1` | 2 | yes | none |
| `sources/mnemosyne/tests/test_hermes_memory_provider_system_prompt.py:1` | 2 | yes | none |
| `sources/mnemosyne/tests/test_hermes_memory_provider_thread_isolation.py:1` | 5 | yes | none |
| `sources/mnemosyne/tests/test_hermes_memory_provider_unified_recall.py:1` | 10 | yes | none |
| `sources/mnemosyne/tests/test_hermes_memory_provider_validation.py:1` | 25 | yes | `test_validate_invalidate_sets_valid_until` `sources/mnemosyne/tests/test_hermes_memory_provider_validation.py:187`; `test_validate_delete_removes_row` `sources/mnemosyne/tests/test_hermes_memory_provider_validation.py:207`; `test_validate_delete_cascades_support_rows` `sources/mnemosyne/tests/test_hermes_memory_provider_validation.py:223`; `test_validate_delete_rolls_back_when_validation_log_fails` `sources/mnemosyne/tests/test_hermes_memory_provider_validation.py:283`; `test_validate_delete_logs_the_failure_it_swallows` `sources/mnemosyne/tests/test_hermes_memory_provider_validation.py:327`; `test_validate_delete_removes_vec_working_row` `sources/mnemosyne/tests/test_hermes_memory_provider_validation.py:357`; `test_validate_delete_refuses_a_foreign_session_private_memory` `sources/mnemosyne/tests/test_hermes_memory_provider_validation.py:392`; `test_validate_delete_handles_missing_gists_table` `sources/mnemosyne/tests/test_hermes_memory_provider_validation.py:447` |
| `sources/mnemosyne/tests/test_hermes_provider_parity.py:1` | 73 | yes | `test_forget_falls_back_to_episodic_in_both_providers` `sources/mnemosyne/tests/test_hermes_provider_parity.py:2461`; `test_forget_returns_not_found_when_both_tiers_miss` `sources/mnemosyne/tests/test_hermes_provider_parity.py:2478`; `test_forget_keeps_older_core_compatibility_in_both_providers` `sources/mnemosyne/tests/test_hermes_provider_parity.py:2495` |
| `sources/mnemosyne/tests/test_hermes_provider_runtime_compat.py:1` | 4 | yes | none |
| `sources/mnemosyne/tests/test_hermes_resolver_contract.py:1` | 10 | yes | none |
| `sources/mnemosyne/tests/test_hermes_sync_adapter.py:1` | 5 | yes | none |
| `sources/mnemosyne/tests/test_heuristic_conflict_safety.py:1` | 6 | no | `test_heuristic_pairs_never_invalidate_without_llm` `sources/mnemosyne/tests/test_heuristic_conflict_safety.py:96`; `test_dry_run_never_invalidates` `sources/mnemosyne/tests/test_heuristic_conflict_safety.py:129`; `test_invalidate_failure_counts_as_detected` `sources/mnemosyne/tests/test_heuristic_conflict_safety.py:232` |
| `sources/mnemosyne/tests/test_hygiene.py:1` | 89 | no | `test_high_score_suggests_delete` `sources/mnemosyne/tests/test_hygiene.py:290`; `test_delete_with_confirm` `sources/mnemosyne/tests/test_hygiene.py:1135` |
| `sources/mnemosyne/tests/test_hygiene_delete_cascade.py:1` | 7 | no | `test_episodic_delete_cascades_side_rows` `sources/mnemosyne/tests/test_hygiene_delete_cascade.py:108`; `test_episodic_delete_cascades_without_vec` `sources/mnemosyne/tests/test_hygiene_delete_cascade.py:127`; `test_working_delete_cascades_vec_working` `sources/mnemosyne/tests/test_hygiene_delete_cascade.py:142`; `test_legacy_memories_delete_has_no_vec_mirror` `sources/mnemosyne/tests/test_hygiene_delete_cascade.py:161`; `test_missing_vec_table_does_not_abort_delete` `sources/mnemosyne/tests/test_hygiene_delete_cascade.py:185`; `test_required_cascade_failure_rolls_back_base_delete` `sources/mnemosyne/tests/test_hygiene_delete_cascade.py:242` |
| `sources/mnemosyne/tests/test_issue_523_query_cache_lifecycle.py:1` | 14 | no | `test_peer_replacement_at_same_timestamp_is_not_deleted` `sources/mnemosyne/tests/test_issue_523_query_cache_lifecycle.py:120`; `test_ttl_expiry_in_process_deletes_persisted_rows` `sources/mnemosyne/tests/test_issue_523_query_cache_lifecycle.py:203` |
| `sources/mnemosyne/tests/test_mcp_server.py:1` | 63 | no | `test_invalidate_schema_documents_scope_safe_failure` `sources/mnemosyne/tests/test_mcp_server.py:114`; `test_forget_canonical_retires_only_the_current_owner_slot` `sources/mnemosyne/tests/test_mcp_server.py:202`; `test_forget_canonical_requires_category_and_name` `sources/mnemosyne/tests/test_mcp_server.py:246`; `test_forget_canonical_uses_mcp_bank_default` `sources/mnemosyne/tests/test_mcp_server.py:255`; `test_handle_invalidate_preserves_session_scope_and_reports_not_found` `sources/mnemosyne/tests/test_mcp_server.py:303`; `test_handle_invalidate_validates_replacement_in_current_scope` `sources/mnemosyne/tests/test_mcp_server.py:350`; `test_handle_batch_wrapper_update_forget_invalidate_and_scope` `sources/mnemosyne/tests/test_mcp_server.py:484` |
| `sources/mnemosyne/tests/test_memory_banks.py:1` | 37 | no | `test_delete_bank_removes_directory` `sources/mnemosyne/tests/test_memory_banks.py:126`; `test_delete_nonexistent_bank_returns_false` `sources/mnemosyne/tests/test_memory_banks.py:135`; `test_delete_default_bank_raises` `sources/mnemosyne/tests/test_memory_banks.py:142`; `test_delete_default_with_force_succeeds` `sources/mnemosyne/tests/test_memory_banks.py:149` |
| `sources/mnemosyne/tests/test_memory_lifecycle.py:1` | 5 | no | `test_invalidate_emits_wrapper_event_only_after_success` `sources/mnemosyne/tests/test_memory_lifecycle.py:14`; `test_invalidate_accepts_authorized_episodic_replacement` `sources/mnemosyne/tests/test_memory_lifecycle.py:30`; `test_beam_invalidate_rejects_self_replacement_without_mutation` `sources/mnemosyne/tests/test_memory_lifecycle.py:45`; `test_invalidate_replacement_starts_write_transaction_before_lookup` `sources/mnemosyne/tests/test_memory_lifecycle.py:56` |
| `sources/mnemosyne/tests/test_mnemosyne_batch_tool.py:1` | 14 | no | `test_batch_update_and_invalidate` `sources/mnemosyne/tests/test_mnemosyne_batch_tool.py:59`; `test_batch_update_then_invalidate_with_replacement_preserves_outer_transaction` `sources/mnemosyne/tests/test_mnemosyne_batch_tool.py:83` |
| `sources/mnemosyne/tests/test_orphan_child_cleanup.py:1` | 8 | no | `test_mcp_validate_delete_cascades_only_target_children` `sources/mnemosyne/tests/test_orphan_child_cleanup.py:57`; `test_mcp_validate_delete_rolls_back_on_child_failure` `sources/mnemosyne/tests/test_orphan_child_cleanup.py:83`; `test_mcp_validate_delete_removes_vec_working_row` `sources/mnemosyne/tests/test_orphan_child_cleanup.py:112`; `test_mcp_validate_delete_handles_missing_vec_working` `sources/mnemosyne/tests/test_orphan_child_cleanup.py:164`; `test_mcp_validate_delete_handles_missing_gists_table` `sources/mnemosyne/tests/test_orphan_child_cleanup.py:181`; `test_mcp_validate_delete_refuses_a_foreign_session_private_memory` `sources/mnemosyne/tests/test_orphan_child_cleanup.py:196`; `test_mcp_validate_delete_allows_a_global_memory_from_another_session` `sources/mnemosyne/tests/test_orphan_child_cleanup.py:224` |
| `sources/mnemosyne/tests/test_patterns.py:1` | 31 | no | `test_get_all_memories_excludes_invalidated` `sources/mnemosyne/tests/test_patterns.py:300` |
| `sources/mnemosyne/tests/test_persona_adapter.py:1` | 19 | no | `test_demote_rolls_back_tombstone_when_delete_fails` `sources/mnemosyne/tests/test_persona_adapter.py:301`; `test_legacy_demote_rolls_back_tombstone_when_delete_fails` `sources/mnemosyne/tests/test_persona_adapter.py:327` |
| `sources/mnemosyne/tests/test_plugins.py:1` | 70 | no | `test_on_invalidate_logs` `sources/mnemosyne/tests/test_plugins.py:402`; `test_notify_invalidate` `sources/mnemosyne/tests/test_plugins.py:602` |
| `sources/mnemosyne/tests/test_provider_all_15_tools.py:1` | 34 | no | `test_forget_existing` `sources/mnemosyne/tests/test_provider_all_15_tools.py:211`; `test_forget_not_found` `sources/mnemosyne/tests/test_provider_all_15_tools.py:220`; `test_forget_missing_id` `sources/mnemosyne/tests/test_provider_all_15_tools.py:225`; `test_dispatch_forget` `sources/mnemosyne/tests/test_provider_all_15_tools.py:230` |
| `sources/mnemosyne/tests/test_query_cache_synonyms.py:1` | 10 | no | `test_cache_invalidation` `sources/mnemosyne/tests/test_query_cache_synonyms.py:74` |
| `sources/mnemosyne/tests/test_repair.py:1` | 28 | no | `test_expiry_changes_only_selected_valid_until_and_never_deletes` `sources/mnemosyne/tests/test_repair.py:663` |
| `sources/mnemosyne/tests/test_sync_correctness.py:1` | 38 | no | `test_discover_local_mutations_emits_create_update_delete` `sources/mnemosyne/tests/test_sync_correctness.py:29`; `test_plaintext_delete_without_payload_is_applied` `sources/mnemosyne/tests/test_sync_correctness.py:375`; `test_blind_relay_never_materializes_encrypted_delete` `sources/mnemosyne/tests/test_sync_correctness.py:517`; `test_upgrade_bootstrap_emits_delete_for_legacy_logged_missing_row` `sources/mnemosyne/tests/test_sync_correctness.py:836`; `test_discovery_recovers_from_delete_event_logged_before_shadow_state` `sources/mnemosyne/tests/test_sync_correctness.py:857`; `test_encrypted_e2e_propagates_create_update_delete` `sources/mnemosyne/tests/test_sync_correctness.py:1072` |
| `sources/mnemosyne/tests/test_trim_working_vectors.py:1` | 11 | no | `test_failed_trim_rolls_back_vector_deletes` `sources/mnemosyne/tests/test_trim_working_vectors.py:138` |

## 7. Corrections to B2

Old values below are copied from the triage row (TRIAGE.md), not reconstructed from memory. A refinement narrows an over-broad field; it need not reverse its original value. Counts are distinct field paths, not the number of factual errors.

- `engine_delete.e`: `["hermes_memory_provider/__init__.py:331", "hermes_memory_provider/__init__.py:3475", "hermes_memory_provider/__init__.py:4013", "hermes_memory_provider/__init__.py:4024", "hermes_memory_provider/__init__.py:4465"]` → Legacy adapter references are insufficient for the catalogue path; installed provider dispatch and BeamMemory storage bodies establish scope. Evidence: `sources/mnemosyne/integrations/hermes-catalog/pyproject.toml:10`, `sources/mnemosyne/mnemosyne/core/beam.py:8003`.
- `context.bounds_status`: `"Source expressions include recall and other output bounds. They are not interchangeable: token vs character limits and static block vs tool output must be read at their cited call sites. No single total bound is established unless explicitly stated."` → Top-k is five, but content truncation defaults to zero/disabled; no fixed aggregate character cap. Evidence: `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:351`.
- `engine_delete.scope`: `"Named deletion/retraction endpoint or method, not a guarantee of physical erasure. Its reach may overlap an agent tool; lack of a native tool is counted separately."` → `yes`: mnemosyne_forget calls `_forget_with_episodic_fallback` and reports deleted/not_found (`sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:4934`). mnemosyne_forget_canonical retires the owner/category/name slot (`sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:4578`). mnemosyne_invalidate marks invalid; mnemosyne_validate(action=delete) and batch/replay paths must not be conflated with plain update (`sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:4237`; `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:4745`). Engine `forget_working` removes session-or-global authorized row and vector, conditionally cascades children only without a competing episodic parent, then invalidates query cache (`sources/mnemosyne/mnemosyne/core/beam.py:8001`; `sources/mnemosyne/mnemosyne/core/beam.py:8008`; `sources/mnemosyne/mnemosyne/core/beam.py:8014`). forget_episodic has analogous ownership guards (`sources/mnemosyne/mnemosyne/core/beam.py:8048`). Derived reach `partial`: ambiguity deliberately preserves shared children, and tier/canonical/sync copies need separate handling. Evidence: `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:4934`, `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:4578`, `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:4237`, `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:4745`, `sources/mnemosyne/mnemosyne/core/beam.py:8001`, `sources/mnemosyne/mnemosyne/core/beam.py:8008`, `sources/mnemosyne/mnemosyne/core/beam.py:8014`, `sources/mnemosyne/mnemosyne/core/beam.py:8048`.
- `context.mechanism.v`: `"Source expressions below identify retrieval, cached/static returns, lifecycle capture, and limits. A hook definition is not a guarantee it emits nonempty context each turn; branches and configuration apply."` → System prompt changes with active backend/persona mode (`sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:2777`). Prefetch filters low-quality or non-distinctive hits, semantically deduplicates, takes _PREFETCH_TOP_K, optionally caps each formatted content and joins timestamp/importance/provenance lines (`sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:2929`; `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:2954`). It can return empty by context/query/backend conditions, not guaranteed recall each turn. Tool JSON assembly is distinct. Five hits is the default top-k; per-memory content truncation defaults to 0 (disabled), so there is no fixed default aggregate character bound (`sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:351`; `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:384`). Evidence: `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:2777`, `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:2929`, `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:2954`, `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:351`, `sources/mnemosyne/integrations/hermes/src/mnemosyne_hermes/__init__.py:384`.
- `sec.telemetry.v`: `"inspect listed emitters/configuration"` → Local SQLite/bank files are reachable independently by the same OS user. Accountless local mode `yes`; optional embedding/model downloads and sync endpoints are not implied to be offline. Credential/env/model/sync paths and file modes are enumerated below. A provider ownership predicate is not an OS access boundary (`sources/mnemosyne/mnemosyne/core/beam.py:8003`). Telemetry defaults across optional dependencies `unknown`. Evidence: `sources/mnemosyne/mnemosyne/core/beam.py:8003`.
- `repro.decls`: `["integrations/hermes/src/mnemosyne_hermes/plugin.yaml:6", "integrations/hermes/src/mnemosyne_hermes/plugin.yaml:7", "integrations/hermes/src/mnemosyne_hermes/plugin.yaml:8", "integrations/hermes/src/mnemosyne_hermes/plugin.yaml:9", "integrations/hermes/src/mnemosyne_hermes/plugin.yaml:10", "integrations/hermes/src/mnemosyne_hermes/plugin.yaml:11", "integrations/hermes/src/mnemosyne_hermes/plugin.yaml:12", "integrations/hermes/src/mnemosyne_hermes/plugin.yaml:13", "integrations/hermes/src/mnemosyne_hermes/plugin.yaml:14", "integrations/hermes/src/mnemosyne_hermes/plugin.yaml:15", "integrations/hermes/src/mnemosyne_hermes/plugin.yaml:16", "integrations/hermes/src/mnemosyne_hermes/plugin.yaml:17", "integrations/hermes/src/mnemosyne_hermes/plugin.yaml:18", "integrations/hermes/src/mnemosyne_hermes/plugin.yaml:19", "integrations/hermes/src/mnemosyne_hermes/plugin.yaml:20", "integrations/hermes/src/mnemosyne_hermes/plugin.yaml:21", "integrations/hermes/src/mnemosyne_hermes/plugin.yaml:22", "integrations/hermes/src/mnemosyne_hermes/plugin.yaml:23", "integrations/hermes/src/mnemosyne_hermes/plugin.yaml:24", "integrations/hermes/src/mnemosyne_hermes/plugin.yaml:25", "integrations/hermes/src/mnemosyne_hermes/plugin.yaml:27", "integrations/hermes/src/mnemosyne_hermes/plugin.yaml:28", "integrations/hermes/src/mnemosyne_hermes/plugin.yaml:29", "integrations/hermes/src/mnemosyne_hermes/plugin.yaml:30", "integrations/hermes-catalog/plugin.yaml:8", "integrations/hermes-catalog/plugin.yaml:17", "integrations/hermes-catalog/plugin.yaml:18", "integrations/hermes-catalog/plugin.yaml:19", "integrations/hermes-catalog/plugin.yaml:20", "integrations/hermes-catalog/plugin.yaml:21", "integrations/hermes-catalog/plugin.yaml:22", "integrations/hermes-catalog/plugin.yaml:23", "integrations/hermes-catalog/plugin.yaml:24", "integrations/hermes-catalog/plugin.yaml:25", "integrations/hermes-catalog/plugin.yaml:26", "integrations/hermes-catalog/plugin.yaml:27", "integrations/hermes-catalog/plugin.yaml:28", "integrations/hermes-catalog/plugin.yaml:29", "integrations/hermes-catalog/plugin.yaml:30", "integrations/hermes-catalog/plugin.yaml:31", "integrations/hermes-catalog/plugin.yaml:32", "integrations/hermes-catalog/plugin.yaml:33", "integrations/hermes-catalog/plugin.yaml:34", "integrations/hermes-catalog/plugin.yaml:35", "integrations/hermes-catalog/plugin.yaml:36", "integrations/hermes-catalog/plugin.yaml:37", "integrations/hermes-catalog/plugin.yaml:38", "integrations/hermes-catalog/plugin.yaml:39", "integrations/hermes-catalog/plugin.yaml:40", "integrations/hermes-catalog/plugin.yaml:41", "integrations/hermes-catalog/plugin.yaml:42", "integrations/hermes-catalog/plugin.yaml:43", "integrations/hermes-catalog/plugin.yaml:44", "integrations/hermes-catalog/plugin.yaml:45", "integrations/hermes-catalog/plugin.yaml:46", "integrations/hermes-catalog/plugin.yaml:47", "integrations/hermes-catalog/plugin.yaml:48", "integrations/hermes-catalog/plugin.yaml:49", "integrations/hermes-catalog/plugin.yaml:50", "integrations/hermes-catalog/plugin.yaml:51", "integrations/hermes-catalog/plugin.yaml:52", "integrations/hermes-catalog/plugin.yaml:53", "integrations/hermes-catalog/plugin.yaml:54", "integrations/hermes-catalog/plugin.yaml:55", "integrations/hermes-catalog/plugin.yaml:56"]` → `range`: mnemosyne-hermes>=0.7.7,<0.8 and mnemosyne-memory[embeddings]>=4.0.0 (`sources/mnemosyne/integrations/hermes-catalog/pyproject.toml:10`). Engine has no upper bound. Legacy adapter source in the repo is not proof that it is the installed entrypoint. Evidence: `sources/mnemosyne/integrations/hermes-catalog/pyproject.toml:10`.
- `tests.count`: `5027` → 771 adapter-related declarations using the stated parser/scope; B2 counted a different, repository-wide inventory. This is a scope refinement, not a count of executed or passing tests. Native integration and legacy provider test families are listed separately by path. Engine forget_episodic/working tests include derived-row/ownership behavior; these are not proof of backup or host-history erasure. Engine test asserts owned row, unambiguous child rows and episodic vector are removed. (`sources/mnemosyne/tests/test_forget_episodic.py:128`). Evidence: `sources/mnemosyne/tests/test_forget_episodic.py:128`.

MATRIX counts these 7 explicit field corrections/refinements. Sections 1-6 provide the body and test inventories. Test-name candidates are not passing tests or measured coverage.
