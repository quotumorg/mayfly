# mem0: B3 static correctness review

Snapshot `43849c6e9d4d773b83a8144b6373aedc67241a5d` (MANIFEST.json row `mem0`). No plugin execution or passing-test claims.

## 1. Architecture

`provider`. The native adapter imports shared backend constructors and selects Platform, self-hosted HTTP, or in-process OSS Memory. Threads do recall and capture; OSS is not a mandatory separate daemon. Ownership is checked before ID mutations. `sources/mem0/integrations/hermes-plugin-mem0/__init__.py:352`

Hook/body inventory (no-op/fallback declarations are not active behavior):

| callable | declaration | first executable body line |
|---|---|---|
| `register` | `sources/mem0/integrations/hermes-plugin-mem0/__init__.py:423` | `sources/mem0/integrations/hermes-plugin-mem0/__init__.py:425` |
| `is_available` | `sources/mem0/integrations/hermes-plugin-mem0/__init__.py:144` | `sources/mem0/integrations/hermes-plugin-mem0/__init__.py:145` |
| `save_config` | `sources/mem0/integrations/hermes-plugin-mem0/__init__.py:150` | `sources/mem0/integrations/hermes-plugin-mem0/__init__.py:152` |
| `get_config_schema` | `sources/mem0/integrations/hermes-plugin-mem0/__init__.py:155` | `sources/mem0/integrations/hermes-plugin-mem0/__init__.py:156` |
| `initialize` | `sources/mem0/integrations/hermes-plugin-mem0/__init__.py:228` | `sources/mem0/integrations/hermes-plugin-mem0/__init__.py:229` |
| `system_prompt_block` | `sources/mem0/integrations/hermes-plugin-mem0/__init__.py:257` | `sources/mem0/integrations/hermes-plugin-mem0/__init__.py:259` |
| `on_turn_start` | `sources/mem0/integrations/hermes-plugin-mem0/__init__.py:263` | `sources/mem0/integrations/hermes-plugin-mem0/__init__.py:264` |
| `prefetch` | `sources/mem0/integrations/hermes-plugin-mem0/__init__.py:294` | `sources/mem0/integrations/hermes-plugin-mem0/__init__.py:296` |
| `sync_turn` | `sources/mem0/integrations/hermes-plugin-mem0/__init__.py:305` | `sources/mem0/integrations/hermes-plugin-mem0/__init__.py:307` |
| `get_tool_schemas` | `sources/mem0/integrations/hermes-plugin-mem0/__init__.py:328` | `sources/mem0/integrations/hermes-plugin-mem0/__init__.py:329` |
| `handle_tool_call` | `sources/mem0/integrations/hermes-plugin-mem0/__init__.py:376` | `sources/mem0/integrations/hermes-plugin-mem0/__init__.py:377` |
| `shutdown` | `sources/mem0/integrations/hermes-plugin-mem0/__init__.py:414` | `sources/mem0/integrations/hermes-plugin-mem0/__init__.py:415` |

## 2. Forget paths and derived copies

`yes`: `mem0_delete` calls `_backend.delete(memory_id)` after the ownership check (body `sources/mem0/integrations/hermes-plugin-mem0/__init__.py:366`). `mem0_update` replaces text, not erasure (`sources/mem0/integrations/hermes-plugin-mem0/__init__.py:363`). OSS `Memory.delete` checks existence then calls `_delete_memory` (`sources/mem0/mem0/memory/main.py:1878`); `delete_all` iterates scoped batches (`sources/mem0/mem0/memory/main.py:1923`); `reset` resets history and vector/entity stores (`sources/mem0/mem0/memory/main.py:2139`). Async counterparts and HTTP/SDK surfaces are catalogued below. Single delete removes the vector, records the old text in history, and attempts entity-link cleanup (`sources/mem0/mem0/memory/main.py:2111`). Thus `partial`: history deliberately retains the fact; no host purge is called. Graph-wide deletion is not established by the single-vector path.

Host state.db is independent (`hermes-agent/hermes_state.py:181`). No reviewed adapter forget body performs a general purge of that database or built-in MEMORY.md/USER.md. Prompt/tool-result copies and independent source transcripts remain separate; source operation bodies below establish the scope of each plugin mutation.

<details>
<summary>Deletion, invalidation, archive and reset body ledger</summary>

Public routes, SDK/CLI dispatch and internal storage operations are distinguished by their source. Cache invalidation and management-object deletion are not additional fact-forget tools. A stub is not working deletion. Call-through means the downstream body still determines erasure. HTTP prefixes/selectors must be read with route registration.

**delete**, body `sources/mem0/cli/python/src/mem0_cli/app.py:671`; implementation/call-through.

- `sources/mem0/cli/python/src/mem0_cli/app.py:616` `@app.command(rich_help_panel="Memory")`
- `sources/mem0/cli/python/src/mem0_cli/app.py:617` `def delete(`
- `sources/mem0/cli/python/src/mem0_cli/app.py:688` `_fire_telemetry("delete", {"delete_mode": "single"})`
- `sources/mem0/cli/python/src/mem0_cli/app.py:702` `_fire_telemetry("delete", {"delete_mode": "all"})`
- `sources/mem0/cli/python/src/mem0_cli/app.py:710` `_fire_telemetry("delete", {"delete_mode": "entity"})`

**entity_delete**, body `sources/mem0/cli/python/src/mem0_cli/app.py:843`; implementation/call-through.

- `sources/mem0/cli/python/src/mem0_cli/app.py:809` `@entity_app.command("delete")`
- `sources/mem0/cli/python/src/mem0_cli/app.py:810` `def entity_delete(`
- `sources/mem0/cli/python/src/mem0_cli/app.py:843` `from mem0_cli.commands.entities import cmd_entities_delete`

**delete**, body `sources/mem0/cli/python/src/mem0_cli/backend/base.py:97`; stub.

- `sources/mem0/cli/python/src/mem0_cli/backend/base.py:86` `@abstractmethod`
- `sources/mem0/cli/python/src/mem0_cli/backend/base.py:87` `def delete(`
- `sources/mem0/cli/python/src/mem0_cli/backend/base.py:97` `) -> dict: ...`

**delete_entities**, body `sources/mem0/cli/python/src/mem0_cli/backend/base.py:107`; stub.

- `sources/mem0/cli/python/src/mem0_cli/backend/base.py:99` `@abstractmethod`
- `sources/mem0/cli/python/src/mem0_cli/backend/base.py:100` `def delete_entities(`
- `sources/mem0/cli/python/src/mem0_cli/backend/base.py:107` `) -> dict: ...`

**delete**, body `sources/mem0/cli/python/src/mem0_cli/backend/platform.py:317`; implementation/call-through.

- `sources/mem0/cli/python/src/mem0_cli/backend/platform.py:306` `def delete(`
- `sources/mem0/cli/python/src/mem0_cli/backend/platform.py:327` `return self._request("DELETE", "/v1/memories/", params=params)`
- `sources/mem0/cli/python/src/mem0_cli/backend/platform.py:333` `"DELETE",`

**delete_entities**, body `sources/mem0/cli/python/src/mem0_cli/backend/platform.py:349`; implementation/call-through.

- `sources/mem0/cli/python/src/mem0_cli/backend/platform.py:340` `def delete_entities(`
- `sources/mem0/cli/python/src/mem0_cli/backend/platform.py:364` `"DELETE",`

**cmd_entities_delete**, body `sources/mem0/cli/python/src/mem0_cli/commands/entities.py:95`; implementation/call-through.

- `sources/mem0/cli/python/src/mem0_cli/commands/entities.py:83` `def cmd_entities_delete(`
- `sources/mem0/cli/python/src/mem0_cli/commands/entities.py:136` `result = backend.delete_entities(`

**cmd_delete**, body `sources/mem0/cli/python/src/mem0_cli/commands/memory.py:545`; implementation/call-through.

- `sources/mem0/cli/python/src/mem0_cli/commands/memory.py:535` `def cmd_delete(`
- `sources/mem0/cli/python/src/mem0_cli/commands/memory.py:547` `set_current_command("delete")`
- `sources/mem0/cli/python/src/mem0_cli/commands/memory.py:564` `result = backend.delete(memory_id=memory_id, delete_linked=delete_linked)`
- `sources/mem0/cli/python/src/mem0_cli/commands/memory.py:573` `command="delete",`

**cmd_delete_all**, body `sources/mem0/cli/python/src/mem0_cli/commands/memory.py:596`; implementation/call-through.

- `sources/mem0/cli/python/src/mem0_cli/commands/memory.py:583` `def cmd_delete_all(`
- `sources/mem0/cli/python/src/mem0_cli/commands/memory.py:619` `result = backend.delete(`
- `sources/mem0/cli/python/src/mem0_cli/commands/memory.py:684` `result = backend.delete(`

**_tool_delete**, body `sources/mem0/integrations/hermes-plugin-mem0/__init__.py:366`; implementation/call-through.

- `sources/mem0/integrations/hermes-plugin-mem0/__init__.py:365` `def _tool_delete(self, args: dict) -> str:`
- `sources/mem0/integrations/hermes-plugin-mem0/__init__.py:367` `return json.dumps(self._backend.delete(args["memory_id"]))`

**_delete**, body `sources/mem0/integrations/hermes-plugin-mem0/_backend.py:39`; stub.

- `sources/mem0/integrations/hermes-plugin-mem0/_backend.py:38` `@abstractmethod`
- `sources/mem0/integrations/hermes-plugin-mem0/_backend.py:39` `def _delete(self, memory_id: str) -> None: ...`

**delete**, body `sources/mem0/integrations/hermes-plugin-mem0/_backend.py:46`; implementation/call-through.

- `sources/mem0/integrations/hermes-plugin-mem0/_backend.py:45` `def delete(self, memory_id: str) -> dict:`
- `sources/mem0/integrations/hermes-plugin-mem0/_backend.py:46` `self._delete(memory_id)`

**_delete**, body `sources/mem0/integrations/hermes-plugin-mem0/_backend.py:73`; implementation/call-through.

- `sources/mem0/integrations/hermes-plugin-mem0/_backend.py:72` `def _delete(self, memory_id: str) -> None:`
- `sources/mem0/integrations/hermes-plugin-mem0/_backend.py:73` `self._client.delete(memory_id=memory_id)`

**_delete**, body `sources/mem0/integrations/hermes-plugin-mem0/_backend.py:109`; implementation/call-through.

- `sources/mem0/integrations/hermes-plugin-mem0/_backend.py:108` `def _delete(self, memory_id: str) -> None:`
- `sources/mem0/integrations/hermes-plugin-mem0/_backend.py:109` `self._json("DELETE", f"/memories/{memory_id}")`

**_delete**, body `sources/mem0/integrations/hermes-plugin-mem0/_backend.py:300`; implementation/call-through.

- `sources/mem0/integrations/hermes-plugin-mem0/_backend.py:299` `def _delete(self, memory_id: str) -> None:`
- `sources/mem0/integrations/hermes-plugin-mem0/_backend.py:300` `self._call("delete", memory_id)`

**delete**, body `sources/mem0/mem0/client/main.py:526`; implementation/call-through.

- `sources/mem0/mem0/client/main.py:503` `@api_error_handler`
- `sources/mem0/mem0/client/main.py:504` `def delete(self, memory_id: str, delete_linked: bool = False) -> Dict[str, Any]:`
- `sources/mem0/mem0/client/main.py:527` `response = self.client.delete(f"/v1/memories/{_encode_path_segment(memory_id)}/", params=params)`
- `sources/mem0/mem0/client/main.py:530` `"client.delete", self, {"memory_id": memory_id, "delete_linked": delete_linked, "sync_type": "sync"}`

**delete_all**, body `sources/mem0/mem0/client/main.py:554`; implementation/call-through.

- `sources/mem0/mem0/client/main.py:534` `@api_error_handler`
- `sources/mem0/mem0/client/main.py:535` `def delete_all(self, options: Optional[DeleteAllMemoryOptions] = None, **kwargs) -> Dict[str, str]:`
- `sources/mem0/mem0/client/main.py:556` `response = self.client.delete("/v1/memories/", params=params)`
- `sources/mem0/mem0/client/main.py:559` `"client.delete_all",`

**delete_users**, body `sources/mem0/mem0/client/main.py:625`; implementation/call-through.

- `sources/mem0/mem0/client/main.py:598` `@api_error_handler`
- `sources/mem0/mem0/client/main.py:599` `def delete_users(`
- `sources/mem0/mem0/client/main.py:645` `response = self.client.delete(`
- `sources/mem0/mem0/client/main.py:652` `"client.delete_users",`

**reset**, body `sources/mem0/mem0/client/main.py:686`; implementation/call-through.

- `sources/mem0/mem0/client/main.py:668` `@api_error_handler`
- `sources/mem0/mem0/client/main.py:669` `def reset(self) -> Dict[str, str]:`
- `sources/mem0/mem0/client/main.py:686` `self.delete_users()`

**batch_delete**, body `sources/mem0/mem0/client/main.py:738`; implementation/call-through.

- `sources/mem0/mem0/client/main.py:718` `@api_error_handler`
- `sources/mem0/mem0/client/main.py:719` `def batch_delete(self, memories: List[Dict[str, Any]]) -> Dict[str, Any]:`
- `sources/mem0/mem0/client/main.py:738` `response = self.client.request("DELETE", "/v1/batch/", json={"memories": memories})`

**delete_webhook**, body `sources/mem0/mem0/client/main.py:1199`; implementation/call-through.

- `sources/mem0/mem0/client/main.py:1180` `@api_error_handler`
- `sources/mem0/mem0/client/main.py:1181` `def delete_webhook(self, webhook_id: int) -> Dict[str, str]:`
- `sources/mem0/mem0/client/main.py:1199` `response = self.client.delete(f"api/v1/webhooks/{webhook_id}/")`
- `sources/mem0/mem0/client/main.py:1202` `"client.delete_webhook",`

**delete**, body `sources/mem0/mem0/client/main.py:1607`; implementation/call-through.

- `sources/mem0/mem0/client/main.py:1584` `@api_error_handler`
- `sources/mem0/mem0/client/main.py:1585` `async def delete(self, memory_id: str, delete_linked: bool = False) -> Dict[str, Any]:`
- `sources/mem0/mem0/client/main.py:1608` `response = await self.async_client.delete(f"/v1/memories/{_encode_path_segment(memory_id)}/", params=params)`
- `sources/mem0/mem0/client/main.py:1611` `"client.delete", self, {"memory_id": memory_id, "delete_linked": delete_linked, "sync_type": "async"}`

**delete_all**, body `sources/mem0/mem0/client/main.py:1634`; implementation/call-through.

- `sources/mem0/mem0/client/main.py:1615` `@api_error_handler`
- `sources/mem0/mem0/client/main.py:1616` `async def delete_all(self, options: Optional[DeleteAllMemoryOptions] = None, **kwargs) -> Dict[str, str]:`
- `sources/mem0/mem0/client/main.py:1636` `response = await self.async_client.delete("/v1/memories/", params=params)`
- `sources/mem0/mem0/client/main.py:1638` `capture_client_event("client.delete_all", self, {"keys": list(kwargs.keys()), "sync_type": "async"})`

**delete_users**, body `sources/mem0/mem0/client/main.py:1704`; implementation/call-through.

- `sources/mem0/mem0/client/main.py:1677` `@api_error_handler`
- `sources/mem0/mem0/client/main.py:1678` `async def delete_users(`
- `sources/mem0/mem0/client/main.py:1724` `response = await self.async_client.delete(`
- `sources/mem0/mem0/client/main.py:1731` `"client.delete_users",`

**reset**, body `sources/mem0/mem0/client/main.py:1765`; implementation/call-through.

- `sources/mem0/mem0/client/main.py:1747` `@api_error_handler`
- `sources/mem0/mem0/client/main.py:1748` `async def reset(self) -> Dict[str, str]:`
- `sources/mem0/mem0/client/main.py:1765` `await self.delete_users()`

**batch_delete**, body `sources/mem0/mem0/client/main.py:1816`; implementation/call-through.

- `sources/mem0/mem0/client/main.py:1796` `@api_error_handler`
- `sources/mem0/mem0/client/main.py:1797` `async def batch_delete(self, memories: List[Dict[str, Any]]) -> Dict[str, Any]:`
- `sources/mem0/mem0/client/main.py:1816` `response = await self.async_client.request("DELETE", "/v1/batch/", json={"memories": memories})`

**delete_webhook**, body `sources/mem0/mem0/client/main.py:2262`; implementation/call-through.

- `sources/mem0/mem0/client/main.py:2243` `@api_error_handler`
- `sources/mem0/mem0/client/main.py:2244` `async def delete_webhook(self, webhook_id: int) -> Dict[str, str]:`
- `sources/mem0/mem0/client/main.py:2262` `response = await self.async_client.delete(f"api/v1/webhooks/{webhook_id}/")`
- `sources/mem0/mem0/client/main.py:2264` `capture_client_event("client.delete_webhook", self, {"webhook_id": webhook_id, "sync_type": "async"})`

**delete**, body `sources/mem0/mem0/client/project.py:217`; stub.

- `sources/mem0/mem0/client/project.py:202` `@abstractmethod`
- `sources/mem0/mem0/client/project.py:203` `def delete(self) -> Dict[str, Any]:`
- `sources/mem0/mem0/client/project.py:217` `pass`

**delete**, body `sources/mem0/mem0/client/project.py:481`; implementation/call-through.

- `sources/mem0/mem0/client/project.py:466` `@api_error_handler`
- `sources/mem0/mem0/client/project.py:467` `def delete(self) -> Dict[str, Any]:`
- `sources/mem0/mem0/client/project.py:481` `response = self._client.delete(`
- `sources/mem0/mem0/client/project.py:486` `"client.project.delete",`

**delete**, body `sources/mem0/mem0/client/project.py:805`; implementation/call-through.

- `sources/mem0/mem0/client/project.py:790` `@api_error_handler`
- `sources/mem0/mem0/client/project.py:791` `async def delete(self) -> Dict[str, Any]:`
- `sources/mem0/mem0/client/project.py:805` `response = await self._client.delete(`
- `sources/mem0/mem0/client/project.py:810` `"client.project.delete",`

**delete**, body `sources/mem0/mem0/memory/base.py:50`; stub.

- `sources/mem0/mem0/memory/base.py:42` `@abstractmethod`
- `sources/mem0/mem0/memory/base.py:43` `def delete(self, memory_id):`
- `sources/mem0/mem0/memory/base.py:50` `pass`

**delete**, body `sources/mem0/mem0/memory/main.py:1876`; implementation/call-through.

- `sources/mem0/mem0/memory/main.py:1869` `def delete(self, memory_id):`
- `sources/mem0/mem0/memory/main.py:1876` `capture_event("mem0.delete", self, {"memory_id": memory_id, "sync_type": "sync"})`
- `sources/mem0/mem0/memory/main.py:1885` `display_decay_usage_notice(self, "sync", "delete", *decay_usage_notice)`
- `sources/mem0/mem0/memory/main.py:1887` `display_first_run_notice(self, "sync", "delete")`

**delete_all**, body `sources/mem0/mem0/memory/main.py:1899`; implementation/call-through.

- `sources/mem0/mem0/memory/main.py:1890` `def delete_all(self, user_id: Optional[str] = None, agent_id: Optional[str] = None, run_id: Optional[str] = None):`
- `sources/mem0/mem0/memory/main.py:1917` `capture_event("mem0.delete_all", self, {"keys": keys, "encoded_ids": encoded_ids, "sync_type": "sync"})`

**_delete_memory**, body `sources/mem0/mem0/memory/main.py:2101`; implementation/call-through.

- `sources/mem0/mem0/memory/main.py:2100` `def _delete_memory(self, memory_id, existing_memory=None):`
- `sources/mem0/mem0/memory/main.py:2111` `self.vector_store.delete(vector_id=memory_id)`
- `sources/mem0/mem0/memory/main.py:2116` `"DELETE",`

**reset**, body `sources/mem0/mem0/memory/main.py:2137`; implementation/call-through.

- `sources/mem0/mem0/memory/main.py:2130` `def reset(self):`
- `sources/mem0/mem0/memory/main.py:2147` `self.vector_store.delete_col()`

**delete**, body `sources/mem0/mem0/memory/main.py:3532`; implementation/call-through.

- `sources/mem0/mem0/memory/main.py:3525` `async def delete(self, memory_id):`
- `sources/mem0/mem0/memory/main.py:3532` `capture_event("mem0.delete", self, {"memory_id": memory_id, "sync_type": "async"})`
- `sources/mem0/mem0/memory/main.py:3541` `await display_decay_usage_notice_async(self, "async", "delete", *decay_usage_notice)`
- `sources/mem0/mem0/memory/main.py:3543` `await display_first_run_notice_async(self, "async", "delete")`

**delete_all**, body `sources/mem0/mem0/memory/main.py:3555`; implementation/call-through.

- `sources/mem0/mem0/memory/main.py:3546` `async def delete_all(self, user_id=None, agent_id=None, run_id=None):`
- `sources/mem0/mem0/memory/main.py:3573` `capture_event("mem0.delete_all", self, {"keys": keys, "encoded_ids": encoded_ids, "sync_type": "async"})`

**_delete_memory**, body `sources/mem0/mem0/memory/main.py:3798`; implementation/call-through.

- `sources/mem0/mem0/memory/main.py:3797` `async def _delete_memory(self, memory_id, existing_memory=None, skip_entity_cleanup=False):`
- `sources/mem0/mem0/memory/main.py:3809` `await asyncio.to_thread(self.vector_store.delete, vector_id=memory_id)`
- `sources/mem0/mem0/memory/main.py:3815` `"DELETE",`

**reset**, body `sources/mem0/mem0/memory/main.py:3835`; implementation/call-through.

- `sources/mem0/mem0/memory/main.py:3828` `async def reset(self):`
- `sources/mem0/mem0/memory/main.py:3836` `await asyncio.to_thread(self.vector_store.delete_col)`

**detect_decay_usage_from_delete**, body `sources/mem0/mem0/memory/notices.py:342`; implementation/call-through.

- `sources/mem0/mem0/memory/notices.py:341` `def detect_decay_usage_from_delete() -> Optional[Tuple[str, str, Optional[int], Optional[int]]]:`
- `sources/mem0/mem0/memory/notices.py:342` `if not telemetry_module.MEM0_TELEMETRY:`

**detect_decay_usage_from_delete_all**, body `sources/mem0/mem0/memory/notices.py:362`; implementation/call-through.

- `sources/mem0/mem0/memory/notices.py:361` `def detect_decay_usage_from_delete_all(deleted_count: Any) -> Optional[Tuple[str, str, Optional[int], Optional[int]]]:`
- `sources/mem0/mem0/memory/notices.py:362` `if not telemetry_module.MEM0_TELEMETRY:`

**reset**, body `sources/mem0/mem0/memory/storage.py:328`; implementation/call-through.

- `sources/mem0/mem0/memory/storage.py:326` `def reset(self) -> None:`
- `sources/mem0/mem0/memory/storage.py:328` `if not self.connection:`

**reset**, body `sources/mem0/mem0/utils/factory.py:222`; implementation/call-through.

- `sources/mem0/mem0/utils/factory.py:220` `@classmethod`
- `sources/mem0/mem0/utils/factory.py:221` `def reset(cls, instance):`
- `sources/mem0/mem0/utils/factory.py:222` `instance.reset()`

**delete**, body `sources/mem0/mem0/vector_stores/azure_ai_search.py:291`; implementation/call-through.

- `sources/mem0/mem0/vector_stores/azure_ai_search.py:284` `def delete(self, vector_id):`
- `sources/mem0/mem0/vector_stores/azure_ai_search.py:291` `response = self.search_client.delete_documents(documents=[{"id": vector_id}])`

**delete_col**, body `sources/mem0/mem0/vector_stores/azure_ai_search.py:353`; implementation/call-through.

- `sources/mem0/mem0/vector_stores/azure_ai_search.py:351` `def delete_col(self):`
- `sources/mem0/mem0/vector_stores/azure_ai_search.py:353` `self.index_client.delete_index(self.index_name)`

**reset**, body `sources/mem0/mem0/vector_stores/azure_ai_search.py:394`; implementation/call-through.

- `sources/mem0/mem0/vector_stores/azure_ai_search.py:392` `def reset(self):`
- `sources/mem0/mem0/vector_stores/azure_ai_search.py:402` `self.delete_col()`

**delete**, body `sources/mem0/mem0/vector_stores/azure_mysql.py:399`; implementation/call-through.

- `sources/mem0/mem0/vector_stores/azure_mysql.py:392` `def delete(self, vector_id: str):`
- `sources/mem0/mem0/vector_stores/azure_mysql.py:400` `cur.execute(f"DELETE FROM '{self.collection_name}' WHERE id = %s", (vector_id,))`

**delete_col**, body `sources/mem0/mem0/vector_stores/azure_mysql.py:465`; implementation/call-through.

- `sources/mem0/mem0/vector_stores/azure_mysql.py:463` `def delete_col(self):`
- `sources/mem0/mem0/vector_stores/azure_mysql.py:465` `with self._get_cursor(commit=True) as cur:`

**reset**, body `sources/mem0/mem0/vector_stores/azure_mysql.py:545`; implementation/call-through.

- `sources/mem0/mem0/vector_stores/azure_mysql.py:543` `def reset(self):`
- `sources/mem0/mem0/vector_stores/azure_mysql.py:546` `self.delete_col()`

**delete**, body `sources/mem0/mem0/vector_stores/baidu.py:290`; implementation/call-through.

- `sources/mem0/mem0/vector_stores/baidu.py:283` `def delete(self, vector_id):`
- `sources/mem0/mem0/vector_stores/baidu.py:290` `self._table.delete(primary_key={"id": vector_id})`

**delete_col**, body `sources/mem0/mem0/vector_stores/baidu.py:331`; implementation/call-through.

- `sources/mem0/mem0/vector_stores/baidu.py:329` `def delete_col(self):`
- `sources/mem0/mem0/vector_stores/baidu.py:331` `try:`

**reset**, body `sources/mem0/mem0/vector_stores/baidu.py:393`; implementation/call-through.

- `sources/mem0/mem0/vector_stores/baidu.py:391` `def reset(self):`
- `sources/mem0/mem0/vector_stores/baidu.py:395` `self.delete_col()`

**delete**, body `sources/mem0/mem0/vector_stores/base.py:31`; stub.

- `sources/mem0/mem0/vector_stores/base.py:28` `@abstractmethod`
- `sources/mem0/mem0/vector_stores/base.py:29` `def delete(self, vector_id):`
- `sources/mem0/mem0/vector_stores/base.py:31` `pass`

**delete_col**, body `sources/mem0/mem0/vector_stores/base.py:51`; stub.

- `sources/mem0/mem0/vector_stores/base.py:48` `@abstractmethod`
- `sources/mem0/mem0/vector_stores/base.py:49` `def delete_col(self):`
- `sources/mem0/mem0/vector_stores/base.py:51` `pass`

**reset**, body `sources/mem0/mem0/vector_stores/base.py:66`; stub.

- `sources/mem0/mem0/vector_stores/base.py:63` `@abstractmethod`
- `sources/mem0/mem0/vector_stores/base.py:64` `def reset(self):`
- `sources/mem0/mem0/vector_stores/base.py:66` `pass`

**delete**, body `sources/mem0/mem0/vector_stores/cassandra.py:298`; implementation/call-through.

- `sources/mem0/mem0/vector_stores/cassandra.py:291` `def delete(self, vector_id: str):`
- `sources/mem0/mem0/vector_stores/cassandra.py:300` `DELETE FROM {self.keyspace}.{self.collection_name}`

**delete_col**, body `sources/mem0/mem0/vector_stores/cassandra.py:398`; implementation/call-through.

- `sources/mem0/mem0/vector_stores/cassandra.py:396` `def delete_col(self):`
- `sources/mem0/mem0/vector_stores/cassandra.py:398` `try:`

**reset**, body `sources/mem0/mem0/vector_stores/cassandra.py:484`; implementation/call-through.

- `sources/mem0/mem0/vector_stores/cassandra.py:482` `def reset(self):`
- `sources/mem0/mem0/vector_stores/cassandra.py:484` `try:`

**delete**, body `sources/mem0/mem0/vector_stores/chroma.py:172`; implementation/call-through.

- `sources/mem0/mem0/vector_stores/chroma.py:165` `def delete(self, vector_id: str):`
- `sources/mem0/mem0/vector_stores/chroma.py:172` `self.collection.delete(ids=[vector_id])`

**delete_col**, body `sources/mem0/mem0/vector_stores/chroma.py:221`; implementation/call-through.

- `sources/mem0/mem0/vector_stores/chroma.py:217` `def delete_col(self):`
- `sources/mem0/mem0/vector_stores/chroma.py:221` `self.client.delete_collection(name=self.collection_name)`

**reset**, body `sources/mem0/mem0/vector_stores/chroma.py:249`; implementation/call-through.

- `sources/mem0/mem0/vector_stores/chroma.py:247` `def reset(self):`
- `sources/mem0/mem0/vector_stores/chroma.py:250` `self.delete_col()`

**delete**, body `sources/mem0/mem0/vector_stores/databricks.py:580`; implementation/call-through.

- `sources/mem0/mem0/vector_stores/databricks.py:573` `def delete(self, vector_id):`
- `sources/mem0/mem0/vector_stores/databricks.py:583` `delete_sql = f"DELETE FROM {self.fully_qualified_table_name} WHERE memory_id = :vector_id"`

**delete_col**, body `sources/mem0/mem0/vector_stores/databricks.py:755`; implementation/call-through.

- `sources/mem0/mem0/vector_stores/databricks.py:751` `def delete_col(self):`
- `sources/mem0/mem0/vector_stores/databricks.py:758` `self.client.vector_search_indexes.delete_index(index_name=self.fully_qualified_index_name)`
- `sources/mem0/mem0/vector_stores/databricks.py:761` `self.client.vector_search_indexes.delete_index(index_name=self.index_name)`

**reset**, body `sources/mem0/mem0/vector_stores/databricks.py:845`; implementation/call-through.

- `sources/mem0/mem0/vector_stores/databricks.py:838` `def reset(self):`
- `sources/mem0/mem0/vector_stores/databricks.py:850` `self.client.vector_search_indexes.delete_index(index_name=fq_index)`
- `sources/mem0/mem0/vector_stores/databricks.py:856` `self.delete_col()`

**delete**, body `sources/mem0/mem0/vector_stores/elasticsearch.py:234`; implementation/call-through.

- `sources/mem0/mem0/vector_stores/elasticsearch.py:232` `def delete(self, vector_id: str) -> None:`
- `sources/mem0/mem0/vector_stores/elasticsearch.py:234` `self.client.delete(index=self.collection_name, id=vector_id)`

