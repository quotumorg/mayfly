# entropicmem: B3 static correctness review

Snapshot `12ca8ae06ca4b9da0271a49d7cf7ce40d019befd` (MANIFEST.json row `entropicmem`). No plugin execution or passing-test claims.

## 1. Architecture

`provider`: catalogue entry imports inherited implementation from bundled scripts/em/provider; facade/store code runs in-process over SQLite and optional projections/vectors. Background prefetch/extraction is separate from sync tool calls (`sources/entropicmem/plugins/entropicmem/scripts/em/provider/provider.py:713`; `sources/entropicmem/plugins/entropicmem/scripts/em/provider/tools.py:1255`).

Hook/body inventory (no-op/fallback declarations are not active behavior):

| callable | declaration | first executable body line |
|---|---|---|
| `register` | `sources/entropicmem/plugins/entropicmem/__init__.py:143` | `sources/entropicmem/plugins/entropicmem/__init__.py:145` |
| `register` | `sources/entropicmem/plugins/entropicmem/scripts/em/jobs/worker.py:68` | `sources/entropicmem/plugins/entropicmem/scripts/em/jobs/worker.py:73` |
| `prefetch` | `sources/entropicmem/plugins/entropicmem/scripts/em/provider/prefetch_service.py:558` | `sources/entropicmem/plugins/entropicmem/scripts/em/provider/prefetch_service.py:574` |
| `is_available` | `sources/entropicmem/plugins/entropicmem/scripts/em/provider/provider.py:323` | `sources/entropicmem/plugins/entropicmem/scripts/em/provider/provider.py:324` |
| `get_config_schema` | `sources/entropicmem/plugins/entropicmem/scripts/em/provider/provider.py:334` | `sources/entropicmem/plugins/entropicmem/scripts/em/provider/provider.py:342` |
| `save_config` | `sources/entropicmem/plugins/entropicmem/scripts/em/provider/provider.py:368` | `sources/entropicmem/plugins/entropicmem/scripts/em/provider/provider.py:401` |
| `initialize` | `sources/entropicmem/plugins/entropicmem/scripts/em/provider/provider.py:531` | `sources/entropicmem/plugins/entropicmem/scripts/em/provider/provider.py:532` |
| `system_prompt_block` | `sources/entropicmem/plugins/entropicmem/scripts/em/provider/provider.py:631` | `sources/entropicmem/plugins/entropicmem/scripts/em/provider/provider.py:632` |
| `queue_prefetch` | `sources/entropicmem/plugins/entropicmem/scripts/em/provider/provider.py:695` | `sources/entropicmem/plugins/entropicmem/scripts/em/provider/provider.py:696` |
| `prefetch` | `sources/entropicmem/plugins/entropicmem/scripts/em/provider/provider.py:713` | `sources/entropicmem/plugins/entropicmem/scripts/em/provider/provider.py:727` |
| `sync_turn` | `sources/entropicmem/plugins/entropicmem/scripts/em/provider/provider.py:1100` | `sources/entropicmem/plugins/entropicmem/scripts/em/provider/provider.py:1119` |
| `on_memory_write` | `sources/entropicmem/plugins/entropicmem/scripts/em/provider/provider.py:1576` | `sources/entropicmem/plugins/entropicmem/scripts/em/provider/provider.py:1591` |
| `on_session_switch` | `sources/entropicmem/plugins/entropicmem/scripts/em/provider/provider.py:1651` | `sources/entropicmem/plugins/entropicmem/scripts/em/provider/provider.py:1669` |
| `on_delegation` | `sources/entropicmem/plugins/entropicmem/scripts/em/provider/provider.py:1685` | `sources/entropicmem/plugins/entropicmem/scripts/em/provider/provider.py:1705` |
| `on_session_end` | `sources/entropicmem/plugins/entropicmem/scripts/em/provider/provider.py:1756` | `sources/entropicmem/plugins/entropicmem/scripts/em/provider/provider.py:1763` |
| `on_turn_start` | `sources/entropicmem/plugins/entropicmem/scripts/em/provider/provider.py:1788` | `sources/entropicmem/plugins/entropicmem/scripts/em/provider/provider.py:1818` |
| `on_pre_compress` | `sources/entropicmem/plugins/entropicmem/scripts/em/provider/provider.py:1859` | `sources/entropicmem/plugins/entropicmem/scripts/em/provider/provider.py:1893` |
| `shutdown` | `sources/entropicmem/plugins/entropicmem/scripts/em/provider/provider.py:2057` | `sources/entropicmem/plugins/entropicmem/scripts/em/provider/provider.py:2060` |
| `get_tool_schemas` | `sources/entropicmem/plugins/entropicmem/scripts/em/provider/tools.py:421` | `sources/entropicmem/plugins/entropicmem/scripts/em/provider/tools.py:422` |
| `handle_tool_call` | `sources/entropicmem/plugins/entropicmem/scripts/em/provider/tools.py:425` | `sources/entropicmem/plugins/entropicmem/scripts/em/provider/tools.py:426` |

## 2. Forget paths and derived copies

`yes`: entropicmem_forget by ID calls _forget_one immediately; query previews candidates and confirmation-token apply is separate (`sources/entropicmem/plugins/entropicmem/scripts/em/provider/tools.py:1177`; `sources/entropicmem/plugins/entropicmem/scripts/em/provider/tools.py:1186`). v3 _forget_one checks scoped row, sets status deleted and enqueues projection (`sources/entropicmem/plugins/entropicmem/scripts/em/provider/tools.py:1264`; `sources/entropicmem/plugins/entropicmem/scripts/em/provider/tools.py:1274`). It does not physically delete the row. Facade forget requires confirm=True, can snapshot before changing status, and preserves history (`sources/entropicmem/plugins/entropicmem/scripts/em/facade/engine.py:825`; `sources/entropicmem/plugins/entropicmem/scripts/em/facade/engine.py:828`). Engine purge is a separate physical operation in the ledger. Derived reach `partial`: visibility/projection changes do not erase versions, audit, backups or host copies.

Host state.db is independent (`hermes-agent/hermes_state.py:181`). No reviewed adapter forget body performs a general purge of that database or built-in MEMORY.md/USER.md. Prompt/tool-result copies and independent source transcripts remain separate; source operation bodies below establish the scope of each plugin mutation.

<details>
<summary>Deletion, invalidation, archive and reset body ledger</summary>

Public routes, SDK/CLI dispatch and internal storage operations are distinguished by their source. Cache invalidation and management-object deletion are not additional fact-forget tools. A stub is not working deletion. Call-through means the downstream body still determines erasure. HTTP prefixes/selectors must be read with route registration.

**_deletes_committed**, body `sources/entropicmem/plugins/entropicmem/scripts/em/embeddings/prune.py:141`; implementation/call-through.

- `sources/entropicmem/plugins/entropicmem/scripts/em/embeddings/prune.py:138` `def _deletes_committed(store, prunable: list[str]) -> Callable[[], None]:`
- `sources/entropicmem/plugins/entropicmem/scripts/em/embeddings/prune.py:141` `def check() -> None:`

**forget**, body `sources/entropicmem/plugins/entropicmem/scripts/em/facade/contract.py:206`; stub.

- `sources/entropicmem/plugins/entropicmem/scripts/em/facade/contract.py:206` `def forget(self, entropic_id: str, *, confirm: bool = False) -> bool: ...`

**archive_pre_compress**, body `sources/entropicmem/plugins/entropicmem/scripts/em/facade/contract.py:296`; stub.

- `sources/entropicmem/plugins/entropicmem/scripts/em/facade/contract.py:287` `def archive_pre_compress(`
- `sources/entropicmem/plugins/entropicmem/scripts/em/facade/contract.py:296` `) -> Dict[str, Any]: ...`

**forget**, body `sources/entropicmem/plugins/entropicmem/scripts/em/facade/engine.py:812`; implementation/call-through.

- `sources/entropicmem/plugins/entropicmem/scripts/em/facade/engine.py:802` `def forget(self, entropic_id: str, *, confirm: bool = False) -> bool:`
- `sources/entropicmem/plugins/entropicmem/scripts/em/facade/engine.py:819` `if row is None or row["status"] == "deleted":`
- `sources/entropicmem/plugins/entropicmem/scripts/em/facade/engine.py:828` `result = MemoryStore(conn).set_status(`

**archive_pre_compress**, body `sources/entropicmem/plugins/entropicmem/scripts/em/facade/engine.py:1112`; implementation/call-through.

- `sources/entropicmem/plugins/entropicmem/scripts/em/facade/engine.py:1089` `def archive_pre_compress(`
- `sources/entropicmem/plugins/entropicmem/scripts/em/facade/engine.py:1112` `new_chunks = 0`

**reset**, body `sources/entropicmem/plugins/entropicmem/scripts/em/provider/hooks.py:131`; implementation/call-through.

- `sources/entropicmem/plugins/entropicmem/scripts/em/provider/hooks.py:130` `def reset(self) -> None:`
- `sources/entropicmem/plugins/entropicmem/scripts/em/provider/hooks.py:131` `with self._lock:`

**reset**, body `sources/entropicmem/plugins/entropicmem/scripts/em/provider/identity.py:132`; implementation/call-through.

- `sources/entropicmem/plugins/entropicmem/scripts/em/provider/identity.py:130` `def reset(self) -> None:`
- `sources/entropicmem/plugins/entropicmem/scripts/em/provider/identity.py:132` `self._key = None`

**reset**, body `sources/entropicmem/plugins/entropicmem/scripts/em/provider/prefetch_service.py:243`; implementation/call-through.

- `sources/entropicmem/plugins/entropicmem/scripts/em/provider/prefetch_service.py:241` `def reset(self) -> None:`
- `sources/entropicmem/plugins/entropicmem/scripts/em/provider/prefetch_service.py:243` `with self._lock:`

**reset**, body `sources/entropicmem/plugins/entropicmem/scripts/em/provider/prefetch_service.py:503`; implementation/call-through.

- `sources/entropicmem/plugins/entropicmem/scripts/em/provider/prefetch_service.py:497` `def reset(self) -> None:`
- `sources/entropicmem/plugins/entropicmem/scripts/em/provider/prefetch_service.py:503` `self._cache.clear()`

**_forget_token_store**, body `sources/entropicmem/plugins/entropicmem/scripts/em/provider/tools.py:514`; implementation/call-through.

- `sources/entropicmem/plugins/entropicmem/scripts/em/provider/tools.py:513` `def _forget_token_store(self) -> Dict[str, Any]:`
- `sources/entropicmem/plugins/entropicmem/scripts/em/provider/tools.py:514` `store = getattr(self, "_forget_tokens", None)`

**_forget**, body `sources/entropicmem/plugins/entropicmem/scripts/em/provider/tools.py:1160`; implementation/call-through.

- `sources/entropicmem/plugins/entropicmem/scripts/em/provider/tools.py:1159` `def _forget(self, args: dict) -> str:`
- `sources/entropicmem/plugins/entropicmem/scripts/em/provider/tools.py:1160` `token = (args.get("confirm_token") or "").strip()`

**_forget_preview**, body `sources/entropicmem/plugins/entropicmem/scripts/em/provider/tools.py:1193`; implementation/call-through.

- `sources/entropicmem/plugins/entropicmem/scripts/em/provider/tools.py:1192` `def _forget_preview(self, query: str) -> str:`
- `sources/entropicmem/plugins/entropicmem/scripts/em/provider/tools.py:1193` `rows = self._search_memories_by_query(query, [], "", "", False, 25)`

**_forget_apply**, body `sources/entropicmem/plugins/entropicmem/scripts/em/provider/tools.py:1224`; implementation/call-through.

