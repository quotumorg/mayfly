# reme: B3 static correctness review

Snapshot `5231f3970c550bb0bc5d1bc7fbc85534274ca530` (MANIFEST.json row `reme`). No plugin execution or passing-test claims.

## 1. Architecture

`provider`: automatic-only integration supports an HTTP backend and an embedded SDK backend, with serialized backend access and a background write queue. Embedded mode imports ReMe; HTTP mode does not make the SDK in-process (`sources/reme/integrations/hermes_agent/__init__.py:134`; `sources/reme/integrations/hermes_agent/embedded_backend.py:131`).

Hook/body inventory (no-op/fallback declarations are not active behavior):

| callable | declaration | first executable body line |
|---|---|---|
| `register` | `sources/reme/integrations/hermes_agent/__init__.py:616` | `sources/reme/integrations/hermes_agent/__init__.py:618` |
| `is_available` | `sources/reme/integrations/hermes_agent/__init__.py:89` | `sources/reme/integrations/hermes_agent/__init__.py:91` |
| `initialize` | `sources/reme/integrations/hermes_agent/__init__.py:107` | `sources/reme/integrations/hermes_agent/__init__.py:109` |
| `get_config_schema` | `sources/reme/integrations/hermes_agent/__init__.py:168` | `sources/reme/integrations/hermes_agent/__init__.py:170` |
| `save_config` | `sources/reme/integrations/hermes_agent/__init__.py:250` | `sources/reme/integrations/hermes_agent/__init__.py:252` |
| `get_tool_schemas` | `sources/reme/integrations/hermes_agent/__init__.py:265` | `sources/reme/integrations/hermes_agent/__init__.py:267` |
| `prefetch` | `sources/reme/integrations/hermes_agent/__init__.py:281` | `sources/reme/integrations/hermes_agent/__init__.py:283` |
| `sync_turn` | `sources/reme/integrations/hermes_agent/__init__.py:348` | `sources/reme/integrations/hermes_agent/__init__.py:357` |
| `on_session_switch` | `sources/reme/integrations/hermes_agent/__init__.py:381` | `sources/reme/integrations/hermes_agent/__init__.py:391` |
| `shutdown` | `sources/reme/integrations/hermes_agent/__init__.py:395` | `sources/reme/integrations/hermes_agent/__init__.py:397` |
| `save_config` | `sources/reme/integrations/hermes_agent/config.py:179` | `sources/reme/integrations/hermes_agent/config.py:181` |

## 2. Forget paths and derived copies

`none`: get_tool_schemas returns [] (`sources/reme/integrations/hermes_agent/__init__.py:267`). Engine `yes`: file-catalog delete, file-store delete, keyword/tag/graph node removal and change-index propagation are different local operations. Concrete implementations, not abstract base-method stubs, are inventoried below (`sources/reme/reme/components/file_catalog/local_file_catalog.py:40`; `sources/reme/reme/steps/index/update_changes.py:75`). Granularity is path/document/index entry, not a native agent fact-ID tool. Derived reach `partial`: index update has coordinated deletion branches, but a low-level catalogue deletion does not delete every source file, summary or host copy.

Host state.db is independent (`hermes-agent/hermes_state.py:181`). No reviewed adapter forget body performs a general purge of that database or built-in MEMORY.md/USER.md. Prompt/tool-result copies and independent source transcripts remain separate; source operation bodies below establish the scope of each plugin mutation.

<details>
<summary>Deletion, invalidation, archive and reset body ledger</summary>

Public routes, SDK/CLI dispatch and internal storage operations are distinguished by their source. Cache invalidation and management-object deletion are not additional fact-forget tools. A stub is not working deletion. Call-through means the downstream body still determines erasure. HTTP prefixes/selectors must be read with route registration.

**delete**, body `sources/reme/reme/components/agent_wrapper/cc_session_store.py:92`; implementation/call-through.

- `sources/reme/reme/components/agent_wrapper/cc_session_store.py:90` `async def delete(self, key: "SessionKey") -> None:`
- `sources/reme/reme/components/agent_wrapper/cc_session_store.py:94` `path.unlink()`

**clear**, body `sources/reme/reme/components/component_registry.py:107`; implementation/call-through.

- `sources/reme/reme/components/component_registry.py:105` `def clear(self) -> None:`
- `sources/reme/reme/components/component_registry.py:107` `with self._lock:`

**delete**, body `sources/reme/reme/components/file_catalog/base_file_catalog.py:35`; implementation/call-through.

- `sources/reme/reme/components/file_catalog/base_file_catalog.py:34` `@abstractmethod`
- `sources/reme/reme/components/file_catalog/base_file_catalog.py:35` `async def delete(self, path: str | list[str]) -> None:`

**delete**, body `sources/reme/reme/components/file_catalog/local_file_catalog.py:41`; implementation/call-through.

- `sources/reme/reme/components/file_catalog/local_file_catalog.py:40` `async def delete(self, path: str | list[str]) -> None:`
- `sources/reme/reme/components/file_catalog/local_file_catalog.py:41` `paths = [path] if isinstance(path, str) else path`