**delete_col**, body `sources/mem0/mem0/vector_stores/elasticsearch.py:271`; implementation/call-through.

- `sources/mem0/mem0/vector_stores/elasticsearch.py:269` `def delete_col(self) -> None:`
- `sources/mem0/mem0/vector_stores/elasticsearch.py:271` `self.client.indices.delete(index=self.collection_name)`

**reset**, body `sources/mem0/mem0/vector_stores/elasticsearch.py:307`; implementation/call-through.

- `sources/mem0/mem0/vector_stores/elasticsearch.py:305` `def reset(self):`
- `sources/mem0/mem0/vector_stores/elasticsearch.py:308` `self.delete_col()`

**delete**, body `sources/mem0/mem0/vector_stores/faiss.py:452`; implementation/call-through.

- `sources/mem0/mem0/vector_stores/faiss.py:445` `def delete(self, vector_id: str):`
- `sources/mem0/mem0/vector_stores/faiss.py:452` `if self.index is None:`

**delete_col**, body `sources/mem0/mem0/vector_stores/faiss.py:568`; implementation/call-through.

- `sources/mem0/mem0/vector_stores/faiss.py:564` `def delete_col(self):`
- `sources/mem0/mem0/vector_stores/faiss.py:575` `os.remove(index_path)`
- `sources/mem0/mem0/vector_stores/faiss.py:577` `os.remove(json_docstore_path)`
- `sources/mem0/mem0/vector_stores/faiss.py:580` `os.remove(pkl_docstore_path)`

**reset**, body `sources/mem0/mem0/vector_stores/faiss.py:646`; implementation/call-through.

- `sources/mem0/mem0/vector_stores/faiss.py:644` `def reset(self):`
- `sources/mem0/mem0/vector_stores/faiss.py:647` `self.delete_col()`

**delete**, body `sources/mem0/mem0/vector_stores/langchain.py:147`; implementation/call-through.

- `sources/mem0/mem0/vector_stores/langchain.py:143` `def delete(self, vector_id):`
- `sources/mem0/mem0/vector_stores/langchain.py:147` `self.client.delete(ids=[vector_id])`

**delete_col**, body `sources/mem0/mem0/vector_stores/langchain.py:177`; implementation/call-through.

- `sources/mem0/mem0/vector_stores/langchain.py:173` `def delete_col(self):`
- `sources/mem0/mem0/vector_stores/langchain.py:179` `self.client.delete_collection()`
- `sources/mem0/mem0/vector_stores/langchain.py:183` `self.client.delete(ids=None)`

**reset**, body `sources/mem0/mem0/vector_stores/langchain.py:215`; implementation/call-through.

- `sources/mem0/mem0/vector_stores/langchain.py:213` `def reset(self):`
- `sources/mem0/mem0/vector_stores/langchain.py:216` `self.delete_col()`

**delete**, body `sources/mem0/mem0/vector_stores/milvus.py:279`; implementation/call-through.

- `sources/mem0/mem0/vector_stores/milvus.py:272` `def delete(self, vector_id):`
- `sources/mem0/mem0/vector_stores/milvus.py:279` `self.client.delete(collection_name=self.collection_name, ids=[vector_id])`

**delete_col**, body `sources/mem0/mem0/vector_stores/milvus.py:340`; implementation/call-through.

- `sources/mem0/mem0/vector_stores/milvus.py:338` `def delete_col(self):`
- `sources/mem0/mem0/vector_stores/milvus.py:340` `return self.client.drop_collection(collection_name=self.collection_name)`

**reset**, body `sources/mem0/mem0/vector_stores/milvus.py:372`; implementation/call-through.

- `sources/mem0/mem0/vector_stores/milvus.py:370` `def reset(self):`
- `sources/mem0/mem0/vector_stores/milvus.py:373` `self.delete_col()`

**delete**, body `sources/mem0/mem0/vector_stores/mongodb.py:287`; implementation/call-through.

- `sources/mem0/mem0/vector_stores/mongodb.py:280` `def delete(self, vector_id: str) -> None:`
- `sources/mem0/mem0/vector_stores/mongodb.py:288` `result = self.collection.delete_one({"_id": vector_id})`
- `sources/mem0/mem0/vector_stores/mongodb.py:289` `if result.deleted_count > 0:`

**delete_col**, body `sources/mem0/mem0/vector_stores/mongodb.py:361`; implementation/call-through.

- `sources/mem0/mem0/vector_stores/mongodb.py:359` `def delete_col(self) -> None:`
- `sources/mem0/mem0/vector_stores/mongodb.py:361` `try:`

**reset**, body `sources/mem0/mem0/vector_stores/mongodb.py:417`; implementation/call-through.

- `sources/mem0/mem0/vector_stores/mongodb.py:415` `def reset(self):`
- `sources/mem0/mem0/vector_stores/mongodb.py:418` `self.delete_col()`

**delete**, body `sources/mem0/mem0/vector_stores/neptune_analytics.py:210`; implementation/call-through.

- `sources/mem0/mem0/vector_stores/neptune_analytics.py:201` `def delete(self, vector_id: str):`
- `sources/mem0/mem0/vector_stores/neptune_analytics.py:210` `params = dict(node_id=vector_id)`

**delete_col**, body `sources/mem0/mem0/vector_stores/neptune_analytics.py:363`; implementation/call-through.

- `sources/mem0/mem0/vector_stores/neptune_analytics.py:356` `def delete_col(self):`
- `sources/mem0/mem0/vector_stores/neptune_analytics.py:363` `self.execute_query(f"MATCH (n :{self.collection_name}) DETACH DELETE n")`

**reset**, body `sources/mem0/mem0/vector_stores/neptune_analytics.py:414`; implementation/call-through.

- `sources/mem0/mem0/vector_stores/neptune_analytics.py:408` `def reset(self):`
- `sources/mem0/mem0/vector_stores/neptune_analytics.py:414` `self.delete_col()`

**delete**, body `sources/mem0/mem0/vector_stores/opensearch.py:306`; implementation/call-through.

- `sources/mem0/mem0/vector_stores/opensearch.py:303` `def delete(self, vector_id: str) -> None:`
- `sources/mem0/mem0/vector_stores/opensearch.py:317` `self.client.delete(index=self.collection_name, id=opensearch_id)`

**delete_col**, body `sources/mem0/mem0/vector_stores/opensearch.py:378`; implementation/call-through.

- `sources/mem0/mem0/vector_stores/opensearch.py:376` `def delete_col(self) -> None:`
- `sources/mem0/mem0/vector_stores/opensearch.py:378` `self.client.indices.delete(index=self.collection_name)`

**reset**, body `sources/mem0/mem0/vector_stores/opensearch.py:412`; implementation/call-through.

- `sources/mem0/mem0/vector_stores/opensearch.py:410` `def reset(self):`
- `sources/mem0/mem0/vector_stores/opensearch.py:413` `self.delete_col()`

**delete**, body `sources/mem0/mem0/vector_stores/oracledb.py:453`; implementation/call-through.

- `sources/mem0/mem0/vector_stores/oracledb.py:446` `def delete(self, vector_id: str) -> None:`
- `sources/mem0/mem0/vector_stores/oracledb.py:454` `cursor.execute(f"DELETE FROM {self.collection_name} WHERE id = :id", id=vector_id)`

**delete_col**, body `sources/mem0/mem0/vector_stores/oracledb.py:519`; implementation/call-through.

- `sources/mem0/mem0/vector_stores/oracledb.py:517` `def delete_col(self) -> None:`
- `sources/mem0/mem0/vector_stores/oracledb.py:519` `with self._get_cursor(commit=True) as cursor:`

**reset**, body `sources/mem0/mem0/vector_stores/oracledb.py:590`; implementation/call-through.

- `sources/mem0/mem0/vector_stores/oracledb.py:588` `def reset(self) -> None:`
- `sources/mem0/mem0/vector_stores/oracledb.py:591` `self.delete_col()`

**delete**, body `sources/mem0/mem0/vector_stores/pgvector.py:412`; implementation/call-through.

- `sources/mem0/mem0/vector_stores/pgvector.py:405` `def delete(self, vector_id: str) -> None:`
- `sources/mem0/mem0/vector_stores/pgvector.py:414` `cur.execute(sql.SQL("DELETE FROM {} WHERE id = %s").format(self._col()), (vector_id,))`

**delete_col**, body `sources/mem0/mem0/vector_stores/pgvector.py:487`; implementation/call-through.

- `sources/mem0/mem0/vector_stores/pgvector.py:485` `def delete_col(self) -> None:`
- `sources/mem0/mem0/vector_stores/pgvector.py:487` `with self._get_cursor(commit=True) as cur:`

**reset**, body `sources/mem0/mem0/vector_stores/pgvector.py:560`; implementation/call-through.

- `sources/mem0/mem0/vector_stores/pgvector.py:558` `def reset(self) -> None:`
- `sources/mem0/mem0/vector_stores/pgvector.py:562` `self.delete_col()`

**delete**, body `sources/mem0/mem0/vector_stores/pinecone.py:305`; implementation/call-through.

- `sources/mem0/mem0/vector_stores/pinecone.py:298` `def delete(self, vector_id: Union[str, int]):`
- `sources/mem0/mem0/vector_stores/pinecone.py:305` `self.index.delete(ids=[str(vector_id)], namespace=self.namespace)`

**delete_col**, body `sources/mem0/mem0/vector_stores/pinecone.py:362`; implementation/call-through.

- `sources/mem0/mem0/vector_stores/pinecone.py:360` `def delete_col(self):`
- `sources/mem0/mem0/vector_stores/pinecone.py:364` `self.index.delete(delete_all=True, namespace=self.namespace)`
- `sources/mem0/mem0/vector_stores/pinecone.py:367` `self.client.delete_index(self.collection_name)`

**reset**, body `sources/mem0/mem0/vector_stores/pinecone.py:437`; implementation/call-through.

- `sources/mem0/mem0/vector_stores/pinecone.py:433` `def reset(self):`
- `sources/mem0/mem0/vector_stores/pinecone.py:438` `self.delete_col()`

**delete**, body `sources/mem0/mem0/vector_stores/qdrant.py:493`; implementation/call-through.

- `sources/mem0/mem0/vector_stores/qdrant.py:486` `def delete(self, vector_id: int):`
- `sources/mem0/mem0/vector_stores/qdrant.py:493` `self.client.delete(`

**delete_col**, body `sources/mem0/mem0/vector_stores/qdrant.py:562`; implementation/call-through.

- `sources/mem0/mem0/vector_stores/qdrant.py:560` `def delete_col(self):`
- `sources/mem0/mem0/vector_stores/qdrant.py:562` `self.client.delete_collection(collection_name=self.collection_name)`

**reset**, body `sources/mem0/mem0/vector_stores/qdrant.py:596`; implementation/call-through.

- `sources/mem0/mem0/vector_stores/qdrant.py:594` `def reset(self):`
- `sources/mem0/mem0/vector_stores/qdrant.py:597` `self.delete_col()`
- `sources/mem0/mem0/vector_stores/qdrant.py:603` `self.client.delete(`

**delete**, body `sources/mem0/mem0/vector_stores/redis.py:246`; implementation/call-through.

- `sources/mem0/mem0/vector_stores/redis.py:245` `def delete(self, vector_id):`
- `sources/mem0/mem0/vector_stores/redis.py:246` `self.index.drop_keys(f"{self.schema['index']['prefix']}:{vector_id}")`

**delete_col**, body `sources/mem0/mem0/vector_stores/redis.py:301`; implementation/call-through.

- `sources/mem0/mem0/vector_stores/redis.py:300` `def delete_col(self):`
- `sources/mem0/mem0/vector_stores/redis.py:301` `self.index.delete()`

**reset**, body `sources/mem0/mem0/vector_stores/redis.py:310`; implementation/call-through.

- `sources/mem0/mem0/vector_stores/redis.py:306` `def reset(self):`
- `sources/mem0/mem0/vector_stores/redis.py:312` `self.delete_col()`

**delete**, body `sources/mem0/mem0/vector_stores/s3_vectors.py:120`; implementation/call-through.

- `sources/mem0/mem0/vector_stores/s3_vectors.py:119` `def delete(self, vector_id):`
- `sources/mem0/mem0/vector_stores/s3_vectors.py:120` `self.client.delete_vectors(`

**delete_col**, body `sources/mem0/mem0/vector_stores/s3_vectors.py:178`; implementation/call-through.

- `sources/mem0/mem0/vector_stores/s3_vectors.py:177` `def delete_col(self):`
- `sources/mem0/mem0/vector_stores/s3_vectors.py:178` `self.client.delete_index(vectorBucketName=self.vector_bucket_name, indexName=self.collection_name)`

**reset**, body `sources/mem0/mem0/vector_stores/s3_vectors.py:211`; implementation/call-through.

- `sources/mem0/mem0/vector_stores/s3_vectors.py:210` `def reset(self):`
- `sources/mem0/mem0/vector_stores/s3_vectors.py:212` `self.delete_col()`

**delete**, body `sources/mem0/mem0/vector_stores/supabase.py:159`; implementation/call-through.

- `sources/mem0/mem0/vector_stores/supabase.py:152` `def delete(self, vector_id: str):`
- `sources/mem0/mem0/vector_stores/supabase.py:159` `self.collection.delete([(vector_id,)])`

**delete_col**, body `sources/mem0/mem0/vector_stores/supabase.py:207`; implementation/call-through.

- `sources/mem0/mem0/vector_stores/supabase.py:205` `def delete_col(self):`
- `sources/mem0/mem0/vector_stores/supabase.py:207` `self.db.delete_collection(self.collection_name)`

**reset**, body `sources/mem0/mem0/vector_stores/supabase.py:257`; implementation/call-through.

- `sources/mem0/mem0/vector_stores/supabase.py:255` `def reset(self):`
- `sources/mem0/mem0/vector_stores/supabase.py:258` `self.delete_col()`

**delete**, body `sources/mem0/mem0/vector_stores/turbopuffer.py:194`; implementation/call-through.

- `sources/mem0/mem0/vector_stores/turbopuffer.py:187` `def delete(self, vector_id: Union[str, int]):`
- `sources/mem0/mem0/vector_stores/turbopuffer.py:194` `self.namespace.write(deletes=[str(vector_id)])`

**delete_col**, body `sources/mem0/mem0/vector_stores/turbopuffer.py:268`; implementation/call-through.

- `sources/mem0/mem0/vector_stores/turbopuffer.py:266` `def delete_col(self):`
- `sources/mem0/mem0/vector_stores/turbopuffer.py:269` `self.namespace.delete_all()`

**reset**, body `sources/mem0/mem0/vector_stores/turbopuffer.py:337`; implementation/call-through.

- `sources/mem0/mem0/vector_stores/turbopuffer.py:335` `def reset(self):`
- `sources/mem0/mem0/vector_stores/turbopuffer.py:337` `self.delete_col()`

**delete**, body `sources/mem0/mem0/vector_stores/upstash_vector.py:223`; implementation/call-through.

- `sources/mem0/mem0/vector_stores/upstash_vector.py:216` `def delete(self, vector_id: int):`
- `sources/mem0/mem0/vector_stores/upstash_vector.py:223` `self.client.delete(`

**delete_col**, body `sources/mem0/mem0/vector_stores/upstash_vector.py:340`; implementation/call-through.

- `sources/mem0/mem0/vector_stores/upstash_vector.py:336` `def delete_col(self):`
- `sources/mem0/mem0/vector_stores/upstash_vector.py:340` `self.client.reset(namespace=self.collection_name)`

**reset**, body `sources/mem0/mem0/vector_stores/upstash_vector.py:360`; implementation/call-through.

- `sources/mem0/mem0/vector_stores/upstash_vector.py:356` `def reset(self):`
- `sources/mem0/mem0/vector_stores/upstash_vector.py:360` `self.delete_col()`

**delete**, body `sources/mem0/mem0/vector_stores/valkey.py:477`; implementation/call-through.

- `sources/mem0/mem0/vector_stores/valkey.py:470` `def delete(self, vector_id):`
- `sources/mem0/mem0/vector_stores/valkey.py:479` `self.client.delete(key)`

**delete_col**, body `sources/mem0/mem0/vector_stores/valkey.py:718`; implementation/call-through.

- `sources/mem0/mem0/vector_stores/valkey.py:714` `def delete_col(self):`
- `sources/mem0/mem0/vector_stores/valkey.py:718` `return self._drop_index(self.collection_name, log_level="info")`

**reset**, body `sources/mem0/mem0/vector_stores/valkey.py:741`; implementation/call-through.

- `sources/mem0/mem0/vector_stores/valkey.py:737` `def reset(self):`
- `sources/mem0/mem0/vector_stores/valkey.py:746` `self.delete_col()`

**delete**, body `sources/mem0/mem0/vector_stores/vertex_ai_vector_search.py:295`; implementation/call-through.

- `sources/mem0/mem0/vector_stores/vertex_ai_vector_search.py:286` `def delete(self, vector_id: Optional[str] = None, ids: Optional[List[str]] = None) -> bool:`
- `sources/mem0/mem0/vector_stores/vertex_ai_vector_search.py:307` `self.index.remove_datapoints(datapoint_ids=datapoint_ids)`

**delete_col**, body `sources/mem0/mem0/vector_stores/vertex_ai_vector_search.py:450`; implementation/call-through.

- `sources/mem0/mem0/vector_stores/vertex_ai_vector_search.py:445` `def delete_col(self):`
- `sources/mem0/mem0/vector_stores/vertex_ai_vector_search.py:450` `logger.warning("Delete collection operation is not supported for Google Matching Engine")`

**reset**, body `sources/mem0/mem0/vector_stores/vertex_ai_vector_search.py:646`; implementation/call-through.

- `sources/mem0/mem0/vector_stores/vertex_ai_vector_search.py:642` `def reset(self):`
- `sources/mem0/mem0/vector_stores/vertex_ai_vector_search.py:646` `logger.warning("Reset operation is not supported for Google Matching Engine")`

**delete**, body `sources/mem0/mem0/vector_stores/weaviate.py:282`; implementation/call-through.

- `sources/mem0/mem0/vector_stores/weaviate.py:275` `def delete(self, vector_id):`
- `sources/mem0/mem0/vector_stores/weaviate.py:283` `collection.data.delete_by_id(vector_id)`

**delete_col**, body `sources/mem0/mem0/vector_stores/weaviate.py:343`; implementation/call-through.

- `sources/mem0/mem0/vector_stores/weaviate.py:341` `def delete_col(self):`
- `sources/mem0/mem0/vector_stores/weaviate.py:343` `self.client.collections.delete(self.collection_name)`

**reset**, body `sources/mem0/mem0/vector_stores/weaviate.py:382`; implementation/call-through.

- `sources/mem0/mem0/vector_stores/weaviate.py:380` `def reset(self):`
- `sources/mem0/mem0/vector_stores/weaviate.py:383` `self.delete_col()`

**operation and route call sites**, body `sources/mem0/mem0-ts/src/client/index.ts:9`; body evidence.

- `sources/mem0/mem0-ts/src/client/index.ts:8` `GetAllMemoryOptions,`
- `sources/mem0/mem0-ts/src/client/index.ts:9` `DeleteAllMemoryOptions,`
- `sources/mem0/mem0-ts/src/client/index.ts:10` `ProjectOptions,`
- `sources/mem0/mem0-ts/src/client/index.ts:11` `Memory,`
- `sources/mem0/mem0-ts/src/client/index.ts:12` `MemoryHistory,`
- `sources/mem0/mem0-ts/src/client/index.ts:13` `MemoryUpdateBody,`

**operation and route call sites**, body `sources/mem0/mem0-ts/src/client/mem0.ts:12`; body evidence.

- `sources/mem0/mem0-ts/src/client/mem0.ts:11` `GetAllMemoryOptions,`
- `sources/mem0/mem0-ts/src/client/mem0.ts:12` `DeleteAllMemoryOptions,`
- `sources/mem0/mem0-ts/src/client/mem0.ts:13` `DeleteMemoryOptions,`
- `sources/mem0/mem0-ts/src/client/mem0.ts:14` `MemoryUpdateBody,`
- `sources/mem0/mem0-ts/src/client/mem0.ts:15` `ProjectResponse,`
- `sources/mem0/mem0-ts/src/client/mem0.ts:16` `PromptUpdatePayload,`
- `sources/mem0/mem0-ts/src/client/mem0.ts:17` `Webhook,`
- `sources/mem0/mem0-ts/src/client/mem0.ts:225` `if (identityByCredentials.size >= this.identityCacheMax) {`
- `sources/mem0/mem0-ts/src/client/mem0.ts:226` `identityByCredentials.delete(`
- `sources/mem0/mem0-ts/src/client/mem0.ts:227` `identityByCredentials.keys().next().value!,`
- `sources/mem0/mem0-ts/src/client/mem0.ts:228` `);`
- `sources/mem0/mem0-ts/src/client/mem0.ts:229` `}`
- `sources/mem0/mem0-ts/src/client/mem0.ts:230` `identityByCredentials.set(credentials, shared);`
- `sources/mem0/mem0-ts/src/client/mem0.ts:232` `shared.then((identity) => {`
- `sources/mem0/mem0-ts/src/client/mem0.ts:233` `if (!identity.telemetryId) identityByCredentials.delete(credentials);`
- `sources/mem0/mem0-ts/src/client/mem0.ts:234` `});`
- `sources/mem0/mem0-ts/src/client/mem0.ts:235` `}`
- `sources/mem0/mem0-ts/src/client/mem0.ts:236` `return shared.then((identity) => {`
- `sources/mem0/mem0-ts/src/client/mem0.ts:237` `this.telemetryId = identity.telemetryId;`
- `sources/mem0/mem0-ts/src/client/mem0.ts:522` ``
- `sources/mem0/mem0-ts/src/client/mem0.ts:523` `async delete(`
- `sources/mem0/mem0-ts/src/client/mem0.ts:524` `memoryId: string,`
- `sources/mem0/mem0-ts/src/client/mem0.ts:525` `options: DeleteMemoryOptions = {},`
- `sources/mem0/mem0-ts/src/client/mem0.ts:526` `): Promise<{ message: string }> {`
- `sources/mem0/mem0-ts/src/client/mem0.ts:527` `this._captureEvent("delete", [Object.keys(options || {})]);`
- `sources/mem0/mem0-ts/src/client/mem0.ts:528` `const snakeOptions = camelToSnakeKeys(this._prepareParams(options));`
- `sources/mem0/mem0-ts/src/client/mem0.ts:529` `// @ts-ignore`
- `sources/mem0/mem0-ts/src/client/mem0.ts:530` `const query = new URLSearchParams(snakeOptions).toString();`
- `sources/mem0/mem0-ts/src/client/mem0.ts:531` `return this._fetchWithErrorHandling(`
- `sources/mem0/mem0-ts/src/client/mem0.ts:533` `{`
- `sources/mem0/mem0-ts/src/client/mem0.ts:534` `method: "DELETE",`
- `sources/mem0/mem0-ts/src/client/mem0.ts:535` `headers: this.headers,`
- `sources/mem0/mem0-ts/src/client/mem0.ts:536` `},`
- `sources/mem0/mem0-ts/src/client/mem0.ts:537` `);`
- `sources/mem0/mem0-ts/src/client/mem0.ts:538` `}`
- `sources/mem0/mem0-ts/src/client/mem0.ts:539` ``
- `sources/mem0/mem0-ts/src/client/mem0.ts:540` `async deleteAll(`
- `sources/mem0/mem0-ts/src/client/mem0.ts:541` `options: DeleteAllMemoryOptions = {},`
- `sources/mem0/mem0-ts/src/client/mem0.ts:542` `): Promise<{ message: string }> {`
- `sources/mem0/mem0-ts/src/client/mem0.ts:543` `const payloadKeys = Object.keys(options || {});`
- `sources/mem0/mem0-ts/src/client/mem0.ts:544` `this._captureEvent("delete_all", [payloadKeys]);`
- `sources/mem0/mem0-ts/src/client/mem0.ts:545` `const snakeOptions = camelToSnakeKeys(this._prepareParams(options));`
- `sources/mem0/mem0-ts/src/client/mem0.ts:546` `// @ts-ignore`
- `sources/mem0/mem0-ts/src/client/mem0.ts:547` `const params = new URLSearchParams(snakeOptions);`
- `sources/mem0/mem0-ts/src/client/mem0.ts:548` `const response = await this._fetchWithErrorHandling(`
- `sources/mem0/mem0-ts/src/client/mem0.ts:550` `{`
- `sources/mem0/mem0-ts/src/client/mem0.ts:551` `method: "DELETE",`
- `sources/mem0/mem0-ts/src/client/mem0.ts:552` `headers: this.headers,`
- `sources/mem0/mem0-ts/src/client/mem0.ts:553` `},`
- `sources/mem0/mem0-ts/src/client/mem0.ts:554` `);`
- `sources/mem0/mem0-ts/src/client/mem0.ts:555` `return response;`
- `sources/mem0/mem0-ts/src/client/mem0.ts:587` `*/`
- `sources/mem0/mem0-ts/src/client/mem0.ts:588` `async deleteUser(data: {`
- `sources/mem0/mem0-ts/src/client/mem0.ts:589` `entity_id: number;`
- `sources/mem0/mem0-ts/src/client/mem0.ts:590` `entity_type: string;`
- `sources/mem0/mem0-ts/src/client/mem0.ts:591` `}): Promise<{ message: string }> {`
- `sources/mem0/mem0-ts/src/client/mem0.ts:592` `this._captureEvent("delete_user", []);`
- `sources/mem0/mem0-ts/src/client/mem0.ts:593` `if (!data.entity_type) {`
- `sources/mem0/mem0-ts/src/client/mem0.ts:594` `data.entity_type = "user";`
- `sources/mem0/mem0-ts/src/client/mem0.ts:595` `}`
- `sources/mem0/mem0-ts/src/client/mem0.ts:596` `const response = await this._fetchWithErrorHandling(`
- `sources/mem0/mem0-ts/src/client/mem0.ts:598` `{`
- `sources/mem0/mem0-ts/src/client/mem0.ts:599` `method: "DELETE",`
- `sources/mem0/mem0-ts/src/client/mem0.ts:600` `headers: this.headers,`
- `sources/mem0/mem0-ts/src/client/mem0.ts:601` `},`
- `sources/mem0/mem0-ts/src/client/mem0.ts:602` `);`
- `sources/mem0/mem0-ts/src/client/mem0.ts:603` `return response;`
- `sources/mem0/mem0-ts/src/client/mem0.ts:605` ``
- `sources/mem0/mem0-ts/src/client/mem0.ts:606` `async deleteUsers(`
- `sources/mem0/mem0-ts/src/client/mem0.ts:607` `params: {`
- `sources/mem0/mem0-ts/src/client/mem0.ts:608` `userId?: string;`
- `sources/mem0/mem0-ts/src/client/mem0.ts:609` `agentId?: string;`
- `sources/mem0/mem0-ts/src/client/mem0.ts:610` `appId?: string;`
- `sources/mem0/mem0-ts/src/client/mem0.ts:613` `): Promise<{ message: string }> {`
- `sources/mem0/mem0-ts/src/client/mem0.ts:614` `let to_delete: Array<{ type: string; name: string }> = [];`
- `sources/mem0/mem0-ts/src/client/mem0.ts:615` `const { userId, agentId, appId, runId } = params;`
- `sources/mem0/mem0-ts/src/client/mem0.ts:616` ``
- `sources/mem0/mem0-ts/src/client/mem0.ts:617` `if (userId) {`
- `sources/mem0/mem0-ts/src/client/mem0.ts:618` `to_delete = [{ type: "user", name: userId }];`
- `sources/mem0/mem0-ts/src/client/mem0.ts:619` `} else if (agentId) {`
- `sources/mem0/mem0-ts/src/client/mem0.ts:620` `to_delete = [{ type: "agent", name: agentId }];`
- `sources/mem0/mem0-ts/src/client/mem0.ts:621` `} else if (appId) {`
- `sources/mem0/mem0-ts/src/client/mem0.ts:622` `to_delete = [{ type: "app", name: appId }];`
- `sources/mem0/mem0-ts/src/client/mem0.ts:623` `} else if (runId) {`
- `sources/mem0/mem0-ts/src/client/mem0.ts:624` `to_delete = [{ type: "run", name: runId }];`
- `sources/mem0/mem0-ts/src/client/mem0.ts:625` `} else {`
- `sources/mem0/mem0-ts/src/client/mem0.ts:626` `const entities = await this.users();`
- `sources/mem0/mem0-ts/src/client/mem0.ts:627` `to_delete = entities.results.map((entity) => ({`
- `sources/mem0/mem0-ts/src/client/mem0.ts:628` `type: entity.type,`
- `sources/mem0/mem0-ts/src/client/mem0.ts:629` `name: entity.name,`
- `sources/mem0/mem0-ts/src/client/mem0.ts:630` `}));`
- `sources/mem0/mem0-ts/src/client/mem0.ts:631` `}`
- `sources/mem0/mem0-ts/src/client/mem0.ts:632` ``
- `sources/mem0/mem0-ts/src/client/mem0.ts:633` `if (to_delete.length === 0) {`
- `sources/mem0/mem0-ts/src/client/mem0.ts:634` `throw new Error("No entities to delete");`
- `sources/mem0/mem0-ts/src/client/mem0.ts:635` `}`
- `sources/mem0/mem0-ts/src/client/mem0.ts:636` ``
- `sources/mem0/mem0-ts/src/client/mem0.ts:637` `for (const entity of to_delete) {`
- `sources/mem0/mem0-ts/src/client/mem0.ts:638` `try {`
- `sources/mem0/mem0-ts/src/client/mem0.ts:639` `// fetch() reuses the pooled connection; axios here defaulted to`
- `sources/mem0/mem0-ts/src/client/mem0.ts:640` `// keepAlive: false, one handshake per entity.`
- `sources/mem0/mem0-ts/src/client/mem0.ts:641` `await this._fetchWithErrorHandling(`
- `sources/mem0/mem0-ts/src/client/mem0.ts:643` `{`
- `sources/mem0/mem0-ts/src/client/mem0.ts:644` `method: "DELETE",`
- `sources/mem0/mem0-ts/src/client/mem0.ts:645` `headers: this.headers,`
- `sources/mem0/mem0-ts/src/client/mem0.ts:646` `},`
- `sources/mem0/mem0-ts/src/client/mem0.ts:647` `);`
- `sources/mem0/mem0-ts/src/client/mem0.ts:648` `} catch (error: any) {`
- `sources/mem0/mem0-ts/src/client/mem0.ts:649` `throw new APIError(`
- `sources/mem0/mem0-ts/src/client/mem0.ts:650` `'Failed to delete ${entity.type} ${entity.name}: ${error.message}',`
- `sources/mem0/mem0-ts/src/client/mem0.ts:651` `);`
- `sources/mem0/mem0-ts/src/client/mem0.ts:652` `}`
- `sources/mem0/mem0-ts/src/client/mem0.ts:653` `}`
- `sources/mem0/mem0-ts/src/client/mem0.ts:654` ``
- `sources/mem0/mem0-ts/src/client/mem0.ts:655` `this._captureEvent("delete_users", [`
- `sources/mem0/mem0-ts/src/client/mem0.ts:656` `{ userId, agentId, appId, runId, sync_type: "sync" },`
- `sources/mem0/mem0-ts/src/client/mem0.ts:657` `]);`
- `sources/mem0/mem0-ts/src/client/mem0.ts:658` ``
- `sources/mem0/mem0-ts/src/client/mem0.ts:659` `return {`
- `sources/mem0/mem0-ts/src/client/mem0.ts:661` `userId || agentId || appId || runId`
- `sources/mem0/mem0-ts/src/client/mem0.ts:662` `? "Entity deleted successfully."`
- `sources/mem0/mem0-ts/src/client/mem0.ts:663` `: "All users, agents, apps and runs deleted.",`
- `sources/mem0/mem0-ts/src/client/mem0.ts:664` `};`
- `sources/mem0/mem0-ts/src/client/mem0.ts:665` `}`
- `sources/mem0/mem0-ts/src/client/mem0.ts:666` ``
- `sources/mem0/mem0-ts/src/client/mem0.ts:667` `async batchUpdate(memories: Array<MemoryUpdateBody>): Promise<string> {`
- `sources/mem0/mem0-ts/src/client/mem0.ts:683` ``
- `sources/mem0/mem0-ts/src/client/mem0.ts:684` `async batchDelete(memories: Array<string>): Promise<string> {`
- `sources/mem0/mem0-ts/src/client/mem0.ts:685` `this._captureEvent("batch_delete", []);`
- `sources/mem0/mem0-ts/src/client/mem0.ts:686` `const memoriesBody = memories.map((memory) => ({`
- `sources/mem0/mem0-ts/src/client/mem0.ts:687` `memory_id: memory,`
- `sources/mem0/mem0-ts/src/client/mem0.ts:688` `}));`
- `sources/mem0/mem0-ts/src/client/mem0.ts:689` `const response = await this._fetchWithErrorHandling(`
- `sources/mem0/mem0-ts/src/client/mem0.ts:691` `{`
- `sources/mem0/mem0-ts/src/client/mem0.ts:692` `method: "DELETE",`
- `sources/mem0/mem0-ts/src/client/mem0.ts:693` `headers: this.headers,`
- `sources/mem0/mem0-ts/src/client/mem0.ts:694` `body: JSON.stringify({ memories: memoriesBody }),`
- `sources/mem0/mem0-ts/src/client/mem0.ts:695` `},`
- `sources/mem0/mem0-ts/src/client/mem0.ts:696` `);`
- `sources/mem0/mem0-ts/src/client/mem0.ts:803` ``
- `sources/mem0/mem0-ts/src/client/mem0.ts:804` `async deleteWebhook(data: {`
- `sources/mem0/mem0-ts/src/client/mem0.ts:805` `webhookId: string;`
- `sources/mem0/mem0-ts/src/client/mem0.ts:806` `}): Promise<{ message: string }> {`
- `sources/mem0/mem0-ts/src/client/mem0.ts:807` `this._captureEvent("delete_webhook", []);`
- `sources/mem0/mem0-ts/src/client/mem0.ts:808` `const webhook_id = data.webhookId || data;`
- `sources/mem0/mem0-ts/src/client/mem0.ts:809` `const response = await this._fetchWithErrorHandling(`
- `sources/mem0/mem0-ts/src/client/mem0.ts:810` `'${this.host}/api/v1/webhooks/${webhook_id}/',`
- `sources/mem0/mem0-ts/src/client/mem0.ts:811` `{`
- `sources/mem0/mem0-ts/src/client/mem0.ts:812` `method: "DELETE",`
- `sources/mem0/mem0-ts/src/client/mem0.ts:813` `headers: this.headers,`
- `sources/mem0/mem0-ts/src/client/mem0.ts:814` `},`
- `sources/mem0/mem0-ts/src/client/mem0.ts:815` `);`
- `sources/mem0/mem0-ts/src/client/mem0.ts:816` `return response;`