- `sources/entropicmem/plugins/entropicmem/scripts/em/provider/tools.py:1223` `def _forget_apply(self, token: str, reason: str) -> str:`
- `sources/entropicmem/plugins/entropicmem/scripts/em/provider/tools.py:1247` `"status": "deleted",`

**_forget_one**, body `sources/entropicmem/plugins/entropicmem/scripts/em/provider/tools.py:1255`; implementation/call-through.

- `sources/entropicmem/plugins/entropicmem/scripts/em/provider/tools.py:1254` `def _forget_one(self, memory_id: str, reason: str) -> Dict[str, Any]:`
- `sources/entropicmem/plugins/entropicmem/scripts/em/provider/tools.py:1267` `if row["status"] == "deleted":`
- `sources/entropicmem/plugins/entropicmem/scripts/em/provider/tools.py:1271` `"status": "deleted",`
- `sources/entropicmem/plugins/entropicmem/scripts/em/provider/tools.py:1274` `result = memories.set_status(`
- `sources/entropicmem/plugins/entropicmem/scripts/em/provider/tools.py:1286` `return {"ok": True, "id": result.id, "status": "deleted", "decision": "forgotten"}`
- `sources/entropicmem/plugins/entropicmem/scripts/em/provider/tools.py:1288` `gone = engine.forget(memory_id, confirm=True)`
- `sources/entropicmem/plugins/entropicmem/scripts/em/provider/tools.py:1291` `return {"ok": True, "id": memory_id, "status": "deleted", "decision": "forgotten"}`

**delete_rows_for_models**, body `sources/entropicmem/plugins/entropicmem/scripts/em/store/embeddings.py:424`; implementation/call-through.

- `sources/entropicmem/plugins/entropicmem/scripts/em/store/embeddings.py:417` `def delete_rows_for_models(conn: sqlite3.Connection, models: Sequence[str]) -> int:`
- `sources/entropicmem/plugins/entropicmem/scripts/em/store/embeddings.py:428` `f"DELETE FROM embeddings WHERE model IN ({marks})", tuple(models)`

**forget_sightings**, body `sources/entropicmem/plugins/entropicmem/scripts/em/store/entities.py:299`; implementation/call-through.

- `sources/entropicmem/plugins/entropicmem/scripts/em/store/entities.py:289` `def forget_sightings(self, memory_id: str, *, profile: str) -> int:`
- `sources/entropicmem/plugins/entropicmem/scripts/em/store/entities.py:316` `self._conn.execute("DELETE FROM meta WHERE key=?", (row["key"],))`

**_deletion_reason**, body `sources/entropicmem/plugins/entropicmem/scripts/em/store/memories.py:855`; implementation/call-through.

- `sources/entropicmem/plugins/entropicmem/scripts/em/store/memories.py:852` `def _deletion_reason(self, memory_id: str) -> str:`
- `sources/entropicmem/plugins/entropicmem/scripts/em/store/memories.py:856` `"SELECT change_reason FROM memory_versions WHERE memory_id=? AND status='deleted'"`

**purge**, body `sources/entropicmem/plugins/entropicmem/scripts/em/store/memories.py:1072`; implementation/call-through.

- `sources/entropicmem/plugins/entropicmem/scripts/em/store/memories.py:1045` `def purge(`
- `sources/entropicmem/plugins/entropicmem/scripts/em/store/memories.py:1077` `self._conn.execute("DELETE FROM memory_versions WHERE memory_id=?", (memory_id,))`
- `sources/entropicmem/plugins/entropicmem/scripts/em/store/memories.py:1078` `self._conn.execute("DELETE FROM relations WHERE memory_id=?", (memory_id,))`
- `sources/entropicmem/plugins/entropicmem/scripts/em/store/memories.py:1079` `self._conn.execute("DELETE FROM embeddings WHERE owner_type='memory' AND owner_id=?", (memory_id,))`
- `sources/entropicmem/plugins/entropicmem/scripts/em/store/memories.py:1080` `self._conn.execute("DELETE FROM sync_outbox WHERE fact_id=?", (memory_id,))`
- `sources/entropicmem/plugins/entropicmem/scripts/em/store/memories.py:1084` `self._conn.execute("DELETE FROM memory_entities WHERE memory_id=?", (memory_id,))`
- `sources/entropicmem/plugins/entropicmem/scripts/em/store/memories.py:1085` `self._conn.execute("DELETE FROM memories WHERE id=?", (memory_id,))`
- `sources/entropicmem/plugins/entropicmem/scripts/em/store/memories.py:1091` `EntityStore(self._conn).forget_sightings(memory_id, profile=row["scope_profile"])`

**check_deletion_reversal**, body `sources/entropicmem/plugins/entropicmem/scripts/em/store/types.py:95`; implementation/call-through.

- `sources/entropicmem/plugins/entropicmem/scripts/em/store/types.py:91` `def check_deletion_reversal(`
- `sources/entropicmem/plugins/entropicmem/scripts/em/store/types.py:95` `if current != "deleted" or target != "active":`

**invalidate_vector_cache**, body `sources/entropicmem/plugins/entropicmem/scripts/em_internal/embeddings.py:153`; implementation/call-through.

- `sources/entropicmem/plugins/entropicmem/scripts/em_internal/embeddings.py:151` `def invalidate_vector_cache() -> None:`
- `sources/entropicmem/plugins/entropicmem/scripts/em_internal/embeddings.py:153` `global _write_gen`

**delete_embedding**, body `sources/entropicmem/plugins/entropicmem/scripts/em_internal/embeddings.py:232`; implementation/call-through.

- `sources/entropicmem/plugins/entropicmem/scripts/em_internal/embeddings.py:230` `def delete_embedding(db: sqlite3.Connection, fact_id: str) -> None:`
- `sources/entropicmem/plugins/entropicmem/scripts/em_internal/embeddings.py:232` `db.execute("DELETE FROM embeddings WHERE fact_id = ?", (fact_id,))`
- `sources/entropicmem/plugins/entropicmem/scripts/em_internal/embeddings.py:234` `invalidate_vector_cache()`

**delete_note**, body `sources/entropicmem/plugins/entropicmem/scripts/em_internal/index.py:617`; implementation/call-through.

- `sources/entropicmem/plugins/entropicmem/scripts/em_internal/index.py:615` `def delete_note(self, note_id: str) -> None:`
- `sources/entropicmem/plugins/entropicmem/scripts/em_internal/index.py:617` `self.db.execute("DELETE FROM notes_fts WHERE note_id = ?", (note_id,))`
- `sources/entropicmem/plugins/entropicmem/scripts/em_internal/index.py:618` `self.db.execute("DELETE FROM notes_meta WHERE note_id = ?", (note_id,))`
- `sources/entropicmem/plugins/entropicmem/scripts/em_internal/index.py:620` `"DELETE FROM graph_edges WHERE source_id = ? OR target_id = ?",`

**delete_note**, body `sources/entropicmem/plugins/entropicmem/scripts/em_internal/vault.py:455`; implementation/call-through.

- `sources/entropicmem/plugins/entropicmem/scripts/em_internal/vault.py:450` `def delete_note(self, path: Path) -> None:`
- `sources/entropicmem/plugins/entropicmem/scripts/em_internal/vault.py:460` `full.unlink()`

**cmd_forget**, body `sources/entropicmem/plugins/entropicmem/scripts/entropicmem.py:1396`; implementation/call-through.

- `sources/entropicmem/plugins/entropicmem/scripts/entropicmem.py:1395` `def cmd_forget(args) -> int:`
- `sources/entropicmem/plugins/entropicmem/scripts/entropicmem.py:1405` `engine.forget(eid, confirm=bool(getattr(args, "confirm", False)))`
- `sources/entropicmem/plugins/entropicmem/scripts/entropicmem.py:1413` `index.delete_note(str(found))`
- `sources/entropicmem/plugins/entropicmem/scripts/entropicmem.py:1414` `vault.delete_note(found)`

**delete_links_for_note**, body `sources/entropicmem/plugins/entropicmem/scripts/graph_query.py:161`; implementation/call-through.

- `sources/entropicmem/plugins/entropicmem/scripts/graph_query.py:159` `def delete_links_for_note(db: sqlite3.Connection, source_path: str) -> None:`
- `sources/entropicmem/plugins/entropicmem/scripts/graph_query.py:172` `"DELETE FROM graph_edges WHERE source_id = ? AND kind = 'wikilink'",`

**forget**, body `sources/entropicmem/plugins/entropicmem/scripts/memory_engine.py:1165`; implementation/call-through.

- `sources/entropicmem/plugins/entropicmem/scripts/memory_engine.py:1163` `def forget(self, entropic_id: str, *, confirm: bool = False) -> bool:`
- `sources/entropicmem/plugins/entropicmem/scripts/memory_engine.py:1180` `self.db.execute("DELETE FROM facts WHERE id = ?", (entropic_id,))`
- `sources/entropicmem/plugins/entropicmem/scripts/memory_engine.py:1182` `self.db.execute("DELETE FROM facts_fts WHERE rowid = ?", (row[0],))`
- `sources/entropicmem/plugins/entropicmem/scripts/memory_engine.py:1187` `self.db.execute("DELETE FROM embeddings WHERE fact_id = ?", (entropic_id,))`
- `sources/entropicmem/plugins/entropicmem/scripts/memory_engine.py:1189` `invalidate_vector_cache()`
- `sources/entropicmem/plugins/entropicmem/scripts/memory_engine.py:1205` `op="delete",`
- `sources/entropicmem/plugins/entropicmem/scripts/memory_engine.py:1206` `# Tombstone uses a NEW version: the shared log's`
- `sources/entropicmem/plugins/entropicmem/scripts/memory_engine.py:1209` `# version would be IGNORED and the tombstone never propagate.`

**archive_pre_compress**, body `sources/entropicmem/plugins/entropicmem/scripts/memory_engine.py:2422`; stub.

- `sources/entropicmem/plugins/entropicmem/scripts/memory_engine.py:2406` `def archive_pre_compress(`
- `sources/entropicmem/plugins/entropicmem/scripts/memory_engine.py:2422` `raise RuntimeError("archive_pre_compress requires a v3 store")`

</details>

## 3. What reaches the agent

System block includes base prompt and optional core memory (`sources/entropicmem/plugins/entropicmem/scripts/em/provider/provider.py:650`). Prefetch combines service-selected fact context and optional conversation context; the fact packing budget alone is not total output size (`sources/entropicmem/plugins/entropicmem/scripts/em/provider/provider.py:755`; `sources/entropicmem/plugins/entropicmem/scripts/em/provider/provider.py:792`). Actual packing/deadline/cache/limit quotes below. Tools return JSON decisions/results, not a universal text memory block.

Host assembly: `hermes-agent/agent/memory_manager.py:352` joins provider system blocks. `hermes-agent/agent/turn_context.py:1081` requests prefetch for nontrivial turns; `hermes-agent/agent/turn_context.py:120` fences it; `hermes-agent/agent/turn_context.py:136` appends it to API-bound user content. Tool results are a third channel. Optional host spill is at `hermes-agent/agent/memory_manager.py:419`, not a universal provider cap. Built-in memory remains separately controlled (`hermes-agent/agent/system_prompt.py:542`).

<details>
<summary>Context and tool-result assembly quotes</summary>

`get_shape`, body `sources/entropicmem/plugins/entropicmem/scripts/em/graph/model.py:54`:

- `sources/entropicmem/plugins/entropicmem/scripts/em/graph/model.py:54` `return TYPE_SHAPES.get(note_type, "circle")`