**delete_nodes**, body `sources/reme/reme/components/file_graph/base_file_graph.py:46`; implementation/call-through.

- `sources/reme/reme/components/file_graph/base_file_graph.py:45` `@abstractmethod`
- `sources/reme/reme/components/file_graph/base_file_graph.py:46` `async def delete_nodes(self, paths: list[str]) -> None:`

**clear**, body `sources/reme/reme/components/file_graph/base_file_graph.py:58`; implementation/call-through.

- `sources/reme/reme/components/file_graph/base_file_graph.py:57` `@abstractmethod`
- `sources/reme/reme/components/file_graph/base_file_graph.py:58` `async def clear(self):`

**delete_nodes**, body `sources/reme/reme/components/file_graph/local_file_graph.py:95`; implementation/call-through.

- `sources/reme/reme/components/file_graph/local_file_graph.py:94` `async def delete_nodes(self, paths: list[str]) -> None:`
- `sources/reme/reme/components/file_graph/local_file_graph.py:95` `for path in paths:`

**clear**, body `sources/reme/reme/components/file_graph/local_file_graph.py:118`; implementation/call-through.

- `sources/reme/reme/components/file_graph/local_file_graph.py:117` `async def clear(self):`
- `sources/reme/reme/components/file_graph/local_file_graph.py:121` `self._graph_file.unlink(missing_ok=True)`

**delete_nodes**, body `sources/reme/reme/components/file_graph/neo4j_file_graph.py:222`; implementation/call-through.

- `sources/reme/reme/components/file_graph/neo4j_file_graph.py:218` `async def delete_nodes(self, paths: list[str]) -> None:`
- `sources/reme/reme/components/file_graph/neo4j_file_graph.py:222` `if not paths:`

**_delete_nodes_tx**, body `sources/reme/reme/components/file_graph/neo4j_file_graph.py:233`; implementation/call-through.

- `sources/reme/reme/components/file_graph/neo4j_file_graph.py:228` `@staticmethod`
- `sources/reme/reme/components/file_graph/neo4j_file_graph.py:229` `async def _delete_nodes_tx(tx, paths):`
- `sources/reme/reme/components/file_graph/neo4j_file_graph.py:233` `remove_clause = ", ".join(f"f.{name}" for name in _REAL_PROPS)`

**clear**, body `sources/reme/reme/components/file_graph/neo4j_file_graph.py:349`; implementation/call-through.

- `sources/reme/reme/components/file_graph/neo4j_file_graph.py:347` `async def clear(self):`
- `sources/reme/reme/components/file_graph/neo4j_file_graph.py:349` `async with self._session() as session:`

**delete_nodes**, body `sources/reme/reme/components/file_graph/nx_file_graph.py:80`; implementation/call-through.

- `sources/reme/reme/components/file_graph/nx_file_graph.py:79` `async def delete_nodes(self, paths: list[str]) -> None:`
- `sources/reme/reme/components/file_graph/nx_file_graph.py:83` `self._graph.remove_edges_from(list(self._graph.out_edges(path, keys=True)))`
- `sources/reme/reme/components/file_graph/nx_file_graph.py:86` `self._graph.remove_node(path)`

**clear**, body `sources/reme/reme/components/file_graph/nx_file_graph.py:102`; implementation/call-through.

- `sources/reme/reme/components/file_graph/nx_file_graph.py:101` `async def clear(self):`
- `sources/reme/reme/components/file_graph/nx_file_graph.py:103` `self._graph_file.unlink(missing_ok=True)`

**delete**, body `sources/reme/reme/components/file_store/base_file_store.py:79`; implementation/call-through.

- `sources/reme/reme/components/file_store/base_file_store.py:78` `@abstractmethod`
- `sources/reme/reme/components/file_store/base_file_store.py:79` `async def delete(self, path: str | list[str]) -> None:`

**clear**, body `sources/reme/reme/components/file_store/base_file_store.py:83`; implementation/call-through.

- `sources/reme/reme/components/file_store/base_file_store.py:82` `@abstractmethod`
- `sources/reme/reme/components/file_store/base_file_store.py:83` `async def clear(self) -> None:`

**_tombstone**, body `sources/reme/reme/components/file_store/faiss_local_file_store.py:160`; implementation/call-through.

- `sources/reme/reme/components/file_store/faiss_local_file_store.py:159` `def _tombstone(self, chunk_id: str) -> None:`
- `sources/reme/reme/components/file_store/faiss_local_file_store.py:162` `self._tombstones.add(row)`

**_tombstone_threshold**, body `sources/reme/reme/components/file_store/faiss_local_file_store.py:185`; implementation/call-through.