**operation and route call sites**, body `sources/mem0/mem0-ts/src/client/mem0.types.ts:49`; body evidence.

- `sources/mem0/mem0-ts/src/client/mem0.types.ts:48` ``
- `sources/mem0/mem0-ts/src/client/mem0.types.ts:49` `export interface DeleteAllMemoryOptions extends EntityOptions {}`
- `sources/mem0/mem0-ts/src/client/mem0.types.ts:50` ``
- `sources/mem0/mem0-ts/src/client/mem0.types.ts:51` `export interface DeleteMemoryOptions {`
- `sources/mem0/mem0-ts/src/client/mem0.types.ts:52` `/**`
- `sources/mem0/mem0-ts/src/client/mem0.types.ts:53` `* When 'true', also delete the older memories this one superseded (the v3`
- `sources/mem0/mem0-ts/src/client/mem0.types.ts:54` `* linked chain), transitively , the delete-side counterpart of 'latestOnly'.`
- `sources/mem0/mem0-ts/src/client/mem0.types.ts:55` `* Off by default. Serialized as 'delete_linked'.`
- `sources/mem0/mem0-ts/src/client/mem0.types.ts:56` `*/`
- `sources/mem0/mem0-ts/src/client/mem0.types.ts:57` `deleteLinked?: boolean;`
- `sources/mem0/mem0-ts/src/client/mem0.types.ts:58` `}`
- `sources/mem0/mem0-ts/src/client/mem0.types.ts:59` ``
- `sources/mem0/mem0-ts/src/client/mem0.types.ts:60` `// ─── Project Options ────────────────────────────────────────`
- `sources/mem0/mem0-ts/src/client/mem0.types.ts:61` `export interface ProjectOptions {`
- `sources/mem0/mem0-ts/src/client/mem0.types.ts:112` `UPDATE = "UPDATE",`
- `sources/mem0/mem0-ts/src/client/mem0.types.ts:113` `DELETE = "DELETE",`
- `sources/mem0/mem0-ts/src/client/mem0.types.ts:114` `NOOP = "NOOP",`
- `sources/mem0/mem0-ts/src/client/mem0.types.ts:115` `}`
- `sources/mem0/mem0-ts/src/client/mem0.types.ts:116` ``
- `sources/mem0/mem0-ts/src/client/mem0.types.ts:117` `export interface Memory {`
- `sources/mem0/mem0-ts/src/client/mem0.types.ts:196` `MEMORY_UPDATED = "memory_update",`
- `sources/mem0/mem0-ts/src/client/mem0.types.ts:197` `MEMORY_DELETED = "memory_delete",`
- `sources/mem0/mem0-ts/src/client/mem0.types.ts:198` `MEMORY_CATEGORIZED = "memory_categorize",`
- `sources/mem0/mem0-ts/src/client/mem0.types.ts:199` `}`
- `sources/mem0/mem0-ts/src/client/mem0.types.ts:200` ``
- `sources/mem0/mem0-ts/src/client/mem0.types.ts:201` `export interface Webhook {`

**operation and route call sites**, body `sources/mem0/mem0-ts/src/oss/examples/basic.ts:104`; body evidence.

- `sources/mem0/mem0-ts/src/oss/examples/basic.ts:103` `if (result1.results && result1.results.length > 0) {`
- `sources/mem0/mem0-ts/src/oss/examples/basic.ts:104` `console.log("\nDeleting a memory...");`
- `sources/mem0/mem0-ts/src/oss/examples/basic.ts:105` `await memory.delete(result1.results[0].id);`
- `sources/mem0/mem0-ts/src/oss/examples/basic.ts:106` `console.log("Memory deleted successfully");`
- `sources/mem0/mem0-ts/src/oss/examples/basic.ts:107` `}`
- `sources/mem0/mem0-ts/src/oss/examples/basic.ts:108` ``
- `sources/mem0/mem0-ts/src/oss/examples/basic.ts:109` `// Reset all memories`
- `sources/mem0/mem0-ts/src/oss/examples/basic.ts:110` `console.log("\nResetting all memories...");`

**operation and route call sites**, body `sources/mem0/mem0-ts/src/oss/examples/utils/test-utils.ts:86`; body evidence.

- `sources/mem0/mem0-ts/src/oss/examples/utils/test-utils.ts:85` `if (result1.results && result1.results.length > 0) {`
- `sources/mem0/mem0-ts/src/oss/examples/utils/test-utils.ts:86` `console.log("\nDeleting a memory...");`
- `sources/mem0/mem0-ts/src/oss/examples/utils/test-utils.ts:87` `await memory.delete(result1.results[0].id);`
- `sources/mem0/mem0-ts/src/oss/examples/utils/test-utils.ts:88` `console.log("Memory deleted successfully");`
- `sources/mem0/mem0-ts/src/oss/examples/utils/test-utils.ts:89` `}`
- `sources/mem0/mem0-ts/src/oss/examples/utils/test-utils.ts:90` ``
- `sources/mem0/mem0-ts/src/oss/examples/utils/test-utils.ts:91` `// Reset all memories`
- `sources/mem0/mem0-ts/src/oss/examples/utils/test-utils.ts:92` `console.log("\nResetting all memories...");`

**operation and route call sites**, body `sources/mem0/mem0-ts/src/oss/src/embeddings/together.ts:10`; body evidence.

- `sources/mem0/mem0-ts/src/oss/src/embeddings/together.ts:9` `const openAICompatibleConfig = { ...config };`
- `sources/mem0/mem0-ts/src/oss/src/embeddings/together.ts:10` `delete openAICompatibleConfig.embeddingDims;`
- `sources/mem0/mem0-ts/src/oss/src/embeddings/together.ts:11` ``
- `sources/mem0/mem0-ts/src/oss/src/embeddings/together.ts:12` `const apiKey = config.apiKey || process.env.TOGETHER_API_KEY;`
- `sources/mem0/mem0-ts/src/oss/src/embeddings/together.ts:13` `if (!apiKey) {`
- `sources/mem0/mem0-ts/src/oss/src/embeddings/together.ts:14` `throw new Error("Together API key is required");`

**operation and route call sites**, body `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:40`; body evidence.

- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:39` `SearchMemoryOptions,`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:40` `DeleteAllMemoryOptions,`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:41` `GetAllMemoryOptions,`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:42` `UpdateMemoryOptions,`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:43` `UpdateProjectOptions,`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:44` `} from "./memory.types";`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:59` `getDecayFeatureErrorMessage,`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:60` `getDecayUsageDeleteCountAfterSuccess,`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:61` `getTemporalFeatureErrorMessage,`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:62` `isDecayUsageDeleteEligible,`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:63` `PerformanceSlowQueryTrigger,`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:64` `ScaleThresholdTrigger,`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:65` `} from "../utils/notices";`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:66` `import { lemmatizeForBm25 } from "../utils/lemmatization";`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:130` `// page limits (~100) to minimize roundtrips while bounded to avoid memory pressure.`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:131` `const DELETE_ALL_BATCH_SIZE = 1000;`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:132` ``
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:133` `/**`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:134` `* Validates that no top-level entity parameters are passed in config.`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:135` `* @throws Error if entity params are found at top level`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:440` `try {`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:441` `await entityStore.delete(row.id);`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:442` `} catch (e) {`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:443` `console.debug('Entity delete failed for id=${row.id}: ${e}');`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:444` `}`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:445` `} else {`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:446` `const newPayload = { ...payload, linkedMemoryIds: remaining };`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:447` `// entityStore.update requires a vector , re-embed entity text.`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:641` `private async _displayDecayUsageNotice(trigger: {`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:642` `triggerFunction: "delete" | "delete_all";`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:643` `triggerSource: "delete_count" | "delete_all";`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:644` `triggerReason: "repeated_deletes" | "bulk_delete";`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:645` `deleteCount?: number;`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:646` `deletedCount?: number;`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:647` `}) {`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:648` `try {`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:649` `await this._getTelemetryId();`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:650` `await displayDecayUsageNotice(this, trigger);`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:1095` `updatedAt: undefined as string | undefined,`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:1096` `isDeleted: 0,`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:1097` `}));`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:1098` ``
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:1099` `if (typeof this.db.batchAddHistory === "function") {`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:1100` `try {`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:1394` `for (const logicalKey of ["AND", "OR", "NOT"]) {`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:1395` `delete effectiveFilters[logicalKey];`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:1396` `}`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:1397` `for (const fk of Object.keys(effectiveFilters)) {`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:1398` `if (`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:1399` `!["AND", "OR", "NOT", "user_id", "agent_id", "run_id"].includes(fk) &&`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:1402` `) {`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:1403` `delete effectiveFilters[fk];`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:1404` `}`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:1405` `}`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:1406` `effectiveFilters = { ...effectiveFilters, ...processedFilters };`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:1407` `}`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:1698` ``
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:1699` `async delete(memoryId: string): Promise<{ message: string }> {`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:1700` `await this._ensureInitialized();`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:1701` `await this._captureEvent("delete", { memory_id: memoryId });`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:1702` `await this.deleteMemory(memoryId);`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:1703` `const result = { message: "Memory deleted successfully!" };`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:1704` `const deleteCount = getDecayUsageDeleteCountAfterSuccess();`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:1705` `if (isDecayUsageDeleteEligible(deleteCount)) {`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:1706` `await this._displayDecayUsageNotice({`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:1707` `triggerFunction: "delete",`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:1708` `triggerSource: "delete_count",`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:1709` `triggerReason: "repeated_deletes",`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:1710` `deleteCount,`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:1711` `});`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:1712` `} else {`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:1713` `await this._displayFirstRunNotice("delete");`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:1714` `}`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:1715` `return result;`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:1716` `}`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:1717` ``
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:1718` `async deleteAll(`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:1719` `config: DeleteAllMemoryOptions,`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:1720` `): Promise<{ message: string }> {`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:1721` `await this._ensureInitialized();`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:1722` `await this._captureEvent("delete_all", {`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:1723` `has_user_id: !!config.userId,`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:1724` `has_agent_id: !!config.agentId,`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:1725` `has_run_id: !!config.runId,`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:1726` `});`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:1738` `throw new Error(`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:1739` `"At least one filter is required to delete all memories. If you want to delete all memories, use the 'reset()' method.",`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:1740` `);`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:1741` `}`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:1742` ``
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:1743` `let deletedCount = 0;`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:1744` `const seenBatches = new Set<string>();`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:1745` ``
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:1746` `while (true) {`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:1747` `const [batch] = await this.vectorStore.list(`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:1748` `filters,`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:1749` `DELETE_ALL_BATCH_SIZE,`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:1750` `);`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:1751` `if (!batch.length) {`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:1752` `break;`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:1753` `}`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:1758` `if (seenBatches.has(batchKey)) {`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:1759` `logger.warn("Stopping deleteAll after a repeated memory batch");`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:1760` `break;`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:1761` `}`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:1762` `seenBatches.add(batchKey);`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:1763` `for (const memory of batch) {`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:1764` `await this.deleteMemory(memory.id);`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:1765` `}`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:1766` `deletedCount += batch.length;`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:1767` `}`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:1768` ``
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:1769` `const result = { message: "Memories deleted successfully!" };`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:1770` `if (deletedCount > 0) {`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:1771` `await this._displayDecayUsageNotice({`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:1772` `triggerFunction: "delete_all",`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:1773` `triggerSource: "delete_all",`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:1774` `triggerReason: "bulk_delete",`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:1775` `deletedCount,`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:1776` `});`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:1777` `} else {`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:1778` `await this._displayFirstRunNotice("delete_all");`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:1779` `}`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:1780` `return result;`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:1781` `}`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:1782` ``
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:1797` `try {`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:1798` `await this.vectorStore.deleteCol();`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:1799` `} catch (e) {`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:1800` `console.error(`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:1801` `'Failed to delete collection for provider '${this.config.vectorStore.provider}':',`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:1802` `e,`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:1803` `);`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:1804` `// Decide if you want to re-throw or just log`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:1805` `}`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:1807` `console.warn(`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:1808` `"Memory.reset(): Skipping vector store collection deletion as 'langchain' provider is used. Underlying Langchain vector store data is not cleared by this operation.",`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:1809` `);`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:1810` `}`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:1811` ``
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:1812` `if (this._entityStore) {`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:1813` `try {`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:1814` `await this._entityStore.deleteCol();`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:1815` `} catch {}`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:1816` `this._entityStore = undefined;`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:1817` `}`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:1818` ``
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:2015` ``
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:2016` `private async deleteMemory(memoryId: string): Promise<string> {`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:2017` `const existingMemory = await this.vectorStore.get(memoryId);`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:2018` `if (!existingMemory) {`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:2019` `throw new Error('Memory with ID ${memoryId} not found');`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:2020` `}`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:2025` `);`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:2026` `await this.vectorStore.delete(memoryId);`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:2027` `await this.db.addHistory(`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:2028` `memoryId,`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:2029` `prevValue,`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:2030` `null,`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:2031` `"DELETE",`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:2032` `undefined,`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:2033` `undefined,`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:2034` `1,`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:2035` `);`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:2041` `} catch (e) {`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:2042` `console.warn('Entity store cleanup failed during delete: ${e}');`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:2043` `}`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:2044` ``
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:2045` `return memoryId;`
- `sources/mem0/mem0-ts/src/oss/src/memory/index.ts:2046` `}`

**operation and route call sites**, body `sources/mem0/mem0-ts/src/oss/src/memory/memory.types.ts:55`; body evidence.

- `sources/mem0/mem0-ts/src/oss/src/memory/memory.types.ts:54` ``
- `sources/mem0/mem0-ts/src/oss/src/memory/memory.types.ts:55` `export interface DeleteAllMemoryOptions extends Entity {}`
- `sources/mem0/mem0-ts/src/oss/src/memory/memory.types.ts:56` ``
- `sources/mem0/mem0-ts/src/oss/src/memory/memory.types.ts:57` `export interface UpdateProjectOptions {`
- `sources/mem0/mem0-ts/src/oss/src/memory/memory.types.ts:58` `decay?: boolean;`
- `sources/mem0/mem0-ts/src/oss/src/memory/memory.types.ts:59` `[key: string]: any;`

**operation and route call sites**, body `sources/mem0/mem0-ts/src/oss/src/prompts/index.ts:27`; body evidence.

- `sources/mem0/mem0-ts/src/oss/src/prompts/index.ts:26` `event: z`
- `sources/mem0/mem0-ts/src/oss/src/prompts/index.ts:27` `.enum(["ADD", "UPDATE", "DELETE", "NONE"])`
- `sources/mem0/mem0-ts/src/oss/src/prompts/index.ts:28` `.describe(`
- `sources/mem0/mem0-ts/src/oss/src/prompts/index.ts:29` `"The action taken for this memory item (ADD, UPDATE, DELETE, or NONE).",`
- `sources/mem0/mem0-ts/src/oss/src/prompts/index.ts:30` `),`
- `sources/mem0/mem0-ts/src/oss/src/prompts/index.ts:31` `old_memory: z`
- `sources/mem0/mem0-ts/src/oss/src/prompts/index.ts:32` `.string()`
- `sources/mem0/mem0-ts/src/oss/src/prompts/index.ts:33` `.optional()`
- `sources/mem0/mem0-ts/src/oss/src/prompts/index.ts:109` `return 'You are a smart memory manager which controls the memory of a system.`
- `sources/mem0/mem0-ts/src/oss/src/prompts/index.ts:110` `You can perform four operations: (1) add into the memory, (2) update the memory, (3) delete from the memory, and (4) no change.`
- `sources/mem0/mem0-ts/src/oss/src/prompts/index.ts:111` ``
- `sources/mem0/mem0-ts/src/oss/src/prompts/index.ts:112` `Based on the above four operations, the memory will change.`
- `sources/mem0/mem0-ts/src/oss/src/prompts/index.ts:113` ``
- `sources/mem0/mem0-ts/src/oss/src/prompts/index.ts:114` `Compare newly retrieved facts with the existing memory. For each new fact, decide whether to:`
- `sources/mem0/mem0-ts/src/oss/src/prompts/index.ts:116` `- UPDATE: Update an existing memory element`
- `sources/mem0/mem0-ts/src/oss/src/prompts/index.ts:117` `- DELETE: Delete an existing memory element`
- `sources/mem0/mem0-ts/src/oss/src/prompts/index.ts:118` `- NONE: Make no change (if the fact is already present or irrelevant)`
- `sources/mem0/mem0-ts/src/oss/src/prompts/index.ts:119` ``
- `sources/mem0/mem0-ts/src/oss/src/prompts/index.ts:120` `There are specific guidelines to select which operation to perform:`
- `sources/mem0/mem0-ts/src/oss/src/prompts/index.ts:121` ``
- `sources/mem0/mem0-ts/src/oss/src/prompts/index.ts:194` ``
- `sources/mem0/mem0-ts/src/oss/src/prompts/index.ts:195` `3. **Delete**: If the retrieved facts contain information that contradicts the information present in the memory, then you have to delete it. Or if the direction is to delete the memory, then you have to delete it.`
- `sources/mem0/mem0-ts/src/oss/src/prompts/index.ts:196` `Please note to return the IDs in the output from the input IDs only and do not generate any new ID.`
- `sources/mem0/mem0-ts/src/oss/src/prompts/index.ts:197` `- **Example**:`
- `sources/mem0/mem0-ts/src/oss/src/prompts/index.ts:198` `- Old Memory:`
- `sources/mem0/mem0-ts/src/oss/src/prompts/index.ts:199` `[`
- `sources/mem0/mem0-ts/src/oss/src/prompts/index.ts:220` `"text" : "Loves cheese pizza",`
- `sources/mem0/mem0-ts/src/oss/src/prompts/index.ts:221` `"event" : "DELETE"`
- `sources/mem0/mem0-ts/src/oss/src/prompts/index.ts:222` `}`
- `sources/mem0/mem0-ts/src/oss/src/prompts/index.ts:223` `]`
- `sources/mem0/mem0-ts/src/oss/src/prompts/index.ts:224` `}`
- `sources/mem0/mem0-ts/src/oss/src/prompts/index.ts:225` ``
- `sources/mem0/mem0-ts/src/oss/src/prompts/index.ts:259` ``
- `sources/mem0/mem0-ts/src/oss/src/prompts/index.ts:260` `The new retrieved facts are mentioned below. You have to analyze the new retrieved facts and determine whether these facts should be added, updated, or deleted in the memory.`
- `sources/mem0/mem0-ts/src/oss/src/prompts/index.ts:261` ``
- `sources/mem0/mem0-ts/src/oss/src/prompts/index.ts:262` `${JSON.stringify(newRetrievedFacts, null, 2)}`
- `sources/mem0/mem0-ts/src/oss/src/prompts/index.ts:263` ``
- `sources/mem0/mem0-ts/src/oss/src/prompts/index.ts:264` `Follow the instruction mentioned below:`
- `sources/mem0/mem0-ts/src/oss/src/prompts/index.ts:268` `- If there is an addition, generate a new key and add the new memory corresponding to it.`
- `sources/mem0/mem0-ts/src/oss/src/prompts/index.ts:269` `- If there is a deletion, the memory key-value pair should be removed from the memory.`
- `sources/mem0/mem0-ts/src/oss/src/prompts/index.ts:270` `- If there is an update, the ID key should remain the same and only the value needs to be updated.`
- `sources/mem0/mem0-ts/src/oss/src/prompts/index.ts:271` `- DO NOT RETURN ANYTHING ELSE OTHER THAN THE JSON FORMAT.`
- `sources/mem0/mem0-ts/src/oss/src/prompts/index.ts:272` `- DO NOT ADD ANY ADDITIONAL TEXT OR CODEBLOCK IN THE JSON FIELDS WHICH MAKE IT INVALID SUCH AS "\'\'\'json" OR "\'\'\'".`
- `sources/mem0/mem0-ts/src/oss/src/prompts/index.ts:273` ``

**operation and route call sites**, body `sources/mem0/mem0-ts/src/oss/src/storage/DummyHistoryManager.ts:11`; body evidence.

- `sources/mem0/mem0-ts/src/oss/src/storage/DummyHistoryManager.ts:10` `updatedAt?: string,`
- `sources/mem0/mem0-ts/src/oss/src/storage/DummyHistoryManager.ts:11` `isDeleted: number = 0,`
- `sources/mem0/mem0-ts/src/oss/src/storage/DummyHistoryManager.ts:12` `): Promise<void> {`
- `sources/mem0/mem0-ts/src/oss/src/storage/DummyHistoryManager.ts:13` `return;`
- `sources/mem0/mem0-ts/src/oss/src/storage/DummyHistoryManager.ts:14` `}`
- `sources/mem0/mem0-ts/src/oss/src/storage/DummyHistoryManager.ts:15` ``

**operation and route call sites**, body `sources/mem0/mem0-ts/src/oss/src/storage/MemoryHistoryManager.ts:11`; body evidence.

- `sources/mem0/mem0-ts/src/oss/src/storage/MemoryHistoryManager.ts:10` `updated_at: string | null;`
- `sources/mem0/mem0-ts/src/oss/src/storage/MemoryHistoryManager.ts:11` `is_deleted: number;`
- `sources/mem0/mem0-ts/src/oss/src/storage/MemoryHistoryManager.ts:12` `}`
- `sources/mem0/mem0-ts/src/oss/src/storage/MemoryHistoryManager.ts:13` ``
- `sources/mem0/mem0-ts/src/oss/src/storage/MemoryHistoryManager.ts:14` `export class MemoryHistoryManager implements HistoryManager {`
- `sources/mem0/mem0-ts/src/oss/src/storage/MemoryHistoryManager.ts:15` `private memoryStore: Map<string, HistoryEntry> = new Map();`
- `sources/mem0/mem0-ts/src/oss/src/storage/MemoryHistoryManager.ts:23` `updatedAt?: string,`
- `sources/mem0/mem0-ts/src/oss/src/storage/MemoryHistoryManager.ts:24` `isDeleted: number = 0,`
- `sources/mem0/mem0-ts/src/oss/src/storage/MemoryHistoryManager.ts:25` `): Promise<void> {`
- `sources/mem0/mem0-ts/src/oss/src/storage/MemoryHistoryManager.ts:26` `const historyEntry: HistoryEntry = {`
- `sources/mem0/mem0-ts/src/oss/src/storage/MemoryHistoryManager.ts:27` `id: uuidv4(),`
- `sources/mem0/mem0-ts/src/oss/src/storage/MemoryHistoryManager.ts:28` `memory_id: memoryId,`
- `sources/mem0/mem0-ts/src/oss/src/storage/MemoryHistoryManager.ts:33` `updated_at: updatedAt || null,`
- `sources/mem0/mem0-ts/src/oss/src/storage/MemoryHistoryManager.ts:34` `is_deleted: isDeleted,`
- `sources/mem0/mem0-ts/src/oss/src/storage/MemoryHistoryManager.ts:35` `};`
- `sources/mem0/mem0-ts/src/oss/src/storage/MemoryHistoryManager.ts:36` ``
- `sources/mem0/mem0-ts/src/oss/src/storage/MemoryHistoryManager.ts:37` `this.memoryStore.set(historyEntry.id, historyEntry);`
- `sources/mem0/mem0-ts/src/oss/src/storage/MemoryHistoryManager.ts:38` `}`

**operation and route call sites**, body `sources/mem0/mem0-ts/src/oss/src/storage/SQLiteManager.ts:27`; body evidence.

- `sources/mem0/mem0-ts/src/oss/src/storage/SQLiteManager.ts:26` `updated_at TEXT,`
- `sources/mem0/mem0-ts/src/oss/src/storage/SQLiteManager.ts:27` `is_deleted INTEGER DEFAULT 0`
- `sources/mem0/mem0-ts/src/oss/src/storage/SQLiteManager.ts:28` `)`
- `sources/mem0/mem0-ts/src/oss/src/storage/SQLiteManager.ts:29` `');`
- `sources/mem0/mem0-ts/src/oss/src/storage/SQLiteManager.ts:30` `this.db.exec('`
- `sources/mem0/mem0-ts/src/oss/src/storage/SQLiteManager.ts:31` `CREATE TABLE IF NOT EXISTS messages (`
- `sources/mem0/mem0-ts/src/oss/src/storage/SQLiteManager.ts:41` `'INSERT INTO memory_history`
- `sources/mem0/mem0-ts/src/oss/src/storage/SQLiteManager.ts:42` `(memory_id, previous_value, new_value, action, created_at, updated_at, is_deleted)`
- `sources/mem0/mem0-ts/src/oss/src/storage/SQLiteManager.ts:43` `VALUES (?, ?, ?, ?, ?, ?, ?)',`
- `sources/mem0/mem0-ts/src/oss/src/storage/SQLiteManager.ts:44` `);`
- `sources/mem0/mem0-ts/src/oss/src/storage/SQLiteManager.ts:45` `this.stmtSelect = this.db.prepare(`
- `sources/mem0/mem0-ts/src/oss/src/storage/SQLiteManager.ts:46` `"SELECT * FROM memory_history WHERE memory_id = ? ORDER BY id DESC",`
- `sources/mem0/mem0-ts/src/oss/src/storage/SQLiteManager.ts:56` `updatedAt?: string,`
- `sources/mem0/mem0-ts/src/oss/src/storage/SQLiteManager.ts:57` `isDeleted: number = 0,`
- `sources/mem0/mem0-ts/src/oss/src/storage/SQLiteManager.ts:58` `): Promise<void> {`
- `sources/mem0/mem0-ts/src/oss/src/storage/SQLiteManager.ts:59` `this.stmtInsert.run(`
- `sources/mem0/mem0-ts/src/oss/src/storage/SQLiteManager.ts:60` `memoryId,`
- `sources/mem0/mem0-ts/src/oss/src/storage/SQLiteManager.ts:61` `previousValue,`
- `sources/mem0/mem0-ts/src/oss/src/storage/SQLiteManager.ts:65` `updatedAt ?? null,`
- `sources/mem0/mem0-ts/src/oss/src/storage/SQLiteManager.ts:66` `isDeleted,`
- `sources/mem0/mem0-ts/src/oss/src/storage/SQLiteManager.ts:67` `);`
- `sources/mem0/mem0-ts/src/oss/src/storage/SQLiteManager.ts:68` `}`
- `sources/mem0/mem0-ts/src/oss/src/storage/SQLiteManager.ts:69` ``
- `sources/mem0/mem0-ts/src/oss/src/storage/SQLiteManager.ts:70` `async getHistory(memoryId: string): Promise<any[]> {`
- `sources/mem0/mem0-ts/src/oss/src/storage/SQLiteManager.ts:84` `const evict = this.db.prepare(`
- `sources/mem0/mem0-ts/src/oss/src/storage/SQLiteManager.ts:85` `'DELETE FROM messages WHERE session_scope = ? AND id NOT IN (`
- `sources/mem0/mem0-ts/src/oss/src/storage/SQLiteManager.ts:86` `SELECT id FROM (`
- `sources/mem0/mem0-ts/src/oss/src/storage/SQLiteManager.ts:87` `SELECT id FROM messages WHERE session_scope = ? ORDER BY created_at DESC LIMIT 10`
- `sources/mem0/mem0-ts/src/oss/src/storage/SQLiteManager.ts:88` `)`
- `sources/mem0/mem0-ts/src/oss/src/storage/SQLiteManager.ts:89` `)',`
- `sources/mem0/mem0-ts/src/oss/src/storage/SQLiteManager.ts:148` `updatedAt?: string;`
- `sources/mem0/mem0-ts/src/oss/src/storage/SQLiteManager.ts:149` `isDeleted?: number;`
- `sources/mem0/mem0-ts/src/oss/src/storage/SQLiteManager.ts:150` `}>,`
- `sources/mem0/mem0-ts/src/oss/src/storage/SQLiteManager.ts:151` `): Promise<void> {`
- `sources/mem0/mem0-ts/src/oss/src/storage/SQLiteManager.ts:152` `const txn = this.db.transaction(() => {`
- `sources/mem0/mem0-ts/src/oss/src/storage/SQLiteManager.ts:153` `for (const record of records) {`
- `sources/mem0/mem0-ts/src/oss/src/storage/SQLiteManager.ts:160` `record.updatedAt ?? null,`
- `sources/mem0/mem0-ts/src/oss/src/storage/SQLiteManager.ts:161` `record.isDeleted ?? 0,`
- `sources/mem0/mem0-ts/src/oss/src/storage/SQLiteManager.ts:162` `);`
- `sources/mem0/mem0-ts/src/oss/src/storage/SQLiteManager.ts:163` `}`
- `sources/mem0/mem0-ts/src/oss/src/storage/SQLiteManager.ts:164` `});`
- `sources/mem0/mem0-ts/src/oss/src/storage/SQLiteManager.ts:165` `txn();`

**operation and route call sites**, body `sources/mem0/mem0-ts/src/oss/src/storage/SupabaseHistoryManager.ts:14`; body evidence.

- `sources/mem0/mem0-ts/src/oss/src/storage/SupabaseHistoryManager.ts:13` `updated_at: string | null;`
- `sources/mem0/mem0-ts/src/oss/src/storage/SupabaseHistoryManager.ts:14` `is_deleted: number;`
- `sources/mem0/mem0-ts/src/oss/src/storage/SupabaseHistoryManager.ts:15` `}`
- `sources/mem0/mem0-ts/src/oss/src/storage/SupabaseHistoryManager.ts:16` ``
- `sources/mem0/mem0-ts/src/oss/src/storage/SupabaseHistoryManager.ts:17` `interface SupabaseHistoryConfig {`
- `sources/mem0/mem0-ts/src/oss/src/storage/SupabaseHistoryManager.ts:18` `supabaseUrl: string;`
- `sources/mem0/mem0-ts/src/oss/src/storage/SupabaseHistoryManager.ts:68` `updated_at timestamp with time zone,`
- `sources/mem0/mem0-ts/src/oss/src/storage/SupabaseHistoryManager.ts:69` `is_deleted integer default 0`
- `sources/mem0/mem0-ts/src/oss/src/storage/SupabaseHistoryManager.ts:70` `);`
- `sources/mem0/mem0-ts/src/oss/src/storage/SupabaseHistoryManager.ts:71` `');`
- `sources/mem0/mem0-ts/src/oss/src/storage/SupabaseHistoryManager.ts:72` `throw error;`
- `sources/mem0/mem0-ts/src/oss/src/storage/SupabaseHistoryManager.ts:73` `}`
- `sources/mem0/mem0-ts/src/oss/src/storage/SupabaseHistoryManager.ts:82` `updatedAt?: string,`
- `sources/mem0/mem0-ts/src/oss/src/storage/SupabaseHistoryManager.ts:83` `isDeleted: number = 0,`
- `sources/mem0/mem0-ts/src/oss/src/storage/SupabaseHistoryManager.ts:84` `): Promise<void> {`
- `sources/mem0/mem0-ts/src/oss/src/storage/SupabaseHistoryManager.ts:85` `await this.ensureClient();`
- `sources/mem0/mem0-ts/src/oss/src/storage/SupabaseHistoryManager.ts:86` `const historyEntry: HistoryEntry = {`
- `sources/mem0/mem0-ts/src/oss/src/storage/SupabaseHistoryManager.ts:87` `id: uuidv4(),`
- `sources/mem0/mem0-ts/src/oss/src/storage/SupabaseHistoryManager.ts:93` `updated_at: updatedAt || null,`
- `sources/mem0/mem0-ts/src/oss/src/storage/SupabaseHistoryManager.ts:94` `is_deleted: isDeleted,`
- `sources/mem0/mem0-ts/src/oss/src/storage/SupabaseHistoryManager.ts:95` `};`
- `sources/mem0/mem0-ts/src/oss/src/storage/SupabaseHistoryManager.ts:96` ``
- `sources/mem0/mem0-ts/src/oss/src/storage/SupabaseHistoryManager.ts:97` `const { error } = await this.supabase`
- `sources/mem0/mem0-ts/src/oss/src/storage/SupabaseHistoryManager.ts:98` `.from(this.tableName)`
- `sources/mem0/mem0-ts/src/oss/src/storage/SupabaseHistoryManager.ts:127` `.from(this.tableName)`
- `sources/mem0/mem0-ts/src/oss/src/storage/SupabaseHistoryManager.ts:128` `.delete()`
- `sources/mem0/mem0-ts/src/oss/src/storage/SupabaseHistoryManager.ts:129` `.neq("id", "");`
- `sources/mem0/mem0-ts/src/oss/src/storage/SupabaseHistoryManager.ts:130` ``
- `sources/mem0/mem0-ts/src/oss/src/storage/SupabaseHistoryManager.ts:131` `if (error) {`
- `sources/mem0/mem0-ts/src/oss/src/storage/SupabaseHistoryManager.ts:132` `console.error("Error resetting Supabase history:", error);`

**operation and route call sites**, body `sources/mem0/mem0-ts/src/oss/src/storage/base.ts:9`; body evidence.

- `sources/mem0/mem0-ts/src/oss/src/storage/base.ts:8` `updatedAt?: string,`
- `sources/mem0/mem0-ts/src/oss/src/storage/base.ts:9` `isDeleted?: number,`
- `sources/mem0/mem0-ts/src/oss/src/storage/base.ts:10` `): Promise<void>;`
- `sources/mem0/mem0-ts/src/oss/src/storage/base.ts:11` `getHistory(memoryId: string): Promise<any[]>;`
- `sources/mem0/mem0-ts/src/oss/src/storage/base.ts:12` `reset(): Promise<void>;`
- `sources/mem0/mem0-ts/src/oss/src/storage/base.ts:13` `close(): void;`
- `sources/mem0/mem0-ts/src/oss/src/storage/base.ts:33` `updatedAt?: string;`
- `sources/mem0/mem0-ts/src/oss/src/storage/base.ts:34` `isDeleted?: number;`
- `sources/mem0/mem0-ts/src/oss/src/storage/base.ts:35` `}>,`
- `sources/mem0/mem0-ts/src/oss/src/storage/base.ts:36` `): Promise<void>;`
- `sources/mem0/mem0-ts/src/oss/src/storage/base.ts:37` `}`

**operation and route call sites**, body `sources/mem0/mem0-ts/src/oss/src/utils/notices.ts:83`; body evidence.

- `sources/mem0/mem0-ts/src/oss/src/utils/notices.ts:82` `"The decay parameter is not supported by the OSS Memory SDK.";`
- `sources/mem0/mem0-ts/src/oss/src/utils/notices.ts:83` `const DECAY_USAGE_DELETE_THRESHOLD = 5;`
- `sources/mem0/mem0-ts/src/oss/src/utils/notices.ts:84` `export const SCALE_MEMORY_COUNT_THRESHOLD = 2000;`
- `sources/mem0/mem0-ts/src/oss/src/utils/notices.ts:85` `export const SCALE_MEMORY_COUNT_CHECK_INTERVAL = 100;`
- `sources/mem0/mem0-ts/src/oss/src/utils/notices.ts:86` `export const SCALE_TOP_K_THRESHOLD = 50;`
- `sources/mem0/mem0-ts/src/oss/src/utils/notices.ts:87` `export const PERFORMANCE_SLOW_QUERY_THRESHOLD_MS = 2000;`
- `sources/mem0/mem0-ts/src/oss/src/utils/notices.ts:96` `let firstRunClaimInProgress = false;`
- `sources/mem0/mem0-ts/src/oss/src/utils/notices.ts:97` `let decayUsageSuccessfulDeleteCount = 0;`
- `sources/mem0/mem0-ts/src/oss/src/utils/notices.ts:98` `const noticeCapacityReachedInProcess = new Set<string>();`
- `sources/mem0/mem0-ts/src/oss/src/utils/notices.ts:99` `let scaleMemoryCountAddsSinceCheck = 0;`
- `sources/mem0/mem0-ts/src/oss/src/utils/notices.ts:100` `let scaleMemoryCountCheckedInProcess = false;`
- `sources/mem0/mem0-ts/src/oss/src/utils/notices.ts:101` `let scaleMemoryCountThresholdEvaluatedInProcess = false;`
- `sources/mem0/mem0-ts/src/oss/src/utils/notices.ts:129` `export interface DecayUsageTrigger {`
- `sources/mem0/mem0-ts/src/oss/src/utils/notices.ts:130` `triggerFunction: "delete" | "delete_all";`
- `sources/mem0/mem0-ts/src/oss/src/utils/notices.ts:131` `triggerSource: "delete_count" | "delete_all";`
- `sources/mem0/mem0-ts/src/oss/src/utils/notices.ts:132` `triggerReason: "repeated_deletes" | "bulk_delete";`
- `sources/mem0/mem0-ts/src/oss/src/utils/notices.ts:133` `deleteCount?: number;`
- `sources/mem0/mem0-ts/src/oss/src/utils/notices.ts:134` `deletedCount?: number;`
- `sources/mem0/mem0-ts/src/oss/src/utils/notices.ts:135` `}`
- `sources/mem0/mem0-ts/src/oss/src/utils/notices.ts:136` ``
- `sources/mem0/mem0-ts/src/oss/src/utils/notices.ts:137` `export interface TemporalFeatureErrorTrigger {`
- `sources/mem0/mem0-ts/src/oss/src/utils/notices.ts:138` `triggerFunction: "add" | "search";`
- `sources/mem0/mem0-ts/src/oss/src/utils/notices.ts:728` ``
- `sources/mem0/mem0-ts/src/oss/src/utils/notices.ts:729` `export function getDecayUsageDeleteCountAfterSuccess(): number {`
- `sources/mem0/mem0-ts/src/oss/src/utils/notices.ts:730` `if (!isTelemetryEnabled()) return 0;`
- `sources/mem0/mem0-ts/src/oss/src/utils/notices.ts:731` `decayUsageSuccessfulDeleteCount += 1;`
- `sources/mem0/mem0-ts/src/oss/src/utils/notices.ts:732` `return decayUsageSuccessfulDeleteCount;`
- `sources/mem0/mem0-ts/src/oss/src/utils/notices.ts:733` `}`
- `sources/mem0/mem0-ts/src/oss/src/utils/notices.ts:734` ``
- `sources/mem0/mem0-ts/src/oss/src/utils/notices.ts:735` `export function isDecayUsageDeleteEligible(deleteCount: number): boolean {`
- `sources/mem0/mem0-ts/src/oss/src/utils/notices.ts:736` `return deleteCount >= DECAY_USAGE_DELETE_THRESHOLD;`
- `sources/mem0/mem0-ts/src/oss/src/utils/notices.ts:737` `}`
- `sources/mem0/mem0-ts/src/oss/src/utils/notices.ts:738` ``
- `sources/mem0/mem0-ts/src/oss/src/utils/notices.ts:739` `function isRecord(value: unknown): value is Record<string, any> {`
- `sources/mem0/mem0-ts/src/oss/src/utils/notices.ts:740` `return (`
- `sources/mem0/mem0-ts/src/oss/src/utils/notices.ts:1335` `trigger_reason: trigger.triggerReason,`
- `sources/mem0/mem0-ts/src/oss/src/utils/notices.ts:1336` `...(trigger.deleteCount !== undefined && {`
- `sources/mem0/mem0-ts/src/oss/src/utils/notices.ts:1337` `delete_count: trigger.deleteCount,`
- `sources/mem0/mem0-ts/src/oss/src/utils/notices.ts:1338` `}),`
- `sources/mem0/mem0-ts/src/oss/src/utils/notices.ts:1339` `...(trigger.deletedCount !== undefined && {`
- `sources/mem0/mem0-ts/src/oss/src/utils/notices.ts:1340` `deleted_count: trigger.deletedCount,`
- `sources/mem0/mem0-ts/src/oss/src/utils/notices.ts:1341` `}),`
- `sources/mem0/mem0-ts/src/oss/src/utils/notices.ts:1342` `};`
- `sources/mem0/mem0-ts/src/oss/src/utils/notices.ts:1343` ``
- `sources/mem0/mem0-ts/src/oss/src/utils/notices.ts:1344` `if (!recordNoticeOpportunity(DECAY_USAGE_NOTICE_ID, opportunity)) {`
- `sources/mem0/mem0-ts/src/oss/src/utils/notices.ts:1361` `trigger_reason: trigger.triggerReason,`
- `sources/mem0/mem0-ts/src/oss/src/utils/notices.ts:1362` `...(trigger.deleteCount !== undefined && {`
- `sources/mem0/mem0-ts/src/oss/src/utils/notices.ts:1363` `delete_count: trigger.deleteCount,`
- `sources/mem0/mem0-ts/src/oss/src/utils/notices.ts:1364` `}),`
- `sources/mem0/mem0-ts/src/oss/src/utils/notices.ts:1365` `...(trigger.deletedCount !== undefined && {`
- `sources/mem0/mem0-ts/src/oss/src/utils/notices.ts:1366` `deleted_count: trigger.deletedCount,`
- `sources/mem0/mem0-ts/src/oss/src/utils/notices.ts:1367` `}),`
- `sources/mem0/mem0-ts/src/oss/src/utils/notices.ts:1368` `});`
- `sources/mem0/mem0-ts/src/oss/src/utils/notices.ts:1369` ``
- `sources/mem0/mem0-ts/src/oss/src/utils/notices.ts:1370` `if (decision.displayed && decision.copy) {`
- `sources/mem0/mem0-ts/src/oss/src/utils/notices.ts:1457` `getTemporalFeatureErrorMessage,`
- `sources/mem0/mem0-ts/src/oss/src/utils/notices.ts:1458` `getDecayUsageDeleteCountAfterSuccess,`
- `sources/mem0/mem0-ts/src/oss/src/utils/notices.ts:1459` `getMem0ConfigPath,`
- `sources/mem0/mem0-ts/src/oss/src/utils/notices.ts:1460` `getMem0Dir,`
- `sources/mem0/mem0-ts/src/oss/src/utils/notices.ts:1461` `getNoticeConfigFromPayload,`
- `sources/mem0/mem0-ts/src/oss/src/utils/notices.ts:1462` `getNoticeState,`
- `sources/mem0/mem0-ts/src/oss/src/utils/notices.ts:1463` `hasNoticeCapRoom,`
- `sources/mem0/mem0-ts/src/oss/src/utils/notices.ts:1464` `isDecayUsageDeleteEligible,`
- `sources/mem0/mem0-ts/src/oss/src/utils/notices.ts:1465` `loadMem0Config,`
- `sources/mem0/mem0-ts/src/oss/src/utils/notices.ts:1466` `recordNoticeOpportunity,`
- `sources/mem0/mem0-ts/src/oss/src/utils/notices.ts:1467` `setNoticeState,`
- `sources/mem0/mem0-ts/src/oss/src/utils/notices.ts:1468` `writeMem0ConfigAtomic,`

**operation and route call sites**, body `sources/mem0/mem0-ts/src/oss/src/vector_stores/azure_ai_search.ts:451`; body evidence.

- `sources/mem0/mem0-ts/src/oss/src/vector_stores/azure_ai_search.ts:450` `*/`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/azure_ai_search.ts:451` `async delete(vectorId: string): Promise<void> {`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/azure_ai_search.ts:452` `await this.initialize();`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/azure_ai_search.ts:453` `const response = await this.searchClient.deleteDocuments([`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/azure_ai_search.ts:454` `{ id: vectorId },`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/azure_ai_search.ts:455` `]);`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/azure_ai_search.ts:456` ``
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/azure_ai_search.ts:457` `for (const result of response.results) {`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/azure_ai_search.ts:459` `throw new Error(`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/azure_ai_search.ts:460` `'Delete failed for document ${vectorId}: ${result.errorMessage}',`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/azure_ai_search.ts:461` `);`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/azure_ai_search.ts:462` `}`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/azure_ai_search.ts:463` `}`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/azure_ai_search.ts:464` ``
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/azure_ai_search.ts:465` `console.log(`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/azure_ai_search.ts:466` `'Deleted document with ID '${vectorId}' from index '${this.indexName}'.',`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/azure_ai_search.ts:467` `);`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/azure_ai_search.ts:468` `}`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/azure_ai_search.ts:469` ``
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/azure_ai_search.ts:470` `/**`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/azure_ai_search.ts:545` `*/`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/azure_ai_search.ts:546` `async deleteCol(): Promise<void> {`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/azure_ai_search.ts:547` `await this.initialize();`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/azure_ai_search.ts:548` `await this.indexClient.deleteIndex(this.indexName);`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/azure_ai_search.ts:549` `}`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/azure_ai_search.ts:550` ``
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/azure_ai_search.ts:551` `/**`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/azure_ai_search.ts:552` `* Get information about the index`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/azure_ai_search.ts:711` `// Delete the index`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/azure_ai_search.ts:712` `await this.deleteCol();`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/azure_ai_search.ts:713` ``
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/azure_ai_search.ts:714` `// Recreate the index`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/azure_ai_search.ts:715` `await this.createCol();`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/azure_ai_search.ts:716` `} catch (error) {`

**operation and route call sites**, body `sources/mem0/mem0-ts/src/oss/src/vector_stores/azure_mysql.ts:296`; body evidence.

- `sources/mem0/mem0-ts/src/oss/src/vector_stores/azure_mysql.ts:295` ``
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/azure_mysql.ts:296` `async delete(vectorId: string): Promise<void> {`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/azure_mysql.ts:297` `await this.initialize();`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/azure_mysql.ts:298` `await this.pool!.execute('DELETE FROM ${this.col()} WHERE id = ?', [`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/azure_mysql.ts:299` `vectorId,`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/azure_mysql.ts:300` `]);`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/azure_mysql.ts:301` `}`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/azure_mysql.ts:302` ``
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/azure_mysql.ts:303` `async deleteCol(): Promise<void> {`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/azure_mysql.ts:304` `await this.initialize();`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/azure_mysql.ts:305` `await this.pool!.execute('DROP TABLE IF EXISTS ${this.col()}');`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/azure_mysql.ts:306` `}`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/azure_mysql.ts:307` ``

**operation and route call sites**, body `sources/mem0/mem0-ts/src/oss/src/vector_stores/baidu.ts:542`; body evidence.

- `sources/mem0/mem0-ts/src/oss/src/vector_stores/baidu.ts:541` ``
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/baidu.ts:542` `async delete(vectorId: string): Promise<void> {`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/baidu.ts:543` `const { client } = await this.ready();`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/baidu.ts:544` ``
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/baidu.ts:545` `check(`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/baidu.ts:546` `await client.delete({ ...this.ns, primaryKey: { id: vectorId } }),`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/baidu.ts:547` `'delete '${vectorId}'',`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/baidu.ts:548` `);`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/baidu.ts:549` `}`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/baidu.ts:550` ``
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/baidu.ts:551` `async deleteCol(): Promise<void> {`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/baidu.ts:552` `// The constructor starts initialize() without awaiting it. Let any in-flight run land`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/baidu.ts:553` `// first, otherwise it recreates the table after dropTable() and reset() is a no-op.`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/baidu.ts:554` `await this._initPromise?.catch(() => undefined);`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/baidu.ts:555` `this._initPromise = undefined;`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/baidu.ts:584` `async reset(): Promise<void> {`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/baidu.ts:585` `await this.deleteCol();`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/baidu.ts:586` `await this.initialize();`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/baidu.ts:587` `}`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/baidu.ts:588` ``
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/baidu.ts:589` `async list(`

**operation and route call sites**, body `sources/mem0/mem0-ts/src/oss/src/vector_stores/base.ts:25`; body evidence.

- `sources/mem0/mem0-ts/src/oss/src/vector_stores/base.ts:24` `): Promise<void>;`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/base.ts:25` `delete(vectorId: string): Promise<void>;`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/base.ts:26` `deleteCol(): Promise<void>;`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/base.ts:27` `list(`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/base.ts:28` `filters?: SearchFilters,`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/base.ts:29` `topK?: number,`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/base.ts:30` `): Promise<[VectorStoreResult[], number]>;`

**operation and route call sites**, body `sources/mem0/mem0-ts/src/oss/src/vector_stores/cassandra.ts:229`; body evidence.

- `sources/mem0/mem0-ts/src/oss/src/vector_stores/cassandra.ts:228` ``
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/cassandra.ts:229` `async delete(vectorId: string): Promise<void> {`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/cassandra.ts:230` `await this.initialize();`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/cassandra.ts:231` ``
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/cassandra.ts:232` `await this.client!.execute(`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/cassandra.ts:233` `'`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/cassandra.ts:234` `DELETE FROM ${this.keyspace}.${this.collectionName}`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/cassandra.ts:235` `WHERE id = ?`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/cassandra.ts:236` `',`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/cassandra.ts:237` `[vectorId],`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/cassandra.ts:238` `{ prepare: true },`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/cassandra.ts:241` ``
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/cassandra.ts:242` `async deleteCol(): Promise<void> {`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/cassandra.ts:243` `await this.initialize();`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/cassandra.ts:244` ``
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/cassandra.ts:245` `await this.client!.execute('`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/cassandra.ts:246` `DROP TABLE IF EXISTS ${this.keyspace}.${this.collectionName}`

**operation and route call sites**, body `sources/mem0/mem0-ts/src/oss/src/vector_stores/chroma.ts:198`; body evidence.

- `sources/mem0/mem0-ts/src/oss/src/vector_stores/chroma.ts:197` ``
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/chroma.ts:198` `async delete(vectorId: string): Promise<void> {`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/chroma.ts:199` `const collection = await this.getCollection();`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/chroma.ts:200` `await collection.delete({ ids: [vectorId] });`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/chroma.ts:201` `}`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/chroma.ts:202` ``
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/chroma.ts:203` `async deleteCol(): Promise<void> {`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/chroma.ts:204` `const client = await this.getClient();`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/chroma.ts:205` `await client.deleteCollection({ name: this.collectionName });`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/chroma.ts:206` `this.collectionPromise = undefined;`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/chroma.ts:207` `}`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/chroma.ts:208` ``
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/chroma.ts:209` `async list(`

**operation and route call sites**, body `sources/mem0/mem0-ts/src/oss/src/vector_stores/databricks.ts:62`; body evidence.

- `sources/mem0/mem0-ts/src/oss/src/vector_stores/databricks.ts:61` `): Promise<{ data: any }>;`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/databricks.ts:62` `delete(url: string, config?: Record<string, any>): Promise<{ data: any }>;`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/databricks.ts:63` `}`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/databricks.ts:64` ``
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/databricks.ts:65` `interface DatabricksVector {`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/databricks.ts:66` `id: string;`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/databricks.ts:685` ``
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/databricks.ts:686` `async delete(vectorId: string): Promise<void> {`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/databricks.ts:687` `await this.initialize();`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/databricks.ts:688` ``
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/databricks.ts:689` `await this.executeSql('`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/databricks.ts:690` `DELETE FROM ${this.fullTableName}`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/databricks.ts:691` `WHERE memory_id = ${formatSqlValue(vectorId)}`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/databricks.ts:692` `');`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/databricks.ts:693` `this.requestIndexSync();`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/databricks.ts:694` `}`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/databricks.ts:695` ``
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/databricks.ts:696` `async deleteCol(): Promise<void> {`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/databricks.ts:697` `await this.initialize();`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/databricks.ts:698` ``
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/databricks.ts:699` `try {`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/databricks.ts:700` `await this.httpClient.delete(`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/databricks.ts:701` `'/indexes/${encodeURIComponent(this.fullIndexName)}',`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/databricks.ts:702` `);`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/databricks.ts:703` `} catch (error: any) {`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/databricks.ts:704` `if (error?.response?.status !== 404) {`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/databricks.ts:803` `await this.executeSql(`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/databricks.ts:804` `'DELETE FROM ${this.catalog}.${this.schema}.memory_migrations',`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/databricks.ts:805` `);`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/databricks.ts:806` `await this.executeSql('`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/databricks.ts:807` `INSERT INTO ${this.catalog}.${this.schema}.memory_migrations (user_id)`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/databricks.ts:808` `VALUES (${formatSqlValue(userId)})`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/databricks.ts:909` `),`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/databricks.ts:910` `delete: async (url: string, config?: Record<string, any>) =>`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/databricks.ts:911` `baseClient.delete(`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/databricks.ts:912` `url,`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/databricks.ts:913` `await this.withOAuthHeaders("DELETE", url, config),`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/databricks.ts:914` `),`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/databricks.ts:915` `} as DatabricksHttpClientLike as AxiosInstance;`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/databricks.ts:916` `}`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/databricks.ts:917` ``
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/databricks.ts:918` `private async withOAuthHeaders(`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/databricks.ts:919` `method: "GET" | "POST" | "DELETE",`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/databricks.ts:920` `url: string,`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/databricks.ts:921` `config?: Record<string, any>,`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/databricks.ts:922` `): Promise<Record<string, any>> {`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/databricks.ts:923` `const token = await this.getOAuthAccessToken(`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/databricks.ts:935` `private getOAuthAuthorizationDetails(`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/databricks.ts:936` `method: "GET" | "POST" | "DELETE",`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/databricks.ts:937` `url: string,`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/databricks.ts:938` `): string | undefined {`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/databricks.ts:939` `if (`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/databricks.ts:940` `method === "POST" &&`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/databricks.ts:952` `url.endsWith("/upsert-data") ||`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/databricks.ts:953` `url.endsWith("/delete-data"))`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/databricks.ts:954` `) {`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/databricks.ts:955` `return buildDatabricksAuthorizationDetails(`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/databricks.ts:956` `this.fullIndexName,`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/databricks.ts:957` `"WriteVectorIndex",`

**operation and route call sites**, body `sources/mem0/mem0-ts/src/oss/src/vector_stores/elasticsearch.ts:252`; body evidence.

- `sources/mem0/mem0-ts/src/oss/src/vector_stores/elasticsearch.ts:251` ``
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/elasticsearch.ts:252` `async delete(vectorId: string): Promise<void> {`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/elasticsearch.ts:253` `await this.initialize();`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/elasticsearch.ts:254` `await this.client.delete({`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/elasticsearch.ts:255` `index: this.collectionName,`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/elasticsearch.ts:256` `id: vectorId,`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/elasticsearch.ts:257` `});`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/elasticsearch.ts:258` `}`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/elasticsearch.ts:259` ``
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/elasticsearch.ts:260` `async deleteCol(): Promise<void> {`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/elasticsearch.ts:261` `await this.initialize();`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/elasticsearch.ts:262` `await this.client.indices.delete({ index: this.collectionName });`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/elasticsearch.ts:263` `}`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/elasticsearch.ts:264` ``
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/elasticsearch.ts:265` `async list(`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/elasticsearch.ts:266` `filters?: SearchFilters,`

**operation and route call sites**, body `sources/mem0/mem0-ts/src/oss/src/vector_stores/langchain.ts:160`; body evidence.

- `sources/mem0/mem0-ts/src/oss/src/vector_stores/langchain.ts:159` `console.error(`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/langchain.ts:160` `'LangchainVectorStore: The 'update' method is not directly supported. Use delete followed by insert.',`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/langchain.ts:161` `);`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/langchain.ts:162` `throw new Error(`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/langchain.ts:163` `"Method 'update' not supported by LangchainVectorStore wrapper.",`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/langchain.ts:164` `);`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/langchain.ts:167` ``
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/langchain.ts:168` `async delete(vectorId: string): Promise<void> {`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/langchain.ts:169` `// Check if the underlying store supports deletion by ID`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/langchain.ts:170` `if (typeof (this.lcStore as any).delete === "function") {`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/langchain.ts:171` `try {`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/langchain.ts:172` `// We need to delete based on our stored _mem0_id.`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/langchain.ts:173` `// Langchain's delete often takes its own internal IDs or filter.`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/langchain.ts:174` `// Attempting deletion via filter is the most likely approach.`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/langchain.ts:175` `console.warn(`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/langchain.ts:176` `"LangchainVectorStore: Attempting delete via filter on '_mem0_id'. Success depends on the specific Langchain VectorStore's delete implementation.",`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/langchain.ts:177` `);`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/langchain.ts:178` `await (this.lcStore as any).delete({ filter: { _mem0_id: vectorId } });`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/langchain.ts:179` `// OR if it takes IDs directly (less common for *our* IDs):`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/langchain.ts:180` `// await (this.lcStore as any).delete({ ids: [vectorId] });`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/langchain.ts:181` `} catch (e) {`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/langchain.ts:182` `console.error(`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/langchain.ts:183` `'LangchainVectorStore: Delete failed. Underlying store's delete method might expect different arguments or filters. Error: ${e}',`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/langchain.ts:184` `);`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/langchain.ts:185` `throw new Error('Delete failed in underlying Langchain store: ${e}');`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/langchain.ts:186` `}`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/langchain.ts:187` `} else {`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/langchain.ts:188` `console.error(`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/langchain.ts:189` `'LangchainVectorStore: The underlying Langchain store instance does not seem to support a 'delete' method.',`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/langchain.ts:190` `);`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/langchain.ts:191` `throw new Error(`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/langchain.ts:192` `"Method 'delete' not available on the provided Langchain VectorStore client.",`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/langchain.ts:193` `);`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/langchain.ts:194` `}`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/langchain.ts:195` `}`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/langchain.ts:196` ``
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/langchain.ts:210` ``
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/langchain.ts:211` `async deleteCol(): Promise<void> {`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/langchain.ts:212` `console.error(`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/langchain.ts:213` `'LangchainVectorStore: The 'deleteCol' method is not supported by the generic LangchainVectorStore wrapper.',`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/langchain.ts:214` `);`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/langchain.ts:215` `throw new Error(`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/langchain.ts:216` `"Method 'deleteCol' not supported by LangchainVectorStore wrapper.",`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/langchain.ts:217` `);`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/langchain.ts:218` `}`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/langchain.ts:219` ``
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/langchain.ts:220` `// --- Wrapper-Specific Methods (In-Memory User ID) ---`

**operation and route call sites**, body `sources/mem0/mem0-ts/src/oss/src/vector_stores/memory.ts:34`; body evidence.

- `sources/mem0/mem0-ts/src/oss/src/vector_stores/memory.ts:33` `payload[snake] = payload[camel];`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/memory.ts:34` `delete payload[camel];`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/memory.ts:35` `}`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/memory.ts:36` `}`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/memory.ts:37` `return payload;`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/memory.ts:38` `}`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/memory.ts:421` ``
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/memory.ts:422` `async delete(vectorId: string): Promise<void> {`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/memory.ts:423` `this.db.prepare('DELETE FROM vectors WHERE id = ?').run(vectorId);`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/memory.ts:424` `}`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/memory.ts:425` ``
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/memory.ts:426` `async deleteCol(): Promise<void> {`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/memory.ts:427` `this.db.exec('DROP TABLE IF EXISTS vectors');`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/memory.ts:428` `this.init();`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/memory.ts:429` `}`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/memory.ts:430` ``
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/memory.ts:481` `async setUserId(userId: string): Promise<void> {`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/memory.ts:482` `this.db.prepare('DELETE FROM memory_migrations').run();`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/memory.ts:483` `this.db`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/memory.ts:484` `.prepare('INSERT INTO memory_migrations (user_id) VALUES (?)')`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/memory.ts:485` `.run(userId);`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/memory.ts:486` `}`

**operation and route call sites**, body `sources/mem0/mem0-ts/src/oss/src/vector_stores/milvus.ts:398`; body evidence.

- `sources/mem0/mem0-ts/src/oss/src/vector_stores/milvus.ts:397` ``
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/milvus.ts:398` `async delete(vectorId: string): Promise<void> {`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/milvus.ts:399` `await this.client.delete({`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/milvus.ts:400` `collection_name: this.collectionName,`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/milvus.ts:401` `ids: [vectorId],`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/milvus.ts:402` `});`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/milvus.ts:403` `}`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/milvus.ts:404` ``
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/milvus.ts:405` `async deleteCol(): Promise<void> {`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/milvus.ts:406` `await this.client.dropCollection({`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/milvus.ts:407` `collection_name: this.collectionName,`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/milvus.ts:408` `});`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/milvus.ts:409` `}`

**operation and route call sites**, body `sources/mem0/mem0-ts/src/oss/src/vector_stores/mongodb.ts:114`; body evidence.

- `sources/mem0/mem0-ts/src/oss/src/vector_stores/mongodb.ts:113` `await this.collection.insertOne({ _id: 0 as any, placeholder: true });`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/mongodb.ts:114` `await this.collection.deleteOne({ _id: 0 as any });`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/mongodb.ts:115` `} else {`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/mongodb.ts:116` `this.collection = this.db.collection(this.collectionName);`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/mongodb.ts:117` `}`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/mongodb.ts:118` ``
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/mongodb.ts:409` ``
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/mongodb.ts:410` `async delete(vectorId: string): Promise<void> {`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/mongodb.ts:411` `await this.initialize();`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/mongodb.ts:412` `try {`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/mongodb.ts:413` `const result = await this.collection.deleteOne({ _id: vectorId as any });`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/mongodb.ts:414` `if (result.deletedCount === 0) {`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/mongodb.ts:415` `console.warn('No document found with ID '${vectorId}' to delete.');`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/mongodb.ts:416` `}`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/mongodb.ts:417` `} catch (error) {`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/mongodb.ts:418` `console.error("Error deleting document:", error);`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/mongodb.ts:419` `throw error;`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/mongodb.ts:420` `}`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/mongodb.ts:421` `}`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/mongodb.ts:422` ``
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/mongodb.ts:423` `async deleteCol(): Promise<void> {`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/mongodb.ts:424` `await this.initialize();`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/mongodb.ts:425` `try {`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/mongodb.ts:426` `await this.collection.drop();`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/mongodb.ts:427` `} catch (error) {`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/mongodb.ts:428` `console.error("Error deleting collection:", error);`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/mongodb.ts:429` `throw error;`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/mongodb.ts:430` `}`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/mongodb.ts:431` `}`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/mongodb.ts:432` ``

**operation and route call sites**, body `sources/mem0/mem0-ts/src/oss/src/vector_stores/neptune_analytics.ts:254`; body evidence.

- `sources/mem0/mem0-ts/src/oss/src/vector_stores/neptune_analytics.ts:253` ``
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/neptune_analytics.ts:254` `async delete(vectorId: string): Promise<void> {`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/neptune_analytics.ts:255` `await this.executeQuery(`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/neptune_analytics.ts:256` `'`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/neptune_analytics.ts:257` `MATCH (n:${this.collectionLabelExpr} {\'~id\': $vectorId})`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/neptune_analytics.ts:258` `DETACH DELETE n`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/neptune_analytics.ts:259` `',`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/neptune_analytics.ts:260` `{`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/neptune_analytics.ts:261` `vectorId,`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/neptune_analytics.ts:262` `},`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/neptune_analytics.ts:265` ``
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/neptune_analytics.ts:266` `async deleteCol(): Promise<void> {`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/neptune_analytics.ts:267` `await this.executeQuery(`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/neptune_analytics.ts:268` `'`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/neptune_analytics.ts:269` `MATCH (n:${this.collectionLabelExpr})`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/neptune_analytics.ts:270` `DETACH DELETE n`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/neptune_analytics.ts:271` `',`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/neptune_analytics.ts:272` `);`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/neptune_analytics.ts:273` `}`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/neptune_analytics.ts:274` ``
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/neptune_analytics.ts:386` `MATCH (n:${this.collectionLabelExpr} {\'~id\': nodeId})`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/neptune_analytics.ts:387` `DETACH DELETE n`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/neptune_analytics.ts:388` `',`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/neptune_analytics.ts:389` `{`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/neptune_analytics.ts:390` `nodeIds,`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/neptune_analytics.ts:391` `},`

**operation and route call sites**, body `sources/mem0/mem0-ts/src/oss/src/vector_stores/opensearch.ts:259`; body evidence.

- `sources/mem0/mem0-ts/src/oss/src/vector_stores/opensearch.ts:258` `const failedItem = response.items?.find((item) => {`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/opensearch.ts:259` `const action = item.index || item.create || item.update || item.delete;`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/opensearch.ts:260` `return action?.error;`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/opensearch.ts:261` `});`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/opensearch.ts:262` ``
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/opensearch.ts:263` `throw new Error(`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/opensearch.ts:389` ``
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/opensearch.ts:390` `async delete(vectorId: string): Promise<void> {`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/opensearch.ts:391` `await this.initialize();`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/opensearch.ts:392` `try {`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/opensearch.ts:393` `await this.client.delete({`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/opensearch.ts:394` `index: this.collectionName,`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/opensearch.ts:395` `id: vectorId,`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/opensearch.ts:396` `refresh: this.autoRefresh,`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/opensearch.ts:397` `});`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/opensearch.ts:405` ``
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/opensearch.ts:406` `async deleteCol(): Promise<void> {`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/opensearch.ts:407` `await this.initialize();`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/opensearch.ts:408` `if (!(await this.indexExists(this.collectionName))) return;`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/opensearch.ts:409` `await this.client.indices.delete({ index: this.collectionName });`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/opensearch.ts:410` `}`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/opensearch.ts:411` ``
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/opensearch.ts:412` `async list(`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/opensearch.ts:413` `filters?: SearchFilters,`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/opensearch.ts:445` `await this.initialize();`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/opensearch.ts:446` `await this.deleteCol();`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/opensearch.ts:447` `await this.createCol(this.collectionName, this.embeddingModelDims);`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/opensearch.ts:448` `}`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/opensearch.ts:449` ``
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/opensearch.ts:450` `async getUserId(): Promise<string> {`

**operation and route call sites**, body `sources/mem0/mem0-ts/src/oss/src/vector_stores/oracledb.ts:683`; body evidence.

- `sources/mem0/mem0-ts/src/oss/src/vector_stores/oracledb.ts:682` ``
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/oracledb.ts:683` `async delete(vectorId: string): Promise<void> {`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/oracledb.ts:684` `await this.initialize();`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/oracledb.ts:685` ``
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/oracledb.ts:686` `await this.withConnection(`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/oracledb.ts:687` `(connection) =>`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/oracledb.ts:688` `connection.execute(`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/oracledb.ts:689` `'DELETE FROM ${this.collectionName} WHERE id = :vector_id',`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/oracledb.ts:690` `{ vector_id: vectorId },`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/oracledb.ts:691` `),`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/oracledb.ts:692` `true,`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/oracledb.ts:693` `);`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/oracledb.ts:695` ``
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/oracledb.ts:696` `async deleteCol(): Promise<void> {`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/oracledb.ts:697` `await this.initialize();`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/oracledb.ts:698` ``
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/oracledb.ts:699` `await this.withConnection(`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/oracledb.ts:700` `(connection) =>`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/oracledb.ts:701` `connection.execute('DROP TABLE ${this.collectionName} PURGE'),`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/oracledb.ts:702` `true,`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/oracledb.ts:703` `);`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/oracledb.ts:704` `}`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/oracledb.ts:705` ``
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/oracledb.ts:750` `await this.withConnection(async (connection) => {`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/oracledb.ts:751` `await connection.execute("DELETE FROM memory_migrations WHERE id = 1");`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/oracledb.ts:752` `await connection.execute(`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/oracledb.ts:753` `"INSERT INTO memory_migrations (id, user_id) VALUES (1, :user_id)",`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/oracledb.ts:754` `{ user_id: userId },`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/oracledb.ts:755` `);`

**operation and route call sites**, body `sources/mem0/mem0-ts/src/oss/src/vector_stores/pgvector.ts:467`; body evidence.

- `sources/mem0/mem0-ts/src/oss/src/vector_stores/pgvector.ts:466` ``
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/pgvector.ts:467` `async delete(vectorId: string): Promise<void> {`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/pgvector.ts:468` `await this.client.query('DELETE FROM ${this.col()} WHERE id = $1', [`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/pgvector.ts:469` `vectorId,`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/pgvector.ts:470` `]);`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/pgvector.ts:471` `}`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/pgvector.ts:472` ``
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/pgvector.ts:473` `async deleteCol(): Promise<void> {`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/pgvector.ts:474` `await this.client.query('DROP TABLE IF EXISTS ${this.col()}');`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/pgvector.ts:475` `}`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/pgvector.ts:476` ``
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/pgvector.ts:477` `private async listCols(): Promise<string[]> {`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/pgvector.ts:551` `async setUserId(userId: string): Promise<void> {`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/pgvector.ts:552` `await this.client.query("DELETE FROM memory_migrations");`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/pgvector.ts:553` `await this.client.query(`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/pgvector.ts:554` `"INSERT INTO memory_migrations (user_id) VALUES ($1)",`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/pgvector.ts:555` `[userId],`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/pgvector.ts:556` `);`

**operation and route call sites**, body `sources/mem0/mem0-ts/src/oss/src/vector_stores/pinecone.ts:313`; body evidence.

- `sources/mem0/mem0-ts/src/oss/src/vector_stores/pinecone.ts:312` ``
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/pinecone.ts:313` `async delete(vectorId: string): Promise<void> {`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/pinecone.ts:314` `await this.initialize();`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/pinecone.ts:315` `await this.namespacedIndex().deleteOne({ id: vectorId });`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/pinecone.ts:316` `}`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/pinecone.ts:317` ``
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/pinecone.ts:318` `async deleteCol(): Promise<void> {`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/pinecone.ts:319` `if (this.namespace) {`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/pinecone.ts:320` `await this.initialize();`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/pinecone.ts:321` `await this.namespacedIndex().deleteAll();`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/pinecone.ts:322` `return;`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/pinecone.ts:323` `}`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/pinecone.ts:324` `if (this._initPromise) {`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/pinecone.ts:325` `await this._initPromise.catch(() => {});`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/pinecone.ts:326` `}`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/pinecone.ts:327` `await this.client.deleteIndex(this.collectionName);`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/pinecone.ts:328` `this._index = undefined;`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/pinecone.ts:329` `this._initPromise = undefined;`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/pinecone.ts:330` `}`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/pinecone.ts:331` ``

**operation and route call sites**, body `sources/mem0/mem0-ts/src/oss/src/vector_stores/qdrant.ts:445`; body evidence.

- `sources/mem0/mem0-ts/src/oss/src/vector_stores/qdrant.ts:444` ``
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/qdrant.ts:445` `async delete(vectorId: string): Promise<void> {`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/qdrant.ts:446` `await this.initialize();`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/qdrant.ts:447` `await this.client.delete(this.collectionName, {`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/qdrant.ts:448` `points: [vectorId],`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/qdrant.ts:449` `});`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/qdrant.ts:450` `}`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/qdrant.ts:451` ``
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/qdrant.ts:452` `async deleteCol(): Promise<void> {`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/qdrant.ts:453` `await this.initialize();`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/qdrant.ts:454` `await this.client.deleteCollection(this.collectionName);`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/qdrant.ts:455` `}`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/qdrant.ts:456` ``
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/qdrant.ts:457` `async list(`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/qdrant.ts:458` `filters?: SearchFilters,`

**operation and route call sites**, body `sources/mem0/mem0-ts/src/oss/src/vector_stores/redis.ts:617`; body evidence.

- `sources/mem0/mem0-ts/src/oss/src/vector_stores/redis.ts:616` ``
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/redis.ts:617` `async delete(vectorId: string): Promise<void> {`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/redis.ts:618` `await this.initialize();`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/redis.ts:619` `try {`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/redis.ts:620` `// Check if memory exists first`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/redis.ts:621` `const key = '${this.indexPrefix}:${vectorId}';`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/redis.ts:632` `if (!result) {`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/redis.ts:633` `throw new Error('Failed to delete memory with ID ${vectorId}');`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/redis.ts:634` `}`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/redis.ts:635` ``
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/redis.ts:636` `console.log('Successfully deleted memory with ID ${vectorId}');`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/redis.ts:637` `} catch (error) {`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/redis.ts:638` `console.error("Error deleting memory:", error);`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/redis.ts:639` `throw error;`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/redis.ts:640` `}`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/redis.ts:641` `}`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/redis.ts:642` ``
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/redis.ts:643` `async deleteCol(): Promise<void> {`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/redis.ts:644` `await this.initialize();`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/redis.ts:645` `await this.client.ft.dropIndex(this.indexName);`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/redis.ts:646` `}`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/redis.ts:647` ``

**operation and route call sites**, body `sources/mem0/mem0-ts/src/oss/src/vector_stores/s3_vectors.ts:254`; body evidence.

- `sources/mem0/mem0-ts/src/oss/src/vector_stores/s3_vectors.ts:253` ``
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/s3_vectors.ts:254` `async delete(vectorId: string): Promise<void> {`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/s3_vectors.ts:255` `await this.initialize();`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/s3_vectors.ts:256` ``
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/s3_vectors.ts:257` `const sdk = await this.getSdk();`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/s3_vectors.ts:258` `const client = await this.getClient();`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/s3_vectors.ts:259` `await client.send(`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/s3_vectors.ts:260` `new sdk.DeleteVectorsCommand({`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/s3_vectors.ts:261` `vectorBucketName: this.vectorBucketName,`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/s3_vectors.ts:262` `indexName: this.collectionName,`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/s3_vectors.ts:263` `keys: [vectorId],`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/s3_vectors.ts:264` `}),`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/s3_vectors.ts:267` ``
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/s3_vectors.ts:268` `async deleteCol(): Promise<void> {`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/s3_vectors.ts:269` `await this.initialize();`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/s3_vectors.ts:270` ``
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/s3_vectors.ts:271` `const sdk = await this.getSdk();`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/s3_vectors.ts:272` `const client = await this.getClient();`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/s3_vectors.ts:274` `await client.send(`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/s3_vectors.ts:275` `new sdk.DeleteIndexCommand({`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/s3_vectors.ts:276` `vectorBucketName: this.vectorBucketName,`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/s3_vectors.ts:277` `indexName: this.collectionName,`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/s3_vectors.ts:278` `}),`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/s3_vectors.ts:279` `);`

**operation and route call sites**, body `sources/mem0/mem0-ts/src/oss/src/vector_stores/supabase.ts:318`; body evidence.

- `sources/mem0/mem0-ts/src/oss/src/vector_stores/supabase.ts:317` ``
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/supabase.ts:318` `async delete(vectorId: string): Promise<void> {`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/supabase.ts:319` `await this.initialize();`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/supabase.ts:320` `try {`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/supabase.ts:321` `const { error } = await this.client`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/supabase.ts:322` `.from(this.tableName)`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/supabase.ts:323` `.delete()`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/supabase.ts:324` `.eq("id", vectorId);`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/supabase.ts:325` ``
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/supabase.ts:326` `if (error) throw error;`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/supabase.ts:327` `} catch (error) {`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/supabase.ts:328` `console.error("Error deleting vector:", error);`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/supabase.ts:329` `throw error;`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/supabase.ts:330` `}`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/supabase.ts:331` `}`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/supabase.ts:332` ``
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/supabase.ts:333` `async deleteCol(): Promise<void> {`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/supabase.ts:334` `await this.initialize();`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/supabase.ts:335` `try {`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/supabase.ts:336` `const { error } = await this.client`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/supabase.ts:337` `.from(this.tableName)`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/supabase.ts:338` `.delete()`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/supabase.ts:339` `.neq("id", ""); // Delete all rows`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/supabase.ts:340` ``
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/supabase.ts:341` `if (error) throw error;`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/supabase.ts:342` `} catch (error) {`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/supabase.ts:343` `console.error("Error deleting collection:", error);`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/supabase.ts:344` `throw error;`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/supabase.ts:345` `}`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/supabase.ts:346` `}`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/supabase.ts:347` ``
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/supabase.ts:453` `try {`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/supabase.ts:454` `const { error: deleteError } = await this.client`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/supabase.ts:455` `.from("memory_migrations")`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/supabase.ts:456` `.delete()`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/supabase.ts:457` `.neq("user_id", "");`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/supabase.ts:458` ``
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/supabase.ts:459` `if (deleteError) throw deleteError;`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/supabase.ts:460` ``
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/supabase.ts:461` `const { error: insertError } = await this.client`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/supabase.ts:462` `.from("memory_migrations")`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/supabase.ts:463` `.insert({ user_id: userId });`

**operation and route call sites**, body `sources/mem0/mem0-ts/src/oss/src/vector_stores/turbopuffer.ts:169`; body evidence.

- `sources/mem0/mem0-ts/src/oss/src/vector_stores/turbopuffer.ts:168` ``
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/turbopuffer.ts:169` `async delete(vectorId: string): Promise<void> {`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/turbopuffer.ts:170` `const ns = await this.getNs();`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/turbopuffer.ts:171` `await ns.write({ deletes: [vectorId] });`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/turbopuffer.ts:172` `}`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/turbopuffer.ts:173` ``
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/turbopuffer.ts:174` `async deleteCol(): Promise<void> {`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/turbopuffer.ts:175` `const ns = await this.getNs();`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/turbopuffer.ts:176` `await ns.deleteAll();`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/turbopuffer.ts:177` `}`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/turbopuffer.ts:178` ``
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/turbopuffer.ts:179` `async list(`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/turbopuffer.ts:180` `filters?: SearchFilters,`

**operation and route call sites**, body `sources/mem0/mem0-ts/src/oss/src/vector_stores/upstash_vector.ts:159`; body evidence.

- `sources/mem0/mem0-ts/src/oss/src/vector_stores/upstash_vector.ts:158` ``
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/upstash_vector.ts:159` `async delete(vectorId: string): Promise<void> {`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/upstash_vector.ts:160` `await this.initialize();`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/upstash_vector.ts:161` `await this.client.delete(vectorId, { namespace: this.collectionName });`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/upstash_vector.ts:162` `}`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/upstash_vector.ts:163` ``
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/upstash_vector.ts:164` `async deleteCol(): Promise<void> {`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/upstash_vector.ts:165` `await this.initialize();`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/upstash_vector.ts:166` `await this.client.reset({ namespace: this.collectionName });`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/upstash_vector.ts:167` `}`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/upstash_vector.ts:168` ``
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/upstash_vector.ts:217` `async reset(): Promise<void> {`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/upstash_vector.ts:218` `await this.deleteCol();`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/upstash_vector.ts:219` `}`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/upstash_vector.ts:220` ``
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/upstash_vector.ts:221` `private parseResult(result: QueryResult<UpstashMetadata>): VectorStoreResult {`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/upstash_vector.ts:222` `return {`

**operation and route call sites**, body `sources/mem0/mem0-ts/src/oss/src/vector_stores/valkey.ts:544`; body evidence.

- `sources/mem0/mem0-ts/src/oss/src/vector_stores/valkey.ts:543` ``
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/valkey.ts:544` `async delete(vectorId: string): Promise<void> {`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/valkey.ts:545` `const key = '${this.indexPrefix}:${vectorId}';`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/valkey.ts:546` `const exists = await this.client.exists(key);`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/valkey.ts:547` `if (!exists) {`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/valkey.ts:548` `console.warn('Memory with ID ${vectorId} does not exist');`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/valkey.ts:553` ``
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/valkey.ts:554` `async deleteCol(): Promise<void> {`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/valkey.ts:555` `try {`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/valkey.ts:556` `await this.client.call("FT.DROPINDEX", this.collectionName);`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/valkey.ts:557` `} catch (error: any) {`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/valkey.ts:558` `const message = String(error?.message ?? error);`

**operation and route call sites**, body `sources/mem0/mem0-ts/src/oss/src/vector_stores/vectorize.ts:191`; body evidence.

- `sources/mem0/mem0-ts/src/oss/src/vector_stores/vectorize.ts:190` ``
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/vectorize.ts:191` `async delete(vectorId: string): Promise<void> {`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/vectorize.ts:192` `await this.initialize();`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/vectorize.ts:193` `try {`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/vectorize.ts:194` `await this.client?.vectorize.indexes.deleteByIds(this.indexName, {`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/vectorize.ts:195` `account_id: this.accountId,`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/vectorize.ts:196` `ids: [vectorId],`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/vectorize.ts:197` `});`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/vectorize.ts:198` `} catch (error) {`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/vectorize.ts:199` `console.error("Error deleting vector:", error);`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/vectorize.ts:200` `throw new Error(`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/vectorize.ts:201` `'Failed to delete vector: ${error instanceof Error ? error.message : String(error)}',`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/vectorize.ts:202` `);`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/vectorize.ts:203` `}`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/vectorize.ts:204` `}`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/vectorize.ts:205` ``
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/vectorize.ts:206` `async deleteCol(): Promise<void> {`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/vectorize.ts:207` `await this.initialize();`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/vectorize.ts:208` `try {`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/vectorize.ts:209` `await this.client?.vectorize.indexes.delete(this.indexName, {`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/vectorize.ts:210` `account_id: this.accountId,`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/vectorize.ts:211` `});`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/vectorize.ts:212` `} catch (error) {`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/vectorize.ts:213` `console.error("Error deleting collection:", error);`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/vectorize.ts:214` `throw new Error(`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/vectorize.ts:215` `'Failed to delete collection: ${error instanceof Error ? error.message : String(error)}',`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/vectorize.ts:216` `);`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/vectorize.ts:217` `}`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/vectorize.ts:218` `}`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/vectorize.ts:219` ``

**operation and route call sites**, body `sources/mem0/mem0-ts/src/oss/src/vector_stores/vertex_ai_vector_search.ts:291`; body evidence.

- `sources/mem0/mem0-ts/src/oss/src/vector_stores/vertex_ai_vector_search.ts:290` ``
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/vertex_ai_vector_search.ts:291` `async delete(vectorId: string): Promise<void> {`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/vertex_ai_vector_search.ts:292` `await this.initialize();`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/vertex_ai_vector_search.ts:293` `try {`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/vertex_ai_vector_search.ts:294` `await this.indexClient.removeDatapoints({`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/vertex_ai_vector_search.ts:295` `index: this.indexPath,`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/vertex_ai_vector_search.ts:306` ``
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/vertex_ai_vector_search.ts:307` `async deleteCol(): Promise<void> {`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/vertex_ai_vector_search.ts:308` `console.warn(`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/vertex_ai_vector_search.ts:309` `"Delete collection operation is not supported for Google Matching Engine",`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/vertex_ai_vector_search.ts:310` `);`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/vertex_ai_vector_search.ts:311` `}`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/vertex_ai_vector_search.ts:312` ``
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/vertex_ai_vector_search.ts:313` `async list(`

**operation and route call sites**, body `sources/mem0/mem0-ts/src/oss/src/vector_stores/weaviate.ts:201`; body evidence.

- `sources/mem0/mem0-ts/src/oss/src/vector_stores/weaviate.ts:200` ``
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/weaviate.ts:201` `async delete(vectorId: string): Promise<void> {`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/weaviate.ts:202` `await this.initialize();`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/weaviate.ts:203` `await this._col.data.deleteById(vectorId);`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/weaviate.ts:204` `}`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/weaviate.ts:205` ``
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/weaviate.ts:206` `async deleteCol(): Promise<void> {`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/weaviate.ts:207` `await this.initialize();`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/weaviate.ts:208` `await this._client.collections.delete(this._config.collectionName);`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/weaviate.ts:209` `}`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/weaviate.ts:210` ``
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/weaviate.ts:211` `async list(`
- `sources/mem0/mem0-ts/src/oss/src/vector_stores/weaviate.ts:212` `filters?: SearchFilters,`

**operation and route call sites**, body `sources/mem0/server/dashboard/src/app/api/auth/refresh/route.ts:45`; body evidence.

- `sources/mem0/server/dashboard/src/app/api/auth/refresh/route.ts:44` `// Refresh token is invalid , clear cookie`
- `sources/mem0/server/dashboard/src/app/api/auth/refresh/route.ts:45` `cookieStore.delete(COOKIE_NAME);`
- `sources/mem0/server/dashboard/src/app/api/auth/refresh/route.ts:46` `return NextResponse.json({ error: "Refresh failed" }, { status: 401 });`
- `sources/mem0/server/dashboard/src/app/api/auth/refresh/route.ts:47` `}`
- `sources/mem0/server/dashboard/src/app/api/auth/refresh/route.ts:48` ``
- `sources/mem0/server/dashboard/src/app/api/auth/refresh/route.ts:49` `const data = await res.json();`
- `sources/mem0/server/dashboard/src/app/api/auth/refresh/route.ts:70` ``
- `sources/mem0/server/dashboard/src/app/api/auth/refresh/route.ts:71` `export async function DELETE() {`
- `sources/mem0/server/dashboard/src/app/api/auth/refresh/route.ts:72` `const cookieStore = await cookies();`
- `sources/mem0/server/dashboard/src/app/api/auth/refresh/route.ts:73` `cookieStore.delete(COOKIE_NAME);`
- `sources/mem0/server/dashboard/src/app/api/auth/refresh/route.ts:74` `return NextResponse.json({ ok: true });`
- `sources/mem0/server/dashboard/src/app/api/auth/refresh/route.ts:75` `}`

**operation and route call sites**, body `sources/mem0/server/dashboard/tailwind.config.ts:56`; body evidence.

- `sources/mem0/server/dashboard/tailwind.config.ts:55` `search: "var(--on-surface-event-search)",`
- `sources/mem0/server/dashboard/tailwind.config.ts:56` `delete: "var(--on-surface-event-delete)",`
- `sources/mem0/server/dashboard/tailwind.config.ts:57` `get: "var(--on-surface-event-get)",`
- `sources/mem0/server/dashboard/tailwind.config.ts:58` `},`
- `sources/mem0/server/dashboard/tailwind.config.ts:59` `positive: {`
- `sources/mem0/server/dashboard/tailwind.config.ts:60` `primary: "var(--on-surface-positive-primary)",`
- `sources/mem0/server/dashboard/tailwind.config.ts:90` `search: "var(--surface-event-search)",`
- `sources/mem0/server/dashboard/tailwind.config.ts:91` `delete: "var(--surface-event-delete)",`
- `sources/mem0/server/dashboard/tailwind.config.ts:92` `get: "var(--surface-event-get)",`
- `sources/mem0/server/dashboard/tailwind.config.ts:93` `},`
- `sources/mem0/server/dashboard/tailwind.config.ts:94` `positive: {`
- `sources/mem0/server/dashboard/tailwind.config.ts:95` `primary: "var(--surface-positive-primary)",`

**delete_memory**, body `sources/mem0/server/main.py:518`; implementation/call-through.

- `sources/mem0/server/main.py:515` `@app.delete("/memories/{memory_id}", summary="Delete a memory", response_model=MessageResponse)`
- `sources/mem0/server/main.py:516` `def delete_memory(memory_id: str, _auth=Depends(verify_auth)):`
- `sources/mem0/server/main.py:519` `get_memory_instance().delete(memory_id=memory_id)`

**delete_all_memories**, body `sources/mem0/server/main.py:535`; implementation/call-through.

- `sources/mem0/server/main.py:527` `@app.delete("/memories", summary="Delete all memories", response_model=MessageResponse)`
- `sources/mem0/server/main.py:528` `def delete_all_memories(`
- `sources/mem0/server/main.py:541` `get_memory_instance().delete_all(**params)`

**delete_entity**, body `sources/mem0/server/routers/entities.py:72`; implementation/call-through.

- `sources/mem0/server/routers/entities.py:70` `@router.delete("/{entity_type}/{entity_id}", response_model=MessageResponse)`
- `sources/mem0/server/routers/entities.py:71` `def delete_entity(entity_type: EntityType, entity_id: str, _auth=Depends(require_admin)):`
- `sources/mem0/server/routers/entities.py:73` `get_memory_instance().delete_all(**{TYPE_TO_FIELD[entity_type]: entity_id})`

</details>

## 3. What reaches the agent

System assembly returns `# Mem0 Memory`, mode, user and tool instructions (`sources/mem0/integrations/hermes-plugin-mem0/__init__.py:261`). `_start_prefetch` assembles `## Mem0 Memory` and bullets of returned memory strings (`sources/mem0/integrations/hermes-plugin-mem0/__init__.py:281`); `prefetch` waits briefly, otherwise returns empty (`sources/mem0/integrations/hermes-plugin-mem0/__init__.py:302`). No character truncation in that assembly. Search tool clamps top_k to 1..50, default 10, and returns IDs/text/scores as JSON (`sources/mem0/integrations/hermes-plugin-mem0/__init__.py:336`). The 450-character setting is capture input, not the recall output cap (`sources/mem0/integrations/hermes-plugin-mem0/__init__.py:313`).

Host assembly: `hermes-agent/agent/memory_manager.py:352` joins provider system blocks. `hermes-agent/agent/turn_context.py:1081` requests prefetch for nontrivial turns; `hermes-agent/agent/turn_context.py:120` fences it; `hermes-agent/agent/turn_context.py:136` appends it to API-bound user content. Tool results are a third channel. Optional host spill is at `hermes-agent/agent/memory_manager.py:419`, not a universal provider cap. Built-in memory remains separately controlled (`hermes-agent/agent/system_prompt.py:542`).

<details>
<summary>Context and tool-result assembly quotes</summary>

`system_prompt_block`, body `sources/mem0/integrations/hermes-plugin-mem0/__init__.py:259`:

- `sources/mem0/integrations/hermes-plugin-mem0/__init__.py:261` `return f"# Mem0 Memory\nActive. Mode: {mode_label}. User: {self._user_id}.\n{_PROMPT_BODY}{rerank_note}"`

`_consume_prefetch_result`, body `sources/mem0/integrations/hermes-plugin-mem0/__init__.py:268`:

- `sources/mem0/integrations/hermes-plugin-mem0/__init__.py:270` `return None`
- `sources/mem0/integrations/hermes-plugin-mem0/__init__.py:272` `return result`

`_start_prefetch`, body `sources/mem0/integrations/hermes-plugin-mem0/__init__.py:275`:

- `sources/mem0/integrations/hermes-plugin-mem0/__init__.py:282` `body = "## Mem0 Memory\n" + "\n".join(f"- {line}" for line in lines) if lines else ""`

`prefetch`, body `sources/mem0/integrations/hermes-plugin-mem0/__init__.py:296`:

- `sources/mem0/integrations/hermes-plugin-mem0/__init__.py:297` `return cached`
- `sources/mem0/integrations/hermes-plugin-mem0/__init__.py:302` `thread.join(timeout=_PREFETCH_WAIT_SECS)`
- `sources/mem0/integrations/hermes-plugin-mem0/__init__.py:303` `return self._consume_prefetch_result(query) or ""  # slow backend: skip injection; mem0_search remains the backstop`

`get_tool_schemas`, body `sources/mem0/integrations/hermes-plugin-mem0/__init__.py:329`:

- `sources/mem0/integrations/hermes-plugin-mem0/__init__.py:329` `return list(TOOL_SCHEMAS)`

`_tool_search`, body `sources/mem0/integrations/hermes-plugin-mem0/__init__.py:336`:

- `sources/mem0/integrations/hermes-plugin-mem0/__init__.py:341` `return json.dumps({"result": "No relevant memories found."})`
- `sources/mem0/integrations/hermes-plugin-mem0/__init__.py:343` `return json.dumps({"results": items, "count": len(items)})`

`_tool_add`, body `sources/mem0/integrations/hermes-plugin-mem0/__init__.py:346`:

- `sources/mem0/integrations/hermes-plugin-mem0/__init__.py:350` `return json.dumps({"result": msg, "event_id": event_id})`

`_tool_update`, body `sources/mem0/integrations/hermes-plugin-mem0/__init__.py:362`:

- `sources/mem0/integrations/hermes-plugin-mem0/__init__.py:363` `return json.dumps(self._backend.update(args["memory_id"], args["text"]))`

`_tool_delete`, body `sources/mem0/integrations/hermes-plugin-mem0/__init__.py:366`:

- `sources/mem0/integrations/hermes-plugin-mem0/__init__.py:367` `return json.dumps(self._backend.delete(args["memory_id"]))`

`handle_tool_call`, body `sources/mem0/integrations/hermes-plugin-mem0/__init__.py:377`:

- `sources/mem0/integrations/hermes-plugin-mem0/__init__.py:379` `return json.dumps({"error": f"Mem0 backend not initialized: {err}.{self._oss_hint(' Check that {vs} is running and reachable.')}"})`
- `sources/mem0/integrations/hermes-plugin-mem0/__init__.py:381` `return json.dumps({"error": f"Mem0 temporarily unavailable (multiple consecutive failures). Will retry automatically.{self._oss_hint(' Check that your {vs} is running.')}"})`
- `sources/mem0/integrations/hermes-plugin-mem0/__init__.py:383` `return tool_error(f"Unknown tool: {tool_name}")`
- `sources/mem0/integrations/hermes-plugin-mem0/__init__.py:386` `return tool_error("Tool arguments must be an object")`
- `sources/mem0/integrations/hermes-plugin-mem0/__init__.py:388` `return tool_error(f"Missing or invalid required parameter: {missing}")`
- `sources/mem0/integrations/hermes-plugin-mem0/__init__.py:393` `return tool_error("top_k must be an integer")`
- `sources/mem0/integrations/hermes-plugin-mem0/__init__.py:397` `return tool_error(str(e))`
- `sources/mem0/integrations/hermes-plugin-mem0/__init__.py:401` `return tool_error(f"Memory not found: {args['memory_id']}")`
- `sources/mem0/integrations/hermes-plugin-mem0/__init__.py:404` `return tool_error(self._format_error(label, e))`
- `sources/mem0/integrations/hermes-plugin-mem0/__init__.py:406` `return result`

</details>

## 4. Data access boundaries

Credentials: active Hermes profile `.env` for MEM0_API_KEY/MEM0_HOST; `mem0.json` settings and mode-dependent OSS credential configuration (`sources/mem0/integrations/hermes-plugin-mem0/__init__.py:86`). Same-user direct access is `yes`, via the configured API or OSS store with the same credentials; tool ownership validation is not OS isolation (`sources/mem0/integrations/hermes-plugin-mem0/__init__.py:354`). OSS/SDK telemetry targets `https://us.i.posthog.com` (`sources/mem0/mem0/memory/telemetry.py:16`); see the exact enable predicate below. Accountless local mode is `yes` with OSS and local model/vector configuration; default Platform is not accountless. Telemetry is `on` by default: MEM0_TELEMETRY defaults to True and recognizes true/1/yes (`sources/mem0/mem0/memory/telemetry.py:14`; `sources/mem0/mem0/memory/telemetry.py:20`).

Boundary model: same OS user, terminal, mounted filesystem and allowed network. Direct reachability means another client can use the same file/endpoint and available credentials, not unauthenticated access or encryption bypass. Hermes setup persists declared secrets through `hermes-agent/hermes_cli/memory_setup.py:321` and `hermes-agent/hermes_cli/config.py:2697`. These are possible configured paths; no live credentials were read.

Host credential-file detail: `get_hermes_home()/.env` is resolved at `hermes-agent/hermes_cli/config.py:473`. New files are secured to 0600 at `hermes-agent/hermes_cli/config.py:506`; the writer preserves an existing mode at `hermes-agent/hermes_cli/config.py:2544` and `:2562`. Therefore 0600 is not an unconditional mode claim for every existing profile.

<details>
<summary>Paths, environment, permissions, egress, telemetry and process-launch evidence</summary>

- `sources/mem0/cli/python/src/mem0_cli/config.py:19` `CONFIG_DIR = Path.home() / ".mem0"`
- `sources/mem0/cli/python/src/mem0_cli/config.py:22` `DEFAULT_BASE_URL = "https://api.mem0.ai"`
- `sources/mem0/cli/python/src/mem0_cli/config.py:50` `class TelemetryConfig:`
- `sources/mem0/cli/python/src/mem0_cli/config.py:66` `telemetry: TelemetryConfig = field(default_factory=TelemetryConfig)`
- `sources/mem0/cli/python/src/mem0_cli/config.py:84` `os.chmod(CONFIG_DIR, stat.S_IRWXU)  # 0700`
- `sources/mem0/cli/python/src/mem0_cli/config.py:113` `telemetry = data.get("telemetry", {})`
- `sources/mem0/cli/python/src/mem0_cli/config.py:114` `config.telemetry.anonymous_id = telemetry.get("anonymous_id", "")`
- `sources/mem0/cli/python/src/mem0_cli/config.py:120` `env_key = os.environ.get("MEM0_API_KEY")`
- `sources/mem0/cli/python/src/mem0_cli/config.py:124` `env_base = os.environ.get("MEM0_BASE_URL")`
- `sources/mem0/cli/python/src/mem0_cli/config.py:128` `env_user_id = os.environ.get("MEM0_USER_ID")`
- `sources/mem0/cli/python/src/mem0_cli/config.py:132` `env_agent_id = os.environ.get("MEM0_AGENT_ID")`
- `sources/mem0/cli/python/src/mem0_cli/config.py:136` `env_app_id = os.environ.get("MEM0_APP_ID")`
- `sources/mem0/cli/python/src/mem0_cli/config.py:140` `env_run_id = os.environ.get("MEM0_RUN_ID")`
- `sources/mem0/cli/python/src/mem0_cli/config.py:169` `"telemetry": {`
- `sources/mem0/cli/python/src/mem0_cli/config.py:170` `"anonymous_id": config.telemetry.anonymous_id,`
- `sources/mem0/cli/python/src/mem0_cli/config.py:180` `os.chmod(CONFIG_FILE, stat.S_IRUSR | stat.S_IWUSR)  # 0600`

- `sources/mem0/cli/python/src/mem0_cli/telemetry.py:1` `"""CLI telemetry , anonymous usage tracking via PostHog.`
- `sources/mem0/cli/python/src/mem0_cli/telemetry.py:3` `Sends fire-and-forget events to PostHog by spawning a detached subprocess`
- `sources/mem0/cli/python/src/mem0_cli/telemetry.py:4` `(telemetry_sender.py). The parent CLI process exits immediately; the`
- `sources/mem0/cli/python/src/mem0_cli/telemetry.py:7` `Disable with: MEM0_TELEMETRY=false`
- `sources/mem0/cli/python/src/mem0_cli/telemetry.py:22` `POSTHOG_API_KEY = "phc_[key cut]"`
- `sources/mem0/cli/python/src/mem0_cli/telemetry.py:23` `POSTHOG_HOST = "https://us.i.posthog.com/i/v0/e/"`
- `sources/mem0/cli/python/src/mem0_cli/telemetry.py:26` `def _is_telemetry_enabled() -> bool:`
- `sources/mem0/cli/python/src/mem0_cli/telemetry.py:27` `val = os.environ.get("MEM0_TELEMETRY", "true").lower()`
- `sources/mem0/cli/python/src/mem0_cli/telemetry.py:34` `Stored in ~/.mem0/config.json under 'telemetry.anonymous_id' so that`
- `sources/mem0/cli/python/src/mem0_cli/telemetry.py:35` `repeat runs on the same machine share one PostHog identity instead of`
- `sources/mem0/cli/python/src/mem0_cli/telemetry.py:41` `if config.telemetry.anonymous_id:`
- `sources/mem0/cli/python/src/mem0_cli/telemetry.py:42` `return config.telemetry.anonymous_id`
- `sources/mem0/cli/python/src/mem0_cli/telemetry.py:45` `config.telemetry.anonymous_id = new_id`
- `sources/mem0/cli/python/src/mem0_cli/telemetry.py:78` `"""Fire a PostHog event via a detached subprocess (non-blocking).`
- `sources/mem0/cli/python/src/mem0_cli/telemetry.py:81` `validation), it is used directly as the PostHog distinct ID and the`
- `sources/mem0/cli/python/src/mem0_cli/telemetry.py:84` `if not _is_telemetry_enabled():`
- `sources/mem0/cli/python/src/mem0_cli/telemetry.py:102` `and config.telemetry.anonymous_id`
- `sources/mem0/cli/python/src/mem0_cli/telemetry.py:104` `anon_id_to_alias = config.telemetry.anonymous_id`
- `sources/mem0/cli/python/src/mem0_cli/telemetry.py:105` `config.telemetry.anonymous_id = ""`
- `sources/mem0/cli/python/src/mem0_cli/telemetry.py:113` `"api_key": POSTHOG_API_KEY,`
- `sources/mem0/cli/python/src/mem0_cli/telemetry.py:125` `"$lib": "posthog-python",`
- `sources/mem0/cli/python/src/mem0_cli/telemetry.py:132` `"posthog_host": POSTHOG_HOST,`
- `sources/mem0/cli/python/src/mem0_cli/telemetry.py:135` `"mem0_base_url": config.platform.base_url or "https://api.mem0.ai",`
- `sources/mem0/cli/python/src/mem0_cli/telemetry.py:140` `child = subprocess.Popen(`
- `sources/mem0/cli/python/src/mem0_cli/telemetry.py:141` `[sys.executable, "-m", "mem0_cli.telemetry_sender"],`
- `sources/mem0/cli/python/src/mem0_cli/telemetry.py:142` `stdin=subprocess.PIPE,`
- `sources/mem0/cli/python/src/mem0_cli/telemetry.py:143` `stdout=subprocess.DEVNULL,`
- `sources/mem0/cli/python/src/mem0_cli/telemetry.py:144` `stderr=subprocess.DEVNULL,`

- `sources/mem0/cli/python/src/mem0_cli/telemetry_sender.py:1` `"""Standalone telemetry sender , runs as a detached subprocess.`
- `sources/mem0/cli/python/src/mem0_cli/telemetry_sender.py:3` `Usage: python -m mem0_cli.telemetry_sender   (JSON context is read from stdin;`
- `sources/mem0/cli/python/src/mem0_cli/telemetry_sender.py:6` `This module is spawned by telemetry.capture_event() and runs independently`
- `sources/mem0/cli/python/src/mem0_cli/telemetry_sender.py:11` `3. Sends the PostHog event`
- `sources/mem0/cli/python/src/mem0_cli/telemetry_sender.py:25` `"""Load telemetry context from stdin, falling back to argv for compatibility."""`
- `sources/mem0/cli/python/src/mem0_cli/telemetry_sender.py:49` `_send_posthog_event(ctx["posthog_host"], payload)`
- `sources/mem0/cli/python/src/mem0_cli/telemetry_sender.py:53` `"""Send a PostHog $identify event aliasing anon_id → payload['distinct_id']."""`
- `sources/mem0/cli/python/src/mem0_cli/telemetry_sender.py:60` `"$lib": payload.get("properties", {}).get("$lib", "posthog-python"),`
- `sources/mem0/cli/python/src/mem0_cli/telemetry_sender.py:63` `_send_posthog_event(ctx["posthog_host"], identify_payload)`
- `sources/mem0/cli/python/src/mem0_cli/telemetry_sender.py:101` `def _send_posthog_event(posthog_host: str, payload: dict) -> None:`
- `sources/mem0/cli/python/src/mem0_cli/telemetry_sender.py:102` `"""POST the event to PostHog."""`
- `sources/mem0/cli/python/src/mem0_cli/telemetry_sender.py:106` `posthog_host,`

- `sources/mem0/integrations/hermes-plugin-mem0/__init__.py:86` `config = {"mode": get_secret("MEM0_MODE", "") or "platform", "host": get_secret("MEM0_HOST", "") or "",`
- `sources/mem0/integrations/hermes-plugin-mem0/__init__.py:87` `"agent_id": get_secret("MEM0_AGENT_ID", "") or "hermes", "oss": {}}`
- `sources/mem0/integrations/hermes-plugin-mem0/__init__.py:88` `if user_id := get_secret("MEM0_USER_ID", ""):  # only when explicitly configured, so initialize() can fall back to the gateway-native id`
- `sources/mem0/integrations/hermes-plugin-mem0/__init__.py:100` `config["api_key"] = get_secret("MEM0_API_KEY", "")`
- `sources/mem0/integrations/hermes-plugin-mem0/__init__.py:153` `atomic_json_write(config_path, {**read_json_or_empty(config_path), **values}, mode=0o600)`
- `sources/mem0/integrations/hermes-plugin-mem0/__init__.py:159` `{"key": "api_key", "description": "Mem0 Platform API key", "secret": True, "required": api_key_required, "env_var": "MEM0_API_KEY", "url": "https://app.mem0.ai"},`

- `sources/mem0/integrations/hermes-plugin-mem0/_backend.py:168` `provider_config["api_key"] = provider_config.get("api_key") or get_secret("OPENAI_API_KEY", "")`
- `sources/mem0/integrations/hermes-plugin-mem0/_backend.py:172` `provider_config.get("openai_base_url") or get_secret("OPENAI_API_BASE", "")`
- `sources/mem0/integrations/hermes-plugin-mem0/_backend.py:173` `or get_secret("OPENAI_BASE_URL", "") or "https://api.openai.com/v1"`
- `sources/mem0/integrations/hermes-plugin-mem0/_backend.py:326` `telemetry = getattr(self._memory, "telemetry", None)`
- `sources/mem0/integrations/hermes-plugin-mem0/_backend.py:327` `if telemetry and hasattr(telemetry, "posthog"):`
- `sources/mem0/integrations/hermes-plugin-mem0/_backend.py:329` `telemetry.posthog.shutdown()`
- `sources/mem0/integrations/hermes-plugin-mem0/_backend.py:331` `telemetry_vs = getattr(self._memory, "_telemetry_vector_store", None)`
- `sources/mem0/integrations/hermes-plugin-mem0/_backend.py:332` `resources = (self._memory, vs, getattr(vs, "client", None), getattr(telemetry_vs, "client", None))`

- `sources/mem0/integrations/hermes-plugin-mem0/_openai_llm.py:43` `api_key = self.config.api_key or get_secret("OPENAI_API_KEY", "")`
- `sources/mem0/integrations/hermes-plugin-mem0/_openai_llm.py:47` `self.client = OpenAI(api_key=api_key, base_url=self.config.openai_base_url or get_secret("OPENAI_API_BASE", "") or get_secret("OPENAI_BASE_URL", "") or "https://api.openai.com/v1", timeout=120.0)`

- `sources/mem0/integrations/hermes-plugin-mem0/_oss_providers.py:12` `"ollama": {"label": "Ollama (local)", "needs_key": False, "default_model": "llama3.1:8b", "default_url": "http://localhost:11434", "base_url_key": "ollama_base_url", "pip_dep": "ollama"},`
- `sources/mem0/integrations/hermes-plugin-mem0/_oss_providers.py:17` `"ollama": {"label": "Ollama (local)", "needs_key": False, "default_model": "nomic-embed-text", "default_url": "http://localhost:11434", "base_url_key": "ollama_base_url", "dims": 768, "pip_dep": "ollama"},`
- `sources/mem0/integrations/hermes-plugin-mem0/_oss_providers.py:26` `"default_config": {"host": "localhost", "port": 5432, "user": os.getenv("USER", "postgres"), "dbname": "postgres"},`

- `sources/mem0/integrations/hermes-plugin-mem0/_setup.py:33` `_OLLAMA_URL = "http://localhost:11434"`
- `sources/mem0/integrations/hermes-plugin-mem0/_setup.py:78` `existing = os.environ.get(env_var, "")`
- `sources/mem0/integrations/hermes-plugin-mem0/_setup.py:91` `existing = os.environ.get("MEM0_API_KEY", "")`
- `sources/mem0/integrations/hermes-plugin-mem0/_setup.py:179` `atomic_write_text(env_path, "\n".join(new_lines) + "\n", mode=0o600)`
- `sources/mem0/integrations/hermes-plugin-mem0/_setup.py:209` `env_writes = _api_key_writes(flags, "Mem0 Platform API key", url="https://app.mem0.ai")`
- `sources/mem0/integrations/hermes-plugin-mem0/_setup.py:223` `if os.environ.get("MEM0_HOST", "").strip():`
- `sources/mem0/integrations/hermes-plugin-mem0/_setup.py:224` `print(f"\n  ⚠ MEM0_HOST is set in your environment ({os.environ['MEM0_HOST']}). It overrides platform mode , remove it from ~/.hermes/.env (or unset it) or Hermes will keep routing to the self-hosted server.")`
- `sources/mem0/integrations/hermes-plugin-mem0/_setup.py:245` `host = flags.get("host") or _prompt("Mem0 server URL (e.g. http://localhost:8888)", default=provider_config.get("host") or None)`
- `sources/mem0/integrations/hermes-plugin-mem0/_setup.py:307` `_finish_oss(hermes_home, config, oss_config, env_writes, flags.get("user_id") or os.getenv("USER", "hermes-user"), "hermes")`
- `sources/mem0/integrations/hermes-plugin-mem0/_setup.py:310` `def _docker(*args: str, timeout: int, **kwargs) -> subprocess.CompletedProcess:`
- `sources/mem0/integrations/hermes-plugin-mem0/_setup.py:311` `return subprocess.run(["docker", *args], capture_output=True, timeout=timeout, stdin=subprocess.DEVNULL, **kwargs)`
- `sources/mem0/integrations/hermes-plugin-mem0/_setup.py:351` `except subprocess.CalledProcessError as e:`
- `sources/mem0/integrations/hermes-plugin-mem0/_setup.py:363` `print("  Ollama not found. Install it:\n    curl -fsSL https://ollama.com/install.sh | sh\n  Or on macOS: brew install ollama")`
- `sources/mem0/integrations/hermes-plugin-mem0/_setup.py:367` `subprocess.Popen([ollama_bin, "serve"], stdin=subprocess.DEVNULL, stdout=subprocess.DEVNULL, stderr=subprocess.DEVNULL)`
- `sources/mem0/integrations/hermes-plugin-mem0/_setup.py:386` `subprocess.run([ollama_bin or "ollama", "pull", model], timeout=600, stdin=subprocess.DEVNULL)`
- `sources/mem0/integrations/hermes-plugin-mem0/_setup.py:454` `pg = {k: _input(f"PostgreSQL {label}", d) for k, label, d in (("user", "user", os.getenv("USER", "postgres")), ("host", "host", "localhost"), ("port", "port", "5432"), ("dbname", "database", "postgres"))}`
- `sources/mem0/integrations/hermes-plugin-mem0/_setup.py:457` `user_id = _input("User ID", os.getenv("USER", "hermes-user"))`

- `sources/mem0/mem0/configs/base.py:13` `mem0_dir = os.environ.get("MEM0_DIR") or os.path.join(home_dir, ".mem0")`
- `sources/mem0/mem0/configs/base.py:42` `history_db_path: str = Field(`

- `sources/mem0/mem0/configs/embeddings/base.py:37` `lmstudio_base_url: Optional[str] = "http://localhost:1234/v1",`
- `sources/mem0/mem0/configs/embeddings/base.py:59` `:param openai_base_url: Openai base URL to be use, defaults to "https://api.openai.com/v1"`
- `sources/mem0/mem0/configs/embeddings/base.py:73` `:param lmstudio_base_url: LM Studio base URL to be use, defaults to "http://localhost:1234/v1"`
- `sources/mem0/mem0/configs/embeddings/base.py:111` `self.aws_region = aws_region or os.environ.get("AWS_REGION") or "us-west-2"`

- `sources/mem0/mem0/configs/llms/aws_bedrock.py:63` `self.aws_region = aws_region or os.getenv("AWS_REGION", "us-west-2")`
- `sources/mem0/mem0/configs/llms/aws_bedrock.py:107` `config["aws_access_key_id"] = self.aws_access_key_id or os.getenv("AWS_ACCESS_KEY_ID")`
- `sources/mem0/mem0/configs/llms/aws_bedrock.py:110` `config["aws_secret_access_key"] = self.aws_secret_access_key or os.getenv("AWS_SECRET_ACCESS_KEY")`
- `sources/mem0/mem0/configs/llms/aws_bedrock.py:113` `config["aws_session_token"] = self.aws_session_token or os.getenv("AWS_SESSION_TOKEN")`
- `sources/mem0/mem0/configs/llms/aws_bedrock.py:116` `config["profile_name"] = self.aws_profile or os.getenv("AWS_PROFILE")`

- `sources/mem0/mem0/configs/llms/gemini.py:61` `vertexai = os.getenv("GOOGLE_GENAI_USE_VERTEXAI", "").lower() in ("true", "1", "yes")`
- `sources/mem0/mem0/configs/llms/gemini.py:63` `self.project = project or os.getenv("GOOGLE_CLOUD_PROJECT")`
- `sources/mem0/mem0/configs/llms/gemini.py:64` `self.location = location or os.getenv("GOOGLE_CLOUD_LOCATION", "us-central1")`

- `sources/mem0/mem0/configs/llms/lmstudio.py:58` `self.lmstudio_base_url = lmstudio_base_url or "http://localhost:1234/v1"`

- `sources/mem0/mem0/configs/llms/vllm.py:57` `self.vllm_base_url = vllm_base_url or os.getenv("VLLM_BASE_URL") or "http://localhost:8000/v1"`

- `sources/mem0/mem0/configs/prompts.py:369` `1. **Agent Action**: Opened URL "https://openai.com"`
- `sources/mem0/mem0/configs/prompts.py:378` `"Navigated to 'https://openai.com/blog/' with the blog listing fully rendered."`
- `sources/mem0/mem0/configs/prompts.py:389` `4. **Agent Action**: Visited URL "https://openai.com/blog/chatgpt-updates"`
- `sources/mem0/mem0/configs/prompts.py:395` `5. **Agent Action**: Extracted blog title and full article content from "https://openai.com/blog/chatgpt-updates"`

- `sources/mem0/mem0/configs/vector_stores/baidu.py:7` `endpoint: str = Field("http://localhost:8287", description="Endpoint URL for Baidu VectorDB")`

- `sources/mem0/mem0/configs/vector_stores/milvus.py:23` `url: str = Field("http://localhost:19530", description="Full URL for Milvus/Zilliz server")`

- `sources/mem0/mem0/configs/vector_stores/neptune.py:2` `Configuration for Amazon Neptune Analytics vector store.`
- `sources/mem0/mem0/configs/vector_stores/neptune.py:4` `This module provides configuration settings for integrating with Amazon Neptune Analytics`
- `sources/mem0/mem0/configs/vector_stores/neptune.py:17` `class NeptuneAnalyticsConfig(BaseModel):`
- `sources/mem0/mem0/configs/vector_stores/neptune.py:19` `Configuration class for Amazon Neptune Analytics vector store.`
- `sources/mem0/mem0/configs/vector_stores/neptune.py:21` `Amazon Neptune Analytics is a graph analytics engine that can be used as a vector store`
- `sources/mem0/mem0/configs/vector_stores/neptune.py:26` `endpoint (str): Neptune Analytics graph endpoint URL or Graph ID for the runtime.`

- `sources/mem0/mem0/configs/vector_stores/pinecone.py:27` `if not api_key and not client and "PINECONE_API_KEY" not in os.environ:`

- `sources/mem0/mem0/configs/vector_stores/turbopuffer.py:26` `if not api_key and "TURBOPUFFER_API_KEY" not in os.environ:`

- `sources/mem0/mem0/configs/vector_stores/upstash_vector.py:27` `url = values.get("url") or os.environ.get("UPSTASH_VECTOR_REST_URL")`
- `sources/mem0/mem0/configs/vector_stores/upstash_vector.py:28` `token = values.get("token") or os.environ.get("UPSTASH_VECTOR_REST_TOKEN")`

- `sources/mem0/mem0/memory/storage.py:12` `def __init__(self, db_path: str = ":memory:"):`
- `sources/mem0/mem0/memory/storage.py:13` `self.db_path = db_path`
- `sources/mem0/mem0/memory/storage.py:14` `self.connection = sqlite3.connect(self.db_path, check_same_thread=False)`

- `sources/mem0/mem0/memory/telemetry.py:9` `from posthog import Posthog`
- `sources/mem0/mem0/memory/telemetry.py:14` `MEM0_TELEMETRY = os.environ.get("MEM0_TELEMETRY", "True")`
- `sources/mem0/mem0/memory/telemetry.py:16` `HOST = "https://us.i.posthog.com"`
- `sources/mem0/mem0/memory/telemetry.py:19` `if isinstance(MEM0_TELEMETRY, str):`
- `sources/mem0/mem0/memory/telemetry.py:20` `MEM0_TELEMETRY = MEM0_TELEMETRY.lower() in ("true", "1", "yes")`
- `sources/mem0/mem0/memory/telemetry.py:22` `if not isinstance(MEM0_TELEMETRY, bool):`
- `sources/mem0/mem0/memory/telemetry.py:23` `raise ValueError("MEM0_TELEMETRY must be a boolean value.")`
- `sources/mem0/mem0/memory/telemetry.py:25` `logging.getLogger("posthog").setLevel(logging.CRITICAL + 1)`
- `sources/mem0/mem0/memory/telemetry.py:36` `"""Parse MEM0_TELEMETRY_SAMPLE_RATE env var. Never raises."""`
- `sources/mem0/mem0/memory/telemetry.py:40` `_logger.debug("MEM0_TELEMETRY_SAMPLE_RATE %r is not a number, defaulting to %s", raw, _DEFAULT_SAMPLE_RATE)`
- `sources/mem0/mem0/memory/telemetry.py:43` `_logger.debug("MEM0_TELEMETRY_SAMPLE_RATE %s out of [0.0, 1.0], defaulting to %s", value, _DEFAULT_SAMPLE_RATE)`
- `sources/mem0/mem0/memory/telemetry.py:48` `MEM0_TELEMETRY_SAMPLE_RATE = _parse_sample_rate(os.environ.get("MEM0_TELEMETRY_SAMPLE_RATE", str(_DEFAULT_SAMPLE_RATE)))`
- `sources/mem0/mem0/memory/telemetry.py:59` `"""PostHog before_send hook: drop sampled hot-path events, annotate survivors with sample_rate."""`
- `sources/mem0/mem0/memory/telemetry.py:67` `if not is_lifecycle and random.random() >= MEM0_TELEMETRY_SAMPLE_RATE:`
- `sources/mem0/mem0/memory/telemetry.py:72` `properties["sample_rate"] = 1.0 if is_lifecycle else MEM0_TELEMETRY_SAMPLE_RATE`
- `sources/mem0/mem0/memory/telemetry.py:76` `class AnonymousTelemetry:`
- `sources/mem0/mem0/memory/telemetry.py:78` `if not MEM0_TELEMETRY:`
- `sources/mem0/mem0/memory/telemetry.py:79` `self.posthog = None`
- `sources/mem0/mem0/memory/telemetry.py:83` `self.posthog = Posthog(`
- `sources/mem0/mem0/memory/telemetry.py:92` `if self.posthog is None:`
- `sources/mem0/mem0/memory/telemetry.py:98` `_logger.debug("Skipping telemetry event %r: no distinct_id available", event_name)`
- `sources/mem0/mem0/memory/telemetry.py:118` `self.posthog.capture(event_name, **capture_kwargs)`
- `sources/mem0/mem0/memory/telemetry.py:120` `_logger.debug("Failed to capture telemetry event %r: %s", event_name, e)`
- `sources/mem0/mem0/memory/telemetry.py:123` `"""Fire $identify with $anon_distinct_id so PostHog merges anon_id into email."""`
- `sources/mem0/mem0/memory/telemetry.py:124` `if self.posthog is None:`
- `sources/mem0/mem0/memory/telemetry.py:129` `self.posthog.capture(`
- `sources/mem0/mem0/memory/telemetry.py:140` `if self.posthog is not None:`
- `sources/mem0/mem0/memory/telemetry.py:141` `self.posthog.shutdown()`
- `sources/mem0/mem0/memory/telemetry.py:142` `self.posthog = None`
- `sources/mem0/mem0/memory/telemetry.py:150` `_oss_telemetry_instance = None`
- `sources/mem0/mem0/memory/telemetry.py:151` `_oss_telemetry_lock = threading.Lock()`
- `sources/mem0/mem0/memory/telemetry.py:152` `_oss_telemetry_shutting_down = False`
- `sources/mem0/mem0/memory/telemetry.py:155` `def _get_oss_telemetry():`
- `sources/mem0/mem0/memory/telemetry.py:156` `"""Return the process-wide AnonymousTelemetry singleton, creating it on first call.`
- `sources/mem0/mem0/memory/telemetry.py:158` `Returns None after _shutdown_oss_telemetry() has run (interpreter exit).`
- `sources/mem0/mem0/memory/telemetry.py:160` `global _oss_telemetry_instance`
- `sources/mem0/mem0/memory/telemetry.py:161` `if _oss_telemetry_shutting_down:`
- `sources/mem0/mem0/memory/telemetry.py:163` `if _oss_telemetry_instance is not None:`
- `sources/mem0/mem0/memory/telemetry.py:164` `return _oss_telemetry_instance`
- `sources/mem0/mem0/memory/telemetry.py:166` `with _oss_telemetry_lock:`
- `sources/mem0/mem0/memory/telemetry.py:167` `if _oss_telemetry_shutting_down:`
- `sources/mem0/mem0/memory/telemetry.py:170` `if _oss_telemetry_instance is not None:`
- `sources/mem0/mem0/memory/telemetry.py:171` `return _oss_telemetry_instance`
- `sources/mem0/mem0/memory/telemetry.py:172` `_oss_telemetry_instance = AnonymousTelemetry(before_send=_sampling_before_send)`
- `sources/mem0/mem0/memory/telemetry.py:173` `atexit.register(_shutdown_oss_telemetry)`
- `sources/mem0/mem0/memory/telemetry.py:174` `return _oss_telemetry_instance`
- `sources/mem0/mem0/memory/telemetry.py:177` `def _shutdown_oss_telemetry():`
- `sources/mem0/mem0/memory/telemetry.py:178` `global _oss_telemetry_instance, _oss_telemetry_shutting_down`
- `sources/mem0/mem0/memory/telemetry.py:179` `with _oss_telemetry_lock:`
- `sources/mem0/mem0/memory/telemetry.py:180` `_oss_telemetry_shutting_down = True`
- `sources/mem0/mem0/memory/telemetry.py:181` `if _oss_telemetry_instance is not None:`
- `sources/mem0/mem0/memory/telemetry.py:182` `_oss_telemetry_instance.close()`
- `sources/mem0/mem0/memory/telemetry.py:183` `_oss_telemetry_instance = None`
- `sources/mem0/mem0/memory/telemetry.py:188` `client_telemetry = AnonymousTelemetry()`
- `sources/mem0/mem0/memory/telemetry.py:189` `atexit.register(client_telemetry.close)`
- `sources/mem0/mem0/memory/telemetry.py:193` `"""Capture telemetry event for OSS Memory instances.`
- `sources/mem0/mem0/memory/telemetry.py:195` `This function is designed to never raise exceptions - telemetry failures`
- `sources/mem0/mem0/memory/telemetry.py:198` `if not MEM0_TELEMETRY:`
- `sources/mem0/mem0/memory/telemetry.py:202` `oss_telemetry = _get_oss_telemetry()`
- `sources/mem0/mem0/memory/telemetry.py:203` `if oss_telemetry is None:`
- `sources/mem0/mem0/memory/telemetry.py:218` `oss_telemetry.capture_event(event_name, event_data)`
- `sources/mem0/mem0/memory/telemetry.py:220` `_logger.debug("Failed to capture OSS telemetry event %r: %s", event_name, e)`
- `sources/mem0/mem0/memory/telemetry.py:224` `"""Capture telemetry event for hosted MemoryClient instances.`
- `sources/mem0/mem0/memory/telemetry.py:226` `This function is designed to never raise exceptions - telemetry failures`
- `sources/mem0/mem0/memory/telemetry.py:229` `if not MEM0_TELEMETRY:`
- `sources/mem0/mem0/memory/telemetry.py:239` `client_telemetry.capture_event(event_name, event_data, instance.user_email)`
- `sources/mem0/mem0/memory/telemetry.py:241` `_logger.debug("Failed to capture client telemetry event %r: %s", event_name, e)`

- `sources/mem0/mem0/vector_stores/configs.py:22` `"neptune": "NeptuneAnalyticsConfig",`

- `sources/mem0/mem0-ts/src/client/config.ts:37` `const dir = process.env.MEM0_DIR || pathMod.join(osMod.homedir(), ".mem0");`
- `sources/mem0/mem0-ts/src/client/config.ts:103` `const telemetry =`
- `sources/mem0/mem0-ts/src/client/config.ts:104` `config.telemetry && typeof config.telemetry === "object"`
- `sources/mem0/mem0-ts/src/client/config.ts:105` `? config.telemetry`
- `sources/mem0/mem0-ts/src/client/config.ts:110` `typeof telemetry.anonymous_id === "string"`
- `sources/mem0/mem0-ts/src/client/config.ts:111` `? telemetry.anonymous_id`
- `sources/mem0/mem0-ts/src/client/config.ts:113` `aliasedPairs: Array.isArray(telemetry.aliased_pairs)`
- `sources/mem0/mem0-ts/src/client/config.ts:114` `? telemetry.aliased_pairs.filter(`
- `sources/mem0/mem0-ts/src/client/config.ts:130` `const telemetry =`
- `sources/mem0/mem0-ts/src/client/config.ts:131` `config.telemetry && typeof config.telemetry === "object"`
- `sources/mem0/mem0-ts/src/client/config.ts:132` `? config.telemetry`
- `sources/mem0/mem0-ts/src/client/config.ts:134` `const aliasedPairs = Array.isArray(telemetry.aliased_pairs)`
- `sources/mem0/mem0-ts/src/client/config.ts:135` `? telemetry.aliased_pairs`
- `sources/mem0/mem0-ts/src/client/config.ts:148` `const telemetry =`
- `sources/mem0/mem0-ts/src/client/config.ts:149` `config.telemetry && typeof config.telemetry === "object"`
- `sources/mem0/mem0-ts/src/client/config.ts:150` `? config.telemetry`
- `sources/mem0/mem0-ts/src/client/config.ts:152` `const aliasedPairs = Array.isArray(telemetry.aliased_pairs)`
- `sources/mem0/mem0-ts/src/client/config.ts:153` `? telemetry.aliased_pairs`
- `sources/mem0/mem0-ts/src/client/config.ts:159` `telemetry.aliased_pairs = aliasedPairs;`
- `sources/mem0/mem0-ts/src/client/config.ts:160` `config.telemetry = telemetry;`

- `sources/mem0/mem0-ts/src/client/telemetry.ts:2` `import type { TelemetryClient, TelemetryOptions } from "./telemetry.types";`
- `sources/mem0/mem0-ts/src/client/telemetry.ts:11` `let MEM0_TELEMETRY = true;`
- `sources/mem0/mem0-ts/src/client/telemetry.ts:13` `MEM0_TELEMETRY = process?.env?.MEM0_TELEMETRY === "false" ? false : true;`
- `sources/mem0/mem0-ts/src/client/telemetry.ts:15` `const POSTHOG_API_KEY = "phc_[key cut]";`
- `sources/mem0/mem0-ts/src/client/telemetry.ts:16` `const POSTHOG_HOST = "https://us.i.posthog.com/i/v0/e/";`
- `sources/mem0/mem0-ts/src/client/telemetry.ts:26` `class UnifiedTelemetry implements TelemetryClient {`
- `sources/mem0/mem0-ts/src/client/telemetry.ts:40` `if (!MEM0_TELEMETRY) return false;`
- `sources/mem0/mem0-ts/src/client/telemetry.ts:47` `$lib: "posthog-node",`
- `sources/mem0/mem0-ts/src/client/telemetry.ts:67` `console.error("Telemetry event capture failed:", await response.text());`
- `sources/mem0/mem0-ts/src/client/telemetry.ts:72` `console.error("Telemetry event capture failed:", error);`
- `sources/mem0/mem0-ts/src/client/telemetry.ts:78` `if (!MEM0_TELEMETRY) return false;`
- `sources/mem0/mem0-ts/src/client/telemetry.ts:88` `$lib: "posthog-node",`
- `sources/mem0/mem0-ts/src/client/telemetry.ts:103` `"Telemetry identify capture failed:",`
- `sources/mem0/mem0-ts/src/client/telemetry.ts:110` `console.error("Telemetry identify capture failed:", error);`
- `sources/mem0/mem0-ts/src/client/telemetry.ts:120` `function isTelemetryEnabled(): boolean {`
- `sources/mem0/mem0-ts/src/client/telemetry.ts:121` `return MEM0_TELEMETRY;`
- `sources/mem0/mem0-ts/src/client/telemetry.ts:124` `const telemetry = new UnifiedTelemetry(POSTHOG_API_KEY, POSTHOG_HOST);`
- `sources/mem0/mem0-ts/src/client/telemetry.ts:131` `if (!instance.telemetryId) {`
- `sources/mem0/mem0-ts/src/client/telemetry.ts:132` `console.warn("No telemetry ID found for instance");`
- `sources/mem0/mem0-ts/src/client/telemetry.ts:146` `await telemetry.captureEvent(`
- `sources/mem0/mem0-ts/src/client/telemetry.ts:147` `instance.telemetryId,`
- `sources/mem0/mem0-ts/src/client/telemetry.ts:153` `export { telemetry, captureClientEvent, generateHash, isTelemetryEnabled };`

- `sources/mem0/mem0-ts/src/client/telemetry.types.ts:1` `export interface TelemetryClient {`
- `sources/mem0/mem0-ts/src/client/telemetry.types.ts:10` `export interface TelemetryInstance {`
- `sources/mem0/mem0-ts/src/client/telemetry.types.ts:11` `telemetryId: string;`
- `sources/mem0/mem0-ts/src/client/telemetry.types.ts:19` `export interface TelemetryEventData {`
- `sources/mem0/mem0-ts/src/client/telemetry.types.ts:29` `export interface TelemetryOptions {`

- `sources/mem0/mem0-ts/src/oss/src/config/defaults.ts:9` `apiKey: process.env.OPENAI_API_KEY || "",`
- `sources/mem0/mem0-ts/src/oss/src/config/defaults.ts:23` `baseURL: "https://api.openai.com/v1",`
- `sources/mem0/mem0-ts/src/oss/src/config/defaults.ts:24` `apiKey: process.env.OPENAI_API_KEY || "",`

- `sources/mem0/mem0-ts/src/oss/src/utils/telemetry.ts:2` `TelemetryClient,`
- `sources/mem0/mem0-ts/src/oss/src/utils/telemetry.ts:3` `TelemetryInstance,`
- `sources/mem0/mem0-ts/src/oss/src/utils/telemetry.ts:4` `TelemetryEventData,`
- `sources/mem0/mem0-ts/src/oss/src/utils/telemetry.ts:5` `} from "./telemetry.types";`
- `sources/mem0/mem0-ts/src/oss/src/utils/telemetry.ts:14` `let MEM0_TELEMETRY = true;`
- `sources/mem0/mem0-ts/src/oss/src/utils/telemetry.ts:16` `MEM0_TELEMETRY =`
- `sources/mem0/mem0-ts/src/oss/src/utils/telemetry.ts:17` `process?.env?.MEM0_TELEMETRY?.toLowerCase() === "false" ? false : true;`
- `sources/mem0/mem0-ts/src/oss/src/utils/telemetry.ts:19` `const POSTHOG_API_KEY = "phc_[key cut]";`
- `sources/mem0/mem0-ts/src/oss/src/utils/telemetry.ts:20` `const POSTHOG_HOST = "https://us.i.posthog.com/i/v0/e/";`
- `sources/mem0/mem0-ts/src/oss/src/utils/telemetry.ts:26` `const MEM0_TELEMETRY_SAMPLE_RATE: number = ((): number => {`
- `sources/mem0/mem0-ts/src/oss/src/utils/telemetry.ts:28` `const raw = process?.env?.MEM0_TELEMETRY_SAMPLE_RATE;`
- `sources/mem0/mem0-ts/src/oss/src/utils/telemetry.ts:46` `class UnifiedTelemetry implements TelemetryClient {`
- `sources/mem0/mem0-ts/src/oss/src/utils/telemetry.ts:56` `if (!MEM0_TELEMETRY) return;`
- `sources/mem0/mem0-ts/src/oss/src/utils/telemetry.ts:66` `$lib: "posthog-node",`
- `sources/mem0/mem0-ts/src/oss/src/utils/telemetry.ts:86` `console.error("Telemetry event capture failed:", await response.text());`
- `sources/mem0/mem0-ts/src/oss/src/utils/telemetry.ts:89` `console.error("Telemetry event capture failed:", error);`
- `sources/mem0/mem0-ts/src/oss/src/utils/telemetry.ts:98` `const telemetry = new UnifiedTelemetry(POSTHOG_API_KEY, POSTHOG_HOST);`
- `sources/mem0/mem0-ts/src/oss/src/utils/telemetry.ts:100` `function isTelemetryEnabled(): boolean {`
- `sources/mem0/mem0-ts/src/oss/src/utils/telemetry.ts:101` `return MEM0_TELEMETRY;`
- `sources/mem0/mem0-ts/src/oss/src/utils/telemetry.ts:106` `instance: TelemetryInstance,`
- `sources/mem0/mem0-ts/src/oss/src/utils/telemetry.ts:109` `if (!instance.telemetryId) {`
- `sources/mem0/mem0-ts/src/oss/src/utils/telemetry.ts:110` `console.warn("No telemetry ID found for instance");`
- `sources/mem0/mem0-ts/src/oss/src/utils/telemetry.ts:116` `if (!alwaysSend && Math.random() >= MEM0_TELEMETRY_SAMPLE_RATE) {`
- `sources/mem0/mem0-ts/src/oss/src/utils/telemetry.ts:120` `const eventData: TelemetryEventData = {`
- `sources/mem0/mem0-ts/src/oss/src/utils/telemetry.ts:129` `sample_rate: alwaysSend ? 1.0 : MEM0_TELEMETRY_SAMPLE_RATE,`
- `sources/mem0/mem0-ts/src/oss/src/utils/telemetry.ts:132` `await telemetry.captureEvent(`
- `sources/mem0/mem0-ts/src/oss/src/utils/telemetry.ts:133` `instance.telemetryId,`
- `sources/mem0/mem0-ts/src/oss/src/utils/telemetry.ts:140` `instance: TelemetryInstance,`
- `sources/mem0/mem0-ts/src/oss/src/utils/telemetry.ts:143` `if (!instance.telemetryId) return;`
- `sources/mem0/mem0-ts/src/oss/src/utils/telemetry.ts:145` `const eventData: TelemetryEventData = {`
- `sources/mem0/mem0-ts/src/oss/src/utils/telemetry.ts:156` `await telemetry.captureEvent(`
- `sources/mem0/mem0-ts/src/oss/src/utils/telemetry.ts:157` `instance.telemetryId,`
- `sources/mem0/mem0-ts/src/oss/src/utils/telemetry.ts:164` `POSTHOG_API_KEY,`
- `sources/mem0/mem0-ts/src/oss/src/utils/telemetry.ts:165` `POSTHOG_HOST,`
- `sources/mem0/mem0-ts/src/oss/src/utils/telemetry.ts:167` `telemetry,`
- `sources/mem0/mem0-ts/src/oss/src/utils/telemetry.ts:170` `isTelemetryEnabled,`

- `sources/mem0/mem0-ts/src/oss/src/utils/telemetry.types.ts:1` `export interface TelemetryClient {`
- `sources/mem0/mem0-ts/src/oss/src/utils/telemetry.types.ts:10` `export interface TelemetryInstance {`
- `sources/mem0/mem0-ts/src/oss/src/utils/telemetry.types.ts:11` `telemetryId: string;`
- `sources/mem0/mem0-ts/src/oss/src/utils/telemetry.types.ts:19` `export interface TelemetryEventData {`
- `sources/mem0/mem0-ts/src/oss/src/utils/telemetry.types.ts:26` `/** Set by the sampling layer so PostHog dashboards can extrapolate via 1/sample_rate. */`
- `sources/mem0/mem0-ts/src/oss/src/utils/telemetry.types.ts:31` `export interface TelemetryOptions {`

- `sources/mem0/mem0-ts/tsconfig.json:2` `"$schema": "https://json.schemastore.org/tsconfig",`

- `sources/mem0/server/dashboard/src/app/(root)/dashboard/configuration/page.tsx:214` `href="https://docs.mem0.ai/open-source/setup#supported-providers"`
- `sources/mem0/server/dashboard/src/app/(root)/dashboard/configuration/page.tsx:234` `ctaUrl="https://app.mem0.ai/enterprise?utm_source=oss&utm_medium=dashboard-configuration-sso"`

- `sources/mem0/server/telemetry.py:1` `"""Anonymous telemetry. Sends at most two events per install:`
- `sources/mem0/server/telemetry.py:7` `Enabled by default (matching the mem0 OSS library). Opt out with 'MEM0_TELEMETRY=false'.`
- `sources/mem0/server/telemetry.py:8` `Fires to the same PostHog project the library uses. Shared properties: email domain,`
- `sources/mem0/server/telemetry.py:25` `HOST = "https://us.i.posthog.com"`
- `sources/mem0/server/telemetry.py:27` `ENABLED = os.environ.get("MEM0_TELEMETRY", "true").lower() not in {"0", "false", "no", "off"}`
- `sources/mem0/server/telemetry.py:28` `STATE_PATH = Path(os.environ.get("MEM0_TELEMETRY_STATE_PATH", "/app/history/telemetry.json"))`
- `sources/mem0/server/telemetry.py:49` `logging.exception("telemetry: failed to persist state")`
- `sources/mem0/server/telemetry.py:66` `from posthog import Posthog`
- `sources/mem0/server/telemetry.py:68` `logging.warning("telemetry: posthog package not installed; disabling")`
- `sources/mem0/server/telemetry.py:70` `_client = Posthog(project_api_key=PROJECT_API_KEY, host=HOST, disable_geoip=True)`
- `sources/mem0/server/telemetry.py:76` `logging.info("telemetry: anonymous telemetry is enabled. Set MEM0_TELEMETRY=false to disable.")`
- `sources/mem0/server/telemetry.py:105` `logging.exception("telemetry: failed to send %s event", event)`
- `sources/mem0/server/telemetry.py:120` `off-box, so it is intentionally NOT gated by MEM0_TELEMETRY.`

</details>

## 5. Reproducibility

`range`: `mem0ai>=2.0.10,<3` (`sources/mem0/integrations/hermes-plugin-mem0/plugin.yaml:5`). This is not an exact engine pin, and snapshot HEAD is not necessarily the installed version.

Retained lock/checksum files: `sources/mem0/mem0-ts/pnpm-lock.yaml:1`, `sources/mem0/integrations/deepseek-plugin/pnpm-lock.yaml:1`, `sources/mem0/integrations/vercel-ai-sdk/pnpm-lock.yaml:1`, `sources/mem0/integrations/openclaw/pnpm-lock.yaml:1`, `sources/mem0/integrations/pi-agent-plugin/pnpm-lock.yaml:1`, `sources/mem0/integrations/n8n-nodes-mem0/pnpm-lock.yaml:1`, `sources/mem0/integrations/zapier-mem0/pnpm-lock.yaml:1`, `sources/mem0/integrations/agent-plugin-core/typescript/pnpm-lock.yaml:1`, `sources/mem0/cli/node/pnpm-lock.yaml:1`, `sources/mem0/server/dashboard/pnpm-lock.yaml:1`. Upstream completeness is unknown: fetch.py:15 excludes .lock files, fetch.py:12 excludes build/vendor directories. Absence here is not absence upstream.

- `sources/mem0/integrations/hermes-plugin-mem0/plugin.yaml:2` `version: 1.3.0`
- `sources/mem0/integrations/hermes-plugin-mem0/plugin.yaml:4` `pip_dependencies:`
- `sources/mem0/integrations/hermes-plugin-mem0/plugin.yaml:5` `- mem0ai>=2.0.10,<3`

- `sources/mem0/integrations/hermes-plugin-mem0/pyproject.toml:3` `version = "1.3.0"`
- `sources/mem0/integrations/hermes-plugin-mem0/pyproject.toml:5` `requires-python = ">=3.11"`
- `sources/mem0/integrations/hermes-plugin-mem0/pyproject.toml:7` `# Installed into the Hermes venv by 'hermes plugins install' / 'enable' and re-applied after`
- `sources/mem0/integrations/hermes-plugin-mem0/pyproject.toml:9` `# and refuses a plugin whose requirements cannot resolve against them.`
- `sources/mem0/integrations/hermes-plugin-mem0/pyproject.toml:10` `dependencies = [`
- `sources/mem0/integrations/hermes-plugin-mem0/pyproject.toml:11` `"mem0ai>=2.0.10,<3",`
- `sources/mem0/integrations/hermes-plugin-mem0/pyproject.toml:12` `"httpx>=0.27,<1",`
- `sources/mem0/integrations/hermes-plugin-mem0/pyproject.toml:15` `[project.optional-dependencies]`
- `sources/mem0/integrations/hermes-plugin-mem0/pyproject.toml:16` `postgres = ['psycopg2-binary>=2.9,<3']`
- `sources/mem0/integrations/hermes-plugin-mem0/pyproject.toml:17` `qdrant = ['qdrant-client>=1.9,<2']`

## 6. Tests

Adapter tests: `integrations/hermes-plugin-mem0/tests/test_plugin.py`, 17 declarations. The adapter deletion path must not be credited from other clients' tests. Engine deletion tests exist, e.g. `sources/mem0/tests/test_main.py:268` and `sources/mem0/tests/test_memory.py:819`; these exercise deletion/history behavior, not universal erasure. Adapter transport delete is explicitly asserted in test_selfhosted_http_auth_and_tool_routes (`sources/mem0/integrations/hermes-plugin-mem0/tests/test_plugin.py:144`); this does not exercise the provider ownership handler.

Adapter-related declarations: **17**. Python AST test_ functions, fixtures excluded; literal TS/JS it/test, Go Test*, Rust test_* declarations. No parametrization expansion, collection, execution, or coverage measurement. Other rows are engine test candidates selected by mutation names; names alone do not verify forget coverage.

| file | declarations | adapter-related | forget-related declarations |
|---|---:|---|---|
| `sources/mem0/cli/node/tests/cli-integration.test.ts:1` | 17 | no | `shows delete help with --all, --entity, --project` `sources/mem0/cli/node/tests/cli-integration.test.ts:91`; `delete with no args errors` `sources/mem0/cli/node/tests/cli-integration.test.ts:101`; `shows entity delete help` `sources/mem0/cli/node/tests/cli-integration.test.ts:114` |
| `sources/mem0/cli/node/tests/commands.test.ts:1` | 44 | no | `deletes memory` `sources/mem0/cli/node/tests/commands.test.ts:446`; `forwards deleteLinked (MEM-5893)` `sources/mem0/cli/node/tests/commands.test.ts:452`; `deletes all with force` `sources/mem0/cli/node/tests/commands.test.ts:466`; `cmdDelete outputs JSON envelope` `sources/mem0/cli/node/tests/commands.test.ts:630` |
| `sources/mem0/cli/node/tests/platform-backend.test.ts:1` | 13 | no | `returns all results keyed by entity type for a multi-entity delete` `sources/mem0/cli/node/tests/platform-backend.test.ts:30`; `keys a single-entity delete by its type` `sources/mem0/cli/node/tests/platform-backend.test.ts:55`; `delete: delete_linked is a query param, not part of the JSON body` `sources/mem0/cli/node/tests/platform-backend.test.ts:199` |
| `sources/mem0/cli/python/tests/test_cli_integration.py:1` | 26 | no | `test_delete_help` `sources/mem0/cli/python/tests/test_cli_integration.py:146`; `test_entity_delete_help` `sources/mem0/cli/python/tests/test_cli_integration.py:160`; `test_delete_no_id_no_all_errors` `sources/mem0/cli/python/tests/test_cli_integration.py:209`; `test_delete_entity_via_delete_flag` `sources/mem0/cli/python/tests/test_cli_integration.py:272`; `test_entity_delete_has_scope_options` `sources/mem0/cli/python/tests/test_cli_integration.py:278` |
| `sources/mem0/cli/python/tests/test_commands.py:1` | 72 | no | `test_delete_single` `sources/mem0/cli/python/tests/test_commands.py:753`; `test_delete_dry_run` `sources/mem0/cli/python/tests/test_commands.py:764`; `test_delete_linked_reaches_backend` `sources/mem0/cli/python/tests/test_commands.py:776`; `test_delete_all_force` `sources/mem0/cli/python/tests/test_commands.py:789`; `test_delete_all_dry_run` `sources/mem0/cli/python/tests/test_commands.py:808`; `test_delete_all_project_wide` `sources/mem0/cli/python/tests/test_commands.py:829`; `test_delete_all_project_wide_async_response` `sources/mem0/cli/python/tests/test_commands.py:854`; `test_delete_entity_with_force` `sources/mem0/cli/python/tests/test_commands.py:1071`; `test_delete_entity_agent_id` `sources/mem0/cli/python/tests/test_commands.py:1093`; `test_delete_entity_no_id_exits` `sources/mem0/cli/python/tests/test_commands.py:1115`; `test_delete_entity_json_output` `sources/mem0/cli/python/tests/test_commands.py:1133`; `test_delete_entity_dry_run` `sources/mem0/cli/python/tests/test_commands.py:1152`; `test_delete_agent_mode_envelope` `sources/mem0/cli/python/tests/test_commands.py:1412` |
| `sources/mem0/cli/python/tests/test_platform_backend_options.py:1` | 8 | no | `test_delete_linked_is_a_query_param_not_json_body` `sources/mem0/cli/python/tests/test_platform_backend_options.py:106` |
| `sources/mem0/integrations/agent-plugin-core/tests/test_spool_delivery.py:1` | 18 | no | `test_a_live_final_attempt_is_not_deleted_by_another_sender` `sources/mem0/integrations/agent-plugin-core/tests/test_spool_delivery.py:107`; `test_a_torn_claim_is_quarantined_not_deleted` `sources/mem0/integrations/agent-plugin-core/tests/test_spool_delivery.py:372` |
| `sources/mem0/integrations/agent-plugin-core/typescript/tests/telemetry.test.ts:1` | 15 | no | `a failed delivery keeps the batch instead of deleting it` `sources/mem0/integrations/agent-plugin-core/typescript/tests/telemetry.test.ts:126` |
| `sources/mem0/integrations/claude-code-plugin/tests/test_memory_core.py:1` | 160 | no | `test_forget_skill_requires_confirmation` `sources/mem0/integrations/claude-code-plugin/tests/test_memory_core.py:2806`; `test_forget_refuses_an_unscoped_delete` `sources/mem0/integrations/claude-code-plugin/tests/test_memory_core.py:3946`; `test_forget_deletes_each_memory_by_id` `sources/mem0/integrations/claude-code-plugin/tests/test_memory_core.py:3959`; `test_forget_only_touches_shared_project_memory_when_asked` `sources/mem0/integrations/claude-code-plugin/tests/test_memory_core.py:3980`; `test_forget_includes_legacy_shared_memories_only_when_requested` `sources/mem0/integrations/claude-code-plugin/tests/test_memory_core.py:3998`; `test_forget_reports_partial_failures` `sources/mem0/integrations/claude-code-plugin/tests/test_memory_core.py:4027` |
| `sources/mem0/integrations/hermes-plugin-mem0/tests/smoke_hermes.py:1` | 0 | yes | none |
| `sources/mem0/integrations/hermes-plugin-mem0/tests/test_plugin.py:1` | 17 | yes | none |
| `sources/mem0/integrations/openclaw/tests/backend-platform.test.ts:1` | 26 | no | `delete() with memoryId sends DELETE to /v1/memories/{id}/` `sources/mem0/integrations/openclaw/tests/backend-platform.test.ts:249`; `delete() with all=true sends DELETE to /v1/memories/ with scope params` `sources/mem0/integrations/openclaw/tests/backend-platform.test.ts:264` |
| `sources/mem0/integrations/openclaw/tests/cli-commands.test.ts:1` | 81 | no | `deletes a single memory by ID` `sources/mem0/integrations/openclaw/tests/cli-commands.test.ts:949`; `bulk deletes with --all and --confirm` `sources/mem0/integrations/openclaw/tests/cli-commands.test.ts:959`; `requires --confirm for bulk delete in non-interactive mode` `sources/mem0/integrations/openclaw/tests/cli-commands.test.ts:971`; `handles delete errors gracefully` `sources/mem0/integrations/openclaw/tests/cli-commands.test.ts:1013` |
| `sources/mem0/integrations/openclaw/tests/providers.test.ts:1` | 17 | no | `calls provider.delete with memoryId` `sources/mem0/integrations/openclaw/tests/providers.test.ts:304`; `calls provider.deleteAll when opts.all is true` `sources/mem0/integrations/openclaw/tests/providers.test.ts:314` |
| `sources/mem0/integrations/openclaw/tests/telemetry.test.ts:1` | 10 | no | `forgets the account when the API key changes` `sources/mem0/integrations/openclaw/tests/telemetry.test.ts:101` |
| `sources/mem0/integrations/openclaw/tests/tools.test.ts:1` | 39 | no | `deletes by memoryId via provider.delete` `sources/mem0/integrations/openclaw/tests/tools.test.ts:501`; `searches and auto-deletes single high-confidence match by query` `sources/mem0/integrations/openclaw/tests/tools.test.ts:513`; `requires confirm:true for bulk delete (all)` `sources/mem0/integrations/openclaw/tests/tools.test.ts:591`; `performs bulk delete when all:true and confirm:true` `sources/mem0/integrations/openclaw/tests/tools.test.ts:601`; `blocks subagent sessions from deleting` `sources/mem0/integrations/openclaw/tests/tools.test.ts:639` |
| `sources/mem0/integrations/pi-agent-plugin/src/commands.test.ts:1` | 22 | no | `asks for confirmation before deleting a single match` `sources/mem0/integrations/pi-agent-plugin/src/commands.test.ts:99`; `does not delete when user cancels confirmation` `sources/mem0/integrations/pi-agent-plugin/src/commands.test.ts:129`; `uses select UI for multiple matches and deletes chosen memory` `sources/mem0/integrations/pi-agent-plugin/src/commands.test.ts:142`; `does not delete when user cancels select` `sources/mem0/integrations/pi-agent-plugin/src/commands.test.ts:172` |
| `sources/mem0/integrations/pi-agent-plugin/tests/tools.test.ts:1` | 6 | no | `delete calls mem0.delete with full memory_id` `sources/mem0/integrations/pi-agent-plugin/tests/tools.test.ts:52`; `delete passes memory_id directly to mem0.delete` `sources/mem0/integrations/pi-agent-plugin/tests/tools.test.ts:58` |
| `sources/mem0/integrations/zapier-mem0/test/mem0.test.ts:1` | 3 | no | `adds, searches, lists, and deletes a memory` `sources/mem0/integrations/zapier-mem0/test/mem0.test.ts:39` |
| `sources/mem0/mem0-ts/src/client/tests/integration/batch.test.ts:1` | 2 | no | `batch deletes memories that exist` `sources/mem0/mem0-ts/src/client/tests/integration/batch.test.ts:56` |
| `sources/mem0/mem0-ts/src/client/tests/integration/crud.test.ts:1` | 15 | no | `deleteAll for non-existent user does not throw` `sources/mem0/mem0-ts/src/client/tests/integration/crud.test.ts:221`; `deletes a single memory by ID` `sources/mem0/mem0-ts/src/client/tests/integration/crud.test.ts:235`; `getting deleted memory throws MemoryError` `sources/mem0/mem0-ts/src/client/tests/integration/crud.test.ts:244`; `deletes all memories for test user` `sources/mem0/mem0-ts/src/client/tests/integration/crud.test.ts:252`; `deletes the test user entity` `sources/mem0/mem0-ts/src/client/tests/integration/crud.test.ts:260` |
| `sources/mem0/mem0-ts/src/client/tests/integration/management.test.ts:1` | 15 | no | `deleteWebhook returns a response` `sources/mem0/mem0-ts/src/client/tests/integration/management.test.ts:193`; `deleteWebhook removes the webhook from the list` `sources/mem0/mem0-ts/src/client/tests/integration/management.test.ts:200` |
| `sources/mem0/mem0-ts/src/client/tests/memoryClient.batch.test.ts:1` | 6 | no | `sends DELETE to /v1/batch/` `sources/mem0/mem0-ts/src/client/tests/memoryClient.batch.test.ts:65` |
| `sources/mem0/mem0-ts/src/client/tests/memoryClient.crud.test.ts:1` | 25 | no | `sends DELETE to /v1/memories/:id/` `sources/mem0/mem0-ts/src/client/tests/memoryClient.crud.test.ts:280`; `serializes deleteLinked as delete_linked query param` `sources/mem0/mem0-ts/src/client/tests/memoryClient.crud.test.ts:297`; `sends DELETE to /v1/memories/ with user_id as query param` `sources/mem0/mem0-ts/src/client/tests/memoryClient.crud.test.ts:333` |
| `sources/mem0/mem0-ts/src/client/tests/memoryClient.users.test.ts:1` | 5 | no | `sends DELETE to /v1/entities/:type/:id/` `sources/mem0/mem0-ts/src/client/tests/memoryClient.users.test.ts:40` |
| `sources/mem0/mem0-ts/src/client/tests/memoryClient.webhooks.test.ts:1` | 14 | no | `sends DELETE to /api/v1/webhooks/:id/` `sources/mem0/mem0-ts/src/client/tests/memoryClient.webhooks.test.ts:177` |
| `sources/mem0/mem0-ts/src/oss/src/tests/better-sqlite3-migration.test.ts:1` | 24 | no | `delete removes the vector` `sources/mem0/mem0-ts/src/oss/src/tests/better-sqlite3-migration.test.ts:288`; `deleteCol drops and recreates table` `sources/mem0/mem0-ts/src/oss/src/tests/better-sqlite3-migration.test.ts:296` |
| `sources/mem0/mem0-ts/src/oss/src/tests/chroma.test.ts:1` | 38 | no | `deletes by id` `sources/mem0/mem0-ts/src/oss/src/tests/chroma.test.ts:285`; `deletes the collection and resets its cached handle` `sources/mem0/mem0-ts/src/oss/src/tests/chroma.test.ts:293` |
| `sources/mem0/mem0-ts/src/oss/src/tests/milvus.test.ts:1` | 25 | no | `deletes a record by id` `sources/mem0/mem0-ts/src/oss/src/tests/milvus.test.ts:360` |
| `sources/mem0/mem0-ts/src/oss/src/tests/mongodb.test.ts:1` | 16 | no | `should delete document correctly` `sources/mem0/mem0-ts/src/oss/src/tests/mongodb.test.ts:346`; `should delete collection correctly` `sources/mem0/mem0-ts/src/oss/src/tests/mongodb.test.ts:354` |
| `sources/mem0/mem0-ts/src/oss/src/tests/pinecone.test.ts:1` | 36 | no | `calls deleteOne with the vectorId` `sources/mem0/mem0-ts/src/oss/src/tests/pinecone.test.ts:414`; `calls deleteIndex and resets internal state so re-init creates fresh index` `sources/mem0/mem0-ts/src/oss/src/tests/pinecone.test.ts:422` |
| `sources/mem0/mem0-ts/src/oss/src/tests/sqlite-backward-compat.test.ts:1` | 21 | no | `insert, search, get, update, delete, list all work` `sources/mem0/mem0-ts/src/oss/src/tests/sqlite-backward-compat.test.ts:208` |
| `sources/mem0/mem0-ts/src/oss/src/tests/vector_stores/turbopuffer.test.ts:1` | 35 | no | `calls write with deletes array` `sources/mem0/mem0-ts/src/oss/src/tests/vector_stores/turbopuffer.test.ts:260`; `calls deleteAll on the namespace` `sources/mem0/mem0-ts/src/oss/src/tests/vector_stores/turbopuffer.test.ts:268` |
| `sources/mem0/mem0-ts/src/oss/tests/baidu.test.ts:1` | 30 | no | `updates and deletes by primary key` `sources/mem0/mem0-ts/src/oss/tests/baidu.test.ts:387` |
| `sources/mem0/mem0-ts/src/oss/tests/elasticsearch.unit.test.ts:1` | 23 | no | `deletes document by id` `sources/mem0/mem0-ts/src/oss/tests/elasticsearch.unit.test.ts:379`; `deletes the index` `sources/mem0/mem0-ts/src/oss/tests/elasticsearch.unit.test.ts:393` |
| `sources/mem0/mem0-ts/src/oss/tests/memory.crud.test.ts:1` | 46 | no | `get() returns null after deletion` `sources/mem0/mem0-ts/src/oss/tests/memory.crud.test.ts:621`; `deletes past the vector store default page limit` `sources/mem0/mem0-ts/src/oss/tests/memory.crud.test.ts:663` |
| `sources/mem0/mem0-ts/src/oss/tests/notices.decay-usage.test.ts:1` | 11 | no | `does not evaluate or write decay state before the 5th successful delete` `sources/mem0/mem0-ts/src/oss/tests/notices.decay-usage.test.ts:211`; `evaluates on the 5th successful delete and records delete_count` `sources/mem0/mem0-ts/src/oss/tests/notices.decay-usage.test.ts:230`; `does not count or emit when delete fails` `sources/mem0/mem0-ts/src/oss/tests/notices.decay-usage.test.ts:264`; `evaluates deleteAll after deleting at least one memory` `sources/mem0/mem0-ts/src/oss/tests/notices.decay-usage.test.ts:280`; `does not evaluate deleteAll when no memories are deleted` `sources/mem0/mem0-ts/src/oss/tests/notices.decay-usage.test.ts:305`; `does not evaluate when deleteAll fails` `sources/mem0/mem0-ts/src/oss/tests/notices.decay-usage.test.ts:318` |
| `sources/mem0/mem0-ts/src/oss/tests/storage.unit.test.ts:1` | 22 | no | `addHistory stores null new_value for DELETE` `sources/mem0/mem0-ts/src/oss/tests/storage.unit.test.ts:65` |
| `sources/mem0/mem0-ts/src/oss/tests/upstash_vector.unit.test.ts:1` | 4 | no | `fetches, updates, deletes, resets, and lists vectors in the namespace` `sources/mem0/mem0-ts/src/oss/tests/upstash_vector.unit.test.ts:82` |
| `sources/mem0/mem0-ts/src/oss/tests/vector-store.unit.test.ts:1` | 17 | no | `delete removes a vector` `sources/mem0/mem0-ts/src/oss/tests/vector-store.unit.test.ts:131`; `deleteCol clears all vectors` `sources/mem0/mem0-ts/src/oss/tests/vector-store.unit.test.ts:138` |
| `sources/mem0/mem0-ts/src/oss/tests/vector-stores-compat.test.ts:1` | 122 | no | `coalesces triggered-index syncs across sequential deletes` `sources/mem0/mem0-ts/src/oss/tests/vector-stores-compat.test.ts:2160`; `supports get, update, delete, and list` `sources/mem0/mem0-ts/src/oss/tests/vector-stores-compat.test.ts:2562`; `deletes the full Neptune collection with deleteCol()` `sources/mem0/mem0-ts/src/oss/tests/vector-stores-compat.test.ts:4247` |
| `sources/mem0/mem0-ts/src/oss/tests/vertex-ai-vector-search.test.ts:1` | 18 | no | `should delete a vector by id` `sources/mem0/mem0-ts/src/oss/tests/vertex-ai-vector-search.test.ts:150`; `should ignore NOT_FOUND (gRPC code 5) on delete` `sources/mem0/mem0-ts/src/oss/tests/vertex-ai-vector-search.test.ts:158`; `should rethrow non-NOT_FOUND errors on delete` `sources/mem0/mem0-ts/src/oss/tests/vertex-ai-vector-search.test.ts:165`; `should warn and no-op on deleteCol (unsupported)` `sources/mem0/mem0-ts/src/oss/tests/vertex-ai-vector-search.test.ts:193` |
| `sources/mem0/tests/memory/test_decay_usage_notice.py:1` | 10 | no | `test_sync_delete_decay_usage_runs_after_success` `sources/mem0/tests/memory/test_decay_usage_notice.py:25`; `test_sync_delete_below_threshold_uses_first_run_notice` `sources/mem0/tests/memory/test_decay_usage_notice.py:56`; `test_sync_delete_failure_does_not_trigger_decay_usage_notice` `sources/mem0/tests/memory/test_decay_usage_notice.py:72`; `test_sync_delete_all_decay_usage_runs_after_success` `sources/mem0/tests/memory/test_decay_usage_notice.py:91`; `test_sync_delete_all_stops_when_vector_store_repeats_batch` `sources/mem0/tests/memory/test_decay_usage_notice.py:120`; `test_sync_delete_all_zero_deletes_uses_first_run_notice` `sources/mem0/tests/memory/test_decay_usage_notice.py:135`; `test_async_delete_decay_usage_runs_after_success` `sources/mem0/tests/memory/test_decay_usage_notice.py:152`; `test_async_delete_failure_does_not_trigger_decay_usage_notice` `sources/mem0/tests/memory/test_decay_usage_notice.py:184`; `test_async_delete_all_decay_usage_runs_after_success` `sources/mem0/tests/memory/test_decay_usage_notice.py:204`; `test_async_delete_all_stops_when_vector_store_repeats_batch` `sources/mem0/tests/memory/test_decay_usage_notice.py:234` |
| `sources/mem0/tests/memory/test_notices.py:1` | 82 | no | `test_decay_usage_delete_detection_reaches_threshold` `sources/mem0/tests/memory/test_notices.py:814`; `test_decay_usage_delete_detection_does_not_write_before_threshold` `sources/mem0/tests/memory/test_notices.py:830`; `test_decay_usage_delete_detection_telemetry_disabled_does_not_write_state` `sources/mem0/tests/memory/test_notices.py:841`; `test_decay_usage_delete_all_detection_requires_deleted_memory` `sources/mem0/tests/memory/test_notices.py:850`; `test_decay_usage_delete_detection_stops_after_cap` `sources/mem0/tests/memory/test_notices.py:1045` |
| `sources/mem0/tests/test_client.py:1` | 40 | no | `test_delete_default_omits_delete_linked` `sources/mem0/tests/test_client.py:430`; `test_delete_linked_true_sets_param` `sources/mem0/tests/test_client.py:440`; `test_delete_linked_false_omits_param` `sources/mem0/tests/test_client.py:449` |
| `sources/mem0/tests/test_main.py:1` | 41 | no | `test_delete` `sources/mem0/tests/test_main.py:270`; `test_delete_all` `sources/mem0/tests/test_main.py:281`; `test_delete_all_paginates_beyond_vector_store_page_size` `sources/mem0/tests/test_main.py:299`; `test_delete_all_rejects_whitespace_only_user_id` `sources/mem0/tests/test_main.py:435`; `test_delete_all_rejects_internal_whitespace_user_id` `sources/mem0/tests/test_main.py:440`; `test_delete_all_trims_user_id_before_list` `sources/mem0/tests/test_main.py:445`; `test_delete_all_coerces_integer_user_id_before_list` `sources/mem0/tests/test_main.py:460` |
| `sources/mem0/tests/test_memory.py:1` | 62 | no | `test_delete_memory` `sources/mem0/tests/test_memory.py:65`; `test_delete_nonexistent_memory_raises_error` `sources/mem0/tests/test_memory.py:511`; `test_async_delete_nonexistent_memory_raises_error` `sources/mem0/tests/test_memory.py:542`; `test_delete_memory_history_has_timestamps` `sources/mem0/tests/test_memory.py:819`; `test_delete_memory_normalizes_non_utc_created_at` `sources/mem0/tests/test_memory.py:857`; `test_delete_memory_missing_created_at` `sources/mem0/tests/test_memory.py:888`; `test_async_delete_memory_history_has_timestamps` `sources/mem0/tests/test_memory.py:921`; `test_async_delete_all_continues_on_partial_failure` `sources/mem0/tests/test_memory.py:960`; `test_async_delete_all_bulk_clears_entity_store` `sources/mem0/tests/test_memory.py:1465` |
| `sources/mem0/tests/test_server_auth.py:1` | 47 | no | `test_delete_memory_without_key` `sources/mem0/tests/test_server_auth.py:104`; `test_delete_all_without_key` `sources/mem0/tests/test_server_auth.py:108`; `test_delete_memory_with_key` `sources/mem0/tests/test_server_auth.py:286`; `test_delete_all_with_key` `sources/mem0/tests/test_server_auth.py:290` |
| `sources/mem0/tests/test_server_params.py:1` | 68 | no | `test_delete_not_found_returns_404` `sources/mem0/tests/test_server_params.py:755` |
| `sources/mem0/tests/vector_stores/test_azure_mysql.py:1` | 19 | no | `test_delete` `sources/mem0/tests/vector_stores/test_azure_mysql.py:130`; `test_delete_col` `sources/mem0/tests/vector_stores/test_azure_mysql.py:185` |
| `sources/mem0/tests/vector_stores/test_baidu.py:1` | 16 | no | `test_delete` `sources/mem0/tests/vector_stores/test_baidu.py:176`; `test_delete_col_not_exists` `sources/mem0/tests/vector_stores/test_baidu.py:242` |
| `sources/mem0/tests/vector_stores/test_cassandra.py:1` | 17 | no | `test_delete` `sources/mem0/tests/vector_stores/test_cassandra.py:117`; `test_delete_col` `sources/mem0/tests/vector_stores/test_cassandra.py:182` |
| `sources/mem0/tests/vector_stores/test_chroma.py:1` | 31 | no | `test_delete_vector` `sources/mem0/tests/vector_stores/test_chroma.py:120` |
| `sources/mem0/tests/vector_stores/test_databricks.py:1` | 45 | no | `test_delete_vector` `sources/mem0/tests/vector_stores/test_databricks.py:258`; `test_delete_prevents_sql_injection` `sources/mem0/tests/vector_stores/test_databricks.py:894` |
| `sources/mem0/tests/vector_stores/test_elasticsearch.py:1` | 22 | no | `test_delete` `sources/mem0/tests/vector_stores/test_elasticsearch.py:332`; `test_delete_col` `sources/mem0/tests/vector_stores/test_elasticsearch.py:350` |
| `sources/mem0/tests/vector_stores/test_faiss.py:1` | 33 | no | `test_delete` `sources/mem0/tests/vector_stores/test_faiss.py:202`; `test_delete_col` `sources/mem0/tests/vector_stores/test_faiss.py:317`; `test_delete_col_removes_json_and_pkl` `sources/mem0/tests/vector_stores/test_faiss.py:675` |
| `sources/mem0/tests/vector_stores/test_milvus.py:1` | 28 | no | `test_delete` `sources/mem0/tests/vector_stores/test_milvus.py:182` |
| `sources/mem0/tests/vector_stores/test_mongodb.py:1` | 21 | no | `test_delete` `sources/mem0/tests/vector_stores/test_mongodb.py:195`; `test_delete_col` `sources/mem0/tests/vector_stores/test_mongodb.py:292` |
| `sources/mem0/tests/vector_stores/test_neptune_analytics.py:1` | 26 | no | `test_delete` `sources/mem0/tests/vector_stores/test_neptune_analytics.py:107`; `test_delete_col` `sources/mem0/tests/vector_stores/test_neptune_analytics.py:155` |
| `sources/mem0/tests/vector_stores/test_opensearch.py:1` | 42 | no | `test_delete` `sources/mem0/tests/vector_stores/test_opensearch.py:327`; `test_delete_col` `sources/mem0/tests/vector_stores/test_opensearch.py:333` |
| `sources/mem0/tests/vector_stores/test_oracledb.py:1` | 49 | no | `test_delete` `sources/mem0/tests/vector_stores/test_oracledb.py:861`; `test_delete_col_isolated` `sources/mem0/tests/vector_stores/test_oracledb.py:913` |
| `sources/mem0/tests/vector_stores/test_pgvector.py:1` | 87 | no | `test_delete_psycopg3` `sources/mem0/tests/vector_stores/test_pgvector.py:541`; `test_delete_psycopg2` `sources/mem0/tests/vector_stores/test_pgvector.py:580`; `test_delete_col_psycopg3` `sources/mem0/tests/vector_stores/test_pgvector.py:874`; `test_delete_col_psycopg2` `sources/mem0/tests/vector_stores/test_pgvector.py:913` |
| `sources/mem0/tests/vector_stores/test_pinecone.py:1` | 25 | no | `test_delete_vector` `sources/mem0/tests/vector_stores/test_pinecone.py:125`; `test_delete_col_scopes_to_namespace` `sources/mem0/tests/vector_stores/test_pinecone.py:146`; `test_delete_col_namespace_delete_error_is_swallowed` `sources/mem0/tests/vector_stores/test_pinecone.py:156` |
| `sources/mem0/tests/vector_stores/test_qdrant.py:1` | 96 | no | `test_delete` `sources/mem0/tests/vector_stores/test_qdrant.py:339`; `test_delete_col` `sources/mem0/tests/vector_stores/test_qdrant.py:481` |
| `sources/mem0/tests/vector_stores/test_s3_vectors.py:1` | 10 | no | `test_delete` `sources/mem0/tests/vector_stores/test_s3_vectors.py:212` |
| `sources/mem0/tests/vector_stores/test_supabase.py:1` | 14 | no | `test_delete_vector` `sources/mem0/tests/vector_stores/test_supabase.py:82` |
| `sources/mem0/tests/vector_stores/test_turbopuffer.py:1` | 64 | no | `test_delete_by_string_id` `sources/mem0/tests/vector_stores/test_turbopuffer.py:337`; `test_delete_by_int_id` `sources/mem0/tests/vector_stores/test_turbopuffer.py:341`; `test_delete_col` `sources/mem0/tests/vector_stores/test_turbopuffer.py:458`; `test_delete_col_handles_error` `sources/mem0/tests/vector_stores/test_turbopuffer.py:462`; `test_list_compatible_with_main_py_delete_all` `sources/mem0/tests/vector_stores/test_turbopuffer.py:561`; `test_reset_calls_delete_all` `sources/mem0/tests/vector_stores/test_turbopuffer.py:599` |
| `sources/mem0/tests/vector_stores/test_upstash_vector.py:1` | 29 | no | `test_delete_vector` `sources/mem0/tests/vector_stores/test_upstash_vector.py:86`; `test_delete_col` `sources/mem0/tests/vector_stores/test_upstash_vector.py:320` |
| `sources/mem0/tests/vector_stores/test_valkey.py:1` | 58 | no | `test_delete` `sources/mem0/tests/vector_stores/test_valkey.py:187`; `test_delete_col` `sources/mem0/tests/vector_stores/test_valkey.py:330`; `test_delete_error` `sources/mem0/tests/vector_stores/test_valkey.py:801`; `test_cluster_mode_delete` `sources/mem0/tests/vector_stores/test_valkey.py:1025` |
| `sources/mem0/tests/vector_stores/test_vertex_ai_vector_search.py:1` | 7 | no | `test_delete` `sources/mem0/tests/vector_stores/test_vertex_ai_vector_search.py:145` |
| `sources/mem0/tests/vector_stores/test_weaviate.py:1` | 12 | no | `test_delete` `sources/mem0/tests/vector_stores/test_weaviate.py:148`; `test_delete_col` `sources/mem0/tests/vector_stores/test_weaviate.py:210` |

## 7. Corrections to B2

Old values below are copied from the triage row (TRIAGE.md), not reconstructed from memory. A refinement narrows an over-broad field; it need not reverse its original value. Counts are distinct field paths, not the number of factual errors.

- `forget_tool.scope`: `"mem0_update replaces current text; history/vector/graph deletion scope requires separate checks."` → mem0_delete is a distinct ID-deletion tool; update is replacement; deletion retains old history text. Evidence: `sources/mem0/integrations/hermes-plugin-mem0/__init__.py:367`, `sources/mem0/mem0/memory/main.py:2112`.
- `engine_delete.scope`: `"Named deletion/retraction endpoint or method, not a guarantee of physical erasure. Its reach may overlap an agent tool; lack of a native tool is counted separately."` → `yes`: `mem0_delete` calls `_backend.delete(memory_id)` after the ownership check (body `sources/mem0/integrations/hermes-plugin-mem0/__init__.py:366`). `mem0_update` replaces text, not erasure (`sources/mem0/integrations/hermes-plugin-mem0/__init__.py:363`). OSS `Memory.delete` checks existence then calls `_delete_memory` (`sources/mem0/mem0/memory/main.py:1878`); `delete_all` iterates scoped batches (`sources/mem0/mem0/memory/main.py:1923`); `reset` resets history and vector/entity stores (`sources/mem0/mem0/memory/main.py:2139`). Async counterparts and HTTP/SDK surfaces are catalogued below. Single delete removes the vector, records the old text in history, and attempts entity-link cleanup (`sources/mem0/mem0/memory/main.py:2111`). Thus `partial`: history deliberately retains the fact; no host purge is called. Graph-wide deletion is not established by the single-vector path. Evidence: `sources/mem0/integrations/hermes-plugin-mem0/__init__.py:366`, `sources/mem0/integrations/hermes-plugin-mem0/__init__.py:363`, `sources/mem0/mem0/memory/main.py:1878`, `sources/mem0/mem0/memory/main.py:1923`, `sources/mem0/mem0/memory/main.py:2139`, `sources/mem0/mem0/memory/main.py:2111`.
- `context.mechanism.v`: `"Source expressions below identify retrieval, cached/static returns, lifecycle capture, and limits. A hook definition is not a guarantee it emits nonempty context each turn; branches and configuration apply."` → System assembly returns `# Mem0 Memory`, mode, user and tool instructions (`sources/mem0/integrations/hermes-plugin-mem0/__init__.py:261`). `_start_prefetch` assembles `## Mem0 Memory` and bullets of returned memory strings (`sources/mem0/integrations/hermes-plugin-mem0/__init__.py:281`); `prefetch` waits briefly, otherwise returns empty (`sources/mem0/integrations/hermes-plugin-mem0/__init__.py:302`). No character truncation in that assembly. Search tool clamps top_k to 1..50, default 10, and returns IDs/text/scores as JSON (`sources/mem0/integrations/hermes-plugin-mem0/__init__.py:336`). The 450-character setting is capture input, not the recall output cap (`sources/mem0/integrations/hermes-plugin-mem0/__init__.py:313`). Evidence: `sources/mem0/integrations/hermes-plugin-mem0/__init__.py:261`, `sources/mem0/integrations/hermes-plugin-mem0/__init__.py:281`, `sources/mem0/integrations/hermes-plugin-mem0/__init__.py:302`, `sources/mem0/integrations/hermes-plugin-mem0/__init__.py:336`, `sources/mem0/integrations/hermes-plugin-mem0/__init__.py:313`.
- `sec.telemetry.v`: `"inspect listed emitters/configuration"` → Credentials: active Hermes profile `.env` for MEM0_API_KEY/MEM0_HOST; `mem0.json` settings and mode-dependent OSS credential configuration (`sources/mem0/integrations/hermes-plugin-mem0/__init__.py:86`). Same-user direct access is `yes`, via the configured API or OSS store with the same credentials; tool ownership validation is not OS isolation (`sources/mem0/integrations/hermes-plugin-mem0/__init__.py:354`). OSS/SDK telemetry targets `https://us.i.posthog.com` (`sources/mem0/mem0/memory/telemetry.py:16`); see the exact enable predicate below. Accountless local mode is `yes` with OSS and local model/vector configuration; default Platform is not accountless. Telemetry is `on` by default: MEM0_TELEMETRY defaults to True and recognizes true/1/yes (`sources/mem0/mem0/memory/telemetry.py:14`; `sources/mem0/mem0/memory/telemetry.py:20`). Evidence: `sources/mem0/integrations/hermes-plugin-mem0/__init__.py:86`, `sources/mem0/integrations/hermes-plugin-mem0/__init__.py:354`, `sources/mem0/mem0/memory/telemetry.py:16`, `sources/mem0/mem0/memory/telemetry.py:14`, `sources/mem0/mem0/memory/telemetry.py:20`.
- `repro.decls`: `["integrations/hermes-plugin-mem0/plugin.yaml:4", "integrations/hermes-plugin-mem0/plugin.yaml:5", "integrations/hermes-plugin-mem0/pyproject.toml:3", "integrations/hermes-plugin-mem0/pyproject.toml:5", "integrations/hermes-plugin-mem0/pyproject.toml:10", "integrations/hermes-plugin-mem0/pyproject.toml:11", "integrations/hermes-plugin-mem0/pyproject.toml:12", "integrations/hermes-plugin-mem0/pyproject.toml:15", "pyproject.toml:7", "pyproject.toml:15", "pyproject.toml:16", "pyproject.toml:17", "pyproject.toml:18", "pyproject.toml:19", "pyproject.toml:20", "pyproject.toml:21", "pyproject.toml:22", "pyproject.toml:23", "pyproject.toml:24", "pyproject.toml:27", "pyproject.toml:29", "pyproject.toml:32", "pyproject.toml:33", "pyproject.toml:34", "pyproject.toml:35", "pyproject.toml:36", "pyproject.toml:37", "pyproject.toml:38", "pyproject.toml:39", "pyproject.toml:40", "pyproject.toml:41", "pyproject.toml:42", "pyproject.toml:43", "pyproject.toml:44", "pyproject.toml:45", "pyproject.toml:46", "pyproject.toml:47", "pyproject.toml:48", "pyproject.toml:49", "pyproject.toml:50", "pyproject.toml:51", "pyproject.toml:52", "pyproject.toml:53", "pyproject.toml:54", "pyproject.toml:55", "pyproject.toml:58", "pyproject.toml:59", "pyproject.toml:60", "pyproject.toml:61", "pyproject.toml:62", "pyproject.toml:63", "pyproject.toml:64", "pyproject.toml:65", "pyproject.toml:68", "pyproject.toml:69", "pyproject.toml:70", "pyproject.toml:71", "pyproject.toml:72", "pyproject.toml:73", "pyproject.toml:74", "pyproject.toml:75", "pyproject.toml:76", "pyproject.toml:79", "pyproject.toml:80", "pyproject.toml:81"]` → `range`: `mem0ai>=2.0.10,<3` (`sources/mem0/integrations/hermes-plugin-mem0/plugin.yaml:5`). This is not an exact engine pin, and snapshot HEAD is not necessarily the installed version. Evidence: `sources/mem0/integrations/hermes-plugin-mem0/plugin.yaml:5`.
- `tests.count`: `4759` → 17 adapter-related declarations using the stated parser/scope; B2 counted a different, repository-wide inventory. This is a scope refinement, not a count of executed or passing tests. Adapter tests: `integrations/hermes-plugin-mem0/tests/test_plugin.py`, 17 declarations. The adapter deletion path must not be credited from other clients' tests. Engine deletion tests exist, e.g. `sources/mem0/tests/test_main.py:268` and `sources/mem0/tests/test_memory.py:819`; these exercise deletion/history behavior, not universal erasure. Adapter transport delete is explicitly asserted in test_selfhosted_http_auth_and_tool_routes (`sources/mem0/integrations/hermes-plugin-mem0/tests/test_plugin.py:144`); this does not exercise the provider ownership handler. Evidence: `sources/mem0/tests/test_main.py:268`, `sources/mem0/tests/test_memory.py:819`, `sources/mem0/integrations/hermes-plugin-mem0/tests/test_plugin.py:144`.

MATRIX counts these 6 explicit field corrections/refinements. Sections 1-6 provide the body and test inventories. Test-name candidates are not passing tests or measured coverage.