`prefetch`, body `sources/entropicmem/plugins/entropicmem/scripts/em/provider/prefetch_service.py:574`:

- `sources/entropicmem/plugins/entropicmem/scripts/em/provider/prefetch_service.py:576` `return PrefetchResult("")`
- `sources/entropicmem/plugins/entropicmem/scripts/em/provider/prefetch_service.py:578` `return PrefetchResult("")`
- `sources/entropicmem/plugins/entropicmem/scripts/em/provider/prefetch_service.py:585` `deadline = time.monotonic() + self._max_latency_ms / 1000.0`
- `sources/entropicmem/plugins/entropicmem/scripts/em/provider/prefetch_service.py:626` `while len(self._cache) > _CACHE_MAX_ENTRIES:`
- `sources/entropicmem/plugins/entropicmem/scripts/em/provider/prefetch_service.py:633` `chosen = pack_chosen(kept, budget=self._token_budget, on_flagged="mark", cite="full")`
- `sources/entropicmem/plugins/entropicmem/scripts/em/provider/prefetch_service.py:635` `cap = min(self._token_budget * 4, OUTPUT_CHAR_CAP)`
- `sources/entropicmem/plugins/entropicmem/scripts/em/provider/prefetch_service.py:637` `chosen = chosen[:-1]`
- `sources/entropicmem/plugins/entropicmem/scripts/em/provider/prefetch_service.py:664` `return result`

`_agent_context`, body `sources/entropicmem/plugins/entropicmem/scripts/em/provider/provider.py:277`:

- `sources/entropicmem/plugins/entropicmem/scripts/em/provider/provider.py:277` `return self._state.agent_context`

`_agent_context`, body `sources/entropicmem/plugins/entropicmem/scripts/em/provider/provider.py:281`:



`system_prompt_block`, body `sources/entropicmem/plugins/entropicmem/scripts/em/provider/provider.py:632`:

- `sources/entropicmem/plugins/entropicmem/scripts/em/provider/provider.py:633` `return (`
- `sources/entropicmem/plugins/entropicmem/scripts/em/provider/provider.py:643` `return base`
- `sources/entropicmem/plugins/entropicmem/scripts/em/provider/provider.py:646` `return base`
- `sources/entropicmem/plugins/entropicmem/scripts/em/provider/provider.py:650` `return base + "\n\n" + core`

`queue_prefetch`, body `sources/entropicmem/plugins/entropicmem/scripts/em/provider/provider.py:696`:

- `sources/entropicmem/plugins/entropicmem/scripts/em/provider/provider.py:697` `self._last_query = query[:2000]`

`prefetch`, body `sources/entropicmem/plugins/entropicmem/scripts/em/provider/provider.py:727`:

- `sources/entropicmem/plugins/entropicmem/scripts/em/provider/provider.py:730` `return ""`
- `sources/entropicmem/plugins/entropicmem/scripts/em/provider/provider.py:792` `response = "\n\n".join(blocks)`
- `sources/entropicmem/plugins/entropicmem/scripts/em/provider/provider.py:794` `return response`
- `sources/entropicmem/plugins/entropicmem/scripts/em/provider/provider.py:798` `return ""`

`_prefetch_service_for_turn`, body `sources/entropicmem/plugins/entropicmem/scripts/em/provider/provider.py:905`:

- `sources/entropicmem/plugins/entropicmem/scripts/em/provider/provider.py:907` `return None`
- `sources/entropicmem/plugins/entropicmem/scripts/em/provider/provider.py:910` `return service if v3_retrieval_enabled() else None`

`_ensure_prefetch_service`, body `sources/entropicmem/plugins/entropicmem/scripts/em/provider/provider.py:2458`:

- `sources/entropicmem/plugins/entropicmem/scripts/em/provider/provider.py:2459` `return self._prefetch_service`
- `sources/entropicmem/plugins/entropicmem/scripts/em/provider/provider.py:2461` `return None`
- `sources/entropicmem/plugins/entropicmem/scripts/em/provider/provider.py:2469` `# ''prefetch_token_budget'' folded its character count into tokens, and`
- `sources/entropicmem/plugins/entropicmem/scripts/em/provider/provider.py:2470` `# ''prefetch_max_latency_ms'' maps straight across); the gate/ranking`
- `sources/entropicmem/plugins/entropicmem/scripts/em/provider/provider.py:2476` `token_budget=int(self._typed_config.prefetch.token_budget),`
- `sources/entropicmem/plugins/entropicmem/scripts/em/provider/provider.py:2477` `max_latency_ms=float(self._typed_config.prefetch.max_latency_ms),`
- `sources/entropicmem/plugins/entropicmem/scripts/em/provider/provider.py:2488` `return service`

`_tool_call_failed`, body `sources/entropicmem/plugins/entropicmem/scripts/em/provider/tools.py:247`:

- `sources/entropicmem/plugins/entropicmem/scripts/em/provider/tools.py:248` `return args[0]._error(f"{tool_name}: internal error (see provider logs)")`

`get_tool_schemas`, body `sources/entropicmem/plugins/entropicmem/scripts/em/provider/tools.py:422`:

- `sources/entropicmem/plugins/entropicmem/scripts/em/provider/tools.py:422` `return list(TOOL_SCHEMAS)`

`handle_tool_call`, body `sources/entropicmem/plugins/entropicmem/scripts/em/provider/tools.py:426`:

- `sources/entropicmem/plugins/entropicmem/scripts/em/provider/tools.py:434` `return self._error(f"Unknown tool: {tool_name}")`
- `sources/entropicmem/plugins/entropicmem/scripts/em/provider/tools.py:443` `return self._blocked(str(exc), "invalid_arguments", deprecated=deprecated)`
- `sources/entropicmem/plugins/entropicmem/scripts/em/provider/tools.py:455` `return _merge_response(handlers[target](cleaned), warnings, deprecated)`

`strip_memory_context`, body `sources/entropicmem/plugins/entropicmem/scripts/em/retrieval/query.py:96`:

- `sources/entropicmem/plugins/entropicmem/scripts/em/retrieval/query.py:96` `return _MEMORY_CONTEXT_RE.sub(" ", text)`

`redact_for_prefetch`, body `sources/entropicmem/plugins/entropicmem/scripts/em_internal/policy.py:158`:

- `sources/entropicmem/plugins/entropicmem/scripts/em_internal/policy.py:162` `one = one[:157] + "..."`
- `sources/entropicmem/plugins/entropicmem/scripts/em_internal/policy.py:163` `return f"[sensitive] {one}"`
- `sources/entropicmem/plugins/entropicmem/scripts/em_internal/policy.py:164` `return content`

`extract_links_with_context`, body `sources/entropicmem/plugins/entropicmem/scripts/graph_query.py:70`:

- `sources/entropicmem/plugins/entropicmem/scripts/graph_query.py:76` `results.append((target, context))`
- `sources/entropicmem/plugins/entropicmem/scripts/graph_query.py:77` `return results`

</details>

## 4. Data access boundaries

Direct local SQLite/vault/projection access `yes` under same UID. Accountless local mode `yes`: bundled stdlib engine has no mandatory dependency/key (`sources/entropicmem/plugins/entropicmem/plugin.yaml:14`). Optional embeddings/extraction can call configured hosts; local-only core must not be generalized to all modes. File modes/env paths and opt-in network flags are listed below; no core remote analytics emitter established.

Boundary model: same OS user, terminal, mounted filesystem and allowed network. Direct reachability means another client can use the same file/endpoint and available credentials, not unauthenticated access or encryption bypass. Hermes setup persists declared secrets through `hermes-agent/hermes_cli/memory_setup.py:321` and `hermes-agent/hermes_cli/config.py:2697`. These are possible configured paths; no live credentials were read.

Host credential-file detail: `get_hermes_home()/.env` is resolved at `hermes-agent/hermes_cli/config.py:473`. New files are secured to 0600 at `hermes-agent/hermes_cli/config.py:506`; the writer preserves an existing mode at `hermes-agent/hermes_cli/config.py:2544` and `:2562`. Therefore 0600 is not an unconditional mode claim for every existing profile.

<details>
<summary>Paths, environment, permissions, egress, telemetry and process-launch evidence</summary>

- `sources/entropicmem/plugins/entropicmem/__init__.py:7` `/learn https://github.com/Ufonik88/EntropicMem`

- `sources/entropicmem/plugins/entropicmem/_backend.py:15` `hh = kwargs.get("hermes_home") or os.environ.get("HERMES_HOME", "")`
- `sources/entropicmem/plugins/entropicmem/_backend.py:18` `return Path.home() / ".hermes"`
- `sources/entropicmem/plugins/entropicmem/_backend.py:114` `or os.environ.get("ENTROPICMEM_INDEX_DB")`
- `sources/entropicmem/plugins/entropicmem/_backend.py:119` `or os.environ.get("ENTROPICMEM_MEMORY_DB")`

- `sources/entropicmem/plugins/entropicmem/_shadow.py:129` `raw = os.environ.get(SHADOW_ENV, "").strip()`
- `sources/entropicmem/plugins/entropicmem/_shadow.py:149` `except Exception:  # noqa: BLE001 - telemetry must never raise`
- `sources/entropicmem/plugins/entropicmem/_shadow.py:155` `except Exception:  # noqa: BLE001 - telemetry must never raise`
- `sources/entropicmem/plugins/entropicmem/_shadow.py:165` `raw = os.environ.get(LOG_ENV, "").strip()`
- `sources/entropicmem/plugins/entropicmem/_shadow.py:326` `except Exception:  # noqa: BLE001 - telemetry must never raise`

- `sources/entropicmem/plugins/entropicmem/config_schema.py:28` `DOCS_URL = "https://github.com/Ufonik88/EntropicMem#configuration"`

- `sources/entropicmem/plugins/entropicmem/scripts/em/__init__.py:8` `Hermes host or the provider layer, and never read ''os.environ["HERMES_HOME"]''`

- `sources/entropicmem/plugins/entropicmem/scripts/em/embeddings/backends.py:66` `return dict(os.environ) if env is None else dict(env)`
- `sources/entropicmem/plugins/entropicmem/scripts/em/embeddings/backends.py:209` `return get_secret("ENTROPICMEM_EMBEDDINGS_API_KEY", "")`

- `sources/entropicmem/plugins/entropicmem/scripts/em/embeddings/prune.py:62` `def _vacuum(db_path: "str | Path", *, check: Callable[[], None]) -> None:`
- `sources/entropicmem/plugins/entropicmem/scripts/em/embeddings/prune.py:65` `con = sqlite3.connect(str(db_path), isolation_level=None, timeout=30.0)`
- `sources/entropicmem/plugins/entropicmem/scripts/em/embeddings/prune.py:73` `def make_prune_handler(db_path: "str | Path") -> Callable:`
- `sources/entropicmem/plugins/entropicmem/scripts/em/embeddings/prune.py:133` `_vacuum(db_path, check=_deletes_committed(ctx.store, prunable))`

- `sources/entropicmem/plugins/entropicmem/scripts/em/embeddings/service.py:69` `db_path: str,`
- `sources/entropicmem/plugins/entropicmem/scripts/em/embeddings/service.py:76` `self.db_path = str(db_path)`
- `sources/entropicmem/plugins/entropicmem/scripts/em/embeddings/service.py:82` `preference = preference or (environ or os.environ).get(`