- `sources/reme/reme/components/file_store/faiss_local_file_store.py:177` `def _tombstone_threshold(self, scale: float = 1.0) -> int:`
- `sources/reme/reme/components/file_store/faiss_local_file_store.py:185` `if self.max_tombstones is not None:`
- `sources/reme/reme/components/file_store/faiss_local_file_store.py:186` `return self.max_tombstones`
- `sources/reme/reme/components/file_store/faiss_local_file_store.py:188` `return max(int(self.tombstone_compact_ratio * ntotal * scale), 128)`

**delete**, body `sources/reme/reme/components/file_store/faiss_local_file_store.py:604`; implementation/call-through.

- `sources/reme/reme/components/file_store/faiss_local_file_store.py:602` `@BaseFileStore.serialized`
- `sources/reme/reme/components/file_store/faiss_local_file_store.py:603` `async def delete(self, path: str | list[str]) -> None:`
- `sources/reme/reme/components/file_store/faiss_local_file_store.py:614` `self._tombstone(cid)`

**clear**, body `sources/reme/reme/components/file_store/faiss_local_file_store.py:636`; implementation/call-through.

- `sources/reme/reme/components/file_store/faiss_local_file_store.py:627` `@BaseFileStore.serialized`
- `sources/reme/reme/components/file_store/faiss_local_file_store.py:628` `async def clear(self) -> None:`
- `sources/reme/reme/components/file_store/faiss_local_file_store.py:642` `self._tombstones.clear()`
- `sources/reme/reme/components/file_store/faiss_local_file_store.py:643` `self.faiss_path.unlink(missing_ok=True)`
- `sources/reme/reme/components/file_store/faiss_local_file_store.py:644` `self.faiss_idmap_path.unlink(missing_ok=True)`

**_invalidate_stale_embeddings**, body `sources/reme/reme/components/file_store/local_file_store.py:333`; implementation/call-through.

- `sources/reme/reme/components/file_store/local_file_store.py:331` `def _invalidate_stale_embeddings(self) -> None:`
- `sources/reme/reme/components/file_store/local_file_store.py:333` `if self.embedding_store is None:`

**_delete_tag_paths**, body `sources/reme/reme/components/file_store/local_file_store.py:706`; implementation/call-through.

- `sources/reme/reme/components/file_store/local_file_store.py:704` `async def _delete_tag_paths(self, paths: list[str]) -> None:`
- `sources/reme/reme/components/file_store/local_file_store.py:713` `await tag_index.delete(paths)`

**delete**, body `sources/reme/reme/components/file_store/local_file_store.py:867`; implementation/call-through.

- `sources/reme/reme/components/file_store/local_file_store.py:865` `@BaseFileStore.serialized`
- `sources/reme/reme/components/file_store/local_file_store.py:866` `async def delete(self, path: str | list[str]) -> None:`
- `sources/reme/reme/components/file_store/local_file_store.py:867` `assert self.file_graph is not None`

**_delete_nodes**, body `sources/reme/reme/components/file_store/local_file_store.py:881`; implementation/call-through.

- `sources/reme/reme/components/file_store/local_file_store.py:874` `async def _delete_nodes(self, nodes: list[FileNode]) -> None:`
- `sources/reme/reme/components/file_store/local_file_store.py:887` `await self.file_graph.delete_nodes([str(n.path) for n in nodes])`
- `sources/reme/reme/components/file_store/local_file_store.py:890` `await self.keyword_index.delete_docs(deleted_chunk_ids)`

**clear**, body `sources/reme/reme/components/file_store/local_file_store.py:914`; implementation/call-through.

- `sources/reme/reme/components/file_store/local_file_store.py:912` `@BaseFileStore.serialized`
- `sources/reme/reme/components/file_store/local_file_store.py:913` `async def clear(self) -> None:`
- `sources/reme/reme/components/file_store/local_file_store.py:916` `self.chunks_path.unlink(missing_ok=True)`

**_delete_docs**, body `sources/reme/reme/components/file_store/zvec_local_file_store.py:149`; implementation/call-through.

- `sources/reme/reme/components/file_store/zvec_local_file_store.py:148` `def _delete_docs(self, chunk_ids: list[str]) -> None:`
- `sources/reme/reme/components/file_store/zvec_local_file_store.py:154` `self._collection.delete(ids=stale)`

**delete**, body `sources/reme/reme/components/file_store/zvec_local_file_store.py:413`; implementation/call-through.

- `sources/reme/reme/components/file_store/zvec_local_file_store.py:411` `@BaseFileStore.serialized`
- `sources/reme/reme/components/file_store/zvec_local_file_store.py:412` `async def delete(self, path: str | list[str]) -> None:`
- `sources/reme/reme/components/file_store/zvec_local_file_store.py:413` `assert self.file_graph is not None`

**clear**, body `sources/reme/reme/components/file_store/zvec_local_file_store.py:426`; implementation/call-through.

- `sources/reme/reme/components/file_store/zvec_local_file_store.py:424` `@BaseFileStore.serialized`
- `sources/reme/reme/components/file_store/zvec_local_file_store.py:425` `async def clear(self) -> None:`
- `sources/reme/reme/components/file_store/zvec_local_file_store.py:435` `self.zvec_sidecar_path.unlink(missing_ok=True)`

