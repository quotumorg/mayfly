# sibyl: B3 static correctness review

Snapshot `8d15cd20b37ce79f39953f684b88b17b2f8e753e` (MANIFEST.json row `sibyl`). No plugin execution or passing-test claims.

## 1. Architecture

`provider`: native catalogue adapter imports sibyl-memory-hermes, which uses the local SQLite client in-process. Tenant/profile scope and account entitlement are separate from local storage (`sources/sibyl/sibyl-memory-hermes/hermes-catalog/__init__.py:667`; `sources/sibyl/sibyl-memory-hermes/hermes-catalog/__init__.py:689`).

Hook/body inventory (no-op/fallback declarations are not active behavior):

| callable | declaration | first executable body line |
|---|---|---|
| `register` | `sources/sibyl/sibyl-memory-hermes/hermes-catalog/__init__.py:1233` | `sources/sibyl/sibyl-memory-hermes/hermes-catalog/__init__.py:1235` |
| `is_available` | `sources/sibyl/sibyl-memory-hermes/hermes-catalog/__init__.py:649` | `sources/sibyl/sibyl-memory-hermes/hermes-catalog/__init__.py:651` |
| `initialize` | `sources/sibyl/sibyl-memory-hermes/hermes-catalog/__init__.py:657` | `sources/sibyl/sibyl-memory-hermes/hermes-catalog/__init__.py:658` |
| `get_tool_schemas` | `sources/sibyl/sibyl-memory-hermes/hermes-catalog/__init__.py:747` | `sources/sibyl/sibyl-memory-hermes/hermes-catalog/__init__.py:748` |
| `system_prompt_block` | `sources/sibyl/sibyl-memory-hermes/hermes-catalog/__init__.py:752` | `sources/sibyl/sibyl-memory-hermes/hermes-catalog/__init__.py:753` |
| `prefetch` | `sources/sibyl/sibyl-memory-hermes/hermes-catalog/__init__.py:787` | `sources/sibyl/sibyl-memory-hermes/hermes-catalog/__init__.py:788` |
| `queue_prefetch` | `sources/sibyl/sibyl-memory-hermes/hermes-catalog/__init__.py:883` | `sources/sibyl/sibyl-memory-hermes/hermes-catalog/__init__.py:886` |
| `sync_turn` | `sources/sibyl/sibyl-memory-hermes/hermes-catalog/__init__.py:888` | `sources/sibyl/sibyl-memory-hermes/hermes-catalog/__init__.py:899` |
| `handle_tool_call` | `sources/sibyl/sibyl-memory-hermes/hermes-catalog/__init__.py:953` | `sources/sibyl/sibyl-memory-hermes/hermes-catalog/__init__.py:954` |
| `shutdown` | `sources/sibyl/sibyl-memory-hermes/hermes-catalog/__init__.py:1071` | `sources/sibyl/sibyl-memory-hermes/hermes-catalog/__init__.py:1075` |
| `on_session_switch` | `sources/sibyl/sibyl-memory-hermes/hermes-catalog/__init__.py:1103` | `sources/sibyl/sibyl-memory-hermes/hermes-catalog/__init__.py:1108` |
| `on_pre_compress` | `sources/sibyl/sibyl-memory-hermes/hermes-catalog/__init__.py:1110` | `sources/sibyl/sibyl-memory-hermes/hermes-catalog/__init__.py:1112` |
| `on_delegation` | `sources/sibyl/sibyl-memory-hermes/hermes-catalog/__init__.py:1161` | `sources/sibyl/sibyl-memory-hermes/hermes-catalog/__init__.py:1163` |
| `on_memory_write` | `sources/sibyl/sibyl-memory-hermes/hermes-catalog/__init__.py:1177` | `sources/sibyl/sibyl-memory-hermes/hermes-catalog/__init__.py:1184` |
| `get_config_schema` | `sources/sibyl/sibyl-memory-hermes/hermes-catalog/__init__.py:1204` | `sources/sibyl/sibyl-memory-hermes/hermes-catalog/__init__.py:1220` |
| `save_config` | `sources/sibyl/sibyl-memory-hermes/hermes-catalog/__init__.py:1222` | `sources/sibyl/sibyl-memory-hermes/hermes-catalog/__init__.py:1226` |
| `register` | `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/_hermes_plugin/adapter.py:1233` | `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/_hermes_plugin/adapter.py:1235` |
| `is_available` | `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/_hermes_plugin/adapter.py:649` | `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/_hermes_plugin/adapter.py:651` |
| `initialize` | `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/_hermes_plugin/adapter.py:657` | `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/_hermes_plugin/adapter.py:658` |
| `get_tool_schemas` | `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/_hermes_plugin/adapter.py:747` | `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/_hermes_plugin/adapter.py:748` |
| `system_prompt_block` | `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/_hermes_plugin/adapter.py:752` | `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/_hermes_plugin/adapter.py:753` |
| `prefetch` | `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/_hermes_plugin/adapter.py:787` | `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/_hermes_plugin/adapter.py:788` |
| `queue_prefetch` | `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/_hermes_plugin/adapter.py:883` | `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/_hermes_plugin/adapter.py:886` |
| `sync_turn` | `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/_hermes_plugin/adapter.py:888` | `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/_hermes_plugin/adapter.py:899` |
| `handle_tool_call` | `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/_hermes_plugin/adapter.py:953` | `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/_hermes_plugin/adapter.py:954` |
| `shutdown` | `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/_hermes_plugin/adapter.py:1071` | `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/_hermes_plugin/adapter.py:1075` |
| `on_session_switch` | `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/_hermes_plugin/adapter.py:1103` | `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/_hermes_plugin/adapter.py:1108` |
| `on_pre_compress` | `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/_hermes_plugin/adapter.py:1110` | `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/_hermes_plugin/adapter.py:1112` |
| `on_delegation` | `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/_hermes_plugin/adapter.py:1161` | `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/_hermes_plugin/adapter.py:1163` |
| `on_memory_write` | `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/_hermes_plugin/adapter.py:1177` | `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/_hermes_plugin/adapter.py:1184` |
| `get_config_schema` | `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/_hermes_plugin/adapter.py:1204` | `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/_hermes_plugin/adapter.py:1220` |
| `save_config` | `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/_hermes_plugin/adapter.py:1222` | `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/_hermes_plugin/adapter.py:1226` |

## 2. Forget paths and derived copies

`update_only`: get_tool_schemas returns remember, recall, search and list, no forget/delete tool (`sources/sibyl/sibyl-memory-hermes/hermes-catalog/__init__.py:748`). Upsert through remember can replace a structured record; it is not erasure. Engine `yes`: `delete_entity` executes DELETE by tenant/category/name and cascades incident links in the same transaction (`sources/sibyl/sibyl-memory-client/src/sibyl_memory_client/client.py:1541`); `delete_reference`/`delete_reference_details` remove reference data, while `archive_entity` preserves history (`sources/sibyl/sibyl-memory-client/src/sibyl_memory_client/client.py:1823`; `sources/sibyl/sibyl-memory-client/src/sibyl_memory_client/client.py:2004`). `partial`: targeted graph-link cleanup is visible; journal, versions, other tiers and host copies are separate. No general whole-history wipe is implied.

Host state.db is independent (`hermes-agent/hermes_state.py:181`). No reviewed adapter forget body performs a general purge of that database or built-in MEMORY.md/USER.md. Prompt/tool-result copies and independent source transcripts remain separate; source operation bodies below establish the scope of each plugin mutation.

<details>
<summary>Deletion, invalidation, archive and reset body ledger</summary>

Public routes, SDK/CLI dispatch and internal storage operations are distinguished by their source. Cache invalidation and management-object deletion are not additional fact-forget tools. A stub is not working deletion. Call-through means the downstream body still determines erasure. HTTP prefixes/selectors must be read with route registration.

**invalidate_tier_cache**, body `sources/sibyl/sibyl-memory-cli/src/sibyl_memory_cli/cli.py:559`; implementation/call-through.

- `sources/sibyl/sibyl-memory-cli/src/sibyl_memory_cli/cli.py:557` `def invalidate_tier_cache(path: Path = DEFAULT_TIER_CACHE_PATH) -> None:`
- `sources/sibyl/sibyl-memory-cli/src/sibyl_memory_cli/cli.py:561` `path.unlink()`

**purge_test_rows**, body `sources/sibyl/sibyl-memory-cli/src/sibyl_memory_cli/doctor.py:162`; implementation/call-through.

- `sources/sibyl/sibyl-memory-cli/src/sibyl_memory_cli/doctor.py:159` `def purge_test_rows(db_path: Path, name: str) -> int:`
- `sources/sibyl/sibyl-memory-cli/src/sibyl_memory_cli/doctor.py:166` `for table in ("entities", "archived_entities"):`
- `sources/sibyl/sibyl-memory-cli/src/sibyl_memory_cli/doctor.py:168` `conn.execute(f"DELETE FROM {table} WHERE category = ? AND name = ?",`
- `sources/sibyl/sibyl-memory-cli/src/sibyl_memory_cli/doctor.py:174` `for table in ("entities", "archived_entities"):`

**clear**, body `sources/sibyl/sibyl-memory-client/src/sibyl_memory_client/_capcheck.py:269`; implementation/call-through.

- `sources/sibyl/sibyl-memory-client/src/sibyl_memory_client/_capcheck.py:268` `def clear(self) -> None:`
- `sources/sibyl/sibyl-memory-client/src/sibyl_memory_client/_capcheck.py:270` `self.path.unlink()`