- `sources/entropicmem/plugins/entropicmem/scripts/em/facade/engine.py:130` `return os.environ.get(V3_RETRIEVAL_ENV) == "1"`
- `sources/entropicmem/plugins/entropicmem/scripts/em/facade/engine.py:289` `db_path: "Path | str",`
- `sources/entropicmem/plugins/entropicmem/scripts/em/facade/engine.py:311` `self.db_path = Path(db_path)`
- `sources/entropicmem/plugins/entropicmem/scripts/em/facade/engine.py:312` `self.store = Store(self.db_path)`
- `sources/entropicmem/plugins/entropicmem/scripts/em/facade/engine.py:467` `"db_path": str(self.db_path),`
- `sources/entropicmem/plugins/entropicmem/scripts/em/facade/engine.py:825` `BackupManager(self.db_path).snapshot_if_due(reason="pre-forget")`
- `sources/entropicmem/plugins/entropicmem/scripts/em/facade/engine.py:1194` `BackupManager(self.db_path).snapshot_if_due(reason="pre-consolidate")`

- `sources/entropicmem/plugins/entropicmem/scripts/em/facade/select.py:46` `def store_kind(db_path: Union[Path, str]) -> str:`
- `sources/entropicmem/plugins/entropicmem/scripts/em/facade/select.py:47` `"""Return :data:'V2' or :data:'V3' for the store at ''db_path''.`
- `sources/entropicmem/plugins/entropicmem/scripts/em/facade/select.py:53` `path = Path(db_path)`
- `sources/entropicmem/plugins/entropicmem/scripts/em/facade/select.py:92` `db_path: Union[Path, str],`
- `sources/entropicmem/plugins/entropicmem/scripts/em/facade/select.py:134` `if store_kind(db_path) == V3:`
- `sources/entropicmem/plugins/entropicmem/scripts/em/facade/select.py:138` `db_path,`
- `sources/entropicmem/plugins/entropicmem/scripts/em/facade/select.py:155` `db_path,`

- `sources/entropicmem/plugins/entropicmem/scripts/em/formation/llm.py:340` `env = os.environ if environ is None else environ`
- `sources/entropicmem/plugins/entropicmem/scripts/em/formation/llm.py:347` `else get_secret("ENTROPICMEM_LLM_API_KEY", "")`

- `sources/entropicmem/plugins/entropicmem/scripts/em/graph/export.py:215` `env = os.environ.get("ENTROPICMEM_VAULT_PATH")`

- `sources/entropicmem/plugins/entropicmem/scripts/em/host_secret.py:5` `process launched with. Reading ''os.environ'' directly in a multi-profile`
- `sources/entropicmem/plugins/entropicmem/scripts/em/host_secret.py:12` `back to ''os.environ'', which is the only source there is;`
- `sources/entropicmem/plugins/entropicmem/scripts/em/host_secret.py:14` `multiplexing , return ''default'' and never borrow ''os.environ''. That case`
- `sources/entropicmem/plugins/entropicmem/scripts/em/host_secret.py:26` `def get_secret(name: str, default: str = "") -> str:`
- `sources/entropicmem/plugins/entropicmem/scripts/em/host_secret.py:31` `return os.environ.get(name, default)`
- `sources/entropicmem/plugins/entropicmem/scripts/em/host_secret.py:33` `value = host_get_secret(name)`

- `sources/entropicmem/plugins/entropicmem/scripts/em/jobs/cli.py:17` `from em.store.migrations import MigrationRefused, assert_safe_db_path`
- `sources/entropicmem/plugins/entropicmem/scripts/em/jobs/cli.py:31` `db_path: Path,`
- `sources/entropicmem/plugins/entropicmem/scripts/em/jobs/cli.py:44` `assert_safe_db_path(db_path)`
- `sources/entropicmem/plugins/entropicmem/scripts/em/jobs/cli.py:49` `if not _is_v3(db_path):`
- `sources/entropicmem/plugins/entropicmem/scripts/em/jobs/cli.py:51` `f"refusing to run: {db_path} is not a v3 database "`
- `sources/entropicmem/plugins/entropicmem/scripts/em/jobs/cli.py:58` `registry = build_registry(db_path, types=selected)`
- `sources/entropicmem/plugins/entropicmem/scripts/em/jobs/cli.py:60` `store = Store(str(db_path))`

- `sources/entropicmem/plugins/entropicmem/scripts/em/jobs/registry.py:38` `db_path: "str | Path", *, types: Optional[Iterable[str]] = None`
- `sources/entropicmem/plugins/entropicmem/scripts/em/jobs/registry.py:40` `"""A ''HandlerRegistry'' for *db_path*, restricted to *types* when given.`
- `sources/entropicmem/plugins/entropicmem/scripts/em/jobs/registry.py:52` `registry.register("backup", make_backup_handler(BackupManager(db_path)))`
- `sources/entropicmem/plugins/entropicmem/scripts/em/jobs/registry.py:78` `registry.register("prune", make_prune_handler(db_path))`
- `sources/entropicmem/plugins/entropicmem/scripts/em/jobs/registry.py:96` `service = EmbeddingService(str(db_path))`

- `sources/entropicmem/plugins/entropicmem/scripts/em/net.py:40` `return (os.environ.get(name) or "").strip().lower() in ("1", "true", "yes", "on")`

- `sources/entropicmem/plugins/entropicmem/scripts/em/policy/scopes.py:30` `Nothing here imports the host, reads ''os.environ'', or touches a database:`

- `sources/entropicmem/plugins/entropicmem/scripts/em/provider/provider.py:441` `os.chmod(tmp, stat.S_IMODE(config_path.stat().st_mode))`
- `sources/entropicmem/plugins/entropicmem/scripts/em/provider/provider.py:528` `return {key: os.environ[name] for name, key in pairs if os.environ.get(name)}`
- `sources/entropicmem/plugins/entropicmem/scripts/em/provider/provider.py:627` `"'/learn https://github.com/Ufonik88/EntropicMem' then 'entropicmem init'."`
- `sources/entropicmem/plugins/entropicmem/scripts/em/provider/provider.py:635` `"Skill not installed. Run '/learn https://github.com/Ufonik88/EntropicMem' "`

- `sources/entropicmem/plugins/entropicmem/scripts/em/store/__init__.py:4` `provider layer, and never read ''os.environ["HERMES_HOME"]'' , paths are`

- `sources/entropicmem/plugins/entropicmem/scripts/em/store/backup.py:193` `def _chmod(path: Path, mode: int) -> None:`
- `sources/entropicmem/plugins/entropicmem/scripts/em/store/backup.py:197` `os.chmod(path, mode)`
- `sources/entropicmem/plugins/entropicmem/scripts/em/store/backup.py:299` `db_path: "os.PathLike[str] | str",`
- `sources/entropicmem/plugins/entropicmem/scripts/em/store/backup.py:307` `self.db_path = Path(db_path)`
- `sources/entropicmem/plugins/entropicmem/scripts/em/store/backup.py:308` `self.backup_dir = Path(backup_dir) if backup_dir is not None else self.db_path.parent / "backups"`
- `sources/entropicmem/plugins/entropicmem/scripts/em/store/backup.py:309` `self.index_path = Path(index_path) if index_path is not None else self.db_path.parent / "index.db"`
- `sources/entropicmem/plugins/entropicmem/scripts/em/store/backup.py:389` `if conn is None and not self.db_path.is_file():`
- `sources/entropicmem/plugins/entropicmem/scripts/em/store/backup.py:390` `raise BackupError(f"no database at {self.db_path}")`
- `sources/entropicmem/plugins/entropicmem/scripts/em/store/backup.py:392` `_chmod(self.backup_dir, 0o700)`
- `sources/entropicmem/plugins/entropicmem/scripts/em/store/backup.py:399` `_chmod(partial, 0o700)`
- `sources/entropicmem/plugins/entropicmem/scripts/em/store/backup.py:403` `files.append(self._snapshot_one(partial, ROLE_MEMORY, self.db_path, conn))`
- `sources/entropicmem/plugins/entropicmem/scripts/em/store/backup.py:431` `_chmod(target, 0o600)`
- `sources/entropicmem/plugins/entropicmem/scripts/em/store/backup.py:484` `_chmod(tmp, 0o600)`
- `sources/entropicmem/plugins/entropicmem/scripts/em/store/backup.py:674` `_chmod(raw, 0o700)`
- `sources/entropicmem/plugins/entropicmem/scripts/em/store/backup.py:675` `for source in (self.db_path, self.index_path):`
- `sources/entropicmem/plugins/entropicmem/scripts/em/store/backup.py:680` `_chmod(raw / src.name, 0o600)`
- `sources/entropicmem/plugins/entropicmem/scripts/em/store/backup.py:684` `"""Replace ''db_path'' with a verified copy of ''info''.`
- `sources/entropicmem/plugins/entropicmem/scripts/em/store/backup.py:693` `from .migrations import MigrationRefused, assert_safe_db_path`
- `sources/entropicmem/plugins/entropicmem/scripts/em/store/backup.py:696` `assert_safe_db_path(self.db_path)`
- `sources/entropicmem/plugins/entropicmem/scripts/em/store/backup.py:699` `lock = self.db_path.with_name(self.db_path.name + ".lock")`
- `sources/entropicmem/plugins/entropicmem/scripts/em/store/backup.py:706` `safety = self._safety_copy() if self.db_path.is_file() else None`
- `sources/entropicmem/plugins/entropicmem/scripts/em/store/backup.py:733` `_chmod(staging, 0o600)`
- `sources/entropicmem/plugins/entropicmem/scripts/em/store/backup.py:754` `return self.db_path`

- `sources/entropicmem/plugins/entropicmem/scripts/em/store/db.py:92` `def _chmod_posix(path: Path, mode: int) -> None:`
- `sources/entropicmem/plugins/entropicmem/scripts/em/store/db.py:96` `os.chmod(str(path), mode)`
- `sources/entropicmem/plugins/entropicmem/scripts/em/store/db.py:131` `_chmod_posix(path.parent, 0o700)`
- `sources/entropicmem/plugins/entropicmem/scripts/em/store/db.py:135` `_chmod_posix(path, 0o600)`
- `sources/entropicmem/plugins/entropicmem/scripts/em/store/db.py:195` `fd = os.open(str(path), os.O_CREAT | os.O_EXCL | os.O_RDWR, 0o600)`

- `sources/entropicmem/plugins/entropicmem/scripts/em/store/locking.py:97` `str(self.path), os.O_RDWR | os.O_CREAT, 0o600`

- `sources/entropicmem/plugins/entropicmem/scripts/em/store/migrations/__init__.py:13` `:func:'assert_safe_db_path' refuses by default, and :func:'migrate' calls it`
- `sources/entropicmem/plugins/entropicmem/scripts/em/store/migrations/__init__.py:44` `"assert_safe_db_path",`
- `sources/entropicmem/plugins/entropicmem/scripts/em/store/migrations/__init__.py:174` `def _db_path_of(conn: sqlite3.Connection) -> str:`
- `sources/entropicmem/plugins/entropicmem/scripts/em/store/migrations/__init__.py:182` `def assert_safe_db_path(path: "os.PathLike[str] | str") -> None:`
- `sources/entropicmem/plugins/entropicmem/scripts/em/store/migrations/__init__.py:201` `if os.environ.get(ALLOW_LIVE_ENV) == "1":`
- `sources/entropicmem/plugins/entropicmem/scripts/em/store/migrations/__init__.py:210` `if _path_decision(resolved, Path.home().resolve()) == "refused":`
- `sources/entropicmem/plugins/entropicmem/scripts/em/store/migrations/__init__.py:288` `conn: sqlite3.Connection, *, db_path: str, from_version: int, to_version: int,`
- `sources/entropicmem/plugins/entropicmem/scripts/em/store/migrations/__init__.py:298` `if not db_path:`
- `sources/entropicmem/plugins/entropicmem/scripts/em/store/migrations/__init__.py:302` `info = BackupManager(db_path, backup_dir).create(`
- `sources/entropicmem/plugins/entropicmem/scripts/em/store/migrations/__init__.py:366` `db_path = _db_path_of(conn)`
- `sources/entropicmem/plugins/entropicmem/scripts/em/store/migrations/__init__.py:368` `assert_safe_db_path(db_path)`
- `sources/entropicmem/plugins/entropicmem/scripts/em/store/migrations/__init__.py:373` `db_path=db_path,`