**delete_docs**, body `sources/reme/reme/components/keyword_index/base_keyword_index.py:50`; implementation/call-through.

- `sources/reme/reme/components/keyword_index/base_keyword_index.py:49` `@abstractmethod`
- `sources/reme/reme/components/keyword_index/base_keyword_index.py:50` `async def delete_docs(self, doc_ids: list[str]) -> None:`

**clear**, body `sources/reme/reme/components/keyword_index/base_keyword_index.py:82`; implementation/call-through.

- `sources/reme/reme/components/keyword_index/base_keyword_index.py:81` `@abstractmethod`
- `sources/reme/reme/components/keyword_index/base_keyword_index.py:82` `async def clear(self) -> None:`

**delete_docs**, body `sources/reme/reme/components/keyword_index/bm25_index.py:279`; implementation/call-through.

- `sources/reme/reme/components/keyword_index/bm25_index.py:277` `async def delete_docs(self, doc_ids: list[str]) -> None:`
- `sources/reme/reme/components/keyword_index/bm25_index.py:279` `for doc_id in doc_ids:`

**clear**, body `sources/reme/reme/components/keyword_index/bm25_index.py:465`; implementation/call-through.

- `sources/reme/reme/components/keyword_index/bm25_index.py:463` `async def clear(self) -> None:`
- `sources/reme/reme/components/keyword_index/bm25_index.py:475` `self.index_file.unlink(missing_ok=True)`

**delete**, body `sources/reme/reme/components/tag_index/base_tag_index.py:98`; implementation/call-through.

- `sources/reme/reme/components/tag_index/base_tag_index.py:97` `@abstractmethod`
- `sources/reme/reme/components/tag_index/base_tag_index.py:98` `async def delete(self, paths: list[str]) -> None:`

**clear**, body `sources/reme/reme/components/tag_index/base_tag_index.py:121`; implementation/call-through.

- `sources/reme/reme/components/tag_index/base_tag_index.py:120` `@abstractmethod`
- `sources/reme/reme/components/tag_index/base_tag_index.py:121` `async def clear(self) -> None:`

**delete**, body `sources/reme/reme/components/tag_index/local_tag_index.py:122`; implementation/call-through.

- `sources/reme/reme/components/tag_index/local_tag_index.py:121` `async def delete(self, paths: list[str]) -> None:`
- `sources/reme/reme/components/tag_index/local_tag_index.py:122` `validated = [self._validate_path(path) for path in paths]`

**clear**, body `sources/reme/reme/components/tag_index/local_tag_index.py:202`; implementation/call-through.

- `sources/reme/reme/components/tag_index/local_tag_index.py:201` `async def clear(self) -> None:`
- `sources/reme/reme/components/tag_index/local_tag_index.py:202` `async with self._maintenance_lock:`

**__delete__**, body `sources/reme/reme/steps/base_step.py:71`; implementation/call-through.

- `sources/reme/reme/steps/base_step.py:70` `def __delete__(self, obj: "BaseStep") -> None:`
- `sources/reme/reme/steps/base_step.py:71` `obj.__dict__.pop(self._cache_attr, None)`

**_handle_delete**, body `sources/reme/reme/steps/evolve/base_auto_resource.py:506`; implementation/call-through.

- `sources/reme/reme/steps/evolve/base_auto_resource.py:505` `async def _handle_delete(self, file_path: str, date_str: str, note_stem: str) -> None:`
- `sources/reme/reme/steps/evolve/base_auto_resource.py:528` `note_abs.unlink()`
- `sources/reme/reme/steps/evolve/base_auto_resource.py:540` `await self.file_store.delete([note_rel])`

**_delete**, body `sources/reme/reme/steps/file_io/delete.py:71`; implementation/call-through.

- `sources/reme/reme/steps/file_io/delete.py:70` `async def _delete(self, path: str) -> dict:`
- `sources/reme/reme/steps/file_io/delete.py:79` `target.unlink()`

**delete_paths**, body `sources/reme/reme/steps/index/update_changes.py:75`; implementation/call-through.

- `sources/reme/reme/steps/index/update_changes.py:74` `@abstractmethod`
- `sources/reme/reme/steps/index/update_changes.py:75` `async def delete_paths(self, paths: list[str]) -> None:`

**_apply_deleted**, body `sources/reme/reme/steps/index/update_changes.py:274`; implementation/call-through.

- `sources/reme/reme/steps/index/update_changes.py:273` `async def _apply_deleted(self, deleted: list[str]) -> list[dict]:`
- `sources/reme/reme/steps/index/update_changes.py:282` `await self.delete_paths([self.to_workspace_relative(p) for p in batch])`

**delete_paths**, body `sources/reme/reme/steps/index/update_changes.py:310`; implementation/call-through.

- `sources/reme/reme/steps/index/update_changes.py:309` `async def delete_paths(self, paths: list[str]) -> None:`
- `sources/reme/reme/steps/index/update_changes.py:312` `await self.file_catalog.delete(paths)`