**invalidate_cache**, body `sources/sibyl/sibyl-memory-client/src/sibyl_memory_client/_capcheck.py:997`; implementation/call-through.

- `sources/sibyl/sibyl-memory-client/src/sibyl_memory_client/_capcheck.py:995` `def invalidate_cache(self) -> None:`
- `sources/sibyl/sibyl-memory-client/src/sibyl_memory_client/_capcheck.py:997` `self._cache.clear()`

**delete**, body `sources/sibyl/sibyl-memory-client/src/sibyl_memory_client/_entitlement.py:532`; implementation/call-through.

- `sources/sibyl/sibyl-memory-client/src/sibyl_memory_client/_entitlement.py:531` `def delete(self) -> bool:`
- `sources/sibyl/sibyl-memory-client/src/sibyl_memory_client/_entitlement.py:534` `self.path.unlink()`

**delete_edges**, body `sources/sibyl/sibyl-memory-client/src/sibyl_memory_client/_links.py:576`; implementation/call-through.

- `sources/sibyl/sibyl-memory-client/src/sibyl_memory_client/_links.py:574` `def delete_edges(conn: sqlite3.Connection, tenant: str, src: tuple[str, str, str],`
- `sources/sibyl/sibyl-memory-client/src/sibyl_memory_client/_links.py:576` `sql = ("DELETE FROM memory_links WHERE tenant_id = ? AND from_tier = ? AND from_category = ? "`

**delete_entity**, body `sources/sibyl/sibyl-memory-client/src/sibyl_memory_client/client.py:1540`; implementation/call-through.

- `sources/sibyl/sibyl-memory-client/src/sibyl_memory_client/client.py:1536` `@_track_op("delete_entity")`
- `sources/sibyl/sibyl-memory-client/src/sibyl_memory_client/client.py:1537` `def delete_entity(self, category: str, name: str) -> bool:`
- `sources/sibyl/sibyl-memory-client/src/sibyl_memory_client/client.py:1542` `"DELETE FROM entities WHERE tenant_id = ? AND category = ? AND name = ?",`

**delete_reference**, body `sources/sibyl/sibyl-memory-client/src/sibyl_memory_client/client.py:1835`; implementation/call-through.

- `sources/sibyl/sibyl-memory-client/src/sibyl_memory_client/client.py:1822` `@_track_op("delete_reference")`
- `sources/sibyl/sibyl-memory-client/src/sibyl_memory_client/client.py:1823` `def delete_reference(self, key: str) -> bool:`
- `sources/sibyl/sibyl-memory-client/src/sibyl_memory_client/client.py:1835` `return self._delete_reference(key)["deleted"]`

**delete_reference_details**, body `sources/sibyl/sibyl-memory-client/src/sibyl_memory_client/client.py:1841`; implementation/call-through.

- `sources/sibyl/sibyl-memory-client/src/sibyl_memory_client/client.py:1837` `@_track_op("delete_reference")`
- `sources/sibyl/sibyl-memory-client/src/sibyl_memory_client/client.py:1838` `def delete_reference_details(self, key: str) -> dict[str, Any]:`
- `sources/sibyl/sibyl-memory-client/src/sibyl_memory_client/client.py:1841` `return self._delete_reference(key)`

**_delete_reference**, body `sources/sibyl/sibyl-memory-client/src/sibyl_memory_client/client.py:1844`; implementation/call-through.

- `sources/sibyl/sibyl-memory-client/src/sibyl_memory_client/client.py:1843` `def _delete_reference(self, key: str) -> dict[str, Any]:`
- `sources/sibyl/sibyl-memory-client/src/sibyl_memory_client/client.py:1848` `"DELETE FROM reference_documents WHERE tenant_id = ? AND doc_key = ?",`

**archive_entity**, body `sources/sibyl/sibyl-memory-client/src/sibyl_memory_client/client.py:2024`; implementation/call-through.

- `sources/sibyl/sibyl-memory-client/src/sibyl_memory_client/client.py:2003` `@_track_op("archive_entity")`
- `sources/sibyl/sibyl-memory-client/src/sibyl_memory_client/client.py:2004` `def archive_entity(self, category: str, name: str, reason: str | None = None) -> dict[str, Any]:`
- `sources/sibyl/sibyl-memory-client/src/sibyl_memory_client/client.py:2032` `# The archive insert copies the body. Delta = body + name + category`
- `sources/sibyl/sibyl-memory-client/src/sibyl_memory_client/client.py:2040` `"INSERT INTO archived_entities (id, tenant_id, original_entity_id, category, name, body, archive_reason) "`
- `sources/sibyl/sibyl-memory-client/src/sibyl_memory_client/client.py:2044` `conn.execute("DELETE FROM entities WHERE id = ?", (row["id"],))`
- `sources/sibyl/sibyl-memory-client/src/sibyl_memory_client/client.py:2045` `return {"archived_id": arch_id, "original_id": row["id"]}`

**clear**, body `sources/sibyl/sibyl-memory-client/src/sibyl_memory_client/lifecycle/signals.py:383`; implementation/call-through.

- `sources/sibyl/sibyl-memory-client/src/sibyl_memory_client/lifecycle/signals.py:380` `def clear(home: Path | str | None = None, *, now: float | None = None) -> bool:`
- `sources/sibyl/sibyl-memory-client/src/sibyl_memory_client/lifecycle/signals.py:393` `os.unlink(p)`

**reset**, body `sources/sibyl/sibyl-memory-client/src/sibyl_memory_client/skills.py:327`; implementation/call-through.

- `sources/sibyl/sibyl-memory-client/src/sibyl_memory_client/skills.py:325` `def reset(home: Path | str, name: str, *, now: float | None = None) -> InstallReport:`
- `sources/sibyl/sibyl-memory-client/src/sibyl_memory_client/skills.py:327` `if name not in SKILL_NAMES:`

**forget**, body `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/provider.py:316`; implementation/call-through.

- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/provider.py:306` `def forget(self, category: str, name: str) -> bool:`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/provider.py:316` `return self._client.delete_entity(category, name)`

**archive**, body `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/provider.py:338`; implementation/call-through.

- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/provider.py:318` `def archive(`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/provider.py:338` `return self._client.archive_entity(category, name, reason=reason)`

**memory_forget**, body `sources/sibyl/sibyl-memory-mcp/src/sibyl_memory_mcp/server.py:1533`; implementation/call-through.

- `sources/sibyl/sibyl-memory-mcp/src/sibyl_memory_mcp/server.py:1521` `@mcp.tool()`
- `sources/sibyl/sibyl-memory-mcp/src/sibyl_memory_mcp/server.py:1522` `@_recorded`
- `sources/sibyl/sibyl-memory-mcp/src/sibyl_memory_mcp/server.py:1523` `def memory_forget(category: str, name: str, reason: str | None = None) -> dict[str, Any]:`
- `sources/sibyl/sibyl-memory-mcp/src/sibyl_memory_mcp/server.py:1535` `client.archive_entity(category, name, reason=reason)`
- `sources/sibyl/sibyl-memory-mcp/src/sibyl_memory_mcp/server.py:1536` `return _with_notice(client, {"ok": True, "archived": {"category": category, "name": name}})`

**memory_delete_reference**, body `sources/sibyl/sibyl-memory-mcp/src/sibyl_memory_mcp/server.py:1976`; implementation/call-through.

- `sources/sibyl/sibyl-memory-mcp/src/sibyl_memory_mcp/server.py:1960` `@mcp.tool()`
- `sources/sibyl/sibyl-memory-mcp/src/sibyl_memory_mcp/server.py:1961` `@_recorded`
- `sources/sibyl/sibyl-memory-mcp/src/sibyl_memory_mcp/server.py:1962` `def memory_delete_reference(key: str, force: bool = False) -> dict[str, Any]:`
- `sources/sibyl/sibyl-memory-mcp/src/sibyl_memory_mcp/server.py:1986` `res = client.delete_reference_details(key)`

</details>

## 3. What reaches the agent

System prompt describes four tiers and four tools (`sources/sibyl/sibyl-memory-hermes/hermes-catalog/__init__.py:753`). Prefetch searches at most five hits, limits each body to 400 characters, frames untrusted memory with a random nonce and budgets the block to 6000 characters; an optional notice is appended outside that block (`sources/sibyl/sibyl-memory-hermes/hermes-catalog/__init__.py:852`; `sources/sibyl/sibyl-memory-hermes/hermes-catalog/__init__.py:878`; `sources/sibyl/sibyl-memory-hermes/hermes-catalog/__init__.py:795`). Thus 6000 is not an unconditional whole-return cap. Tool results have separate per-hit/output budgets below.

Host assembly: `hermes-agent/agent/memory_manager.py:352` joins provider system blocks. `hermes-agent/agent/turn_context.py:1081` requests prefetch for nontrivial turns; `hermes-agent/agent/turn_context.py:120` fences it; `hermes-agent/agent/turn_context.py:136` appends it to API-bound user content. Tool results are a third channel. Optional host spill is at `hermes-agent/agent/memory_manager.py:419`, not a universal provider cap. Built-in memory remains separately controlled (`hermes-agent/agent/system_prompt.py:542`).

<details>
<summary>Context and tool-result assembly quotes</summary>

`_handoff_tool_error`, body `sources/sibyl/sibyl-memory-hermes/hermes-catalog/__init__.py:180`:

- `sources/sibyl/sibyl-memory-hermes/hermes-catalog/__init__.py:180` `return _with_handoff(tool_error(msg), msg, nxt, human)`

`_entitlement_tool_error`, body `sources/sibyl/sibyl-memory-hermes/hermes-catalog/__init__.py:195`:

- `sources/sibyl/sibyl-memory-hermes/hermes-catalog/__init__.py:196` `return _handoff_tool_error(_SIGNED_OUT_TOOL_ERROR, "sibyl init", None)`
- `sources/sibyl/sibyl-memory-hermes/hermes-catalog/__init__.py:198` `return _handoff_tool_error(getattr(exc, "summary", None) or str(exc), nxt, human)`

`get_tool_schemas`, body `sources/sibyl/sibyl-memory-hermes/hermes-catalog/__init__.py:748`:

- `sources/sibyl/sibyl-memory-hermes/hermes-catalog/__init__.py:748` `return [REMEMBER_SCHEMA, RECALL_SCHEMA, SEARCH_SCHEMA, LIST_SCHEMA]`

`system_prompt_block`, body `sources/sibyl/sibyl-memory-hermes/hermes-catalog/__init__.py:753`:

- `sources/sibyl/sibyl-memory-hermes/hermes-catalog/__init__.py:753` `return (`

`prefetch`, body `sources/sibyl/sibyl-memory-hermes/hermes-catalog/__init__.py:788`:

- `sources/sibyl/sibyl-memory-hermes/hermes-catalog/__init__.py:789` `return ""`
- `sources/sibyl/sibyl-memory-hermes/hermes-catalog/__init__.py:795` `return (block + "\n\n" if block else "") + "## Sibyl Memory: note\n" + line`
- `sources/sibyl/sibyl-memory-hermes/hermes-catalog/__init__.py:796` `return block`

`_prefetch_block`, body `sources/sibyl/sibyl-memory-hermes/hermes-catalog/__init__.py:804`:

- `sources/sibyl/sibyl-memory-hermes/hermes-catalog/__init__.py:804` `clean = query.strip()[:1000]`
- `sources/sibyl/sibyl-memory-hermes/hermes-catalog/__init__.py:819` `_absorb(self._sibyl.search(clean, limit=_PREFETCH_LIMIT))`
- `sources/sibyl/sibyl-memory-hermes/hermes-catalog/__init__.py:824` `return ""`
- `sources/sibyl/sibyl-memory-hermes/hermes-catalog/__init__.py:839` `for tok in tokens[:5]:  # cap to keep prefetch cheap`
- `sources/sibyl/sibyl-memory-hermes/hermes-catalog/__init__.py:841` `_absorb(self._sibyl.search(tok, limit=_PREFETCH_LIMIT))`
- `sources/sibyl/sibyl-memory-hermes/hermes-catalog/__init__.py:848` `return ""`
- `sources/sibyl/sibyl-memory-hermes/hermes-catalog/__init__.py:852` `hits = [x["hit"] for x in ranked[:_PREFETCH_LIMIT]]`
- `sources/sibyl/sibyl-memory-hermes/hermes-catalog/__init__.py:859` `body_repr = json.dumps(body, ensure_ascii=False, default=str) if body else ""`
- `sources/sibyl/sibyl-memory-hermes/hermes-catalog/__init__.py:861` `body_repr = body_repr[:400] + "…"`
- `sources/sibyl/sibyl-memory-hermes/hermes-catalog/__init__.py:864` `body_lines.append(f"- [{label}] {body_repr}")`
- `sources/sibyl/sibyl-memory-hermes/hermes-catalog/__init__.py:877` `body = "\n".join(body_lines)`
- `sources/sibyl/sibyl-memory-hermes/hermes-catalog/__init__.py:878` `budget = _MAX_PREFETCH_CHARS - len(header) - len(guard_open) - len(guard_close) - 8`
- `sources/sibyl/sibyl-memory-hermes/hermes-catalog/__init__.py:879` `if budget > 0 and len(body) > budget:`
- `sources/sibyl/sibyl-memory-hermes/hermes-catalog/__init__.py:880` `body = body[:budget] + "…"`
- `sources/sibyl/sibyl-memory-hermes/hermes-catalog/__init__.py:881` `return "\n".join([header, guard_open, body, guard_close])`

`queue_prefetch`, body `sources/sibyl/sibyl-memory-hermes/hermes-catalog/__init__.py:886`:



`handle_tool_call`, body `sources/sibyl/sibyl-memory-hermes/hermes-catalog/__init__.py:954`:

- `sources/sibyl/sibyl-memory-hermes/hermes-catalog/__init__.py:956` `return _handoff_tool_error("Sibyl provider not initialized", "done", human)`
- `sources/sibyl/sibyl-memory-hermes/hermes-catalog/__init__.py:960` `return self._handle_tool_call(tool_name, args)`

`_handle_tool_call`, body `sources/sibyl/sibyl-memory-hermes/hermes-catalog/__init__.py:990`:

- `sources/sibyl/sibyl-memory-hermes/hermes-catalog/__init__.py:996` `return tool_error("category, name, and body are required")`
- `sources/sibyl/sibyl-memory-hermes/hermes-catalog/__init__.py:999` `return json.dumps({"ok": True, "entity": result}, default=str)`
- `sources/sibyl/sibyl-memory-hermes/hermes-catalog/__init__.py:1005` `return tool_error("category and name are required")`
- `sources/sibyl/sibyl-memory-hermes/hermes-catalog/__init__.py:1010` `return json.dumps({"entity": _scrub_value(result)}, default=str)`
- `sources/sibyl/sibyl-memory-hermes/hermes-catalog/__init__.py:1015` `return tool_error("query is required")`
- `sources/sibyl/sibyl-memory-hermes/hermes-catalog/__init__.py:1017` `# (mirrors the MCP server's clamp) so a junk 'limit' can't`
- `sources/sibyl/sibyl-memory-hermes/hermes-catalog/__init__.py:1019` `limit = _clamp_limit(args.get("limit"), _DEFAULT_SEARCH_LIMIT, _MAX_SEARCH_LIMIT)`
- `sources/sibyl/sibyl-memory-hermes/hermes-catalog/__init__.py:1025` `return tool_error("scope must be 'all' or 'graph'")`
- `sources/sibyl/sibyl-memory-hermes/hermes-catalog/__init__.py:1026` `found = self._sibyl.search_multi_record(query, limit=limit, scope=scope)`
- `sources/sibyl/sibyl-memory-hermes/hermes-catalog/__init__.py:1045` `return json.dumps(payload, default=str)`
- `sources/sibyl/sibyl-memory-hermes/hermes-catalog/__init__.py:1051` `limit = _clamp_limit(args.get("limit"), _DEFAULT_LIST_LIMIT, _MAX_LIST_LIMIT)`
- `sources/sibyl/sibyl-memory-hermes/hermes-catalog/__init__.py:1052` `rows = self._sibyl.list(category=category, status=status, limit=limit)`
- `sources/sibyl/sibyl-memory-hermes/hermes-catalog/__init__.py:1053` `return json.dumps({"entities": rows}, default=str)`
- `sources/sibyl/sibyl-memory-hermes/hermes-catalog/__init__.py:1055` `return tool_error(f"Unknown tool: {tool_name}")`
- `sources/sibyl/sibyl-memory-hermes/hermes-catalog/__init__.py:1060` `return _entitlement_tool_error(e)`
- `sources/sibyl/sibyl-memory-hermes/hermes-catalog/__init__.py:1068` `return _handoff_tool_error(f"{type(e).__name__}", nxt, human)`
- `sources/sibyl/sibyl-memory-hermes/hermes-catalog/__init__.py:1069` `return tool_error(f"{type(e).__name__}")`

`_handoff_tool_error`, body `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/_hermes_plugin/adapter.py:180`:

- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/_hermes_plugin/adapter.py:180` `return _with_handoff(tool_error(msg), msg, nxt, human)`

`_entitlement_tool_error`, body `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/_hermes_plugin/adapter.py:195`:

- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/_hermes_plugin/adapter.py:196` `return _handoff_tool_error(_SIGNED_OUT_TOOL_ERROR, "sibyl init", None)`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/_hermes_plugin/adapter.py:198` `return _handoff_tool_error(getattr(exc, "summary", None) or str(exc), nxt, human)`

`get_tool_schemas`, body `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/_hermes_plugin/adapter.py:748`:

- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/_hermes_plugin/adapter.py:748` `return [REMEMBER_SCHEMA, RECALL_SCHEMA, SEARCH_SCHEMA, LIST_SCHEMA]`

`system_prompt_block`, body `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/_hermes_plugin/adapter.py:753`:

- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/_hermes_plugin/adapter.py:753` `return (`

`prefetch`, body `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/_hermes_plugin/adapter.py:788`:

- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/_hermes_plugin/adapter.py:789` `return ""`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/_hermes_plugin/adapter.py:795` `return (block + "\n\n" if block else "") + "## Sibyl Memory: note\n" + line`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/_hermes_plugin/adapter.py:796` `return block`

`_prefetch_block`, body `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/_hermes_plugin/adapter.py:804`:

- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/_hermes_plugin/adapter.py:804` `clean = query.strip()[:1000]`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/_hermes_plugin/adapter.py:819` `_absorb(self._sibyl.search(clean, limit=_PREFETCH_LIMIT))`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/_hermes_plugin/adapter.py:824` `return ""`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/_hermes_plugin/adapter.py:839` `for tok in tokens[:5]:  # cap to keep prefetch cheap`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/_hermes_plugin/adapter.py:841` `_absorb(self._sibyl.search(tok, limit=_PREFETCH_LIMIT))`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/_hermes_plugin/adapter.py:848` `return ""`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/_hermes_plugin/adapter.py:852` `hits = [x["hit"] for x in ranked[:_PREFETCH_LIMIT]]`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/_hermes_plugin/adapter.py:859` `body_repr = json.dumps(body, ensure_ascii=False, default=str) if body else ""`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/_hermes_plugin/adapter.py:861` `body_repr = body_repr[:400] + "…"`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/_hermes_plugin/adapter.py:864` `body_lines.append(f"- [{label}] {body_repr}")`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/_hermes_plugin/adapter.py:877` `body = "\n".join(body_lines)`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/_hermes_plugin/adapter.py:878` `budget = _MAX_PREFETCH_CHARS - len(header) - len(guard_open) - len(guard_close) - 8`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/_hermes_plugin/adapter.py:879` `if budget > 0 and len(body) > budget:`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/_hermes_plugin/adapter.py:880` `body = body[:budget] + "…"`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/_hermes_plugin/adapter.py:881` `return "\n".join([header, guard_open, body, guard_close])`

`queue_prefetch`, body `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/_hermes_plugin/adapter.py:886`:



`handle_tool_call`, body `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/_hermes_plugin/adapter.py:954`:

- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/_hermes_plugin/adapter.py:956` `return _handoff_tool_error("Sibyl provider not initialized", "done", human)`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/_hermes_plugin/adapter.py:960` `return self._handle_tool_call(tool_name, args)`

`_handle_tool_call`, body `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/_hermes_plugin/adapter.py:990`:

- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/_hermes_plugin/adapter.py:996` `return tool_error("category, name, and body are required")`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/_hermes_plugin/adapter.py:999` `return json.dumps({"ok": True, "entity": result}, default=str)`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/_hermes_plugin/adapter.py:1005` `return tool_error("category and name are required")`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/_hermes_plugin/adapter.py:1010` `return json.dumps({"entity": _scrub_value(result)}, default=str)`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/_hermes_plugin/adapter.py:1015` `return tool_error("query is required")`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/_hermes_plugin/adapter.py:1017` `# (mirrors the MCP server's clamp) so a junk 'limit' can't`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/_hermes_plugin/adapter.py:1019` `limit = _clamp_limit(args.get("limit"), _DEFAULT_SEARCH_LIMIT, _MAX_SEARCH_LIMIT)`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/_hermes_plugin/adapter.py:1025` `return tool_error("scope must be 'all' or 'graph'")`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/_hermes_plugin/adapter.py:1026` `found = self._sibyl.search_multi_record(query, limit=limit, scope=scope)`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/_hermes_plugin/adapter.py:1045` `return json.dumps(payload, default=str)`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/_hermes_plugin/adapter.py:1051` `limit = _clamp_limit(args.get("limit"), _DEFAULT_LIST_LIMIT, _MAX_LIST_LIMIT)`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/_hermes_plugin/adapter.py:1052` `rows = self._sibyl.list(category=category, status=status, limit=limit)`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/_hermes_plugin/adapter.py:1053` `return json.dumps({"entities": rows}, default=str)`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/_hermes_plugin/adapter.py:1055` `return tool_error(f"Unknown tool: {tool_name}")`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/_hermes_plugin/adapter.py:1060` `return _entitlement_tool_error(e)`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/_hermes_plugin/adapter.py:1068` `return _handoff_tool_error(f"{type(e).__name__}", nxt, human)`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/_hermes_plugin/adapter.py:1069` `return tool_error(f"{type(e).__name__}")`

`save_context`, body `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/provider.py:231`:

- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/provider.py:231` `return self._client.write_event(`

`load_context`, body `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/provider.py:239`:

- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/provider.py:239` `return self._client.read_events(limit=limit)`

`clear_context`, body `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/provider.py:247`:

- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/provider.py:247` `return None`

</details>

## 4. Data access boundaries

Credentials at `~/.sibyl-memory/credentials.json`; writer creates temporary file mode 0600 and parent mode 0700 (`sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/credentials.py:189`; `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/credentials.py:165`). Store is profile-local memory.db and independently reachable by the same UID (`sources/sibyl/sibyl-memory-hermes/hermes-catalog/__init__.py:689`). Accountless usable provider: `no`, despite local SQLite; entitlement refusal is tested in `sources/sibyl/sibyl-memory-hermes/tests/test_account_gate_2026_09_27.py:38`. Heartbeat is enabled by default when an account exists, subject to telemetry opt-out (`sources/sibyl/sibyl-memory-client/src/sibyl_memory_client/_heartbeat.py:100`; `sources/sibyl/sibyl-memory-client/src/sibyl_memory_client/_heartbeat.py:114`).

Boundary model: same OS user, terminal, mounted filesystem and allowed network. Direct reachability means another client can use the same file/endpoint and available credentials, not unauthenticated access or encryption bypass. Hermes setup persists declared secrets through `hermes-agent/hermes_cli/memory_setup.py:321` and `hermes-agent/hermes_cli/config.py:2697`. These are possible configured paths; no live credentials were read.

Host credential-file detail: `get_hermes_home()/.env` is resolved at `hermes-agent/hermes_cli/config.py:473`. New files are secured to 0600 at `hermes-agent/hermes_cli/config.py:506`; the writer preserves an existing mode at `hermes-agent/hermes_cli/config.py:2544` and `:2562`. Therefore 0600 is not an unconditional mode claim for every existing profile.

<details>
<summary>Paths, environment, permissions, egress, telemetry and process-launch evidence</summary>

- `sources/sibyl/sibyl-memory-client/src/sibyl_memory_client/_heartbeat.py:19` `- Opt out entirely with the env var SIBYL_MEMORY_TELEMETRY=0.`
- `sources/sibyl/sibyl-memory-client/src/sibyl_memory_client/_heartbeat.py:38` `_DEFAULT_URL = "https://api.sibyllabs.org/api/plugin/heartbeat"`
- `sources/sibyl/sibyl-memory-client/src/sibyl_memory_client/_heartbeat.py:65` `def _telemetry_enabled() -> bool:`
- `sources/sibyl/sibyl-memory-client/src/sibyl_memory_client/_heartbeat.py:66` `val = os.environ.get("SIBYL_MEMORY_TELEMETRY", "1").strip().lower()`
- `sources/sibyl/sibyl-memory-client/src/sibyl_memory_client/_heartbeat.py:75` `string matching) so ''https://api.sibyllabs.org@evil.com/'' resolves to the`
- `sources/sibyl/sibyl-memory-client/src/sibyl_memory_client/_heartbeat.py:108` `self._url = url or os.environ.get("SIBYL_MEMORY_HEARTBEAT_URL", _DEFAULT_URL)`
- `sources/sibyl/sibyl-memory-client/src/sibyl_memory_client/_heartbeat.py:114` `self._enabled = bool(account_id) and enabled and _telemetry_enabled()`
- `sources/sibyl/sibyl-memory-client/src/sibyl_memory_client/_heartbeat.py:190` `pass  # fire-and-forget: telemetry must never disturb local memory`

- `sources/sibyl/sibyl-memory-client/src/sibyl_memory_client/storage.py:79` `_DB_FILE_MODE = 0o600`
- `sources/sibyl/sibyl-memory-client/src/sibyl_memory_client/storage.py:134` `def db_size_bytes(db_path: str | Path) -> int:`
- `sources/sibyl/sibyl-memory-client/src/sibyl_memory_client/storage.py:156` `main = Path(db_path)`
- `sources/sibyl/sibyl-memory-client/src/sibyl_memory_client/storage.py:184` `def __init__(self, db_path: str | Path):`
- `sources/sibyl/sibyl-memory-client/src/sibyl_memory_client/storage.py:185` `raw = Path(db_path).expanduser()`
- `sources/sibyl/sibyl-memory-client/src/sibyl_memory_client/storage.py:208` `self.db_path = raw.resolve()`
- `sources/sibyl/sibyl-memory-client/src/sibyl_memory_client/storage.py:216` `self.db_path.parent.mkdir(parents=True, exist_ok=True, mode=0o700)`
- `sources/sibyl/sibyl-memory-client/src/sibyl_memory_client/storage.py:220` `if hasattr(os, "chmod"):`
- `sources/sibyl/sibyl-memory-client/src/sibyl_memory_client/storage.py:222` `os.chmod(self.db_path.parent, 0o700)`
- `sources/sibyl/sibyl-memory-client/src/sibyl_memory_client/storage.py:256` `perms-tightening chmod could retarget through it. Mirrors the main-file`
- `sources/sibyl/sibyl-memory-client/src/sibyl_memory_client/storage.py:259` `sidecar = self.db_path.with_name(self.db_path.name + suffix)`
- `sources/sibyl/sibyl-memory-client/src/sibyl_memory_client/storage.py:330` `str(self.db_path),`
- `sources/sibyl/sibyl-memory-client/src/sibyl_memory_client/storage.py:355` `StorageError without leaking db_path or query text."""`
- `sources/sibyl/sibyl-memory-client/src/sibyl_memory_client/storage.py:708` `Idempotent. Safe on systems where chmod is a no-op (Windows): we`
- `sources/sibyl/sibyl-memory-client/src/sibyl_memory_client/storage.py:709` `guard with hasattr. Errors during chmod are non-fatal: we want`
- `sources/sibyl/sibyl-memory-client/src/sibyl_memory_client/storage.py:710` `secure-by-default but won't block a working DB if the chmod call`
- `sources/sibyl/sibyl-memory-client/src/sibyl_memory_client/storage.py:713` `if not hasattr(os, "chmod"):`
- `sources/sibyl/sibyl-memory-client/src/sibyl_memory_client/storage.py:714` `return  # platform without POSIX chmod (Windows)`
- `sources/sibyl/sibyl-memory-client/src/sibyl_memory_client/storage.py:715` `targets = [self.db_path]`
- `sources/sibyl/sibyl-memory-client/src/sibyl_memory_client/storage.py:717` `sidecar = self.db_path.with_name(self.db_path.name + suffix)`
- `sources/sibyl/sibyl-memory-client/src/sibyl_memory_client/storage.py:725` `follow_supported = os.chmod in getattr(os, "supports_follow_symlinks", set())`
- `sources/sibyl/sibyl-memory-client/src/sibyl_memory_client/storage.py:734` `os.chmod(path, _DB_FILE_MODE, follow_symlinks=False)`
- `sources/sibyl/sibyl-memory-client/src/sibyl_memory_client/storage.py:736` `os.chmod(path, _DB_FILE_MODE)`

- `sources/sibyl/sibyl-memory-hermes/hermes-catalog/__init__.py:22` `Credentials live in ~/.sibyl-memory/credentials.json (managed by the`
- `sources/sibyl/sibyl-memory-hermes/hermes-catalog/__init__.py:24` `SDK auto-load credentials. The only Hermes-side option is 'db_path',`
- `sources/sibyl/sibyl-memory-hermes/hermes-catalog/__init__.py:30` `the active tenant and wins over any tenant in credentials.json, matching`
- `sources/sibyl/sibyl-memory-hermes/hermes-catalog/__init__.py:521` `self._db_path: Path | None = None`
- `sources/sibyl/sibyl-memory-hermes/hermes-catalog/__init__.py:563` `client.enable_lifecycle_counting(Path.home())`
- `sources/sibyl/sibyl-memory-hermes/hermes-catalog/__init__.py:569` `self._notices.start(client, home=Path.home(), cwd=Path(os.getcwd()))`
- `sources/sibyl/sibyl-memory-hermes/hermes-catalog/__init__.py:592` `run_learner(client, home=Path.home())`
- `sources/sibyl/sibyl-memory-hermes/hermes-catalog/__init__.py:604` `line = n.next_notice(client, home=Path.home(), cwd=Path(os.getcwd()))`
- `sources/sibyl/sibyl-memory-hermes/hermes-catalog/__init__.py:667` `env_tenant = os.environ.get("SIBYL_TENANT_ID", "").strip()`
- `sources/sibyl/sibyl-memory-hermes/hermes-catalog/__init__.py:689` `db_path = db_dir / "memory.db"`
- `sources/sibyl/sibyl-memory-hermes/hermes-catalog/__init__.py:690` `self._db_path = db_path`
- `sources/sibyl/sibyl-memory-hermes/hermes-catalog/__init__.py:697` `db_path=db_path,`
- `sources/sibyl/sibyl-memory-hermes/hermes-catalog/__init__.py:703` `db_path, session_id, self._profile,`
- `sources/sibyl/sibyl-memory-hermes/hermes-catalog/__init__.py:732` `candidates.append(Path.home() / ".hermes" / "active_profile")`
- `sources/sibyl/sibyl-memory-hermes/hermes-catalog/__init__.py:974` `buf = _sg.recorder(Path.home())`
- `sources/sibyl/sibyl-memory-hermes/hermes-catalog/__init__.py:985` `_sg.recorder(Path.home()).flush()`
- `sources/sibyl/sibyl-memory-hermes/hermes-catalog/__init__.py:1208` `running 'sibyl init' writes ~/.sibyl-memory/credentials.json,`
- `sources/sibyl/sibyl-memory-hermes/hermes-catalog/__init__.py:1217` `returns [] and never double-prompts. Secrets stay in credentials.json as the`

- `sources/sibyl/sibyl-memory-hermes/hermes-catalog/plugin.yaml:4` `homepage: https://sibyllabs.org/memory`

- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/__init__.py:5` `DEFAULT_DB_PATH         ~/.sibyl-memory/memory.db`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/__init__.py:6` `DEFAULT_CRED_PATH       ~/.sibyl-memory/credentials.json`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/__init__.py:49` `provider = SibylMemoryProvider()         # auto-loads credentials.json`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/__init__.py:54` `See https://docs.sibyllabs.org/memory/integrations for the full integration`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/__init__.py:63` `DEFAULT_DB_PATH,`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/__init__.py:84` `"DEFAULT_DB_PATH",`

- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/_aesthetic.py:54` `if os.environ.get("NO_COLOR"):`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/_aesthetic.py:56` `if os.environ.get("TERM", "").lower() == "dumb":`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/_aesthetic.py:60` `if os.environ.get("SIBYL_FORCE_COLOR") == "1":`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/_aesthetic.py:64` `colorterm = os.environ.get("COLORTERM", "").lower()`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/_aesthetic.py:67` `term_program = os.environ.get("TERM_PROGRAM", "").lower()`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/_aesthetic.py:70` `term = os.environ.get("TERM", "").lower()`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/_aesthetic.py:78` `if os.environ.get("NO_COLOR"):`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/_aesthetic.py:80` `if os.environ.get("TERM", "").lower() == "dumb":`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/_aesthetic.py:82` `if os.environ.get("SIBYL_FORCE_COLOR") == "1":`

- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/_banner.py:53` `if os.environ.get("NO_COLOR"):`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/_banner.py:55` `if os.environ.get("TERM", "").lower() == "dumb":`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/_banner.py:59` `colorterm = os.environ.get("COLORTERM", "").lower()`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/_banner.py:64` `term_program = os.environ.get("TERM_PROGRAM", "").lower()`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/_banner.py:67` `term = os.environ.get("TERM", "").lower()`

- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/_hermes_plugin/adapter.py:22` `Credentials live in ~/.sibyl-memory/credentials.json (managed by the`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/_hermes_plugin/adapter.py:24` `SDK auto-load credentials. The only Hermes-side option is 'db_path',`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/_hermes_plugin/adapter.py:30` `the active tenant and wins over any tenant in credentials.json, matching`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/_hermes_plugin/adapter.py:521` `self._db_path: Path | None = None`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/_hermes_plugin/adapter.py:563` `client.enable_lifecycle_counting(Path.home())`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/_hermes_plugin/adapter.py:569` `self._notices.start(client, home=Path.home(), cwd=Path(os.getcwd()))`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/_hermes_plugin/adapter.py:592` `run_learner(client, home=Path.home())`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/_hermes_plugin/adapter.py:604` `line = n.next_notice(client, home=Path.home(), cwd=Path(os.getcwd()))`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/_hermes_plugin/adapter.py:667` `env_tenant = os.environ.get("SIBYL_TENANT_ID", "").strip()`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/_hermes_plugin/adapter.py:689` `db_path = db_dir / "memory.db"`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/_hermes_plugin/adapter.py:690` `self._db_path = db_path`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/_hermes_plugin/adapter.py:697` `db_path=db_path,`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/_hermes_plugin/adapter.py:703` `db_path, session_id, self._profile,`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/_hermes_plugin/adapter.py:732` `candidates.append(Path.home() / ".hermes" / "active_profile")`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/_hermes_plugin/adapter.py:974` `buf = _sg.recorder(Path.home())`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/_hermes_plugin/adapter.py:985` `_sg.recorder(Path.home()).flush()`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/_hermes_plugin/adapter.py:1208` `running 'sibyl init' writes ~/.sibyl-memory/credentials.json,`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/_hermes_plugin/adapter.py:1217` `returns [] and never double-prompts. Secrets stay in credentials.json as the`

- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/_hermes_plugin/plugin.yaml:4` `homepage: https://sibyllabs.org/memory`

- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/credentials.py:3` `'sibyl init' writes '~/.sibyl-memory/credentials.json' after a successful`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/credentials.py:26` `DEFAULT_DB_PATH = "~/.sibyl-memory/memory.db"`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/credentials.py:27` `DEFAULT_CRED_PATH = "~/.sibyl-memory/credentials.json"`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/credentials.py:151` `''os.open(O_WRONLY|O_CREAT|O_EXCL|O_NOFOLLOW, 0o600)''. Previously`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/credentials.py:152` `used ''tmp.write_text()'' then ''os.chmod(0o600)'', leaving a`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/credentials.py:160` `resolved.parent.mkdir(parents=True, exist_ok=True, mode=0o700)`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/credentials.py:165` `os.chmod(resolved.parent, 0o700)`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/credentials.py:189` `fd = os.open(str(tmp), flags, 0o600)`

- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/install_plugin.py:50` `env = os.environ.get("HERMES_HOME")`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/install_plugin.py:53` `return Path.home() / ".hermes"`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/install_plugin.py:203` `rep = sk.install(Path.home())`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/install_plugin.py:208` `st = sk.install_shim(Path.home(), p, sk.shim_text(name, "hermes"),`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/install_plugin.py:282` `p = Path(os.environ.get("SIBYL_CREDENTIALS") or "~/.sibyl-memory/credentials.json").expanduser()`

- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/provider.py:55` `DEFAULT_DB_PATH,`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/provider.py:83` `db_path:        path to the local SQLite database. Defaults to`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/provider.py:86` `tenant_id:      explicit tenant override. If None, credentials.json`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/provider.py:90` `credentials_path: override for credentials.json discovery.`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/provider.py:98` `autoload_credentials: if True (default), read credentials.json on`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/provider.py:104` `db_path: str | Path = DEFAULT_DB_PATH,`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/provider.py:157` `db_path,`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/provider.py:165` `telemetry=(os.environ.get("SIBYL_MEMORY_TELEMETRY") == "1"),`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/provider.py:528` `db_path = self._client.storage.db_path`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/provider.py:532` `"db_path": str(db_path),`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/provider.py:536` `"db_size_bytes": db_size_bytes(db_path) if db_path.exists() else 0,`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/provider.py:559` `f"SibylMemoryProvider(db={self._client.storage.db_path}, "`

</details>

## 5. Reproducibility

`range`: sibyl-memory-hermes>=0.6.0,<0.7 and sibyl-memory-client>=0.10.0,<0.11 (`sources/sibyl/sibyl-memory-hermes/hermes-catalog/plugin.yaml:10`). Account API/server revisions are not pinned by those ranges.

Retained lock/checksum files: none in snapshot. Upstream completeness is unknown: fetch.py:15 excludes .lock files, fetch.py:12 excludes build/vendor directories. Absence here is not absence upstream.

- `sources/sibyl/sibyl-memory-hermes/hermes-catalog/plugin.yaml:3` `version: 0.6.0`
- `sources/sibyl/sibyl-memory-hermes/hermes-catalog/plugin.yaml:8` `# installs these into Hermes' environment. Keep the bounds within one minor line.`
- `sources/sibyl/sibyl-memory-hermes/hermes-catalog/plugin.yaml:9` `python_dependencies:`
- `sources/sibyl/sibyl-memory-hermes/hermes-catalog/plugin.yaml:10` `- sibyl-memory-hermes>=0.6.0,<0.7`
- `sources/sibyl/sibyl-memory-hermes/hermes-catalog/plugin.yaml:11` `- sibyl-memory-client>=0.10.0,<0.11`

- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/_hermes_plugin/plugin.yaml:3` `version: 0.6.0`

- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/install_plugin.py:1` `"""'sibyl-memory-hermes install-plugin': installs the Sibyl adapter into Hermes.`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/install_plugin.py:3` `Hermes' loader does NOT use pip entry points (verified against`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/install_plugin.py:10` `After 'pip install sibyl-memory-hermes', the user runs this script to drop`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/install_plugin.py:15` `sibyl-memory-hermes install-plugin`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/install_plugin.py:16` `sibyl-memory-hermes install-plugin --hermes-home /custom/path`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/install_plugin.py:17` `sibyl-memory-hermes install-plugin --force`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/install_plugin.py:18` `sibyl-memory-hermes install-plugin --dry-run`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/install_plugin.py:19` `sibyl-memory-hermes uninstall-plugin`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/install_plugin.py:23` `actual prior Sibyl install (existing plugin.yaml with name: sibyl).`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/install_plugin.py:57` `"""User-install location for a Hermes memory provider plugin.`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/install_plugin.py:73` `only under the installed package's ''plugins/memory/<name>/'' directory.`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/install_plugin.py:74` `The tester's workaround was to mount our user-path install into that scan`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/install_plugin.py:75` `path. This resolves that scan path directly so the installer can write it.`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/install_plugin.py:80` `unit-testable without a real Hermes install.`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/install_plugin.py:98` `memory-provider-path installs. Returns 0 on success, or a non-zero refusal`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/install_plugin.py:99` `code matching the original ''install()'' contract (2 not-empty, 3 symlink,`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/install_plugin.py:105` `print(a.dim("  Sibyl will not install through symlinks. Remove the symlink and rerun."))`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/install_plugin.py:112` `if not _looks_like_sibyl_install(dest):`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/install_plugin.py:113` `print(a.err_line(f"Refused: {dest} is not empty but does not contain a prior Sibyl install."))`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/install_plugin.py:160` `hermes-agent isn't installed.`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/install_plugin.py:173` `def _looks_like_sibyl_install(dest: Path) -> bool:`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/install_plugin.py:175` `install before we'll rmtree it.`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/install_plugin.py:177` `Recognizes the install by 'plugin.yaml' with 'name: sibyl' in it (the`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/install_plugin.py:195` `def _install_skills(hermes_home: Path) -> None:`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/install_plugin.py:200` `person's edit is never overwritten. Never fails the install."""`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/install_plugin.py:203` `rep = sk.install(Path.home())`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/install_plugin.py:204` `lines = [f"procedures v{rep.version}: "`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/install_plugin.py:208` `st = sk.install_shim(Path.home(), p, sk.shim_text(name, "hermes"),`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/install_plugin.py:212` `lines = [f"save/debloat skills not installed ({type(e).__name__})"]`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/install_plugin.py:219` `_INSTALL_CMD = "sibyl-memory-hermes install-plugin"`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/install_plugin.py:234` `if rc == 2:`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/install_plugin.py:235` `_handoff(f"{_INSTALL_CMD} --force", human=f"{dest} already has files; confirm replacing them")`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/install_plugin.py:236` `elif rc == 3:`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/install_plugin.py:237` `_handoff(_INSTALL_CMD, human=f"{dest} is a symlink; remove it, then tell me")`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/install_plugin.py:239` `_handoff(_INSTALL_CMD, human=f"{dest} holds files that are not a Sibyl install; move them "`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/install_plugin.py:257` `return isinstance(mem, dict) and mem.get("provider") == "sibyl"`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/install_plugin.py:271` `in_memory = line.strip() == "memory:"`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/install_plugin.py:275` `if sep and key.strip() == "provider" and val.strip().strip("'\"") == "sibyl":`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/install_plugin.py:286` `def install(hermes_home: Path, force: bool, dry_run: bool,`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/install_plugin.py:293` `# ── HEAVY: install moment. Full SIBYL banner + section header. ──`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/install_plugin.py:295` `print(a.section_header("install-plugin",`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/install_plugin.py:304` `#    user-plugin scan + shows in 'hermes plugins list' on all versions.`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/install_plugin.py:324` `provider_written = (prc == 0)`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/install_plugin.py:326` `print(a.dim("  Provider-path install refused (see above). User-plugin path stands."))`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/install_plugin.py:339` `print(a.dim("  shows Plugin: NOT installed, rerun with --memory-provider-path"))`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/install_plugin.py:340` `print(a.dim("  pointing at your Hermes install's plugins/memory directory, e.g.:"))`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/install_plugin.py:341` `print(a.dim("    sibyl-memory-hermes install-plugin --memory-provider-path /opt/hermes/plugins/memory"))`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/install_plugin.py:342` `provider_human = ("where your Hermes install's plugins/memory directory is (Hermes 0.7+ reads "`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/install_plugin.py:344` `provider_later = (f"{_INSTALL_CMD} --memory-provider-path <dir>",)`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/install_plugin.py:352` `# Surface which Hermes versions read which path so the split is never a mystery.`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/install_plugin.py:363` `print(a.success_line("Plugin installed."))`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/install_plugin.py:364` `_install_skills(hermes_home)`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/install_plugin.py:395` `print(f"  {a.dim('uninstall later:')} {a.color('sibyl-memory-hermes uninstall-plugin', a.INK)}")`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/install_plugin.py:412` `Sibyl-install sentinel). Returns: 0 removed / 1 absent / 3 symlink /`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/install_plugin.py:422` `if not _looks_like_sibyl_install(dest):`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/install_plugin.py:423` `print(a.err_line(f"Refused: {dest} is not recognized as a Sibyl install."))`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/install_plugin.py:434` `def uninstall(hermes_home: Path, dry_run: bool,`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/install_plugin.py:437` `# MH-7: the installer (Hermes 0.7+) also writes the provider-scan-path copy`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/install_plugin.py:438` `# under <hermes pkg>/plugins/memory/sibyl. Uninstall must remove that too,`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/install_plugin.py:439` `# or the adapter lingers and Hermes keeps loading it after "uninstall".`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/install_plugin.py:443` `print(a.section_header("uninstall-plugin",`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/install_plugin.py:444` `subtitle="remove sibyl from this hermes install"))`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/install_plugin.py:482` `if user_rc == 1 and provider_rc in (1, 5):`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/install_plugin.py:484` `print(a.warn_line("Nothing was removed: no Sibyl install found at either path."))`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/install_plugin.py:500` `description="Install the Sibyl memory provider plugin into Hermes.",`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/install_plugin.py:504` `p_install = sub.add_parser("install-plugin", help="Install the Sibyl plugin into HERMES_HOME.")`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/install_plugin.py:505` `p_install.add_argument("--hermes-home", help="Override HERMES_HOME (defaults to env var or ~/.hermes).")`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/install_plugin.py:506` `p_install.add_argument("--memory-provider-path",`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/install_plugin.py:507` `help="Path to your Hermes install's plugins/memory directory "`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/install_plugin.py:510` `p_install.add_argument("--force", action="store_true", help="Overwrite an existing Sibyl plugin directory (refuses non-Sibyl content).")`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/install_plugin.py:511` `p_install.add_argument("--dry-run", action="store_true", help="Show what would happen without writing.")`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/install_plugin.py:513` `p_uninstall = sub.add_parser("uninstall-plugin", help="Remove the Sibyl plugin from HERMES_HOME.")`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/install_plugin.py:514` `p_uninstall.add_argument("--hermes-home", help="Override HERMES_HOME (defaults to env var or ~/.hermes).")`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/install_plugin.py:515` `p_uninstall.add_argument("--memory-provider-path",`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/install_plugin.py:516` `help="Path to your Hermes install's plugins/memory directory; the "`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/install_plugin.py:519` `p_uninstall.add_argument("--dry-run", action="store_true", help="Show what would happen without writing.")`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/install_plugin.py:524` `if args.cmd == "install-plugin":`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/install_plugin.py:525` `return install(hermes_home, force=args.force, dry_run=args.dry_run,`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/install_plugin.py:527` `if args.cmd == "uninstall-plugin":`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/install_plugin.py:528` `return uninstall(hermes_home, dry_run=args.dry_run,`
- `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/install_plugin.py:534` `if __name__ == "__main__":`

## 6. Tests

There is no adapter forget-path test because no forget tool is exported. Engine delete/edge-cascade tests are separately inventoried. Adapter tests include signed-out refusals, not only import success (`sources/sibyl/sibyl-memory-hermes/tests/test_account_gate_2026_09_27.py:38`). Engine test deletes an entity and asserts that touching graph edges cascade while an unrelated edge survives. (`sources/sibyl/sibyl-memory-client/tests/test_links_2026_10_22.py:265`).

Adapter-related declarations: **172**. Python AST test_ functions, fixtures excluded; literal TS/JS it/test, Go Test*, Rust test_* declarations. No parametrization expansion, collection, execution, or coverage measurement. Other rows are engine test candidates selected by mutation names; names alone do not verify forget coverage.

| file | declarations | adapter-related | forget-related declarations |
|---|---:|---|---|
| `sources/sibyl/sibyl-memory-cli/tests/test_doctor_2026_10_28.py:1` | 9 | no | `test_leftover_row_is_purged_and_verified` `sources/sibyl/sibyl-memory-cli/tests/test_doctor_2026_10_28.py:121`; `test_purge_failure_is_reported` `sources/sibyl/sibyl-memory-cli/tests/test_doctor_2026_10_28.py:128` |
| `sources/sibyl/sibyl-memory-client/tests/test_capcheck.py:1` | 17 | no | `test_cap_gate_invalidate_cache` `sources/sibyl/sibyl-memory-client/tests/test_capcheck.py:378` |
| `sources/sibyl/sibyl-memory-client/tests/test_entitlement_2026_09_27.py:1` | 73 | no | `test_deleting_credentials_mid_session` `sources/sibyl/sibyl-memory-client/tests/test_entitlement_2026_09_27.py:531`; `test_deleted_entitlement_mid_session_bootstraps_not_bypasses` `sources/sibyl/sibyl-memory-client/tests/test_entitlement_2026_09_27.py:974` |
| `sources/sibyl/sibyl-memory-client/tests/test_gate_hardening_2026_09_27.py:1` | 19 | no | `test_deleting_entitlement_json_does_not_restart_anything` `sources/sibyl/sibyl-memory-client/tests/test_gate_hardening_2026_09_27.py:118`; `test_denied_then_file_deleted_stays_locked` `sources/sibyl/sibyl-memory-client/tests/test_gate_hardening_2026_09_27.py:154`; `test_deleting_the_file_after_a_denial_does_not_unlock` `sources/sibyl/sibyl-memory-client/tests/test_gate_hardening_2026_09_27.py:269` |
| `sources/sibyl/sibyl-memory-client/tests/test_learning_signals_2026_09_29.py:1` | 19 | no | `test_delete_then_set_within_24_hours` `sources/sibyl/sibyl-memory-client/tests/test_learning_signals_2026_09_29.py:194` |
| `sources/sibyl/sibyl-memory-client/tests/test_links_2026_10_22.py:1` | 35 | no | `test_delete_entity_cascades_every_touching_edge` `sources/sibyl/sibyl-memory-client/tests/test_links_2026_10_22.py:265`; `test_cascade_helper_is_the_delete_reference_merge_point` `sources/sibyl/sibyl-memory-client/tests/test_links_2026_10_22.py:297`; `test_delete_reference_cascades_edges` `sources/sibyl/sibyl-memory-client/tests/test_links_2026_10_22.py:305` |
| `sources/sibyl/sibyl-memory-client/tests/test_reference_tier_2026_10_22.py:1` | 27 | no | `test_delete_round_trip_and_search_index` `sources/sibyl/sibyl-memory-client/tests/test_reference_tier_2026_10_22.py:205`; `test_delete_reference_is_a_write_op_everywhere` `sources/sibyl/sibyl-memory-client/tests/test_reference_tier_2026_10_22.py:267` |
| `sources/sibyl/sibyl-memory-client/tests/test_search_fixes_v1_2026_10_22.py:1` | 17 | no | `test_delete_reference_supersedes_the_accepted_proposal` `sources/sibyl/sibyl-memory-client/tests/test_search_fixes_v1_2026_10_22.py:246`; `test_delete_reference_details_reports_edges_and_proposals` `sources/sibyl/sibyl-memory-client/tests/test_search_fixes_v1_2026_10_22.py:259`; `test_delete_reference_proposal_update_is_tenant_scoped` `sources/sibyl/sibyl-memory-client/tests/test_search_fixes_v1_2026_10_22.py:273`; `test_delete_reference_rolls_back_the_proposal_with_the_delete` `sources/sibyl/sibyl-memory-client/tests/test_search_fixes_v1_2026_10_22.py:285` |
| `sources/sibyl/sibyl-memory-client/tests/test_signals_2026_09_29.py:1` | 26 | no | `test_clear_deletes_the_file_and_a_running_buffer_cannot_bring_it_back` `sources/sibyl/sibyl-memory-client/tests/test_signals_2026_09_29.py:199` |
| `sources/sibyl/sibyl-memory-hermes/tests/conftest.py:1` | 0 | yes | none |
| `sources/sibyl/sibyl-memory-hermes/tests/test_account_gate_2026_09_27.py:1` | 13 | yes | none |
| `sources/sibyl/sibyl-memory-hermes/tests/test_adapter.py:1` | 21 | yes | none |
| `sources/sibyl/sibyl-memory-hermes/tests/test_adapter_hardening_2026_06_25.py:1` | 11 | yes | none |
| `sources/sibyl/sibyl-memory-hermes/tests/test_audit_fixes_2026_06_30.py:1` | 8 | yes | none |
| `sources/sibyl/sibyl-memory-hermes/tests/test_catalog_install_2026_10_22.py:1` | 5 | yes | none |
| `sources/sibyl/sibyl-memory-hermes/tests/test_coa_coercion_2026_05_30.py:1` | 4 | yes | none |
| `sources/sibyl/sibyl-memory-hermes/tests/test_handoff_2026_10_03.py:1` | 16 | yes | none |
| `sources/sibyl/sibyl-memory-hermes/tests/test_health_tier_2026_10_08.py:1` | 3 | yes | none |
| `sources/sibyl/sibyl-memory-hermes/tests/test_install_hardening_2026_06_25.py:1` | 9 | yes | none |
| `sources/sibyl/sibyl-memory-hermes/tests/test_lifecycle_hermes_2026_09_27.py:1` | 9 | yes | none |
| `sources/sibyl/sibyl-memory-hermes/tests/test_network_guard_2026_09_27.py:1` | 5 | yes | none |
| `sources/sibyl/sibyl-memory-hermes/tests/test_prefetch_fence.py:1` | 2 | yes | none |
| `sources/sibyl/sibyl-memory-hermes/tests/test_provider_path_2026_06_11.py:1` | 6 | yes | none |
| `sources/sibyl/sibyl-memory-hermes/tests/test_r1_hermes_2026_09_29.py:1` | 3 | yes | none |
| `sources/sibyl/sibyl-memory-hermes/tests/test_release_floor_2026_10_22.py:1` | 2 | yes | none |
| `sources/sibyl/sibyl-memory-hermes/tests/test_search_multi_record_diagnostics_2026_08_22.py:1` | 2 | yes | none |
| `sources/sibyl/sibyl-memory-hermes/tests/test_sibyl_search_question_query_2026_08_16.py:1` | 2 | yes | none |
| `sources/sibyl/sibyl-memory-hermes/tests/test_signals_capture_hermes_2026_09_29.py:1` | 4 | yes | none |
| `sources/sibyl/sibyl-memory-hermes/tests/test_smoke.py:1` | 21 | yes | `test_remember_recall_forget` `sources/sibyl/sibyl-memory-hermes/tests/test_smoke.py:143` |
| `sources/sibyl/sibyl-memory-hermes/tests/test_snippet_cap_2026_10_22.py:1` | 1 | yes | none |
| `sources/sibyl/sibyl-memory-hermes/tests/test_superpatch_h_2026_07_05.py:1` | 5 | yes | none |
| `sources/sibyl/sibyl-memory-hermes/tests/test_telemetry_default_off_2026_10_28.py:1` | 2 | yes | none |
| `sources/sibyl/sibyl-memory-hermes/tests/test_tier_search_2026_10_22.py:1` | 4 | yes | none |
| `sources/sibyl/sibyl-memory-hermes/tests/test_verdict_contract_2026_08_31.py:1` | 14 | yes | none |
| `sources/sibyl/sibyl-memory-langgraph/tests/test_adv_fidelity.py:1` | 19 | no | `test_none_inside_dict_vs_putop_none_delete` `sources/sibyl/sibyl-memory-langgraph/tests/test_adv_fidelity.py:162` |
| `sources/sibyl/sibyl-memory-langgraph/tests/test_adv_fuzz.py:1` | 64 | no | `test_value_none_is_delete` `sources/sibyl/sibyl-memory-langgraph/tests/test_adv_fuzz.py:202`; `test_put_get_delete_sequence` `sources/sibyl/sibyl-memory-langgraph/tests/test_adv_fuzz.py:469` |
| `sources/sibyl/sibyl-memory-langgraph/tests/test_adv_scale.py:1` | 33 | no | `test_cap_exceeded_then_delete_allows_new_write` `sources/sibyl/sibyl-memory-langgraph/tests/test_adv_scale.py:806`; `test_delete_on_instance1_visible_to_instance2` `sources/sibyl/sibyl-memory-langgraph/tests/test_adv_scale.py:593` |
| `sources/sibyl/sibyl-memory-langgraph/tests/test_adv_security.py:1` | 25 | no | `test_defense_delete_does_not_cross_collide` `sources/sibyl/sibyl-memory-langgraph/tests/test_adv_security.py:140` |
| `sources/sibyl/sibyl-memory-langgraph/tests/test_async.py:1` | 19 | no | `test_adelete_and_parity_with_sync_delete` `sources/sibyl/sibyl-memory-langgraph/tests/test_async.py:131`; `test_aput_none_value_deletes` `sources/sibyl/sibyl-memory-langgraph/tests/test_async.py:148` |
| `sources/sibyl/sibyl-memory-langgraph/tests/test_conformance.py:1` | 25 | no | `test_delete_removes` `sources/sibyl/sibyl-memory-langgraph/tests/test_conformance.py:145`; `test_delete_missing_is_noop` `sources/sibyl/sibyl-memory-langgraph/tests/test_conformance.py:154` |
| `sources/sibyl/sibyl-memory-langgraph/tests/test_crud.py:1` | 34 | no | `test_delete_then_get_none` `sources/sibyl/sibyl-memory-langgraph/tests/test_crud.py:129`; `test_delete_missing_key_is_idempotent_no_crash` `sources/sibyl/sibyl-memory-langgraph/tests/test_crud.py:137`; `test_delete_via_batch_put_none_sentinel` `sources/sibyl/sibyl-memory-langgraph/tests/test_crud.py:149`; `test_durability_overwrite_and_delete_persist` `sources/sibyl/sibyl-memory-langgraph/tests/test_crud.py:397` |
| `sources/sibyl/sibyl-memory-langgraph/tests/test_namespaces.py:1` | 30 | no | `test_list_namespaces_reflects_delete` `sources/sibyl/sibyl-memory-langgraph/tests/test_namespaces.py:112`; `test_delete_rejects_invalid_namespace` `sources/sibyl/sibyl-memory-langgraph/tests/test_namespaces.py:332` |
| `sources/sibyl/sibyl-memory-mcp/tests/test_account_gate_2026_09_27.py:1` | 16 | no | `test_deleting_credentials_mid_session_is_no_account` `sources/sibyl/sibyl-memory-mcp/tests/test_account_gate_2026_09_27.py:126` |
| `sources/sibyl/sibyl-memory-mcp/tests/test_reference_tools_2026_10_22.py:1` | 25 | no | `test_delete_reference` `sources/sibyl/sibyl-memory-mcp/tests/test_reference_tools_2026_10_22.py:262` |
| `sources/sibyl/sibyl-memory-mcp/tests/test_v1_panel_fixes_2026_10_22.py:1` | 12 | no | `test_ordinary_keys_delete_without_force_and_report_edges` `sources/sibyl/sibyl-memory-mcp/tests/test_v1_panel_fixes_2026_10_22.py:171`; `test_force_delete_supersedes_an_accepted_skill_proposal` `sources/sibyl/sibyl-memory-mcp/tests/test_v1_panel_fixes_2026_10_22.py:184` |

## 7. Corrections to B2

Old values below are copied from the triage row (TRIAGE.md), not reconstructed from memory. A refinement narrows an over-broad field; it need not reverse its original value. Counts are distinct field paths, not the number of factual errors.

- `context.bounds_status`: `"Source expressions include recall and other output bounds. They are not interchangeable: token vs character limits and static block vs tool output must be read at their cited call sites. No single total bound is established unless explicitly stated."` → 6000-character fenced recall body plus an optional notice outside that budget; five hits, 400-character bodies. Evidence: `sources/sibyl/sibyl-memory-hermes/hermes-catalog/__init__.py:878`, `sources/sibyl/sibyl-memory-hermes/hermes-catalog/__init__.py:795`.
- `engine_delete.scope`: `"Named deletion/retraction endpoint or method, not a guarantee of physical erasure. Its reach may overlap an agent tool; lack of a native tool is counted separately."` → `update_only`: get_tool_schemas returns remember, recall, search and list, no forget/delete tool (`sources/sibyl/sibyl-memory-hermes/hermes-catalog/__init__.py:748`). Upsert through remember can replace a structured record; it is not erasure. Engine `yes`: `delete_entity` executes DELETE by tenant/category/name and cascades incident links in the same transaction (`sources/sibyl/sibyl-memory-client/src/sibyl_memory_client/client.py:1541`); `delete_reference`/`delete_reference_details` remove reference data, while `archive_entity` preserves history (`sources/sibyl/sibyl-memory-client/src/sibyl_memory_client/client.py:1823`; `sources/sibyl/sibyl-memory-client/src/sibyl_memory_client/client.py:2004`). `partial`: targeted graph-link cleanup is visible; journal, versions, other tiers and host copies are separate. No general whole-history wipe is implied. Evidence: `sources/sibyl/sibyl-memory-hermes/hermes-catalog/__init__.py:748`, `sources/sibyl/sibyl-memory-client/src/sibyl_memory_client/client.py:1541`, `sources/sibyl/sibyl-memory-client/src/sibyl_memory_client/client.py:1823`, `sources/sibyl/sibyl-memory-client/src/sibyl_memory_client/client.py:2004`.
- `context.mechanism.v`: `"Source expressions below identify retrieval, cached/static returns, lifecycle capture, and limits. A hook definition is not a guarantee it emits nonempty context each turn; branches and configuration apply."` → System prompt describes four tiers and four tools (`sources/sibyl/sibyl-memory-hermes/hermes-catalog/__init__.py:753`). Prefetch searches at most five hits, limits each body to 400 characters, frames untrusted memory with a random nonce and budgets the block to 6000 characters; an optional notice is appended outside that block (`sources/sibyl/sibyl-memory-hermes/hermes-catalog/__init__.py:852`; `sources/sibyl/sibyl-memory-hermes/hermes-catalog/__init__.py:878`; `sources/sibyl/sibyl-memory-hermes/hermes-catalog/__init__.py:795`). Thus 6000 is not an unconditional whole-return cap. Tool results have separate per-hit/output budgets below. Evidence: `sources/sibyl/sibyl-memory-hermes/hermes-catalog/__init__.py:753`, `sources/sibyl/sibyl-memory-hermes/hermes-catalog/__init__.py:852`, `sources/sibyl/sibyl-memory-hermes/hermes-catalog/__init__.py:878`, `sources/sibyl/sibyl-memory-hermes/hermes-catalog/__init__.py:795`.
- `sec.telemetry.v`: `"Account-linked aggregate operation heartbeat is enabled by default; SIBYL_MEMORY_TELEMETRY=0 disables it. URL defaults to https://api.sibyllabs.org/api/plugin/heartbeat; implementation is present. This is separate from entitlement requests."` → Credentials at `~/.sibyl-memory/credentials.json`; writer creates temporary file mode 0600 and parent mode 0700 (`sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/credentials.py:189`; `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/credentials.py:165`). Store is profile-local memory.db and independently reachable by the same UID (`sources/sibyl/sibyl-memory-hermes/hermes-catalog/__init__.py:689`). Accountless usable provider: `no`, despite local SQLite; entitlement refusal is tested in `sources/sibyl/sibyl-memory-hermes/tests/test_account_gate_2026_09_27.py:38`. Heartbeat is enabled by default when an account exists, subject to telemetry opt-out (`sources/sibyl/sibyl-memory-client/src/sibyl_memory_client/_heartbeat.py:100`; `sources/sibyl/sibyl-memory-client/src/sibyl_memory_client/_heartbeat.py:114`). Evidence: `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/credentials.py:189`, `sources/sibyl/sibyl-memory-hermes/src/sibyl_memory_hermes/credentials.py:165`, `sources/sibyl/sibyl-memory-hermes/hermes-catalog/__init__.py:689`, `sources/sibyl/sibyl-memory-hermes/tests/test_account_gate_2026_09_27.py:38`, `sources/sibyl/sibyl-memory-client/src/sibyl_memory_client/_heartbeat.py:100`, `sources/sibyl/sibyl-memory-client/src/sibyl_memory_client/_heartbeat.py:114`.
- `repro.decls`: `["sibyl-memory-hermes/hermes-catalog/plugin.yaml:9", "sibyl-memory-hermes/hermes-catalog/plugin.yaml:10", "sibyl-memory-hermes/hermes-catalog/plugin.yaml:11"]` → `range`: sibyl-memory-hermes>=0.6.0,<0.7 and sibyl-memory-client>=0.10.0,<0.11 (`sources/sibyl/sibyl-memory-hermes/hermes-catalog/plugin.yaml:10`). Account API/server revisions are not pinned by those ranges. Evidence: `sources/sibyl/sibyl-memory-hermes/hermes-catalog/plugin.yaml:10`.
- `tests.count`: `2495` → 172 adapter-related declarations using the stated parser/scope; B2 counted a different, repository-wide inventory. This is a scope refinement, not a count of executed or passing tests. There is no adapter forget-path test because no forget tool is exported. Engine delete/edge-cascade tests are separately inventoried. Adapter tests include signed-out refusals, not only import success (`sources/sibyl/sibyl-memory-hermes/tests/test_account_gate_2026_09_27.py:38`). Engine test deletes an entity and asserts that touching graph edges cascade while an unrelated edge survives. (`sources/sibyl/sibyl-memory-client/tests/test_links_2026_10_22.py:265`). Evidence: `sources/sibyl/sibyl-memory-hermes/tests/test_account_gate_2026_09_27.py:38`, `sources/sibyl/sibyl-memory-client/tests/test_links_2026_10_22.py:265`.

MATRIX counts these 6 explicit field corrections/refinements. Sections 1-6 provide the body and test inventories. Test-name candidates are not passing tests or measured coverage.