- `sources/entropicmem/plugins/entropicmem/scripts/em/store/types.py:4` `''os.environ'' reads. Values are frozen dataclasses so a draft cannot be`

- `sources/entropicmem/plugins/entropicmem/scripts/em/util/fsperm.py:12` `__all__ = ["chmod_posix"]`
- `sources/entropicmem/plugins/entropicmem/scripts/em/util/fsperm.py:15` `def chmod_posix(path: "os.PathLike[str] | str", mode: int) -> None:`
- `sources/entropicmem/plugins/entropicmem/scripts/em/util/fsperm.py:20` `os.chmod(str(path), mode)`

- `sources/entropicmem/plugins/entropicmem/scripts/em/vault/projection.py:84` `from em.util.fsperm import chmod_posix`
- `sources/entropicmem/plugins/entropicmem/scripts/em/vault/projection.py:281` `os.chmod(temp_name, target.stat().st_mode & 0o7777)`
- `sources/entropicmem/plugins/entropicmem/scripts/em/vault/projection.py:313` `chmod_posix(target.parent, 0o700)`

- `sources/entropicmem/plugins/entropicmem/scripts/em_internal/embeddings.py:59` `return os.environ.get(EMBEDDINGS_ENV, "").strip().lower() in _TRUTHY`

- `sources/entropicmem/plugins/entropicmem/scripts/em_internal/index.py:18` `from em.util.fsperm import chmod_posix`
- `sources/entropicmem/plugins/entropicmem/scripts/em_internal/index.py:182` `def __init__(self, db_path: Path, *, readonly: bool = False):`
- `sources/entropicmem/plugins/entropicmem/scripts/em_internal/index.py:183` `self.db_path = Path(db_path).resolve()`
- `sources/entropicmem/plugins/entropicmem/scripts/em_internal/index.py:188` `self.db = sqlite3.connect(str(self.db_path), timeout=30)`
- `sources/entropicmem/plugins/entropicmem/scripts/em_internal/index.py:189` `chmod_posix(self.db_path, 0o600)`
- `sources/entropicmem/plugins/entropicmem/scripts/em_internal/index.py:199` `migrate, chmod or WAL-write the file. A GET-time reader that could do`
- `sources/entropicmem/plugins/entropicmem/scripts/em_internal/index.py:203` `if not self.db_path.is_file():`
- `sources/entropicmem/plugins/entropicmem/scripts/em_internal/index.py:205` `f"index database not found (read-only open): {self.db_path}"`
- `sources/entropicmem/plugins/entropicmem/scripts/em_internal/index.py:207` `conn = sqlite3.connect(self.db_path.as_uri() + "?mode=ro", uri=True, timeout=30)`
- `sources/entropicmem/plugins/entropicmem/scripts/em_internal/index.py:214` `f"{self.db_path} is not a readable index: {exc}"`
- `sources/entropicmem/plugins/entropicmem/scripts/em_internal/index.py:219` `f"{self.db_path} is schema v{version}, this build reads "`
- `sources/entropicmem/plugins/entropicmem/scripts/em_internal/index.py:835` `"db_path": str(self.db_path),`

- `sources/entropicmem/plugins/entropicmem/scripts/em_internal/security.py:25` `from em.util.fsperm import chmod_posix`
- `sources/entropicmem/plugins/entropicmem/scripts/em_internal/security.py:64` `def _ciphertext_files(db_path: Path) -> list:`
- `sources/entropicmem/plugins/entropicmem/scripts/em_internal/security.py:70` `enc_db = db_path.with_suffix(db_path.suffix + ".enc")`
- `sources/entropicmem/plugins/entropicmem/scripts/em_internal/security.py:74` `enc_wal = db_path.parent / (db_path.name + suffix + ".enc")`
- `sources/entropicmem/plugins/entropicmem/scripts/em_internal/security.py:77` `vault_dir = db_path.parent / "vault"`
- `sources/entropicmem/plugins/entropicmem/scripts/em_internal/security.py:82` `def _validate_passphrase(fernet: "Fernet", db_path: Path) -> None:`
- `sources/entropicmem/plugins/entropicmem/scripts/em_internal/security.py:93` `marker = json.loads((db_path.parent / ENCRYPTED_MARKER).read_text())`
- `sources/entropicmem/plugins/entropicmem/scripts/em_internal/security.py:107` `probe = next(iter(_ciphertext_files(db_path)), None)`
- `sources/entropicmem/plugins/entropicmem/scripts/em_internal/security.py:115` `def is_encrypted(db_path: Path) -> bool:`
- `sources/entropicmem/plugins/entropicmem/scripts/em_internal/security.py:117` `return (db_path.parent / ENCRYPTED_MARKER).exists()`
- `sources/entropicmem/plugins/entropicmem/scripts/em_internal/security.py:129` `chmod_posix(temp_name, 0o600)`
- `sources/entropicmem/plugins/entropicmem/scripts/em_internal/security.py:152` `def _staging_dir(db_path: Path) -> Path:`
- `sources/entropicmem/plugins/entropicmem/scripts/em_internal/security.py:154` `parent = db_path.parent`
- `sources/entropicmem/plugins/entropicmem/scripts/em_internal/security.py:157` `chmod_posix(path, 0o700)`
- `sources/entropicmem/plugins/entropicmem/scripts/em_internal/security.py:161` `def _plaintext_files(db_path: Path) -> list:`
- `sources/entropicmem/plugins/entropicmem/scripts/em_internal/security.py:164` `if db_path.exists():`
- `sources/entropicmem/plugins/entropicmem/scripts/em_internal/security.py:165` `files.append(db_path)`
- `sources/entropicmem/plugins/entropicmem/scripts/em_internal/security.py:167` `companion = db_path.with_suffix(db_path.suffix + suffix)`
- `sources/entropicmem/plugins/entropicmem/scripts/em_internal/security.py:170` `vault_dir = db_path.parent / "vault"`
- `sources/entropicmem/plugins/entropicmem/scripts/em_internal/security.py:176` `def _remove_plaintext_remnants(db_path: Path) -> None:`
- `sources/entropicmem/plugins/entropicmem/scripts/em_internal/security.py:183` `for enc_path in _ciphertext_files(db_path):`
- `sources/entropicmem/plugins/entropicmem/scripts/em_internal/security.py:213` `def encrypt_db(db_path: Path, passphrase: str) -> dict:`
- `sources/entropicmem/plugins/entropicmem/scripts/em_internal/security.py:227` `if is_encrypted(db_path):`
- `sources/entropicmem/plugins/entropicmem/scripts/em_internal/security.py:228` `_remove_plaintext_remnants(db_path)`
- `sources/entropicmem/plugins/entropicmem/scripts/em_internal/security.py:233` `sources = _plaintext_files(db_path)`
- `sources/entropicmem/plugins/entropicmem/scripts/em_internal/security.py:235` `staging = _staging_dir(db_path)`
- `sources/entropicmem/plugins/entropicmem/scripts/em_internal/security.py:240` `relative = source.relative_to(db_path.parent)`
- `sources/entropicmem/plugins/entropicmem/scripts/em_internal/security.py:259` `os.replace(staged_salt, db_path.parent / SALT_FILE)`
- `sources/entropicmem/plugins/entropicmem/scripts/em_internal/security.py:260` `os.replace(staged_marker, db_path.parent / ENCRYPTED_MARKER)`
- `sources/entropicmem/plugins/entropicmem/scripts/em_internal/security.py:261` `_fsync_dir(db_path.parent)`
- `sources/entropicmem/plugins/entropicmem/scripts/em_internal/security.py:272` `def decrypt_db(db_path: Path, passphrase: str) -> dict:`
- `sources/entropicmem/plugins/entropicmem/scripts/em_internal/security.py:287` `if not is_encrypted(db_path):`
- `sources/entropicmem/plugins/entropicmem/scripts/em_internal/security.py:290` `salt_path = db_path.parent / SALT_FILE`
- `sources/entropicmem/plugins/entropicmem/scripts/em_internal/security.py:299` `_validate_passphrase(fernet, db_path)`
- `sources/entropicmem/plugins/entropicmem/scripts/em_internal/security.py:300` `ciphertexts = _ciphertext_files(db_path)`
- `sources/entropicmem/plugins/entropicmem/scripts/em_internal/security.py:302` `staging = _staging_dir(db_path)`
- `sources/entropicmem/plugins/entropicmem/scripts/em_internal/security.py:307` `relative = target.relative_to(db_path.parent)`
- `sources/entropicmem/plugins/entropicmem/scripts/em_internal/security.py:322` `(db_path.parent / ENCRYPTED_MARKER).unlink(missing_ok=True)`
- `sources/entropicmem/plugins/entropicmem/scripts/em_internal/security.py:324` `_fsync_dir(db_path.parent)`
- `sources/entropicmem/plugins/entropicmem/scripts/em_internal/security.py:336` `db_path: Path,`
- `sources/entropicmem/plugins/entropicmem/scripts/em_internal/security.py:351` `chmod_posix(path, mode)`
- `sources/entropicmem/plugins/entropicmem/scripts/em_internal/security.py:358` `_tighten(Path(dirpath), 0o700)`
- `sources/entropicmem/plugins/entropicmem/scripts/em_internal/security.py:361` `_tighten(Path(dirpath) / name, 0o600)`
- `sources/entropicmem/plugins/entropicmem/scripts/em_internal/security.py:364` `_tighten(Path(index_db), 0o600)`
- `sources/entropicmem/plugins/entropicmem/scripts/em_internal/security.py:365` `_tighten(db_path.parent / SALT_FILE, 0o600)`
- `sources/entropicmem/plugins/entropicmem/scripts/em_internal/security.py:366` `_tighten(db_path.parent / ENCRYPTED_MARKER, 0o600)`
- `sources/entropicmem/plugins/entropicmem/scripts/em_internal/security.py:367` `for enc_path in db_path.parent.glob("*.enc"):`
- `sources/entropicmem/plugins/entropicmem/scripts/em_internal/security.py:368` `_tighten(enc_path, 0o600)`
- `sources/entropicmem/plugins/entropicmem/scripts/em_internal/security.py:371` `_tighten(enc_path, 0o600)`
- `sources/entropicmem/plugins/entropicmem/scripts/em_internal/security.py:375` `def security_status(db_path: Path) -> dict:`
- `sources/entropicmem/plugins/entropicmem/scripts/em_internal/security.py:377` `encrypted = is_encrypted(db_path)`
- `sources/entropicmem/plugins/entropicmem/scripts/em_internal/security.py:383` `marker_path = db_path.parent / ENCRYPTED_MARKER`