**delete_paths**, body `sources/reme/reme/steps/index/update_changes.py:333`; implementation/call-through.

- `sources/reme/reme/steps/index/update_changes.py:332` `async def delete_paths(self, paths: list[str]) -> None:`
- `sources/reme/reme/steps/index/update_changes.py:333` `await self.file_store.delete(paths)`

**delete**, body `sources/reme/reme/utils/agent_state_io.py:44`; implementation/call-through.

- `sources/reme/reme/utils/agent_state_io.py:42` `async def delete(self) -> bool:`
- `sources/reme/reme/utils/agent_state_io.py:46` `self.path.unlink()`

</details>

## 3. What reaches the agent

Prefetch calls backend.search with configured recall_limit and returns the response's `answer` string, stripped, without an additional character truncation at this layer (`sources/reme/integrations/hermes_agent/__init__.py:309`; `sources/reme/integrations/hermes_agent/__init__.py:326`). No model-visible tools. The provider inherits any empty system block unless implemented in the body inventory; do not infer a prompt from README prose.

Host assembly: `hermes-agent/agent/memory_manager.py:352` joins provider system blocks. `hermes-agent/agent/turn_context.py:1081` requests prefetch for nontrivial turns; `hermes-agent/agent/turn_context.py:120` fences it; `hermes-agent/agent/turn_context.py:136` appends it to API-bound user content. Tool results are a third channel. Optional host spill is at `hermes-agent/agent/memory_manager.py:419`, not a universal provider cap. Built-in memory remains separately controlled (`hermes-agent/agent/system_prompt.py:542`).

<details>
<summary>Context and tool-result assembly quotes</summary>

`get_tool_schemas`, body `sources/reme/integrations/hermes_agent/__init__.py:267`:

- `sources/reme/integrations/hermes_agent/__init__.py:267` `return []`

`prefetch`, body `sources/reme/integrations/hermes_agent/__init__.py:283`:

- `sources/reme/integrations/hermes_agent/__init__.py:287` `return ""`
- `sources/reme/integrations/hermes_agent/__init__.py:296` `return ""`
- `sources/reme/integrations/hermes_agent/__init__.py:299` `return ""`
- `sources/reme/integrations/hermes_agent/__init__.py:308` `return ""`
- `sources/reme/integrations/hermes_agent/__init__.py:311` `limit=self._recall_limit,`
- `sources/reme/integrations/hermes_agent/__init__.py:321` `return ""`
- `sources/reme/integrations/hermes_agent/__init__.py:333` `return answer`

</details>

## 4. Data access boundaries

Direct access `yes`: workspace files/indexes in embedded mode or configured HTTP endpoint. Configurable workspace/model endpoints and environment reads are quoted below. Accountless local mode `yes` with local model configuration; installation/model availability is not tested. No adapter telemetry emitter was identified; transitive SDK/model telemetry remains `unknown`.

Boundary model: same OS user, terminal, mounted filesystem and allowed network. Direct reachability means another client can use the same file/endpoint and available credentials, not unauthenticated access or encryption bypass. Hermes setup persists declared secrets through `hermes-agent/hermes_cli/memory_setup.py:321` and `hermes-agent/hermes_cli/config.py:2697`. These are possible configured paths; no live credentials were read.

Host credential-file detail: `get_hermes_home()/.env` is resolved at `hermes-agent/hermes_cli/config.py:473`. New files are secured to 0600 at `hermes-agent/hermes_cli/config.py:506`; the writer preserves an existing mode at `hermes-agent/hermes_cli/config.py:2544` and `:2562`. Therefore 0600 is not an unconditional mode claim for every existing profile.

<details>
<summary>Paths, environment, permissions, egress, telemetry and process-launch evidence</summary>

- `sources/reme/integrations/hermes_agent/config.py:31` `endpoint: str = "http://127.0.0.1:2333"`
- `sources/reme/integrations/hermes_agent/config.py:203` `os.chmod(tmp_name, 0o600)`

- `sources/reme/integrations/hermes_agent/config_schema.py:17` `docs_url="https://github.com/agentscope-ai/ReMe/tree/main/integrations/hermes_agent",`
- `sources/reme/integrations/hermes_agent/config_schema.py:43` `default="http://127.0.0.1:2333",`
- `sources/reme/integrations/hermes_agent/config_schema.py:45` `placeholder="http://127.0.0.1:2333",`

- `sources/reme/integrations/hermes_agent/plugin.yaml:7` `homepage: https://github.com/agentscope-ai/ReMe`

- `sources/reme/reme/config/benchmark.yaml:373` `base_url: ${EMBEDDING_BASE_URL:-https://dashscope.aliyuncs.com/compatible-mode/v1}`

- `sources/reme/reme/config/config_parser.py:33` `v = os.environ.get(name)`

- `sources/reme/reme/config/cookbook.yaml:68` `base_url: ${EMBEDDING_BASE_URL:-https://dashscope.aliyuncs.com/compatible-mode/v1}`
- `sources/reme/reme/config/cookbook.yaml:84` `base_url: ${LLM_BASE_URL:-https://dashscope.aliyuncs.com/compatible-mode/v1}`
- `sources/reme/reme/config/cookbook.yaml:91` `base_url: ${CLAUDE_CODE_BASE_URL:-https://dashscope.aliyuncs.com/apps/anthropic}`

- `sources/reme/reme/config/default.yaml:882` `base_url: ${CLAUDE_CODE_BASE_URL:-https://dashscope.aliyuncs.com/apps/anthropic}`

- `sources/reme/reme/config/demo.yaml:113` `base_url: ${LLM_BASE_URL:-https://dashscope.aliyuncs.com/compatible-mode/v1}`
- `sources/reme/reme/config/demo.yaml:137` `base_url: ${CLAUDE_CODE_BASE_URL:-https://dashscope.aliyuncs.com/apps/anthropic}`

- `sources/reme/reme/schema/application_config.py:32` `app_name: str = Field(default=os.getenv("APP_NAME", "ReMe"), description="Application display name")`

- `sources/reme/reme_studio/vite.config.ts:15` `const isCodexSeatbeltSandbox = process.env.CODEX_SANDBOX === "seatbelt";`
- `sources/reme/reme_studio/vite.config.ts:42` `process.env.WRANGLER_WRITE_LOGS ??= "false";`
- `sources/reme/reme_studio/vite.config.ts:43` `process.env.WRANGLER_LOG_PATH ??= ".wrangler/logs";`
- `sources/reme/reme_studio/vite.config.ts:44` `process.env.MINIFLARE_REGISTRY_PATH ??= ".wrangler/registry";`

- `sources/reme/reme_studio/vite.static.config.ts:19` `"process.env.NEXT_PUBLIC_REME_API_URL": JSON.stringify(`
- `sources/reme/reme_studio/vite.static.config.ts:22` `"process.env.NEXT_PUBLIC_REME_WORKSPACE_EXTENSIONS": JSON.stringify(`

</details>

## 5. Reproducibility

Hermes manifest pip_dependencies is empty (`sources/reme/integrations/hermes_agent/plugin.yaml:10`). Embedded backend's installation hint is unpinned reme-ai[core] (`sources/reme/integrations/hermes_agent/embedded_backend.py:131`). HTTP server version is external. Engine pin classification `unpinned`.

Retained lock/checksum files: `sources/reme/github-pages/package-lock.json:1`, `sources/reme/reme_studio/package-lock.json:1`, `sources/reme/integrations/openclaw/package-lock.json:1`, `sources/reme/integrations/dsh/package-lock.json:1`. Upstream completeness is unknown: fetch.py:15 excludes .lock files, fetch.py:12 excludes build/vendor directories. Absence here is not absence upstream.

- `sources/reme/integrations/hermes_agent/plugin.yaml:2` `version: 0.1.0`
- `sources/reme/integrations/hermes_agent/plugin.yaml:5` `requires_hermes: ">=0.21"`
- `sources/reme/integrations/hermes_agent/plugin.yaml:10` `pip_dependencies: []`

## 6. Tests

Adapter tests: tests/unit/test_hermes_agent_integration.py. No adapter forget tool/test. Storage and index-deletion tests are separate, with counts below. Storage test deletes a single path and a list and asserts remaining catalogue paths. (`sources/reme/tests/unit/test_file_catalog.py:91`).

Adapter-related declarations: **28**. Python AST test_ functions, fixtures excluded; literal TS/JS it/test, Go Test*, Rust test_* declarations. No parametrization expansion, collection, execution, or coverage measurement. Other rows are engine test candidates selected by mutation names; names alone do not verify forget coverage.