- `sources/entropicmem/plugins/entropicmem/scripts/em_internal/vault.py:19` `from em.util.fsperm import chmod_posix`
- `sources/entropicmem/plugins/entropicmem/scripts/em_internal/vault.py:394` `chmod_posix(filepath.parent, 0o700)`
- `sources/entropicmem/plugins/entropicmem/scripts/em_internal/vault.py:413` `chmod_posix(filepath, 0o600)`
- `sources/entropicmem/plugins/entropicmem/scripts/em_internal/vault.py:624` `os.environ.get("EDITOR"),`
- `sources/entropicmem/plugins/entropicmem/scripts/em_internal/vault.py:625` `os.environ.get("VISUAL"),`
- `sources/entropicmem/plugins/entropicmem/scripts/em_internal/vault.py:632` `subprocess.run([editor, str(path)], check=False)`
- `sources/entropicmem/plugins/entropicmem/scripts/em_internal/vault.py:756` `env = os.environ.get("HERMES_HOME")`
- `sources/entropicmem/plugins/entropicmem/scripts/em_internal/vault.py:757` `base = Path(env) if env else Path.home() / ".hermes"`
- `sources/entropicmem/plugins/entropicmem/scripts/em_internal/vault.py:779` `''OBSIDIAN_VAULT_PATH'' directly from ''os.environ'' themselves.`
- `sources/entropicmem/plugins/entropicmem/scripts/em_internal/vault.py:784` `env_path = os.environ.get("ENTROPICMEM_VAULT_PATH")`

- `sources/entropicmem/plugins/entropicmem/scripts/entropicmem.py:178` `''http://[ip]/'' or a loopback port would reach it anyway (SSRF`
- `sources/entropicmem/plugins/entropicmem/scripts/entropicmem.py:205` `os.environ.get("ENTROPICMEM_INDEX_DB",`
- `sources/entropicmem/plugins/entropicmem/scripts/entropicmem.py:461` `path = db if db is not None else _memory_db_path()`
- `sources/entropicmem/plugins/entropicmem/scripts/entropicmem.py:502` `return store_kind(_memory_db_path()) == "v3"`
- `sources/entropicmem/plugins/entropicmem/scripts/entropicmem.py:516` `def _memory_db_path() -> Path:`
- `sources/entropicmem/plugins/entropicmem/scripts/entropicmem.py:517` `return Path(os.environ.get(`
- `sources/entropicmem/plugins/entropicmem/scripts/entropicmem.py:877` `db_path = _memory_db_path()`
- `sources/entropicmem/plugins/entropicmem/scripts/entropicmem.py:878` `if db_path.is_file():`
- `sources/entropicmem/plugins/entropicmem/scripts/entropicmem.py:888` `is_url = source.url.startswith(("http://", "https://"))`
- `sources/entropicmem/plugins/entropicmem/scripts/entropicmem.py:917` `if not db_path.is_file():`
- `sources/entropicmem/plugins/entropicmem/scripts/entropicmem.py:918` `print(f"  Memory: no store at {db_path} - source note only")`
- `sources/entropicmem/plugins/entropicmem/scripts/entropicmem.py:936` `if source.startswith(("http://", "https://")):`
- `sources/entropicmem/plugins/entropicmem/scripts/entropicmem.py:1130` `_memory_db_path(),`
- `sources/entropicmem/plugins/entropicmem/scripts/entropicmem.py:1193` `print(f"Serving graph at http://{shown}:{actual_port}/graph.html (Ctrl+C to stop)")`
- `sources/entropicmem/plugins/entropicmem/scripts/entropicmem.py:1206` `db_path = index_path  # reuse the index DB`
- `sources/entropicmem/plugins/entropicmem/scripts/entropicmem.py:1207` `conn = _sqlite.connect(str(db_path))`
- `sources/entropicmem/plugins/entropicmem/scripts/entropicmem.py:1263` `print(f"Memory DB: {s['db_path']}")`
- `sources/entropicmem/plugins/entropicmem/scripts/entropicmem.py:1673` `db_path = _memory_db_path()`
- `sources/entropicmem/plugins/entropicmem/scripts/entropicmem.py:1676` `status = security_status(db_path)`
- `sources/entropicmem/plugins/entropicmem/scripts/entropicmem.py:1690` `result = fix_permissions(db_path, vault_path, index_db)`
- `sources/entropicmem/plugins/entropicmem/scripts/entropicmem.py:1710` `result = encrypt_db(db_path, passphrase)`
- `sources/entropicmem/plugins/entropicmem/scripts/entropicmem.py:1722` `result = decrypt_db(db_path, passphrase)`
- `sources/entropicmem/plugins/entropicmem/scripts/entropicmem.py:1803` `db_path = _memory_db_path()`
- `sources/entropicmem/plugins/entropicmem/scripts/entropicmem.py:1807` `if not db_path.exists():`
- `sources/entropicmem/plugins/entropicmem/scripts/entropicmem.py:1808` `raise ValueError(f"memory DB not found at {db_path}")`
- `sources/entropicmem/plugins/entropicmem/scripts/entropicmem.py:1812` `if is_encrypted(db_path):`
- `sources/entropicmem/plugins/entropicmem/scripts/entropicmem.py:1818` `for arcname, src in (("memory.db", db_path), ("index.db", index_db)):`
- `sources/entropicmem/plugins/entropicmem/scripts/entropicmem.py:1838` `"db_path": str(db_path),`
- `sources/entropicmem/plugins/entropicmem/scripts/entropicmem.py:1866` `def _provider_running(db_path: Path) -> bool:`
- `sources/entropicmem/plugins/entropicmem/scripts/entropicmem.py:1873` `return FileLock.probe(db_path.parent / f"{db_path.name}.lock")`
- `sources/entropicmem/plugins/entropicmem/scripts/entropicmem.py:1893` `db_path = _memory_db_path()`
- `sources/entropicmem/plugins/entropicmem/scripts/entropicmem.py:1896` `lock = db_path.parent / MemoryEngine.MIGRATION_LOCK_FILENAME`
- `sources/entropicmem/plugins/entropicmem/scripts/entropicmem.py:1899` `if _provider_running(db_path):`
- `sources/entropicmem/plugins/entropicmem/scripts/entropicmem.py:1971` `if db_path.is_file():`
- `sources/entropicmem/plugins/entropicmem/scripts/entropicmem.py:1974` `BackupManager(db_path).snapshot(reason="pre-import")`
- `sources/entropicmem/plugins/entropicmem/scripts/entropicmem.py:1981` `target = db_path`
- `sources/entropicmem/plugins/entropicmem/scripts/entropicmem.py:2012` `db_path = _memory_db_path()`
- `sources/entropicmem/plugins/entropicmem/scripts/entropicmem.py:2015` `if db_path.exists():`
- `sources/entropicmem/plugins/entropicmem/scripts/entropicmem.py:2016` `print(f"Warning: existing DB at {db_path} will be replaced.")`
- `sources/entropicmem/plugins/entropicmem/scripts/entropicmem.py:2030` `print(f"Capsule imported to {db_path.parent}")`
- `sources/entropicmem/plugins/entropicmem/scripts/entropicmem.py:2107` `db = _memory_db_path()`
- `sources/entropicmem/plugins/entropicmem/scripts/entropicmem.py:2202` `db_path = _memory_db_path()`
- `sources/entropicmem/plugins/entropicmem/scripts/entropicmem.py:2203` `if not db_path.is_file():`
- `sources/entropicmem/plugins/entropicmem/scripts/entropicmem.py:2204` `checks.append(("WARN", "store", f"no store at {db_path} (created on first use)"))`
- `sources/entropicmem/plugins/entropicmem/scripts/entropicmem.py:2207` `kind = store_kind(db_path)`
- `sources/entropicmem/plugins/entropicmem/scripts/entropicmem.py:2208` `ro = _ro_connect(db_path)`
- `sources/entropicmem/plugins/entropicmem/scripts/entropicmem.py:2215` `detail = f"{kind} store at {db_path}, user_version={uv}, integrity={integrity}"`
- `sources/entropicmem/plugins/entropicmem/scripts/entropicmem.py:2220` `checks.append(("FAIL", "store", f"{db_path}: {type(exc).__name__}: {exc}"))`
- `sources/entropicmem/plugins/entropicmem/scripts/entropicmem.py:2367` `db_path = _memory_db_path()`
- `sources/entropicmem/plugins/entropicmem/scripts/entropicmem.py:2368` `if not db_path.is_file():`
- `sources/entropicmem/plugins/entropicmem/scripts/entropicmem.py:2369` `print(f"No store at {db_path} , nothing to upgrade (a fresh install creates a v3 store).")`
- `sources/entropicmem/plugins/entropicmem/scripts/entropicmem.py:2372` `kind = store_kind(db_path)`
- `sources/entropicmem/plugins/entropicmem/scripts/entropicmem.py:2374` `print(f"Store at {db_path} is already a v3 store , nothing to upgrade.")`
- `sources/entropicmem/plugins/entropicmem/scripts/entropicmem.py:2378` `ro = _ro_connect(db_path)`
- `sources/entropicmem/plugins/entropicmem/scripts/entropicmem.py:2391` `lock = db_path.parent / MemoryEngine.MIGRATION_LOCK_FILENAME`
- `sources/entropicmem/plugins/entropicmem/scripts/entropicmem.py:2395` `if _provider_running(db_path):`
- `sources/entropicmem/plugins/entropicmem/scripts/entropicmem.py:2407` `_saved_allow_live = os.environ.get("ENTROPICMEM_ALLOW_LIVE_MIGRATION")`
- `sources/entropicmem/plugins/entropicmem/scripts/entropicmem.py:2408` `os.environ["ENTROPICMEM_ALLOW_LIVE_MIGRATION"] = "1"`
- `sources/entropicmem/plugins/entropicmem/scripts/entropicmem.py:2411` `conn = open_db(db_path)`
- `sources/entropicmem/plugins/entropicmem/scripts/entropicmem.py:2417` `os.environ.pop("ENTROPICMEM_ALLOW_LIVE_MIGRATION", None)`
- `sources/entropicmem/plugins/entropicmem/scripts/entropicmem.py:2419` `os.environ["ENTROPICMEM_ALLOW_LIVE_MIGRATION"] = _saved_allow_live`
- `sources/entropicmem/plugins/entropicmem/scripts/entropicmem.py:2423` `ro = _ro_connect(db_path)`
- `sources/entropicmem/plugins/entropicmem/scripts/entropicmem.py:2429` `backup_dir = db_path.parent / "backups"`
- `sources/entropicmem/plugins/entropicmem/scripts/entropicmem.py:2433` `f"integrity={'ok' if (ro := _ro_connect(db_path)) and _integrity(ro) else 'FAILED'}")`
- `sources/entropicmem/plugins/entropicmem/scripts/entropicmem.py:2452` `db = _memory_db_path()`
- `sources/entropicmem/plugins/entropicmem/scripts/entropicmem.py:2542` `_memory_db_path(),`

- `sources/entropicmem/plugins/entropicmem/scripts/graph_static.py:34` `"style-src 'self' 'unsafe-inline' https://fonts.googleapis.com; "`
- `sources/entropicmem/plugins/entropicmem/scripts/graph_static.py:35` `"font-src https://fonts.gstatic.com; "`
- `sources/entropicmem/plugins/entropicmem/scripts/graph_static.py:66` `return os.environ.get("ENTROPICMEM_GRAPH_EXPOSE", "").strip() == "1"`
- `sources/entropicmem/plugins/entropicmem/scripts/graph_static.py:100` `extra = os.environ.get("ENTROPICMEM_GRAPH_ALLOWED_HOSTS") or ""`