| file | declarations | adapter-related | forget-related declarations |
|---|---:|---|---|
| `sources/reme/tests/integration/test_auto_resource.py:1` | 3 | no | `test_auto_resource_delete` `sources/reme/tests/integration/test_auto_resource.py:276` |
| `sources/reme/tests/unit/test_auto_resource_batch_lookup.py:1` | 15 | no | `test_create_update_delete_recreate_maintains_ownership_across_midnight` `sources/reme/tests/unit/test_auto_resource_batch_lookup.py:241`; `test_repeated_source_retries_after_partial_write_or_delete` `sources/reme/tests/unit/test_auto_resource_batch_lookup.py:277`; `test_same_day_duplicates_keep_first_match_after_deletion` `sources/reme/tests/unit/test_auto_resource_batch_lookup.py:386` |
| `sources/reme/tests/unit/test_auto_resource_review_regressions.py:1` | 9 | no | `test_loose_root_text_updates_and_deletes_original_daily_card` `sources/reme/tests/unit/test_auto_resource_review_regressions.py:360` |
| `sources/reme/tests/unit/test_background_steps.py:1` | 64 | no | `test_scan_changes_detect_modify_delete` `sources/reme/tests/unit/test_background_steps.py:349`; `test_update_catalog_deletes_in_batches_of_at_most_100_paths` `sources/reme/tests/unit/test_background_steps.py:527`; `test_auto_resource_batch_deleted_changes` `sources/reme/tests/unit/test_background_steps.py:1220`; `test_auto_resource_rejects_traversal_delete_without_touching_note` `sources/reme/tests/unit/test_background_steps.py:1421`; `test_auto_resource_deletes_renamed_note_by_source_resource` `sources/reme/tests/unit/test_background_steps.py:1968` |
| `sources/reme/tests/unit/test_codex_agent_wrapper.py:1` | 44 | no | `test_codex_skills_add_all_without_deleting_existing_content` `sources/reme/tests/unit/test_codex_agent_wrapper.py:470` |
| `sources/reme/tests/unit/test_crud_steps.py:1` | 55 | no | `test_list_mtime_sort_skips_files_deleted_during_scan` `sources/reme/tests/unit/test_crud_steps.py:354`; `test_delete_removes_file` `sources/reme/tests/unit/test_crud_steps.py:515`; `test_delete_missing_returns_error` `sources/reme/tests/unit/test_crud_steps.py:532`; `test_delete_reports_inbound_refs` `sources/reme/tests/unit/test_crud_steps.py:548`; `test_delete_folder_removes_tree` `sources/reme/tests/unit/test_crud_steps.py:587`; `test_delete_folder_reports_only_external_inbound` `sources/reme/tests/unit/test_crud_steps.py:618`; `test_delete_folder_empty_has_no_inbound` `sources/reme/tests/unit/test_crud_steps.py:658` |
| `sources/reme/tests/unit/test_file_catalog.py:1` | 6 | no | `test_delete_single_and_list` `sources/reme/tests/unit/test_file_catalog.py:91` |
| `sources/reme/tests/unit/test_file_graph.py:1` | 13 | no | `test_delete_keeps_inbound_view` `sources/reme/tests/unit/test_file_graph.py:205`; `test_delete_outgoing_links_cleared` `sources/reme/tests/unit/test_file_graph.py:250` |
| `sources/reme/tests/unit/test_file_store_consistency.py:1` | 68 | no | `test_faiss_delete_queries_graph_once` `sources/reme/tests/unit/test_file_store_consistency.py:2371` |
| `sources/reme/tests/unit/test_frontmatter_steps.py:1` | 11 | no | `test_delete_no_suffix_autoappends_md` `sources/reme/tests/unit/test_frontmatter_steps.py:146`; `test_delete_missing_reports_probed_path` `sources/reme/tests/unit/test_frontmatter_steps.py:207` |
| `sources/reme/tests/unit/test_hermes_agent_integration.py:1` | 28 | yes | none |
| `sources/reme/tests/unit/test_keyword_index.py:1` | 48 | no | `test_delete_single_doc` `sources/reme/tests/unit/test_keyword_index.py:215`; `test_delete_multiple_docs` `sources/reme/tests/unit/test_keyword_index.py:237`; `test_delete_nonexistent_is_noop` `sources/reme/tests/unit/test_keyword_index.py:254`; `test_re_add_after_delete` `sources/reme/tests/unit/test_keyword_index.py:268`; `test_optimize_drops_deleted_only_terms` `sources/reme/tests/unit/test_keyword_index.py:940`; `test_optimize_when_all_deleted_clears_index` `sources/reme/tests/unit/test_keyword_index.py:970`; `test_optimize_noop_when_no_deletions` `sources/reme/tests/unit/test_keyword_index.py:987`; `test_idf_cache_populates_and_invalidates` `sources/reme/tests/unit/test_keyword_index.py:1024`; `test_deleted_docs_excluded_from_scoring` `sources/reme/tests/unit/test_keyword_index.py:1073`; `test_inverted_index_hides_deleted_postings` `sources/reme/tests/unit/test_keyword_index.py:1090` |
| `sources/reme/tests/unit/test_tag_index.py:1` | 18 | no | `test_tag_delete_failures_do_not_block_core_deletion` `sources/reme/tests/unit/test_tag_index.py:512` |
| `sources/reme/tests/unit/test_zvec_file_store.py:1` | 20 | no | `test_zvec_delete_removes_vectors` `sources/reme/tests/unit/test_zvec_file_store.py:226` |

## 7. Corrections to B2

Old values below are copied from the triage row (TRIAGE.md), not reconstructed from memory. A refinement narrows an over-broad field; it need not reverse its original value. Counts are distinct field paths, not the number of factual errors.

- `engine_delete.e`: `["reme/components/file_catalog/local_file_catalog.py:40"]` → This path is relative to the plugin repo: sources/reme/reme/components/file_catalog/local_file_catalog.py. It is catalogue deletion, not a fact-forget endpoint or proof of source-file erasure. Evidence: `sources/reme/reme/components/file_catalog/local_file_catalog.py:40`.
- `engine_delete.scope`: `"Named deletion/retraction endpoint or method, not a guarantee of physical erasure. Its reach may overlap an agent tool; lack of a native tool is counted separately."` → `none`: get_tool_schemas returns [] (`sources/reme/integrations/hermes_agent/__init__.py:267`). Engine `yes`: file-catalog delete, file-store delete, keyword/tag/graph node removal and change-index propagation are different local operations. Concrete implementations, not abstract base-method stubs, are inventoried below (`sources/reme/reme/components/file_catalog/local_file_catalog.py:40`; `sources/reme/reme/steps/index/update_changes.py:75`). Granularity is path/document/index entry, not a native agent fact-ID tool. Derived reach `partial`: index update has coordinated deletion branches, but a low-level catalogue deletion does not delete every source file, summary or host copy. Evidence: `sources/reme/integrations/hermes_agent/__init__.py:267`, `sources/reme/reme/components/file_catalog/local_file_catalog.py:40`, `sources/reme/reme/steps/index/update_changes.py:75`.
- `context.mechanism.v`: `"Source expressions below identify retrieval, cached/static returns, lifecycle capture, and limits. A hook definition is not a guarantee it emits nonempty context each turn; branches and configuration apply."` → Prefetch calls backend.search with configured recall_limit and returns the response's `answer` string, stripped, without an additional character truncation at this layer (`sources/reme/integrations/hermes_agent/__init__.py:309`; `sources/reme/integrations/hermes_agent/__init__.py:326`). No model-visible tools. The provider inherits any empty system block unless implemented in the body inventory; do not infer a prompt from README prose. Evidence: `sources/reme/integrations/hermes_agent/__init__.py:309`, `sources/reme/integrations/hermes_agent/__init__.py:326`.
- `sec.telemetry.v`: `"unknown: no emitter identified in the selected adapter files; external dependencies not cleared"` → Direct access `yes`: workspace files/indexes in embedded mode or configured HTTP endpoint. Configurable workspace/model endpoints and environment reads are quoted below. Accountless local mode `yes` with local model configuration; installation/model availability is not tested. No adapter telemetry emitter was identified; transitive SDK/model telemetry remains `unknown`. Evidence: `sources/reme/integrations/hermes_agent/config.py:31`, `sources/reme/integrations/hermes_agent/config.py:203`, `sources/reme/integrations/hermes_agent/config_schema.py:17`, `sources/reme/integrations/hermes_agent/config_schema.py:43`.
- `repro.decls`: `["integrations/hermes_agent/plugin.yaml:7", "integrations/hermes_agent/plugin.yaml:10", "pyproject.toml:3", "pyproject.toml:10", "pyproject.toml:23", "pyproject.toml:24", "pyproject.toml:25", "pyproject.toml:26", "pyproject.toml:27", "pyproject.toml:28", "pyproject.toml:29", "pyproject.toml:30", "pyproject.toml:31", "pyproject.toml:32", "pyproject.toml:33", "pyproject.toml:34", "pyproject.toml:35", "pyproject.toml:36", "pyproject.toml:37", "pyproject.toml:38", "pyproject.toml:39", "pyproject.toml:40", "pyproject.toml:43", "pyproject.toml:45", "pyproject.toml:52", "pyproject.toml:53", "pyproject.toml:54", "pyproject.toml:55", "pyproject.toml:56", "pyproject.toml:57", "pyproject.toml:58", "pyproject.toml:59", "pyproject.toml:60", "pyproject.toml:61", "pyproject.toml:62", "pyproject.toml:66", "pyproject.toml:67", "pyproject.toml:70", "pyproject.toml:71", "pyproject.toml:72", "pyproject.toml:73", "pyproject.toml:82", "pyproject.toml:84", "pyproject.toml:98", "pyproject.toml:101"]` → Hermes manifest pip_dependencies is empty (`sources/reme/integrations/hermes_agent/plugin.yaml:10`). Embedded backend's installation hint is unpinned reme-ai[core] (`sources/reme/integrations/hermes_agent/embedded_backend.py:131`). HTTP server version is external. Engine pin classification `unpinned`. Evidence: `sources/reme/integrations/hermes_agent/plugin.yaml:10`, `sources/reme/integrations/hermes_agent/embedded_backend.py:131`.
- `tests.count`: `1312` → 28 adapter-related declarations using the stated parser/scope; B2 counted a different, repository-wide inventory. This is a scope refinement, not a count of executed or passing tests. Adapter tests: tests/unit/test_hermes_agent_integration.py. No adapter forget tool/test. Storage and index-deletion tests are separate, with counts below. Storage test deletes a single path and a list and asserts remaining catalogue paths. (`sources/reme/tests/unit/test_file_catalog.py:91`). Evidence: `sources/reme/tests/unit/test_file_catalog.py:91`.

MATRIX counts these 6 explicit field corrections/refinements. Sections 1-6 provide the body and test inventories. Test-name candidates are not passing tests or measured coverage.