- `sources/entropicmem/plugins/entropicmem/scripts/memory_engine.py:500` `def __init__(self, db_path: Path, profile_id: Optional[str] = None, publish_scope: Optional[str] = None, hermes_home: Optional[Path] = None, pii_locales: Optional[list] = None):`
- `sources/entropicmem/plugins/entropicmem/scripts/memory_engine.py:502` `self.db_path = Path(db_path).resolve()`
- `sources/entropicmem/plugins/entropicmem/scripts/memory_engine.py:503` `self.db_path.parent.mkdir(parents=True, exist_ok=True)`
- `sources/entropicmem/plugins/entropicmem/scripts/memory_engine.py:507` `publish_scope or os.environ.get("ENTROPICMEM_PUBLISH_SCOPE") or "shared"`
- `sources/entropicmem/plugins/entropicmem/scripts/memory_engine.py:509` `self.db = sqlite3.connect(str(self.db_path), timeout=30)`
- `sources/entropicmem/plugins/entropicmem/scripts/memory_engine.py:512` `os.chmod(self.db_path.parent, 0o700)`
- `sources/entropicmem/plugins/entropicmem/scripts/memory_engine.py:513` `if self.db_path.exists():`
- `sources/entropicmem/plugins/entropicmem/scripts/memory_engine.py:514` `os.chmod(self.db_path, 0o600)`
- `sources/entropicmem/plugins/entropicmem/scripts/memory_engine.py:525` `lock_path = self.db_path.parent / f"{self.db_path.name}.lock"`
- `sources/entropicmem/plugins/entropicmem/scripts/memory_engine.py:577` `if (self.db_path.parent / self.MIGRATION_LOCK_FILENAME).exists():`
- `sources/entropicmem/plugins/entropicmem/scripts/memory_engine.py:1124` `backup_dir = self.db_path.parent / "backups"`
- `sources/entropicmem/plugins/entropicmem/scripts/memory_engine.py:1151` `backup_dir = self.db_path.parent / "backups"`
- `sources/entropicmem/plugins/entropicmem/scripts/memory_engine.py:1156` `with sqlite3.connect(str(self.db_path)) as src, sqlite3.connect(str(backup_path)) as dst:`
- `sources/entropicmem/plugins/entropicmem/scripts/memory_engine.py:1426` `env = os.environ.get("ENTROPICMEM_SHARED_DB")`
- `sources/entropicmem/plugins/entropicmem/scripts/memory_engine.py:1429` `base = Path(hermes_home).expanduser() if hermes_home else Path.home() / ".hermes"`
- `sources/entropicmem/plugins/entropicmem/scripts/memory_engine.py:1827` `"db_path": str(self.db_path),`

- `sources/entropicmem/plugins/entropicmem/scripts/parity_audit.py:28` `return Path.home() / ".hermes"`

- `sources/entropicmem/plugins/entropicmem/scripts/triple_extract.py:90` `base = Path(home).expanduser() if home else Path.home() / ".hermes"`

</details>

## 5. Reproducibility

Bundled engine is pinned by repository SHA; manifest pip_dependencies=[] (`sources/entropicmem/plugins/entropicmem/plugin.yaml:14`). Optional extras have separate constraints. This avoids an unpinned mandatory engine download but does not establish model reproducibility.

Retained lock/checksum files: none in snapshot. Upstream completeness is unknown: fetch.py:15 excludes .lock files, fetch.py:12 excludes build/vendor directories. Absence here is not absence upstream.

- `sources/entropicmem/plugins/entropicmem/plugin.yaml:2` `version: 3.0.2`
- `sources/entropicmem/plugins/entropicmem/plugin.yaml:5` `# EM-806: the two manifest fields a host judges a **pip install** on. The floor`
- `sources/entropicmem/plugins/entropicmem/plugin.yaml:7` `# 'v2026.9.14' (version 0.21.3) and 'RecallStatus' predates it, so 0.21.3 is the`
- `sources/entropicmem/plugins/entropicmem/plugin.yaml:8` `# first release this provider runs on; 'version_satisfies' compares the first`
- `sources/entropicmem/plugins/entropicmem/plugin.yaml:9` `# three numeric segments. The engine is stdlib-only ('project.dependencies' is`
- `sources/entropicmem/plugins/entropicmem/plugin.yaml:10` `# empty), and the optional extras are opt-in installs, not requirements , a`
- `sources/entropicmem/plugins/entropicmem/plugin.yaml:11` `# catalog install is a directory copy with no 'pyproject.toml', which is why the`
- `sources/entropicmem/plugins/entropicmem/plugin.yaml:13` `requires_hermes: ">=0.21.3"`
- `sources/entropicmem/plugins/entropicmem/plugin.yaml:14` `pip_dependencies: []`

## 6. Tests

Provider tests exercise ID deletion, query preview/token expiration/consume and scope refusal; facade tests explicitly assert that forget leaves row/history. Purge tests are separate and are not evidence that the agent tool purges. Provider tests cover logical forget and query-confirmation handling; physical purge is separate. (`sources/entropicmem/tests/unit/test_em404_tool_behaviour.py:405`).

Adapter-related declarations: **112**. Python AST test_ functions, fixtures excluded; literal TS/JS it/test, Go Test*, Rust test_* declarations. No parametrization expansion, collection, execution, or coverage measurement. Other rows are engine test candidates selected by mutation names; names alone do not verify forget coverage.

| file | declarations | adapter-related | forget-related declarations |
|---|---:|---|---|
| `sources/entropicmem/tests/harness/test_f004_shared_profile.py:1` | 13 | no | `test_forget_by_id_refuses_another_user_s_row` `sources/entropicmem/tests/harness/test_f004_shared_profile.py:235`; `test_the_forget_confirm_flow_never_offers_another_user_s_row` `sources/entropicmem/tests/harness/test_f004_shared_profile.py:246` |
| `sources/entropicmem/tests/harness/test_provider_hook_surface.py:1` | 6 | yes | none |
| `sources/entropicmem/tests/parity/test_engine_parity.py:1` | 18 | no | `test_forget_requires_confirmation` `sources/entropicmem/tests/parity/test_engine_parity.py:160` |
| `sources/entropicmem/tests/test_engine_hardening.py:1` | 44 | no | `test_forget_keeps_lock_through_nested_backup` `sources/entropicmem/tests/test_engine_hardening.py:115`; `test_cache_invalidated_on_store` `sources/entropicmem/tests/test_engine_hardening.py:474`; `test_cache_invalidated_on_delete` `sources/entropicmem/tests/test_engine_hardening.py:481`; `test_reinforce_history_forget_validate_ids` `sources/entropicmem/tests/test_engine_hardening.py:533` |
| `sources/entropicmem/tests/test_memvid_phases.py:1` | 35 | no | `test_delete_embedding` `sources/entropicmem/tests/test_memvid_phases.py:252` |
| `sources/entropicmem/tests/test_mirror_lifecycle.py:1` | 8 | no | `test_remove_forgets_mirror` `sources/entropicmem/tests/test_mirror_lifecycle.py:63` |
| `sources/entropicmem/tests/test_phase2.py:1` | 21 | no | `test_remember_forget_roundtrip` `sources/entropicmem/tests/test_phase2.py:167` |
| `sources/entropicmem/tests/test_phase4.py:1` | 16 | no | `test_forget_removes_fact` `sources/entropicmem/tests/test_phase4.py:99`; `test_forget_returns_false_for_missing` `sources/entropicmem/tests/test_phase4.py:104`; `test_cli_remember_forget_roundtrip` `sources/entropicmem/tests/test_phase4.py:149` |
| `sources/entropicmem/tests/test_plugin_hardening.py:1` | 25 | yes | none |
| `sources/entropicmem/tests/test_plugin_hooks.py:1` | 12 | yes | none |
| `sources/entropicmem/tests/test_plugin_imports.py:1` | 4 | yes | none |
| `sources/entropicmem/tests/test_provider_contract.py:1` | 15 | yes | none |
| `sources/entropicmem/tests/test_security_phase2.py:1` | 14 | no | `test_forget_requires_confirm` `sources/entropicmem/tests/test_security_phase2.py:52`; `test_forget_removes_embedding_without_embedder` `sources/entropicmem/tests/test_security_phase2.py:90` |
| `sources/entropicmem/tests/test_sync_layer.py:1` | 9 | no | `test_delete_propagates_as_tombstone` `sources/entropicmem/tests/test_sync_layer.py:186` |
| `sources/entropicmem/tests/test_vault.py:1` | 40 | no | `test_delete_note` `sources/entropicmem/tests/test_vault.py:249`; `test_delete_note_from_index` `sources/entropicmem/tests/test_vault.py:390` |
| `sources/entropicmem/tests/unit/test_em303_prune.py:1` | 12 | no | `test_after_the_grace_only_models_beyond_current_and_previous_are_deleted` `sources/entropicmem/tests/unit/test_em303_prune.py:143`; `test_inside_the_grace_nothing_is_deleted` `sources/entropicmem/tests/unit/test_em303_prune.py:153`; `test_dry_run_reports_counts_and_deletes_nothing` `sources/entropicmem/tests/unit/test_em303_prune.py:195`; `test_vacuum_runs_only_after_the_delete_commits` `sources/entropicmem/tests/unit/test_em303_prune.py:220` |
| `sources/entropicmem/tests/unit/test_em404_tool_behaviour.py:1` | 50 | yes | `test_forget_by_id_marks_deleted` `sources/entropicmem/tests/unit/test_em404_tool_behaviour.py:407`; `test_forget_by_query_returns_candidates_and_token` `sources/entropicmem/tests/unit/test_em404_tool_behaviour.py:419`; `test_forget_tokens_differ_between_previews` `sources/entropicmem/tests/unit/test_em404_tool_behaviour.py:433`; `test_forget_apply_consumes_the_token` `sources/entropicmem/tests/unit/test_em404_tool_behaviour.py:442`; `test_forget_token_expires_after_five_minutes` `sources/entropicmem/tests/unit/test_em404_tool_behaviour.py:456`; `test_forget_preview_allowed_apply_blocked_in_non_primary` `sources/entropicmem/tests/unit/test_em404_tool_behaviour.py:471`; `test_forget_by_id_on_a_v2_store` `sources/entropicmem/tests/unit/test_em404_tool_behaviour.py:482`; `test_review_discard_deletes` `sources/entropicmem/tests/unit/test_em404_tool_behaviour.py:598` |
| `sources/entropicmem/tests/unit/test_em405_retract.py:1` | 11 | no | `test_retract_session_soft_deletes_only_live_stamped_rows` `sources/entropicmem/tests/unit/test_em405_retract.py:161` |
| `sources/entropicmem/tests/unit/test_em601_projection.py:1` | 42 | no | `test_forget_removes_the_line_from_the_entity_page` `sources/entropicmem/tests/unit/test_em601_projection.py:307`; `test_forget_clears_the_relation_that_memory_put_there` `sources/entropicmem/tests/unit/test_em601_projection.py:325`; `test_a_file_whose_markers_were_deleted_is_left_untouched_and_reported` `sources/entropicmem/tests/unit/test_em601_projection.py:377` |
| `sources/entropicmem/tests/unit/test_em705_purge.py:1` | 8 | no | `test_purge_removes_the_memory_embedding_rows` `sources/entropicmem/tests/unit/test_em705_purge.py:109`; `test_purge_leaves_episode_vectors_alone` `sources/entropicmem/tests/unit/test_em705_purge.py:124`; `test_no_vector_insert_for_a_purged_id_survives_in_the_iterdump` `sources/entropicmem/tests/unit/test_em705_purge.py:144`; `test_a_reembed_of_a_purged_id_writes_nothing` `sources/entropicmem/tests/unit/test_em705_purge.py:170`; `test_purge_works_without_a_callback` `sources/entropicmem/tests/unit/test_em705_purge.py:196`; `test_the_projection_callback_fires_exactly_once_with_the_purged_id` `sources/entropicmem/tests/unit/test_em705_purge.py:207`; `test_purge_then_one_reprojection_leaves_no_vault_file_with_the_text` `sources/entropicmem/tests/unit/test_em705_purge.py:247` |
| `sources/entropicmem/tests/unit/test_em_audit.py:1` | 20 | no | `test_deleted_row_breaks_the_chain` `sources/entropicmem/tests/unit/test_em_audit.py:123`; `test_rechained_deletion_is_a_known_limitation` `sources/entropicmem/tests/unit/test_em_audit.py:136` |
| `sources/entropicmem/tests/unit/test_em_cli_maintenance.py:1` | 15 | no | `test_discard_pending_returns_true_and_deletes` `sources/entropicmem/tests/unit/test_em_cli_maintenance.py:91` |
| `sources/entropicmem/tests/unit/test_em_embedding_cache.py:1` | 9 | no | `test_a_deleted_row_invalidates_the_cache` `sources/entropicmem/tests/unit/test_em_embedding_cache.py:129` |
| `sources/entropicmem/tests/unit/test_em_entities.py:1` | 43 | no | `test_entity_card_excludes_deleted_memories` `sources/entropicmem/tests/unit/test_em_entities.py:351` |
| `sources/entropicmem/tests/unit/test_em_episodes.py:1` | 44 | no | `test_prune_deletes_a_pending_sessions_chunks_when_not_keeping` `sources/entropicmem/tests/unit/test_em_episodes.py:364` |
| `sources/entropicmem/tests/unit/test_em_facade_engine.py:1` | 22 | no | `test_get_fact_hides_deleted_rows` `sources/entropicmem/tests/unit/test_em_facade_engine.py:150` |
| `sources/entropicmem/tests/unit/test_em_facade_writes.py:1` | 58 | no | `test_forget_without_confirm_raises_and_deletes_nothing` `sources/entropicmem/tests/unit/test_em_facade_writes.py:245`; `test_forget_confirmed_returns_true_then_false` `sources/entropicmem/tests/unit/test_em_facade_writes.py:253`; `test_forget_of_an_unknown_id_returns_false` `sources/entropicmem/tests/unit/test_em_facade_writes.py:260`; `test_forget_hides_the_row_from_get_fact_and_stats` `sources/entropicmem/tests/unit/test_em_facade_writes.py:264`; `test_forget_leaves_the_row_and_its_version_history` `sources/entropicmem/tests/unit/test_em_facade_writes.py:274`; `test_forget_writes_audit_rows_for_the_refusal_and_the_take` `sources/entropicmem/tests/unit/test_em_facade_writes.py:289`; `test_repeated_forgets_write_at_most_one_snapshot_per_hour` `sources/entropicmem/tests/unit/test_em_facade_writes.py:301`; `test_prune_pending_deletes_only_pending_rows_past_the_cutoff` `sources/entropicmem/tests/unit/test_em_facade_writes.py:451` |
| `sources/entropicmem/tests/unit/test_em_memories.py:1` | 54 | no | `test_update_of_deleted_memory_is_refused` `sources/entropicmem/tests/unit/test_em_memories.py:201`; `test_supersede_of_a_deleted_memory_is_refused` `sources/entropicmem/tests/unit/test_em_memories.py:241`; `test_purge_removes_the_row_and_its_versions` `sources/entropicmem/tests/unit/test_em_memories.py:345`; `test_purge_of_a_missing_row_raises` `sources/entropicmem/tests/unit/test_em_memories.py:355`; `test_purge_drops_the_row_from_fts` `sources/entropicmem/tests/unit/test_em_memories.py:521`; `test_soft_delete_keeps_the_row_in_fts` `sources/entropicmem/tests/unit/test_em_memories.py:529`; `test_purge_forgets_entity_sightings_from_the_whole_db` `sources/entropicmem/tests/unit/test_em_memories.py:591`; `test_promotion_deletes_the_sighting_row` `sources/entropicmem/tests/unit/test_em_memories.py:624`; `test_purge_removes_a_sighting_row_that_becomes_empty` `sources/entropicmem/tests/unit/test_em_memories.py:643` |
| `sources/entropicmem/tests/unit/test_em_migration_0003.py:1` | 5 | no | `test_fresh_install_refuses_audit_update_and_delete` `sources/entropicmem/tests/unit/test_em_migration_0003.py:44`; `test_purge_token_still_allows_a_sanctioned_delete` `sources/entropicmem/tests/unit/test_em_migration_0003.py:88` |
| `sources/entropicmem/tests/unit/test_em_migration_v3_core.py:1` | 66 | no | `test_audit_purge_token_allows_deletion` `sources/entropicmem/tests/unit/test_em_migration_v3_core.py:868` |
| `sources/entropicmem/tests/unit/test_em_prefetch_service.py:1` | 23 | no | `test_an_in_process_write_invalidates_the_cache` `sources/entropicmem/tests/unit/test_em_prefetch_service.py:182`; `test_a_commit_by_another_connection_invalidates_the_cache` `sources/entropicmem/tests/unit/test_em_prefetch_service.py:195` |
| `sources/entropicmem/tests/unit/test_provider_shadow.py:1` | 37 | no | `test_the_fabrication_verdict_fires_on_a_deleted_row` `sources/entropicmem/tests/unit/test_provider_shadow.py:587` |

## 7. Corrections to B2

Old values below are copied from the triage row (TRIAGE.md), not reconstructed from memory. A refinement narrows an over-broad field; it need not reverse its original value. Counts are distinct field paths, not the number of factual errors.

- `forget_tool.scope`: `"Inherited provider implementation supplies tools and hooks; forget has confirmation handling."` → ID path applies logical deletion directly; query path requires confirmation token. v3 preserves row/history; facade can create a pre-forget backup. Evidence: `sources/entropicmem/plugins/entropicmem/scripts/em/provider/tools.py:1177`, `sources/entropicmem/plugins/entropicmem/scripts/em/facade/engine.py:825`.
- `engine_delete.scope`: `"Named deletion/retraction endpoint or method, not a guarantee of physical erasure. Its reach may overlap an agent tool; lack of a native tool is counted separately."` → `yes`: entropicmem_forget by ID calls _forget_one immediately; query previews candidates and confirmation-token apply is separate (`sources/entropicmem/plugins/entropicmem/scripts/em/provider/tools.py:1177`; `sources/entropicmem/plugins/entropicmem/scripts/em/provider/tools.py:1186`). v3 _forget_one checks scoped row, sets status deleted and enqueues projection (`sources/entropicmem/plugins/entropicmem/scripts/em/provider/tools.py:1264`; `sources/entropicmem/plugins/entropicmem/scripts/em/provider/tools.py:1274`). It does not physically delete the row. Facade forget requires confirm=True, can snapshot before changing status, and preserves history (`sources/entropicmem/plugins/entropicmem/scripts/em/facade/engine.py:825`; `sources/entropicmem/plugins/entropicmem/scripts/em/facade/engine.py:828`). Engine purge is a separate physical operation in the ledger. Derived reach `partial`: visibility/projection changes do not erase versions, audit, backups or host copies. Evidence: `sources/entropicmem/plugins/entropicmem/scripts/em/provider/tools.py:1177`, `sources/entropicmem/plugins/entropicmem/scripts/em/provider/tools.py:1186`, `sources/entropicmem/plugins/entropicmem/scripts/em/provider/tools.py:1264`, `sources/entropicmem/plugins/entropicmem/scripts/em/provider/tools.py:1274`, `sources/entropicmem/plugins/entropicmem/scripts/em/facade/engine.py:825`, `sources/entropicmem/plugins/entropicmem/scripts/em/facade/engine.py:828`.
- `context.mechanism.v`: `"Source expressions below identify retrieval, cached/static returns, lifecycle capture, and limits. A hook definition is not a guarantee it emits nonempty context each turn; branches and configuration apply."` → System block includes base prompt and optional core memory (`sources/entropicmem/plugins/entropicmem/scripts/em/provider/provider.py:650`). Prefetch combines service-selected fact context and optional conversation context; the fact packing budget alone is not total output size (`sources/entropicmem/plugins/entropicmem/scripts/em/provider/provider.py:755`; `sources/entropicmem/plugins/entropicmem/scripts/em/provider/provider.py:792`). Actual packing/deadline/cache/limit quotes below. Tools return JSON decisions/results, not a universal text memory block. Evidence: `sources/entropicmem/plugins/entropicmem/scripts/em/provider/provider.py:650`, `sources/entropicmem/plugins/entropicmem/scripts/em/provider/provider.py:755`, `sources/entropicmem/plugins/entropicmem/scripts/em/provider/provider.py:792`.
- `sec.telemetry.v`: `"inspect listed emitters/configuration"` → Direct local SQLite/vault/projection access `yes` under same UID. Accountless local mode `yes`: bundled stdlib engine has no mandatory dependency/key (`sources/entropicmem/plugins/entropicmem/plugin.yaml:14`). Optional embeddings/extraction can call configured hosts; local-only core must not be generalized to all modes. File modes/env paths and opt-in network flags are listed below; no core remote analytics emitter established. Evidence: `sources/entropicmem/plugins/entropicmem/plugin.yaml:14`.
- `repro.decls`: `["plugins/entropicmem/plugin.yaml:9", "plugins/entropicmem/plugin.yaml:14", "plugins/entropicmem/plugin.yaml:28", "plugins/entropicmem/plugin.yaml:29", "plugins/entropicmem/plugin.yaml:30", "plugins/entropicmem/plugin.yaml:31", "plugins/entropicmem/plugin.yaml:32", "plugins/entropicmem/plugin.yaml:33", "plugins/entropicmem/plugin.yaml:34", "plugins/entropicmem/plugin.yaml:35", "plugins/entropicmem/plugin.yaml:36", "plugins/entropicmem/plugin.yaml:38", "plugins/entropicmem/plugin.yaml:39", "plugins/entropicmem/plugin.yaml:40", "plugins/entropicmem/plugin.yaml:41", "plugins/entropicmem/plugin.yaml:42", "plugins/entropicmem/plugin.yaml:43", "pyproject.toml:2", "pyproject.toml:7", "pyproject.toml:11", "pyproject.toml:29", "pyproject.toml:31", "pyproject.toml:36", "pyproject.toml:41", "pyproject.toml:42", "pyproject.toml:43", "pyproject.toml:45", "pyproject.toml:52", "pyproject.toml:53", "pyproject.toml:54", "pyproject.toml:66", "pyproject.toml:67", "pyproject.toml:68", "pyproject.toml:69", "pyproject.toml:119", "pyproject.toml:129"]` → Bundled engine is pinned by repository SHA; manifest pip_dependencies=[] (`sources/entropicmem/plugins/entropicmem/plugin.yaml:14`). Optional extras have separate constraints. This avoids an unpinned mandatory engine download but does not establish model reproducibility. Evidence: `sources/entropicmem/plugins/entropicmem/plugin.yaml:14`.
- `tests.count`: `2915` → 112 adapter-related declarations using the stated parser/scope; B2 counted a different, repository-wide inventory. This is a scope refinement, not a count of executed or passing tests. Provider tests exercise ID deletion, query preview/token expiration/consume and scope refusal; facade tests explicitly assert that forget leaves row/history. Purge tests are separate and are not evidence that the agent tool purges. Provider tests cover logical forget and query-confirmation handling; physical purge is separate. (`sources/entropicmem/tests/unit/test_em404_tool_behaviour.py:405`). Evidence: `sources/entropicmem/tests/unit/test_em404_tool_behaviour.py:405`.

MATRIX counts these 6 explicit field corrections/refinements. Sections 1-6 provide the body and test inventories. Test-name candidates are not passing tests or measured coverage.
