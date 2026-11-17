# hindsight: B3 static correctness review

Snapshot `f14cc1eb5624b0374cb4d82768eb2e0b9c21ae24` (MANIFEST.json row `hindsight`). No plugin execution or passing-test claims.

## 1. Architecture

`provider`. Hermes uses a Hindsight client; embedded mode starts a local daemon, with embedded PostgreSQL supplied by an external pg0 dependency. Remote mode uses configured HTTP. The adapter composes the client/daemon rather than treating PostgreSQL as provider-owned Python memory (`sources/hindsight/hindsight-integrations/hermes/__init__.py:900`).

Hook/body inventory (no-op/fallback declarations are not active behavior):

| callable | declaration | first executable body line |
|---|---|---|
| `register` | `sources/hindsight/hindsight-integrations/hermes/__init__.py:2283` | `sources/hindsight/hindsight-integrations/hermes/__init__.py:2285` |
| `is_available` | `sources/hindsight/hindsight-integrations/hermes/__init__.py:601` | `sources/hindsight/hindsight-integrations/hermes/__init__.py:602` |
| `save_config` | `sources/hindsight/hindsight-integrations/hermes/__init__.py:636` | `sources/hindsight/hindsight-integrations/hermes/__init__.py:638` |
| `get_config_schema` | `sources/hindsight/hindsight-integrations/hermes/__init__.py:649` | `sources/hindsight/hindsight-integrations/hermes/__init__.py:650` |
| `initialize` | `sources/hindsight/hindsight-integrations/hermes/__init__.py:1284` | `sources/hindsight/hindsight-integrations/hermes/__init__.py:1285` |
| `system_prompt_block` | `sources/hindsight/hindsight-integrations/hermes/__init__.py:1661` | `sources/hindsight/hindsight-integrations/hermes/__init__.py:1662` |
| `prefetch` | `sources/hindsight/hindsight-integrations/hermes/__init__.py:1801` | `sources/hindsight/hindsight-integrations/hermes/__init__.py:1805` |
| `queue_prefetch` | `sources/hindsight/hindsight-integrations/hermes/__init__.py:1820` | `sources/hindsight/hindsight-integrations/hermes/__init__.py:1822` |
| `sync_turn` | `sources/hindsight/hindsight-integrations/hermes/__init__.py:2007` | `sources/hindsight/hindsight-integrations/hermes/__init__.py:2010` |
| `on_memory_write` | `sources/hindsight/hindsight-integrations/hermes/__init__.py:2074` | `sources/hindsight/hindsight-integrations/hermes/__init__.py:2078` |
| `get_tool_schemas` | `sources/hindsight/hindsight-integrations/hermes/__init__.py:2105` | `sources/hindsight/hindsight-integrations/hermes/__init__.py:2106` |
| `handle_tool_call` | `sources/hindsight/hindsight-integrations/hermes/__init__.py:2153` | `sources/hindsight/hindsight-integrations/hermes/__init__.py:2154` |
| `on_session_switch` | `sources/hindsight/hindsight-integrations/hermes/__init__.py:2180` | `sources/hindsight/hindsight-integrations/hermes/__init__.py:2199` |
| `shutdown` | `sources/hindsight/hindsight-integrations/hermes/__init__.py:2241` | `sources/hindsight/hindsight-integrations/hermes/__init__.py:2242` |

## 2. Forget paths and derived copies

`none`: native tools are retain, recall, reflect, not forget (`sources/hindsight/hindsight-integrations/hermes/__init__.py:2105`). Engine `yes` for reversible per-fact invalidation and document/bank deletion. PATCH `/v1/default/banks/{bank_id}/memories/{memory_id}` dispatches to `update_memory_unit(state=...)` (`sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:5339`). It is not a single-fact DELETE. CLI `delete_memory` is `stub`: body `anyhow::bail!("Individual memory deletion is no longer supported...")` (`sources/hindsight/hindsight-cli/src/api.rs:468`). DELETE document invokes `delete_document` (`sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:7522`); bank deletion calls `delete_bank` (`sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:8030`); clear memories retains the profile (`sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:9882`). Derived reach is `partial`: invalidation retains audit content; document/bank cleanup is a different operation. No host MEMORY.md/state.db erasure. PostgreSQL invalidation actually copies the fact plus entity IDs/causal descriptors into invalidated_memory_units, deletes live links and live row, and preserves reversible content (`sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/pg/writes.py:439`; `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/pg/writes.py:453`). The high-level update_memory_unit implementation itself is absent from the filtered snapshot; route dispatch and storage primitives are present, so complete orchestration/caches cannot be established.

Host state.db is independent (`hermes-agent/hermes_state.py:181`). No reviewed adapter forget body performs a general purge of that database or built-in MEMORY.md/USER.md. Prompt/tool-result copies and independent source transcripts remain separate; source operation bodies below establish the scope of each plugin mutation.

<details>
<summary>Deletion, invalidation, archive and reset body ledger</summary>

Public routes, SDK/CLI dispatch and internal storage operations are distinguished by their source. Cache invalidation and management-object deletion are not additional fact-forget tools. A stub is not working deletion. Call-through means the downstream body still determines erasure. HTTP prefixes/selectors must be read with route registration.

**delete**, body `sources/hindsight/hindsight-all/hindsight/api_namespaces.py:40`; implementation/call-through.

- `sources/hindsight/hindsight-all/hindsight/api_namespaces.py:38` `def delete(self, bank_id: str):`
- `sources/hindsight/hindsight-all/hindsight/api_namespaces.py:41` `return self._embedded._client.delete_bank(bank_id=bank_id)`

**delete**, body `sources/hindsight/hindsight-all/hindsight/api_namespaces.py:115`; implementation/call-through.

- `sources/hindsight/hindsight-all/hindsight/api_namespaces.py:113` `def delete(self, bank_id: str, mental_model_id: str):`
- `sources/hindsight/hindsight-all/hindsight/api_namespaces.py:116` `return self._embedded._client.delete_mental_model(bank_id=bank_id, mental_model_id=mental_model_id)`

**delete**, body `sources/hindsight/hindsight-all/hindsight/api_namespaces.py:171`; implementation/call-through.

- `sources/hindsight/hindsight-all/hindsight/api_namespaces.py:169` `def delete(self, bank_id: str, directive_id: str):`
- `sources/hindsight/hindsight-all/hindsight/api_namespaces.py:172` `return self._embedded._client.delete_directive(bank_id=bank_id, directive_id=directive_id)`

**delete**, body `sources/hindsight/hindsight-all/hindsight/client_wrapper.py:58`; implementation/call-through.

- `sources/hindsight/hindsight-all/hindsight/client_wrapper.py:49` `def delete(self, bank_id: str) -> Any:`
- `sources/hindsight/hindsight-all/hindsight/client_wrapper.py:58` `return self._client.delete_bank(bank_id=bank_id)`

**delete**, body `sources/hindsight/hindsight-all/hindsight/client_wrapper.py:213`; implementation/call-through.

- `sources/hindsight/hindsight-all/hindsight/client_wrapper.py:203` `def delete(self, bank_id: str, mental_model_id: str) -> Any:`
- `sources/hindsight/hindsight-all/hindsight/client_wrapper.py:213` `return self._client.delete_mental_model(bank_id=bank_id, mental_model_id=mental_model_id)`

**delete**, body `sources/hindsight/hindsight-all/hindsight/client_wrapper.py:312`; implementation/call-through.

- `sources/hindsight/hindsight-all/hindsight/client_wrapper.py:302` `def delete(self, bank_id: str, directive_id: str) -> Any:`
- `sources/hindsight/hindsight-all/hindsight/client_wrapper.py:312` `return self._client.delete_directive(bank_id=bank_id, directive_id=directive_id)`

**api_update_memory**, body `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:5323`; implementation/call-through.

- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:5305` `@app.patch(`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:5306` `"/v1/default/banks/{bank_id}/memories/{memory_id}",`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:5307` `summary="Curate memory unit",`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:5308` `description="Edit a memory's text and/or change its curation state "`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:5309` `"(invalidate / revert). Invalidated memories are excluded from recall, "`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:5310` `"consolidation, and graph maintenance but kept for audit (reversible). "`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:5311` `"Only world/experience facts can be curated; observations are derived.",`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:5312` `operation_id="update_memory",`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:5313` `tags=["Memory"],`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:5314` `)`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:5315` `@audited("update_memory")`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:5316` `async def api_update_memory(`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:5323` `try:`

**api_update_mental_model**, body `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:6503`; implementation/call-through.

- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:6487` `@app.patch(`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:6488` `"/v1/default/banks/{bank_id}/mental-models/{mental_model_id}",`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:6489` `response_model=MentalModelResponse,`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:6490` `summary="Update mental model",`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:6491` `description="Update a mental model's name and/or source query.",`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:6492` `operation_id="update_mental_model",`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:6493` `tags=["Mental Models"],`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:6494` `)`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:6495` `@audited("update_mental_model", request_param="body")`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:6496` `async def api_update_mental_model(`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:6503` `try:`

**api_delete_mental_model**, body `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:6544`; implementation/call-through.

- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:6530` `@app.delete(`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:6531` `"/v1/default/banks/{bank_id}/mental-models/{mental_model_id}",`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:6532` `summary="Delete mental model",`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:6533` `description="Delete a mental model.",`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:6534` `operation_id="delete_mental_model",`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:6535` `tags=["Mental Models"],`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:6536` `)`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:6537` `@audited("delete_mental_model", request_param=None)`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:6538` `async def api_delete_mental_model(`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:6545` `deleted = await app.state.memory.delete_mental_model(`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:6552` `return {"status": "deleted"}`

**api_update_knowledge_node**, body `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:6807`; implementation/call-through.

- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:6788` `@app.patch(`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:6789` `"/v1/default/banks/{bank_id}/knowledge-base/nodes/{node_id}",`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:6790` `response_model=KnowledgeNode,`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:6791` `summary="Rename/move a knowledge-base node or update a page's options",`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:6792` `description="Rename a node (set 'name'), move it under another folder (set 'parent_id', null "`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:6793` `"for the root), and/or update a page's options ('source_query', 'tags', 'max_tokens', 'trigger'). "`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:6794` `"Changing 'source_query' schedules an async refresh so the page rebuilds against the new question. "`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:6795` `"'trigger' is applied as a patch: the fields you send are updated and the rest keep the page's "`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:6796` `"current values.",`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:6797` `operation_id="update_knowledge_node",`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:6798` `tags=["Knowledge Base"],`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:6799` `)`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:6800` `async def api_update_knowledge_node(`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:6807` `try:`

**api_delete_knowledge_node**, body `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:6870`; implementation/call-through.

- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:6857` `@app.delete(`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:6858` `"/v1/default/banks/{bank_id}/knowledge-base/nodes/{node_id}",`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:6859` `summary="Delete a knowledge-base node",`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:6860` `description="Delete a folder or page and its whole subtree (pages' mental models are removed too).",`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:6861` `operation_id="delete_knowledge_node",`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:6862` `tags=["Knowledge Base"],`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:6863` `)`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:6864` `async def api_delete_knowledge_node(`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:6871` `deleted = await app.state.memory.delete_knowledge_node(`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:6876` `return {"status": "deleted"}`

**api_update_directive**, body `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:7020`; implementation/call-through.

- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:7004` `@app.patch(`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:7005` `"/v1/default/banks/{bank_id}/directives/{directive_id}",`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:7006` `response_model=DirectiveResponse,`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:7007` `summary="Update directive",`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:7008` `description="Update a directive's properties.",`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:7009` `operation_id="update_directive",`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:7010` `tags=["Directives"],`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:7011` `)`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:7012` `@audited("update_directive", request_param="body")`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:7013` `async def api_update_directive(`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:7020` `try:`

**api_delete_directive**, body `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:7057`; implementation/call-through.

- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:7043` `@app.delete(`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:7044` `"/v1/default/banks/{bank_id}/directives/{directive_id}",`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:7045` `summary="Delete directive",`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:7046` `description="Delete a directive.",`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:7047` `operation_id="delete_directive",`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:7048` `tags=["Directives"],`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:7049` `)`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:7050` `@audited("delete_directive", request_param=None)`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:7051` `async def api_delete_directive(`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:7058` `deleted = await app.state.memory.delete_directive(`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:7065` `return {"status": "deleted"}`

**api_update_document**, body `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:7475`; implementation/call-through.

- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:7437` `@app.patch(`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:7438` `"/v1/default/banks/{bank_id}/documents/{document_id:path}",`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:7439` `response_model=UpdateDocumentResponse,`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:7440` `summary="Update document",`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:7441` `description="Update mutable fields on a document without re-processing its content.\n\n"`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:7442` `"**Tags** ('tags'): The array REPLACES the document's tags, it is not merged into them , "`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:7443` `"send the complete set you want the document to end up with, and any tag you leave out is "`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:7444` `"dropped. An empty array ('[]') therefore clears every tag; only omitting the field "`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:7445` `"entirely is rejected (422).\n\n"`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:7446` `"The new tags are propagated to all associated memory units.\n\n"`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:7447` `"**Observation scopes** ('observation_scopes'): Replaces how the document's memories are "`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:7448` `"scoped during consolidation, with the same values retain accepts. Use it to move a bank "`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:7449` `"whose documents were retained under per-session tags to 'shared' without re-extracting "`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:7450` `"anything. It is also what a later reprocess of the document retains with.\n\n"`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:7451` `"A memory is re-consolidated only when the change moves the observation scopes it resolves "`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:7452` `"to: its observations are deleted and it is queued again, along with co-source memories from "`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:7453` `"other documents that shared those observations. A change that resolves to the same scopes "`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:7454` `"(a tag change under 'shared', 'combined' over one tag vs '[[that tag]]', re-sending the "`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:7455` `"current values) keeps every observation and queues nothing.\n\n"`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:7456` `"At least one field must be provided.",`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:7457` `operation_id="update_document",`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:7458` `tags=["Documents"],`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:7459` `)`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:7460` `@audited("update_document", request_param="body")`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:7461` `async def api_update_document(`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:7475` `if body.tags is None and body.observation_scopes is None:`

**api_delete_document**, body `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:7521`; implementation/call-through.

- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:7497` `@app.delete(`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:7498` `"/v1/default/banks/{bank_id}/documents/{document_id:path}",`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:7499` `response_model=DeleteDocumentResponse,`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:7500` `summary="Delete a document",`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:7501` `description="Delete a document and all its associated memory units and links.\n\n"`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:7502` `"This will cascade delete:\n"`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:7503` `"- The document itself\n"`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:7504` `"- All memory units extracted from this document\n"`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:7505` `"- All links (temporal, semantic, entity) associated with those memory units\n\n"`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:7506` `"This operation cannot be undone.",`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:7507` `operation_id="delete_document",`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:7508` `tags=["Documents"],`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:7509` `)`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:7510` `@audited("delete_document", request_param=None)`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:7511` `async def api_delete_document(`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:7522` `result = await app.state.memory.delete_document(document_id, bank_id, request_context=request_context)`

**api_cancel_operation**, body `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:7645`; implementation/call-through.

- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:7626` `@app.delete(`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:7627` `"/v1/default/banks/{bank_id}/operations/{operation_id}",`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:7628` `response_model=CancelOperationResponse,`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:7629` `summary="Cancel a pending or in-flight async operation",`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:7630` `description=(`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:7631` `"Cancel a queued or running async operation. A 'pending' operation is never started. "`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:7632` `"A 'processing' one is cancelled cooperatively: the row is marked 'cancelled' immediately "`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:7633` `"and the worker running it stops at its next checkpoint, so work already in flight may "`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:7634` `"finish the batch it is on. This also clears operations stranded in 'processing' by a "`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:7635` `"crashed worker. Returns 409 for operations that already reached a terminal state."`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:7636` `),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:7637` `operation_id="cancel_operation",`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:7638` `tags=["Operations"],`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:7639` `)`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:7640` `@audited("cancel_operation", request_param=None)`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:7641` `async def api_cancel_operation(`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:7645` `try:`

**api_delete_operation**, body `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:7709`; implementation/call-through.

- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:7696` `@app.delete(`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:7697` `"/v1/default/banks/{bank_id}/operations/{operation_id}/delete",`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:7698` `response_model=DeleteOperationResponse,`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:7699` `summary="Delete a terminal async operation",`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:7700` `description="Permanently remove a failed, cancelled, or completed async operation record",`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:7701` `operation_id="delete_operation",`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:7702` `tags=["Operations"],`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:7703` `)`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:7704` `@audited("delete_operation", request_param=None)`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:7705` `async def api_delete_operation(`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:7715` `result = await app.state.memory.delete_operation(bank_id, operation_id, request_context=request_context)`

**api_set_bank_alias_primary**, body `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:7885`; implementation/call-through.

- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:7863` `@app.patch(`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:7864` `"/v1/default/banks/{bank_id}/aliases/{alias}",`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:7865` `response_model=BankAliasesResponse,`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:7866` `summary="Show this alias in place of the bank id",`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:7867` `description=(`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:7868` `"Present the bank under one of its aliases. Purely cosmetic: the bank keeps its own "`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:7869` `"'bank_id', which every other part of the system , authorisation, metering, exports, "`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:7870` `"audit logs , continues to use.\n\n"`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:7871` `"Promoting an alias demotes whichever one was shown before, so a bank is presented "`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:7872` `"under at most one alias. Send 'primary: false' to go back to showing its own id."`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:7873` `),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:7874` `operation_id="set_bank_alias_primary",`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:7875` `tags=["Banks"],`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:7876` `)`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:7877` `@audited("set_bank_alias_primary")`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:7878` `async def api_set_bank_alias_primary(`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:7885` `try:`

**api_delete_bank_alias**, body `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:7915`; implementation/call-through.

- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:7898` `@app.delete(`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:7899` `"/v1/default/banks/{bank_id}/aliases/{alias}",`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:7900` `response_model=BankAliasesResponse,`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:7901` `summary="Remove an alias from the bank",`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:7902` `description=(`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:7903` `"Stop an id reaching this bank. The bank and its memories are untouched; only the extra "`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:7904` `"name goes away, and callers still using it get the same 404 (or new empty bank) they "`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:7905` `"would have got before it existed."`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:7906` `),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:7907` `operation_id="delete_bank_alias",`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:7908` `tags=["Banks"],`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:7909` `)`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:7910` `@audited("delete_bank_alias")`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:7911` `async def api_delete_bank_alias(`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:7916` `if not await app.state.memory.delete_bank_alias(bank_id, alias, request_context=request_context):`

**api_update_bank**, body `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:7984`; implementation/call-through.

- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:7971` `@app.patch(`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:7972` `"/v1/default/banks/{bank_id}",`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:7973` `response_model=BankProfileResponse,`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:7974` `summary="Partial update memory bank",`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:7975` `description="Partially update an agent's profile. Only provided fields will be updated.",`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:7976` `operation_id="update_bank",`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:7977` `tags=["Banks"],`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:7978` `)`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:7979` `@audited("update_bank")`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:7980` `async def api_update_bank(`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:7984` `try:`

**api_delete_bank**, body `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:8029`; implementation/call-through.

- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:8017` `@app.delete(`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:8018` `"/v1/default/banks/{bank_id}",`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:8019` `response_model=DeleteResponse,`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:8020` `summary="Delete memory bank",`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:8021` `description="Delete an entire memory bank including all memories, entities, documents, and the bank profile itself. "`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:8022` `"This is a destructive operation that cannot be undone.",`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:8023` `operation_id="delete_bank",`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:8024` `tags=["Banks"],`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:8025` `)`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:8026` `@audited("delete_bank", request_param=None)`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:8027` `async def api_delete_bank(bank_id: str, request_context: RequestContext = Depends(get_request_context)):`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:8030` `result = await app.state.memory.delete_bank(bank_id, request_context=request_context)`

**api_clear_observations**, body `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:8813`; implementation/call-through.

- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:8802` `@app.delete(`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:8803` `"/v1/default/banks/{bank_id}/observations",`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:8804` `response_model=DeleteResponse,`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:8805` `summary="Clear all observations",`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:8806` `description="Delete all observations for a memory bank. This is useful for resetting the consolidated knowledge.",`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:8807` `operation_id="clear_observations",`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:8808` `tags=["Banks"],`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:8809` `)`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:8810` `@audited("clear_observations", request_param=None)`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:8811` `async def api_clear_observations(bank_id: str, request_context: RequestContext = Depends(get_request_context)):`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:8813` `try:`

**api_clear_memory_observations**, body `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:8938`; implementation/call-through.

- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:8921` `@app.delete(`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:8922` `"/v1/default/banks/{bank_id}/memories/{memory_id}/observations",`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:8923` `response_model=ClearMemoryObservationsResponse,`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:8924` `summary="Clear observations for a memory",`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:8925` `description="Delete all observations derived from a specific memory and reset it for re-consolidation. "`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:8926` `"The memory itself is not deleted. A consolidation job is triggered automatically so the memory "`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:8927` `"will produce fresh observations on the next consolidation run.",`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:8928` `operation_id="clear_memory_observations",`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:8929` `tags=["Memory"],`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:8930` `)`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:8931` `@audited("clear_memory_observations", request_param=None)`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:8932` `async def api_clear_memory_observations(`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:8938` `try:`

**api_update_bank_config**, body `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:8998`; implementation/call-through.

- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:8983` `@app.patch(`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:8984` `"/v1/default/banks/{bank_id}/config",`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:8985` `response_model=BankConfigResponse,`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:8986` `summary="Update bank configuration",`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:8987` `description="Update configuration overrides for a bank. Only hierarchical behavioral settings can be "`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:8988` `"overridden (retention parameters, recall settings, etc.). Keys can be provided in Python field format "`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:8989` `"(retain_extraction_mode) or environment variable format (HINDSIGHT_API_RETAIN_EXTRACTION_MODE).",`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:8990` `operation_id="update_bank_config",`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:8991` `tags=["Banks"],`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:8992` `)`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:8993` `@audited("update_bank_config")`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:8994` `async def api_update_bank_config(`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:8998` `if not get_config().enable_bank_config_api:`

**api_reset_bank_config**, body `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:9032`; implementation/call-through.

- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:9020` `@app.delete(`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:9021` `"/v1/default/banks/{bank_id}/config",`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:9022` `response_model=BankConfigResponse,`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:9023` `summary="Reset bank configuration",`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:9024` `description="Reset bank configuration to defaults by removing all bank-specific overrides. "`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:9025` `"The bank will then use global and tenant-level configuration only.",`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:9026` `operation_id="reset_bank_config",`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:9027` `tags=["Banks"],`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:9028` `)`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:9029` `@audited("reset_bank_config", request_param=None)`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:9030` `async def api_reset_bank_config(bank_id: str, request_context: RequestContext = Depends(get_request_context)):`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:9032` `if not get_config().enable_bank_config_api:`

**api_delete_webhook**, body `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:9249`; implementation/call-through.

- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:9234` `@app.delete(`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:9235` `"/v1/default/banks/{bank_id}/webhooks/{webhook_id}",`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:9236` `response_model=DeleteResponse,`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:9237` `summary="Delete webhook",`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:9238` `description="Remove a registered webhook.",`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:9239` `operation_id="delete_webhook",`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:9240` `tags=["Webhooks"],`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:9241` `)`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:9242` `@audited("delete_webhook", request_param=None)`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:9243` `async def api_delete_webhook(`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:9250` `deleted = await app.state.memory.delete_webhook(`

**api_update_webhook**, body `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:9281`; implementation/call-through.

- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:9265` `@app.patch(`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:9266` `"/v1/default/banks/{bank_id}/webhooks/{webhook_id}",`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:9267` `response_model=WebhookResponse,`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:9268` `summary="Update webhook",`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:9269` `description="Update one or more fields of a registered webhook. Only provided fields are changed.",`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:9270` `operation_id="update_webhook",`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:9271` `tags=["Webhooks"],`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:9272` `)`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:9273` `@audited("update_webhook")`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:9274` `async def api_update_webhook(`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:9281` `try:`

**api_clear_bank_memories**, body `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:9881`; implementation/call-through.

- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:9866` `@app.delete(`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:9867` `"/v1/default/banks/{bank_id}/memories",`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:9868` `response_model=DeleteResponse,`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:9869` `summary="Clear memory bank memories",`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:9870` `description="Delete memory units for a memory bank. Optionally filter by type (world, experience, observation) to delete only specific types. This is a destructive operation that cannot be undone. The bank profile (disposition and background) will be preserved.",`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:9871` `operation_id="clear_bank_memories",`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:9872` `tags=["Memory"],`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:9873` `)`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:9874` `@audited("clear_memories", request_param=None)`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:9875` `async def api_clear_bank_memories(`
- `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:9882` `result = await app.state.memory.delete_bank(`

**log_fire_and_forget**, body `sources/hindsight/hindsight-api-slim/hindsight_api/engine/audit.py:259`; implementation/call-through.

- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/audit.py:253` `def log_fire_and_forget(self, entry: AuditEntry) -> None:`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/audit.py:259` `if not self.action_allowed(entry.action):`

**invalidate**, body `sources/hindsight/hindsight-api-slim/hindsight_api/engine/bank_aliases.py:135`; implementation/call-through.

- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/bank_aliases.py:130` `async def invalidate(alias: str) -> None:`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/bank_aliases.py:136` `await _get_cache().invalidate(schema, key)`

**delete_alias**, body `sources/hindsight/hindsight-api-slim/hindsight_api/engine/bank_aliases.py:278`; implementation/call-through.

- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/bank_aliases.py:272` `async def delete_alias(pool, bank_id: str, alias: str) -> bool:`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/bank_aliases.py:280` `f"DELETE FROM {fq_table('bank_aliases')} WHERE alias = $1 AND bank_id = $2 RETURNING alias",`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/bank_aliases.py:284` `await invalidate(alias)`

**invalidate**, body `sources/hindsight/hindsight-api-slim/hindsight_api/engine/bank_info_cache.py:107`; implementation/call-through.

- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/bank_info_cache.py:100` `async def invalidate(bank_id: str, kind: str | None = None) -> None:`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/bank_info_cache.py:110` `await cache.invalidate(schema, key)`

**invalidate**, body `sources/hindsight/hindsight-api-slim/hindsight_api/engine/bank_stats_cache.py:162`; implementation/call-through.

- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/bank_stats_cache.py:160` `async def invalidate(self, schema: str, bank_id: str) -> None:`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/bank_stats_cache.py:166` `# snapshot they requested, while post-invalidation callers reload.`

**clear**, body `sources/hindsight/hindsight-api-slim/hindsight_api/engine/bank_stats_cache.py:172`; implementation/call-through.

- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/bank_stats_cache.py:171` `async def clear(self) -> None:`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/bank_stats_cache.py:172` `with self._lock:`

**invalidate**, body `sources/hindsight/hindsight-api-slim/hindsight_api/engine/bank_stats_cache.py:260`; implementation/call-through.

- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/bank_stats_cache.py:258` `async def invalidate(self, schema: str, bank_id: str) -> None:`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/bank_stats_cache.py:264` `await conn.execute(f"DELETE FROM {self._qualified(schema)} WHERE bank_id = $1", bank_id)`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/bank_stats_cache.py:265` `except Exception as exc:  # noqa: BLE001 , invalidation must never break the write path`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/bank_stats_cache.py:266` `logger.debug("bank_stats_cache invalidate failed for %s.%s (%s)", schema, bank_id, exc)`

**_execute_delete_action**, body `sources/hindsight/hindsight-api-slim/hindsight_api/engine/consolidation/consolidator.py:3547`; implementation/call-through.

- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/consolidation/consolidator.py:3541` `async def _execute_delete_action(`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/consolidation/consolidator.py:3553` `await get_memories().delete_observation(`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/consolidation/consolidator.py:3561` `f"DELETE FROM {fq_table('observation_history')} WHERE bank_id = $1 AND observation_id = $2",`

**_fold_deleted_evidence**, body `sources/hindsight/hindsight-api-slim/hindsight_api/engine/consolidation/consolidator.py:3586`; implementation/call-through.

- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/consolidation/consolidator.py:3568` `async def _fold_deleted_evidence(`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/consolidation/consolidator.py:3586` `live = await _filter_live_source_memories(conn, bank_id, [uuid.UUID(str(sid)) for sid in gone.source_memory_ids])`

**delete_unit_links**, body `sources/hindsight/hindsight-api-slim/hindsight_api/engine/db/ops.py:450`; stub.

- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/db/ops.py:432` `@abstractmethod`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/db/ops.py:433` `async def delete_unit_links(`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/db/ops.py:450` `...`

**delete_webhook**, body `sources/hindsight/hindsight-api-slim/hindsight_api/engine/db/ops.py:767`; stub.

- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/db/ops.py:758` `@abstractmethod`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/db/ops.py:759` `async def delete_webhook(`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/db/ops.py:767` `...`

**delete_unit_links**, body `sources/hindsight/hindsight-api-slim/hindsight_api/engine/db/ops_oracle.py:206`; implementation/call-through.

- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/db/ops_oracle.py:198` `async def delete_unit_links(`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/db/ops_oracle.py:211` `f"DELETE FROM {table} WHERE bank_id = $2 "`

**delete_webhook**, body `sources/hindsight/hindsight-api-slim/hindsight_api/engine/db/ops_oracle.py:1093`; implementation/call-through.

- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/db/ops_oracle.py:1092` `async def delete_webhook(self, conn, table, webhook_id, bank_id):`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/db/ops_oracle.py:1094` `f"DELETE FROM {table} WHERE id = $1 AND bank_id = $2",`

**delete_unit_links**, body `sources/hindsight/hindsight-api-slim/hindsight_api/engine/db/ops_postgresql.py:278`; implementation/call-through.

- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/db/ops_postgresql.py:270` `async def delete_unit_links(`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/db/ops_postgresql.py:307` `DELETE FROM {table} ml`

**delete_webhook**, body `sources/hindsight/hindsight-api-slim/hindsight_api/engine/db/ops_postgresql.py:1446`; implementation/call-through.

- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/db/ops_postgresql.py:1445` `async def delete_webhook(self, conn, table, webhook_id, bank_id):`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/db/ops_postgresql.py:1447` `f"DELETE FROM {table} WHERE id = $1 AND bank_id = $2",`

**invalidate**, body `sources/hindsight/hindsight-api-slim/hindsight_api/engine/db/optional_routines.py:187`; implementation/call-through.

- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/db/optional_routines.py:182` `def invalidate(self, routine_name: str | None = None) -> None:`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/db/optional_routines.py:187` `if routine_name is None:`

**deletion_events**, body `sources/hindsight/hindsight-api-slim/hindsight_api/engine/document_history.py:144`; implementation/call-through.

- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/document_history.py:141` `def deletion_events(deleted: dict[str, str | None], changes: ObservationChanges) -> list[DocumentEvent]:`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/document_history.py:155` `obs_id for obs_id, sources in changes.removed.items() if facts.intersection(sources)`

**delete_bank**, body `sources/hindsight/hindsight-api-slim/hindsight_api/engine/interface.py:334`; stub.

- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/interface.py:312` `@abstractmethod`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/interface.py:313` `async def delete_bank(`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/interface.py:334` `...`

**delete_document**, body `sources/hindsight/hindsight-api-slim/hindsight_api/engine/interface.py:483`; stub.

- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/interface.py:464` `@abstractmethod`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/interface.py:465` `async def delete_document(`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/interface.py:483` `...`

**delete_operation**, body `sources/hindsight/hindsight-api-slim/hindsight_api/engine/interface.py:692`; stub.

- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/interface.py:670` `@abstractmethod`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/interface.py:671` `async def delete_operation(`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/interface.py:692` `...`

**_record_fire_and_forget**, body `sources/hindsight/hindsight-api-slim/hindsight_api/engine/llm_trace.py:868`; implementation/call-through.

- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/llm_trace.py:866` `def _record_fire_and_forget(self, record: LLMRequestRecord) -> None:`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/llm_trace.py:868` `entry = _BufferedRow(self._schema_getter(), record)`

**_purge_expired**, body `sources/hindsight/hindsight-api-slim/hindsight_api/engine/maintenance.py:304`; implementation/call-through.

- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/maintenance.py:297` `async def _purge_expired(self, table: str, ts_col: str, days: int) -> int:`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/maintenance.py:304` `backend = self._engine._backend`

**_purge_table_in_batches**, body `sources/hindsight/hindsight-api-slim/hindsight_api/engine/maintenance.py:342`; implementation/call-through.

- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/maintenance.py:332` `async def _purge_table_in_batches(self, table: str, ts_col: str, cutoff: datetime) -> int:`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/maintenance.py:359` `DELETE FROM {table} t USING expired e WHERE t.id = e.id RETURNING 1`

**delete_knowledge_pages**, body `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/base.py:1279`; stub.

- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/base.py:1276` `async def delete_knowledge_pages(self, bank_id: str, page_ids: list[str]) -> None:`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/base.py:1279` `raise NotImplementedError("this store does not index knowledge pages")`

**delete_facts**, body `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/base.py:1479`; implementation/call-through.

- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/base.py:1478` `@abstractmethod`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/base.py:1479` `async def delete_facts(self, bank_id: str, unit_ids: list[str]) -> None:`

**delete_where**, body `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/base.py:1488`; stub.

- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/base.py:1482` `async def delete_where(self, bank_id: str, predicate: DeletePredicate) -> int:`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/base.py:1488` `raise NotImplementedError`

**delete_document**, body `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/base.py:1491`; implementation/call-through.

- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/base.py:1490` `@abstractmethod`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/base.py:1491` `async def delete_document(self, *, conn, fq_table, bank_id: str, document_id: str) -> None:`

**delete_document_record**, body `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/base.py:1826`; stub.

- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/base.py:1821` `async def delete_document_record(self, *, bank_id: str, document_id: str) -> None:`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/base.py:1826` `raise NotImplementedError`

**delete_observations**, body `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/base.py:1836`; stub.

- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/base.py:1834` `async def delete_observations(self, *, conn, fq_table, bank_id: str) -> None:`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/base.py:1836` `raise NotImplementedError`

**get_archived_memory**, body `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/base.py:2458`; implementation/call-through.

- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/base.py:2457` `@abstractmethod`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/base.py:2458` `async def get_archived_memory(self, *, conn, fq_table, bank_id: str, unit_id: str) -> StoredMemory | None:`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/base.py:2459` `"""An *invalidated* memory read from the archive, or ''None''.`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/base.py:2461` `Only invalidated memories are in the archive, so a live or missing id`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/base.py:2462` `returns ''None'' , which is how a caller tells "invalidated" from "live"`

**invalidate_memory**, body `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/base.py:2467`; implementation/call-through.

- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/base.py:2466` `@abstractmethod`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/base.py:2467` `async def invalidate_memory(self, *, conn, fq_table, bank_id: str, unit_id: str, reason: str | None) -> bool:`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/base.py:2468` `"""Move a live memory into the archive, out of every recall surface.`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/base.py:2470` `Returns ''True'' if it was live and is now archived, ''False'' if there was`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/base.py:2472` `:meth:'get_archived_memory' and restorable via :meth:'restore_memory';`

**set_invalidation_reason**, body `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/base.py:2477`; implementation/call-through.

- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/base.py:2476` `@abstractmethod`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/base.py:2477` `async def set_invalidation_reason(self, *, conn, fq_table, bank_id: str, unit_id: str, reason: str | None) -> None:`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/base.py:2478` `"""Update the recorded reason on a memory that is already archived."""`

**delete_stale_observations**, body `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/base.py:2715`; implementation/call-through.

- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/base.py:2714` `@abstractmethod`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/base.py:2715` `async def delete_stale_observations(self, *, conn, ops, fq_table, bank_id: str, fact_ids: list) -> int:`

**delete_document_rows**, body `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/base.py:3069`; implementation/call-through.

- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/base.py:3042` `async def delete_document_rows(`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/base.py:3071` `await self.delete_document(conn=conn, fq_table=fq_table, bank_id=bank_id, document_id=document_id)`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/base.py:3075` `await self.delete_document_record(bank_id=bank_id, document_id=document_id)`

**delete_memory**, body `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/base.py:3311`; implementation/call-through.

- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/base.py:3305` `async def delete_memory(self, *, conn, ops, fq_table, bank_id: str | None, unit_id: str) -> str | None:`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/base.py:3313` `await self.delete_facts(bank_id, [unit_id])`

**delete_memories**, body `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/base.py:3321`; implementation/call-through.

- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/base.py:3316` `async def delete_memories(self, *, conn, ops, fq_table, bank_id: str, unit_ids: list[str]) -> int:`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/base.py:3321` `await self.delete_facts(bank_id, unit_ids)`

**delete_bank_memories_of_type**, body `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/base.py:3338`; implementation/call-through.

- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/base.py:3338` `async def delete_bank_memories_of_type(`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/base.py:3341` `"""Delete a bank's live and archived memories of one fact_type, inside the caller's`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/base.py:3346` `in lock order) and the matching archive rows."""`

**purge_bank_rows**, body `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/base.py:3370`; implementation/call-through.

- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/base.py:3359` `async def purge_bank_rows(self, *, conn, fq_table, bank_id: str) -> list[str]:`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/base.py:3378` `await conn.execute(f"DELETE FROM {fq_table('attachments')} WHERE bank_id = $1", bank_id)`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/base.py:3379` `await conn.execute(f"DELETE FROM {fq_table('observation_history')} WHERE bank_id = $1", bank_id)`

**delete_document_for_replace**, body `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/base.py:3635`; implementation/call-through.

- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/base.py:3629` `async def delete_document_for_replace(`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/base.py:3635` `await self.delete_document(conn=conn, fq_table=fq_table, bank_id=bank_id, document_id=document_id)`

**delete_chunks**, body `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/base.py:3747`; implementation/call-through.

- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/base.py:3743` `async def delete_chunks(self, *, conn, fq_table, bank_id: str, chunk_ids: list[str]) -> None:`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/base.py:3748` `await self.delete_where(bank_id, DeletePredicate(metadata_equals={META_CHUNK_ID: chunk_id}))`

**delete_observation**, body `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/base.py:4132`; implementation/call-through.

- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/base.py:4130` `async def delete_observation(self, *, conn, fq_table, bank_id: str, observation_id: str) -> None:`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/base.py:4132` `await self.delete_facts(bank_id, [observation_id])`

**iter_archived_memories**, body `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/base.py:4306`; implementation/call-through.

- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/base.py:4294` `async def iter_archived_memories(`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/base.py:4306` `raise NotImplementedError("this store cannot list its archived memories")`

**restore_archived_memories**, body `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/base.py:4327`; stub.

- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/base.py:4309` `async def restore_archived_memories(`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/base.py:4327` `raise NotImplementedError("this store cannot restore archived memories")`

**delete_observation**, body `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/pg/consolidation.py:379`; implementation/call-through.

- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/pg/consolidation.py:377` `async def delete_observation(*, conn, fq_table: Callable[[str], str], bank_id: str, observation_id: str) -> None:`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/pg/consolidation.py:380` `f"DELETE FROM {fq_table('memory_units')} WHERE id = $1 AND bank_id = $2 AND fact_type = 'observation'",`

**delete_document_rows**, body `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/pg/documents.py:327`; implementation/call-through.

- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/pg/documents.py:321` `async def delete_document_rows(`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/pg/documents.py:334` `await ops.delete_unit_links(conn, fq_table("memory_links"), bank_id, unit_ids)`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/pg/documents.py:342` `f"DELETE FROM {fq_table('documents')} WHERE id = $1 AND bank_id = $2 RETURNING id",`

**delete_memory**, body `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/pg/engine_curation.py:50`; implementation/call-through.

- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/pg/engine_curation.py:47` `async def delete_memory(*, conn, ops, fq_table: Callable[[str], str], bank_id: str | None, unit_id: str) -> str | None:`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/pg/engine_curation.py:51` `await ops.delete_unit_links(conn, fq_table("memory_links"), bank_id, [unit_id])`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/pg/engine_curation.py:52` `deleted = await conn.fetchval(f"DELETE FROM {fq_table('memory_units')} WHERE id = $1 RETURNING id", unit_id)`

**delete_memories**, body `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/pg/engine_curation.py:59`; implementation/call-through.

- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/pg/engine_curation.py:56` `async def delete_memories(*, conn, ops, fq_table: Callable[[str], str], bank_id: str, unit_ids: list[str]) -> int:`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/pg/engine_curation.py:59` `await ops.delete_unit_links(conn, fq_table("memory_links"), bank_id, unit_ids)`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/pg/engine_curation.py:66` `f"DELETE FROM {fq_table('memory_units')} WHERE id = ANY($1::uuid[])",`

**delete_bank_memories_of_type**, body `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/pg/engine_curation.py:104`; implementation/call-through.

- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/pg/engine_curation.py:99` `async def delete_bank_memories_of_type(`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/pg/engine_curation.py:104` `await ops.delete_unit_links(conn, fq_table("memory_links"), bank_id, unit_ids)`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/pg/engine_curation.py:106` `f"DELETE FROM {fq_table('memory_units')} WHERE bank_id = $1 AND fact_type = $2",`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/pg/engine_curation.py:110` `# Curation archive holds invalidated facts of the same types.`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/pg/engine_curation.py:112` `f"DELETE FROM {fq_table('invalidated_memory_units')} WHERE bank_id = $1 AND fact_type = $2",`

**purge_bank_rows**, body `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/pg/engine_curation.py:133`; implementation/call-through.

- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/pg/engine_curation.py:126` `async def purge_bank_rows(*, conn, fq_table: Callable[[str], str], bank_id: str) -> list[str]:`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/pg/engine_curation.py:146` `await conn.execute(f"DELETE FROM {fq_table('documents')} WHERE bank_id = $1", bank_id)`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/pg/engine_curation.py:149` `await conn.execute(f"DELETE FROM {fq_table('attachments')} WHERE bank_id = $1", bank_id)`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/pg/engine_curation.py:152` `await conn.execute(f"DELETE FROM {fq_table('memory_units')} WHERE bank_id = $1", bank_id)`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/pg/engine_curation.py:157` `await conn.execute(f"DELETE FROM {fq_table('observation_history')} WHERE bank_id = $1", bank_id)`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/pg/engine_curation.py:159` `# Curation archive (rows with NULL document_id aren't covered by`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/pg/engine_curation.py:161` `await conn.execute(f"DELETE FROM {fq_table('invalidated_memory_units')} WHERE bank_id = $1", bank_id)`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/pg/engine_curation.py:164` `await conn.execute(f"DELETE FROM {fq_table('entities')} WHERE bank_id = $1", bank_id)`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/pg/engine_curation.py:169` `await conn.execute(f"DELETE FROM {fq_table('graph_maintenance_queue')} WHERE bank_id = $1", bank_id)`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/pg/engine_curation.py:170` `await conn.execute(f"DELETE FROM {fq_table('entity_maintenance_queue')} WHERE bank_id = $1", bank_id)`

**delete_document_for_replace**, body `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/pg/retain.py:227`; implementation/call-through.

- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/pg/retain.py:217` `async def delete_document_for_replace(`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/pg/retain.py:233` `await ops.delete_unit_links(conn, fq_table("memory_links"), bank_id, outgoing_unit_ids)`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/pg/retain.py:240` `await writes.delete_document(conn=conn, fq_table=fq_table, bank_id=bank_id, document_id=document_id)`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/pg/retain.py:243` `f"DELETE FROM {fq_table('documents')} WHERE id = $1 AND bank_id = $2 RETURNING created_at",`

**delete_chunks**, body `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/pg/retain.py:405`; implementation/call-through.

- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/pg/retain.py:403` `async def delete_chunks(*, conn, fq_table: Callable[[str], str], bank_id: str, chunk_ids: list[str]) -> None:`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/pg/retain.py:408` `# OracleOps.delete_unit_links (see prune_stale_cooccurrences for that asymmetry).`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/pg/retain.py:412` `DELETE FROM {fq_table("memory_links")}`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/pg/retain.py:421` `f"DELETE FROM {fq_table('memory_units')} WHERE chunk_id = ANY($1::text[]) AND bank_id = $2",`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/pg/retain.py:426` `f"DELETE FROM {fq_table('chunks')} WHERE chunk_id = ANY($1::text[]) AND bank_id = $2",`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/pg/retain.py:477` `-- otherwise lock in either order (#6051). Same order as ops.delete_unit_links.`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/pg/retain.py:481` `DELETE FROM {fq_table("memory_links")} ml`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/pg/retain.py:498` `DELETE FROM {fq_table("chunks")} c`

**iter_archived_memories**, body `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/pg/transfer.py:376`; implementation/call-through.

- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/pg/transfer.py:364` `async def iter_archived_memories(`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/pg/transfer.py:378` `f"SELECT * FROM {fq_table('invalidated_memory_units')} WHERE bank_id = $1",`

**restore_archived_memories**, body `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/pg/transfer.py:490`; implementation/call-through.

- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/pg/transfer.py:471` `async def restore_archived_memories(`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/pg/transfer.py:514` `"invalidated_memory_units",`

**delete_document**, body `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/pg/writes.py:160`; implementation/call-through.

- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/pg/writes.py:146` `async def delete_document(*, conn, fq_table: Callable[[str], str], bank_id: str, document_id: str) -> None:`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/pg/writes.py:161` `f"DELETE FROM {fq_table('memory_units')} WHERE document_id = $1 AND bank_id = $2",`

**delete_observations**, body `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/pg/writes.py:174`; implementation/call-through.

- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/pg/writes.py:167` `async def delete_observations(*, conn, fq_table: Callable[[str], str], bank_id: str) -> None:`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/pg/writes.py:175` `f"DELETE FROM {fq_table('memory_units')} WHERE bank_id = $1 AND fact_type = 'observation'",`

**delete_stale_observations**, body `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/pg/writes.py:271`; implementation/call-through.

- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/pg/writes.py:243` `async def delete_stale_observations(`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/pg/writes.py:297` `# writes next and deadlock (#4251). The engine's delete, edit and invalidate paths, which`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/pg/writes.py:314` `f"DELETE FROM {fq_table('memory_units')} WHERE id = ANY($1::uuid[])",`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/pg/writes.py:322` `f"DELETE FROM {fq_table('observation_history')} WHERE bank_id = $1 AND observation_id = ANY($2::uuid[])",`

**_archive_columns**, body `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/pg/writes.py:379`; implementation/call-through.

- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/pg/writes.py:377` `async def _archive_columns(conn, fq_table: Callable[[str], str]) -> str:`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/pg/writes.py:380` `return ", ".join(c for c in (s.strip() for s in collist.split(",")) if c not in _ARCHIVE_OMITTED)`

**_archived_stored**, body `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/pg/writes.py:392`; implementation/call-through.

- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/pg/writes.py:390` `def _archived_stored(row: Any) -> StoredMemory:`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/pg/writes.py:392` `return StoredMemory(`

**get_archived_memory**, body `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/pg/writes.py:413`; implementation/call-through.

- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/pg/writes.py:412` `async def get_archived_memory(*, conn, fq_table, bank_id: str, unit_id: str) -> StoredMemory | None:`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/pg/writes.py:414` `f"SELECT {_ARCHIVE_SELECT} FROM {fq_table('invalidated_memory_units')} WHERE id = $1 AND bank_id = $2",`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/pg/writes.py:418` `return _archived_stored(row) if row else None`

**invalidate_memory**, body `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/pg/writes.py:422`; implementation/call-through.

- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/pg/writes.py:421` `async def invalidate_memory(*, conn, fq_table, bank_id: str, unit_id: str, reason: str | None) -> bool:`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/pg/writes.py:423` `arch = fq_table("invalidated_memory_units")`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/pg/writes.py:425` `arch_cols = await _archive_columns(conn, fq_table)`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/pg/writes.py:434` `# the archive row and revert rematerializes them (#2864).`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/pg/writes.py:439` `f"INSERT INTO {arch} ({arch_cols}, invalidation_reason, invalidated_at, entity_ids, causal_links) "`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/pg/writes.py:453` `await _ops_for(conn).delete_unit_links(conn, fq_table("memory_links"), bank_id, [str(unit_id)])`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/pg/writes.py:455` `await conn.execute(f"DELETE FROM {mu} WHERE id = $1 AND bank_id = $2", str(unit_id), bank_id)`

**set_invalidation_reason**, body `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/pg/writes.py:460`; implementation/call-through.

- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/pg/writes.py:459` `async def set_invalidation_reason(*, conn, fq_table, bank_id: str, unit_id: str, reason: str | None) -> None:`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/pg/writes.py:461` `f"UPDATE {fq_table('invalidated_memory_units')} SET invalidation_reason = $3, updated_at = now() "`

**delete_facts**, body `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/postgres.py:105`; implementation/call-through.

- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/postgres.py:105` `async def delete_facts(self, bank_id: str, unit_ids: list[str]) -> None:`

**delete_where**, body `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/postgres.py:110`; implementation/call-through.

- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/postgres.py:108` `async def delete_where(self, bank_id: str, predicate: DeletePredicate) -> int:`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/postgres.py:110` `return 0`

**delete_document**, body `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/postgres.py:113`; implementation/call-through.

- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/postgres.py:112` `async def delete_document(self, *, conn, fq_table, bank_id: str, document_id: str) -> None:`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/postgres.py:113` `await writes.delete_document(conn=conn, fq_table=fq_store_table, bank_id=bank_id, document_id=document_id)`

**delete_observations**, body `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/postgres.py:119`; implementation/call-through.

- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/postgres.py:118` `async def delete_observations(self, *, conn, fq_table, bank_id: str) -> None:`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/postgres.py:119` `await writes.delete_observations(conn=conn, fq_table=fq_store_table, bank_id=bank_id)`

**delete_stale_observations**, body `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/postgres.py:615`; implementation/call-through.

- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/postgres.py:614` `async def delete_stale_observations(self, *, conn, ops, fq_table, bank_id: str, fact_ids: list) -> int:`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/postgres.py:615` `return await writes.delete_stale_observations(`

**get_archived_memory**, body `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/postgres.py:674`; implementation/call-through.

- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/postgres.py:673` `async def get_archived_memory(self, *, conn, fq_table, bank_id: str, unit_id: str) -> StoredMemory | None:`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/postgres.py:674` `return await writes.get_archived_memory(conn=conn, fq_table=fq_store_table, bank_id=bank_id, unit_id=unit_id)`

**invalidate_memory**, body `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/postgres.py:677`; implementation/call-through.

- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/postgres.py:676` `async def invalidate_memory(self, *, conn, fq_table, bank_id: str, unit_id: str, reason: str | None) -> bool:`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/postgres.py:677` `return await writes.invalidate_memory(`

**set_invalidation_reason**, body `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/postgres.py:682`; implementation/call-through.

- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/postgres.py:681` `async def set_invalidation_reason(self, *, conn, fq_table, bank_id: str, unit_id: str, reason: str | None) -> None:`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/postgres.py:682` `await writes.set_invalidation_reason(`

**delete_document_rows**, body `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/postgres.py:953`; implementation/call-through.

- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/postgres.py:950` `async def delete_document_rows(`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/postgres.py:953` `return await documents.delete_document_rows(`

**delete_memory**, body `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/postgres.py:1069`; implementation/call-through.

- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/postgres.py:1068` `async def delete_memory(self, *, conn, ops, fq_table, bank_id: str | None, unit_id: str) -> str | None:`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/postgres.py:1069` `return await engine_curation.delete_memory(`

**delete_memories**, body `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/postgres.py:1074`; implementation/call-through.

- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/postgres.py:1073` `async def delete_memories(self, *, conn, ops, fq_table, bank_id: str, unit_ids: list[str]) -> int:`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/postgres.py:1074` `return await engine_curation.delete_memories(`

**delete_bank_memories_of_type**, body `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/postgres.py:1086`; implementation/call-through.

- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/postgres.py:1083` `async def delete_bank_memories_of_type(`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/postgres.py:1086` `await engine_curation.delete_bank_memories_of_type(`

**purge_bank_rows**, body `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/postgres.py:1094`; implementation/call-through.

- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/postgres.py:1093` `async def purge_bank_rows(self, *, conn, fq_table, bank_id: str) -> list[str]:`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/postgres.py:1094` `return await engine_curation.purge_bank_rows(conn=conn, fq_table=fq_store_table, bank_id=bank_id)`

**delete_document_for_replace**, body `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/postgres.py:1242`; implementation/call-through.

- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/postgres.py:1239` `async def delete_document_for_replace(`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/postgres.py:1242` `return await pg_retain.delete_document_for_replace(`

**delete_chunks**, body `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/postgres.py:1305`; implementation/call-through.

- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/postgres.py:1304` `async def delete_chunks(self, *, conn, fq_table, bank_id: str, chunk_ids: list[str]) -> None:`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/postgres.py:1305` `await pg_retain.delete_chunks(conn=conn, fq_table=fq_store_table, bank_id=bank_id, chunk_ids=chunk_ids)`

**delete_observation**, body `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/postgres.py:1617`; implementation/call-through.

- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/postgres.py:1616` `async def delete_observation(self, *, conn, fq_table, bank_id: str, observation_id: str) -> None:`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/postgres.py:1617` `await pg_consolidation.delete_observation(`

**iter_archived_memories**, body `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/postgres.py:1691`; implementation/call-through.

- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/postgres.py:1688` `async def iter_archived_memories(`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/postgres.py:1691` `async for page in pg_transfer.iter_archived_memories(`

**restore_archived_memories**, body `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/postgres.py:1707`; implementation/call-through.

- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/postgres.py:1696` `async def restore_archived_memories(`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/memories/postgres.py:1707` `return await pg_transfer.restore_archived_memories(`

**invalidate**, body `sources/hindsight/hindsight-api-slim/hindsight_api/engine/providers/gemini_cache.py:205`; implementation/call-through.

- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/providers/gemini_cache.py:196` `def invalidate(self, name: str) -> None:`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/providers/gemini_cache.py:205` `for key, entry in list(self._entries.items()):`

**invalidate**, body `sources/hindsight/hindsight-api-slim/hindsight_api/engine/providers/github_copilot_llm.py:192`; implementation/call-through.

- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/providers/github_copilot_llm.py:190` `async def invalidate(self, expected_client: CopilotClient, reason: str) -> None:`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/providers/github_copilot_llm.py:192` `async with self._lifecycle_lock:`

**delete_chunks_by_ids**, body `sources/hindsight/hindsight-api-slim/hindsight_api/engine/retain/chunk_storage.py:61`; implementation/call-through.

- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/retain/chunk_storage.py:44` `async def delete_chunks_by_ids(conn, chunk_ids: list[str], bank_id: str, ops=None) -> int:`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/retain/chunk_storage.py:71` `invalidated = 0`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/retain/chunk_storage.py:77` `invalidated = await delete_stale_observations_for_memories(conn, bank_id, outgoing_unit_ids, ops=ops)`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/retain/chunk_storage.py:101` `await get_memories().delete_chunks(conn=conn, fq_table=fq_table, bank_id=bank_id, chunk_ids=chunk_ids)`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/retain/chunk_storage.py:102` `return invalidated`

**delete_stale_observations_for_memories**, body `sources/hindsight/hindsight-api-slim/hindsight_api/engine/retain/fact_storage.py:175`; implementation/call-through.

- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/retain/fact_storage.py:146` `async def delete_stale_observations_for_memories(`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/retain/fact_storage.py:180` `return await get_memories().delete_stale_observations(`

**delete_object_store_prefix**, body `sources/hindsight/hindsight-api-slim/hindsight_api/engine/storage/base.py:136`; implementation/call-through.

- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/storage/base.py:134` `async def delete_object_store_prefix(store, prefix: str) -> int:`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/storage/base.py:139` `await obs.delete_async(store, [meta["path"] for meta in page])`

**delete**, body `sources/hindsight/hindsight-api-slim/hindsight_api/engine/storage/base.py:87`; stub.

- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/storage/base.py:79` `@abstractmethod`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/storage/base.py:80` `async def delete(self, key: str) -> None:`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/storage/base.py:87` `pass`

**delete_prefix**, body `sources/hindsight/hindsight-api-slim/hindsight_api/engine/storage/base.py:97`; stub.

- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/storage/base.py:89` `async def delete_prefix(self, prefix: str) -> int:`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/storage/base.py:97` `raise NotImplementedError(f"{type(self).__name__} does not support delete_prefix")`

**delete**, body `sources/hindsight/hindsight-api-slim/hindsight_api/engine/storage/base.py:216`; implementation/call-through.

- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/storage/base.py:215` `async def delete(self, key: str) -> None:`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/storage/base.py:216` `await obs.delete_async(self._store, key)`

**delete_prefix**, body `sources/hindsight/hindsight-api-slim/hindsight_api/engine/storage/base.py:219`; implementation/call-through.

- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/storage/base.py:218` `async def delete_prefix(self, prefix: str) -> int:`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/storage/base.py:219` `return await delete_object_store_prefix(self._store, prefix)`

**_delete_with_chunks**, body `sources/hindsight/hindsight-api-slim/hindsight_api/engine/storage/postgresql.py:145`; implementation/call-through.

- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/storage/postgresql.py:137` `async def _delete_with_chunks(self, conn: Any, key: str, *, keep_key: bool = False) -> bool:`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/storage/postgresql.py:153` `f"DELETE FROM {fq_table('file_storage', self._schema)} WHERE storage_key = ANY($1)",`

**delete**, body `sources/hindsight/hindsight-api-slim/hindsight_api/engine/storage/postgresql.py:301`; implementation/call-through.

- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/storage/postgresql.py:299` `async def delete(self, key: str) -> None:`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/storage/postgresql.py:301` `pool = self._pool_getter()`

**delete_prefix**, body `sources/hindsight/hindsight-api-slim/hindsight_api/engine/storage/postgresql.py:308`; implementation/call-through.

- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/storage/postgresql.py:306` `async def delete_prefix(self, prefix: str) -> int:`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/storage/postgresql.py:313` `f"DELETE FROM {fq_table('file_storage', self._schema)} WHERE storage_key LIKE $1 ESCAPE '!'",`

**build_bank_archive**, body `sources/hindsight/hindsight-api-slim/hindsight_api/engine/transfer/export.py:945`; implementation/call-through.

- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/transfer/export.py:937` `async def build_bank_archive(payload: BankExportPayload) -> bytes:`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/transfer/export.py:945` `archive_bytes = await anyio.to_thread.run_sync(_build_bank_archive_bytes, payload)`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/transfer/export.py:963` `len(archive_bytes),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/transfer/export.py:965` `return archive_bytes`

**_build_bank_archive_bytes**, body `sources/hindsight/hindsight-api-slim/hindsight_api/engine/transfer/export.py:970`; implementation/call-through.

- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/transfer/export.py:968` `def _build_bank_archive_bytes(payload: BankExportPayload) -> bytes:`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/transfer/export.py:982` `archive = io.BytesIO()`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/transfer/export.py:984` `with zipfile.ZipFile(archive, "w", zipfile.ZIP_DEFLATED) as zf:`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/transfer/export.py:1007` `# importer can distinguish "no pages" from a pre-tree archive.`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/transfer/export.py:1025` `archive_type="bank",`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/transfer/export.py:1034` `invalidated_memory_count=len(data_rows.get("invalidated_memory_units", [])),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/transfer/export.py:1039` `return archive.getvalue()`

**_iter_invalidated_units**, body `sources/hindsight/hindsight-api-slim/hindsight_api/engine/transfer/export.py:1118`; implementation/call-through.

- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/transfer/export.py:1112` `async def _iter_invalidated_units(conn_or_backend: Any, bank_id: str, memories: Any) -> AsyncIterator[list[dict]]:`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/transfer/export.py:1119` `async for page in memories.iter_archived_memories(`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/transfer/export.py:1123` `except NotImplementedError:`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/transfer/export.py:1125` `"[transfer] The memories store cannot list archived memories; bank %s is exported without "`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/transfer/export.py:1126` `"its curation archive",`

**_open_archive**, body `sources/hindsight/hindsight-api-slim/hindsight_api/engine/transfer/importer.py:204`; implementation/call-through.

- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/transfer/importer.py:191` `def _open_archive(archive: Archive, *, produced_by: str) -> zipfile.ZipFile:`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/transfer/importer.py:205` `zf = zipfile.ZipFile(io.BytesIO(archive) if isinstance(archive, bytes) else archive, "r")`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/transfer/importer.py:207` `raise ValueError(f"Invalid transfer archive: the uploaded file is not a readable .zip ({e})") from e`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/transfer/importer.py:211` `f"Invalid transfer archive: manifest.json is missing. This endpoint only accepts a .zip produced "`

**parse_archive**, body `sources/hindsight/hindsight-api-slim/hindsight_api/engine/transfer/importer.py:285`; implementation/call-through.

- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/transfer/importer.py:280` `def parse_archive(archive: Archive) -> ParsedArchive:`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/transfer/importer.py:285` `with _open_archive(archive, produced_by="the document export endpoint") as zf:`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/transfer/importer.py:286` `return ParsedArchive(`

**iter_archive_documents**, body `sources/hindsight/hindsight-api-slim/hindsight_api/engine/transfer/importer.py:300`; implementation/call-through.

- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/transfer/importer.py:295` `def iter_archive_documents(archive: Archive) -> Iterator[TransferDocument]:`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/transfer/importer.py:300` `with _open_archive(archive, produced_by="the document export endpoint") as zf:`

**parse_bank_archive**, body `sources/hindsight/hindsight-api-slim/hindsight_api/engine/transfer/importer.py:594`; implementation/call-through.

- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/transfer/importer.py:592` `def parse_bank_archive(zf: zipfile.ZipFile) -> ParsedBankArchive:`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/transfer/importer.py:595` `if manifest.archive_type != "bank":`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/transfer/importer.py:597` `f"Not a whole-bank archive (archive_type={manifest.archive_type!r}); use import_documents instead"`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/transfer/importer.py:600` `# Typed tree , absent on pre-tree archives, which restore with no pages.`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/transfer/importer.py:606` `f"Invalid transfer archive: attachment {attachment.short_id} names entry "`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/transfer/importer.py:607` `f"{attachment.entry!r}, which the archive does not contain"`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/transfer/importer.py:609` `return ParsedBankArchive(`

**_restore_invalidated_units**, body `sources/hindsight/hindsight-api-slim/hindsight_api/engine/transfer/importer.py:1121`; implementation/call-through.

- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/transfer/importer.py:1105` `async def _restore_invalidated_units(`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/transfer/importer.py:1127` `restored += await get_memories().restore_archived_memories(`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/transfer/importer.py:1136` `except NotImplementedError:`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/transfer/importer.py:1138` `"[transfer] The memories store cannot restore archived memories; %d archived fact(s) of bank %s "`

**reset**, body `sources/hindsight/hindsight-api-slim/hindsight_api/extensions/llm_request.py:198`; implementation/call-through.

- `sources/hindsight/hindsight-api-slim/hindsight_api/extensions/llm_request.py:197` `def reset(self) -> None:`
- `sources/hindsight/hindsight-api-slim/hindsight_api/extensions/llm_request.py:198` `if self._overrides_token is not None:`

**_register_delete_mental_model**, body `sources/hindsight/hindsight-api-slim/hindsight_api/mcp_tools.py:2300`; implementation/call-through.

- `sources/hindsight/hindsight-api-slim/hindsight_api/mcp_tools.py:2297` `def _register_delete_mental_model(mcp: FastMCP, memory: MemoryEngine, config: MCPToolsConfig) -> None:`
- `sources/hindsight/hindsight-api-slim/hindsight_api/mcp_tools.py:2301` `deleted = await memory.delete_mental_model(`
- `sources/hindsight/hindsight-api-slim/hindsight_api/mcp_tools.py:2308` `return {"status": "deleted", "mental_model_id": mental_model_id}`

**_do_delete_knowledge_node**, body `sources/hindsight/hindsight-api-slim/hindsight_api/mcp_tools.py:2719`; implementation/call-through.

- `sources/hindsight/hindsight-api-slim/hindsight_api/mcp_tools.py:2715` `async def _do_delete_knowledge_node(`
- `sources/hindsight/hindsight-api-slim/hindsight_api/mcp_tools.py:2719` `deleted = await memory.delete_knowledge_node(bank_id=target_bank, node_id=node_id, request_context=request_context)`
- `sources/hindsight/hindsight-api-slim/hindsight_api/mcp_tools.py:2722` `return {"status": "deleted", "node_id": node_id}`

**_register_delete_knowledge_node**, body `sources/hindsight/hindsight-api-slim/hindsight_api/mcp_tools.py:3239`; implementation/call-through.

- `sources/hindsight/hindsight-api-slim/hindsight_api/mcp_tools.py:3236` `def _register_delete_knowledge_node(mcp: FastMCP, memory: MemoryEngine, config: MCPToolsConfig) -> None:`
- `sources/hindsight/hindsight-api-slim/hindsight_api/mcp_tools.py:3239` `async def _run(target_bank: str, node_id: str) -> Any:`

**_register_delete_directive**, body `sources/hindsight/hindsight-api-slim/hindsight_api/mcp_tools.py:3458`; implementation/call-through.

- `sources/hindsight/hindsight-api-slim/hindsight_api/mcp_tools.py:3455` `def _register_delete_directive(mcp: FastMCP, memory: MemoryEngine, config: MCPToolsConfig) -> None:`
- `sources/hindsight/hindsight-api-slim/hindsight_api/mcp_tools.py:3459` `deleted = await memory.delete_directive(`
- `sources/hindsight/hindsight-api-slim/hindsight_api/mcp_tools.py:3466` `return {"status": "deleted", "directive_id": directive_id}`

**_register_invalidate_memory**, body `sources/hindsight/hindsight-api-slim/hindsight_api/mcp_tools.py:3813`; implementation/call-through.

- `sources/hindsight/hindsight-api-slim/hindsight_api/mcp_tools.py:3810` `def _register_invalidate_memory(mcp: FastMCP, memory: MemoryEngine, config: MCPToolsConfig) -> None:`
- `sources/hindsight/hindsight-api-slim/hindsight_api/mcp_tools.py:3813` `_INVALIDATE_DOC = """`
- `sources/hindsight/hindsight-api-slim/hindsight_api/mcp_tools.py:3816` `Invalidating moves the fact out of the active set: it's excluded from`
- `sources/hindsight/hindsight-api-slim/hindsight_api/mcp_tools.py:3821` `Only raw world/experience facts can be invalidated; observations are derived.`
- `sources/hindsight/hindsight-api-slim/hindsight_api/mcp_tools.py:3828` `state="valid" if restore else "invalidated",`
- `sources/hindsight/hindsight-api-slim/hindsight_api/mcp_tools.py:3838` `@mcp.tool(description=_INVALIDATE_DOC, annotations=_tool_annotations("invalidate_memory"))`
- `sources/hindsight/hindsight-api-slim/hindsight_api/mcp_tools.py:3839` `async def invalidate_memory(`
- `sources/hindsight/hindsight-api-slim/hindsight_api/mcp_tools.py:3848` `reason: Optional free-text reason recorded when invalidating.`
- `sources/hindsight/hindsight-api-slim/hindsight_api/mcp_tools.py:3849` `restore: Set True to restore a previously invalidated fact.`
- `sources/hindsight/hindsight-api-slim/hindsight_api/mcp_tools.py:3856` `action="invalidating memory",`
- `sources/hindsight/hindsight-api-slim/hindsight_api/mcp_tools.py:3863` `@mcp.tool(description=_INVALIDATE_DOC, annotations=_tool_annotations("invalidate_memory"))`
- `sources/hindsight/hindsight-api-slim/hindsight_api/mcp_tools.py:3864` `async def invalidate_memory(`
- `sources/hindsight/hindsight-api-slim/hindsight_api/mcp_tools.py:3872` `reason: Optional free-text reason recorded when invalidating.`
- `sources/hindsight/hindsight-api-slim/hindsight_api/mcp_tools.py:3873` `restore: Set True to restore a previously invalidated fact.`
- `sources/hindsight/hindsight-api-slim/hindsight_api/mcp_tools.py:3879` `action="invalidating memory",`

**_register_delete_document**, body `sources/hindsight/hindsight-api-slim/hindsight_api/mcp_tools.py:4029`; implementation/call-through.

- `sources/hindsight/hindsight-api-slim/hindsight_api/mcp_tools.py:4026` `def _register_delete_document(mcp: FastMCP, memory: MemoryEngine, config: MCPToolsConfig) -> None:`
- `sources/hindsight/hindsight-api-slim/hindsight_api/mcp_tools.py:4030` `result = await memory.delete_document(`
- `sources/hindsight/hindsight-api-slim/hindsight_api/mcp_tools.py:4035` `return {"status": "deleted", "document_id": document_id, **result}`

**_register_delete_bank**, body `sources/hindsight/hindsight-api-slim/hindsight_api/mcp_tools.py:4596`; implementation/call-through.

- `sources/hindsight/hindsight-api-slim/hindsight_api/mcp_tools.py:4593` `def _register_delete_bank(mcp: FastMCP, memory: MemoryEngine, config: MCPToolsConfig) -> None:`
- `sources/hindsight/hindsight-api-slim/hindsight_api/mcp_tools.py:4597` `result = await memory.delete_bank(`
- `sources/hindsight/hindsight-api-slim/hindsight_api/mcp_tools.py:4601` `return {"status": "deleted", "bank_id": target_bank, **result}`

**delete_mental_model**, body `sources/hindsight/hindsight-api-slim/hindsight_api/mcp_tools.py:2326`; implementation/call-through.

- `sources/hindsight/hindsight-api-slim/hindsight_api/mcp_tools.py:2312` `@mcp.tool(annotations=_tool_annotations("delete_mental_model"))`
- `sources/hindsight/hindsight-api-slim/hindsight_api/mcp_tools.py:2313` `async def delete_mental_model(`
- `sources/hindsight/hindsight-api-slim/hindsight_api/mcp_tools.py:2326` `return await _run_tool(`

**delete_mental_model**, body `sources/hindsight/hindsight-api-slim/hindsight_api/mcp_tools.py:2349`; implementation/call-through.

- `sources/hindsight/hindsight-api-slim/hindsight_api/mcp_tools.py:2337` `@mcp.tool(annotations=_tool_annotations("delete_mental_model"))`
- `sources/hindsight/hindsight-api-slim/hindsight_api/mcp_tools.py:2338` `async def delete_mental_model(`
- `sources/hindsight/hindsight-api-slim/hindsight_api/mcp_tools.py:2349` `return await _run_tool(`

**delete_knowledge_node**, body `sources/hindsight/hindsight-api-slim/hindsight_api/mcp_tools.py:3259`; implementation/call-through.

- `sources/hindsight/hindsight-api-slim/hindsight_api/mcp_tools.py:3244` `@mcp.tool(annotations=_tool_annotations("delete_knowledge_node"))`
- `sources/hindsight/hindsight-api-slim/hindsight_api/mcp_tools.py:3245` `async def delete_knowledge_node(`
- `sources/hindsight/hindsight-api-slim/hindsight_api/mcp_tools.py:3259` `return await _run_tool(`

**delete_knowledge_node**, body `sources/hindsight/hindsight-api-slim/hindsight_api/mcp_tools.py:3283`; implementation/call-through.

- `sources/hindsight/hindsight-api-slim/hindsight_api/mcp_tools.py:3270` `@mcp.tool(annotations=_tool_annotations("delete_knowledge_node"))`
- `sources/hindsight/hindsight-api-slim/hindsight_api/mcp_tools.py:3271` `async def delete_knowledge_node(`
- `sources/hindsight/hindsight-api-slim/hindsight_api/mcp_tools.py:3283` `return await _run_tool(`

**delete_directive**, body `sources/hindsight/hindsight-api-slim/hindsight_api/mcp_tools.py:3484`; implementation/call-through.

- `sources/hindsight/hindsight-api-slim/hindsight_api/mcp_tools.py:3470` `@mcp.tool(annotations=_tool_annotations("delete_directive"))`
- `sources/hindsight/hindsight-api-slim/hindsight_api/mcp_tools.py:3471` `async def delete_directive(`
- `sources/hindsight/hindsight-api-slim/hindsight_api/mcp_tools.py:3484` `return await _run_tool(`

**delete_directive**, body `sources/hindsight/hindsight-api-slim/hindsight_api/mcp_tools.py:3507`; implementation/call-through.

- `sources/hindsight/hindsight-api-slim/hindsight_api/mcp_tools.py:3495` `@mcp.tool(annotations=_tool_annotations("delete_directive"))`
- `sources/hindsight/hindsight-api-slim/hindsight_api/mcp_tools.py:3496` `async def delete_directive(`
- `sources/hindsight/hindsight-api-slim/hindsight_api/mcp_tools.py:3507` `return await _run_tool(`

**invalidate_memory**, body `sources/hindsight/hindsight-api-slim/hindsight_api/mcp_tools.py:3852`; implementation/call-through.

- `sources/hindsight/hindsight-api-slim/hindsight_api/mcp_tools.py:3838` `@mcp.tool(description=_INVALIDATE_DOC, annotations=_tool_annotations("invalidate_memory"))`
- `sources/hindsight/hindsight-api-slim/hindsight_api/mcp_tools.py:3839` `async def invalidate_memory(`
- `sources/hindsight/hindsight-api-slim/hindsight_api/mcp_tools.py:3856` `action="invalidating memory",`

**invalidate_memory**, body `sources/hindsight/hindsight-api-slim/hindsight_api/mcp_tools.py:3875`; implementation/call-through.

- `sources/hindsight/hindsight-api-slim/hindsight_api/mcp_tools.py:3863` `@mcp.tool(description=_INVALIDATE_DOC, annotations=_tool_annotations("invalidate_memory"))`
- `sources/hindsight/hindsight-api-slim/hindsight_api/mcp_tools.py:3864` `async def invalidate_memory(`
- `sources/hindsight/hindsight-api-slim/hindsight_api/mcp_tools.py:3879` `action="invalidating memory",`

**delete_document**, body `sources/hindsight/hindsight-api-slim/hindsight_api/mcp_tools.py:4053`; implementation/call-through.

- `sources/hindsight/hindsight-api-slim/hindsight_api/mcp_tools.py:4039` `@mcp.tool(annotations=_tool_annotations("delete_document"))`
- `sources/hindsight/hindsight-api-slim/hindsight_api/mcp_tools.py:4040` `async def delete_document(`
- `sources/hindsight/hindsight-api-slim/hindsight_api/mcp_tools.py:4053` `return await _run_tool(`

**delete_document**, body `sources/hindsight/hindsight-api-slim/hindsight_api/mcp_tools.py:4076`; implementation/call-through.

- `sources/hindsight/hindsight-api-slim/hindsight_api/mcp_tools.py:4064` `@mcp.tool(annotations=_tool_annotations("delete_document"))`
- `sources/hindsight/hindsight-api-slim/hindsight_api/mcp_tools.py:4065` `async def delete_document(`
- `sources/hindsight/hindsight-api-slim/hindsight_api/mcp_tools.py:4076` `return await _run_tool(`

**delete_bank**, body `sources/hindsight/hindsight-api-slim/hindsight_api/mcp_tools.py:4618`; implementation/call-through.

- `sources/hindsight/hindsight-api-slim/hindsight_api/mcp_tools.py:4605` `@mcp.tool(annotations=_tool_annotations("delete_bank"))`
- `sources/hindsight/hindsight-api-slim/hindsight_api/mcp_tools.py:4606` `async def delete_bank(`
- `sources/hindsight/hindsight-api-slim/hindsight_api/mcp_tools.py:4618` `return await _run_tool(`

**delete_bank**, body `sources/hindsight/hindsight-api-slim/hindsight_api/mcp_tools.py:4637`; implementation/call-through.

- `sources/hindsight/hindsight-api-slim/hindsight_api/mcp_tools.py:4629` `@mcp.tool(annotations=_tool_annotations("delete_bank"))`
- `sources/hindsight/hindsight-api-slim/hindsight_api/mcp_tools.py:4630` `async def delete_bank() -> dict:`
- `sources/hindsight/hindsight-api-slim/hindsight_api/mcp_tools.py:4637` `return await _run_tool(`

**operation and route call sites**, body `sources/hindsight/hindsight-cli/src/api.rs:461`; body evidence.

- `sources/hindsight/hindsight-cli/src/api.rs:460` ``
- `sources/hindsight/hindsight-cli/src/api.rs:461` `pub fn delete_memory(`
- `sources/hindsight/hindsight-cli/src/api.rs:462` `&self,`
- `sources/hindsight/hindsight-cli/src/api.rs:463` `_agent_id: &str,`
- `sources/hindsight/hindsight-cli/src/api.rs:464` `_unit_id: &str,`
- `sources/hindsight/hindsight-cli/src/api.rs:465` `_verbose: bool,`
- `sources/hindsight/hindsight-cli/src/api.rs:466` `) -> Result<types::DeleteResponse> {`
- `sources/hindsight/hindsight-cli/src/api.rs:467` `// Note: Individual memory deletion is no longer supported in the API`
- `sources/hindsight/hindsight-cli/src/api.rs:468` `anyhow::bail!("Individual memory deletion is no longer supported. Use 'memory clear' to clear all memories.")`
- `sources/hindsight/hindsight-cli/src/api.rs:469` `}`
- `sources/hindsight/hindsight-cli/src/api.rs:470` ``
- `sources/hindsight/hindsight-cli/src/api.rs:471` `pub fn clear_memories(`
- `sources/hindsight/hindsight-cli/src/api.rs:472` `&self,`
- `sources/hindsight/hindsight-cli/src/api.rs:475` `_verbose: bool,`
- `sources/hindsight/hindsight-cli/src/api.rs:476` `) -> Result<types::DeleteResponse> {`
- `sources/hindsight/hindsight-cli/src/api.rs:477` `self.runtime.block_on(async {`
- `sources/hindsight/hindsight-cli/src/api.rs:478` `let response = self`
- `sources/hindsight/hindsight-cli/src/api.rs:479` `.client`
- `sources/hindsight/hindsight-cli/src/api.rs:480` `.clear_bank_memories(agent_id, None, Some(fact_type))`
- `sources/hindsight/hindsight-cli/src/api.rs:554` ``
- `sources/hindsight/hindsight-cli/src/api.rs:555` `pub fn delete_document(`
- `sources/hindsight/hindsight-cli/src/api.rs:556` `&self,`
- `sources/hindsight/hindsight-cli/src/api.rs:557` `agent_id: &str,`
- `sources/hindsight/hindsight-cli/src/api.rs:558` `document_id: &str,`
- `sources/hindsight/hindsight-cli/src/api.rs:559` `_verbose: bool,`
- `sources/hindsight/hindsight-cli/src/api.rs:560` `) -> Result<types::DeleteResponse> {`
- `sources/hindsight/hindsight-cli/src/api.rs:561` `self.runtime.block_on(async {`
- `sources/hindsight/hindsight-cli/src/api.rs:562` `let response = self`
- `sources/hindsight/hindsight-cli/src/api.rs:563` `.client`
- `sources/hindsight/hindsight-cli/src/api.rs:564` `.delete_document(agent_id, document_id, None)`
- `sources/hindsight/hindsight-cli/src/api.rs:565` `.humanized()`
- `sources/hindsight/hindsight-cli/src/api.rs:566` `.await?;`
- `sources/hindsight/hindsight-cli/src/api.rs:567` `let value = response.into_inner();`
- `sources/hindsight/hindsight-cli/src/api.rs:568` `// Convert typed response to DeleteResponse`
- `sources/hindsight/hindsight-cli/src/api.rs:569` `Ok(types::DeleteResponse {`
- `sources/hindsight/hindsight-cli/src/api.rs:570` `deleted_count: Some(value.memory_units_deleted),`
- `sources/hindsight/hindsight-cli/src/api.rs:571` `message: Some(value.message),`
- `sources/hindsight/hindsight-cli/src/api.rs:572` `success: value.success,`
- `sources/hindsight/hindsight-cli/src/api.rs:573` `})`
- `sources/hindsight/hindsight-cli/src/api.rs:574` `})`
- `sources/hindsight/hindsight-cli/src/api.rs:596` `_verbose: bool,`
- `sources/hindsight/hindsight-cli/src/api.rs:597` `) -> Result<types::DeleteResponse> {`
- `sources/hindsight/hindsight-cli/src/api.rs:598` `self.runtime.block_on(async {`
- `sources/hindsight/hindsight-cli/src/api.rs:599` `let response = self`
- `sources/hindsight/hindsight-cli/src/api.rs:600` `.client`
- `sources/hindsight/hindsight-cli/src/api.rs:601` `.cancel_operation(agent_id, operation_id, None)`
- `sources/hindsight/hindsight-cli/src/api.rs:605` `// Convert typed response to DeleteResponse`
- `sources/hindsight/hindsight-cli/src/api.rs:606` `Ok(types::DeleteResponse {`
- `sources/hindsight/hindsight-cli/src/api.rs:607` `deleted_count: None,`
- `sources/hindsight/hindsight-cli/src/api.rs:608` `message: Some(value.message),`
- `sources/hindsight/hindsight-cli/src/api.rs:609` `success: value.success,`
- `sources/hindsight/hindsight-cli/src/api.rs:610` `})`
- `sources/hindsight/hindsight-cli/src/api.rs:611` `})`
- `sources/hindsight/hindsight-cli/src/api.rs:705` ``
- `sources/hindsight/hindsight-cli/src/api.rs:706` `pub fn delete_bank(&self, bank_id: &str, _verbose: bool) -> Result<types::DeleteResponse> {`
- `sources/hindsight/hindsight-cli/src/api.rs:707` `self.runtime.block_on(async {`
- `sources/hindsight/hindsight-cli/src/api.rs:708` `let response = self.client.delete_bank(bank_id, None).humanized().await?;`
- `sources/hindsight/hindsight-cli/src/api.rs:709` `Ok(response.into_inner())`
- `sources/hindsight/hindsight-cli/src/api.rs:710` `})`
- `sources/hindsight/hindsight-cli/src/api.rs:711` `}`
- `sources/hindsight/hindsight-cli/src/api.rs:712` `}`
- `sources/hindsight/hindsight-cli/src/api.rs:1021` ``
- `sources/hindsight/hindsight-cli/src/api.rs:1022` `pub fn delete_mental_model(`
- `sources/hindsight/hindsight-cli/src/api.rs:1023` `&self,`
- `sources/hindsight/hindsight-cli/src/api.rs:1024` `bank_id: &str,`
- `sources/hindsight/hindsight-cli/src/api.rs:1025` `mental_model_id: &str,`
- `sources/hindsight/hindsight-cli/src/api.rs:1026` `_verbose: bool,`
- `sources/hindsight/hindsight-cli/src/api.rs:1030` `.client`
- `sources/hindsight/hindsight-cli/src/api.rs:1031` `.delete_mental_model(bank_id, mental_model_id, None)`
- `sources/hindsight/hindsight-cli/src/api.rs:1032` `.humanized()`
- `sources/hindsight/hindsight-cli/src/api.rs:1033` `.await?;`
- `sources/hindsight/hindsight-cli/src/api.rs:1034` `Ok(response.into_inner())`
- `sources/hindsight/hindsight-cli/src/api.rs:1035` `})`
- `sources/hindsight/hindsight-cli/src/api.rs:1205` ``
- `sources/hindsight/hindsight-cli/src/api.rs:1206` `pub fn delete_knowledge_node(`
- `sources/hindsight/hindsight-cli/src/api.rs:1207` `&self,`
- `sources/hindsight/hindsight-cli/src/api.rs:1208` `bank_id: &str,`
- `sources/hindsight/hindsight-cli/src/api.rs:1209` `node_id: &str,`
- `sources/hindsight/hindsight-cli/src/api.rs:1210` `_verbose: bool,`
- `sources/hindsight/hindsight-cli/src/api.rs:1214` `.client`
- `sources/hindsight/hindsight-cli/src/api.rs:1215` `.delete_knowledge_node(bank_id, node_id, None)`
- `sources/hindsight/hindsight-cli/src/api.rs:1216` `.humanized()`
- `sources/hindsight/hindsight-cli/src/api.rs:1217` `.await?;`
- `sources/hindsight/hindsight-cli/src/api.rs:1218` `Ok(response.into_inner())`
- `sources/hindsight/hindsight-cli/src/api.rs:1219` `})`
- `sources/hindsight/hindsight-cli/src/api.rs:1302` ``
- `sources/hindsight/hindsight-cli/src/api.rs:1303` `pub fn delete_directive(`
- `sources/hindsight/hindsight-cli/src/api.rs:1304` `&self,`
- `sources/hindsight/hindsight-cli/src/api.rs:1305` `bank_id: &str,`
- `sources/hindsight/hindsight-cli/src/api.rs:1306` `directive_id: &str,`
- `sources/hindsight/hindsight-cli/src/api.rs:1307` `_verbose: bool,`
- `sources/hindsight/hindsight-cli/src/api.rs:1311` `.client`
- `sources/hindsight/hindsight-cli/src/api.rs:1312` `.delete_directive(bank_id, directive_id, None)`
- `sources/hindsight/hindsight-cli/src/api.rs:1313` `.humanized()`
- `sources/hindsight/hindsight-cli/src/api.rs:1314` `.await?;`
- `sources/hindsight/hindsight-cli/src/api.rs:1315` `Ok(response.into_inner())`
- `sources/hindsight/hindsight-cli/src/api.rs:1316` `})`
- `sources/hindsight/hindsight-cli/src/api.rs:1375` ``
- `sources/hindsight/hindsight-cli/src/api.rs:1376` `pub fn delete_bank_alias(`
- `sources/hindsight/hindsight-cli/src/api.rs:1377` `&self,`
- `sources/hindsight/hindsight-cli/src/api.rs:1378` `bank_id: &str,`
- `sources/hindsight/hindsight-cli/src/api.rs:1379` `alias: &str,`
- `sources/hindsight/hindsight-cli/src/api.rs:1380` `_verbose: bool,`
- `sources/hindsight/hindsight-cli/src/api.rs:1384` `.client`
- `sources/hindsight/hindsight-cli/src/api.rs:1385` `.delete_bank_alias(bank_id, alias, None)`
- `sources/hindsight/hindsight-cli/src/api.rs:1386` `.humanized()`
- `sources/hindsight/hindsight-cli/src/api.rs:1387` `.await?;`
- `sources/hindsight/hindsight-cli/src/api.rs:1388` `Ok(response.into_inner())`
- `sources/hindsight/hindsight-cli/src/api.rs:1389` `})`
- `sources/hindsight/hindsight-cli/src/api.rs:1413` `_verbose: bool,`
- `sources/hindsight/hindsight-cli/src/api.rs:1414` `) -> Result<types::DeleteResponse> {`
- `sources/hindsight/hindsight-cli/src/api.rs:1415` `self.runtime.block_on(async {`
- `sources/hindsight/hindsight-cli/src/api.rs:1416` `let response = self`
- `sources/hindsight/hindsight-cli/src/api.rs:1417` `.client`
- `sources/hindsight/hindsight-cli/src/api.rs:1418` `.clear_observations(bank_id, None)`
- `sources/hindsight/hindsight-cli/src/api.rs:1490` ``
- `sources/hindsight/hindsight-cli/src/api.rs:1491` `pub fn delete_webhook(`
- `sources/hindsight/hindsight-cli/src/api.rs:1492` `&self,`
- `sources/hindsight/hindsight-cli/src/api.rs:1493` `bank_id: &str,`
- `sources/hindsight/hindsight-cli/src/api.rs:1494` `webhook_id: &str,`
- `sources/hindsight/hindsight-cli/src/api.rs:1495` `_verbose: bool,`
- `sources/hindsight/hindsight-cli/src/api.rs:1496` `) -> Result<types::DeleteResponse> {`
- `sources/hindsight/hindsight-cli/src/api.rs:1497` `self.runtime.block_on(async {`
- `sources/hindsight/hindsight-cli/src/api.rs:1498` `let response = self`
- `sources/hindsight/hindsight-cli/src/api.rs:1499` `.client`
- `sources/hindsight/hindsight-cli/src/api.rs:1500` `.delete_webhook(bank_id, webhook_id, None)`
- `sources/hindsight/hindsight-cli/src/api.rs:1501` `.humanized()`
- `sources/hindsight/hindsight-cli/src/api.rs:1502` `.await?;`
- `sources/hindsight/hindsight-cli/src/api.rs:1503` `Ok(response.into_inner())`
- `sources/hindsight/hindsight-cli/src/api.rs:1504` `})`
- `sources/hindsight/hindsight-cli/src/api.rs:1716` ``
- `sources/hindsight/hindsight-cli/src/api.rs:1717` `pub fn delete_operation(`
- `sources/hindsight/hindsight-cli/src/api.rs:1718` `&self,`
- `sources/hindsight/hindsight-cli/src/api.rs:1719` `bank_id: &str,`
- `sources/hindsight/hindsight-cli/src/api.rs:1720` `operation_id: &str,`
- `sources/hindsight/hindsight-cli/src/api.rs:1721` `_verbose: bool,`
- `sources/hindsight/hindsight-cli/src/api.rs:1722` `) -> Result<types::DeleteOperationResponse> {`
- `sources/hindsight/hindsight-cli/src/api.rs:1723` `self.runtime.block_on(async {`
- `sources/hindsight/hindsight-cli/src/api.rs:1724` `let response = self`
- `sources/hindsight/hindsight-cli/src/api.rs:1725` `.client`
- `sources/hindsight/hindsight-cli/src/api.rs:1726` `.delete_operation(bank_id, operation_id, None)`
- `sources/hindsight/hindsight-cli/src/api.rs:1727` `.humanized()`
- `sources/hindsight/hindsight-cli/src/api.rs:1728` `.await?;`
- `sources/hindsight/hindsight-cli/src/api.rs:1729` `Ok(response.into_inner())`
- `sources/hindsight/hindsight-cli/src/api.rs:1730` `})`

**operation and route call sites**, body `sources/hindsight/hindsight-cli/src/commands/bank.rs:483`; body evidence.

- `sources/hindsight/hindsight-cli/src/commands/bank.rs:482` ``
- `sources/hindsight/hindsight-cli/src/commands/bank.rs:483` `pub fn delete(`
- `sources/hindsight/hindsight-cli/src/commands/bank.rs:484` `client: &ApiClient,`
- `sources/hindsight/hindsight-cli/src/commands/bank.rs:485` `bank_id: &str,`
- `sources/hindsight/hindsight-cli/src/commands/bank.rs:486` `yes: bool,`
- `sources/hindsight/hindsight-cli/src/commands/bank.rs:487` `verbose: bool,`
- `sources/hindsight/hindsight-cli/src/commands/bank.rs:492` `let message = format!(`
- `sources/hindsight/hindsight-cli/src/commands/bank.rs:493` `"Are you sure you want to delete bank '{}' and ALL its data? This cannot be undone.",`
- `sources/hindsight/hindsight-cli/src/commands/bank.rs:494` `bank_id`
- `sources/hindsight/hindsight-cli/src/commands/bank.rs:495` `);`
- `sources/hindsight/hindsight-cli/src/commands/bank.rs:496` ``
- `sources/hindsight/hindsight-cli/src/commands/bank.rs:497` `let confirmed = ui::prompt_confirmation(&message)?;`
- `sources/hindsight/hindsight-cli/src/commands/bank.rs:505` `let spinner = if output_format == OutputFormat::Pretty {`
- `sources/hindsight/hindsight-cli/src/commands/bank.rs:506` `Some(ui::create_spinner("Deleting bank..."))`
- `sources/hindsight/hindsight-cli/src/commands/bank.rs:507` `} else {`
- `sources/hindsight/hindsight-cli/src/commands/bank.rs:508` `None`
- `sources/hindsight/hindsight-cli/src/commands/bank.rs:509` `};`
- `sources/hindsight/hindsight-cli/src/commands/bank.rs:510` ``
- `sources/hindsight/hindsight-cli/src/commands/bank.rs:511` `let response = client.delete_bank(bank_id, verbose);`
- `sources/hindsight/hindsight-cli/src/commands/bank.rs:512` ``
- `sources/hindsight/hindsight-cli/src/commands/bank.rs:513` `if let Some(mut sp) = spinner {`
- `sources/hindsight/hindsight-cli/src/commands/bank.rs:514` `sp.finish();`
- `sources/hindsight/hindsight-cli/src/commands/bank.rs:515` `}`
- `sources/hindsight/hindsight-cli/src/commands/bank.rs:520` `if result.success {`
- `sources/hindsight/hindsight-cli/src/commands/bank.rs:521` `ui::print_success(&format!("Bank '{}' deleted successfully", bank_id));`
- `sources/hindsight/hindsight-cli/src/commands/bank.rs:522` `if let Some(count) = result.deleted_count {`
- `sources/hindsight/hindsight-cli/src/commands/bank.rs:523` `println!("  Items deleted: {}", count);`
- `sources/hindsight/hindsight-cli/src/commands/bank.rs:524` `}`
- `sources/hindsight/hindsight-cli/src/commands/bank.rs:525` `} else {`
- `sources/hindsight/hindsight-cli/src/commands/bank.rs:526` `ui::print_error("Failed to delete bank");`
- `sources/hindsight/hindsight-cli/src/commands/bank.rs:527` `}`
- `sources/hindsight/hindsight-cli/src/commands/bank.rs:528` `} else {`
- `sources/hindsight/hindsight-cli/src/commands/bank.rs:529` `output::print_output(&result, output_format)?;`
- `sources/hindsight/hindsight-cli/src/commands/bank.rs:530` `}`
- `sources/hindsight/hindsight-cli/src/commands/bank.rs:637` ``
- `sources/hindsight/hindsight-cli/src/commands/bank.rs:638` `let result = client.delete_bank_alias(bank_id, alias, verbose)?;`
- `sources/hindsight/hindsight-cli/src/commands/bank.rs:639` ``
- `sources/hindsight/hindsight-cli/src/commands/bank.rs:640` `if output_format == OutputFormat::Pretty {`
- `sources/hindsight/hindsight-cli/src/commands/bank.rs:641` `ui::print_success(&format!(`
- `sources/hindsight/hindsight-cli/src/commands/bank.rs:642` `"'{}' no longer reaches bank '{}'",`
- `sources/hindsight/hindsight-cli/src/commands/bank.rs:814` `ui::print_success(&format!("Observations cleared for bank '{}'", bank_id));`
- `sources/hindsight/hindsight-cli/src/commands/bank.rs:815` `if let Some(count) = result.deleted_count {`
- `sources/hindsight/hindsight-cli/src/commands/bank.rs:816` `println!("  Observations deleted: {}", count);`
- `sources/hindsight/hindsight-cli/src/commands/bank.rs:817` `}`
- `sources/hindsight/hindsight-cli/src/commands/bank.rs:818` `} else {`
- `sources/hindsight/hindsight-cli/src/commands/bank.rs:819` `ui::print_error("Failed to clear observations");`
- `sources/hindsight/hindsight-cli/src/commands/bank.rs:820` `}`

**operation and route call sites**, body `sources/hindsight/hindsight-cli/src/commands/directive.rs:212`; body evidence.

- `sources/hindsight/hindsight-cli/src/commands/directive.rs:211` `/// Delete a directive`
- `sources/hindsight/hindsight-cli/src/commands/directive.rs:212` `pub fn delete(`
- `sources/hindsight/hindsight-cli/src/commands/directive.rs:213` `client: &ApiClient,`
- `sources/hindsight/hindsight-cli/src/commands/directive.rs:214` `bank_id: &str,`
- `sources/hindsight/hindsight-cli/src/commands/directive.rs:215` `directive_id: &str,`
- `sources/hindsight/hindsight-cli/src/commands/directive.rs:216` `yes: bool,`
- `sources/hindsight/hindsight-cli/src/commands/directive.rs:222` `let message = format!(`
- `sources/hindsight/hindsight-cli/src/commands/directive.rs:223` `"Are you sure you want to delete directive '{}'? This cannot be undone.",`
- `sources/hindsight/hindsight-cli/src/commands/directive.rs:224` `directive_id`
- `sources/hindsight/hindsight-cli/src/commands/directive.rs:225` `);`
- `sources/hindsight/hindsight-cli/src/commands/directive.rs:226` ``
- `sources/hindsight/hindsight-cli/src/commands/directive.rs:227` `let confirmed = ui::prompt_confirmation(&message)?;`
- `sources/hindsight/hindsight-cli/src/commands/directive.rs:235` `let spinner = if output_format == OutputFormat::Pretty {`
- `sources/hindsight/hindsight-cli/src/commands/directive.rs:236` `Some(ui::create_spinner("Deleting directive..."))`
- `sources/hindsight/hindsight-cli/src/commands/directive.rs:237` `} else {`
- `sources/hindsight/hindsight-cli/src/commands/directive.rs:238` `None`
- `sources/hindsight/hindsight-cli/src/commands/directive.rs:239` `};`
- `sources/hindsight/hindsight-cli/src/commands/directive.rs:240` ``
- `sources/hindsight/hindsight-cli/src/commands/directive.rs:241` `let response = client.delete_directive(bank_id, directive_id, verbose);`
- `sources/hindsight/hindsight-cli/src/commands/directive.rs:242` ``
- `sources/hindsight/hindsight-cli/src/commands/directive.rs:243` `if let Some(mut sp) = spinner {`
- `sources/hindsight/hindsight-cli/src/commands/directive.rs:244` `sp.finish();`
- `sources/hindsight/hindsight-cli/src/commands/directive.rs:245` `}`
- `sources/hindsight/hindsight-cli/src/commands/directive.rs:250` `ui::print_success(&format!(`
- `sources/hindsight/hindsight-cli/src/commands/directive.rs:251` `"Directive '{}' deleted successfully",`
- `sources/hindsight/hindsight-cli/src/commands/directive.rs:252` `directive_id`
- `sources/hindsight/hindsight-cli/src/commands/directive.rs:253` `));`
- `sources/hindsight/hindsight-cli/src/commands/directive.rs:254` `} else {`
- `sources/hindsight/hindsight-cli/src/commands/directive.rs:255` `println!("{{\"success\": true}}");`

**operation and route call sites**, body `sources/hindsight/hindsight-cli/src/commands/document.rs:282`; body evidence.

- `sources/hindsight/hindsight-cli/src/commands/document.rs:281` ``
- `sources/hindsight/hindsight-cli/src/commands/document.rs:282` `pub fn delete(`
- `sources/hindsight/hindsight-cli/src/commands/document.rs:283` `client: &ApiClient,`
- `sources/hindsight/hindsight-cli/src/commands/document.rs:284` `agent_id: &str,`
- `sources/hindsight/hindsight-cli/src/commands/document.rs:285` `document_id: &str,`
- `sources/hindsight/hindsight-cli/src/commands/document.rs:286` `verbose: bool,`
- `sources/hindsight/hindsight-cli/src/commands/document.rs:289` `let spinner = if output_format == OutputFormat::Pretty {`
- `sources/hindsight/hindsight-cli/src/commands/document.rs:290` `Some(ui::create_spinner("Deleting document..."))`
- `sources/hindsight/hindsight-cli/src/commands/document.rs:291` `} else {`
- `sources/hindsight/hindsight-cli/src/commands/document.rs:292` `None`
- `sources/hindsight/hindsight-cli/src/commands/document.rs:293` `};`
- `sources/hindsight/hindsight-cli/src/commands/document.rs:294` ``
- `sources/hindsight/hindsight-cli/src/commands/document.rs:295` `let response = client.delete_document(agent_id, document_id, verbose);`
- `sources/hindsight/hindsight-cli/src/commands/document.rs:296` ``
- `sources/hindsight/hindsight-cli/src/commands/document.rs:297` `if let Some(mut sp) = spinner {`
- `sources/hindsight/hindsight-cli/src/commands/document.rs:298` `sp.finish();`
- `sources/hindsight/hindsight-cli/src/commands/document.rs:299` `}`
- `sources/hindsight/hindsight-cli/src/commands/document.rs:304` `if result.success {`
- `sources/hindsight/hindsight-cli/src/commands/document.rs:305` `ui::print_success("Document deleted successfully");`
- `sources/hindsight/hindsight-cli/src/commands/document.rs:306` `} else {`
- `sources/hindsight/hindsight-cli/src/commands/document.rs:307` `ui::print_error("Failed to delete document");`
- `sources/hindsight/hindsight-cli/src/commands/document.rs:308` `}`
- `sources/hindsight/hindsight-cli/src/commands/document.rs:309` `} else {`
- `sources/hindsight/hindsight-cli/src/commands/document.rs:310` `output::print_output(&result, output_format)?;`
- `sources/hindsight/hindsight-cli/src/commands/document.rs:311` `}`

**operation and route call sites**, body `sources/hindsight/hindsight-cli/src/commands/explore.rs:642`; body evidence.

- `sources/hindsight/hindsight-cli/src/commands/explore.rs:641` ``
- `sources/hindsight/hindsight-cli/src/commands/explore.rs:642` `fn delete_selected_document(&mut self) -> Result<()> {`
- `sources/hindsight/hindsight-cli/src/commands/explore.rs:643` `if let View::Documents(bank_id) = &self.view {`
- `sources/hindsight/hindsight-cli/src/commands/explore.rs:644` `if let Some(i) = selected_headered_data_index(&self.documents_state) {`
- `sources/hindsight/hindsight-cli/src/commands/explore.rs:645` `if let Some(doc) = self.documents.get(i) {`
- `sources/hindsight/hindsight-cli/src/commands/explore.rs:646` `let doc_id = doc.get("id").and_then(|v| v.as_str()).unwrap_or("");`
- `sources/hindsight/hindsight-cli/src/commands/explore.rs:648` `if !doc_id.is_empty() {`
- `sources/hindsight/hindsight-cli/src/commands/explore.rs:649` `match self.client.delete_document(bank_id, doc_id, false) {`
- `sources/hindsight/hindsight-cli/src/commands/explore.rs:650` `Ok(_) => {`
- `sources/hindsight/hindsight-cli/src/commands/explore.rs:651` `self.status_message = format!("Deleted document: {}", doc_id);`
- `sources/hindsight/hindsight-cli/src/commands/explore.rs:652` `self.refresh()?;`
- `sources/hindsight/hindsight-cli/src/commands/explore.rs:653` `}`
- `sources/hindsight/hindsight-cli/src/commands/explore.rs:654` `Err(e) => {`
- `sources/hindsight/hindsight-cli/src/commands/explore.rs:655` `self.error_message = format!("Failed to delete document: {}", e);`
- `sources/hindsight/hindsight-cli/src/commands/explore.rs:656` `}`
- `sources/hindsight/hindsight-cli/src/commands/explore.rs:657` `}`
- `sources/hindsight/hindsight-cli/src/commands/explore.rs:658` `}`
- `sources/hindsight/hindsight-cli/src/commands/explore.rs:659` `}`
- `sources/hindsight/hindsight-cli/src/commands/explore.rs:790` `("←→", "Scroll", BRAND_START),`
- `sources/hindsight/hindsight-cli/src/commands/explore.rs:791` `("Del", "Delete", Color::Red),`
- `sources/hindsight/hindsight-cli/src/commands/explore.rs:792` `("Esc", "Back", BRAND_END),`
- `sources/hindsight/hindsight-cli/src/commands/explore.rs:793` `("R", "Refresh", BRAND_END),`
- `sources/hindsight/hindsight-cli/src/commands/explore.rs:794` `("?", "Help", BRAND_END),`
- `sources/hindsight/hindsight-cli/src/commands/explore.rs:795` `("q", "Quit", Color::Red),`
- `sources/hindsight/hindsight-cli/src/commands/explore.rs:1615` `// Delete document`
- `sources/hindsight/hindsight-cli/src/commands/explore.rs:1616` `KeyCode::Delete => {`
- `sources/hindsight/hindsight-cli/src/commands/explore.rs:1617` `if matches!(app.view, View::Documents(_)) {`
- `sources/hindsight/hindsight-cli/src/commands/explore.rs:1618` `app.delete_selected_document()?;`
- `sources/hindsight/hindsight-cli/src/commands/explore.rs:1619` `}`
- `sources/hindsight/hindsight-cli/src/commands/explore.rs:1620` `}`
- `sources/hindsight/hindsight-cli/src/commands/explore.rs:1621` ``
- `sources/hindsight/hindsight-cli/src/commands/explore.rs:1622` `// Pagination for memories`

**operation and route call sites**, body `sources/hindsight/hindsight-cli/src/commands/knowledge_base.rs:419`; body evidence.

- `sources/hindsight/hindsight-cli/src/commands/knowledge_base.rs:418` `/// Delete a folder or page (and its whole subtree)`
- `sources/hindsight/hindsight-cli/src/commands/knowledge_base.rs:419` `pub fn delete(`
- `sources/hindsight/hindsight-cli/src/commands/knowledge_base.rs:420` `client: &ApiClient,`
- `sources/hindsight/hindsight-cli/src/commands/knowledge_base.rs:421` `bank_id: &str,`
- `sources/hindsight/hindsight-cli/src/commands/knowledge_base.rs:422` `node_id: &str,`
- `sources/hindsight/hindsight-cli/src/commands/knowledge_base.rs:423` `yes: bool,`
- `sources/hindsight/hindsight-cli/src/commands/knowledge_base.rs:430` `let message = format!(`
- `sources/hindsight/hindsight-cli/src/commands/knowledge_base.rs:431` `"Are you sure you want to delete '{}' and everything under it? This cannot be undone.",`
- `sources/hindsight/hindsight-cli/src/commands/knowledge_base.rs:432` `node_id`
- `sources/hindsight/hindsight-cli/src/commands/knowledge_base.rs:433` `);`
- `sources/hindsight/hindsight-cli/src/commands/knowledge_base.rs:434` ``
- `sources/hindsight/hindsight-cli/src/commands/knowledge_base.rs:435` `if !ui::prompt_confirmation(&message)? {`
- `sources/hindsight/hindsight-cli/src/commands/knowledge_base.rs:441` `let spinner = if output_format == OutputFormat::Pretty {`
- `sources/hindsight/hindsight-cli/src/commands/knowledge_base.rs:442` `Some(ui::create_spinner("Deleting node..."))`
- `sources/hindsight/hindsight-cli/src/commands/knowledge_base.rs:443` `} else {`
- `sources/hindsight/hindsight-cli/src/commands/knowledge_base.rs:444` `None`
- `sources/hindsight/hindsight-cli/src/commands/knowledge_base.rs:445` `};`
- `sources/hindsight/hindsight-cli/src/commands/knowledge_base.rs:446` ``
- `sources/hindsight/hindsight-cli/src/commands/knowledge_base.rs:447` `let response = client.delete_knowledge_node(bank_id, node_id, verbose);`
- `sources/hindsight/hindsight-cli/src/commands/knowledge_base.rs:448` ``
- `sources/hindsight/hindsight-cli/src/commands/knowledge_base.rs:449` `if let Some(mut sp) = spinner {`
- `sources/hindsight/hindsight-cli/src/commands/knowledge_base.rs:450` `sp.finish();`
- `sources/hindsight/hindsight-cli/src/commands/knowledge_base.rs:451` `}`
- `sources/hindsight/hindsight-cli/src/commands/knowledge_base.rs:455` `if output_format == OutputFormat::Pretty {`
- `sources/hindsight/hindsight-cli/src/commands/knowledge_base.rs:456` `ui::print_success(&format!("Node '{}' deleted successfully", node_id));`
- `sources/hindsight/hindsight-cli/src/commands/knowledge_base.rs:457` `} else {`
- `sources/hindsight/hindsight-cli/src/commands/knowledge_base.rs:458` `println!("{{\"success\": true}}");`
- `sources/hindsight/hindsight-cli/src/commands/knowledge_base.rs:459` `}`
- `sources/hindsight/hindsight-cli/src/commands/knowledge_base.rs:460` `Ok(())`

**operation and route call sites**, body `sources/hindsight/hindsight-cli/src/commands/memory.rs:704`; body evidence.

- `sources/hindsight/hindsight-cli/src/commands/memory.rs:703` ``
- `sources/hindsight/hindsight-cli/src/commands/memory.rs:704` `pub fn delete(`
- `sources/hindsight/hindsight-cli/src/commands/memory.rs:705` `client: &ApiClient,`
- `sources/hindsight/hindsight-cli/src/commands/memory.rs:706` `agent_id: &str,`
- `sources/hindsight/hindsight-cli/src/commands/memory.rs:707` `unit_id: &str,`
- `sources/hindsight/hindsight-cli/src/commands/memory.rs:708` `verbose: bool,`
- `sources/hindsight/hindsight-cli/src/commands/memory.rs:711` `let spinner = if output_format == OutputFormat::Pretty {`
- `sources/hindsight/hindsight-cli/src/commands/memory.rs:712` `Some(ui::create_spinner("Deleting memory unit..."))`
- `sources/hindsight/hindsight-cli/src/commands/memory.rs:713` `} else {`
- `sources/hindsight/hindsight-cli/src/commands/memory.rs:714` `None`
- `sources/hindsight/hindsight-cli/src/commands/memory.rs:715` `};`
- `sources/hindsight/hindsight-cli/src/commands/memory.rs:716` ``
- `sources/hindsight/hindsight-cli/src/commands/memory.rs:717` `let response = client.delete_memory(agent_id, unit_id, verbose);`
- `sources/hindsight/hindsight-cli/src/commands/memory.rs:718` ``
- `sources/hindsight/hindsight-cli/src/commands/memory.rs:719` `if let Some(mut sp) = spinner {`
- `sources/hindsight/hindsight-cli/src/commands/memory.rs:720` `sp.finish();`
- `sources/hindsight/hindsight-cli/src/commands/memory.rs:721` `}`
- `sources/hindsight/hindsight-cli/src/commands/memory.rs:726` `if result.success {`
- `sources/hindsight/hindsight-cli/src/commands/memory.rs:727` `ui::print_success("Memory unit deleted successfully");`
- `sources/hindsight/hindsight-cli/src/commands/memory.rs:728` `} else {`
- `sources/hindsight/hindsight-cli/src/commands/memory.rs:729` `ui::print_error("Failed to delete memory unit");`
- `sources/hindsight/hindsight-cli/src/commands/memory.rs:730` `}`
- `sources/hindsight/hindsight-cli/src/commands/memory.rs:731` `} else {`
- `sources/hindsight/hindsight-cli/src/commands/memory.rs:732` `output::print_output(&result, output_format)?;`
- `sources/hindsight/hindsight-cli/src/commands/memory.rs:733` `}`

**operation and route call sites**, body `sources/hindsight/hindsight-cli/src/commands/mental_model.rs:442`; body evidence.

- `sources/hindsight/hindsight-cli/src/commands/mental_model.rs:441` `/// Delete a mental model`
- `sources/hindsight/hindsight-cli/src/commands/mental_model.rs:442` `pub fn delete(`
- `sources/hindsight/hindsight-cli/src/commands/mental_model.rs:443` `client: &ApiClient,`
- `sources/hindsight/hindsight-cli/src/commands/mental_model.rs:444` `bank_id: &str,`
- `sources/hindsight/hindsight-cli/src/commands/mental_model.rs:445` `mental_model_id: &str,`
- `sources/hindsight/hindsight-cli/src/commands/mental_model.rs:446` `yes: bool,`
- `sources/hindsight/hindsight-cli/src/commands/mental_model.rs:452` `let message = format!(`
- `sources/hindsight/hindsight-cli/src/commands/mental_model.rs:453` `"Are you sure you want to delete mental model '{}'? This cannot be undone.",`
- `sources/hindsight/hindsight-cli/src/commands/mental_model.rs:454` `mental_model_id`
- `sources/hindsight/hindsight-cli/src/commands/mental_model.rs:455` `);`
- `sources/hindsight/hindsight-cli/src/commands/mental_model.rs:456` ``
- `sources/hindsight/hindsight-cli/src/commands/mental_model.rs:457` `let confirmed = ui::prompt_confirmation(&message)?;`
- `sources/hindsight/hindsight-cli/src/commands/mental_model.rs:465` `let spinner = if output_format == OutputFormat::Pretty {`
- `sources/hindsight/hindsight-cli/src/commands/mental_model.rs:466` `Some(ui::create_spinner("Deleting mental model..."))`
- `sources/hindsight/hindsight-cli/src/commands/mental_model.rs:467` `} else {`
- `sources/hindsight/hindsight-cli/src/commands/mental_model.rs:468` `None`
- `sources/hindsight/hindsight-cli/src/commands/mental_model.rs:469` `};`
- `sources/hindsight/hindsight-cli/src/commands/mental_model.rs:470` ``
- `sources/hindsight/hindsight-cli/src/commands/mental_model.rs:471` `let response = client.delete_mental_model(bank_id, mental_model_id, verbose);`
- `sources/hindsight/hindsight-cli/src/commands/mental_model.rs:472` ``
- `sources/hindsight/hindsight-cli/src/commands/mental_model.rs:473` `if let Some(mut sp) = spinner {`
- `sources/hindsight/hindsight-cli/src/commands/mental_model.rs:474` `sp.finish();`
- `sources/hindsight/hindsight-cli/src/commands/mental_model.rs:475` `}`
- `sources/hindsight/hindsight-cli/src/commands/mental_model.rs:480` `ui::print_success(&format!(`
- `sources/hindsight/hindsight-cli/src/commands/mental_model.rs:481` `"Mental model '{}' deleted successfully",`
- `sources/hindsight/hindsight-cli/src/commands/mental_model.rs:482` `mental_model_id`
- `sources/hindsight/hindsight-cli/src/commands/mental_model.rs:483` `));`
- `sources/hindsight/hindsight-cli/src/commands/mental_model.rs:484` `} else {`
- `sources/hindsight/hindsight-cli/src/commands/mental_model.rs:485` `println!("{{\"success\": true}}");`

**operation and route call sites**, body `sources/hindsight/hindsight-cli/src/commands/operation.rs:220`; body evidence.

- `sources/hindsight/hindsight-cli/src/commands/operation.rs:219` `/// Permanently delete a terminal async operation`
- `sources/hindsight/hindsight-cli/src/commands/operation.rs:220` `pub fn delete(`
- `sources/hindsight/hindsight-cli/src/commands/operation.rs:221` `client: &ApiClient,`
- `sources/hindsight/hindsight-cli/src/commands/operation.rs:222` `bank_id: &str,`
- `sources/hindsight/hindsight-cli/src/commands/operation.rs:223` `operation_id: &str,`
- `sources/hindsight/hindsight-cli/src/commands/operation.rs:224` `yes: bool,`
- `sources/hindsight/hindsight-cli/src/commands/operation.rs:229` `let message = format!(`
- `sources/hindsight/hindsight-cli/src/commands/operation.rs:230` `"Are you sure you want to permanently delete operation '{}'? This cannot be undone.",`
- `sources/hindsight/hindsight-cli/src/commands/operation.rs:231` `operation_id`
- `sources/hindsight/hindsight-cli/src/commands/operation.rs:232` `);`
- `sources/hindsight/hindsight-cli/src/commands/operation.rs:233` `if !ui::prompt_confirmation(&message)? {`
- `sources/hindsight/hindsight-cli/src/commands/operation.rs:234` `ui::print_info("Operation cancelled");`
- `sources/hindsight/hindsight-cli/src/commands/operation.rs:239` `let spinner = if output_format == OutputFormat::Pretty {`
- `sources/hindsight/hindsight-cli/src/commands/operation.rs:240` `Some(ui::create_spinner("Deleting operation..."))`
- `sources/hindsight/hindsight-cli/src/commands/operation.rs:241` `} else {`
- `sources/hindsight/hindsight-cli/src/commands/operation.rs:242` `None`
- `sources/hindsight/hindsight-cli/src/commands/operation.rs:243` `};`
- `sources/hindsight/hindsight-cli/src/commands/operation.rs:244` ``
- `sources/hindsight/hindsight-cli/src/commands/operation.rs:245` `let response = client.delete_operation(bank_id, operation_id, verbose);`
- `sources/hindsight/hindsight-cli/src/commands/operation.rs:246` ``
- `sources/hindsight/hindsight-cli/src/commands/operation.rs:247` `if let Some(mut sp) = spinner {`
- `sources/hindsight/hindsight-cli/src/commands/operation.rs:248` `sp.finish();`
- `sources/hindsight/hindsight-cli/src/commands/operation.rs:249` `}`

**operation and route call sites**, body `sources/hindsight/hindsight-cli/src/commands/webhook.rs:162`; body evidence.

- `sources/hindsight/hindsight-cli/src/commands/webhook.rs:161` `/// Delete a webhook`
- `sources/hindsight/hindsight-cli/src/commands/webhook.rs:162` `pub fn delete(`
- `sources/hindsight/hindsight-cli/src/commands/webhook.rs:163` `client: &ApiClient,`
- `sources/hindsight/hindsight-cli/src/commands/webhook.rs:164` `bank_id: &str,`
- `sources/hindsight/hindsight-cli/src/commands/webhook.rs:165` `webhook_id: &str,`
- `sources/hindsight/hindsight-cli/src/commands/webhook.rs:166` `yes: bool,`
- `sources/hindsight/hindsight-cli/src/commands/webhook.rs:171` `let message = format!(`
- `sources/hindsight/hindsight-cli/src/commands/webhook.rs:172` `"Are you sure you want to delete webhook '{}'? This cannot be undone.",`
- `sources/hindsight/hindsight-cli/src/commands/webhook.rs:173` `webhook_id`
- `sources/hindsight/hindsight-cli/src/commands/webhook.rs:174` `);`
- `sources/hindsight/hindsight-cli/src/commands/webhook.rs:175` `if !ui::prompt_confirmation(&message)? {`
- `sources/hindsight/hindsight-cli/src/commands/webhook.rs:176` `ui::print_info("Operation cancelled");`
- `sources/hindsight/hindsight-cli/src/commands/webhook.rs:181` `let spinner = if output_format == OutputFormat::Pretty {`
- `sources/hindsight/hindsight-cli/src/commands/webhook.rs:182` `Some(ui::create_spinner("Deleting webhook..."))`
- `sources/hindsight/hindsight-cli/src/commands/webhook.rs:183` `} else {`
- `sources/hindsight/hindsight-cli/src/commands/webhook.rs:184` `None`
- `sources/hindsight/hindsight-cli/src/commands/webhook.rs:185` `};`
- `sources/hindsight/hindsight-cli/src/commands/webhook.rs:186` ``
- `sources/hindsight/hindsight-cli/src/commands/webhook.rs:187` `let response = client.delete_webhook(bank_id, webhook_id, verbose);`
- `sources/hindsight/hindsight-cli/src/commands/webhook.rs:188` ``
- `sources/hindsight/hindsight-cli/src/commands/webhook.rs:189` `if let Some(mut sp) = spinner {`
- `sources/hindsight/hindsight-cli/src/commands/webhook.rs:190` `sp.finish();`
- `sources/hindsight/hindsight-cli/src/commands/webhook.rs:191` `}`
- `sources/hindsight/hindsight-cli/src/commands/webhook.rs:195` `if result.success {`
- `sources/hindsight/hindsight-cli/src/commands/webhook.rs:196` `ui::print_success(&format!("Webhook '{}' deleted", webhook_id));`
- `sources/hindsight/hindsight-cli/src/commands/webhook.rs:197` `} else {`
- `sources/hindsight/hindsight-cli/src/commands/webhook.rs:198` `ui::print_error("Failed to delete webhook");`
- `sources/hindsight/hindsight-cli/src/commands/webhook.rs:199` `}`
- `sources/hindsight/hindsight-cli/src/commands/webhook.rs:200` `} else {`
- `sources/hindsight/hindsight-cli/src/commands/webhook.rs:201` `output::print_output(&result, output_format)?;`
- `sources/hindsight/hindsight-cli/src/commands/webhook.rs:202` `}`

**operation and route call sites**, body `sources/hindsight/hindsight-cli/src/config.rs:210`; body evidence.

- `sources/hindsight/hindsight-cli/src/config.rs:209` `/// does not exist.`
- `sources/hindsight/hindsight-cli/src/config.rs:210` `pub fn delete_profile(name: &str) -> Result<PathBuf> {`
- `sources/hindsight/hindsight-cli/src/config.rs:211` `validate_profile_name(name)?;`
- `sources/hindsight/hindsight-cli/src/config.rs:212` `let path = Self::profile_file_path(name)`
- `sources/hindsight/hindsight-cli/src/config.rs:213` `.ok_or_else(|| anyhow::anyhow!("Could not determine home directory"))?;`
- `sources/hindsight/hindsight-cli/src/config.rs:214` `if !path.exists() {`
- `sources/hindsight/hindsight-cli/src/config.rs:217` `fs::remove_file(&path)`
- `sources/hindsight/hindsight-cli/src/config.rs:218` `.with_context(|| format!("Failed to delete profile file: {}", path.display()))?;`
- `sources/hindsight/hindsight-cli/src/config.rs:219` `Ok(path)`
- `sources/hindsight/hindsight-cli/src/config.rs:220` `}`
- `sources/hindsight/hindsight-cli/src/config.rs:221` `}`
- `sources/hindsight/hindsight-cli/src/config.rs:222` ``

**operation and route call sites**, body `sources/hindsight/hindsight-cli/src/main.rs:182`; body evidence.

- `sources/hindsight/hindsight-cli/src/main.rs:181` `/// Delete a profile`
- `sources/hindsight/hindsight-cli/src/main.rs:182` `Delete {`
- `sources/hindsight/hindsight-cli/src/main.rs:183` `/// Profile name`
- `sources/hindsight/hindsight-cli/src/main.rs:184` `name: String,`
- `sources/hindsight/hindsight-cli/src/main.rs:185` `/// Skip confirmation prompt`
- `sources/hindsight/hindsight-cli/src/main.rs:186` `#[arg(short = 'y', long)]`
- `sources/hindsight/hindsight-cli/src/main.rs:292` `/// Delete a bank and all its data`
- `sources/hindsight/hindsight-cli/src/main.rs:293` `Delete {`
- `sources/hindsight/hindsight-cli/src/main.rs:294` `/// Bank ID`
- `sources/hindsight/hindsight-cli/src/main.rs:295` `bank_id: String,`
- `sources/hindsight/hindsight-cli/src/main.rs:296` ``
- `sources/hindsight/hindsight-cli/src/main.rs:297` `/// Skip confirmation prompt`
- `sources/hindsight/hindsight-cli/src/main.rs:697` `/// Delete a memory unit`
- `sources/hindsight/hindsight-cli/src/main.rs:698` `Delete {`
- `sources/hindsight/hindsight-cli/src/main.rs:699` `/// Bank ID`
- `sources/hindsight/hindsight-cli/src/main.rs:700` `bank_id: String,`
- `sources/hindsight/hindsight-cli/src/main.rs:701` ``
- `sources/hindsight/hindsight-cli/src/main.rs:702` `/// Memory unit ID`
- `sources/hindsight/hindsight-cli/src/main.rs:793` `/// Delete a document and all its memory units`
- `sources/hindsight/hindsight-cli/src/main.rs:794` `Delete {`
- `sources/hindsight/hindsight-cli/src/main.rs:795` `/// Bank ID`
- `sources/hindsight/hindsight-cli/src/main.rs:796` `bank_id: String,`
- `sources/hindsight/hindsight-cli/src/main.rs:797` ``
- `sources/hindsight/hindsight-cli/src/main.rs:798` `/// Document ID`
- `sources/hindsight/hindsight-cli/src/main.rs:888` `/// Permanently delete a terminal async operation`
- `sources/hindsight/hindsight-cli/src/main.rs:889` `Delete {`
- `sources/hindsight/hindsight-cli/src/main.rs:890` `/// Bank ID`
- `sources/hindsight/hindsight-cli/src/main.rs:891` `bank_id: String,`
- `sources/hindsight/hindsight-cli/src/main.rs:892` ``
- `sources/hindsight/hindsight-cli/src/main.rs:893` `/// Operation ID`
- `sources/hindsight/hindsight-cli/src/main.rs:956` `/// Delete a webhook`
- `sources/hindsight/hindsight-cli/src/main.rs:957` `Delete {`
- `sources/hindsight/hindsight-cli/src/main.rs:958` `/// Bank ID`
- `sources/hindsight/hindsight-cli/src/main.rs:959` `bank_id: String,`
- `sources/hindsight/hindsight-cli/src/main.rs:960` ``
- `sources/hindsight/hindsight-cli/src/main.rs:961` `/// Webhook ID`
- `sources/hindsight/hindsight-cli/src/main.rs:1195` `/// Delete a mental model`
- `sources/hindsight/hindsight-cli/src/main.rs:1196` `Delete {`
- `sources/hindsight/hindsight-cli/src/main.rs:1197` `/// Bank ID`
- `sources/hindsight/hindsight-cli/src/main.rs:1198` `bank_id: String,`
- `sources/hindsight/hindsight-cli/src/main.rs:1199` ``
- `sources/hindsight/hindsight-cli/src/main.rs:1200` `/// Mental model ID`
- `sources/hindsight/hindsight-cli/src/main.rs:1373` `/// Delete a folder or page and its whole subtree`
- `sources/hindsight/hindsight-cli/src/main.rs:1374` `Delete {`
- `sources/hindsight/hindsight-cli/src/main.rs:1375` `/// Bank ID`
- `sources/hindsight/hindsight-cli/src/main.rs:1376` `bank_id: String,`
- `sources/hindsight/hindsight-cli/src/main.rs:1377` ``
- `sources/hindsight/hindsight-cli/src/main.rs:1378` `/// Folder or page ID`
- `sources/hindsight/hindsight-cli/src/main.rs:1451` `/// Delete a directive`
- `sources/hindsight/hindsight-cli/src/main.rs:1452` `Delete {`
- `sources/hindsight/hindsight-cli/src/main.rs:1453` `/// Bank ID`
- `sources/hindsight/hindsight-cli/src/main.rs:1454` `bank_id: String,`
- `sources/hindsight/hindsight-cli/src/main.rs:1455` ``
- `sources/hindsight/hindsight-cli/src/main.rs:1456` `/// Directive ID`
- `sources/hindsight/hindsight-cli/src/main.rs:1598` `} => commands::bank::graph(&client, &bank_id, fact_type, limit, verbose, output_format),`
- `sources/hindsight/hindsight-cli/src/main.rs:1599` `BankCommands::Delete { bank_id, yes } => {`
- `sources/hindsight/hindsight-cli/src/main.rs:1600` `commands::bank::delete(&client, &bank_id, yes, verbose, output_format)`
- `sources/hindsight/hindsight-cli/src/main.rs:1601` `}`
- `sources/hindsight/hindsight-cli/src/main.rs:1602` `BankCommands::Alias { command } => match command {`
- `sources/hindsight/hindsight-cli/src/main.rs:1603` `BankAliasCommands::List { bank_id } => {`
- `sources/hindsight/hindsight-cli/src/main.rs:1604` `commands::bank::alias_list(&client, &bank_id, verbose, output_format)`
- `sources/hindsight/hindsight-cli/src/main.rs:1849` `),`
- `sources/hindsight/hindsight-cli/src/main.rs:1850` `MemoryCommands::Delete { bank_id, unit_id } => {`
- `sources/hindsight/hindsight-cli/src/main.rs:1851` `commands::memory::delete(&client, &bank_id, &unit_id, verbose, output_format)`
- `sources/hindsight/hindsight-cli/src/main.rs:1852` `}`
- `sources/hindsight/hindsight-cli/src/main.rs:1853` `MemoryCommands::Clear {`
- `sources/hindsight/hindsight-cli/src/main.rs:1854` `bank_id,`
- `sources/hindsight/hindsight-cli/src/main.rs:1855` `fact_type,`
- `sources/hindsight/hindsight-cli/src/main.rs:1909` `),`
- `sources/hindsight/hindsight-cli/src/main.rs:1910` `DocumentCommands::Delete {`
- `sources/hindsight/hindsight-cli/src/main.rs:1911` `bank_id,`
- `sources/hindsight/hindsight-cli/src/main.rs:1912` `document_id,`
- `sources/hindsight/hindsight-cli/src/main.rs:1913` `} => {`
- `sources/hindsight/hindsight-cli/src/main.rs:1914` `commands::document::delete(&client, &bank_id, &document_id, verbose, output_format)`
- `sources/hindsight/hindsight-cli/src/main.rs:1915` `}`
- `sources/hindsight/hindsight-cli/src/main.rs:1916` `DocumentCommands::Update {`
- `sources/hindsight/hindsight-cli/src/main.rs:1917` `bank_id,`
- `sources/hindsight/hindsight-cli/src/main.rs:1918` `document_id,`
- `sources/hindsight/hindsight-cli/src/main.rs:1999` `}`
- `sources/hindsight/hindsight-cli/src/main.rs:2000` `OperationCommands::Delete {`
- `sources/hindsight/hindsight-cli/src/main.rs:2001` `bank_id,`
- `sources/hindsight/hindsight-cli/src/main.rs:2002` `operation_id,`
- `sources/hindsight/hindsight-cli/src/main.rs:2003` `yes,`
- `sources/hindsight/hindsight-cli/src/main.rs:2004` `} => commands::operation::delete(`
- `sources/hindsight/hindsight-cli/src/main.rs:2005` `&client,`
- `sources/hindsight/hindsight-cli/src/main.rs:2006` `&bank_id,`
- `sources/hindsight/hindsight-cli/src/main.rs:2007` `&operation_id,`
- `sources/hindsight/hindsight-cli/src/main.rs:2008` `yes,`
- `sources/hindsight/hindsight-cli/src/main.rs:2088` `),`
- `sources/hindsight/hindsight-cli/src/main.rs:2089` `MentalModelCommands::Delete {`
- `sources/hindsight/hindsight-cli/src/main.rs:2090` `bank_id,`
- `sources/hindsight/hindsight-cli/src/main.rs:2091` `mental_model_id,`
- `sources/hindsight/hindsight-cli/src/main.rs:2092` `yes,`
- `sources/hindsight/hindsight-cli/src/main.rs:2093` `} => commands::mental_model::delete(`
- `sources/hindsight/hindsight-cli/src/main.rs:2094` `&client,`
- `sources/hindsight/hindsight-cli/src/main.rs:2095` `&bank_id,`
- `sources/hindsight/hindsight-cli/src/main.rs:2096` `&mental_model_id,`
- `sources/hindsight/hindsight-cli/src/main.rs:2097` `yes,`
- `sources/hindsight/hindsight-cli/src/main.rs:2227` `),`
- `sources/hindsight/hindsight-cli/src/main.rs:2228` `KnowledgeBaseCommands::Delete {`
- `sources/hindsight/hindsight-cli/src/main.rs:2229` `bank_id,`
- `sources/hindsight/hindsight-cli/src/main.rs:2230` `node_id,`
- `sources/hindsight/hindsight-cli/src/main.rs:2231` `yes,`
- `sources/hindsight/hindsight-cli/src/main.rs:2232` `} => commands::knowledge_base::delete(`
- `sources/hindsight/hindsight-cli/src/main.rs:2233` `&client,`
- `sources/hindsight/hindsight-cli/src/main.rs:2234` `&bank_id,`
- `sources/hindsight/hindsight-cli/src/main.rs:2235` `&node_id,`
- `sources/hindsight/hindsight-cli/src/main.rs:2236` `yes,`
- `sources/hindsight/hindsight-cli/src/main.rs:2285` `),`
- `sources/hindsight/hindsight-cli/src/main.rs:2286` `DirectiveCommands::Delete {`
- `sources/hindsight/hindsight-cli/src/main.rs:2287` `bank_id,`
- `sources/hindsight/hindsight-cli/src/main.rs:2288` `directive_id,`
- `sources/hindsight/hindsight-cli/src/main.rs:2289` `yes,`
- `sources/hindsight/hindsight-cli/src/main.rs:2290` `} => commands::directive::delete(`
- `sources/hindsight/hindsight-cli/src/main.rs:2291` `&client,`
- `sources/hindsight/hindsight-cli/src/main.rs:2292` `&bank_id,`
- `sources/hindsight/hindsight-cli/src/main.rs:2293` `&directive_id,`
- `sources/hindsight/hindsight-cli/src/main.rs:2294` `yes,`
- `sources/hindsight/hindsight-cli/src/main.rs:2338` `),`
- `sources/hindsight/hindsight-cli/src/main.rs:2339` `WebhookCommands::Delete {`
- `sources/hindsight/hindsight-cli/src/main.rs:2340` `bank_id,`
- `sources/hindsight/hindsight-cli/src/main.rs:2341` `webhook_id,`
- `sources/hindsight/hindsight-cli/src/main.rs:2342` `yes,`
- `sources/hindsight/hindsight-cli/src/main.rs:2343` `} => commands::webhook::delete(`
- `sources/hindsight/hindsight-cli/src/main.rs:2344` `&client,`
- `sources/hindsight/hindsight-cli/src/main.rs:2345` `&bank_id,`
- `sources/hindsight/hindsight-cli/src/main.rs:2346` `&webhook_id,`
- `sources/hindsight/hindsight-cli/src/main.rs:2347` `yes,`
- `sources/hindsight/hindsight-cli/src/main.rs:2614` `}`
- `sources/hindsight/hindsight-cli/src/main.rs:2615` `ProfileCommands::Delete { name, yes } => {`
- `sources/hindsight/hindsight-cli/src/main.rs:2616` `let path = Config::profile_file_path(&name)`
- `sources/hindsight/hindsight-cli/src/main.rs:2617` `.ok_or_else(|| anyhow::anyhow!("Could not determine home directory"))?;`
- `sources/hindsight/hindsight-cli/src/main.rs:2618` `if !path.exists() {`
- `sources/hindsight/hindsight-cli/src/main.rs:2619` `anyhow::bail!("profile '{}' not found at {}", name, path.display());`
- `sources/hindsight/hindsight-cli/src/main.rs:2621` `if !yes && output_format == OutputFormat::Pretty {`
- `sources/hindsight/hindsight-cli/src/main.rs:2622` `print!("Delete profile '{}' at {}? [y/N]: ", name, path.display());`
- `sources/hindsight/hindsight-cli/src/main.rs:2623` `std::io::Write::flush(&mut std::io::stdout())?;`
- `sources/hindsight/hindsight-cli/src/main.rs:2624` `let mut input = String::new();`
- `sources/hindsight/hindsight-cli/src/main.rs:2625` `std::io::stdin().read_line(&mut input)?;`
- `sources/hindsight/hindsight-cli/src/main.rs:2626` `if !matches!(input.trim().to_lowercase().as_str(), "y" | "yes") {`
- `sources/hindsight/hindsight-cli/src/main.rs:2630` `}`
- `sources/hindsight/hindsight-cli/src/main.rs:2631` `let deleted = Config::delete_profile(&name)?;`
- `sources/hindsight/hindsight-cli/src/main.rs:2632` `if output_format == OutputFormat::Pretty {`
- `sources/hindsight/hindsight-cli/src/main.rs:2633` `ui::print_success(&format!(`
- `sources/hindsight/hindsight-cli/src/main.rs:2634` `"Deleted profile '{}' ({})",`
- `sources/hindsight/hindsight-cli/src/main.rs:2635` `name,`
- `sources/hindsight/hindsight-cli/src/main.rs:2636` `deleted.display()`
- `sources/hindsight/hindsight-cli/src/main.rs:2637` `));`
- `sources/hindsight/hindsight-cli/src/main.rs:2638` `} else {`
- `sources/hindsight/hindsight-cli/src/main.rs:2639` `output::print_output(`
- `sources/hindsight/hindsight-cli/src/main.rs:2640` `&serde_json::json!({`
- `sources/hindsight/hindsight-cli/src/main.rs:2641` `"name": name,`
- `sources/hindsight/hindsight-cli/src/main.rs:2642` `"path": deleted.display().to_string(),`
- `sources/hindsight/hindsight-cli/src/main.rs:2643` `"deleted": true,`
- `sources/hindsight/hindsight-cli/src/main.rs:2644` `}),`
- `sources/hindsight/hindsight-cli/src/main.rs:2645` `output_format,`
- `sources/hindsight/hindsight-cli/src/main.rs:2646` `)?;`
- `sources/hindsight/hindsight-cli/src/main.rs:2647` `}`
- `sources/hindsight/hindsight-cli/src/main.rs:2658` `#[test]`
- `sources/hindsight/hindsight-cli/src/main.rs:2659` `fn parses_operation_delete_with_confirmation_bypass() {`
- `sources/hindsight/hindsight-cli/src/main.rs:2660` `let cli = Cli::try_parse_from([`
- `sources/hindsight/hindsight-cli/src/main.rs:2661` `"hindsight",`
- `sources/hindsight/hindsight-cli/src/main.rs:2662` `"operation",`
- `sources/hindsight/hindsight-cli/src/main.rs:2663` `"delete",`
- `sources/hindsight/hindsight-cli/src/main.rs:2664` `"bank-1",`
- `sources/hindsight/hindsight-cli/src/main.rs:2665` `"operation-1",`
- `sources/hindsight/hindsight-cli/src/main.rs:2666` `"--yes",`
- `sources/hindsight/hindsight-cli/src/main.rs:2667` `])`
- `sources/hindsight/hindsight-cli/src/main.rs:2668` `.expect("operation delete should be a valid command");`
- `sources/hindsight/hindsight-cli/src/main.rs:2669` ``
- `sources/hindsight/hindsight-cli/src/main.rs:2670` `match cli.command {`
- `sources/hindsight/hindsight-cli/src/main.rs:2671` `Commands::Operation(OperationCommands::Delete {`
- `sources/hindsight/hindsight-cli/src/main.rs:2672` `bank_id,`
- `sources/hindsight/hindsight-cli/src/main.rs:2673` `operation_id,`
- `sources/hindsight/hindsight-cli/src/main.rs:2674` `yes,`
- `sources/hindsight/hindsight-cli/src/main.rs:2675` `}) => {`
- `sources/hindsight/hindsight-cli/src/main.rs:2679` `}`
- `sources/hindsight/hindsight-cli/src/main.rs:2680` `_ => panic!("expected operation delete command"),`
- `sources/hindsight/hindsight-cli/src/main.rs:2681` `}`
- `sources/hindsight/hindsight-cli/src/main.rs:2682` `}`
- `sources/hindsight/hindsight-cli/src/main.rs:2683` `}`

**operation and route call sites**, body `sources/hindsight/hindsight-clients/go/api_bank_transfer.go:77`; body evidence.

- `sources/hindsight/hindsight-clients/go/api_bank_transfer.go:76` ``
- `sources/hindsight/hindsight-clients/go/api_bank_transfer.go:77` `This is the export and import above run back to back on this instance, so nothing is re-extracted and no LLM is called , facts are re-embedded and entities re-resolved, exactly as a restore does. The same three flags choose what the clone inherits: include_data (documents, facts, observations, attachments, the curation archive, the operations log, and the mental models and knowledge pages synthesized from them), include_bank_config (the bank's config overrides, directives and **webhooks**) and include_history (audit_log, llm_requests).`
- `sources/hindsight/hindsight-clients/go/api_bank_transfer.go:78` ``
- `sources/hindsight/hindsight-clients/go/api_bank_transfer.go:79` `Note the webhooks: they travel with the bank's configuration, so a clone made with the default flags will call the source's webhook endpoints. Pass include_bank_config=false, or delete them on the clone, when they point at a per-bank consumer.`
- `sources/hindsight/hindsight-clients/go/api_bank_transfer.go:80` ``
- `sources/hindsight/hindsight-clients/go/api_bank_transfer.go:81` `target_bank_id must not already exist. Returns an operation_id, recorded against the source bank (the target does not exist yet); poll GET /v1/default/banks/{bank_id}/operations/{operation_id} for status and the per-component counts.`
- `sources/hindsight/hindsight-clients/go/api_bank_transfer.go:82` ``
- `sources/hindsight/hindsight-clients/go/api_bank_transfer.go:83` `@param ctx context.Context - for authentication, logging, cancellation, deadlines, tracing, etc. Passed from http.Request or context.Background().`
- `sources/hindsight/hindsight-clients/go/api_bank_transfer.go:252` ``
- `sources/hindsight/hindsight-clients/go/api_bank_transfer.go:253` `Submit an async export of a bank as a transfer ZIP archive. Three flags choose what the archive carries: include_data (documents, facts, observations, attachments and their bytes, the curation archive, the operations log and the maintenance queues), include_bank_config (bank config, mental models and their history, knowledge pages), include_bank_config (the bank's config overrides, directives and webhooks) and include_history (audit_log, llm_requests). Embeddings and database ids are never carried , importing re-embeds with the target bank's model and re-resolves entities, so an archive moves between instances configured with different embedding models. Returns an operation_id; poll GET /v1/default/banks/{bank_id}/operations/{operation_id}, then fetch the archive from the download_url in its result_metadata. Pass document_id to export specific documents instead of the whole bank (a document subset carries no bank-level sections).`
- `sources/hindsight/hindsight-clients/go/api_bank_transfer.go:254` ``
- `sources/hindsight/hindsight-clients/go/api_bank_transfer.go:255` `@param ctx context.Context - for authentication, logging, cancellation, deadlines, tracing, etc. Passed from http.Request or context.Background().`
- `sources/hindsight/hindsight-clients/go/api_bank_transfer.go:256` `@param bankId`
- `sources/hindsight/hindsight-clients/go/api_bank_transfer.go:257` `@return ApiExportBankTransferRequest`
- `sources/hindsight/hindsight-clients/go/api_bank_transfer.go:452` ``
- `sources/hindsight/hindsight-clients/go/api_bank_transfer.go:453` `Submit a transfer archive (produced by the export endpoint) for import. Runs as a background operation: facts are re-embedded with the target bank's embedding model and entities are re-resolved , no LLM extraction, so the import costs no tokens and invents no new facts.`
- `sources/hindsight/hindsight-clients/go/api_bank_transfer.go:454` ``
- `sources/hindsight/hindsight-clients/go/api_bank_transfer.go:455` `Two modes. 'restore' (default) writes a whole bank into target_bank_id, which must NOT already exist , it restores a bank rather than merging into one, and is how a bank is moved between instances or copied under a new id. 'merge' folds an archive's documents into this bank, with document_conflict deciding what happens to ids that already exist (skip, replace, new-id).`
- `sources/hindsight/hindsight-clients/go/api_bank_transfer.go:456` ``
- `sources/hindsight/hindsight-clients/go/api_bank_transfer.go:457` `The include flags narrow what is restored to a subset of what the archive holds; they cannot add what the producer did not export. Returns an operation_id; poll GET /v1/default/banks/{bank_id}/operations/{operation_id} for status and per-component counts. The operation is recorded against {bank_id} even in restore mode, because the target bank does not exist yet.`
- `sources/hindsight/hindsight-clients/go/api_bank_transfer.go:458` ``
- `sources/hindsight/hindsight-clients/go/api_bank_transfer.go:459` `@param ctx context.Context - for authentication, logging, cancellation, deadlines, tracing, etc. Passed from http.Request or context.Background().`
- `sources/hindsight/hindsight-clients/go/api_bank_transfer.go:460` `@param bankId`
- `sources/hindsight/hindsight-clients/go/api_bank_transfer.go:461` `@return ApiImportBankTransferRequest`

**operation and route call sites**, body `sources/hindsight/hindsight-clients/go/api_banks.go:174`; body evidence.

- `sources/hindsight/hindsight-clients/go/api_banks.go:173` ``
- `sources/hindsight/hindsight-clients/go/api_banks.go:174` `func (r ApiClearObservationsRequest) Execute() (*DeleteResponse, *http.Response, error) {`
- `sources/hindsight/hindsight-clients/go/api_banks.go:175` `return r.ApiService.ClearObservationsExecute(r)`
- `sources/hindsight/hindsight-clients/go/api_banks.go:176` `}`
- `sources/hindsight/hindsight-clients/go/api_banks.go:177` ``
- `sources/hindsight/hindsight-clients/go/api_banks.go:178` `/*`
- `sources/hindsight/hindsight-clients/go/api_banks.go:180` ``
- `sources/hindsight/hindsight-clients/go/api_banks.go:181` `Delete all observations for a memory bank. This is useful for resetting the consolidated knowledge.`
- `sources/hindsight/hindsight-clients/go/api_banks.go:182` ``
- `sources/hindsight/hindsight-clients/go/api_banks.go:183` `@param ctx context.Context - for authentication, logging, cancellation, deadlines, tracing, etc. Passed from http.Request or context.Background().`
- `sources/hindsight/hindsight-clients/go/api_banks.go:184` `@param bankId`
- `sources/hindsight/hindsight-clients/go/api_banks.go:185` `@return ApiClearObservationsRequest`
- `sources/hindsight/hindsight-clients/go/api_banks.go:196` `//  @return DeleteResponse`
- `sources/hindsight/hindsight-clients/go/api_banks.go:197` `func (a *BanksAPIService) ClearObservationsExecute(r ApiClearObservationsRequest) (*DeleteResponse, *http.Response, error) {`
- `sources/hindsight/hindsight-clients/go/api_banks.go:198` `var (`
- `sources/hindsight/hindsight-clients/go/api_banks.go:199` `localVarHTTPMethod   = http.MethodDelete`
- `sources/hindsight/hindsight-clients/go/api_banks.go:200` `localVarPostBody     interface{}`
- `sources/hindsight/hindsight-clients/go/api_banks.go:201` `formFiles            []formFile`
- `sources/hindsight/hindsight-clients/go/api_banks.go:202` `localVarReturnValue  *DeleteResponse`
- `sources/hindsight/hindsight-clients/go/api_banks.go:203` `)`
- `sources/hindsight/hindsight-clients/go/api_banks.go:204` ``
- `sources/hindsight/hindsight-clients/go/api_banks.go:205` `localBasePath, err := a.client.cfg.ServerURLWithContext(r.ctx, "BanksAPIService.ClearObservations")`
- `sources/hindsight/hindsight-clients/go/api_banks.go:206` `if err != nil {`
- `sources/hindsight/hindsight-clients/go/api_banks.go:551` ``
- `sources/hindsight/hindsight-clients/go/api_banks.go:552` `type ApiDeleteBankRequest struct {`
- `sources/hindsight/hindsight-clients/go/api_banks.go:553` `ctx context.Context`
- `sources/hindsight/hindsight-clients/go/api_banks.go:554` `ApiService *BanksAPIService`
- `sources/hindsight/hindsight-clients/go/api_banks.go:555` `bankId string`
- `sources/hindsight/hindsight-clients/go/api_banks.go:556` `authorization *string`
- `sources/hindsight/hindsight-clients/go/api_banks.go:558` ``
- `sources/hindsight/hindsight-clients/go/api_banks.go:559` `func (r ApiDeleteBankRequest) Authorization(authorization string) ApiDeleteBankRequest {`
- `sources/hindsight/hindsight-clients/go/api_banks.go:560` `r.authorization = &authorization`
- `sources/hindsight/hindsight-clients/go/api_banks.go:561` `return r`
- `sources/hindsight/hindsight-clients/go/api_banks.go:562` `}`
- `sources/hindsight/hindsight-clients/go/api_banks.go:563` ``
- `sources/hindsight/hindsight-clients/go/api_banks.go:564` `func (r ApiDeleteBankRequest) Execute() (*DeleteResponse, *http.Response, error) {`
- `sources/hindsight/hindsight-clients/go/api_banks.go:565` `return r.ApiService.DeleteBankExecute(r)`
- `sources/hindsight/hindsight-clients/go/api_banks.go:566` `}`
- `sources/hindsight/hindsight-clients/go/api_banks.go:567` ``
- `sources/hindsight/hindsight-clients/go/api_banks.go:568` `/*`
- `sources/hindsight/hindsight-clients/go/api_banks.go:569` `DeleteBank Delete memory bank`
- `sources/hindsight/hindsight-clients/go/api_banks.go:570` ``
- `sources/hindsight/hindsight-clients/go/api_banks.go:571` `Delete an entire memory bank including all memories, entities, documents, and the bank profile itself. This is a destructive operation that cannot be undone.`
- `sources/hindsight/hindsight-clients/go/api_banks.go:572` ``
- `sources/hindsight/hindsight-clients/go/api_banks.go:573` `@param ctx context.Context - for authentication, logging, cancellation, deadlines, tracing, etc. Passed from http.Request or context.Background().`
- `sources/hindsight/hindsight-clients/go/api_banks.go:574` `@param bankId`
- `sources/hindsight/hindsight-clients/go/api_banks.go:575` `@return ApiDeleteBankRequest`
- `sources/hindsight/hindsight-clients/go/api_banks.go:576` `*/`
- `sources/hindsight/hindsight-clients/go/api_banks.go:577` `func (a *BanksAPIService) DeleteBank(ctx context.Context, bankId string) ApiDeleteBankRequest {`
- `sources/hindsight/hindsight-clients/go/api_banks.go:578` `return ApiDeleteBankRequest{`
- `sources/hindsight/hindsight-clients/go/api_banks.go:579` `ApiService: a,`
- `sources/hindsight/hindsight-clients/go/api_banks.go:580` `ctx: ctx,`
- `sources/hindsight/hindsight-clients/go/api_banks.go:581` `bankId: bankId,`
- `sources/hindsight/hindsight-clients/go/api_banks.go:582` `}`
- `sources/hindsight/hindsight-clients/go/api_banks.go:586` `//  @return DeleteResponse`
- `sources/hindsight/hindsight-clients/go/api_banks.go:587` `func (a *BanksAPIService) DeleteBankExecute(r ApiDeleteBankRequest) (*DeleteResponse, *http.Response, error) {`
- `sources/hindsight/hindsight-clients/go/api_banks.go:588` `var (`
- `sources/hindsight/hindsight-clients/go/api_banks.go:589` `localVarHTTPMethod   = http.MethodDelete`
- `sources/hindsight/hindsight-clients/go/api_banks.go:590` `localVarPostBody     interface{}`
- `sources/hindsight/hindsight-clients/go/api_banks.go:591` `formFiles            []formFile`
- `sources/hindsight/hindsight-clients/go/api_banks.go:592` `localVarReturnValue  *DeleteResponse`
- `sources/hindsight/hindsight-clients/go/api_banks.go:593` `)`
- `sources/hindsight/hindsight-clients/go/api_banks.go:594` ``
- `sources/hindsight/hindsight-clients/go/api_banks.go:595` `localBasePath, err := a.client.cfg.ServerURLWithContext(r.ctx, "BanksAPIService.DeleteBank")`
- `sources/hindsight/hindsight-clients/go/api_banks.go:596` `if err != nil {`
- `sources/hindsight/hindsight-clients/go/api_banks.go:597` `return localVarReturnValue, nil, &GenericOpenAPIError{error: err.Error()}`
- `sources/hindsight/hindsight-clients/go/api_banks.go:598` `}`
- `sources/hindsight/hindsight-clients/go/api_banks.go:599` ``
- `sources/hindsight/hindsight-clients/go/api_banks.go:673` ``
- `sources/hindsight/hindsight-clients/go/api_banks.go:674` `type ApiDeleteBankAliasRequest struct {`
- `sources/hindsight/hindsight-clients/go/api_banks.go:675` `ctx context.Context`
- `sources/hindsight/hindsight-clients/go/api_banks.go:676` `ApiService *BanksAPIService`
- `sources/hindsight/hindsight-clients/go/api_banks.go:677` `bankId string`
- `sources/hindsight/hindsight-clients/go/api_banks.go:678` `alias string`
- `sources/hindsight/hindsight-clients/go/api_banks.go:681` ``
- `sources/hindsight/hindsight-clients/go/api_banks.go:682` `func (r ApiDeleteBankAliasRequest) Authorization(authorization string) ApiDeleteBankAliasRequest {`
- `sources/hindsight/hindsight-clients/go/api_banks.go:683` `r.authorization = &authorization`
- `sources/hindsight/hindsight-clients/go/api_banks.go:684` `return r`
- `sources/hindsight/hindsight-clients/go/api_banks.go:685` `}`
- `sources/hindsight/hindsight-clients/go/api_banks.go:686` ``
- `sources/hindsight/hindsight-clients/go/api_banks.go:687` `func (r ApiDeleteBankAliasRequest) Execute() (*BankAliasesResponse, *http.Response, error) {`
- `sources/hindsight/hindsight-clients/go/api_banks.go:688` `return r.ApiService.DeleteBankAliasExecute(r)`
- `sources/hindsight/hindsight-clients/go/api_banks.go:689` `}`
- `sources/hindsight/hindsight-clients/go/api_banks.go:690` ``
- `sources/hindsight/hindsight-clients/go/api_banks.go:691` `/*`
- `sources/hindsight/hindsight-clients/go/api_banks.go:692` `DeleteBankAlias Remove an alias from the bank`
- `sources/hindsight/hindsight-clients/go/api_banks.go:693` ``
- `sources/hindsight/hindsight-clients/go/api_banks.go:694` `Stop an id reaching this bank. The bank and its memories are untouched; only the extra name goes away, and callers still using it get the same 404 (or new empty bank) they would have got before it existed.`
- `sources/hindsight/hindsight-clients/go/api_banks.go:695` ``
- `sources/hindsight/hindsight-clients/go/api_banks.go:696` `@param ctx context.Context - for authentication, logging, cancellation, deadlines, tracing, etc. Passed from http.Request or context.Background().`
- `sources/hindsight/hindsight-clients/go/api_banks.go:698` `@param alias`
- `sources/hindsight/hindsight-clients/go/api_banks.go:699` `@return ApiDeleteBankAliasRequest`
- `sources/hindsight/hindsight-clients/go/api_banks.go:700` `*/`
- `sources/hindsight/hindsight-clients/go/api_banks.go:701` `func (a *BanksAPIService) DeleteBankAlias(ctx context.Context, bankId string, alias string) ApiDeleteBankAliasRequest {`
- `sources/hindsight/hindsight-clients/go/api_banks.go:702` `return ApiDeleteBankAliasRequest{`
- `sources/hindsight/hindsight-clients/go/api_banks.go:703` `ApiService: a,`
- `sources/hindsight/hindsight-clients/go/api_banks.go:704` `ctx: ctx,`
- `sources/hindsight/hindsight-clients/go/api_banks.go:705` `bankId: bankId,`
- `sources/hindsight/hindsight-clients/go/api_banks.go:706` `alias: alias,`
- `sources/hindsight/hindsight-clients/go/api_banks.go:711` `//  @return BankAliasesResponse`
- `sources/hindsight/hindsight-clients/go/api_banks.go:712` `func (a *BanksAPIService) DeleteBankAliasExecute(r ApiDeleteBankAliasRequest) (*BankAliasesResponse, *http.Response, error) {`
- `sources/hindsight/hindsight-clients/go/api_banks.go:713` `var (`
- `sources/hindsight/hindsight-clients/go/api_banks.go:714` `localVarHTTPMethod   = http.MethodDelete`
- `sources/hindsight/hindsight-clients/go/api_banks.go:715` `localVarPostBody     interface{}`
- `sources/hindsight/hindsight-clients/go/api_banks.go:716` `formFiles            []formFile`
- `sources/hindsight/hindsight-clients/go/api_banks.go:717` `localVarReturnValue  *BankAliasesResponse`
- `sources/hindsight/hindsight-clients/go/api_banks.go:718` `)`
- `sources/hindsight/hindsight-clients/go/api_banks.go:719` ``
- `sources/hindsight/hindsight-clients/go/api_banks.go:720` `localBasePath, err := a.client.cfg.ServerURLWithContext(r.ctx, "BanksAPIService.DeleteBankAlias")`
- `sources/hindsight/hindsight-clients/go/api_banks.go:721` `if err != nil {`
- `sources/hindsight/hindsight-clients/go/api_banks.go:722` `return localVarReturnValue, nil, &GenericOpenAPIError{error: err.Error()}`
- `sources/hindsight/hindsight-clients/go/api_banks.go:723` `}`
- `sources/hindsight/hindsight-clients/go/api_banks.go:724` ``
- `sources/hindsight/hindsight-clients/go/api_banks.go:1756` ``
- `sources/hindsight/hindsight-clients/go/api_banks.go:1757` `Reset all memories that were permanently marked as failed during consolidation (after exhausting all LLM retries and adaptive batch splitting) so they are picked up again on the next consolidation run. Does not delete any observations.`
- `sources/hindsight/hindsight-clients/go/api_banks.go:1758` ``
- `sources/hindsight/hindsight-clients/go/api_banks.go:1759` `@param ctx context.Context - for authentication, logging, cancellation, deadlines, tracing, etc. Passed from http.Request or context.Background().`
- `sources/hindsight/hindsight-clients/go/api_banks.go:1760` `@param bankId`
- `sources/hindsight/hindsight-clients/go/api_banks.go:1761` `@return ApiRecoverConsolidationRequest`
- `sources/hindsight/hindsight-clients/go/api_banks.go:1896` `var (`
- `sources/hindsight/hindsight-clients/go/api_banks.go:1897` `localVarHTTPMethod   = http.MethodDelete`
- `sources/hindsight/hindsight-clients/go/api_banks.go:1898` `localVarPostBody     interface{}`
- `sources/hindsight/hindsight-clients/go/api_banks.go:1899` `formFiles            []formFile`
- `sources/hindsight/hindsight-clients/go/api_banks.go:1900` `localVarReturnValue  *BankConfigResponse`
- `sources/hindsight/hindsight-clients/go/api_banks.go:1901` `)`

**operation and route call sites**, body `sources/hindsight/hindsight-clients/go/api_directives.go:160`; body evidence.

- `sources/hindsight/hindsight-clients/go/api_directives.go:159` ``
- `sources/hindsight/hindsight-clients/go/api_directives.go:160` `type ApiDeleteDirectiveRequest struct {`
- `sources/hindsight/hindsight-clients/go/api_directives.go:161` `ctx context.Context`
- `sources/hindsight/hindsight-clients/go/api_directives.go:162` `ApiService *DirectivesAPIService`
- `sources/hindsight/hindsight-clients/go/api_directives.go:163` `bankId string`
- `sources/hindsight/hindsight-clients/go/api_directives.go:164` `directiveId string`
- `sources/hindsight/hindsight-clients/go/api_directives.go:167` ``
- `sources/hindsight/hindsight-clients/go/api_directives.go:168` `func (r ApiDeleteDirectiveRequest) Authorization(authorization string) ApiDeleteDirectiveRequest {`
- `sources/hindsight/hindsight-clients/go/api_directives.go:169` `r.authorization = &authorization`
- `sources/hindsight/hindsight-clients/go/api_directives.go:170` `return r`
- `sources/hindsight/hindsight-clients/go/api_directives.go:171` `}`
- `sources/hindsight/hindsight-clients/go/api_directives.go:172` ``
- `sources/hindsight/hindsight-clients/go/api_directives.go:173` `func (r ApiDeleteDirectiveRequest) Execute() (interface{}, *http.Response, error) {`
- `sources/hindsight/hindsight-clients/go/api_directives.go:174` `return r.ApiService.DeleteDirectiveExecute(r)`
- `sources/hindsight/hindsight-clients/go/api_directives.go:175` `}`
- `sources/hindsight/hindsight-clients/go/api_directives.go:176` ``
- `sources/hindsight/hindsight-clients/go/api_directives.go:177` `/*`
- `sources/hindsight/hindsight-clients/go/api_directives.go:178` `DeleteDirective Delete directive`
- `sources/hindsight/hindsight-clients/go/api_directives.go:179` ``
- `sources/hindsight/hindsight-clients/go/api_directives.go:180` `Delete a directive.`
- `sources/hindsight/hindsight-clients/go/api_directives.go:181` ``
- `sources/hindsight/hindsight-clients/go/api_directives.go:182` `@param ctx context.Context - for authentication, logging, cancellation, deadlines, tracing, etc. Passed from http.Request or context.Background().`
- `sources/hindsight/hindsight-clients/go/api_directives.go:183` `@param bankId`
- `sources/hindsight/hindsight-clients/go/api_directives.go:184` `@param directiveId`
- `sources/hindsight/hindsight-clients/go/api_directives.go:185` `@return ApiDeleteDirectiveRequest`
- `sources/hindsight/hindsight-clients/go/api_directives.go:186` `*/`
- `sources/hindsight/hindsight-clients/go/api_directives.go:187` `func (a *DirectivesAPIService) DeleteDirective(ctx context.Context, bankId string, directiveId string) ApiDeleteDirectiveRequest {`
- `sources/hindsight/hindsight-clients/go/api_directives.go:188` `return ApiDeleteDirectiveRequest{`
- `sources/hindsight/hindsight-clients/go/api_directives.go:189` `ApiService: a,`
- `sources/hindsight/hindsight-clients/go/api_directives.go:190` `ctx: ctx,`
- `sources/hindsight/hindsight-clients/go/api_directives.go:191` `bankId: bankId,`
- `sources/hindsight/hindsight-clients/go/api_directives.go:192` `directiveId: directiveId,`
- `sources/hindsight/hindsight-clients/go/api_directives.go:197` `//  @return interface{}`
- `sources/hindsight/hindsight-clients/go/api_directives.go:198` `func (a *DirectivesAPIService) DeleteDirectiveExecute(r ApiDeleteDirectiveRequest) (interface{}, *http.Response, error) {`
- `sources/hindsight/hindsight-clients/go/api_directives.go:199` `var (`
- `sources/hindsight/hindsight-clients/go/api_directives.go:200` `localVarHTTPMethod   = http.MethodDelete`
- `sources/hindsight/hindsight-clients/go/api_directives.go:201` `localVarPostBody     interface{}`
- `sources/hindsight/hindsight-clients/go/api_directives.go:202` `formFiles            []formFile`
- `sources/hindsight/hindsight-clients/go/api_directives.go:203` `localVarReturnValue  interface{}`
- `sources/hindsight/hindsight-clients/go/api_directives.go:204` `)`
- `sources/hindsight/hindsight-clients/go/api_directives.go:205` ``
- `sources/hindsight/hindsight-clients/go/api_directives.go:206` `localBasePath, err := a.client.cfg.ServerURLWithContext(r.ctx, "DirectivesAPIService.DeleteDirective")`
- `sources/hindsight/hindsight-clients/go/api_directives.go:207` `if err != nil {`
- `sources/hindsight/hindsight-clients/go/api_directives.go:208` `return localVarReturnValue, nil, &GenericOpenAPIError{error: err.Error()}`
- `sources/hindsight/hindsight-clients/go/api_directives.go:209` `}`
- `sources/hindsight/hindsight-clients/go/api_directives.go:210` ``

**operation and route call sites**, body `sources/hindsight/hindsight-clients/go/api_document_transfer.go:45`; body evidence.

- `sources/hindsight/hindsight-clients/go/api_document_transfer.go:44` `/*`
- `sources/hindsight/hindsight-clients/go/api_document_transfer.go:45` `DownloadFile Download a stored file (async export archive)`
- `sources/hindsight/hindsight-clients/go/api_document_transfer.go:46` ``
- `sources/hindsight/hindsight-clients/go/api_document_transfer.go:47` `Stream a file previously written to file storage , currently the transfer ZIP produced by an async document export. The key comes from the export operation's result_metadata (storage_key / download_url). Access is authorized against the bank the key belongs to.`
- `sources/hindsight/hindsight-clients/go/api_document_transfer.go:48` ``
- `sources/hindsight/hindsight-clients/go/api_document_transfer.go:49` `@param ctx context.Context - for authentication, logging, cancellation, deadlines, tracing, etc. Passed from http.Request or context.Background().`
- `sources/hindsight/hindsight-clients/go/api_document_transfer.go:189` ``
- `sources/hindsight/hindsight-clients/go/api_document_transfer.go:190` `Submit an async export of a bank's documents (extracted facts, entity names, causal links, chunks) as a transfer ZIP archive. Embeddings and database ids are not included , importing re-embeds with the target bank's model and re-resolves entities. Runs as a background operation to avoid pinning the API on large banks. Returns an operation_id; poll GET /v1/default/banks/{bank_id}/operations/{operation_id}. On completion the operation's result_metadata carries download_url (fetch the ZIP from GET /v1/default/files/download/{key}), storage_key, byte_size, and filename. Pass document_id query params to export specific documents, or omit to export the whole bank; include_observations=true carries consolidated observations and include_knowledge_base=true carries Mental Models plus Knowledge Pages (all whole-bank export only).`
- `sources/hindsight/hindsight-clients/go/api_document_transfer.go:191` ``
- `sources/hindsight/hindsight-clients/go/api_document_transfer.go:192` `@param ctx context.Context - for authentication, logging, cancellation, deadlines, tracing, etc. Passed from http.Request or context.Background().`
- `sources/hindsight/hindsight-clients/go/api_document_transfer.go:193` `@param bankId`
- `sources/hindsight/hindsight-clients/go/api_document_transfer.go:194` `@return ApiExportDocumentsRequest`
- `sources/hindsight/hindsight-clients/go/api_document_transfer.go:476` ``
- `sources/hindsight/hindsight-clients/go/api_document_transfer.go:477` `Submit a transfer archive (produced by the export endpoint) for import into a bank. Runs as a background operation: facts are re-embedded with the target bank's embedding model and entities are re-resolved , no LLM extraction. Returns an operation_id; poll GET /v1/default/banks/{bank_id}/operations/{operation_id} for status and the imported/skipped counts in result_metadata. Use on_conflict to control existing document ids: skip (default), replace, or new-id.`
- `sources/hindsight/hindsight-clients/go/api_document_transfer.go:478` ``
- `sources/hindsight/hindsight-clients/go/api_document_transfer.go:479` `@param ctx context.Context - for authentication, logging, cancellation, deadlines, tracing, etc. Passed from http.Request or context.Background().`
- `sources/hindsight/hindsight-clients/go/api_document_transfer.go:480` `@param bankId`
- `sources/hindsight/hindsight-clients/go/api_document_transfer.go:481` `@return ApiImportDocumentsRequest`

**operation and route call sites**, body `sources/hindsight/hindsight-clients/go/api_documents.go:27`; body evidence.

- `sources/hindsight/hindsight-clients/go/api_documents.go:26` ``
- `sources/hindsight/hindsight-clients/go/api_documents.go:27` `type ApiDeleteDocumentRequest struct {`
- `sources/hindsight/hindsight-clients/go/api_documents.go:28` `ctx context.Context`
- `sources/hindsight/hindsight-clients/go/api_documents.go:29` `ApiService *DocumentsAPIService`
- `sources/hindsight/hindsight-clients/go/api_documents.go:30` `bankId string`
- `sources/hindsight/hindsight-clients/go/api_documents.go:31` `documentId string`
- `sources/hindsight/hindsight-clients/go/api_documents.go:34` ``
- `sources/hindsight/hindsight-clients/go/api_documents.go:35` `func (r ApiDeleteDocumentRequest) Authorization(authorization string) ApiDeleteDocumentRequest {`
- `sources/hindsight/hindsight-clients/go/api_documents.go:36` `r.authorization = &authorization`
- `sources/hindsight/hindsight-clients/go/api_documents.go:37` `return r`
- `sources/hindsight/hindsight-clients/go/api_documents.go:38` `}`
- `sources/hindsight/hindsight-clients/go/api_documents.go:39` ``
- `sources/hindsight/hindsight-clients/go/api_documents.go:40` `func (r ApiDeleteDocumentRequest) Execute() (*DeleteDocumentResponse, *http.Response, error) {`
- `sources/hindsight/hindsight-clients/go/api_documents.go:41` `return r.ApiService.DeleteDocumentExecute(r)`
- `sources/hindsight/hindsight-clients/go/api_documents.go:42` `}`
- `sources/hindsight/hindsight-clients/go/api_documents.go:43` ``
- `sources/hindsight/hindsight-clients/go/api_documents.go:44` `/*`
- `sources/hindsight/hindsight-clients/go/api_documents.go:45` `DeleteDocument Delete a document`
- `sources/hindsight/hindsight-clients/go/api_documents.go:46` ``
- `sources/hindsight/hindsight-clients/go/api_documents.go:47` `Delete a document and all its associated memory units and links.`
- `sources/hindsight/hindsight-clients/go/api_documents.go:48` ``
- `sources/hindsight/hindsight-clients/go/api_documents.go:49` `This will cascade delete:`
- `sources/hindsight/hindsight-clients/go/api_documents.go:50` `- The document itself`
- `sources/hindsight/hindsight-clients/go/api_documents.go:51` `- All memory units extracted from this document`
- `sources/hindsight/hindsight-clients/go/api_documents.go:52` `- All links (temporal, semantic, entity) associated with those memory units`
- `sources/hindsight/hindsight-clients/go/api_documents.go:53` ``
- `sources/hindsight/hindsight-clients/go/api_documents.go:58` `@param documentId`
- `sources/hindsight/hindsight-clients/go/api_documents.go:59` `@return ApiDeleteDocumentRequest`
- `sources/hindsight/hindsight-clients/go/api_documents.go:60` `*/`
- `sources/hindsight/hindsight-clients/go/api_documents.go:61` `func (a *DocumentsAPIService) DeleteDocument(ctx context.Context, bankId string, documentId string) ApiDeleteDocumentRequest {`
- `sources/hindsight/hindsight-clients/go/api_documents.go:62` `return ApiDeleteDocumentRequest{`
- `sources/hindsight/hindsight-clients/go/api_documents.go:63` `ApiService: a,`
- `sources/hindsight/hindsight-clients/go/api_documents.go:64` `ctx: ctx,`
- `sources/hindsight/hindsight-clients/go/api_documents.go:65` `bankId: bankId,`
- `sources/hindsight/hindsight-clients/go/api_documents.go:66` `documentId: documentId,`
- `sources/hindsight/hindsight-clients/go/api_documents.go:71` `//  @return DeleteDocumentResponse`
- `sources/hindsight/hindsight-clients/go/api_documents.go:72` `func (a *DocumentsAPIService) DeleteDocumentExecute(r ApiDeleteDocumentRequest) (*DeleteDocumentResponse, *http.Response, error) {`
- `sources/hindsight/hindsight-clients/go/api_documents.go:73` `var (`
- `sources/hindsight/hindsight-clients/go/api_documents.go:74` `localVarHTTPMethod   = http.MethodDelete`
- `sources/hindsight/hindsight-clients/go/api_documents.go:75` `localVarPostBody     interface{}`
- `sources/hindsight/hindsight-clients/go/api_documents.go:76` `formFiles            []formFile`
- `sources/hindsight/hindsight-clients/go/api_documents.go:77` `localVarReturnValue  *DeleteDocumentResponse`
- `sources/hindsight/hindsight-clients/go/api_documents.go:78` `)`
- `sources/hindsight/hindsight-clients/go/api_documents.go:79` ``
- `sources/hindsight/hindsight-clients/go/api_documents.go:80` `localBasePath, err := a.client.cfg.ServerURLWithContext(r.ctx, "DocumentsAPIService.DeleteDocument")`
- `sources/hindsight/hindsight-clients/go/api_documents.go:81` `if err != nil {`
- `sources/hindsight/hindsight-clients/go/api_documents.go:82` `return localVarReturnValue, nil, &GenericOpenAPIError{error: err.Error()}`
- `sources/hindsight/hindsight-clients/go/api_documents.go:83` `}`
- `sources/hindsight/hindsight-clients/go/api_documents.go:84` ``
- `sources/hindsight/hindsight-clients/go/api_documents.go:441` ``
- `sources/hindsight/hindsight-clients/go/api_documents.go:442` `What each change to a document did, newest first: every retain, append and reprocess (facts added and removed), every fact edit, invalidation, restore, delete, retag and per-fact observation clear (observations removed), and every consolidation run (observations created, updated and removed). Observations are rebuilt, not versioned, so this is the only record of the ones a change removed. Ids and counts only, with each id list capped at 100.`
- `sources/hindsight/hindsight-clients/go/api_documents.go:443` ``
- `sources/hindsight/hindsight-clients/go/api_documents.go:444` `Rebuilt from the bank's async operations, which carry these events: it reaches back only as far as operations are kept (HINDSIGHT_API_OPERATION_RETENTION_DAYS, unlimited by default), and a change that ran without an operation is not in it , a synchronous retain (async=false), or a fact edit, delete or retag while auto-consolidation is off. A fact edit, delete or retag is recorded on the consolidation operation it queues. Clearing every observation in the bank is not listed. total, limit and offset count operations, not events.`
- `sources/hindsight/hindsight-clients/go/api_documents.go:445` ``
- `sources/hindsight/hindsight-clients/go/api_documents.go:446` `@param ctx context.Context - for authentication, logging, cancellation, deadlines, tracing, etc. Passed from http.Request or context.Background().`
- `sources/hindsight/hindsight-clients/go/api_documents.go:447` `@param bankId`
- `sources/hindsight/hindsight-clients/go/api_documents.go:448` `@param documentId`
- `sources/hindsight/hindsight-clients/go/api_documents.go:950` ``
- `sources/hindsight/hindsight-clients/go/api_documents.go:951` `Re-run the retain pipeline on an existing document without changing its content. This deletes the existing memory units and re-extracts facts using the current engine configuration. Useful when the LLM model, chunking strategy, or extraction settings have changed.`
- `sources/hindsight/hindsight-clients/go/api_documents.go:952` ``
- `sources/hindsight/hindsight-clients/go/api_documents.go:953` `@param ctx context.Context - for authentication, logging, cancellation, deadlines, tracing, etc. Passed from http.Request or context.Background().`
- `sources/hindsight/hindsight-clients/go/api_documents.go:954` `@param bankId`
- `sources/hindsight/hindsight-clients/go/api_documents.go:955` `@param documentId`
- `sources/hindsight/hindsight-clients/go/api_documents.go:1090` ``
- `sources/hindsight/hindsight-clients/go/api_documents.go:1091` `A memory is re-consolidated only when the change moves the observation scopes it resolves to: its observations are deleted and it is queued again, along with co-source memories from other documents that shared those observations. A change that resolves to the same scopes (a tag change under 'shared', 'combined' over one tag vs '[[that tag]]', re-sending the current values) keeps every observation and queues nothing.`
- `sources/hindsight/hindsight-clients/go/api_documents.go:1092` ``
- `sources/hindsight/hindsight-clients/go/api_documents.go:1093` `At least one field must be provided.`
- `sources/hindsight/hindsight-clients/go/api_documents.go:1094` ``
- `sources/hindsight/hindsight-clients/go/api_documents.go:1095` `@param ctx context.Context - for authentication, logging, cancellation, deadlines, tracing, etc. Passed from http.Request or context.Background().`

**operation and route call sites**, body `sources/hindsight/hindsight-clients/go/api_knowledge_base.go:293`; body evidence.

- `sources/hindsight/hindsight-clients/go/api_knowledge_base.go:292` ``
- `sources/hindsight/hindsight-clients/go/api_knowledge_base.go:293` `type ApiDeleteKnowledgeNodeRequest struct {`
- `sources/hindsight/hindsight-clients/go/api_knowledge_base.go:294` `ctx context.Context`
- `sources/hindsight/hindsight-clients/go/api_knowledge_base.go:295` `ApiService *KnowledgeBaseAPIService`
- `sources/hindsight/hindsight-clients/go/api_knowledge_base.go:296` `bankId string`
- `sources/hindsight/hindsight-clients/go/api_knowledge_base.go:297` `nodeId string`
- `sources/hindsight/hindsight-clients/go/api_knowledge_base.go:300` ``
- `sources/hindsight/hindsight-clients/go/api_knowledge_base.go:301` `func (r ApiDeleteKnowledgeNodeRequest) Authorization(authorization string) ApiDeleteKnowledgeNodeRequest {`
- `sources/hindsight/hindsight-clients/go/api_knowledge_base.go:302` `r.authorization = &authorization`
- `sources/hindsight/hindsight-clients/go/api_knowledge_base.go:303` `return r`
- `sources/hindsight/hindsight-clients/go/api_knowledge_base.go:304` `}`
- `sources/hindsight/hindsight-clients/go/api_knowledge_base.go:305` ``
- `sources/hindsight/hindsight-clients/go/api_knowledge_base.go:306` `func (r ApiDeleteKnowledgeNodeRequest) Execute() (interface{}, *http.Response, error) {`
- `sources/hindsight/hindsight-clients/go/api_knowledge_base.go:307` `return r.ApiService.DeleteKnowledgeNodeExecute(r)`
- `sources/hindsight/hindsight-clients/go/api_knowledge_base.go:308` `}`
- `sources/hindsight/hindsight-clients/go/api_knowledge_base.go:309` ``
- `sources/hindsight/hindsight-clients/go/api_knowledge_base.go:310` `/*`
- `sources/hindsight/hindsight-clients/go/api_knowledge_base.go:311` `DeleteKnowledgeNode Delete a knowledge-base node`
- `sources/hindsight/hindsight-clients/go/api_knowledge_base.go:312` ``
- `sources/hindsight/hindsight-clients/go/api_knowledge_base.go:313` `Delete a folder or page and its whole subtree (pages' mental models are removed too).`
- `sources/hindsight/hindsight-clients/go/api_knowledge_base.go:314` ``
- `sources/hindsight/hindsight-clients/go/api_knowledge_base.go:315` `@param ctx context.Context - for authentication, logging, cancellation, deadlines, tracing, etc. Passed from http.Request or context.Background().`
- `sources/hindsight/hindsight-clients/go/api_knowledge_base.go:316` `@param bankId`
- `sources/hindsight/hindsight-clients/go/api_knowledge_base.go:317` `@param nodeId`
- `sources/hindsight/hindsight-clients/go/api_knowledge_base.go:318` `@return ApiDeleteKnowledgeNodeRequest`
- `sources/hindsight/hindsight-clients/go/api_knowledge_base.go:319` `*/`
- `sources/hindsight/hindsight-clients/go/api_knowledge_base.go:320` `func (a *KnowledgeBaseAPIService) DeleteKnowledgeNode(ctx context.Context, bankId string, nodeId string) ApiDeleteKnowledgeNodeRequest {`
- `sources/hindsight/hindsight-clients/go/api_knowledge_base.go:321` `return ApiDeleteKnowledgeNodeRequest{`
- `sources/hindsight/hindsight-clients/go/api_knowledge_base.go:322` `ApiService: a,`
- `sources/hindsight/hindsight-clients/go/api_knowledge_base.go:323` `ctx: ctx,`
- `sources/hindsight/hindsight-clients/go/api_knowledge_base.go:324` `bankId: bankId,`
- `sources/hindsight/hindsight-clients/go/api_knowledge_base.go:325` `nodeId: nodeId,`
- `sources/hindsight/hindsight-clients/go/api_knowledge_base.go:330` `//  @return interface{}`
- `sources/hindsight/hindsight-clients/go/api_knowledge_base.go:331` `func (a *KnowledgeBaseAPIService) DeleteKnowledgeNodeExecute(r ApiDeleteKnowledgeNodeRequest) (interface{}, *http.Response, error) {`
- `sources/hindsight/hindsight-clients/go/api_knowledge_base.go:332` `var (`
- `sources/hindsight/hindsight-clients/go/api_knowledge_base.go:333` `localVarHTTPMethod   = http.MethodDelete`
- `sources/hindsight/hindsight-clients/go/api_knowledge_base.go:334` `localVarPostBody     interface{}`
- `sources/hindsight/hindsight-clients/go/api_knowledge_base.go:335` `formFiles            []formFile`
- `sources/hindsight/hindsight-clients/go/api_knowledge_base.go:336` `localVarReturnValue  interface{}`
- `sources/hindsight/hindsight-clients/go/api_knowledge_base.go:337` `)`
- `sources/hindsight/hindsight-clients/go/api_knowledge_base.go:338` ``
- `sources/hindsight/hindsight-clients/go/api_knowledge_base.go:339` `localBasePath, err := a.client.cfg.ServerURLWithContext(r.ctx, "KnowledgeBaseAPIService.DeleteKnowledgeNode")`
- `sources/hindsight/hindsight-clients/go/api_knowledge_base.go:340` `if err != nil {`
- `sources/hindsight/hindsight-clients/go/api_knowledge_base.go:341` `return localVarReturnValue, nil, &GenericOpenAPIError{error: err.Error()}`
- `sources/hindsight/hindsight-clients/go/api_knowledge_base.go:342` `}`
- `sources/hindsight/hindsight-clients/go/api_knowledge_base.go:343` ``

**operation and route call sites**, body `sources/hindsight/hindsight-clients/go/api_llm_traces.go:136`; body evidence.

- `sources/hindsight/hindsight-clients/go/api_llm_traces.go:135` ``
- `sources/hindsight/hindsight-clients/go/api_llm_traces.go:136` `List traced LLM requests for a bank, ordered by most recent first. Requires LLM request tracing to be enabled (HINDSIGHT_API_LLM_TRACE_ENABLED). Requests older than HINDSIGHT_API_LLM_TRACE_RETENTION_DAYS are deleted; 'retention_days' in the response says how many days are kept (null = forever).`
- `sources/hindsight/hindsight-clients/go/api_llm_traces.go:137` ``
- `sources/hindsight/hindsight-clients/go/api_llm_traces.go:138` `@param ctx context.Context - for authentication, logging, cancellation, deadlines, tracing, etc. Passed from http.Request or context.Background().`
- `sources/hindsight/hindsight-clients/go/api_llm_traces.go:139` `@param bankId`
- `sources/hindsight/hindsight-clients/go/api_llm_traces.go:140` `@return ApiListLlmRequestsRequest`

**operation and route call sites**, body `sources/hindsight/hindsight-clients/go/api_memory.go:47`; body evidence.

- `sources/hindsight/hindsight-clients/go/api_memory.go:46` ``
- `sources/hindsight/hindsight-clients/go/api_memory.go:47` `func (r ApiClearBankMemoriesRequest) Execute() (*DeleteResponse, *http.Response, error) {`
- `sources/hindsight/hindsight-clients/go/api_memory.go:48` `return r.ApiService.ClearBankMemoriesExecute(r)`
- `sources/hindsight/hindsight-clients/go/api_memory.go:49` `}`
- `sources/hindsight/hindsight-clients/go/api_memory.go:50` ``
- `sources/hindsight/hindsight-clients/go/api_memory.go:51` `/*`
- `sources/hindsight/hindsight-clients/go/api_memory.go:53` ``
- `sources/hindsight/hindsight-clients/go/api_memory.go:54` `Delete memory units for a memory bank. Optionally filter by type (world, experience, observation) to delete only specific types. This is a destructive operation that cannot be undone. The bank profile (disposition and background) will be preserved.`
- `sources/hindsight/hindsight-clients/go/api_memory.go:55` ``
- `sources/hindsight/hindsight-clients/go/api_memory.go:56` `@param ctx context.Context - for authentication, logging, cancellation, deadlines, tracing, etc. Passed from http.Request or context.Background().`
- `sources/hindsight/hindsight-clients/go/api_memory.go:57` `@param bankId`
- `sources/hindsight/hindsight-clients/go/api_memory.go:58` `@return ApiClearBankMemoriesRequest`
- `sources/hindsight/hindsight-clients/go/api_memory.go:69` `//  @return DeleteResponse`
- `sources/hindsight/hindsight-clients/go/api_memory.go:70` `func (a *MemoryAPIService) ClearBankMemoriesExecute(r ApiClearBankMemoriesRequest) (*DeleteResponse, *http.Response, error) {`
- `sources/hindsight/hindsight-clients/go/api_memory.go:71` `var (`
- `sources/hindsight/hindsight-clients/go/api_memory.go:72` `localVarHTTPMethod   = http.MethodDelete`
- `sources/hindsight/hindsight-clients/go/api_memory.go:73` `localVarPostBody     interface{}`
- `sources/hindsight/hindsight-clients/go/api_memory.go:74` `formFiles            []formFile`
- `sources/hindsight/hindsight-clients/go/api_memory.go:75` `localVarReturnValue  *DeleteResponse`
- `sources/hindsight/hindsight-clients/go/api_memory.go:76` `)`
- `sources/hindsight/hindsight-clients/go/api_memory.go:77` ``
- `sources/hindsight/hindsight-clients/go/api_memory.go:78` `localBasePath, err := a.client.cfg.ServerURLWithContext(r.ctx, "MemoryAPIService.ClearBankMemories")`
- `sources/hindsight/hindsight-clients/go/api_memory.go:79` `if err != nil {`
- `sources/hindsight/hindsight-clients/go/api_memory.go:179` ``
- `sources/hindsight/hindsight-clients/go/api_memory.go:180` `Delete all observations derived from a specific memory and reset it for re-consolidation. The memory itself is not deleted. A consolidation job is triggered automatically so the memory will produce fresh observations on the next consolidation run.`
- `sources/hindsight/hindsight-clients/go/api_memory.go:181` ``
- `sources/hindsight/hindsight-clients/go/api_memory.go:182` `@param ctx context.Context - for authentication, logging, cancellation, deadlines, tracing, etc. Passed from http.Request or context.Background().`
- `sources/hindsight/hindsight-clients/go/api_memory.go:183` `@param bankId`
- `sources/hindsight/hindsight-clients/go/api_memory.go:184` `@param memoryId`
- `sources/hindsight/hindsight-clients/go/api_memory.go:199` `var (`
- `sources/hindsight/hindsight-clients/go/api_memory.go:200` `localVarHTTPMethod   = http.MethodDelete`
- `sources/hindsight/hindsight-clients/go/api_memory.go:201` `localVarPostBody     interface{}`
- `sources/hindsight/hindsight-clients/go/api_memory.go:202` `formFiles            []formFile`
- `sources/hindsight/hindsight-clients/go/api_memory.go:203` `localVarReturnValue  *ClearMemoryObservationsResponse`
- `sources/hindsight/hindsight-clients/go/api_memory.go:204` `)`
- `sources/hindsight/hindsight-clients/go/api_memory.go:2172` ``
- `sources/hindsight/hindsight-clients/go/api_memory.go:2173` `Edit a memory's text and/or change its curation state (invalidate / revert). Invalidated memories are excluded from recall, consolidation, and graph maintenance but kept for audit (reversible). Only world/experience facts can be curated; observations are derived.`
- `sources/hindsight/hindsight-clients/go/api_memory.go:2174` ``
- `sources/hindsight/hindsight-clients/go/api_memory.go:2175` `@param ctx context.Context - for authentication, logging, cancellation, deadlines, tracing, etc. Passed from http.Request or context.Background().`
- `sources/hindsight/hindsight-clients/go/api_memory.go:2176` `@param bankId`
- `sources/hindsight/hindsight-clients/go/api_memory.go:2177` `@param memoryId`

**operation and route call sites**, body `sources/hindsight/hindsight-clients/go/api_mental_models.go:286`; body evidence.

- `sources/hindsight/hindsight-clients/go/api_mental_models.go:285` ``
- `sources/hindsight/hindsight-clients/go/api_mental_models.go:286` `type ApiDeleteMentalModelRequest struct {`
- `sources/hindsight/hindsight-clients/go/api_mental_models.go:287` `ctx context.Context`
- `sources/hindsight/hindsight-clients/go/api_mental_models.go:288` `ApiService *MentalModelsAPIService`
- `sources/hindsight/hindsight-clients/go/api_mental_models.go:289` `bankId string`
- `sources/hindsight/hindsight-clients/go/api_mental_models.go:290` `mentalModelId string`
- `sources/hindsight/hindsight-clients/go/api_mental_models.go:293` ``
- `sources/hindsight/hindsight-clients/go/api_mental_models.go:294` `func (r ApiDeleteMentalModelRequest) Authorization(authorization string) ApiDeleteMentalModelRequest {`
- `sources/hindsight/hindsight-clients/go/api_mental_models.go:295` `r.authorization = &authorization`
- `sources/hindsight/hindsight-clients/go/api_mental_models.go:296` `return r`
- `sources/hindsight/hindsight-clients/go/api_mental_models.go:297` `}`
- `sources/hindsight/hindsight-clients/go/api_mental_models.go:298` ``
- `sources/hindsight/hindsight-clients/go/api_mental_models.go:299` `func (r ApiDeleteMentalModelRequest) Execute() (interface{}, *http.Response, error) {`
- `sources/hindsight/hindsight-clients/go/api_mental_models.go:300` `return r.ApiService.DeleteMentalModelExecute(r)`
- `sources/hindsight/hindsight-clients/go/api_mental_models.go:301` `}`
- `sources/hindsight/hindsight-clients/go/api_mental_models.go:302` ``
- `sources/hindsight/hindsight-clients/go/api_mental_models.go:303` `/*`
- `sources/hindsight/hindsight-clients/go/api_mental_models.go:304` `DeleteMentalModel Delete mental model`
- `sources/hindsight/hindsight-clients/go/api_mental_models.go:305` ``
- `sources/hindsight/hindsight-clients/go/api_mental_models.go:306` `Delete a mental model.`
- `sources/hindsight/hindsight-clients/go/api_mental_models.go:307` ``
- `sources/hindsight/hindsight-clients/go/api_mental_models.go:308` `@param ctx context.Context - for authentication, logging, cancellation, deadlines, tracing, etc. Passed from http.Request or context.Background().`
- `sources/hindsight/hindsight-clients/go/api_mental_models.go:309` `@param bankId`
- `sources/hindsight/hindsight-clients/go/api_mental_models.go:310` `@param mentalModelId`
- `sources/hindsight/hindsight-clients/go/api_mental_models.go:311` `@return ApiDeleteMentalModelRequest`
- `sources/hindsight/hindsight-clients/go/api_mental_models.go:312` `*/`
- `sources/hindsight/hindsight-clients/go/api_mental_models.go:313` `func (a *MentalModelsAPIService) DeleteMentalModel(ctx context.Context, bankId string, mentalModelId string) ApiDeleteMentalModelRequest {`
- `sources/hindsight/hindsight-clients/go/api_mental_models.go:314` `return ApiDeleteMentalModelRequest{`
- `sources/hindsight/hindsight-clients/go/api_mental_models.go:315` `ApiService: a,`
- `sources/hindsight/hindsight-clients/go/api_mental_models.go:316` `ctx: ctx,`
- `sources/hindsight/hindsight-clients/go/api_mental_models.go:317` `bankId: bankId,`
- `sources/hindsight/hindsight-clients/go/api_mental_models.go:318` `mentalModelId: mentalModelId,`
- `sources/hindsight/hindsight-clients/go/api_mental_models.go:323` `//  @return interface{}`
- `sources/hindsight/hindsight-clients/go/api_mental_models.go:324` `func (a *MentalModelsAPIService) DeleteMentalModelExecute(r ApiDeleteMentalModelRequest) (interface{}, *http.Response, error) {`
- `sources/hindsight/hindsight-clients/go/api_mental_models.go:325` `var (`
- `sources/hindsight/hindsight-clients/go/api_mental_models.go:326` `localVarHTTPMethod   = http.MethodDelete`
- `sources/hindsight/hindsight-clients/go/api_mental_models.go:327` `localVarPostBody     interface{}`
- `sources/hindsight/hindsight-clients/go/api_mental_models.go:328` `formFiles            []formFile`
- `sources/hindsight/hindsight-clients/go/api_mental_models.go:329` `localVarReturnValue  interface{}`
- `sources/hindsight/hindsight-clients/go/api_mental_models.go:330` `)`
- `sources/hindsight/hindsight-clients/go/api_mental_models.go:331` ``
- `sources/hindsight/hindsight-clients/go/api_mental_models.go:332` `localBasePath, err := a.client.cfg.ServerURLWithContext(r.ctx, "MentalModelsAPIService.DeleteMentalModel")`
- `sources/hindsight/hindsight-clients/go/api_mental_models.go:333` `if err != nil {`
- `sources/hindsight/hindsight-clients/go/api_mental_models.go:334` `return localVarReturnValue, nil, &GenericOpenAPIError{error: err.Error()}`
- `sources/hindsight/hindsight-clients/go/api_mental_models.go:335` `}`
- `sources/hindsight/hindsight-clients/go/api_mental_models.go:336` ``

**operation and route call sites**, body `sources/hindsight/hindsight-clients/go/api_operations.go:66`; body evidence.

- `sources/hindsight/hindsight-clients/go/api_operations.go:65` `var (`
- `sources/hindsight/hindsight-clients/go/api_operations.go:66` `localVarHTTPMethod   = http.MethodDelete`
- `sources/hindsight/hindsight-clients/go/api_operations.go:67` `localVarPostBody     interface{}`
- `sources/hindsight/hindsight-clients/go/api_operations.go:68` `formFiles            []formFile`
- `sources/hindsight/hindsight-clients/go/api_operations.go:69` `localVarReturnValue  *CancelOperationResponse`
- `sources/hindsight/hindsight-clients/go/api_operations.go:70` `)`
- `sources/hindsight/hindsight-clients/go/api_operations.go:151` ``
- `sources/hindsight/hindsight-clients/go/api_operations.go:152` `type ApiDeleteOperationRequest struct {`
- `sources/hindsight/hindsight-clients/go/api_operations.go:153` `ctx context.Context`
- `sources/hindsight/hindsight-clients/go/api_operations.go:154` `ApiService *OperationsAPIService`
- `sources/hindsight/hindsight-clients/go/api_operations.go:155` `bankId string`
- `sources/hindsight/hindsight-clients/go/api_operations.go:156` `operationId string`
- `sources/hindsight/hindsight-clients/go/api_operations.go:159` ``
- `sources/hindsight/hindsight-clients/go/api_operations.go:160` `func (r ApiDeleteOperationRequest) Authorization(authorization string) ApiDeleteOperationRequest {`
- `sources/hindsight/hindsight-clients/go/api_operations.go:161` `r.authorization = &authorization`
- `sources/hindsight/hindsight-clients/go/api_operations.go:162` `return r`
- `sources/hindsight/hindsight-clients/go/api_operations.go:163` `}`
- `sources/hindsight/hindsight-clients/go/api_operations.go:164` ``
- `sources/hindsight/hindsight-clients/go/api_operations.go:165` `func (r ApiDeleteOperationRequest) Execute() (*DeleteOperationResponse, *http.Response, error) {`
- `sources/hindsight/hindsight-clients/go/api_operations.go:166` `return r.ApiService.DeleteOperationExecute(r)`
- `sources/hindsight/hindsight-clients/go/api_operations.go:167` `}`
- `sources/hindsight/hindsight-clients/go/api_operations.go:168` ``
- `sources/hindsight/hindsight-clients/go/api_operations.go:169` `/*`
- `sources/hindsight/hindsight-clients/go/api_operations.go:170` `DeleteOperation Delete a terminal async operation`
- `sources/hindsight/hindsight-clients/go/api_operations.go:171` ``
- `sources/hindsight/hindsight-clients/go/api_operations.go:172` `Permanently remove a failed, cancelled, or completed async operation record`
- `sources/hindsight/hindsight-clients/go/api_operations.go:173` ``
- `sources/hindsight/hindsight-clients/go/api_operations.go:174` `@param ctx context.Context - for authentication, logging, cancellation, deadlines, tracing, etc. Passed from http.Request or context.Background().`
- `sources/hindsight/hindsight-clients/go/api_operations.go:176` `@param operationId`
- `sources/hindsight/hindsight-clients/go/api_operations.go:177` `@return ApiDeleteOperationRequest`
- `sources/hindsight/hindsight-clients/go/api_operations.go:178` `*/`
- `sources/hindsight/hindsight-clients/go/api_operations.go:179` `func (a *OperationsAPIService) DeleteOperation(ctx context.Context, bankId string, operationId string) ApiDeleteOperationRequest {`
- `sources/hindsight/hindsight-clients/go/api_operations.go:180` `return ApiDeleteOperationRequest{`
- `sources/hindsight/hindsight-clients/go/api_operations.go:181` `ApiService: a,`
- `sources/hindsight/hindsight-clients/go/api_operations.go:182` `ctx: ctx,`
- `sources/hindsight/hindsight-clients/go/api_operations.go:183` `bankId: bankId,`
- `sources/hindsight/hindsight-clients/go/api_operations.go:184` `operationId: operationId,`
- `sources/hindsight/hindsight-clients/go/api_operations.go:189` `//  @return DeleteOperationResponse`
- `sources/hindsight/hindsight-clients/go/api_operations.go:190` `func (a *OperationsAPIService) DeleteOperationExecute(r ApiDeleteOperationRequest) (*DeleteOperationResponse, *http.Response, error) {`
- `sources/hindsight/hindsight-clients/go/api_operations.go:191` `var (`
- `sources/hindsight/hindsight-clients/go/api_operations.go:192` `localVarHTTPMethod   = http.MethodDelete`
- `sources/hindsight/hindsight-clients/go/api_operations.go:193` `localVarPostBody     interface{}`
- `sources/hindsight/hindsight-clients/go/api_operations.go:194` `formFiles            []formFile`
- `sources/hindsight/hindsight-clients/go/api_operations.go:195` `localVarReturnValue  *DeleteOperationResponse`
- `sources/hindsight/hindsight-clients/go/api_operations.go:196` `)`
- `sources/hindsight/hindsight-clients/go/api_operations.go:197` ``
- `sources/hindsight/hindsight-clients/go/api_operations.go:198` `localBasePath, err := a.client.cfg.ServerURLWithContext(r.ctx, "OperationsAPIService.DeleteOperation")`
- `sources/hindsight/hindsight-clients/go/api_operations.go:199` `if err != nil {`
- `sources/hindsight/hindsight-clients/go/api_operations.go:200` `return localVarReturnValue, nil, &GenericOpenAPIError{error: err.Error()}`
- `sources/hindsight/hindsight-clients/go/api_operations.go:201` `}`
- `sources/hindsight/hindsight-clients/go/api_operations.go:202` ``
- `sources/hindsight/hindsight-clients/go/api_operations.go:203` `localVarPath := localBasePath + "/v1/default/banks/{bank_id}/operations/{operation_id}/delete"`
- `sources/hindsight/hindsight-clients/go/api_operations.go:204` `localVarPath = strings.Replace(localVarPath, "{"+"bank_id"+"}", url.PathEscape(parameterValueToString(r.bankId, "bankId")), -1)`
- `sources/hindsight/hindsight-clients/go/api_operations.go:205` `localVarPath = strings.Replace(localVarPath, "{"+"operation_id"+"}", url.PathEscape(parameterValueToString(r.operationId, "operationId")), -1)`
- `sources/hindsight/hindsight-clients/go/api_operations.go:206` ``
- `sources/hindsight/hindsight-clients/go/api_operations.go:207` `localVarHeaderParams := make(map[string]string)`

**operation and route call sites**, body `sources/hindsight/hindsight-clients/go/api_webhooks.go:159`; body evidence.

- `sources/hindsight/hindsight-clients/go/api_webhooks.go:158` ``
- `sources/hindsight/hindsight-clients/go/api_webhooks.go:159` `type ApiDeleteWebhookRequest struct {`
- `sources/hindsight/hindsight-clients/go/api_webhooks.go:160` `ctx context.Context`
- `sources/hindsight/hindsight-clients/go/api_webhooks.go:161` `ApiService *WebhooksAPIService`
- `sources/hindsight/hindsight-clients/go/api_webhooks.go:162` `bankId string`
- `sources/hindsight/hindsight-clients/go/api_webhooks.go:163` `webhookId string`
- `sources/hindsight/hindsight-clients/go/api_webhooks.go:166` ``
- `sources/hindsight/hindsight-clients/go/api_webhooks.go:167` `func (r ApiDeleteWebhookRequest) Authorization(authorization string) ApiDeleteWebhookRequest {`
- `sources/hindsight/hindsight-clients/go/api_webhooks.go:168` `r.authorization = &authorization`
- `sources/hindsight/hindsight-clients/go/api_webhooks.go:169` `return r`
- `sources/hindsight/hindsight-clients/go/api_webhooks.go:170` `}`
- `sources/hindsight/hindsight-clients/go/api_webhooks.go:171` ``
- `sources/hindsight/hindsight-clients/go/api_webhooks.go:172` `func (r ApiDeleteWebhookRequest) Execute() (*DeleteResponse, *http.Response, error) {`
- `sources/hindsight/hindsight-clients/go/api_webhooks.go:173` `return r.ApiService.DeleteWebhookExecute(r)`
- `sources/hindsight/hindsight-clients/go/api_webhooks.go:174` `}`
- `sources/hindsight/hindsight-clients/go/api_webhooks.go:175` ``
- `sources/hindsight/hindsight-clients/go/api_webhooks.go:176` `/*`
- `sources/hindsight/hindsight-clients/go/api_webhooks.go:177` `DeleteWebhook Delete webhook`
- `sources/hindsight/hindsight-clients/go/api_webhooks.go:178` ``
- `sources/hindsight/hindsight-clients/go/api_webhooks.go:179` `Remove a registered webhook.`
- `sources/hindsight/hindsight-clients/go/api_webhooks.go:180` ``
- `sources/hindsight/hindsight-clients/go/api_webhooks.go:181` `@param ctx context.Context - for authentication, logging, cancellation, deadlines, tracing, etc. Passed from http.Request or context.Background().`
- `sources/hindsight/hindsight-clients/go/api_webhooks.go:183` `@param webhookId`
- `sources/hindsight/hindsight-clients/go/api_webhooks.go:184` `@return ApiDeleteWebhookRequest`
- `sources/hindsight/hindsight-clients/go/api_webhooks.go:185` `*/`
- `sources/hindsight/hindsight-clients/go/api_webhooks.go:186` `func (a *WebhooksAPIService) DeleteWebhook(ctx context.Context, bankId string, webhookId string) ApiDeleteWebhookRequest {`
- `sources/hindsight/hindsight-clients/go/api_webhooks.go:187` `return ApiDeleteWebhookRequest{`
- `sources/hindsight/hindsight-clients/go/api_webhooks.go:188` `ApiService: a,`
- `sources/hindsight/hindsight-clients/go/api_webhooks.go:189` `ctx: ctx,`
- `sources/hindsight/hindsight-clients/go/api_webhooks.go:190` `bankId: bankId,`
- `sources/hindsight/hindsight-clients/go/api_webhooks.go:191` `webhookId: webhookId,`
- `sources/hindsight/hindsight-clients/go/api_webhooks.go:196` `//  @return DeleteResponse`
- `sources/hindsight/hindsight-clients/go/api_webhooks.go:197` `func (a *WebhooksAPIService) DeleteWebhookExecute(r ApiDeleteWebhookRequest) (*DeleteResponse, *http.Response, error) {`
- `sources/hindsight/hindsight-clients/go/api_webhooks.go:198` `var (`
- `sources/hindsight/hindsight-clients/go/api_webhooks.go:199` `localVarHTTPMethod   = http.MethodDelete`
- `sources/hindsight/hindsight-clients/go/api_webhooks.go:200` `localVarPostBody     interface{}`
- `sources/hindsight/hindsight-clients/go/api_webhooks.go:201` `formFiles            []formFile`
- `sources/hindsight/hindsight-clients/go/api_webhooks.go:202` `localVarReturnValue  *DeleteResponse`
- `sources/hindsight/hindsight-clients/go/api_webhooks.go:203` `)`
- `sources/hindsight/hindsight-clients/go/api_webhooks.go:204` ``
- `sources/hindsight/hindsight-clients/go/api_webhooks.go:205` `localBasePath, err := a.client.cfg.ServerURLWithContext(r.ctx, "WebhooksAPIService.DeleteWebhook")`
- `sources/hindsight/hindsight-clients/go/api_webhooks.go:206` `if err != nil {`
- `sources/hindsight/hindsight-clients/go/api_webhooks.go:207` `return localVarReturnValue, nil, &GenericOpenAPIError{error: err.Error()}`
- `sources/hindsight/hindsight-clients/go/api_webhooks.go:208` `}`
- `sources/hindsight/hindsight-clients/go/api_webhooks.go:209` ``

**operation and route call sites**, body `sources/hindsight/hindsight-clients/go/model_clear_memory_observations_response.go:24`; body evidence.

- `sources/hindsight/hindsight-clients/go/model_clear_memory_observations_response.go:23` `type ClearMemoryObservationsResponse struct {`
- `sources/hindsight/hindsight-clients/go/model_clear_memory_observations_response.go:24` `DeletedCount int32 'json:"deleted_count"'`
- `sources/hindsight/hindsight-clients/go/model_clear_memory_observations_response.go:25` `}`
- `sources/hindsight/hindsight-clients/go/model_clear_memory_observations_response.go:26` ``
- `sources/hindsight/hindsight-clients/go/model_clear_memory_observations_response.go:27` `type _ClearMemoryObservationsResponse ClearMemoryObservationsResponse`
- `sources/hindsight/hindsight-clients/go/model_clear_memory_observations_response.go:28` ``
- `sources/hindsight/hindsight-clients/go/model_clear_memory_observations_response.go:32` `// will change when the set of required properties is changed`
- `sources/hindsight/hindsight-clients/go/model_clear_memory_observations_response.go:33` `func NewClearMemoryObservationsResponse(deletedCount int32) *ClearMemoryObservationsResponse {`
- `sources/hindsight/hindsight-clients/go/model_clear_memory_observations_response.go:34` `this := ClearMemoryObservationsResponse{}`
- `sources/hindsight/hindsight-clients/go/model_clear_memory_observations_response.go:35` `this.DeletedCount = deletedCount`
- `sources/hindsight/hindsight-clients/go/model_clear_memory_observations_response.go:36` `return &this`
- `sources/hindsight/hindsight-clients/go/model_clear_memory_observations_response.go:37` `}`
- `sources/hindsight/hindsight-clients/go/model_clear_memory_observations_response.go:38` ``
- `sources/hindsight/hindsight-clients/go/model_clear_memory_observations_response.go:39` `// NewClearMemoryObservationsResponseWithDefaults instantiates a new ClearMemoryObservationsResponse object`
- `sources/hindsight/hindsight-clients/go/model_clear_memory_observations_response.go:47` `// GetDeletedCount returns the DeletedCount field value`
- `sources/hindsight/hindsight-clients/go/model_clear_memory_observations_response.go:48` `func (o *ClearMemoryObservationsResponse) GetDeletedCount() int32 {`
- `sources/hindsight/hindsight-clients/go/model_clear_memory_observations_response.go:49` `if o == nil {`
- `sources/hindsight/hindsight-clients/go/model_clear_memory_observations_response.go:50` `var ret int32`
- `sources/hindsight/hindsight-clients/go/model_clear_memory_observations_response.go:51` `return ret`
- `sources/hindsight/hindsight-clients/go/model_clear_memory_observations_response.go:52` `}`
- `sources/hindsight/hindsight-clients/go/model_clear_memory_observations_response.go:53` ``
- `sources/hindsight/hindsight-clients/go/model_clear_memory_observations_response.go:54` `return o.DeletedCount`
- `sources/hindsight/hindsight-clients/go/model_clear_memory_observations_response.go:55` `}`
- `sources/hindsight/hindsight-clients/go/model_clear_memory_observations_response.go:56` ``
- `sources/hindsight/hindsight-clients/go/model_clear_memory_observations_response.go:57` `// GetDeletedCountOk returns a tuple with the DeletedCount field value`
- `sources/hindsight/hindsight-clients/go/model_clear_memory_observations_response.go:58` `// and a boolean to check if the value has been set.`
- `sources/hindsight/hindsight-clients/go/model_clear_memory_observations_response.go:59` `func (o *ClearMemoryObservationsResponse) GetDeletedCountOk() (*int32, bool) {`
- `sources/hindsight/hindsight-clients/go/model_clear_memory_observations_response.go:60` `if o == nil {`
- `sources/hindsight/hindsight-clients/go/model_clear_memory_observations_response.go:61` `return nil, false`
- `sources/hindsight/hindsight-clients/go/model_clear_memory_observations_response.go:62` `}`
- `sources/hindsight/hindsight-clients/go/model_clear_memory_observations_response.go:63` `return &o.DeletedCount, true`
- `sources/hindsight/hindsight-clients/go/model_clear_memory_observations_response.go:64` `}`
- `sources/hindsight/hindsight-clients/go/model_clear_memory_observations_response.go:65` ``
- `sources/hindsight/hindsight-clients/go/model_clear_memory_observations_response.go:66` `// SetDeletedCount sets field value`
- `sources/hindsight/hindsight-clients/go/model_clear_memory_observations_response.go:67` `func (o *ClearMemoryObservationsResponse) SetDeletedCount(v int32) {`
- `sources/hindsight/hindsight-clients/go/model_clear_memory_observations_response.go:68` `o.DeletedCount = v`
- `sources/hindsight/hindsight-clients/go/model_clear_memory_observations_response.go:69` `}`
- `sources/hindsight/hindsight-clients/go/model_clear_memory_observations_response.go:70` ``
- `sources/hindsight/hindsight-clients/go/model_clear_memory_observations_response.go:71` `func (o ClearMemoryObservationsResponse) MarshalJSON() ([]byte, error) {`
- `sources/hindsight/hindsight-clients/go/model_clear_memory_observations_response.go:72` `toSerialize,err := o.ToMap()`
- `sources/hindsight/hindsight-clients/go/model_clear_memory_observations_response.go:80` `toSerialize := map[string]interface{}{}`
- `sources/hindsight/hindsight-clients/go/model_clear_memory_observations_response.go:81` `toSerialize["deleted_count"] = o.DeletedCount`
- `sources/hindsight/hindsight-clients/go/model_clear_memory_observations_response.go:82` `return toSerialize, nil`
- `sources/hindsight/hindsight-clients/go/model_clear_memory_observations_response.go:83` `}`
- `sources/hindsight/hindsight-clients/go/model_clear_memory_observations_response.go:84` ``
- `sources/hindsight/hindsight-clients/go/model_clear_memory_observations_response.go:85` `func (o *ClearMemoryObservationsResponse) UnmarshalJSON(data []byte) (err error) {`
- `sources/hindsight/hindsight-clients/go/model_clear_memory_observations_response.go:89` `requiredProperties := []string{`
- `sources/hindsight/hindsight-clients/go/model_clear_memory_observations_response.go:90` `"deleted_count",`
- `sources/hindsight/hindsight-clients/go/model_clear_memory_observations_response.go:91` `}`
- `sources/hindsight/hindsight-clients/go/model_clear_memory_observations_response.go:92` ``
- `sources/hindsight/hindsight-clients/go/model_clear_memory_observations_response.go:93` `allProperties := make(map[string]interface{})`
- `sources/hindsight/hindsight-clients/go/model_clear_memory_observations_response.go:94` ``

**operation and route call sites**, body `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:20`; body evidence.

- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:19` `// checks if the DeleteDocumentResponse type satisfies the MappedNullable interface at compile time`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:20` `var _ MappedNullable = &DeleteDocumentResponse{}`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:21` ``
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:22` `// DeleteDocumentResponse Response model for delete document endpoint.`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:23` `type DeleteDocumentResponse struct {`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:24` `Success bool 'json:"success"'`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:25` `Message string 'json:"message"'`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:26` `DocumentId string 'json:"document_id"'`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:27` `MemoryUnitsDeleted int32 'json:"memory_units_deleted"'`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:28` `}`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:29` ``
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:30` `type _DeleteDocumentResponse DeleteDocumentResponse`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:31` ``
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:32` `// NewDeleteDocumentResponse instantiates a new DeleteDocumentResponse object`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:33` `// This constructor will assign default values to properties that have it defined,`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:34` `// and makes sure properties required by API are set, but the set of arguments`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:35` `// will change when the set of required properties is changed`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:36` `func NewDeleteDocumentResponse(success bool, message string, documentId string, memoryUnitsDeleted int32) *DeleteDocumentResponse {`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:37` `this := DeleteDocumentResponse{}`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:38` `this.Success = success`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:39` `this.Message = message`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:40` `this.DocumentId = documentId`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:41` `this.MemoryUnitsDeleted = memoryUnitsDeleted`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:42` `return &this`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:43` `}`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:44` ``
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:45` `// NewDeleteDocumentResponseWithDefaults instantiates a new DeleteDocumentResponse object`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:47` `// but it doesn't guarantee that properties required by API are set`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:48` `func NewDeleteDocumentResponseWithDefaults() *DeleteDocumentResponse {`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:49` `this := DeleteDocumentResponse{}`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:50` `return &this`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:51` `}`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:52` ``
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:53` `// GetSuccess returns the Success field value`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:54` `func (o *DeleteDocumentResponse) GetSuccess() bool {`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:55` `if o == nil {`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:56` `var ret bool`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:57` `return ret`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:58` `}`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:64` `// and a boolean to check if the value has been set.`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:65` `func (o *DeleteDocumentResponse) GetSuccessOk() (*bool, bool) {`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:66` `if o == nil {`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:67` `return nil, false`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:68` `}`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:69` `return &o.Success, true`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:72` `// SetSuccess sets field value`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:73` `func (o *DeleteDocumentResponse) SetSuccess(v bool) {`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:74` `o.Success = v`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:75` `}`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:76` ``
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:77` `// GetMessage returns the Message field value`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:78` `func (o *DeleteDocumentResponse) GetMessage() string {`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:79` `if o == nil {`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:80` `var ret string`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:81` `return ret`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:82` `}`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:88` `// and a boolean to check if the value has been set.`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:89` `func (o *DeleteDocumentResponse) GetMessageOk() (*string, bool) {`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:90` `if o == nil {`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:91` `return nil, false`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:92` `}`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:93` `return &o.Message, true`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:96` `// SetMessage sets field value`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:97` `func (o *DeleteDocumentResponse) SetMessage(v string) {`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:98` `o.Message = v`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:99` `}`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:100` ``
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:101` `// GetDocumentId returns the DocumentId field value`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:102` `func (o *DeleteDocumentResponse) GetDocumentId() string {`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:103` `if o == nil {`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:104` `var ret string`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:105` `return ret`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:106` `}`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:112` `// and a boolean to check if the value has been set.`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:113` `func (o *DeleteDocumentResponse) GetDocumentIdOk() (*string, bool) {`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:114` `if o == nil {`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:115` `return nil, false`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:116` `}`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:117` `return &o.DocumentId, true`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:120` `// SetDocumentId sets field value`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:121` `func (o *DeleteDocumentResponse) SetDocumentId(v string) {`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:122` `o.DocumentId = v`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:123` `}`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:124` ``
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:125` `// GetMemoryUnitsDeleted returns the MemoryUnitsDeleted field value`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:126` `func (o *DeleteDocumentResponse) GetMemoryUnitsDeleted() int32 {`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:127` `if o == nil {`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:128` `var ret int32`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:129` `return ret`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:130` `}`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:131` ``
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:132` `return o.MemoryUnitsDeleted`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:133` `}`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:134` ``
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:135` `// GetMemoryUnitsDeletedOk returns a tuple with the MemoryUnitsDeleted field value`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:136` `// and a boolean to check if the value has been set.`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:137` `func (o *DeleteDocumentResponse) GetMemoryUnitsDeletedOk() (*int32, bool) {`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:138` `if o == nil {`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:139` `return nil, false`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:140` `}`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:141` `return &o.MemoryUnitsDeleted, true`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:142` `}`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:143` ``
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:144` `// SetMemoryUnitsDeleted sets field value`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:145` `func (o *DeleteDocumentResponse) SetMemoryUnitsDeleted(v int32) {`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:146` `o.MemoryUnitsDeleted = v`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:147` `}`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:148` ``
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:149` `func (o DeleteDocumentResponse) MarshalJSON() ([]byte, error) {`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:150` `toSerialize,err := o.ToMap()`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:151` `if err != nil {`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:152` `return []byte{}, err`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:153` `}`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:156` ``
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:157` `func (o DeleteDocumentResponse) ToMap() (map[string]interface{}, error) {`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:158` `toSerialize := map[string]interface{}{}`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:159` `toSerialize["success"] = o.Success`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:160` `toSerialize["message"] = o.Message`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:161` `toSerialize["document_id"] = o.DocumentId`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:162` `toSerialize["memory_units_deleted"] = o.MemoryUnitsDeleted`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:163` `return toSerialize, nil`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:164` `}`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:165` ``
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:166` `func (o *DeleteDocumentResponse) UnmarshalJSON(data []byte) (err error) {`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:167` `// This validates that all required properties are included in the JSON object`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:168` `// by unmarshalling the object into a generic map with string keys and checking`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:169` `// that every required field exists as a key in the generic map.`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:170` `requiredProperties := []string{`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:173` `"document_id",`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:174` `"memory_units_deleted",`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:175` `}`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:176` ``
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:177` `allProperties := make(map[string]interface{})`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:178` ``
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:190` ``
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:191` `varDeleteDocumentResponse := _DeleteDocumentResponse{}`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:192` ``
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:193` `decoder := json.NewDecoder(bytes.NewReader(data))`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:194` `err = decoder.Decode(&varDeleteDocumentResponse)`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:195` ``
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:196` `if err != nil {`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:197` `return err`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:198` `}`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:204` ``
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:205` `type NullableDeleteDocumentResponse struct {`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:206` `value *DeleteDocumentResponse`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:207` `isSet bool`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:208` `}`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:209` ``
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:210` `func (v NullableDeleteDocumentResponse) Get() *DeleteDocumentResponse {`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:211` `return v.value`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:212` `}`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:213` ``
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:214` `func (v *NullableDeleteDocumentResponse) Set(val *DeleteDocumentResponse) {`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:215` `v.value = val`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:216` `v.isSet = true`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:217` `}`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:218` ``
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:219` `func (v NullableDeleteDocumentResponse) IsSet() bool {`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:220` `return v.isSet`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:221` `}`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:222` ``
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:223` `func (v *NullableDeleteDocumentResponse) Unset() {`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:224` `v.value = nil`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:225` `v.isSet = false`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:226` `}`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:227` ``
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:228` `func NewNullableDeleteDocumentResponse(val *DeleteDocumentResponse) *NullableDeleteDocumentResponse {`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:229` `return &NullableDeleteDocumentResponse{value: val, isSet: true}`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:230` `}`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:231` ``
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:232` `func (v NullableDeleteDocumentResponse) MarshalJSON() ([]byte, error) {`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:233` `return json.Marshal(v.value)`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:234` `}`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:235` ``
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:236` `func (v *NullableDeleteDocumentResponse) UnmarshalJSON(src []byte) error {`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:237` `v.isSet = true`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:238` `return json.Unmarshal(src, &v.value)`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:239` `}`
- `sources/hindsight/hindsight-clients/go/model_delete_document_response.go:240` ``

**operation and route call sites**, body `sources/hindsight/hindsight-clients/go/model_delete_operation_response.go:20`; body evidence.

- `sources/hindsight/hindsight-clients/go/model_delete_operation_response.go:19` `// checks if the DeleteOperationResponse type satisfies the MappedNullable interface at compile time`
- `sources/hindsight/hindsight-clients/go/model_delete_operation_response.go:20` `var _ MappedNullable = &DeleteOperationResponse{}`
- `sources/hindsight/hindsight-clients/go/model_delete_operation_response.go:21` ``
- `sources/hindsight/hindsight-clients/go/model_delete_operation_response.go:22` `// DeleteOperationResponse Response model for delete operation endpoint.`
- `sources/hindsight/hindsight-clients/go/model_delete_operation_response.go:23` `type DeleteOperationResponse struct {`
- `sources/hindsight/hindsight-clients/go/model_delete_operation_response.go:24` `Success bool 'json:"success"'`
- `sources/hindsight/hindsight-clients/go/model_delete_operation_response.go:25` `Message string 'json:"message"'`
- `sources/hindsight/hindsight-clients/go/model_delete_operation_response.go:26` `OperationId string 'json:"operation_id"'`
- `sources/hindsight/hindsight-clients/go/model_delete_operation_response.go:27` `}`
- `sources/hindsight/hindsight-clients/go/model_delete_operation_response.go:28` ``
- `sources/hindsight/hindsight-clients/go/model_delete_operation_response.go:29` `type _DeleteOperationResponse DeleteOperationResponse`
- `sources/hindsight/hindsight-clients/go/model_delete_operation_response.go:30` ``
- `sources/hindsight/hindsight-clients/go/model_delete_operation_response.go:31` `// NewDeleteOperationResponse instantiates a new DeleteOperationResponse object`
- `sources/hindsight/hindsight-clients/go/model_delete_operation_response.go:32` `// This constructor will assign default values to properties that have it defined,`
- `sources/hindsight/hindsight-clients/go/model_delete_operation_response.go:33` `// and makes sure properties required by API are set, but the set of arguments`
- `sources/hindsight/hindsight-clients/go/model_delete_operation_response.go:34` `// will change when the set of required properties is changed`
- `sources/hindsight/hindsight-clients/go/model_delete_operation_response.go:35` `func NewDeleteOperationResponse(success bool, message string, operationId string) *DeleteOperationResponse {`
- `sources/hindsight/hindsight-clients/go/model_delete_operation_response.go:36` `this := DeleteOperationResponse{}`
- `sources/hindsight/hindsight-clients/go/model_delete_operation_response.go:37` `this.Success = success`
- `sources/hindsight/hindsight-clients/go/model_delete_operation_response.go:38` `this.Message = message`
- `sources/hindsight/hindsight-clients/go/model_delete_operation_response.go:39` `this.OperationId = operationId`
- `sources/hindsight/hindsight-clients/go/model_delete_operation_response.go:40` `return &this`
- `sources/hindsight/hindsight-clients/go/model_delete_operation_response.go:45` `// but it doesn't guarantee that properties required by API are set`
- `sources/hindsight/hindsight-clients/go/model_delete_operation_response.go:46` `func NewDeleteOperationResponseWithDefaults() *DeleteOperationResponse {`
- `sources/hindsight/hindsight-clients/go/model_delete_operation_response.go:47` `this := DeleteOperationResponse{}`
- `sources/hindsight/hindsight-clients/go/model_delete_operation_response.go:48` `return &this`
- `sources/hindsight/hindsight-clients/go/model_delete_operation_response.go:49` `}`
- `sources/hindsight/hindsight-clients/go/model_delete_operation_response.go:50` ``
- `sources/hindsight/hindsight-clients/go/model_delete_operation_response.go:51` `// GetSuccess returns the Success field value`
- `sources/hindsight/hindsight-clients/go/model_delete_operation_response.go:52` `func (o *DeleteOperationResponse) GetSuccess() bool {`
- `sources/hindsight/hindsight-clients/go/model_delete_operation_response.go:53` `if o == nil {`
- `sources/hindsight/hindsight-clients/go/model_delete_operation_response.go:54` `var ret bool`
- `sources/hindsight/hindsight-clients/go/model_delete_operation_response.go:55` `return ret`
- `sources/hindsight/hindsight-clients/go/model_delete_operation_response.go:56` `}`
- `sources/hindsight/hindsight-clients/go/model_delete_operation_response.go:62` `// and a boolean to check if the value has been set.`
- `sources/hindsight/hindsight-clients/go/model_delete_operation_response.go:63` `func (o *DeleteOperationResponse) GetSuccessOk() (*bool, bool) {`
- `sources/hindsight/hindsight-clients/go/model_delete_operation_response.go:64` `if o == nil {`
- `sources/hindsight/hindsight-clients/go/model_delete_operation_response.go:65` `return nil, false`
- `sources/hindsight/hindsight-clients/go/model_delete_operation_response.go:66` `}`
- `sources/hindsight/hindsight-clients/go/model_delete_operation_response.go:67` `return &o.Success, true`
- `sources/hindsight/hindsight-clients/go/model_delete_operation_response.go:70` `// SetSuccess sets field value`
- `sources/hindsight/hindsight-clients/go/model_delete_operation_response.go:71` `func (o *DeleteOperationResponse) SetSuccess(v bool) {`
- `sources/hindsight/hindsight-clients/go/model_delete_operation_response.go:72` `o.Success = v`
- `sources/hindsight/hindsight-clients/go/model_delete_operation_response.go:73` `}`
- `sources/hindsight/hindsight-clients/go/model_delete_operation_response.go:74` ``
- `sources/hindsight/hindsight-clients/go/model_delete_operation_response.go:75` `// GetMessage returns the Message field value`
- `sources/hindsight/hindsight-clients/go/model_delete_operation_response.go:76` `func (o *DeleteOperationResponse) GetMessage() string {`
- `sources/hindsight/hindsight-clients/go/model_delete_operation_response.go:77` `if o == nil {`
- `sources/hindsight/hindsight-clients/go/model_delete_operation_response.go:78` `var ret string`
- `sources/hindsight/hindsight-clients/go/model_delete_operation_response.go:79` `return ret`
- `sources/hindsight/hindsight-clients/go/model_delete_operation_response.go:80` `}`
- `sources/hindsight/hindsight-clients/go/model_delete_operation_response.go:86` `// and a boolean to check if the value has been set.`
- `sources/hindsight/hindsight-clients/go/model_delete_operation_response.go:87` `func (o *DeleteOperationResponse) GetMessageOk() (*string, bool) {`
- `sources/hindsight/hindsight-clients/go/model_delete_operation_response.go:88` `if o == nil {`
- `sources/hindsight/hindsight-clients/go/model_delete_operation_response.go:89` `return nil, false`
- `sources/hindsight/hindsight-clients/go/model_delete_operation_response.go:90` `}`
- `sources/hindsight/hindsight-clients/go/model_delete_operation_response.go:91` `return &o.Message, true`
- `sources/hindsight/hindsight-clients/go/model_delete_operation_response.go:94` `// SetMessage sets field value`
- `sources/hindsight/hindsight-clients/go/model_delete_operation_response.go:95` `func (o *DeleteOperationResponse) SetMessage(v string) {`
- `sources/hindsight/hindsight-clients/go/model_delete_operation_response.go:96` `o.Message = v`
- `sources/hindsight/hindsight-clients/go/model_delete_operation_response.go:97` `}`
- `sources/hindsight/hindsight-clients/go/model_delete_operation_response.go:98` ``
- `sources/hindsight/hindsight-clients/go/model_delete_operation_response.go:99` `// GetOperationId returns the OperationId field value`
- `sources/hindsight/hindsight-clients/go/model_delete_operation_response.go:100` `func (o *DeleteOperationResponse) GetOperationId() string {`
- `sources/hindsight/hindsight-clients/go/model_delete_operation_response.go:101` `if o == nil {`
- `sources/hindsight/hindsight-clients/go/model_delete_operation_response.go:102` `var ret string`
- `sources/hindsight/hindsight-clients/go/model_delete_operation_response.go:103` `return ret`
- `sources/hindsight/hindsight-clients/go/model_delete_operation_response.go:104` `}`
- `sources/hindsight/hindsight-clients/go/model_delete_operation_response.go:110` `// and a boolean to check if the value has been set.`
- `sources/hindsight/hindsight-clients/go/model_delete_operation_response.go:111` `func (o *DeleteOperationResponse) GetOperationIdOk() (*string, bool) {`
- `sources/hindsight/hindsight-clients/go/model_delete_operation_response.go:112` `if o == nil {`
- `sources/hindsight/hindsight-clients/go/model_delete_operation_response.go:113` `return nil, false`
- `sources/hindsight/hindsight-clients/go/model_delete_operation_response.go:114` `}`
- `sources/hindsight/hindsight-clients/go/model_delete_operation_response.go:115` `return &o.OperationId, true`
- `sources/hindsight/hindsight-clients/go/model_delete_operation_response.go:118` `// SetOperationId sets field value`
- `sources/hindsight/hindsight-clients/go/model_delete_operation_response.go:119` `func (o *DeleteOperationResponse) SetOperationId(v string) {`
- `sources/hindsight/hindsight-clients/go/model_delete_operation_response.go:120` `o.OperationId = v`
- `sources/hindsight/hindsight-clients/go/model_delete_operation_response.go:121` `}`
- `sources/hindsight/hindsight-clients/go/model_delete_operation_response.go:122` ``
- `sources/hindsight/hindsight-clients/go/model_delete_operation_response.go:123` `func (o DeleteOperationResponse) MarshalJSON() ([]byte, error) {`
- `sources/hindsight/hindsight-clients/go/model_delete_operation_response.go:124` `toSerialize,err := o.ToMap()`
- `sources/hindsight/hindsight-clients/go/model_delete_operation_response.go:125` `if err != nil {`
- `sources/hindsight/hindsight-clients/go/model_delete_operation_response.go:126` `return []byte{}, err`
- `sources/hindsight/hindsight-clients/go/model_delete_operation_response.go:127` `}`
- `sources/hindsight/hindsight-clients/go/model_delete_operation_response.go:130` ``
- `sources/hindsight/hindsight-clients/go/model_delete_operation_response.go:131` `func (o DeleteOperationResponse) ToMap() (map[string]interface{}, error) {`
- `sources/hindsight/hindsight-clients/go/model_delete_operation_response.go:132` `toSerialize := map[string]interface{}{}`
- `sources/hindsight/hindsight-clients/go/model_delete_operation_response.go:133` `toSerialize["success"] = o.Success`
- `sources/hindsight/hindsight-clients/go/model_delete_operation_response.go:134` `toSerialize["message"] = o.Message`
- `sources/hindsight/hindsight-clients/go/model_delete_operation_response.go:135` `toSerialize["operation_id"] = o.OperationId`
- `sources/hindsight/hindsight-clients/go/model_delete_operation_response.go:138` ``
- `sources/hindsight/hindsight-clients/go/model_delete_operation_response.go:139` `func (o *DeleteOperationResponse) UnmarshalJSON(data []byte) (err error) {`
- `sources/hindsight/hindsight-clients/go/model_delete_operation_response.go:140` `// This validates that all required properties are included in the JSON object`
- `sources/hindsight/hindsight-clients/go/model_delete_operation_response.go:141` `// by unmarshalling the object into a generic map with string keys and checking`
- `sources/hindsight/hindsight-clients/go/model_delete_operation_response.go:142` `// that every required field exists as a key in the generic map.`
- `sources/hindsight/hindsight-clients/go/model_delete_operation_response.go:143` `requiredProperties := []string{`
- `sources/hindsight/hindsight-clients/go/model_delete_operation_response.go:162` ``
- `sources/hindsight/hindsight-clients/go/model_delete_operation_response.go:163` `varDeleteOperationResponse := _DeleteOperationResponse{}`
- `sources/hindsight/hindsight-clients/go/model_delete_operation_response.go:164` ``
- `sources/hindsight/hindsight-clients/go/model_delete_operation_response.go:165` `decoder := json.NewDecoder(bytes.NewReader(data))`
- `sources/hindsight/hindsight-clients/go/model_delete_operation_response.go:166` `err = decoder.Decode(&varDeleteOperationResponse)`
- `sources/hindsight/hindsight-clients/go/model_delete_operation_response.go:167` ``
- `sources/hindsight/hindsight-clients/go/model_delete_operation_response.go:168` `if err != nil {`
- `sources/hindsight/hindsight-clients/go/model_delete_operation_response.go:169` `return err`
- `sources/hindsight/hindsight-clients/go/model_delete_operation_response.go:170` `}`
- `sources/hindsight/hindsight-clients/go/model_delete_operation_response.go:176` ``
- `sources/hindsight/hindsight-clients/go/model_delete_operation_response.go:177` `type NullableDeleteOperationResponse struct {`
- `sources/hindsight/hindsight-clients/go/model_delete_operation_response.go:178` `value *DeleteOperationResponse`
- `sources/hindsight/hindsight-clients/go/model_delete_operation_response.go:179` `isSet bool`
- `sources/hindsight/hindsight-clients/go/model_delete_operation_response.go:180` `}`
- `sources/hindsight/hindsight-clients/go/model_delete_operation_response.go:181` ``
- `sources/hindsight/hindsight-clients/go/model_delete_operation_response.go:182` `func (v NullableDeleteOperationResponse) Get() *DeleteOperationResponse {`
- `sources/hindsight/hindsight-clients/go/model_delete_operation_response.go:183` `return v.value`
- `sources/hindsight/hindsight-clients/go/model_delete_operation_response.go:184` `}`
- `sources/hindsight/hindsight-clients/go/model_delete_operation_response.go:185` ``
- `sources/hindsight/hindsight-clients/go/model_delete_operation_response.go:186` `func (v *NullableDeleteOperationResponse) Set(val *DeleteOperationResponse) {`
- `sources/hindsight/hindsight-clients/go/model_delete_operation_response.go:187` `v.value = val`
- `sources/hindsight/hindsight-clients/go/model_delete_operation_response.go:188` `v.isSet = true`
- `sources/hindsight/hindsight-clients/go/model_delete_operation_response.go:189` `}`
- `sources/hindsight/hindsight-clients/go/model_delete_operation_response.go:190` ``
- `sources/hindsight/hindsight-clients/go/model_delete_operation_response.go:191` `func (v NullableDeleteOperationResponse) IsSet() bool {`
- `sources/hindsight/hindsight-clients/go/model_delete_operation_response.go:192` `return v.isSet`
- `sources/hindsight/hindsight-clients/go/model_delete_operation_response.go:193` `}`
- `sources/hindsight/hindsight-clients/go/model_delete_operation_response.go:194` ``
- `sources/hindsight/hindsight-clients/go/model_delete_operation_response.go:195` `func (v *NullableDeleteOperationResponse) Unset() {`
- `sources/hindsight/hindsight-clients/go/model_delete_operation_response.go:196` `v.value = nil`
- `sources/hindsight/hindsight-clients/go/model_delete_operation_response.go:197` `v.isSet = false`
- `sources/hindsight/hindsight-clients/go/model_delete_operation_response.go:198` `}`
- `sources/hindsight/hindsight-clients/go/model_delete_operation_response.go:199` ``
- `sources/hindsight/hindsight-clients/go/model_delete_operation_response.go:200` `func NewNullableDeleteOperationResponse(val *DeleteOperationResponse) *NullableDeleteOperationResponse {`
- `sources/hindsight/hindsight-clients/go/model_delete_operation_response.go:201` `return &NullableDeleteOperationResponse{value: val, isSet: true}`
- `sources/hindsight/hindsight-clients/go/model_delete_operation_response.go:202` `}`
- `sources/hindsight/hindsight-clients/go/model_delete_operation_response.go:203` ``
- `sources/hindsight/hindsight-clients/go/model_delete_operation_response.go:204` `func (v NullableDeleteOperationResponse) MarshalJSON() ([]byte, error) {`
- `sources/hindsight/hindsight-clients/go/model_delete_operation_response.go:205` `return json.Marshal(v.value)`
- `sources/hindsight/hindsight-clients/go/model_delete_operation_response.go:206` `}`
- `sources/hindsight/hindsight-clients/go/model_delete_operation_response.go:207` ``
- `sources/hindsight/hindsight-clients/go/model_delete_operation_response.go:208` `func (v *NullableDeleteOperationResponse) UnmarshalJSON(src []byte) error {`
- `sources/hindsight/hindsight-clients/go/model_delete_operation_response.go:209` `v.isSet = true`
- `sources/hindsight/hindsight-clients/go/model_delete_operation_response.go:210` `return json.Unmarshal(src, &v.value)`
- `sources/hindsight/hindsight-clients/go/model_delete_operation_response.go:211` `}`
- `sources/hindsight/hindsight-clients/go/model_delete_operation_response.go:212` ``

**operation and route call sites**, body `sources/hindsight/hindsight-clients/go/model_delete_response.go:20`; body evidence.

- `sources/hindsight/hindsight-clients/go/model_delete_response.go:19` `// checks if the DeleteResponse type satisfies the MappedNullable interface at compile time`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:20` `var _ MappedNullable = &DeleteResponse{}`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:21` ``
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:22` `// DeleteResponse Response model for delete operations.`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:23` `type DeleteResponse struct {`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:24` `Success bool 'json:"success"'`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:25` `Message NullableString 'json:"message,omitempty"'`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:26` `DeletedCount NullableInt32 'json:"deleted_count,omitempty"'`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:27` `}`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:28` ``
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:29` `type _DeleteResponse DeleteResponse`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:30` ``
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:31` `// NewDeleteResponse instantiates a new DeleteResponse object`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:32` `// This constructor will assign default values to properties that have it defined,`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:33` `// and makes sure properties required by API are set, but the set of arguments`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:34` `// will change when the set of required properties is changed`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:35` `func NewDeleteResponse(success bool) *DeleteResponse {`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:36` `this := DeleteResponse{}`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:37` `this.Success = success`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:38` `return &this`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:39` `}`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:40` ``
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:43` `// but it doesn't guarantee that properties required by API are set`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:44` `func NewDeleteResponseWithDefaults() *DeleteResponse {`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:45` `this := DeleteResponse{}`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:46` `return &this`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:47` `}`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:48` ``
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:49` `// GetSuccess returns the Success field value`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:50` `func (o *DeleteResponse) GetSuccess() bool {`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:51` `if o == nil {`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:52` `var ret bool`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:53` `return ret`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:54` `}`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:60` `// and a boolean to check if the value has been set.`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:61` `func (o *DeleteResponse) GetSuccessOk() (*bool, bool) {`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:62` `if o == nil {`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:63` `return nil, false`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:64` `}`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:65` `return &o.Success, true`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:68` `// SetSuccess sets field value`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:69` `func (o *DeleteResponse) SetSuccess(v bool) {`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:70` `o.Success = v`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:71` `}`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:72` ``
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:73` `// GetMessage returns the Message field value if set, zero value otherwise (both if not set or set to explicit null).`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:74` `func (o *DeleteResponse) GetMessage() string {`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:75` `if o == nil || IsNil(o.Message.Get()) {`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:76` `var ret string`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:77` `return ret`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:78` `}`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:84` `// NOTE: If the value is an explicit nil, 'nil, true' will be returned`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:85` `func (o *DeleteResponse) GetMessageOk() (*string, bool) {`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:86` `if o == nil {`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:87` `return nil, false`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:88` `}`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:89` `return o.Message.Get(), o.Message.IsSet()`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:92` `// HasMessage returns a boolean if a field has been set.`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:93` `func (o *DeleteResponse) HasMessage() bool {`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:94` `if o != nil && o.Message.IsSet() {`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:95` `return true`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:96` `}`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:97` ``
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:101` `// SetMessage gets a reference to the given NullableString and assigns it to the Message field.`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:102` `func (o *DeleteResponse) SetMessage(v string) {`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:103` `o.Message.Set(&v)`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:104` `}`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:105` `// SetMessageNil sets the value for Message to be an explicit nil`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:106` `func (o *DeleteResponse) SetMessageNil() {`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:107` `o.Message.Set(nil)`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:108` `}`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:109` ``
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:110` `// UnsetMessage ensures that no value is present for Message, not even an explicit nil`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:111` `func (o *DeleteResponse) UnsetMessage() {`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:112` `o.Message.Unset()`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:113` `}`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:114` ``
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:115` `// GetDeletedCount returns the DeletedCount field value if set, zero value otherwise (both if not set or set to explicit null).`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:116` `func (o *DeleteResponse) GetDeletedCount() int32 {`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:117` `if o == nil || IsNil(o.DeletedCount.Get()) {`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:118` `var ret int32`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:119` `return ret`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:120` `}`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:121` `return *o.DeletedCount.Get()`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:122` `}`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:123` ``
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:124` `// GetDeletedCountOk returns a tuple with the DeletedCount field value if set, nil otherwise`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:125` `// and a boolean to check if the value has been set.`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:126` `// NOTE: If the value is an explicit nil, 'nil, true' will be returned`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:127` `func (o *DeleteResponse) GetDeletedCountOk() (*int32, bool) {`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:128` `if o == nil {`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:129` `return nil, false`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:130` `}`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:131` `return o.DeletedCount.Get(), o.DeletedCount.IsSet()`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:132` `}`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:133` ``
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:134` `// HasDeletedCount returns a boolean if a field has been set.`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:135` `func (o *DeleteResponse) HasDeletedCount() bool {`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:136` `if o != nil && o.DeletedCount.IsSet() {`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:137` `return true`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:138` `}`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:139` ``
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:140` `return false`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:143` `// SetDeletedCount gets a reference to the given NullableInt32 and assigns it to the DeletedCount field.`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:144` `func (o *DeleteResponse) SetDeletedCount(v int32) {`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:145` `o.DeletedCount.Set(&v)`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:146` `}`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:147` `// SetDeletedCountNil sets the value for DeletedCount to be an explicit nil`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:148` `func (o *DeleteResponse) SetDeletedCountNil() {`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:149` `o.DeletedCount.Set(nil)`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:150` `}`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:151` ``
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:152` `// UnsetDeletedCount ensures that no value is present for DeletedCount, not even an explicit nil`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:153` `func (o *DeleteResponse) UnsetDeletedCount() {`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:154` `o.DeletedCount.Unset()`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:155` `}`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:156` ``
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:157` `func (o DeleteResponse) MarshalJSON() ([]byte, error) {`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:158` `toSerialize,err := o.ToMap()`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:159` `if err != nil {`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:160` `return []byte{}, err`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:161` `}`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:164` ``
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:165` `func (o DeleteResponse) ToMap() (map[string]interface{}, error) {`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:166` `toSerialize := map[string]interface{}{}`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:167` `toSerialize["success"] = o.Success`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:168` `if o.Message.IsSet() {`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:169` `toSerialize["message"] = o.Message.Get()`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:170` `}`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:171` `if o.DeletedCount.IsSet() {`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:172` `toSerialize["deleted_count"] = o.DeletedCount.Get()`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:173` `}`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:174` `return toSerialize, nil`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:175` `}`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:176` ``
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:177` `func (o *DeleteResponse) UnmarshalJSON(data []byte) (err error) {`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:178` `// This validates that all required properties are included in the JSON object`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:179` `// by unmarshalling the object into a generic map with string keys and checking`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:180` `// that every required field exists as a key in the generic map.`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:181` `requiredProperties := []string{`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:198` ``
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:199` `varDeleteResponse := _DeleteResponse{}`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:200` ``
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:201` `decoder := json.NewDecoder(bytes.NewReader(data))`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:202` `err = decoder.Decode(&varDeleteResponse)`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:203` ``
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:204` `if err != nil {`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:205` `return err`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:206` `}`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:212` ``
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:213` `type NullableDeleteResponse struct {`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:214` `value *DeleteResponse`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:215` `isSet bool`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:216` `}`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:217` ``
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:218` `func (v NullableDeleteResponse) Get() *DeleteResponse {`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:219` `return v.value`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:220` `}`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:221` ``
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:222` `func (v *NullableDeleteResponse) Set(val *DeleteResponse) {`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:223` `v.value = val`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:224` `v.isSet = true`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:225` `}`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:226` ``
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:227` `func (v NullableDeleteResponse) IsSet() bool {`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:228` `return v.isSet`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:229` `}`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:230` ``
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:231` `func (v *NullableDeleteResponse) Unset() {`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:232` `v.value = nil`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:233` `v.isSet = false`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:234` `}`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:235` ``
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:236` `func NewNullableDeleteResponse(val *DeleteResponse) *NullableDeleteResponse {`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:237` `return &NullableDeleteResponse{value: val, isSet: true}`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:238` `}`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:239` ``
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:240` `func (v NullableDeleteResponse) MarshalJSON() ([]byte, error) {`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:241` `return json.Marshal(v.value)`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:242` `}`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:243` ``
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:244` `func (v *NullableDeleteResponse) UnmarshalJSON(src []byte) error {`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:245` `v.isSet = true`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:246` `return json.Unmarshal(src, &v.value)`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:247` `}`
- `sources/hindsight/hindsight-clients/go/model_delete_response.go:248` ``

**operation and route call sites**, body `sources/hindsight/hindsight-clients/go/model_document_event_kind.go:28`; body evidence.

- `sources/hindsight/hindsight-clients/go/model_document_event_kind.go:27` `FACT_EDITED DocumentEventKind = "fact_edited"`
- `sources/hindsight/hindsight-clients/go/model_document_event_kind.go:28` `FACT_INVALIDATED DocumentEventKind = "fact_invalidated"`
- `sources/hindsight/hindsight-clients/go/model_document_event_kind.go:29` `FACT_RESTORED DocumentEventKind = "fact_restored"`
- `sources/hindsight/hindsight-clients/go/model_document_event_kind.go:30` `FACT_DELETED DocumentEventKind = "fact_deleted"`
- `sources/hindsight/hindsight-clients/go/model_document_event_kind.go:31` `OBSERVATIONS_CLEARED DocumentEventKind = "observations_cleared"`
- `sources/hindsight/hindsight-clients/go/model_document_event_kind.go:32` `CONSOLIDATION DocumentEventKind = "consolidation"`
- `sources/hindsight/hindsight-clients/go/model_document_event_kind.go:33` `)`
- `sources/hindsight/hindsight-clients/go/model_document_event_kind.go:34` ``
- `sources/hindsight/hindsight-clients/go/model_document_event_kind.go:41` `"fact_edited",`
- `sources/hindsight/hindsight-clients/go/model_document_event_kind.go:42` `"fact_invalidated",`
- `sources/hindsight/hindsight-clients/go/model_document_event_kind.go:43` `"fact_restored",`
- `sources/hindsight/hindsight-clients/go/model_document_event_kind.go:44` `"fact_deleted",`
- `sources/hindsight/hindsight-clients/go/model_document_event_kind.go:45` `"observations_cleared",`
- `sources/hindsight/hindsight-clients/go/model_document_event_kind.go:46` `"consolidation",`
- `sources/hindsight/hindsight-clients/go/model_document_event_kind.go:47` `}`
- `sources/hindsight/hindsight-clients/go/model_document_event_kind.go:48` ``

**operation and route call sites**, body `sources/hindsight/hindsight-clients/go/model_memory_unit_list_item.go:50`; body evidence.

- `sources/hindsight/hindsight-clients/go/model_memory_unit_list_item.go:49` `State *string 'json:"state,omitempty"'`
- `sources/hindsight/hindsight-clients/go/model_memory_unit_list_item.go:50` `InvalidationReason NullableString 'json:"invalidation_reason,omitempty"'`
- `sources/hindsight/hindsight-clients/go/model_memory_unit_list_item.go:51` `InvalidatedAt NullableString 'json:"invalidated_at,omitempty"'`
- `sources/hindsight/hindsight-clients/go/model_memory_unit_list_item.go:52` `EditedAt NullableString 'json:"edited_at,omitempty"'`
- `sources/hindsight/hindsight-clients/go/model_memory_unit_list_item.go:53` `UpdatedAt NullableString 'json:"updated_at,omitempty"'`
- `sources/hindsight/hindsight-clients/go/model_memory_unit_list_item.go:54` `// An observation's source facts; empty for a source fact`
- `sources/hindsight/hindsight-clients/go/model_memory_unit_list_item.go:55` `SourceMemoryIds []string 'json:"source_memory_ids,omitempty"'`
- `sources/hindsight/hindsight-clients/go/model_memory_unit_list_item.go:718` `// GetInvalidationReason returns the InvalidationReason field value if set, zero value otherwise (both if not set or set to explicit null).`
- `sources/hindsight/hindsight-clients/go/model_memory_unit_list_item.go:719` `func (o *MemoryUnitListItem) GetInvalidationReason() string {`
- `sources/hindsight/hindsight-clients/go/model_memory_unit_list_item.go:720` `if o == nil || IsNil(o.InvalidationReason.Get()) {`
- `sources/hindsight/hindsight-clients/go/model_memory_unit_list_item.go:721` `var ret string`
- `sources/hindsight/hindsight-clients/go/model_memory_unit_list_item.go:722` `return ret`
- `sources/hindsight/hindsight-clients/go/model_memory_unit_list_item.go:723` `}`
- `sources/hindsight/hindsight-clients/go/model_memory_unit_list_item.go:724` `return *o.InvalidationReason.Get()`
- `sources/hindsight/hindsight-clients/go/model_memory_unit_list_item.go:725` `}`
- `sources/hindsight/hindsight-clients/go/model_memory_unit_list_item.go:726` ``
- `sources/hindsight/hindsight-clients/go/model_memory_unit_list_item.go:727` `// GetInvalidationReasonOk returns a tuple with the InvalidationReason field value if set, nil otherwise`
- `sources/hindsight/hindsight-clients/go/model_memory_unit_list_item.go:728` `// and a boolean to check if the value has been set.`
- `sources/hindsight/hindsight-clients/go/model_memory_unit_list_item.go:729` `// NOTE: If the value is an explicit nil, 'nil, true' will be returned`
- `sources/hindsight/hindsight-clients/go/model_memory_unit_list_item.go:730` `func (o *MemoryUnitListItem) GetInvalidationReasonOk() (*string, bool) {`
- `sources/hindsight/hindsight-clients/go/model_memory_unit_list_item.go:731` `if o == nil {`
- `sources/hindsight/hindsight-clients/go/model_memory_unit_list_item.go:732` `return nil, false`
- `sources/hindsight/hindsight-clients/go/model_memory_unit_list_item.go:733` `}`
- `sources/hindsight/hindsight-clients/go/model_memory_unit_list_item.go:734` `return o.InvalidationReason.Get(), o.InvalidationReason.IsSet()`
- `sources/hindsight/hindsight-clients/go/model_memory_unit_list_item.go:735` `}`
- `sources/hindsight/hindsight-clients/go/model_memory_unit_list_item.go:736` ``
- `sources/hindsight/hindsight-clients/go/model_memory_unit_list_item.go:737` `// HasInvalidationReason returns a boolean if a field has been set.`
- `sources/hindsight/hindsight-clients/go/model_memory_unit_list_item.go:738` `func (o *MemoryUnitListItem) HasInvalidationReason() bool {`
- `sources/hindsight/hindsight-clients/go/model_memory_unit_list_item.go:739` `if o != nil && o.InvalidationReason.IsSet() {`
- `sources/hindsight/hindsight-clients/go/model_memory_unit_list_item.go:740` `return true`
- `sources/hindsight/hindsight-clients/go/model_memory_unit_list_item.go:741` `}`
- `sources/hindsight/hindsight-clients/go/model_memory_unit_list_item.go:742` ``
- `sources/hindsight/hindsight-clients/go/model_memory_unit_list_item.go:743` `return false`
- `sources/hindsight/hindsight-clients/go/model_memory_unit_list_item.go:746` `// SetInvalidationReason gets a reference to the given NullableString and assigns it to the InvalidationReason field.`
- `sources/hindsight/hindsight-clients/go/model_memory_unit_list_item.go:747` `func (o *MemoryUnitListItem) SetInvalidationReason(v string) {`
- `sources/hindsight/hindsight-clients/go/model_memory_unit_list_item.go:748` `o.InvalidationReason.Set(&v)`
- `sources/hindsight/hindsight-clients/go/model_memory_unit_list_item.go:749` `}`
- `sources/hindsight/hindsight-clients/go/model_memory_unit_list_item.go:750` `// SetInvalidationReasonNil sets the value for InvalidationReason to be an explicit nil`
- `sources/hindsight/hindsight-clients/go/model_memory_unit_list_item.go:751` `func (o *MemoryUnitListItem) SetInvalidationReasonNil() {`
- `sources/hindsight/hindsight-clients/go/model_memory_unit_list_item.go:752` `o.InvalidationReason.Set(nil)`
- `sources/hindsight/hindsight-clients/go/model_memory_unit_list_item.go:753` `}`
- `sources/hindsight/hindsight-clients/go/model_memory_unit_list_item.go:754` ``
- `sources/hindsight/hindsight-clients/go/model_memory_unit_list_item.go:755` `// UnsetInvalidationReason ensures that no value is present for InvalidationReason, not even an explicit nil`
- `sources/hindsight/hindsight-clients/go/model_memory_unit_list_item.go:756` `func (o *MemoryUnitListItem) UnsetInvalidationReason() {`
- `sources/hindsight/hindsight-clients/go/model_memory_unit_list_item.go:757` `o.InvalidationReason.Unset()`
- `sources/hindsight/hindsight-clients/go/model_memory_unit_list_item.go:758` `}`
- `sources/hindsight/hindsight-clients/go/model_memory_unit_list_item.go:759` ``
- `sources/hindsight/hindsight-clients/go/model_memory_unit_list_item.go:760` `// GetInvalidatedAt returns the InvalidatedAt field value if set, zero value otherwise (both if not set or set to explicit null).`
- `sources/hindsight/hindsight-clients/go/model_memory_unit_list_item.go:761` `func (o *MemoryUnitListItem) GetInvalidatedAt() string {`
- `sources/hindsight/hindsight-clients/go/model_memory_unit_list_item.go:762` `if o == nil || IsNil(o.InvalidatedAt.Get()) {`
- `sources/hindsight/hindsight-clients/go/model_memory_unit_list_item.go:763` `var ret string`
- `sources/hindsight/hindsight-clients/go/model_memory_unit_list_item.go:764` `return ret`
- `sources/hindsight/hindsight-clients/go/model_memory_unit_list_item.go:765` `}`
- `sources/hindsight/hindsight-clients/go/model_memory_unit_list_item.go:766` `return *o.InvalidatedAt.Get()`
- `sources/hindsight/hindsight-clients/go/model_memory_unit_list_item.go:767` `}`
- `sources/hindsight/hindsight-clients/go/model_memory_unit_list_item.go:768` ``
- `sources/hindsight/hindsight-clients/go/model_memory_unit_list_item.go:769` `// GetInvalidatedAtOk returns a tuple with the InvalidatedAt field value if set, nil otherwise`
- `sources/hindsight/hindsight-clients/go/model_memory_unit_list_item.go:770` `// and a boolean to check if the value has been set.`
- `sources/hindsight/hindsight-clients/go/model_memory_unit_list_item.go:771` `// NOTE: If the value is an explicit nil, 'nil, true' will be returned`
- `sources/hindsight/hindsight-clients/go/model_memory_unit_list_item.go:772` `func (o *MemoryUnitListItem) GetInvalidatedAtOk() (*string, bool) {`
- `sources/hindsight/hindsight-clients/go/model_memory_unit_list_item.go:773` `if o == nil {`
- `sources/hindsight/hindsight-clients/go/model_memory_unit_list_item.go:774` `return nil, false`
- `sources/hindsight/hindsight-clients/go/model_memory_unit_list_item.go:775` `}`
- `sources/hindsight/hindsight-clients/go/model_memory_unit_list_item.go:776` `return o.InvalidatedAt.Get(), o.InvalidatedAt.IsSet()`
- `sources/hindsight/hindsight-clients/go/model_memory_unit_list_item.go:777` `}`
- `sources/hindsight/hindsight-clients/go/model_memory_unit_list_item.go:778` ``
- `sources/hindsight/hindsight-clients/go/model_memory_unit_list_item.go:779` `// HasInvalidatedAt returns a boolean if a field has been set.`
- `sources/hindsight/hindsight-clients/go/model_memory_unit_list_item.go:780` `func (o *MemoryUnitListItem) HasInvalidatedAt() bool {`
- `sources/hindsight/hindsight-clients/go/model_memory_unit_list_item.go:781` `if o != nil && o.InvalidatedAt.IsSet() {`
- `sources/hindsight/hindsight-clients/go/model_memory_unit_list_item.go:782` `return true`
- `sources/hindsight/hindsight-clients/go/model_memory_unit_list_item.go:783` `}`
- `sources/hindsight/hindsight-clients/go/model_memory_unit_list_item.go:784` ``
- `sources/hindsight/hindsight-clients/go/model_memory_unit_list_item.go:785` `return false`
- `sources/hindsight/hindsight-clients/go/model_memory_unit_list_item.go:788` `// SetInvalidatedAt gets a reference to the given NullableString and assigns it to the InvalidatedAt field.`
- `sources/hindsight/hindsight-clients/go/model_memory_unit_list_item.go:789` `func (o *MemoryUnitListItem) SetInvalidatedAt(v string) {`
- `sources/hindsight/hindsight-clients/go/model_memory_unit_list_item.go:790` `o.InvalidatedAt.Set(&v)`
- `sources/hindsight/hindsight-clients/go/model_memory_unit_list_item.go:791` `}`
- `sources/hindsight/hindsight-clients/go/model_memory_unit_list_item.go:792` `// SetInvalidatedAtNil sets the value for InvalidatedAt to be an explicit nil`
- `sources/hindsight/hindsight-clients/go/model_memory_unit_list_item.go:793` `func (o *MemoryUnitListItem) SetInvalidatedAtNil() {`
- `sources/hindsight/hindsight-clients/go/model_memory_unit_list_item.go:794` `o.InvalidatedAt.Set(nil)`
- `sources/hindsight/hindsight-clients/go/model_memory_unit_list_item.go:795` `}`
- `sources/hindsight/hindsight-clients/go/model_memory_unit_list_item.go:796` ``
- `sources/hindsight/hindsight-clients/go/model_memory_unit_list_item.go:797` `// UnsetInvalidatedAt ensures that no value is present for InvalidatedAt, not even an explicit nil`
- `sources/hindsight/hindsight-clients/go/model_memory_unit_list_item.go:798` `func (o *MemoryUnitListItem) UnsetInvalidatedAt() {`
- `sources/hindsight/hindsight-clients/go/model_memory_unit_list_item.go:799` `o.InvalidatedAt.Unset()`
- `sources/hindsight/hindsight-clients/go/model_memory_unit_list_item.go:800` `}`
- `sources/hindsight/hindsight-clients/go/model_memory_unit_list_item.go:801` ``
- `sources/hindsight/hindsight-clients/go/model_memory_unit_list_item.go:802` `// GetEditedAt returns the EditedAt field value if set, zero value otherwise (both if not set or set to explicit null).`
- `sources/hindsight/hindsight-clients/go/model_memory_unit_list_item.go:803` `func (o *MemoryUnitListItem) GetEditedAt() string {`
- `sources/hindsight/hindsight-clients/go/model_memory_unit_list_item.go:976` `}`
- `sources/hindsight/hindsight-clients/go/model_memory_unit_list_item.go:977` `if o.InvalidationReason.IsSet() {`
- `sources/hindsight/hindsight-clients/go/model_memory_unit_list_item.go:978` `toSerialize["invalidation_reason"] = o.InvalidationReason.Get()`
- `sources/hindsight/hindsight-clients/go/model_memory_unit_list_item.go:979` `}`
- `sources/hindsight/hindsight-clients/go/model_memory_unit_list_item.go:980` `if o.InvalidatedAt.IsSet() {`
- `sources/hindsight/hindsight-clients/go/model_memory_unit_list_item.go:981` `toSerialize["invalidated_at"] = o.InvalidatedAt.Get()`
- `sources/hindsight/hindsight-clients/go/model_memory_unit_list_item.go:982` `}`
- `sources/hindsight/hindsight-clients/go/model_memory_unit_list_item.go:983` `if o.EditedAt.IsSet() {`
- `sources/hindsight/hindsight-clients/go/model_memory_unit_list_item.go:984` `toSerialize["edited_at"] = o.EditedAt.Get()`
- `sources/hindsight/hindsight-clients/go/model_memory_unit_list_item.go:985` `}`

**delete_mental_model**, body `sources/hindsight/hindsight-clients/python/hindsight_client/hindsight_client.py:1929`; implementation/call-through.

- `sources/hindsight/hindsight-clients/python/hindsight_client/hindsight_client.py:1923` `def delete_mental_model(self, bank_id: str, mental_model_id: str):`
- `sources/hindsight/hindsight-clients/python/hindsight_client/hindsight_client.py:1929` `return _run_async(self.adelete_mental_model(bank_id=bank_id, mental_model_id=mental_model_id))`

**adelete_mental_model**, body `sources/hindsight/hindsight-clients/python/hindsight_client/hindsight_client.py:1939`; implementation/call-through.

- `sources/hindsight/hindsight-clients/python/hindsight_client/hindsight_client.py:1931` `async def adelete_mental_model(self, bank_id: str, mental_model_id: str):`
- `sources/hindsight/hindsight-clients/python/hindsight_client/hindsight_client.py:1939` `return await self._mental_models_api.delete_mental_model(`

**delete_knowledge_node**, body `sources/hindsight/hindsight-clients/python/hindsight_client/hindsight_client.py:2295`; implementation/call-through.

- `sources/hindsight/hindsight-clients/python/hindsight_client/hindsight_client.py:2289` `def delete_knowledge_node(self, bank_id: str, node_id: str):`
- `sources/hindsight/hindsight-clients/python/hindsight_client/hindsight_client.py:2295` `return _run_async(self.adelete_knowledge_node(bank_id=bank_id, node_id=node_id))`

**adelete_knowledge_node**, body `sources/hindsight/hindsight-clients/python/hindsight_client/hindsight_client.py:2305`; implementation/call-through.

- `sources/hindsight/hindsight-clients/python/hindsight_client/hindsight_client.py:2297` `async def adelete_knowledge_node(self, bank_id: str, node_id: str):`
- `sources/hindsight/hindsight-clients/python/hindsight_client/hindsight_client.py:2305` `return await self._knowledge_base_api.delete_knowledge_node(bank_id, node_id, _request_timeout=self._timeout)`

**_download_operation_archive**, body `sources/hindsight/hindsight-clients/python/hindsight_client/hindsight_client.py:2626`; implementation/call-through.

- `sources/hindsight/hindsight-clients/python/hindsight_client/hindsight_client.py:2614` `async def _download_operation_archive(`
- `sources/hindsight/hindsight-clients/python/hindsight_client/hindsight_client.py:2681` `archive = await response.read()`
- `sources/hindsight/hindsight-clients/python/hindsight_client/hindsight_client.py:2683` `# would otherwise hand the error body back as the archive.`
- `sources/hindsight/hindsight-clients/python/hindsight_client/hindsight_client.py:2685` `return bytes(archive)`

**delete_directive**, body `sources/hindsight/hindsight-clients/python/hindsight_client/hindsight_client.py:2883`; implementation/call-through.

- `sources/hindsight/hindsight-clients/python/hindsight_client/hindsight_client.py:2877` `def delete_directive(self, bank_id: str, directive_id: str):`
- `sources/hindsight/hindsight-clients/python/hindsight_client/hindsight_client.py:2883` `return _run_async(self.adelete_directive(bank_id=bank_id, directive_id=directive_id))`

**adelete_directive**, body `sources/hindsight/hindsight-clients/python/hindsight_client/hindsight_client.py:2893`; implementation/call-through.

- `sources/hindsight/hindsight-clients/python/hindsight_client/hindsight_client.py:2885` `async def adelete_directive(self, bank_id: str, directive_id: str):`
- `sources/hindsight/hindsight-clients/python/hindsight_client/hindsight_client.py:2893` `return await self._directives_api.delete_directive(bank_id, directive_id, _request_timeout=self._timeout)`

**delete_bank**, body `sources/hindsight/hindsight-clients/python/hindsight_client/hindsight_client.py:3358`; implementation/call-through.

- `sources/hindsight/hindsight-clients/python/hindsight_client/hindsight_client.py:3351` `def delete_bank(self, bank_id: str):`
- `sources/hindsight/hindsight-clients/python/hindsight_client/hindsight_client.py:3358` `return _run_async(self._banks_api.delete_bank(bank_id, _request_timeout=self._timeout))`

**adelete_bank**, body `sources/hindsight/hindsight-clients/python/hindsight_client/hindsight_client.py:3367`; implementation/call-through.

- `sources/hindsight/hindsight-clients/python/hindsight_client/hindsight_client.py:3360` `async def adelete_bank(self, bank_id: str):`
- `sources/hindsight/hindsight-clients/python/hindsight_client/hindsight_client.py:3367` `return await self._banks_api.delete_bank(bank_id, _request_timeout=self._timeout)`

**delete_bank**, body `sources/hindsight/hindsight-clients/python/hindsight_client_api/api/banks_api.py:1308`; implementation/call-through.

- `sources/hindsight/hindsight-clients/python/hindsight_client_api/api/banks_api.py:1260` `@validate_call`
- `sources/hindsight/hindsight-clients/python/hindsight_client_api/api/banks_api.py:1261` `async def delete_bank(`
- `sources/hindsight/hindsight-clients/python/hindsight_client_api/api/banks_api.py:1308` `_param = self._delete_bank_serialize(`

**delete_bank_with_http_info**, body `sources/hindsight/hindsight-clients/python/hindsight_client_api/api/banks_api.py:1380`; implementation/call-through.

- `sources/hindsight/hindsight-clients/python/hindsight_client_api/api/banks_api.py:1332` `@validate_call`
- `sources/hindsight/hindsight-clients/python/hindsight_client_api/api/banks_api.py:1333` `async def delete_bank_with_http_info(`
- `sources/hindsight/hindsight-clients/python/hindsight_client_api/api/banks_api.py:1380` `_param = self._delete_bank_serialize(`

**delete_bank_without_preload_content**, body `sources/hindsight/hindsight-clients/python/hindsight_client_api/api/banks_api.py:1452`; implementation/call-through.

- `sources/hindsight/hindsight-clients/python/hindsight_client_api/api/banks_api.py:1404` `@validate_call`
- `sources/hindsight/hindsight-clients/python/hindsight_client_api/api/banks_api.py:1405` `async def delete_bank_without_preload_content(`
- `sources/hindsight/hindsight-clients/python/hindsight_client_api/api/banks_api.py:1452` `_param = self._delete_bank_serialize(`

**_delete_bank_serialize**, body `sources/hindsight/hindsight-clients/python/hindsight_client_api/api/banks_api.py:1482`; implementation/call-through.

- `sources/hindsight/hindsight-clients/python/hindsight_client_api/api/banks_api.py:1472` `def _delete_bank_serialize(`
- `sources/hindsight/hindsight-clients/python/hindsight_client_api/api/banks_api.py:1482` `_host = None`

**delete_bank_alias**, body `sources/hindsight/hindsight-clients/python/hindsight_client_api/api/banks_api.py:1589`; implementation/call-through.

- `sources/hindsight/hindsight-clients/python/hindsight_client_api/api/banks_api.py:1538` `@validate_call`
- `sources/hindsight/hindsight-clients/python/hindsight_client_api/api/banks_api.py:1539` `async def delete_bank_alias(`
- `sources/hindsight/hindsight-clients/python/hindsight_client_api/api/banks_api.py:1589` `_param = self._delete_bank_alias_serialize(`

**delete_bank_alias_with_http_info**, body `sources/hindsight/hindsight-clients/python/hindsight_client_api/api/banks_api.py:1665`; implementation/call-through.

- `sources/hindsight/hindsight-clients/python/hindsight_client_api/api/banks_api.py:1614` `@validate_call`
- `sources/hindsight/hindsight-clients/python/hindsight_client_api/api/banks_api.py:1615` `async def delete_bank_alias_with_http_info(`
- `sources/hindsight/hindsight-clients/python/hindsight_client_api/api/banks_api.py:1665` `_param = self._delete_bank_alias_serialize(`

**delete_bank_alias_without_preload_content**, body `sources/hindsight/hindsight-clients/python/hindsight_client_api/api/banks_api.py:1741`; implementation/call-through.

- `sources/hindsight/hindsight-clients/python/hindsight_client_api/api/banks_api.py:1690` `@validate_call`
- `sources/hindsight/hindsight-clients/python/hindsight_client_api/api/banks_api.py:1691` `async def delete_bank_alias_without_preload_content(`
- `sources/hindsight/hindsight-clients/python/hindsight_client_api/api/banks_api.py:1741` `_param = self._delete_bank_alias_serialize(`

**_delete_bank_alias_serialize**, body `sources/hindsight/hindsight-clients/python/hindsight_client_api/api/banks_api.py:1773`; implementation/call-through.

- `sources/hindsight/hindsight-clients/python/hindsight_client_api/api/banks_api.py:1762` `def _delete_bank_alias_serialize(`
- `sources/hindsight/hindsight-clients/python/hindsight_client_api/api/banks_api.py:1773` `_host = None`

**delete_directive**, body `sources/hindsight/hindsight-clients/python/hindsight_client_api/api/directives_api.py:402`; implementation/call-through.

- `sources/hindsight/hindsight-clients/python/hindsight_client_api/api/directives_api.py:351` `@validate_call`
- `sources/hindsight/hindsight-clients/python/hindsight_client_api/api/directives_api.py:352` `async def delete_directive(`
- `sources/hindsight/hindsight-clients/python/hindsight_client_api/api/directives_api.py:402` `_param = self._delete_directive_serialize(`

**delete_directive_with_http_info**, body `sources/hindsight/hindsight-clients/python/hindsight_client_api/api/directives_api.py:478`; implementation/call-through.

- `sources/hindsight/hindsight-clients/python/hindsight_client_api/api/directives_api.py:427` `@validate_call`
- `sources/hindsight/hindsight-clients/python/hindsight_client_api/api/directives_api.py:428` `async def delete_directive_with_http_info(`
- `sources/hindsight/hindsight-clients/python/hindsight_client_api/api/directives_api.py:478` `_param = self._delete_directive_serialize(`

**delete_directive_without_preload_content**, body `sources/hindsight/hindsight-clients/python/hindsight_client_api/api/directives_api.py:554`; implementation/call-through.

- `sources/hindsight/hindsight-clients/python/hindsight_client_api/api/directives_api.py:503` `@validate_call`
- `sources/hindsight/hindsight-clients/python/hindsight_client_api/api/directives_api.py:504` `async def delete_directive_without_preload_content(`
- `sources/hindsight/hindsight-clients/python/hindsight_client_api/api/directives_api.py:554` `_param = self._delete_directive_serialize(`

**_delete_directive_serialize**, body `sources/hindsight/hindsight-clients/python/hindsight_client_api/api/directives_api.py:586`; implementation/call-through.

- `sources/hindsight/hindsight-clients/python/hindsight_client_api/api/directives_api.py:575` `def _delete_directive_serialize(`
- `sources/hindsight/hindsight-clients/python/hindsight_client_api/api/directives_api.py:586` `_host = None`

**delete_document**, body `sources/hindsight/hindsight-clients/python/hindsight_client_api/api/documents_api.py:101`; implementation/call-through.

- `sources/hindsight/hindsight-clients/python/hindsight_client_api/api/documents_api.py:50` `@validate_call`
- `sources/hindsight/hindsight-clients/python/hindsight_client_api/api/documents_api.py:51` `async def delete_document(`
- `sources/hindsight/hindsight-clients/python/hindsight_client_api/api/documents_api.py:101` `_param = self._delete_document_serialize(`

**delete_document_with_http_info**, body `sources/hindsight/hindsight-clients/python/hindsight_client_api/api/documents_api.py:177`; implementation/call-through.

- `sources/hindsight/hindsight-clients/python/hindsight_client_api/api/documents_api.py:126` `@validate_call`
- `sources/hindsight/hindsight-clients/python/hindsight_client_api/api/documents_api.py:127` `async def delete_document_with_http_info(`
- `sources/hindsight/hindsight-clients/python/hindsight_client_api/api/documents_api.py:177` `_param = self._delete_document_serialize(`

**delete_document_without_preload_content**, body `sources/hindsight/hindsight-clients/python/hindsight_client_api/api/documents_api.py:253`; implementation/call-through.

- `sources/hindsight/hindsight-clients/python/hindsight_client_api/api/documents_api.py:202` `@validate_call`
- `sources/hindsight/hindsight-clients/python/hindsight_client_api/api/documents_api.py:203` `async def delete_document_without_preload_content(`
- `sources/hindsight/hindsight-clients/python/hindsight_client_api/api/documents_api.py:253` `_param = self._delete_document_serialize(`

**_delete_document_serialize**, body `sources/hindsight/hindsight-clients/python/hindsight_client_api/api/documents_api.py:285`; implementation/call-through.

- `sources/hindsight/hindsight-clients/python/hindsight_client_api/api/documents_api.py:274` `def _delete_document_serialize(`
- `sources/hindsight/hindsight-clients/python/hindsight_client_api/api/documents_api.py:285` `_host = None`

**delete_knowledge_node**, body `sources/hindsight/hindsight-clients/python/hindsight_client_api/api/knowledge_base_api.py:713`; implementation/call-through.

- `sources/hindsight/hindsight-clients/python/hindsight_client_api/api/knowledge_base_api.py:662` `@validate_call`
- `sources/hindsight/hindsight-clients/python/hindsight_client_api/api/knowledge_base_api.py:663` `async def delete_knowledge_node(`
- `sources/hindsight/hindsight-clients/python/hindsight_client_api/api/knowledge_base_api.py:713` `_param = self._delete_knowledge_node_serialize(`

**delete_knowledge_node_with_http_info**, body `sources/hindsight/hindsight-clients/python/hindsight_client_api/api/knowledge_base_api.py:789`; implementation/call-through.

- `sources/hindsight/hindsight-clients/python/hindsight_client_api/api/knowledge_base_api.py:738` `@validate_call`
- `sources/hindsight/hindsight-clients/python/hindsight_client_api/api/knowledge_base_api.py:739` `async def delete_knowledge_node_with_http_info(`
- `sources/hindsight/hindsight-clients/python/hindsight_client_api/api/knowledge_base_api.py:789` `_param = self._delete_knowledge_node_serialize(`

**delete_knowledge_node_without_preload_content**, body `sources/hindsight/hindsight-clients/python/hindsight_client_api/api/knowledge_base_api.py:865`; implementation/call-through.

- `sources/hindsight/hindsight-clients/python/hindsight_client_api/api/knowledge_base_api.py:814` `@validate_call`
- `sources/hindsight/hindsight-clients/python/hindsight_client_api/api/knowledge_base_api.py:815` `async def delete_knowledge_node_without_preload_content(`
- `sources/hindsight/hindsight-clients/python/hindsight_client_api/api/knowledge_base_api.py:865` `_param = self._delete_knowledge_node_serialize(`

**_delete_knowledge_node_serialize**, body `sources/hindsight/hindsight-clients/python/hindsight_client_api/api/knowledge_base_api.py:897`; implementation/call-through.

- `sources/hindsight/hindsight-clients/python/hindsight_client_api/api/knowledge_base_api.py:886` `def _delete_knowledge_node_serialize(`
- `sources/hindsight/hindsight-clients/python/hindsight_client_api/api/knowledge_base_api.py:897` `_host = None`

**delete_mental_model**, body `sources/hindsight/hindsight-clients/python/hindsight_client_api/api/mental_models_api.py:699`; implementation/call-through.

- `sources/hindsight/hindsight-clients/python/hindsight_client_api/api/mental_models_api.py:648` `@validate_call`
- `sources/hindsight/hindsight-clients/python/hindsight_client_api/api/mental_models_api.py:649` `async def delete_mental_model(`
- `sources/hindsight/hindsight-clients/python/hindsight_client_api/api/mental_models_api.py:699` `_param = self._delete_mental_model_serialize(`

**delete_mental_model_with_http_info**, body `sources/hindsight/hindsight-clients/python/hindsight_client_api/api/mental_models_api.py:775`; implementation/call-through.

- `sources/hindsight/hindsight-clients/python/hindsight_client_api/api/mental_models_api.py:724` `@validate_call`
- `sources/hindsight/hindsight-clients/python/hindsight_client_api/api/mental_models_api.py:725` `async def delete_mental_model_with_http_info(`
- `sources/hindsight/hindsight-clients/python/hindsight_client_api/api/mental_models_api.py:775` `_param = self._delete_mental_model_serialize(`

**delete_mental_model_without_preload_content**, body `sources/hindsight/hindsight-clients/python/hindsight_client_api/api/mental_models_api.py:851`; implementation/call-through.

- `sources/hindsight/hindsight-clients/python/hindsight_client_api/api/mental_models_api.py:800` `@validate_call`
- `sources/hindsight/hindsight-clients/python/hindsight_client_api/api/mental_models_api.py:801` `async def delete_mental_model_without_preload_content(`
- `sources/hindsight/hindsight-clients/python/hindsight_client_api/api/mental_models_api.py:851` `_param = self._delete_mental_model_serialize(`

**_delete_mental_model_serialize**, body `sources/hindsight/hindsight-clients/python/hindsight_client_api/api/mental_models_api.py:883`; implementation/call-through.

- `sources/hindsight/hindsight-clients/python/hindsight_client_api/api/mental_models_api.py:872` `def _delete_mental_model_serialize(`
- `sources/hindsight/hindsight-clients/python/hindsight_client_api/api/mental_models_api.py:883` `_host = None`

**delete_operation**, body `sources/hindsight/hindsight-clients/python/hindsight_client_api/api/operations_api.py:390`; implementation/call-through.

- `sources/hindsight/hindsight-clients/python/hindsight_client_api/api/operations_api.py:339` `@validate_call`
- `sources/hindsight/hindsight-clients/python/hindsight_client_api/api/operations_api.py:340` `async def delete_operation(`
- `sources/hindsight/hindsight-clients/python/hindsight_client_api/api/operations_api.py:390` `_param = self._delete_operation_serialize(`

**delete_operation_with_http_info**, body `sources/hindsight/hindsight-clients/python/hindsight_client_api/api/operations_api.py:466`; implementation/call-through.

- `sources/hindsight/hindsight-clients/python/hindsight_client_api/api/operations_api.py:415` `@validate_call`
- `sources/hindsight/hindsight-clients/python/hindsight_client_api/api/operations_api.py:416` `async def delete_operation_with_http_info(`
- `sources/hindsight/hindsight-clients/python/hindsight_client_api/api/operations_api.py:466` `_param = self._delete_operation_serialize(`

**delete_operation_without_preload_content**, body `sources/hindsight/hindsight-clients/python/hindsight_client_api/api/operations_api.py:542`; implementation/call-through.

- `sources/hindsight/hindsight-clients/python/hindsight_client_api/api/operations_api.py:491` `@validate_call`
- `sources/hindsight/hindsight-clients/python/hindsight_client_api/api/operations_api.py:492` `async def delete_operation_without_preload_content(`
- `sources/hindsight/hindsight-clients/python/hindsight_client_api/api/operations_api.py:542` `_param = self._delete_operation_serialize(`

**_delete_operation_serialize**, body `sources/hindsight/hindsight-clients/python/hindsight_client_api/api/operations_api.py:574`; implementation/call-through.

- `sources/hindsight/hindsight-clients/python/hindsight_client_api/api/operations_api.py:563` `def _delete_operation_serialize(`
- `sources/hindsight/hindsight-clients/python/hindsight_client_api/api/operations_api.py:574` `_host = None`

**delete_webhook**, body `sources/hindsight/hindsight-clients/python/hindsight_client_api/api/webhooks_api.py:404`; implementation/call-through.

- `sources/hindsight/hindsight-clients/python/hindsight_client_api/api/webhooks_api.py:353` `@validate_call`
- `sources/hindsight/hindsight-clients/python/hindsight_client_api/api/webhooks_api.py:354` `async def delete_webhook(`
- `sources/hindsight/hindsight-clients/python/hindsight_client_api/api/webhooks_api.py:404` `_param = self._delete_webhook_serialize(`

**delete_webhook_with_http_info**, body `sources/hindsight/hindsight-clients/python/hindsight_client_api/api/webhooks_api.py:480`; implementation/call-through.

- `sources/hindsight/hindsight-clients/python/hindsight_client_api/api/webhooks_api.py:429` `@validate_call`
- `sources/hindsight/hindsight-clients/python/hindsight_client_api/api/webhooks_api.py:430` `async def delete_webhook_with_http_info(`
- `sources/hindsight/hindsight-clients/python/hindsight_client_api/api/webhooks_api.py:480` `_param = self._delete_webhook_serialize(`

**delete_webhook_without_preload_content**, body `sources/hindsight/hindsight-clients/python/hindsight_client_api/api/webhooks_api.py:556`; implementation/call-through.

- `sources/hindsight/hindsight-clients/python/hindsight_client_api/api/webhooks_api.py:505` `@validate_call`
- `sources/hindsight/hindsight-clients/python/hindsight_client_api/api/webhooks_api.py:506` `async def delete_webhook_without_preload_content(`
- `sources/hindsight/hindsight-clients/python/hindsight_client_api/api/webhooks_api.py:556` `_param = self._delete_webhook_serialize(`

**_delete_webhook_serialize**, body `sources/hindsight/hindsight-clients/python/hindsight_client_api/api/webhooks_api.py:588`; implementation/call-through.

- `sources/hindsight/hindsight-clients/python/hindsight_client_api/api/webhooks_api.py:577` `def _delete_webhook_serialize(`
- `sources/hindsight/hindsight-clients/python/hindsight_client_api/api/webhooks_api.py:588` `_host = None`

**operation and route call sites**, body `sources/hindsight/hindsight-clients/typescript/generated/client/client.gen.ts:64`; body evidence.

- `sources/hindsight/hindsight-clients/typescript/generated/client/client.gen.ts:63` `if (opts.body === undefined || opts.serializedBody === "") {`
- `sources/hindsight/hindsight-clients/typescript/generated/client/client.gen.ts:64` `opts.headers.delete("Content-Type");`
- `sources/hindsight/hindsight-clients/typescript/generated/client/client.gen.ts:65` `}`
- `sources/hindsight/hindsight-clients/typescript/generated/client/client.gen.ts:66` ``
- `sources/hindsight/hindsight-clients/typescript/generated/client/client.gen.ts:67` `const resolvedOpts = opts as typeof opts &`
- `sources/hindsight/hindsight-clients/typescript/generated/client/client.gen.ts:68` `ResolvedRequestOptions<TResponseStyle, ThrowOnError, Url>;`
- `sources/hindsight/hindsight-clients/typescript/generated/client/client.gen.ts:254` `connect: makeMethodFn("CONNECT"),`
- `sources/hindsight/hindsight-clients/typescript/generated/client/client.gen.ts:255` `delete: makeMethodFn("DELETE"),`
- `sources/hindsight/hindsight-clients/typescript/generated/client/client.gen.ts:256` `get: makeMethodFn("GET"),`
- `sources/hindsight/hindsight-clients/typescript/generated/client/client.gen.ts:257` `getConfig,`
- `sources/hindsight/hindsight-clients/typescript/generated/client/client.gen.ts:258` `head: makeMethodFn("HEAD"),`
- `sources/hindsight/hindsight-clients/typescript/generated/client/client.gen.ts:259` `interceptors,`
- `sources/hindsight/hindsight-clients/typescript/generated/client/client.gen.ts:267` `connect: makeSseFn("CONNECT"),`
- `sources/hindsight/hindsight-clients/typescript/generated/client/client.gen.ts:268` `delete: makeSseFn("DELETE"),`
- `sources/hindsight/hindsight-clients/typescript/generated/client/client.gen.ts:269` `get: makeSseFn("GET"),`
- `sources/hindsight/hindsight-clients/typescript/generated/client/client.gen.ts:270` `head: makeSseFn("HEAD"),`
- `sources/hindsight/hindsight-clients/typescript/generated/client/client.gen.ts:271` `options: makeSseFn("OPTIONS"),`
- `sources/hindsight/hindsight-clients/typescript/generated/client/client.gen.ts:272` `patch: makeSseFn("PATCH"),`

**operation and route call sites**, body `sources/hindsight/hindsight-clients/typescript/generated/client/utils.gen.ts:199`; body evidence.

- `sources/hindsight/hindsight-clients/typescript/generated/client/utils.gen.ts:198` `if (value === null) {`
- `sources/hindsight/hindsight-clients/typescript/generated/client/utils.gen.ts:199` `mergedHeaders.delete(key);`
- `sources/hindsight/hindsight-clients/typescript/generated/client/utils.gen.ts:200` `} else if (Array.isArray(value)) {`
- `sources/hindsight/hindsight-clients/typescript/generated/client/utils.gen.ts:201` `for (const v of value) {`
- `sources/hindsight/hindsight-clients/typescript/generated/client/utils.gen.ts:202` `mergedHeaders.append(key, v as string);`
- `sources/hindsight/hindsight-clients/typescript/generated/client/utils.gen.ts:203` `}`

**operation and route call sites**, body `sources/hindsight/hindsight-clients/typescript/generated/core/params.gen.ts:100`; body evidence.

- `sources/hindsight/hindsight-clients/typescript/generated/core/params.gen.ts:99` `if (value && typeof value === "object" && !Array.isArray(value) && !Object.keys(value).length) {`
- `sources/hindsight/hindsight-clients/typescript/generated/core/params.gen.ts:100` `delete params[slot as Slot];`
- `sources/hindsight/hindsight-clients/typescript/generated/core/params.gen.ts:101` `}`
- `sources/hindsight/hindsight-clients/typescript/generated/core/params.gen.ts:102` `}`
- `sources/hindsight/hindsight-clients/typescript/generated/core/params.gen.ts:103` `};`
- `sources/hindsight/hindsight-clients/typescript/generated/core/params.gen.ts:104` ``

**operation and route call sites**, body `sources/hindsight/hindsight-clients/typescript/generated/core/types.gen.ts:8`; body evidence.

- `sources/hindsight/hindsight-clients/typescript/generated/core/types.gen.ts:7` `| "connect"`
- `sources/hindsight/hindsight-clients/typescript/generated/core/types.gen.ts:8` `| "delete"`
- `sources/hindsight/hindsight-clients/typescript/generated/core/types.gen.ts:9` `| "get"`
- `sources/hindsight/hindsight-clients/typescript/generated/core/types.gen.ts:10` `| "head"`
- `sources/hindsight/hindsight-clients/typescript/generated/core/types.gen.ts:11` `| "options"`
- `sources/hindsight/hindsight-clients/typescript/generated/core/types.gen.ts:12` `| "patch"`

**operation and route call sites**, body `sources/hindsight/hindsight-clients/typescript/generated/index.ts:19`; body evidence.

- `sources/hindsight/hindsight-clients/typescript/generated/index.ts:18` `createWebhook,`
- `sources/hindsight/hindsight-clients/typescript/generated/index.ts:19` `deleteBank,`
- `sources/hindsight/hindsight-clients/typescript/generated/index.ts:20` `deleteBankAlias,`
- `sources/hindsight/hindsight-clients/typescript/generated/index.ts:21` `deleteDirective,`
- `sources/hindsight/hindsight-clients/typescript/generated/index.ts:22` `deleteDocument,`
- `sources/hindsight/hindsight-clients/typescript/generated/index.ts:23` `deleteKnowledgeNode,`
- `sources/hindsight/hindsight-clients/typescript/generated/index.ts:24` `deleteMentalModel,`
- `sources/hindsight/hindsight-clients/typescript/generated/index.ts:25` `deleteOperation,`
- `sources/hindsight/hindsight-clients/typescript/generated/index.ts:26` `deleteWebhook,`
- `sources/hindsight/hindsight-clients/typescript/generated/index.ts:27` `downloadFile,`
- `sources/hindsight/hindsight-clients/typescript/generated/index.ts:28` `dryRunExtractMemories,`
- `sources/hindsight/hindsight-clients/typescript/generated/index.ts:29` `dryRunRefreshMentalModel,`
- `sources/hindsight/hindsight-clients/typescript/generated/index.ts:30` `exportBankTemplate,`
- `sources/hindsight/hindsight-clients/typescript/generated/index.ts:235` `DefaultScopesPreview,`
- `sources/hindsight/hindsight-clients/typescript/generated/index.ts:236` `DeleteBankAliasData,`
- `sources/hindsight/hindsight-clients/typescript/generated/index.ts:237` `DeleteBankAliasError,`
- `sources/hindsight/hindsight-clients/typescript/generated/index.ts:238` `DeleteBankAliasErrors,`
- `sources/hindsight/hindsight-clients/typescript/generated/index.ts:239` `DeleteBankAliasResponse,`
- `sources/hindsight/hindsight-clients/typescript/generated/index.ts:240` `DeleteBankAliasResponses,`
- `sources/hindsight/hindsight-clients/typescript/generated/index.ts:241` `DeleteBankData,`
- `sources/hindsight/hindsight-clients/typescript/generated/index.ts:242` `DeleteBankError,`
- `sources/hindsight/hindsight-clients/typescript/generated/index.ts:243` `DeleteBankErrors,`
- `sources/hindsight/hindsight-clients/typescript/generated/index.ts:244` `DeleteBankResponse,`
- `sources/hindsight/hindsight-clients/typescript/generated/index.ts:245` `DeleteBankResponses,`
- `sources/hindsight/hindsight-clients/typescript/generated/index.ts:246` `DeleteDirectiveData,`
- `sources/hindsight/hindsight-clients/typescript/generated/index.ts:247` `DeleteDirectiveError,`
- `sources/hindsight/hindsight-clients/typescript/generated/index.ts:248` `DeleteDirectiveErrors,`
- `sources/hindsight/hindsight-clients/typescript/generated/index.ts:249` `DeleteDirectiveResponses,`
- `sources/hindsight/hindsight-clients/typescript/generated/index.ts:250` `DeleteDocumentData,`
- `sources/hindsight/hindsight-clients/typescript/generated/index.ts:251` `DeleteDocumentError,`
- `sources/hindsight/hindsight-clients/typescript/generated/index.ts:252` `DeleteDocumentErrors,`
- `sources/hindsight/hindsight-clients/typescript/generated/index.ts:253` `DeleteDocumentResponse,`
- `sources/hindsight/hindsight-clients/typescript/generated/index.ts:254` `DeleteDocumentResponse2,`
- `sources/hindsight/hindsight-clients/typescript/generated/index.ts:255` `DeleteDocumentResponses,`
- `sources/hindsight/hindsight-clients/typescript/generated/index.ts:256` `DeleteKnowledgeNodeData,`
- `sources/hindsight/hindsight-clients/typescript/generated/index.ts:257` `DeleteKnowledgeNodeError,`
- `sources/hindsight/hindsight-clients/typescript/generated/index.ts:258` `DeleteKnowledgeNodeErrors,`
- `sources/hindsight/hindsight-clients/typescript/generated/index.ts:259` `DeleteKnowledgeNodeResponses,`
- `sources/hindsight/hindsight-clients/typescript/generated/index.ts:260` `DeleteMentalModelData,`
- `sources/hindsight/hindsight-clients/typescript/generated/index.ts:261` `DeleteMentalModelError,`
- `sources/hindsight/hindsight-clients/typescript/generated/index.ts:262` `DeleteMentalModelErrors,`
- `sources/hindsight/hindsight-clients/typescript/generated/index.ts:263` `DeleteMentalModelResponses,`
- `sources/hindsight/hindsight-clients/typescript/generated/index.ts:264` `DeleteOperationData,`
- `sources/hindsight/hindsight-clients/typescript/generated/index.ts:265` `DeleteOperationError,`
- `sources/hindsight/hindsight-clients/typescript/generated/index.ts:266` `DeleteOperationErrors,`
- `sources/hindsight/hindsight-clients/typescript/generated/index.ts:267` `DeleteOperationResponse,`
- `sources/hindsight/hindsight-clients/typescript/generated/index.ts:268` `DeleteOperationResponse2,`
- `sources/hindsight/hindsight-clients/typescript/generated/index.ts:269` `DeleteOperationResponses,`
- `sources/hindsight/hindsight-clients/typescript/generated/index.ts:270` `DeleteResponse,`
- `sources/hindsight/hindsight-clients/typescript/generated/index.ts:271` `DeleteWebhookData,`
- `sources/hindsight/hindsight-clients/typescript/generated/index.ts:272` `DeleteWebhookError,`
- `sources/hindsight/hindsight-clients/typescript/generated/index.ts:273` `DeleteWebhookErrors,`
- `sources/hindsight/hindsight-clients/typescript/generated/index.ts:274` `DeleteWebhookResponse,`
- `sources/hindsight/hindsight-clients/typescript/generated/index.ts:275` `DeleteWebhookResponses,`
- `sources/hindsight/hindsight-clients/typescript/generated/index.ts:276` `DirectiveListResponse,`
- `sources/hindsight/hindsight-clients/typescript/generated/index.ts:277` `DirectiveResponse,`
- `sources/hindsight/hindsight-clients/typescript/generated/index.ts:278` `DispositionTraits,`
- `sources/hindsight/hindsight-clients/typescript/generated/index.ts:279` `DocumentEvent,`

**operation and route call sites**, body `sources/hindsight/hindsight-clients/typescript/generated/sdk.gen.ts:56`; body evidence.

- `sources/hindsight/hindsight-clients/typescript/generated/sdk.gen.ts:55` `CreateWebhookResponses,`
- `sources/hindsight/hindsight-clients/typescript/generated/sdk.gen.ts:56` `DeleteBankAliasData,`
- `sources/hindsight/hindsight-clients/typescript/generated/sdk.gen.ts:57` `DeleteBankAliasErrors,`
- `sources/hindsight/hindsight-clients/typescript/generated/sdk.gen.ts:58` `DeleteBankAliasResponses,`
- `sources/hindsight/hindsight-clients/typescript/generated/sdk.gen.ts:59` `DeleteBankData,`
- `sources/hindsight/hindsight-clients/typescript/generated/sdk.gen.ts:60` `DeleteBankErrors,`
- `sources/hindsight/hindsight-clients/typescript/generated/sdk.gen.ts:61` `DeleteBankResponses,`
- `sources/hindsight/hindsight-clients/typescript/generated/sdk.gen.ts:62` `DeleteDirectiveData,`
- `sources/hindsight/hindsight-clients/typescript/generated/sdk.gen.ts:63` `DeleteDirectiveErrors,`
- `sources/hindsight/hindsight-clients/typescript/generated/sdk.gen.ts:64` `DeleteDirectiveResponses,`
- `sources/hindsight/hindsight-clients/typescript/generated/sdk.gen.ts:65` `DeleteDocumentData,`
- `sources/hindsight/hindsight-clients/typescript/generated/sdk.gen.ts:66` `DeleteDocumentErrors,`
- `sources/hindsight/hindsight-clients/typescript/generated/sdk.gen.ts:67` `DeleteDocumentResponses,`
- `sources/hindsight/hindsight-clients/typescript/generated/sdk.gen.ts:68` `DeleteKnowledgeNodeData,`
- `sources/hindsight/hindsight-clients/typescript/generated/sdk.gen.ts:69` `DeleteKnowledgeNodeErrors,`
- `sources/hindsight/hindsight-clients/typescript/generated/sdk.gen.ts:70` `DeleteKnowledgeNodeResponses,`
- `sources/hindsight/hindsight-clients/typescript/generated/sdk.gen.ts:71` `DeleteMentalModelData,`
- `sources/hindsight/hindsight-clients/typescript/generated/sdk.gen.ts:72` `DeleteMentalModelErrors,`
- `sources/hindsight/hindsight-clients/typescript/generated/sdk.gen.ts:73` `DeleteMentalModelResponses,`
- `sources/hindsight/hindsight-clients/typescript/generated/sdk.gen.ts:74` `DeleteOperationData,`
- `sources/hindsight/hindsight-clients/typescript/generated/sdk.gen.ts:75` `DeleteOperationErrors,`
- `sources/hindsight/hindsight-clients/typescript/generated/sdk.gen.ts:76` `DeleteOperationResponses,`
- `sources/hindsight/hindsight-clients/typescript/generated/sdk.gen.ts:77` `DeleteWebhookData,`
- `sources/hindsight/hindsight-clients/typescript/generated/sdk.gen.ts:78` `DeleteWebhookErrors,`
- `sources/hindsight/hindsight-clients/typescript/generated/sdk.gen.ts:79` `DeleteWebhookResponses,`
- `sources/hindsight/hindsight-clients/typescript/generated/sdk.gen.ts:80` `DownloadFileData,`
- `sources/hindsight/hindsight-clients/typescript/generated/sdk.gen.ts:81` `DownloadFileErrors,`
- `sources/hindsight/hindsight-clients/typescript/generated/sdk.gen.ts:82` `DownloadFileResponses,`
- `sources/hindsight/hindsight-clients/typescript/generated/sdk.gen.ts:83` `DryRunExtractMemoriesData,`
- `sources/hindsight/hindsight-clients/typescript/generated/sdk.gen.ts:695` `*/`
- `sources/hindsight/hindsight-clients/typescript/generated/sdk.gen.ts:696` `export const deleteMentalModel = <ThrowOnError extends boolean = false>(`
- `sources/hindsight/hindsight-clients/typescript/generated/sdk.gen.ts:697` `options: Options<DeleteMentalModelData, ThrowOnError>`
- `sources/hindsight/hindsight-clients/typescript/generated/sdk.gen.ts:698` `) =>`
- `sources/hindsight/hindsight-clients/typescript/generated/sdk.gen.ts:699` `(options.client ?? client).delete<`
- `sources/hindsight/hindsight-clients/typescript/generated/sdk.gen.ts:700` `DeleteMentalModelResponses,`
- `sources/hindsight/hindsight-clients/typescript/generated/sdk.gen.ts:701` `DeleteMentalModelErrors,`
- `sources/hindsight/hindsight-clients/typescript/generated/sdk.gen.ts:702` `ThrowOnError`
- `sources/hindsight/hindsight-clients/typescript/generated/sdk.gen.ts:703` `>({ url: "/v1/default/banks/{bank_id}/mental-models/{mental_model_id}", ...options });`
- `sources/hindsight/hindsight-clients/typescript/generated/sdk.gen.ts:704` ``
- `sources/hindsight/hindsight-clients/typescript/generated/sdk.gen.ts:705` `/**`
- `sources/hindsight/hindsight-clients/typescript/generated/sdk.gen.ts:909` `*/`
- `sources/hindsight/hindsight-clients/typescript/generated/sdk.gen.ts:910` `export const deleteKnowledgeNode = <ThrowOnError extends boolean = false>(`
- `sources/hindsight/hindsight-clients/typescript/generated/sdk.gen.ts:911` `options: Options<DeleteKnowledgeNodeData, ThrowOnError>`
- `sources/hindsight/hindsight-clients/typescript/generated/sdk.gen.ts:912` `) =>`
- `sources/hindsight/hindsight-clients/typescript/generated/sdk.gen.ts:913` `(options.client ?? client).delete<`
- `sources/hindsight/hindsight-clients/typescript/generated/sdk.gen.ts:914` `DeleteKnowledgeNodeResponses,`
- `sources/hindsight/hindsight-clients/typescript/generated/sdk.gen.ts:915` `DeleteKnowledgeNodeErrors,`
- `sources/hindsight/hindsight-clients/typescript/generated/sdk.gen.ts:916` `ThrowOnError`
- `sources/hindsight/hindsight-clients/typescript/generated/sdk.gen.ts:917` `>({ url: "/v1/default/banks/{bank_id}/knowledge-base/nodes/{node_id}", ...options });`
- `sources/hindsight/hindsight-clients/typescript/generated/sdk.gen.ts:918` ``
- `sources/hindsight/hindsight-clients/typescript/generated/sdk.gen.ts:919` `/**`
- `sources/hindsight/hindsight-clients/typescript/generated/sdk.gen.ts:974` `*/`
- `sources/hindsight/hindsight-clients/typescript/generated/sdk.gen.ts:975` `export const deleteDirective = <ThrowOnError extends boolean = false>(`
- `sources/hindsight/hindsight-clients/typescript/generated/sdk.gen.ts:976` `options: Options<DeleteDirectiveData, ThrowOnError>`
- `sources/hindsight/hindsight-clients/typescript/generated/sdk.gen.ts:977` `) =>`
- `sources/hindsight/hindsight-clients/typescript/generated/sdk.gen.ts:978` `(options.client ?? client).delete<DeleteDirectiveResponses, DeleteDirectiveErrors, ThrowOnError>({`
- `sources/hindsight/hindsight-clients/typescript/generated/sdk.gen.ts:979` `url: "/v1/default/banks/{bank_id}/directives/{directive_id}",`
- `sources/hindsight/hindsight-clients/typescript/generated/sdk.gen.ts:980` `...options,`
- `sources/hindsight/hindsight-clients/typescript/generated/sdk.gen.ts:981` `});`
- `sources/hindsight/hindsight-clients/typescript/generated/sdk.gen.ts:982` ``
- `sources/hindsight/hindsight-clients/typescript/generated/sdk.gen.ts:1081` `*/`
- `sources/hindsight/hindsight-clients/typescript/generated/sdk.gen.ts:1082` `export const deleteDocument = <ThrowOnError extends boolean = false>(`
- `sources/hindsight/hindsight-clients/typescript/generated/sdk.gen.ts:1083` `options: Options<DeleteDocumentData, ThrowOnError>`
- `sources/hindsight/hindsight-clients/typescript/generated/sdk.gen.ts:1084` `) =>`
- `sources/hindsight/hindsight-clients/typescript/generated/sdk.gen.ts:1085` `(options.client ?? client).delete<DeleteDocumentResponses, DeleteDocumentErrors, ThrowOnError>({`
- `sources/hindsight/hindsight-clients/typescript/generated/sdk.gen.ts:1086` `url: "/v1/default/banks/{bank_id}/documents/{document_id}",`
- `sources/hindsight/hindsight-clients/typescript/generated/sdk.gen.ts:1087` `...options,`
- `sources/hindsight/hindsight-clients/typescript/generated/sdk.gen.ts:1088` `});`
- `sources/hindsight/hindsight-clients/typescript/generated/sdk.gen.ts:1089` ``
- `sources/hindsight/hindsight-clients/typescript/generated/sdk.gen.ts:1176` `) =>`
- `sources/hindsight/hindsight-clients/typescript/generated/sdk.gen.ts:1177` `(options.client ?? client).delete<CancelOperationResponses, CancelOperationErrors, ThrowOnError>({`
- `sources/hindsight/hindsight-clients/typescript/generated/sdk.gen.ts:1178` `url: "/v1/default/banks/{bank_id}/operations/{operation_id}",`
- `sources/hindsight/hindsight-clients/typescript/generated/sdk.gen.ts:1179` `...options,`
- `sources/hindsight/hindsight-clients/typescript/generated/sdk.gen.ts:1180` `});`
- `sources/hindsight/hindsight-clients/typescript/generated/sdk.gen.ts:1181` ``
- `sources/hindsight/hindsight-clients/typescript/generated/sdk.gen.ts:1213` `*/`
- `sources/hindsight/hindsight-clients/typescript/generated/sdk.gen.ts:1214` `export const deleteOperation = <ThrowOnError extends boolean = false>(`
- `sources/hindsight/hindsight-clients/typescript/generated/sdk.gen.ts:1215` `options: Options<DeleteOperationData, ThrowOnError>`
- `sources/hindsight/hindsight-clients/typescript/generated/sdk.gen.ts:1216` `) =>`
- `sources/hindsight/hindsight-clients/typescript/generated/sdk.gen.ts:1217` `(options.client ?? client).delete<DeleteOperationResponses, DeleteOperationErrors, ThrowOnError>({`
- `sources/hindsight/hindsight-clients/typescript/generated/sdk.gen.ts:1218` `url: "/v1/default/banks/{bank_id}/operations/{operation_id}/delete",`
- `sources/hindsight/hindsight-clients/typescript/generated/sdk.gen.ts:1219` `...options,`
- `sources/hindsight/hindsight-clients/typescript/generated/sdk.gen.ts:1220` `});`
- `sources/hindsight/hindsight-clients/typescript/generated/sdk.gen.ts:1221` ``
- `sources/hindsight/hindsight-clients/typescript/generated/sdk.gen.ts:1222` `/**`
- `sources/hindsight/hindsight-clients/typescript/generated/sdk.gen.ts:1319` `*/`
- `sources/hindsight/hindsight-clients/typescript/generated/sdk.gen.ts:1320` `export const deleteBankAlias = <ThrowOnError extends boolean = false>(`
- `sources/hindsight/hindsight-clients/typescript/generated/sdk.gen.ts:1321` `options: Options<DeleteBankAliasData, ThrowOnError>`
- `sources/hindsight/hindsight-clients/typescript/generated/sdk.gen.ts:1322` `) =>`
- `sources/hindsight/hindsight-clients/typescript/generated/sdk.gen.ts:1323` `(options.client ?? client).delete<DeleteBankAliasResponses, DeleteBankAliasErrors, ThrowOnError>({`
- `sources/hindsight/hindsight-clients/typescript/generated/sdk.gen.ts:1324` `url: "/v1/default/banks/{bank_id}/aliases/{alias}",`
- `sources/hindsight/hindsight-clients/typescript/generated/sdk.gen.ts:1325` `...options,`
- `sources/hindsight/hindsight-clients/typescript/generated/sdk.gen.ts:1326` `});`
- `sources/hindsight/hindsight-clients/typescript/generated/sdk.gen.ts:1327` ``
- `sources/hindsight/hindsight-clients/typescript/generated/sdk.gen.ts:1355` `*/`
- `sources/hindsight/hindsight-clients/typescript/generated/sdk.gen.ts:1356` `export const deleteBank = <ThrowOnError extends boolean = false>(`
- `sources/hindsight/hindsight-clients/typescript/generated/sdk.gen.ts:1357` `options: Options<DeleteBankData, ThrowOnError>`
- `sources/hindsight/hindsight-clients/typescript/generated/sdk.gen.ts:1358` `) =>`
- `sources/hindsight/hindsight-clients/typescript/generated/sdk.gen.ts:1359` `(options.client ?? client).delete<DeleteBankResponses, DeleteBankErrors, ThrowOnError>({`
- `sources/hindsight/hindsight-clients/typescript/generated/sdk.gen.ts:1360` `url: "/v1/default/banks/{bank_id}",`
- `sources/hindsight/hindsight-clients/typescript/generated/sdk.gen.ts:1361` `...options,`
- `sources/hindsight/hindsight-clients/typescript/generated/sdk.gen.ts:1362` `});`
- `sources/hindsight/hindsight-clients/typescript/generated/sdk.gen.ts:1363` ``
- `sources/hindsight/hindsight-clients/typescript/generated/sdk.gen.ts:1594` `) =>`
- `sources/hindsight/hindsight-clients/typescript/generated/sdk.gen.ts:1595` `(options.client ?? client).delete<`
- `sources/hindsight/hindsight-clients/typescript/generated/sdk.gen.ts:1596` `ClearObservationsResponses,`
- `sources/hindsight/hindsight-clients/typescript/generated/sdk.gen.ts:1597` `ClearObservationsErrors,`
- `sources/hindsight/hindsight-clients/typescript/generated/sdk.gen.ts:1598` `ThrowOnError`
- `sources/hindsight/hindsight-clients/typescript/generated/sdk.gen.ts:1599` `>({ url: "/v1/default/banks/{bank_id}/observations", ...options });`
- `sources/hindsight/hindsight-clients/typescript/generated/sdk.gen.ts:1657` `) =>`
- `sources/hindsight/hindsight-clients/typescript/generated/sdk.gen.ts:1658` `(options.client ?? client).delete<`
- `sources/hindsight/hindsight-clients/typescript/generated/sdk.gen.ts:1659` `ClearMemoryObservationsResponses,`
- `sources/hindsight/hindsight-clients/typescript/generated/sdk.gen.ts:1660` `ClearMemoryObservationsErrors,`
- `sources/hindsight/hindsight-clients/typescript/generated/sdk.gen.ts:1661` `ThrowOnError`
- `sources/hindsight/hindsight-clients/typescript/generated/sdk.gen.ts:1662` `>({ url: "/v1/default/banks/{bank_id}/memories/{memory_id}/observations", ...options });`
- `sources/hindsight/hindsight-clients/typescript/generated/sdk.gen.ts:1671` `) =>`
- `sources/hindsight/hindsight-clients/typescript/generated/sdk.gen.ts:1672` `(options.client ?? client).delete<ResetBankConfigResponses, ResetBankConfigErrors, ThrowOnError>({`
- `sources/hindsight/hindsight-clients/typescript/generated/sdk.gen.ts:1673` `url: "/v1/default/banks/{bank_id}/config",`
- `sources/hindsight/hindsight-clients/typescript/generated/sdk.gen.ts:1674` `...options,`
- `sources/hindsight/hindsight-clients/typescript/generated/sdk.gen.ts:1675` `});`
- `sources/hindsight/hindsight-clients/typescript/generated/sdk.gen.ts:1676` ``
- `sources/hindsight/hindsight-clients/typescript/generated/sdk.gen.ts:1764` `*/`
- `sources/hindsight/hindsight-clients/typescript/generated/sdk.gen.ts:1765` `export const deleteWebhook = <ThrowOnError extends boolean = false>(`
- `sources/hindsight/hindsight-clients/typescript/generated/sdk.gen.ts:1766` `options: Options<DeleteWebhookData, ThrowOnError>`
- `sources/hindsight/hindsight-clients/typescript/generated/sdk.gen.ts:1767` `) =>`
- `sources/hindsight/hindsight-clients/typescript/generated/sdk.gen.ts:1768` `(options.client ?? client).delete<DeleteWebhookResponses, DeleteWebhookErrors, ThrowOnError>({`
- `sources/hindsight/hindsight-clients/typescript/generated/sdk.gen.ts:1769` `url: "/v1/default/banks/{bank_id}/webhooks/{webhook_id}",`
- `sources/hindsight/hindsight-clients/typescript/generated/sdk.gen.ts:1770` `...options,`
- `sources/hindsight/hindsight-clients/typescript/generated/sdk.gen.ts:1771` `});`
- `sources/hindsight/hindsight-clients/typescript/generated/sdk.gen.ts:1772` ``
- `sources/hindsight/hindsight-clients/typescript/generated/sdk.gen.ts:1811` `) =>`
- `sources/hindsight/hindsight-clients/typescript/generated/sdk.gen.ts:1812` `(options.client ?? client).delete<`
- `sources/hindsight/hindsight-clients/typescript/generated/sdk.gen.ts:1813` `ClearBankMemoriesResponses,`
- `sources/hindsight/hindsight-clients/typescript/generated/sdk.gen.ts:1814` `ClearBankMemoriesErrors,`
- `sources/hindsight/hindsight-clients/typescript/generated/sdk.gen.ts:1815` `ThrowOnError`
- `sources/hindsight/hindsight-clients/typescript/generated/sdk.gen.ts:1816` `>({ url: "/v1/default/banks/{bank_id}/memories", ...options });`

**operation and route call sites**, body `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:1404`; body evidence.

- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:1403` `*/`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:1404` `deleted_count: number;`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:1405` `};`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:1406` ``
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:1407` `/**`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:1408` `* ConsolidationRequest`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:1940` `*/`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:1941` `export type DeleteDocumentResponse = {`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:1942` `/**`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:1943` `* Success`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:1944` `*/`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:1945` `success: boolean;`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:1956` `*/`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:1957` `memory_units_deleted: number;`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:1958` `};`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:1959` ``
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:1960` `/**`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:1961` `* DeleteOperationResponse`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:1964` `*/`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:1965` `export type DeleteOperationResponse = {`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:1966` `/**`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:1967` `* Success`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:1968` `*/`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:1969` `success: boolean;`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:1984` `*/`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:1985` `export type DeleteResponse = {`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:1986` `/**`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:1987` `* Success`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:1988` `*/`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:1989` `success: boolean;`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:1996` `*/`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:1997` `deleted_count?: number | null;`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:1998` `};`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:1999` ``
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:2000` `/**`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:2001` `* DirectiveListResponse`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:2151` `| "fact_edited"`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:2152` `| "fact_invalidated"`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:2153` `| "fact_restored"`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:2154` `| "fact_deleted"`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:2155` `| "observations_cleared"`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:2156` `| "consolidation";`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:2157` ``
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:2158` `/**`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:4357` `*/`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:4358` `invalidation_reason?: string | null;`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:4359` `/**`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:4360` `* Invalidated At`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:4361` `*`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:4362` `* When the fact was invalidated (ISO 8601)`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:4363` `*/`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:4364` `invalidated_at?: string | null;`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:4365` `/**`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:4366` `* Edited At`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:4367` `*`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:4368` `* When the fact was last edited by hand (ISO 8601)`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:8502` ``
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:8503` `export type DeleteMentalModelData = {`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:8504` `body?: never;`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:8505` `headers?: {`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:8506` `/**`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:8507` `* Authorization`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:8524` ``
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:8525` `export type DeleteMentalModelErrors = {`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:8526` `/**`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:8527` `* Validation Error`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:8528` `*/`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:8529` `422: HttpValidationError;`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:8531` ``
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:8532` `export type DeleteMentalModelError = DeleteMentalModelErrors[keyof DeleteMentalModelErrors];`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:8533` ``
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:8534` `export type DeleteMentalModelResponses = {`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:8535` `/**`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:8536` `* Successful Response`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:8537` `*/`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:8538` `200: unknown;`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:9082` ``
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:9083` `export type DeleteKnowledgeNodeData = {`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:9084` `body?: never;`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:9085` `headers?: {`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:9086` `/**`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:9087` `* Authorization`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:9104` ``
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:9105` `export type DeleteKnowledgeNodeErrors = {`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:9106` `/**`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:9107` `* Validation Error`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:9108` `*/`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:9109` `422: HttpValidationError;`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:9111` ``
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:9112` `export type DeleteKnowledgeNodeError = DeleteKnowledgeNodeErrors[keyof DeleteKnowledgeNodeErrors];`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:9113` ``
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:9114` `export type DeleteKnowledgeNodeResponses = {`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:9115` `/**`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:9116` `* Successful Response`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:9117` `*/`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:9118` `200: unknown;`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:9264` ``
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:9265` `export type DeleteDirectiveData = {`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:9266` `body?: never;`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:9267` `headers?: {`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:9268` `/**`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:9269` `* Authorization`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:9286` ``
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:9287` `export type DeleteDirectiveErrors = {`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:9288` `/**`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:9289` `* Validation Error`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:9290` `*/`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:9291` `422: HttpValidationError;`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:9293` ``
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:9294` `export type DeleteDirectiveError = DeleteDirectiveErrors[keyof DeleteDirectiveErrors];`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:9295` ``
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:9296` `export type DeleteDirectiveResponses = {`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:9297` `/**`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:9298` `* Successful Response`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:9299` `*/`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:9300` `200: unknown;`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:9616` ``
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:9617` `export type DeleteDocumentData = {`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:9618` `body?: never;`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:9619` `headers?: {`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:9620` `/**`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:9621` `* Authorization`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:9638` ``
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:9639` `export type DeleteDocumentErrors = {`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:9640` `/**`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:9641` `* Validation Error`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:9642` `*/`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:9643` `422: HttpValidationError;`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:9645` ``
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:9646` `export type DeleteDocumentError = DeleteDocumentErrors[keyof DeleteDocumentErrors];`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:9647` ``
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:9648` `export type DeleteDocumentResponses = {`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:9649` `/**`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:9650` `* Successful Response`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:9651` `*/`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:9652` `200: DeleteDocumentResponse;`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:9653` `};`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:9654` ``
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:9655` `export type DeleteDocumentResponse2 = DeleteDocumentResponses[keyof DeleteDocumentResponses];`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:9656` ``
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:9657` `export type GetDocumentData = {`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:9658` `body?: never;`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:9659` `headers?: {`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:10036` ``
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:10037` `export type DeleteOperationData = {`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:10038` `body?: never;`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:10039` `headers?: {`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:10040` `/**`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:10041` `* Authorization`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:10055` `query?: never;`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:10056` `url: "/v1/default/banks/{bank_id}/operations/{operation_id}/delete";`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:10057` `};`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:10058` ``
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:10059` `export type DeleteOperationErrors = {`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:10060` `/**`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:10061` `* Validation Error`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:10062` `*/`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:10063` `422: HttpValidationError;`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:10065` ``
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:10066` `export type DeleteOperationError = DeleteOperationErrors[keyof DeleteOperationErrors];`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:10067` ``
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:10068` `export type DeleteOperationResponses = {`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:10069` `/**`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:10070` `* Successful Response`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:10071` `*/`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:10072` `200: DeleteOperationResponse;`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:10073` `};`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:10074` ``
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:10075` `export type DeleteOperationResponse2 = DeleteOperationResponses[keyof DeleteOperationResponses];`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:10076` ``
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:10077` `export type GetBankProfileData = {`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:10078` `body?: never;`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:10079` `headers?: {`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:10259` ``
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:10260` `export type DeleteBankAliasData = {`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:10261` `body?: never;`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:10262` `headers?: {`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:10263` `/**`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:10264` `* Authorization`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:10281` ``
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:10282` `export type DeleteBankAliasErrors = {`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:10283` `/**`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:10284` `* Validation Error`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:10285` `*/`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:10286` `422: HttpValidationError;`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:10288` ``
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:10289` `export type DeleteBankAliasError = DeleteBankAliasErrors[keyof DeleteBankAliasErrors];`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:10290` ``
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:10291` `export type DeleteBankAliasResponses = {`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:10292` `/**`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:10293` `* Successful Response`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:10294` `*/`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:10295` `200: BankAliasesResponse;`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:10297` ``
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:10298` `export type DeleteBankAliasResponse = DeleteBankAliasResponses[keyof DeleteBankAliasResponses];`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:10299` ``
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:10300` `export type SetBankAliasPrimaryData = {`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:10301` `body: SetBankAliasPrimaryRequest;`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:10302` `headers?: {`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:10340` ``
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:10341` `export type DeleteBankData = {`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:10342` `body?: never;`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:10343` `headers?: {`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:10344` `/**`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:10345` `* Authorization`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:10358` ``
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:10359` `export type DeleteBankErrors = {`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:10360` `/**`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:10361` `* Validation Error`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:10362` `*/`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:10363` `422: HttpValidationError;`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:10365` ``
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:10366` `export type DeleteBankError = DeleteBankErrors[keyof DeleteBankErrors];`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:10367` ``
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:10368` `export type DeleteBankResponses = {`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:10369` `/**`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:10370` `* Successful Response`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:10371` `*/`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:10372` `200: DeleteResponse;`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:10373` `};`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:10374` ``
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:10375` `export type DeleteBankResponse = DeleteBankResponses[keyof DeleteBankResponses];`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:10376` ``
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:10377` `export type UpdateBankData = {`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:10378` `body: CreateBankRequest;`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:10379` `headers?: {`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:10999` `*/`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:11000` `200: DeleteResponse;`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:11001` `};`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:11002` ``
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:11003` `export type ClearObservationsResponse =`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:11004` `ClearObservationsResponses[keyof ClearObservationsResponses];`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:11424` ``
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:11425` `export type DeleteWebhookData = {`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:11426` `body?: never;`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:11427` `headers?: {`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:11428` `/**`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:11429` `* Authorization`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:11446` ``
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:11447` `export type DeleteWebhookErrors = {`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:11448` `/**`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:11449` `* Validation Error`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:11450` `*/`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:11451` `422: HttpValidationError;`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:11453` ``
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:11454` `export type DeleteWebhookError = DeleteWebhookErrors[keyof DeleteWebhookErrors];`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:11455` ``
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:11456` `export type DeleteWebhookResponses = {`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:11457` `/**`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:11458` `* Successful Response`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:11459` `*/`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:11460` `200: DeleteResponse;`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:11461` `};`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:11462` ``
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:11463` `export type DeleteWebhookResponse = DeleteWebhookResponses[keyof DeleteWebhookResponses];`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:11464` ``
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:11465` `export type UpdateWebhookData = {`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:11466` `body: UpdateWebhookRequest;`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:11467` `headers?: {`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:11597` `*/`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:11598` `200: DeleteResponse;`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:11599` `};`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:11600` ``
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:11601` `export type ClearBankMemoriesResponse =`
- `sources/hindsight/hindsight-clients/typescript/generated/types.gen.ts:11602` `ClearBankMemoriesResponses[keyof ClearBankMemoriesResponses];`

**operation and route call sites**, body `sources/hindsight/hindsight-clients/typescript/src/index.ts:734`; body evidence.

- `sources/hindsight/hindsight-clients/typescript/src/index.ts:733` `consolidationState?: "failed" | "pending" | "done";`
- `sources/hindsight/hindsight-clients/typescript/src/index.ts:734` `state?: "valid" | "invalidated";`
- `sources/hindsight/hindsight-clients/typescript/src/index.ts:735` `documentId?: string;`
- `sources/hindsight/hindsight-clients/typescript/src/index.ts:736` `entityId?: string;`
- `sources/hindsight/hindsight-clients/typescript/src/index.ts:737` `/**`
- `sources/hindsight/hindsight-clients/typescript/src/index.ts:738` `* Time axis to filter and order by. Also drops memories with no value on`
- `sources/hindsight/hindsight-clients/typescript/src/index.ts:1190` `*/`
- `sources/hindsight/hindsight-clients/typescript/src/index.ts:1191` `async deleteBank(bankId: string, options?: { signal?: AbortSignal }): Promise<void> {`
- `sources/hindsight/hindsight-clients/typescript/src/index.ts:1192` `const response = await sdk.deleteBank({`
- `sources/hindsight/hindsight-clients/typescript/src/index.ts:1193` `client: this.client,`
- `sources/hindsight/hindsight-clients/typescript/src/index.ts:1194` `path: { bank_id: bankId },`
- `sources/hindsight/hindsight-clients/typescript/src/index.ts:1195` `signal: options?.signal,`
- `sources/hindsight/hindsight-clients/typescript/src/index.ts:1196` `});`
- `sources/hindsight/hindsight-clients/typescript/src/index.ts:1197` `if (response.error) {`
- `sources/hindsight/hindsight-clients/typescript/src/index.ts:1198` `throw new Error('deleteBank failed: ${JSON.stringify(response.error)}');`
- `sources/hindsight/hindsight-clients/typescript/src/index.ts:1199` `}`
- `sources/hindsight/hindsight-clients/typescript/src/index.ts:1200` `}`
- `sources/hindsight/hindsight-clients/typescript/src/index.ts:1201` ``
- `sources/hindsight/hindsight-clients/typescript/src/index.ts:1202` `// Directive methods`
- `sources/hindsight/hindsight-clients/typescript/src/index.ts:1305` `*/`
- `sources/hindsight/hindsight-clients/typescript/src/index.ts:1306` `async deleteDirective(`
- `sources/hindsight/hindsight-clients/typescript/src/index.ts:1307` `bankId: string,`
- `sources/hindsight/hindsight-clients/typescript/src/index.ts:1308` `directiveId: string,`
- `sources/hindsight/hindsight-clients/typescript/src/index.ts:1309` `options?: { signal?: AbortSignal }`
- `sources/hindsight/hindsight-clients/typescript/src/index.ts:1310` `): Promise<void> {`
- `sources/hindsight/hindsight-clients/typescript/src/index.ts:1311` `const response = await sdk.deleteDirective({`
- `sources/hindsight/hindsight-clients/typescript/src/index.ts:1312` `client: this.client,`
- `sources/hindsight/hindsight-clients/typescript/src/index.ts:1313` `path: { bank_id: bankId, directive_id: directiveId },`
- `sources/hindsight/hindsight-clients/typescript/src/index.ts:1314` `signal: options?.signal,`
- `sources/hindsight/hindsight-clients/typescript/src/index.ts:1315` `});`
- `sources/hindsight/hindsight-clients/typescript/src/index.ts:1316` `if (response.error) {`
- `sources/hindsight/hindsight-clients/typescript/src/index.ts:1317` `throw new Error('deleteDirective failed: ${JSON.stringify(response.error)}');`
- `sources/hindsight/hindsight-clients/typescript/src/index.ts:1318` `}`
- `sources/hindsight/hindsight-clients/typescript/src/index.ts:1319` `}`
- `sources/hindsight/hindsight-clients/typescript/src/index.ts:1320` ``
- `sources/hindsight/hindsight-clients/typescript/src/index.ts:1321` `// Mental Model methods`
- `sources/hindsight/hindsight-clients/typescript/src/index.ts:1513` `*/`
- `sources/hindsight/hindsight-clients/typescript/src/index.ts:1514` `async deleteMentalModel(`
- `sources/hindsight/hindsight-clients/typescript/src/index.ts:1515` `bankId: string,`
- `sources/hindsight/hindsight-clients/typescript/src/index.ts:1516` `mentalModelId: string,`
- `sources/hindsight/hindsight-clients/typescript/src/index.ts:1517` `options?: { signal?: AbortSignal }`
- `sources/hindsight/hindsight-clients/typescript/src/index.ts:1518` `): Promise<void> {`
- `sources/hindsight/hindsight-clients/typescript/src/index.ts:1519` `const response = await sdk.deleteMentalModel({`
- `sources/hindsight/hindsight-clients/typescript/src/index.ts:1520` `client: this.client,`
- `sources/hindsight/hindsight-clients/typescript/src/index.ts:1521` `path: { bank_id: bankId, mental_model_id: mentalModelId },`
- `sources/hindsight/hindsight-clients/typescript/src/index.ts:1522` `signal: options?.signal,`
- `sources/hindsight/hindsight-clients/typescript/src/index.ts:1523` `});`
- `sources/hindsight/hindsight-clients/typescript/src/index.ts:1524` `if (response.error) {`
- `sources/hindsight/hindsight-clients/typescript/src/index.ts:1525` `throw new Error('deleteMentalModel failed: ${JSON.stringify(response.error)}');`
- `sources/hindsight/hindsight-clients/typescript/src/index.ts:1526` `}`
- `sources/hindsight/hindsight-clients/typescript/src/index.ts:1527` `}`
- `sources/hindsight/hindsight-clients/typescript/src/index.ts:1528` ``
- `sources/hindsight/hindsight-clients/typescript/src/index.ts:1529` `/**`
- `sources/hindsight/hindsight-clients/typescript/src/index.ts:1708` `*/`
- `sources/hindsight/hindsight-clients/typescript/src/index.ts:1709` `async deleteKnowledgeNode(`
- `sources/hindsight/hindsight-clients/typescript/src/index.ts:1710` `bankId: string,`
- `sources/hindsight/hindsight-clients/typescript/src/index.ts:1711` `nodeId: string,`
- `sources/hindsight/hindsight-clients/typescript/src/index.ts:1712` `options?: { signal?: AbortSignal }`
- `sources/hindsight/hindsight-clients/typescript/src/index.ts:1713` `): Promise<unknown> {`
- `sources/hindsight/hindsight-clients/typescript/src/index.ts:1714` `const response = await sdk.deleteKnowledgeNode({`
- `sources/hindsight/hindsight-clients/typescript/src/index.ts:1715` `client: this.client,`
- `sources/hindsight/hindsight-clients/typescript/src/index.ts:1716` `path: { bank_id: bankId, node_id: nodeId },`
- `sources/hindsight/hindsight-clients/typescript/src/index.ts:1717` `signal: options?.signal,`
- `sources/hindsight/hindsight-clients/typescript/src/index.ts:1718` `});`
- `sources/hindsight/hindsight-clients/typescript/src/index.ts:1719` ``
- `sources/hindsight/hindsight-clients/typescript/src/index.ts:1720` `return this.validateResponse(response, "deleteKnowledgeNode");`
- `sources/hindsight/hindsight-clients/typescript/src/index.ts:1721` `}`
- `sources/hindsight/hindsight-clients/typescript/src/index.ts:1722` ``
- `sources/hindsight/hindsight-clients/typescript/src/index.ts:1723` `/**`
- `sources/hindsight/hindsight-clients/typescript/src/index.ts:1724` `* Export the knowledge base as a portable markdown bundle.`
- `sources/hindsight/hindsight-clients/typescript/src/index.ts:1822` `*/`
- `sources/hindsight/hindsight-clients/typescript/src/index.ts:1823` `async deleteDocument(`
- `sources/hindsight/hindsight-clients/typescript/src/index.ts:1824` `bankId: string,`
- `sources/hindsight/hindsight-clients/typescript/src/index.ts:1825` `documentId: string,`
- `sources/hindsight/hindsight-clients/typescript/src/index.ts:1826` `options?: { signal?: AbortSignal }`
- `sources/hindsight/hindsight-clients/typescript/src/index.ts:1827` `): Promise<void> {`
- `sources/hindsight/hindsight-clients/typescript/src/index.ts:1828` `const response = await sdk.deleteDocument({`
- `sources/hindsight/hindsight-clients/typescript/src/index.ts:1829` `client: this.client,`
- `sources/hindsight/hindsight-clients/typescript/src/index.ts:1830` `path: { bank_id: bankId, document_id: documentId },`
- `sources/hindsight/hindsight-clients/typescript/src/index.ts:1831` `signal: options?.signal,`
- `sources/hindsight/hindsight-clients/typescript/src/index.ts:1832` `});`
- `sources/hindsight/hindsight-clients/typescript/src/index.ts:1833` `if (response.error) {`
- `sources/hindsight/hindsight-clients/typescript/src/index.ts:1834` `throw new Error('deleteDocument failed: ${JSON.stringify(response.error)}');`
- `sources/hindsight/hindsight-clients/typescript/src/index.ts:1835` `}`
- `sources/hindsight/hindsight-clients/typescript/src/index.ts:1836` `}`
- `sources/hindsight/hindsight-clients/typescript/src/index.ts:1837` ``
- `sources/hindsight/hindsight-clients/typescript/src/index.ts:1838` `/**`
- `sources/hindsight/hindsight-clients/typescript/src/index.ts:1893` `const submission = this.validateResponse(submitResponse, "exportDocuments");`
- `sources/hindsight/hindsight-clients/typescript/src/index.ts:1894` `return this.downloadOperationArchive(bankId, submission.operation_id, options);`
- `sources/hindsight/hindsight-clients/typescript/src/index.ts:1895` `}`
- `sources/hindsight/hindsight-clients/typescript/src/index.ts:1896` ``
- `sources/hindsight/hindsight-clients/typescript/src/index.ts:1897` `/**`
- `sources/hindsight/hindsight-clients/typescript/src/index.ts:1898` `* Export a whole bank as a transfer ZIP archive (blocking convenience).`
- `sources/hindsight/hindsight-clients/typescript/src/index.ts:1937` `const submission = this.validateResponse(submitResponse, "exportBankTransfer");`
- `sources/hindsight/hindsight-clients/typescript/src/index.ts:1938` `return this.downloadOperationArchive(bankId, submission.operation_id, options);`
- `sources/hindsight/hindsight-clients/typescript/src/index.ts:1939` `}`
- `sources/hindsight/hindsight-clients/typescript/src/index.ts:1940` ``
- `sources/hindsight/hindsight-clients/typescript/src/index.ts:1941` `/**`
- `sources/hindsight/hindsight-clients/typescript/src/index.ts:1942` `* Restore a bank archive into a fresh bank, and resolve with the operation id.`
- `sources/hindsight/hindsight-clients/typescript/src/index.ts:1953` `bankId: string,`
- `sources/hindsight/hindsight-clients/typescript/src/index.ts:1954` `archive: Blob | File,`
- `sources/hindsight/hindsight-clients/typescript/src/index.ts:1955` `options?: {`
- `sources/hindsight/hindsight-clients/typescript/src/index.ts:1956` `targetBankId?: string;`
- `sources/hindsight/hindsight-clients/typescript/src/index.ts:1957` `includeData?: boolean;`
- `sources/hindsight/hindsight-clients/typescript/src/index.ts:1958` `includeBankConfig?: boolean;`
- `sources/hindsight/hindsight-clients/typescript/src/index.ts:1975` `},`
- `sources/hindsight/hindsight-clients/typescript/src/index.ts:1976` `body: { file: archive },`
- `sources/hindsight/hindsight-clients/typescript/src/index.ts:1977` `signal: options?.signal,`
- `sources/hindsight/hindsight-clients/typescript/src/index.ts:1978` `});`
- `sources/hindsight/hindsight-clients/typescript/src/index.ts:1979` `const submission = this.validateResponse(response, "importBankTransfer");`
- `sources/hindsight/hindsight-clients/typescript/src/index.ts:1980` `return submission.operation_id;`
- `sources/hindsight/hindsight-clients/typescript/src/index.ts:2028` `*/`
- `sources/hindsight/hindsight-clients/typescript/src/index.ts:2029` `private async downloadOperationArchive(`
- `sources/hindsight/hindsight-clients/typescript/src/index.ts:2030` `bankId: string,`
- `sources/hindsight/hindsight-clients/typescript/src/index.ts:2031` `operationId: string,`
- `sources/hindsight/hindsight-clients/typescript/src/index.ts:2032` `options?: { pollIntervalMs?: number; timeoutMs?: number; signal?: AbortSignal }`
- `sources/hindsight/hindsight-clients/typescript/src/index.ts:2033` `): Promise<Uint8Array> {`

</details>

## 3. What reaches the agent

System prompt identifies bank and budget (`sources/hindsight/hindsight-integrations/hermes/__init__.py:1666`). Recall uses a default 4096-token budget per bank, not a single global character limit (`sources/hindsight/hindsight-integrations/hermes/__init__.py:1569`; `sources/hindsight/hindsight-integrations/hermes/__init__.py:1700`). Prefetch can use reflect or recall and returns nothing on disabled/failed branches; assembly and tool-result bodies are quoted below. Multiple banks can enlarge the total.

Host assembly: `hermes-agent/agent/memory_manager.py:352` joins provider system blocks. `hermes-agent/agent/turn_context.py:1081` requests prefetch for nontrivial turns; `hermes-agent/agent/turn_context.py:120` fences it; `hermes-agent/agent/turn_context.py:136` appends it to API-bound user content. Tool results are a third channel. Optional host spill is at `hermes-agent/agent/memory_manager.py:419`, not a universal provider cap. Built-in memory remains separately controlled (`hermes-agent/agent/system_prompt.py:542`).

<details>
<summary>Context and tool-result assembly quotes</summary>

`system_prompt_block`, body `sources/hindsight/hindsight-integrations/hermes/__init__.py:1662`:

- `sources/hindsight/hindsight-integrations/hermes/__init__.py:1666` `return f"# Hindsight Memory\nActive{label}. Bank: {self._bank_id}, budget: {self._budget}.\n{tail}"`

`_finish_prefetch`, body `sources/hindsight/hindsight-integrations/hermes/__init__.py:1782`:

- `sources/hindsight/hindsight-integrations/hermes/__init__.py:1785` `return ""`
- `sources/hindsight/hindsight-integrations/hermes/__init__.py:1792` `return f"{header}\n\n{result}"`

`_join_prefetch`, body `sources/hindsight/hindsight-integrations/hermes/__init__.py:1795`:

- `sources/hindsight/hindsight-integrations/hermes/__init__.py:1799` `self._prefetch_thread.join(timeout=timeout)`

`prefetch`, body `sources/hindsight/hindsight-integrations/hermes/__init__.py:1805`:

- `sources/hindsight/hindsight-integrations/hermes/__init__.py:1806` `return self._finish_prefetch(*(("", 0) if self._recall_disabled() else self._do_recall(query)))`
- `sources/hindsight/hindsight-integrations/hermes/__init__.py:1812` `return self._finish_prefetch(result, count)`

`queue_prefetch`, body `sources/hindsight/hindsight-integrations/hermes/__init__.py:1822`:



`get_tool_schemas`, body `sources/hindsight/hindsight-integrations/hermes/__init__.py:2106`:

- `sources/hindsight/hindsight-integrations/hermes/__init__.py:2106` `return [] if self._memory_mode == "context" else [RETAIN_SCHEMA, RECALL_SCHEMA, REFLECT_SCHEMA]`

`_tool_retain`, body `sources/hindsight/hindsight-integrations/hermes/__init__.py:2109`:

- `sources/hindsight/hindsight-integrations/hermes/__init__.py:2128` `return "Memory queued for storage." if self._retain_async else "Memory stored successfully."`

`_tool_recall`, body `sources/hindsight/hindsight-integrations/hermes/__init__.py:2131`:

- `sources/hindsight/hindsight-integrations/hermes/__init__.py:2132` `logger.debug("Tool hindsight_recall: bank=%s, query_len=%d, budget=%s", self._bank_id, len(query), self._budget)`
- `sources/hindsight/hindsight-integrations/hermes/__init__.py:2135` `return "\n".join(f"{i}. {r.text}" for i, r in enumerate(results, 1)) or "No relevant memories found."`

`_tool_reflect`, body `sources/hindsight/hindsight-integrations/hermes/__init__.py:2138`:

- `sources/hindsight/hindsight-integrations/hermes/__init__.py:2140` `"Tool hindsight_reflect: bank=%s, query_len=%d, budget=%s", self._bank_id, len(query), self._budget`
- `sources/hindsight/hindsight-integrations/hermes/__init__.py:2144` `return text or "No relevant memories found."`

`handle_tool_call`, body `sources/hindsight/hindsight-integrations/hermes/__init__.py:2154`:

- `sources/hindsight/hindsight-integrations/hermes/__init__.py:2155` `return tool_error(f"Unknown tool: {tool_name}")`
- `sources/hindsight/hindsight-integrations/hermes/__init__.py:2158` `return tool_error(f"Missing required parameter: {required}")`
- `sources/hindsight/hindsight-integrations/hermes/__init__.py:2160` `return json.dumps({"result": handler(self, args)})`
- `sources/hindsight/hindsight-integrations/hermes/__init__.py:2163` `return tool_error(f"{failure}: {str(e) or type(e).__name__}")`

</details>

## 4. Data access boundaries

Direct access `yes`: configured HTTP client or local daemon/database, subject to API/database auth. Profile configuration is `$HERMES_HOME/hindsight/config.json` (`sources/hindsight/hindsight-integrations/hermes/__init__.py:640`), plus profile `.env`; legacy shared config is a fallback. Do not assert an `instance.json` path or mode from this repo: pg0's credential writer is outside this snapshot. Earlier observed database-password access is a historical run finding, not proof of the current dependency's layout. Accountless local mode `yes` with local LLM/embedding services; their configuration and resource needs remain. Telemetry default and configured outbound targets are recorded below; transitive pg0 behavior is unknown. OTLP tracing is off by default in retained configuration; explicit initialization exports to configured endpoint (`sources/hindsight/hindsight-api-slim/hindsight_api/config.py:2185`; `sources/hindsight/hindsight-api-slim/hindsight_api/tracing.py:198`).

Boundary model: same OS user, terminal, mounted filesystem and allowed network. Direct reachability means another client can use the same file/endpoint and available credentials, not unauthenticated access or encryption bypass. Hermes setup persists declared secrets through `hermes-agent/hermes_cli/memory_setup.py:321` and `hermes-agent/hermes_cli/config.py:2697`. These are possible configured paths; no live credentials were read.

Host credential-file detail: `get_hermes_home()/.env` is resolved at `hermes-agent/hermes_cli/config.py:473`. New files are secured to 0600 at `hermes-agent/hermes_cli/config.py:506`; the writer preserves an existing mode at `hermes-agent/hermes_cli/config.py:2544` and `:2562`. Therefore 0600 is not an unconditional mode claim for every existing profile.

<details>
<summary>Paths, environment, permissions, egress, telemetry and process-launch evidence</summary>

- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:32` `the host application's ''os.environ''. See issue #2961: doing so at module`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:322` `raw = os.getenv(operation_env)`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:324` `raw = os.getenv(ENV_LLM_TEMPERATURE)`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:337` `raw = os.getenv(operation_env)`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:350` `raw = os.getenv(operation_env)`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:352` `raw = os.getenv(ENV_LLM_STRICT_SCHEMA)`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:397` `raw = os.getenv(env_name)`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:415` `raw = os.getenv(env_name, "").strip()`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:482` `DEFAULT_FIREWORKS_BATCH_BASE_URL = "https://api.fireworks.ai"`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:1087` `raw_reserved = os.getenv(reserved_env)`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:1088` `raw_legacy = os.getenv(legacy_env)`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:1545` `DEFAULT_RERANKER_OPENROUTER_BASE_URL = "https://openrouter.ai/api/v1/rerank"`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:1553` `DEFAULT_ZEROENTROPY_BASE_URL = "https://api.zeroentropy.dev"`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:1564` `DEFAULT_RERANKER_SILICONFLOW_BASE_URL = "https://api.siliconflow.cn/v1"`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:1568` `DEFAULT_RERANKER_NVIDIA_BASE_URL = "https://ai.api.nvidia.com/v1"`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:1574` `DEFAULT_EMBEDDINGS_NVIDIA_BASE_URL = "https://integrate.api.nvidia.com/v1"`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:1581` `DEFAULT_RERANKER_TYPESAFE_BASE_URL = "https://api.typesafe.ai"`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:1585` `HINDSIGHT_INFERENCE_URL = "https://inference.hindsight.vectorize.io"`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:1622` `"typesafe": _DecisionProviderDefaults(model="jev-latest", base_url="https://api.typesafe.ai", tokenizer="jev"),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:1624` `"openai": _DecisionProviderDefaults(model="gpt-6-luna", base_url="https://api.openai.com"),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:1689` `DEFAULT_LITELLM_API_BASE = "http://localhost:4000"`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:1729` `DEFAULT_XAI_OAUTH_BASE_URL = "https://api.x.ai/v1"`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:2633` `explicit = os.getenv(ENV_REFLECT_LLM_TIMEOUT)`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:2636` `if os.getenv(ENV_LLM_TIMEOUT):`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:2644` `Reads the variable ONCE. The idiom this replaces -- ''int(os.getenv(X)) if os.getenv(X) else`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:2649` `raw = os.getenv(env_var)`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:2659` `raw = os.getenv(env_var)`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:2665` `raw = os.getenv(env_var)`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:2671` `raw = os.getenv(env_var)`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:2680` `provider = os.getenv(provider_env)`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:2691` `https://docs.litellm.ai/docs/routing.`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:2693` `raw = os.getenv(env_var, "").strip()`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:2875` `provider = os.getenv(base + "PROVIDER")`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:2879` `api_key = os.getenv(base + "API_KEY") or None`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:2885` `gemini_service_tier = os.getenv(base + "GEMINI_SERVICE_TIER")`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:2890` `model=os.getenv(base + "MODEL") or _get_default_model_for_provider(provider),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:2891` `base_url=os.getenv(base + "BASE_URL") or None,`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:2892` `reasoning_effort=os.getenv(base + "REASONING_EFFORT") or None,`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:2893` `extra_body=json.loads(os.getenv(base + "EXTRA_BODY", "null")),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:2894` `default_headers=json.loads(os.getenv(base + "DEFAULT_HEADERS", "null")),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:2895` `cache_affinity=os.getenv(base + "CACHE_AFFINITY") or None,`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:2896` `bedrock_service_tier=os.getenv(base + "BEDROCK_SERVICE_TIER") or None,`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:2900` `codex_home=os.getenv(base + "CODEX_HOME") or None,`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:2901` `vertexai_project_id=os.getenv(base + "VERTEXAI_PROJECT_ID") or None,`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:2902` `vertexai_region=os.getenv(base + "VERTEXAI_REGION") or None,`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:2903` `vertexai_service_account_key=os.getenv(base + "VERTEXAI_SERVICE_ACCOUNT_KEY") or None,`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:3056` `return os.getenv(base + suffix) or default`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:3061` `return os.getenv(base + suffix) or None`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:3065` `raw = os.getenv(base + suffix)`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:3070` `raw = os.getenv(base + suffix)`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:3080` `raw = os.getenv(base + suffix)`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:3090` `raw = os.getenv(base + suffix)`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:3100` `raw = os.getenv(base + suffix)`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:3123` `provider = os.getenv(base + "PROVIDER")`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4303` `flashrank_model=os.environ.get(ENV_RERANKER_FLASHRANK_MODEL, DEFAULT_RERANKER_FLASHRANK_MODEL),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4304` `flashrank_cache_dir=os.environ.get(ENV_RERANKER_FLASHRANK_CACHE_DIR, DEFAULT_RERANKER_FLASHRANK_CACHE_DIR),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4305` `flashrank_cpu_mem_arena=os.environ.get(`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4312` `os.environ.get(ENV_RERANKER_FLASHRANK_BATCH_SIZE, "").strip() or DEFAULT_RERANKER_FLASHRANK_BATCH_SIZE`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4598` `llm_provider = os.getenv(ENV_LLM_PROVIDER, DEFAULT_LLM_PROVIDER)`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4599` `llm_model = os.getenv(ENV_LLM_MODEL) or _get_default_model_for_provider(llm_provider)`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4615` `os.getenv(ENV_DATABASE_BACKEND, DEFAULT_DATABASE_BACKEND).lower(),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4617` `database_url=os.getenv(ENV_DATABASE_URL, DEFAULT_DATABASE_URL),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4618` `read_database_url=os.getenv(ENV_READ_DATABASE_URL) or None,`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4619` `read_db_pool_min_size=int(os.getenv(ENV_READ_DB_POOL_MIN_SIZE, str(DEFAULT_DB_POOL_MIN_SIZE))),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4620` `read_db_pool_max_size=int(os.getenv(ENV_READ_DB_POOL_MAX_SIZE, str(DEFAULT_DB_POOL_MAX_SIZE))),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4621` `migration_database_url=os.getenv(ENV_MIGRATION_DATABASE_URL) or None,`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4622` `database_schema=os.getenv(ENV_DATABASE_SCHEMA, DEFAULT_DATABASE_SCHEMA),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4623` `vector_extension=os.getenv(ENV_VECTOR_EXTENSION, DEFAULT_VECTOR_EXTENSION).lower(),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4626` `ENV_ANN_MAX_SCAN_TUPLES, os.getenv(ENV_ANN_MAX_SCAN_TUPLES), DEFAULT_ANN_MAX_SCAN_TUPLES`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4628` `text_search_extension=os.getenv(ENV_TEXT_SEARCH_EXTENSION, DEFAULT_TEXT_SEARCH_EXTENSION).lower(),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4629` `text_search_extension_native_language=os.getenv(`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4633` `text_search_extension_pg_search_tokenizer=os.getenv(`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4637` `text_search_extension_pg_search_function_schema=os.getenv(`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4643` `if (raw := os.getenv(ENV_QUERY_ANALYZER_LANGUAGES, "").strip())`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4646` `llm_output_language=(os.getenv(ENV_LLM_OUTPUT_LANGUAGE) or None),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4649` `llm_api_key=os.getenv(ENV_LLM_API_KEY),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4651` `llm_base_url=os.getenv(ENV_LLM_BASE_URL) or None,`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4652` `llm_max_concurrent=int(os.getenv(ENV_LLM_MAX_CONCURRENT, str(DEFAULT_LLM_MAX_CONCURRENT))),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4653` `llm_max_retries=int(os.getenv(ENV_LLM_MAX_RETRIES, str(DEFAULT_LLM_MAX_RETRIES))),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4654` `llm_initial_backoff=float(os.getenv(ENV_LLM_INITIAL_BACKOFF, str(DEFAULT_LLM_INITIAL_BACKOFF))),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4655` `llm_max_backoff=float(os.getenv(ENV_LLM_MAX_BACKOFF, str(DEFAULT_LLM_MAX_BACKOFF))),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4656` `llm_timeout=float(os.getenv(ENV_LLM_TIMEOUT, str(DEFAULT_LLM_TIMEOUT))),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4657` `llm_connect_timeout=float(os.getenv(ENV_LLM_CONNECT_TIMEOUT, str(DEFAULT_LLM_CONNECT_TIMEOUT))),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4658` `llm_http_log_level=os.getenv(ENV_LLM_HTTP_LOG_LEVEL, DEFAULT_LLM_HTTP_LOG_LEVEL),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4659` `llm_reasoning_effort=os.getenv(ENV_LLM_REASONING_EFFORT) or None,`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4660` `llm_groq_service_tier=os.getenv(ENV_LLM_GROQ_SERVICE_TIER, DEFAULT_LLM_GROQ_SERVICE_TIER),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4661` `llm_openai_service_tier=os.getenv(ENV_LLM_OPENAI_SERVICE_TIER, DEFAULT_LLM_OPENAI_SERVICE_TIER),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4662` `llm_bedrock_service_tier=os.getenv(ENV_LLM_BEDROCK_SERVICE_TIER) or None,`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4664` `parse_gemini_service_tier(os.getenv(ENV_LLM_GEMINI_SERVICE_TIER) or DEFAULT_LLM_GEMINI_SERVICE_TIER)`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4668` `llm_extra_body=json.loads(os.getenv(ENV_LLM_EXTRA_BODY, "null")),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4669` `llm_default_headers=json.loads(os.getenv(ENV_LLM_DEFAULT_HEADERS, "null")),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4670` `llm_cache_affinity=os.getenv(ENV_LLM_CACHE_AFFINITY, DEFAULT_LLM_CACHE_AFFINITY) or None,`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4671` `llm_strict_schema=os.getenv(ENV_LLM_STRICT_SCHEMA, str(DEFAULT_LLM_STRICT_SCHEMA)).lower() in ("true", "1"),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4683` `llm_send_bank_as_user=os.getenv(ENV_LLM_SEND_BANK_AS_USER, str(DEFAULT_LLM_SEND_BANK_AS_USER)).lower()`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4685` `llm_vision=_parse_tristate_bool(ENV_LLM_VISION, os.getenv(ENV_LLM_VISION)),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4687` `ENV_LLM_OPENAI_COMPATIBLE_JSON_MODE, os.getenv(ENV_LLM_OPENAI_COMPATIBLE_JSON_MODE)`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4691` `os.getenv(ENV_LLM_OLLAMA_NUM_CTX),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4708` `llm_codex_home=os.getenv(ENV_LLM_CODEX_HOME) or DEFAULT_LLM_CODEX_HOME,`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4709` `llm_chatgpt_home=os.getenv(ENV_LLM_CHATGPT_HOME, "").strip() or DEFAULT_LLM_CHATGPT_HOME,`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4711` `llm_vertexai_project_id=os.getenv(ENV_LLM_VERTEXAI_PROJECT_ID) or DEFAULT_LLM_VERTEXAI_PROJECT_ID,`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4712` `llm_vertexai_region=os.getenv(ENV_LLM_VERTEXAI_REGION, DEFAULT_LLM_VERTEXAI_REGION),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4713` `llm_vertexai_service_account_key=os.getenv(ENV_LLM_VERTEXAI_SERVICE_ACCOUNT_KEY)`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4716` `llm_gemini_safety_settings=json.loads(os.getenv(ENV_LLM_GEMINI_SAFETY_SETTINGS, "null")),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4717` `llm_prompt_cache_enabled=os.getenv(`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4721` `llm_debug_dump_4xx=os.getenv(ENV_LLM_DEBUG_DUMP_4XX, str(DEFAULT_LLM_DEBUG_DUMP_4XX)).lower()`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4724` `llamacpp_model_path=os.getenv(ENV_LLAMACPP_MODEL_PATH) or None,`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4725` `llamacpp_gpu_layers=int(os.getenv(ENV_LLAMACPP_GPU_LAYERS, str(DEFAULT_LLAMACPP_GPU_LAYERS))),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4726` `llamacpp_context_size=int(os.getenv(ENV_LLAMACPP_CONTEXT_SIZE, str(DEFAULT_LLAMACPP_CONTEXT_SIZE))),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4727` `llamacpp_chat_format=os.getenv(ENV_LLAMACPP_CHAT_FORMAT) or DEFAULT_LLAMACPP_CHAT_FORMAT,`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4728` `llamacpp_no_grammar=os.getenv(ENV_LLAMACPP_NO_GRAMMAR, str(DEFAULT_LLAMACPP_NO_GRAMMAR)).lower()`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4730` `llamacpp_extra_args=os.getenv(ENV_LLAMACPP_EXTRA_ARGS) or DEFAULT_LLAMACPP_EXTRA_ARGS,`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4732` `retain_llm_provider=os.getenv(ENV_RETAIN_LLM_PROVIDER) or None,`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4733` `retain_llm_api_key=os.getenv(ENV_RETAIN_LLM_API_KEY) or None,`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4734` `retain_llm_model=os.getenv(ENV_RETAIN_LLM_MODEL) or _env_default_model(ENV_RETAIN_LLM_PROVIDER),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4735` `retain_llm_base_url=os.getenv(ENV_RETAIN_LLM_BASE_URL) or None,`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4736` `vlm_provider=os.getenv(ENV_VLM_PROVIDER) or None,`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4737` `vlm_api_key=os.getenv(ENV_VLM_API_KEY) or None,`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4738` `vlm_model=os.getenv(ENV_VLM_MODEL) or _env_default_model(ENV_VLM_PROVIDER),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4739` `vlm_base_url=os.getenv(ENV_VLM_BASE_URL) or None,`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4740` `vlm_reasoning_effort=os.getenv(ENV_VLM_REASONING_EFFORT) or None,`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4741` `vlm_extra_body=json.loads(os.getenv(ENV_VLM_EXTRA_BODY, "null")),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4742` `vlm_cache_affinity=os.getenv(ENV_VLM_CACHE_AFFINITY) or None,`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4747` `fireworks_account_id=os.getenv(ENV_FIREWORKS_ACCOUNT_ID) or None,`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4748` `fireworks_batch_base_url=os.getenv(ENV_FIREWORKS_BATCH_BASE_URL) or DEFAULT_FIREWORKS_BATCH_BASE_URL,`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4750` `os.getenv(ENV_FIREWORKS_BATCH_MAX_WAIT_SECONDS, str(DEFAULT_FIREWORKS_BATCH_MAX_WAIT_SECONDS))`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4758` `retain_llm_reasoning_effort=os.getenv(ENV_RETAIN_LLM_REASONING_EFFORT) or None,`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4759` `retain_llm_extra_body=json.loads(os.getenv(ENV_RETAIN_LLM_EXTRA_BODY, "null")),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4760` `retain_llm_cache_affinity=os.getenv(ENV_RETAIN_LLM_CACHE_AFFINITY) or None,`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4761` `reflect_llm_provider=os.getenv(ENV_REFLECT_LLM_PROVIDER) or None,`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4762` `reflect_llm_api_key=os.getenv(ENV_REFLECT_LLM_API_KEY) or None,`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4763` `reflect_llm_model=os.getenv(ENV_REFLECT_LLM_MODEL) or _env_default_model(ENV_REFLECT_LLM_PROVIDER),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4764` `reflect_llm_base_url=os.getenv(ENV_REFLECT_LLM_BASE_URL) or None,`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4771` `reflect_llm_reasoning_effort=os.getenv(ENV_REFLECT_LLM_REASONING_EFFORT) or None,`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4772` `reflect_llm_extra_body=json.loads(os.getenv(ENV_REFLECT_LLM_EXTRA_BODY, "null")),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4773` `reflect_llm_cache_affinity=os.getenv(ENV_REFLECT_LLM_CACHE_AFFINITY) or None,`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4774` `consolidation_llm_provider=os.getenv(ENV_CONSOLIDATION_LLM_PROVIDER) or None,`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4775` `consolidation_llm_api_key=os.getenv(ENV_CONSOLIDATION_LLM_API_KEY) or None,`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4776` `consolidation_llm_model=os.getenv(ENV_CONSOLIDATION_LLM_MODEL)`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4778` `consolidation_llm_base_url=os.getenv(ENV_CONSOLIDATION_LLM_BASE_URL) or None,`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4785` `consolidation_llm_reasoning_effort=os.getenv(ENV_CONSOLIDATION_LLM_REASONING_EFFORT) or None,`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4786` `consolidation_llm_extra_body=json.loads(os.getenv(ENV_CONSOLIDATION_LLM_EXTRA_BODY, "null")),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4787` `consolidation_llm_cache_affinity=os.getenv(ENV_CONSOLIDATION_LLM_CACHE_AFFINITY) or None,`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4788` `mental_model_refresh_llm_provider=os.getenv(ENV_MENTAL_MODEL_REFRESH_LLM_PROVIDER) or None,`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4789` `mental_model_refresh_llm_api_key=os.getenv(ENV_MENTAL_MODEL_REFRESH_LLM_API_KEY) or None,`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4790` `mental_model_refresh_llm_model=os.getenv(ENV_MENTAL_MODEL_REFRESH_LLM_MODEL)`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4792` `mental_model_refresh_llm_base_url=os.getenv(ENV_MENTAL_MODEL_REFRESH_LLM_BASE_URL) or None,`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4801` `mental_model_refresh_llm_reasoning_effort=os.getenv(ENV_MENTAL_MODEL_REFRESH_LLM_REASONING_EFFORT) or None,`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4802` `mental_model_refresh_llm_extra_body=json.loads(os.getenv(ENV_MENTAL_MODEL_REFRESH_LLM_EXTRA_BODY, "null")),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4803` `mental_model_refresh_llm_cache_affinity=os.getenv(ENV_MENTAL_MODEL_REFRESH_LLM_CACHE_AFFINITY) or None,`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4806` `llm_strategy=_parse_llm_strategy(os.getenv(ENV_LLM_STRATEGY)),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4808` `retain_llm_strategy=_parse_llm_strategy(os.getenv(ENV_RETAIN_LLM_STRATEGY)),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4810` `reflect_llm_strategy=_parse_llm_strategy(os.getenv(ENV_REFLECT_LLM_STRATEGY)),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4812` `consolidation_llm_strategy=_parse_llm_strategy(os.getenv(ENV_CONSOLIDATION_LLM_STRATEGY)),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4814` `mental_model_refresh_llm_strategy=_parse_llm_strategy(os.getenv(ENV_MENTAL_MODEL_REFRESH_LLM_STRATEGY)),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4816` `embeddings_provider=os.getenv(ENV_EMBEDDINGS_PROVIDER, DEFAULT_EMBEDDINGS_PROVIDER),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4822` `v := os.getenv(ENV_EMBEDDINGS_MAX_INPUT_TOKENS)`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4823` `or os.getenv(ENV_EMBEDDINGS_LITELLM_SDK_MAX_INPUT_TOKENS)`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4826` `embeddings_local_model=os.getenv(ENV_EMBEDDINGS_LOCAL_MODEL, DEFAULT_EMBEDDINGS_LOCAL_MODEL),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4827` `embeddings_local_force_cpu=os.getenv(`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4831` `embeddings_local_trust_remote_code=os.getenv(`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4836` `os.getenv(ENV_EMBEDDINGS_LOCAL_MAX_CONCURRENT, str(DEFAULT_EMBEDDINGS_LOCAL_MAX_CONCURRENT))`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4838` `embeddings_onnx_model_id=os.getenv(ENV_EMBEDDINGS_ONNX_MODEL_ID, DEFAULT_EMBEDDINGS_ONNX_MODEL_ID),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4839` `embeddings_onnx_model_path=os.getenv(ENV_EMBEDDINGS_ONNX_MODEL_PATH) or None,`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4840` `embeddings_onnx_tokenizer_name_or_path=os.getenv(ENV_EMBEDDINGS_ONNX_TOKENIZER_NAME_OR_PATH) or None,`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4841` `embeddings_onnx_file=os.getenv(ENV_EMBEDDINGS_ONNX_FILE, DEFAULT_EMBEDDINGS_ONNX_FILE),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4844` `os.getenv(ENV_EMBEDDINGS_ONNX_DIMENSIONS),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4848` `os.getenv(ENV_EMBEDDINGS_ONNX_MAX_TOKENS),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4853` `os.getenv(ENV_EMBEDDINGS_ONNX_POOLING),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4857` `embeddings_onnx_normalize=os.getenv(`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4861` `embeddings_onnx_query_prefix=os.getenv(`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4864` `embeddings_onnx_passage_prefix=os.getenv(`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4867` `embeddings_onnx_output_name=os.getenv(ENV_EMBEDDINGS_ONNX_OUTPUT_NAME) or None,`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4870` `os.getenv(ENV_EMBEDDINGS_ONNX_BATCH_SIZE),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4873` `embeddings_onnx_cpu_mem_arena=os.getenv(`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4879` `os.getenv(ENV_EMBEDDINGS_ONNX_DEVICE),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4885` `os.getenv(ENV_EMBEDDINGS_ONNX_CUDA_DEVICE_ID),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4888` `embeddings_tei_url=os.getenv(ENV_EMBEDDINGS_TEI_URL),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4891` `embeddings_openai_api_key=(os.getenv(ENV_EMBEDDINGS_OPENAI_API_KEY) or os.getenv(ENV_LLM_API_KEY)),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4892` `embeddings_openai_model=os.getenv(ENV_EMBEDDINGS_OPENAI_MODEL, DEFAULT_EMBEDDINGS_OPENAI_MODEL),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4893` `embeddings_openai_base_url=os.getenv(ENV_EMBEDDINGS_OPENAI_BASE_URL) or None,`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4896` `os.getenv(ENV_EMBEDDINGS_OPENAI_BATCH_SIZE),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4901` `os.getenv(ENV_EMBEDDINGS_TEI_BATCH_SIZE),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4906` `os.getenv(ENV_EMBEDDINGS_MAX_CONCURRENT_REQUESTS),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4911` `os.getenv(ENV_EMBEDDINGS_OPENAI_DIMENSIONS),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4913` `embeddings_openai_extra_body=json.loads(os.getenv(ENV_EMBEDDINGS_OPENAI_EXTRA_BODY, "null")),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4914` `embeddings_query_prefix=os.getenv(ENV_EMBEDDINGS_QUERY_PREFIX, DEFAULT_EMBEDDINGS_QUERY_PREFIX),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4915` `embeddings_passage_prefix=os.getenv(ENV_EMBEDDINGS_PASSAGE_PREFIX, DEFAULT_EMBEDDINGS_PASSAGE_PREFIX),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4918` `os.getenv(ENV_EMBEDDINGS_MAX_RETRIES),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4923` `os.getenv(ENV_EMBEDDINGS_INITIAL_BACKOFF),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4928` `os.getenv(ENV_EMBEDDINGS_MAX_BACKOFF),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4933` `os.getenv(ENV_EMBEDDINGS_RETRY_BUDGET),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4938` `os.getenv(ENV_RERANKER_MAX_RETRIES),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4943` `os.getenv(ENV_RERANKER_INITIAL_BACKOFF),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4948` `os.getenv(ENV_RERANKER_MAX_BACKOFF),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4953` `os.getenv(ENV_RERANKER_RETRY_BUDGET),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4957` `embeddings_cohere_api_key=os.getenv(ENV_EMBEDDINGS_COHERE_API_KEY) or os.getenv(ENV_COHERE_API_KEY),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4958` `embeddings_cohere_model=os.getenv(ENV_EMBEDDINGS_COHERE_MODEL, DEFAULT_EMBEDDINGS_COHERE_MODEL),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4959` `embeddings_cohere_base_url=os.getenv(ENV_EMBEDDINGS_COHERE_BASE_URL) or None,`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4961` `if (v := os.getenv(ENV_EMBEDDINGS_COHERE_OUTPUT_DIMENSIONS))`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4964` `embeddings_openrouter_api_key=os.getenv(ENV_EMBEDDINGS_OPENROUTER_API_KEY)`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4965` `or os.getenv(ENV_OPENROUTER_API_KEY)`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4966` `or os.getenv(ENV_LLM_API_KEY),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4967` `embeddings_openrouter_model=os.getenv(ENV_EMBEDDINGS_OPENROUTER_MODEL, DEFAULT_EMBEDDINGS_OPENROUTER_MODEL),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4969` `embeddings_requesty_api_key=os.getenv(ENV_EMBEDDINGS_REQUESTY_API_KEY)`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4970` `or os.getenv(ENV_REQUESTY_API_KEY)`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4971` `or os.getenv(ENV_LLM_API_KEY),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4972` `embeddings_requesty_model=os.getenv(ENV_EMBEDDINGS_REQUESTY_MODEL, DEFAULT_EMBEDDINGS_REQUESTY_MODEL),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4974` `embeddings_zeroentropy_api_key=os.getenv(ENV_EMBEDDINGS_ZEROENTROPY_API_KEY)`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4975` `or os.getenv("ZEROENTROPY_API_KEY"),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4976` `embeddings_zeroentropy_model=os.getenv(`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4979` `embeddings_zeroentropy_base_url=os.getenv(`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4984` `os.getenv(ENV_EMBEDDINGS_ZEROENTROPY_DIMENSIONS),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4989` `os.getenv(ENV_EMBEDDINGS_ZEROENTROPY_ENCODING_FORMAT),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:4995` `os.getenv(ENV_EMBEDDINGS_ZEROENTROPY_LATENCY),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5000` `os.getenv(ENV_EMBEDDINGS_ZEROENTROPY_BATCH_SIZE),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5004` `embeddings_nvidia_api_key=os.getenv(ENV_EMBEDDINGS_NVIDIA_API_KEY),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5005` `embeddings_nvidia_model=os.getenv(ENV_EMBEDDINGS_NVIDIA_MODEL, DEFAULT_EMBEDDINGS_NVIDIA_MODEL),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5006` `embeddings_nvidia_base_url=os.getenv(ENV_EMBEDDINGS_NVIDIA_BASE_URL, DEFAULT_EMBEDDINGS_NVIDIA_BASE_URL),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5009` `os.getenv(ENV_EMBEDDINGS_NVIDIA_DIMENSIONS),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5014` `os.getenv(ENV_EMBEDDINGS_NVIDIA_BATCH_SIZE),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5018` `embeddings_litellm_api_base=os.getenv(ENV_EMBEDDINGS_LITELLM_API_BASE)`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5019` `or os.getenv(ENV_LITELLM_API_BASE, DEFAULT_LITELLM_API_BASE),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5020` `embeddings_litellm_api_key=os.getenv(ENV_EMBEDDINGS_LITELLM_API_KEY) or os.getenv(ENV_LITELLM_API_KEY),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5021` `embeddings_litellm_model=os.getenv(ENV_EMBEDDINGS_LITELLM_MODEL, DEFAULT_EMBEDDINGS_LITELLM_MODEL),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5024` `os.getenv(ENV_EMBEDDINGS_LITELLM_DIMENSIONS),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5027` `embeddings_litellm_sdk_api_key=os.getenv(ENV_EMBEDDINGS_LITELLM_SDK_API_KEY),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5028` `embeddings_litellm_sdk_model=os.getenv(`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5031` `embeddings_litellm_sdk_model_id=os.getenv(ENV_EMBEDDINGS_LITELLM_SDK_MODEL_ID) or None,`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5032` `embeddings_litellm_sdk_api_base=os.getenv(ENV_EMBEDDINGS_LITELLM_SDK_API_BASE) or None,`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5034` `if (v := os.getenv(ENV_EMBEDDINGS_LITELLM_SDK_OUTPUT_DIMENSIONS))`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5036` `embeddings_litellm_sdk_encoding_format=os.getenv(`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5040` `embeddings_gemini_api_key=os.getenv(ENV_EMBEDDINGS_GEMINI_API_KEY) or os.getenv(ENV_LLM_API_KEY),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5041` `embeddings_gemini_model=os.getenv(ENV_EMBEDDINGS_GEMINI_MODEL, DEFAULT_EMBEDDINGS_GEMINI_MODEL),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5043` `os.getenv(`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5048` `embeddings_gemini_force_ipv4=os.getenv(`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5055` `os.getenv(ENV_EMBEDDINGS_GEMINI_BATCH_SIZE),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5058` `embeddings_vertexai_project_id=os.getenv(ENV_EMBEDDINGS_VERTEXAI_PROJECT_ID)`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5059` `or os.getenv(ENV_LLM_VERTEXAI_PROJECT_ID),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5060` `embeddings_vertexai_region=os.getenv(ENV_EMBEDDINGS_VERTEXAI_REGION) or os.getenv(ENV_LLM_VERTEXAI_REGION),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5061` `embeddings_vertexai_service_account_key=os.getenv(ENV_EMBEDDINGS_VERTEXAI_SERVICE_ACCOUNT_KEY)`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5062` `or os.getenv(ENV_LLM_VERTEXAI_SERVICE_ACCOUNT_KEY),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5064` `reranker_provider=os.getenv(ENV_RERANKER_PROVIDER, DEFAULT_RERANKER_PROVIDER),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5068` `v := os.getenv(ENV_RERANKER_MAX_TOKENS_PER_CANDIDATE)`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5069` `or os.getenv(ENV_RERANKER_LITELLM_MAX_TOKENS_PER_DOC)`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5072` `reranker_send_bank_as_header=os.getenv(`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5077` `reranker_local_model=os.getenv(ENV_RERANKER_LOCAL_MODEL, DEFAULT_RERANKER_LOCAL_MODEL),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5078` `reranker_local_force_cpu=os.getenv(`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5083` `os.getenv(ENV_RERANKER_LOCAL_MAX_CONCURRENT, str(DEFAULT_RERANKER_LOCAL_MAX_CONCURRENT))`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5085` `reranker_local_trust_remote_code=os.getenv(`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5089` `reranker_local_fp16=os.getenv(ENV_RERANKER_LOCAL_FP16, str(DEFAULT_RERANKER_LOCAL_FP16)).lower()`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5091` `reranker_local_bucket_batching=os.getenv(`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5096` `os.getenv(ENV_RERANKER_LOCAL_BATCH_SIZE, str(DEFAULT_RERANKER_LOCAL_BATCH_SIZE))`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5098` `reranker_local_timeout=float(os.getenv(ENV_RERANKER_LOCAL_TIMEOUT, str(DEFAULT_RERANKER_LOCAL_TIMEOUT))),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5099` `reranker_tei_url=os.getenv(ENV_RERANKER_TEI_URL),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5100` `reranker_tei_batch_size=int(os.getenv(ENV_RERANKER_TEI_BATCH_SIZE, str(DEFAULT_RERANKER_TEI_BATCH_SIZE))),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5102` `os.getenv(ENV_RERANKER_TEI_MAX_CONCURRENT, str(DEFAULT_RERANKER_TEI_MAX_CONCURRENT))`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5105` `os.getenv(ENV_RERANKER_TEI_HTTP_TIMEOUT, str(DEFAULT_RERANKER_TEI_HTTP_TIMEOUT))`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5107` `reranker_max_candidates=int(os.getenv(ENV_RERANKER_MAX_CANDIDATES, str(DEFAULT_RERANKER_MAX_CANDIDATES))),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5109` `os.getenv(ENV_RERANKER_MAX_CANDIDATES_LOW, str(DEFAULT_RERANKER_MAX_CANDIDATES_LOW))`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5112` `os.getenv(ENV_RERANKER_MAX_CANDIDATES_MID, str(DEFAULT_RERANKER_MAX_CANDIDATES_MID))`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5115` `os.getenv(ENV_RERANKER_MAX_CANDIDATES_HIGH, str(DEFAULT_RERANKER_MAX_CANDIDATES_HIGH))`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5117` `semantic_min_similarity=float(os.getenv(ENV_SEMANTIC_MIN_SIMILARITY, str(DEFAULT_SEMANTIC_MIN_SIMILARITY))),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5119` `os.getenv(ENV_GRAPH_SEED_MIN_SIMILARITY, str(DEFAULT_GRAPH_SEED_MIN_SIMILARITY))`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5122` `os.getenv(ENV_TEMPORAL_SEMANTIC_MIN_SIMILARITY, str(DEFAULT_TEMPORAL_SEMANTIC_MIN_SIMILARITY))`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5125` `os.getenv(ENV_SEMANTIC_LINK_MIN_SIMILARITY, str(DEFAULT_SEMANTIC_LINK_MIN_SIMILARITY))`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5127` `bm25_min_score=float(os.getenv(ENV_BM25_MIN_SCORE, str(DEFAULT_BM25_MIN_SCORE))),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5130` `os.getenv(ENV_BM25_MAX_QUERY_TERMS),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5135` `os.getenv(ENV_RECALL_MAX_CANDIDATES_PER_SOURCE, str(DEFAULT_RECALL_MAX_CANDIDATES_PER_SOURCE))`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5138` `os.getenv(ENV_RECALL_STRATEGY_BOOSTS, DEFAULT_RECALL_STRATEGY_BOOSTS)`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5141` `os.getenv(ENV_RECENCY_DECAY_FUNCTION, DEFAULT_RECENCY_DECAY_FUNCTION)`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5144` `os.getenv(ENV_RECENCY_DECAY_LINEAR_WINDOW_DAYS, str(DEFAULT_RECENCY_DECAY_LINEAR_WINDOW_DAYS))`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5147` `os.getenv(ENV_RECENCY_DECAY_HALFLIFE_DAYS, str(DEFAULT_RECENCY_DECAY_HALFLIFE_DAYS))`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5150` `reranker_cohere_api_key=os.getenv(ENV_RERANKER_COHERE_API_KEY) or os.getenv(ENV_COHERE_API_KEY),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5151` `reranker_cohere_model=os.getenv(ENV_RERANKER_COHERE_MODEL, DEFAULT_RERANKER_COHERE_MODEL),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5152` `reranker_cohere_base_url=os.getenv(ENV_RERANKER_COHERE_BASE_URL) or None,`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5153` `reranker_cohere_timeout=float(os.getenv(ENV_RERANKER_COHERE_TIMEOUT, str(DEFAULT_RERANKER_COHERE_TIMEOUT))),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5155` `os.getenv(ENV_RERANKER_COHERE_MAX_CONCURRENT, str(DEFAULT_RERANKER_COHERE_MAX_CONCURRENT))`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5158` `reranker_openrouter_api_key=os.getenv(ENV_RERANKER_OPENROUTER_API_KEY)`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5159` `or os.getenv(ENV_OPENROUTER_API_KEY)`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5160` `or os.getenv(ENV_LLM_API_KEY),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5161` `reranker_openrouter_model=os.getenv(ENV_RERANKER_OPENROUTER_MODEL, DEFAULT_RERANKER_OPENROUTER_MODEL),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5162` `reranker_openrouter_base_url=os.getenv(`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5166` `os.getenv(ENV_RERANKER_OPENROUTER_TIMEOUT, str(DEFAULT_RERANKER_OPENROUTER_TIMEOUT))`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5169` `reranker_litellm_api_base=os.getenv(ENV_RERANKER_LITELLM_API_BASE)`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5170` `or os.getenv(ENV_LITELLM_API_BASE, DEFAULT_LITELLM_API_BASE),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5171` `reranker_litellm_api_key=os.getenv(ENV_RERANKER_LITELLM_API_KEY) or os.getenv(ENV_LITELLM_API_KEY),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5172` `reranker_litellm_model=os.getenv(ENV_RERANKER_LITELLM_MODEL, DEFAULT_RERANKER_LITELLM_MODEL),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5174` `os.getenv(ENV_RERANKER_LITELLM_TIMEOUT, str(DEFAULT_RERANKER_LITELLM_TIMEOUT))`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5177` `reranker_litellm_sdk_api_key=os.getenv(ENV_RERANKER_LITELLM_SDK_API_KEY),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5178` `reranker_litellm_sdk_model=os.getenv(ENV_RERANKER_LITELLM_SDK_MODEL, DEFAULT_RERANKER_LITELLM_SDK_MODEL),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5179` `reranker_litellm_sdk_api_base=os.getenv(ENV_RERANKER_LITELLM_SDK_API_BASE) or None,`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5181` `os.getenv(ENV_RERANKER_LITELLM_SDK_TIMEOUT, str(DEFAULT_RERANKER_LITELLM_SDK_TIMEOUT))`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5184` `reranker_zeroentropy_api_key=os.getenv(ENV_RERANKER_ZEROENTROPY_API_KEY),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5185` `reranker_zeroentropy_model=os.getenv(ENV_RERANKER_ZEROENTROPY_MODEL, DEFAULT_RERANKER_ZEROENTROPY_MODEL),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5186` `reranker_zeroentropy_base_url=os.getenv(ENV_RERANKER_ZEROENTROPY_BASE_URL) or None,`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5188` `os.getenv(ENV_RERANKER_ZEROENTROPY_TIMEOUT, str(DEFAULT_RERANKER_ZEROENTROPY_TIMEOUT))`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5191` `reranker_siliconflow_api_key=os.getenv(ENV_RERANKER_SILICONFLOW_API_KEY),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5192` `reranker_siliconflow_model=os.getenv(ENV_RERANKER_SILICONFLOW_MODEL, DEFAULT_RERANKER_SILICONFLOW_MODEL),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5193` `reranker_siliconflow_base_url=os.getenv(`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5197` `os.getenv(ENV_RERANKER_SILICONFLOW_TIMEOUT, str(DEFAULT_RERANKER_SILICONFLOW_TIMEOUT))`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5200` `reranker_nvidia_api_key=os.getenv(ENV_RERANKER_NVIDIA_API_KEY),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5201` `reranker_nvidia_model=os.getenv(ENV_RERANKER_NVIDIA_MODEL, DEFAULT_RERANKER_NVIDIA_MODEL),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5202` `reranker_nvidia_base_url=os.getenv(ENV_RERANKER_NVIDIA_BASE_URL, DEFAULT_RERANKER_NVIDIA_BASE_URL),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5203` `reranker_nvidia_timeout=float(os.getenv(ENV_RERANKER_NVIDIA_TIMEOUT, str(DEFAULT_RERANKER_NVIDIA_TIMEOUT))),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5206` `os.getenv(ENV_RERANKER_NVIDIA_BATCH_SIZE),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5211` `os.getenv(ENV_RERANKER_NVIDIA_MAX_CONCURRENT),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5215` `reranker_typesafe_api_key=os.getenv(ENV_RERANKER_TYPESAFE_API_KEY),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5216` `reranker_hindsight_api_key=os.getenv(ENV_RERANKER_HINDSIGHT_API_KEY) or None,`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5217` `embeddings_hindsight_api_key=os.getenv(ENV_EMBEDDINGS_HINDSIGHT_API_KEY) or None,`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5218` `reranker_typesafe_model=os.getenv(ENV_RERANKER_TYPESAFE_MODEL, DEFAULT_RERANKER_TYPESAFE_MODEL),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5219` `reranker_typesafe_base_url=os.getenv(ENV_RERANKER_TYPESAFE_BASE_URL, DEFAULT_RERANKER_TYPESAFE_BASE_URL),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5221` `os.getenv(ENV_RERANKER_TYPESAFE_TIMEOUT, str(DEFAULT_RERANKER_TYPESAFE_TIMEOUT))`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5224` `os.getenv(ENV_RERANKER_TYPESAFE_MAX_CONCURRENT, "").strip() or DEFAULT_RERANKER_TYPESAFE_MAX_CONCURRENT`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5230` `reranker_alibaba_api_key=os.getenv(ENV_RERANKER_ALIBABA_API_KEY),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5231` `reranker_alibaba_model=os.getenv(ENV_RERANKER_ALIBABA_MODEL, DEFAULT_RERANKER_ALIBABA_MODEL),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5233` `os.getenv(ENV_RERANKER_ALIBABA_TIMEOUT, str(DEFAULT_RERANKER_ALIBABA_TIMEOUT))`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5236` `reranker_google_model=os.getenv(ENV_RERANKER_GOOGLE_MODEL, DEFAULT_RERANKER_GOOGLE_MODEL),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5237` `reranker_google_project_id=os.getenv(ENV_RERANKER_GOOGLE_PROJECT_ID)`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5238` `or os.getenv(ENV_LLM_VERTEXAI_PROJECT_ID),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5239` `reranker_google_service_account_key=os.getenv(ENV_RERANKER_GOOGLE_SERVICE_ACCOUNT_KEY)`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5240` `or os.getenv(ENV_LLM_VERTEXAI_SERVICE_ACCOUNT_KEY),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5241` `reranker_google_timeout=float(os.getenv(ENV_RERANKER_GOOGLE_TIMEOUT, str(DEFAULT_RERANKER_GOOGLE_TIMEOUT))),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5244` `decision_model_provider=os.getenv(ENV_DECISION_MODEL_PROVIDER) or DEFAULT_DECISION_MODEL_PROVIDER,`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5245` `decision_model_api_key=os.getenv(ENV_DECISION_MODEL_API_KEY) or None,`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5246` `decision_model_model=os.getenv(ENV_DECISION_MODEL_MODEL) or None,`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5247` `decision_model_base_url=os.getenv(ENV_DECISION_MODEL_BASE_URL) or None,`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5248` `decision_model_timeout=float(os.getenv(ENV_DECISION_MODEL_TIMEOUT) or DEFAULT_DECISION_MODEL_TIMEOUT),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5250` `os.getenv(ENV_DECISION_MODEL_MAX_CONCURRENT) or DEFAULT_DECISION_MODEL_MAX_CONCURRENT`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5254` `if (max_request_tokens := os.getenv(ENV_DECISION_MODEL_MAX_REQUEST_TOKENS))`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5257` `decision_model_tokenizer=os.getenv(ENV_DECISION_MODEL_TOKENIZER) or None,`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5259` `f"{operation}_decision_model_{setting.lower()}": os.getenv(_decision_model_env(operation, setting))`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5271` `host=os.getenv(ENV_HOST) or None,`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5272` `port=int(os.getenv(ENV_PORT, DEFAULT_PORT)),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5273` `base_path=os.getenv(ENV_BASE_PATH, DEFAULT_BASE_PATH),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5274` `log_level=os.getenv(ENV_LOG_LEVEL, DEFAULT_LOG_LEVEL),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5275` `workers=int(os.getenv(ENV_WORKERS, str(DEFAULT_WORKERS))),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5278` `access_log=os.getenv(ENV_ACCESS_LOG, "").lower() in ("1", "true", "yes", "on") or DEFAULT_ACCESS_LOG,`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5279` `daemon_log=os.getenv(ENV_DAEMON_LOG) or None,`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5280` `profile=os.getenv(ENV_PROFILE, "").strip() or None,`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5281` `mcp_auth_token=os.getenv(ENV_MCP_AUTH_TOKEN) or None,`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5282` `file_storage_extension=os.getenv(ENV_FILE_STORAGE_EXTENSION) or None,`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5284` `os.getenv(ENV_BACKPRESSURE_DEFER_SECONDS, str(DEFAULT_BACKPRESSURE_DEFER_SECONDS))`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5286` `xai_oauth_token_path=os.getenv(ENV_XAI_OAUTH_TOKEN_PATH, "").strip() or None,`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5287` `xai_oauth_client_id=os.getenv(ENV_XAI_OAUTH_CLIENT_ID, "").strip() or DEFAULT_XAI_OAUTH_CLIENT_ID,`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5288` `xai_oauth_scope=os.getenv(ENV_XAI_OAUTH_SCOPE, "").strip() or DEFAULT_XAI_OAUTH_SCOPE,`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5295` `xai_oauth_base_url=(os.getenv(ENV_XAI_OAUTH_BASE_URL, "").strip() or None),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5296` `xai_oauth_debug_headers=os.getenv(ENV_XAI_OAUTH_DEBUG_HEADERS, str(DEFAULT_XAI_OAUTH_DEBUG_HEADERS)).lower()`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5298` `log_format=os.getenv(ENV_LOG_FORMAT, DEFAULT_LOG_FORMAT).lower(),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5299` `log_json_fields=_parse_str_list(os.getenv(ENV_LOG_JSON_FIELDS, "")) or None,`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5300` `mcp_enabled=os.getenv(ENV_MCP_ENABLED, str(DEFAULT_MCP_ENABLED)).lower() == "true",`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5301` `mcp_enabled_tools=[t.strip() for t in os.getenv(ENV_MCP_ENABLED_TOOLS).split(",") if t.strip()]`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5302` `if os.getenv(ENV_MCP_ENABLED_TOOLS)`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5304` `mcp_stateless=os.getenv(ENV_MCP_STATELESS, str(DEFAULT_MCP_STATELESS)).lower() == "true",`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5305` `mcp_instructions=os.getenv(ENV_MCP_INSTRUCTIONS) or DEFAULT_MCP_INSTRUCTIONS,`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5306` `enable_bank_llm_health=os.getenv(ENV_ENABLE_BANK_LLM_HEALTH, str(DEFAULT_ENABLE_BANK_LLM_HEALTH)).lower()`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5308` `enable_bank_config_api=os.getenv(ENV_ENABLE_BANK_CONFIG_API, str(DEFAULT_ENABLE_BANK_CONFIG_API)).lower()`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5310` `enable_dry_run_extract=os.getenv(ENV_ENABLE_DRY_RUN_EXTRACT, str(DEFAULT_ENABLE_DRY_RUN_EXTRACT)).lower()`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5312` `default_bank_template=_parse_default_bank_template(os.getenv(ENV_DEFAULT_BANK_TEMPLATE)),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5314` `tokenizer_encoding=os.getenv(ENV_TOKENIZER_ENCODING, DEFAULT_TOKENIZER_ENCODING),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5316` `graph_retriever=os.getenv(ENV_GRAPH_RETRIEVER, DEFAULT_GRAPH_RETRIEVER),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5317` `recall_max_concurrent=int(os.getenv(ENV_RECALL_MAX_CONCURRENT, str(DEFAULT_RECALL_MAX_CONCURRENT))),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5319` `int(os.getenv(ENV_ADMISSION_RECALL_MAX_IN_FLIGHT, str(DEFAULT_ADMISSION_RECALL_MAX_IN_FLIGHT))),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5321` `int(os.getenv(ENV_WORKERS, "1")),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5324` `os.getenv(ENV_ADMISSION_RECALL_MAX_WAIT_MS, str(DEFAULT_ADMISSION_RECALL_MAX_WAIT_MS))`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5328` `int(os.getenv(ENV_ADMISSION_REFLECT_MAX_IN_FLIGHT, str(DEFAULT_ADMISSION_REFLECT_MAX_IN_FLIGHT))),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5330` `int(os.getenv(ENV_WORKERS, "1")),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5333` `os.getenv(ENV_ADMISSION_REFLECT_MAX_WAIT_MS, str(DEFAULT_ADMISSION_REFLECT_MAX_WAIT_MS))`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5337` `int(os.getenv(ENV_ADMISSION_RETAIN_MAX_IN_FLIGHT, str(DEFAULT_ADMISSION_RETAIN_MAX_IN_FLIGHT))),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5339` `int(os.getenv(ENV_WORKERS, "1")),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5342` `os.getenv(ENV_ADMISSION_RETAIN_MAX_WAIT_MS, str(DEFAULT_ADMISSION_RETAIN_MAX_WAIT_MS))`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5346` `os.getenv(ENV_RECALL_CONNECTION_BUDGET, str(DEFAULT_RECALL_CONNECTION_BUDGET))`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5348` `recall_max_query_tokens=int(os.getenv(ENV_RECALL_MAX_QUERY_TOKENS, str(DEFAULT_RECALL_MAX_QUERY_TOKENS))),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5349` `recall_diagnostic_phases=os.getenv(`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5353` `recall_trace_reranker_provider=os.getenv(`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5358` `1, int(os.getenv(ENV_RECALL_PHASE_SAMPLE_EVERY, str(DEFAULT_RECALL_PHASE_SAMPLE_EVERY)))`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5360` `gzip_min_size=int(os.getenv(ENV_GZIP_MIN_SIZE, str(DEFAULT_GZIP_MIN_SIZE))),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5361` `loop_lag_report_seconds=float(os.getenv(ENV_LOOP_LAG_REPORT_SECONDS, str(DEFAULT_LOOP_LAG_REPORT_SECONDS))),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5365` `os.getenv(ENV_LINK_EXPANSION_PER_ENTITY_LIMIT, str(DEFAULT_LINK_EXPANSION_PER_ENTITY_LIMIT))`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5367` `link_expansion_timeout=float(os.getenv(ENV_LINK_EXPANSION_TIMEOUT, str(DEFAULT_LINK_EXPANSION_TIMEOUT))),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5368` `retain_batch_document_writes=os.getenv(`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5373` `os.getenv(ENV_BANK_INFO_CACHE_TTL_SECONDS, str(DEFAULT_BANK_INFO_CACHE_TTL_SECONDS))`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5376` `os.getenv(ENV_BANK_INFO_CACHE_MAX_ENTRIES, str(DEFAULT_BANK_INFO_CACHE_MAX_ENTRIES))`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5379` `os.getenv(ENV_BANK_ALIAS_CACHE_TTL_SECONDS, str(DEFAULT_BANK_ALIAS_CACHE_TTL_SECONDS))`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5382` `os.getenv(ENV_BANK_ALIAS_CACHE_MAX_ENTRIES, str(DEFAULT_BANK_ALIAS_CACHE_MAX_ENTRIES))`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5385` `os.getenv(ENV_BANK_STATS_CACHE_TTL_SECONDS, str(DEFAULT_BANK_STATS_CACHE_TTL_SECONDS))`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5388` `os.getenv(ENV_BANK_STATS_CACHE_MAX_ENTRIES, str(DEFAULT_BANK_STATS_CACHE_MAX_ENTRIES))`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5391` `skip_llm_verification=os.getenv(ENV_SKIP_LLM_VERIFICATION, "false").lower() == "true",`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5394` `os.getenv(ENV_RETAIN_MAX_COMPLETION_TOKENS, str(DEFAULT_RETAIN_MAX_COMPLETION_TOKENS))`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5396` `retain_chunk_size=int(os.getenv(ENV_RETAIN_CHUNK_SIZE, str(DEFAULT_RETAIN_CHUNK_SIZE))),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5399` `os.getenv(ENV_RETAIN_STRUCTURED_CHUNK_SIZE),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5401` `retain_extract_causal_links=os.getenv(`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5410` `os.getenv(ENV_RETAIN_EXTRACTION_MODE, DEFAULT_RETAIN_EXTRACTION_MODE)`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5412` `retain_mission=os.getenv(ENV_RETAIN_MISSION) or DEFAULT_RETAIN_MISSION,`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5413` `retain_custom_instructions=os.getenv(ENV_RETAIN_CUSTOM_INSTRUCTIONS) or DEFAULT_RETAIN_CUSTOM_INSTRUCTIONS,`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5414` `retain_default_strategy=os.getenv(ENV_RETAIN_DEFAULT_STRATEGY) or DEFAULT_RETAIN_DEFAULT_STRATEGY,`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5416` `retain_batch_tokens=int(os.getenv(ENV_RETAIN_BATCH_TOKENS, str(DEFAULT_RETAIN_BATCH_TOKENS))),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5417` `retain_entity_lookup=os.getenv(ENV_RETAIN_ENTITY_LOOKUP, DEFAULT_RETAIN_ENTITY_LOOKUP),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5420` `os.getenv(ENV_RETAIN_ENTITY_RESOLUTION_BATCH_SIZE),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5425` `os.getenv(ENV_RETAIN_ENTITY_RESOLUTION_MAX_CANDIDATES),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5428` `retain_batch_enabled=os.getenv(ENV_RETAIN_BATCH_ENABLED, str(DEFAULT_RETAIN_BATCH_ENABLED)).lower()`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5431` `os.getenv(ENV_RETAIN_BATCH_POLL_INTERVAL_SECONDS, str(DEFAULT_RETAIN_BATCH_POLL_INTERVAL_SECONDS))`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5433` `retain_chunk_batch_size=int(os.getenv(ENV_RETAIN_CHUNK_BATCH_SIZE, str(DEFAULT_RETAIN_CHUNK_BATCH_SIZE))),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5434` `retain_memory_budget_mb=int(os.getenv(ENV_RETAIN_MEMORY_BUDGET_MB, str(DEFAULT_RETAIN_MEMORY_BUDGET_MB))),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5436` `os.getenv(ENV_RETAIN_ATTACHMENT_MAX_SIZE_MB, str(DEFAULT_RETAIN_ATTACHMENT_MAX_SIZE_MB))`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5439` `os.getenv(ENV_RETAIN_ATTACHMENT_MAX_COUNT, str(DEFAULT_RETAIN_ATTACHMENT_MAX_COUNT))`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5442` `os.getenv(ENV_RETAIN_MAX_ATTACHMENTS_PER_CHUNK, str(DEFAULT_RETAIN_MAX_ATTACHMENTS_PER_CHUNK))`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5445` `file_storage_type=os.getenv(ENV_FILE_STORAGE_TYPE, DEFAULT_FILE_STORAGE_TYPE),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5446` `file_storage_s3_bucket=os.getenv(ENV_FILE_STORAGE_S3_BUCKET) or None,`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5447` `file_storage_s3_region=os.getenv(ENV_FILE_STORAGE_S3_REGION) or None,`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5448` `file_storage_s3_endpoint=os.getenv(ENV_FILE_STORAGE_S3_ENDPOINT) or None,`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5449` `file_storage_s3_access_key_id=os.getenv(ENV_FILE_STORAGE_S3_ACCESS_KEY_ID) or None,`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5450` `file_storage_s3_secret_access_key=os.getenv(ENV_FILE_STORAGE_S3_SECRET_ACCESS_KEY) or None,`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5451` `file_storage_gcs_bucket=os.getenv(ENV_FILE_STORAGE_GCS_BUCKET) or None,`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5452` `file_storage_gcs_service_account_key=os.getenv(ENV_FILE_STORAGE_GCS_SERVICE_ACCOUNT_KEY) or None,`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5453` `file_storage_azure_container=os.getenv(ENV_FILE_STORAGE_AZURE_CONTAINER) or None,`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5454` `file_storage_azure_account_name=os.getenv(ENV_FILE_STORAGE_AZURE_ACCOUNT_NAME) or None,`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5455` `file_storage_azure_account_key=os.getenv(ENV_FILE_STORAGE_AZURE_ACCOUNT_KEY) or None,`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5456` `file_parser=_parse_str_list(os.getenv(ENV_FILE_PARSER, DEFAULT_FILE_PARSER)),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5458` `file_parser_markitdown_ocr_enabled=os.getenv(`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5463` `file_parser_markitdown_ocr_api_key=os.getenv(ENV_FILE_PARSER_MARKITDOWN_OCR_API_KEY) or None,`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5464` `file_parser_markitdown_ocr_base_url=os.getenv(ENV_FILE_PARSER_MARKITDOWN_OCR_BASE_URL) or None,`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5465` `file_parser_markitdown_ocr_model=os.getenv(ENV_FILE_PARSER_MARKITDOWN_OCR_MODEL) or None,`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5466` `file_parser_markitdown_ocr_prompt=os.getenv(`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5471` `os.getenv(ENV_FILE_PARSER_MARKITDOWN_OCR_DEFAULT_HEADERS, "null")`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5473` `file_parser_iris_token=os.getenv(ENV_FILE_PARSER_IRIS_TOKEN) or None,`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5474` `file_parser_iris_org_id=os.getenv(ENV_FILE_PARSER_IRIS_ORG_ID) or None,`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5475` `file_parser_llama_parse_api_key=os.getenv(ENV_FILE_PARSER_LLAMA_PARSE_API_KEY) or None,`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5477` `os.getenv(ENV_FILE_CONVERSION_MAX_BATCH_SIZE_MB, str(DEFAULT_FILE_CONVERSION_MAX_BATCH_SIZE_MB))`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5480` `os.getenv(ENV_FILE_CONVERSION_MAX_BATCH_SIZE, str(DEFAULT_FILE_CONVERSION_MAX_BATCH_SIZE))`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5482` `enable_file_upload_api=os.getenv(ENV_ENABLE_FILE_UPLOAD_API, str(DEFAULT_ENABLE_FILE_UPLOAD_API)).lower()`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5484` `file_delete_after_retain=os.getenv(`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5488` `store_document_text=os.getenv(ENV_STORE_DOCUMENT_TEXT, str(DEFAULT_STORE_DOCUMENT_TEXT)).lower() == "true",`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5489` `default_timezone=_validate_default_timezone(os.getenv(ENV_DEFAULT_TIMEZONE, DEFAULT_DEFAULT_TIMEZONE)),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5490` `enable_document_export_api=os.getenv(`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5494` `enable_document_import_api=os.getenv(`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5499` `enable_observations=os.getenv(ENV_ENABLE_OBSERVATIONS, str(DEFAULT_ENABLE_OBSERVATIONS)).lower() == "true",`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5500` `enable_auto_consolidation=os.getenv(`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5504` `consolidation_use_decision_model=os.getenv(`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5508` `enable_observation_history=os.getenv(`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5513` `os.getenv(`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5518` `enable_mental_model_history=os.getenv(`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5523` `os.getenv(`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5534` `os.getenv(ENV_MENTAL_MODEL_MIN_REFRESH_INTERVAL_SECONDS, "").strip()`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5539` `os.getenv(ENV_KNOWLEDGE_PAGE_DEFAULT_TRIGGER, "").strip() or "null"`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5543` `os.getenv(ENV_CONSOLIDATION_BATCH_SIZE, str(DEFAULT_CONSOLIDATION_BATCH_SIZE))`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5546` `os.getenv(`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5552` `os.getenv(ENV_CONSOLIDATION_DEDUP_THRESHOLD, str(DEFAULT_CONSOLIDATION_DEDUP_THRESHOLD))`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5555` `os.getenv(ENV_CONSOLIDATION_LLM_BATCH_SIZE, str(DEFAULT_CONSOLIDATION_LLM_BATCH_SIZE))`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5560` `os.getenv(`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5567` `os.getenv(ENV_CONSOLIDATION_MAX_TOKENS, str(DEFAULT_CONSOLIDATION_MAX_TOKENS))`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5572` `consolidation_recall_budget=os.getenv(ENV_CONSOLIDATION_RECALL_BUDGET, DEFAULT_CONSOLIDATION_RECALL_BUDGET),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5574` `os.getenv(ENV_CONSOLIDATION_SOURCE_FACTS_MAX_TOKENS, str(DEFAULT_CONSOLIDATION_SOURCE_FACTS_MAX_TOKENS))`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5577` `os.getenv(`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5583` `os.getenv(ENV_CONSOLIDATION_MAX_ATTEMPTS, str(DEFAULT_CONSOLIDATION_MAX_ATTEMPTS))`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5585` `observations_mission=os.getenv(ENV_OBSERVATIONS_MISSION) or DEFAULT_OBSERVATIONS_MISSION,`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5587` `os.getenv(ENV_MAX_OBSERVATIONS_PER_SCOPE, str(DEFAULT_MAX_OBSERVATIONS_PER_SCOPE))`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5589` `observation_scope_limits=json.loads(os.getenv(ENV_OBSERVATION_SCOPE_LIMITS, "null"))`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5591` `default_observation_scopes=_parse_observation_scopes_env(os.getenv(ENV_DEFAULT_OBSERVATION_SCOPES))`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5593` `consolidation_strategies=json.loads(os.getenv(ENV_CONSOLIDATION_STRATEGIES, "null"))`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5599` `run_migrations_on_startup=os.getenv(ENV_RUN_MIGRATIONS_ON_STARTUP, "true").lower() == "true",`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5601` `migration_concurrency=int(os.getenv(ENV_MIGRATION_CONCURRENCY, str(DEFAULT_MIGRATION_CONCURRENCY))),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5603` `db_pool_min_size=int(os.getenv(ENV_DB_POOL_MIN_SIZE, str(DEFAULT_DB_POOL_MIN_SIZE))),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5604` `db_pool_max_size=int(os.getenv(ENV_DB_POOL_MAX_SIZE, str(DEFAULT_DB_POOL_MAX_SIZE))),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5605` `db_command_timeout=int(os.getenv(ENV_DB_COMMAND_TIMEOUT, str(DEFAULT_DB_COMMAND_TIMEOUT))),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5606` `db_acquire_timeout=int(os.getenv(ENV_DB_ACQUIRE_TIMEOUT, str(DEFAULT_DB_ACQUIRE_TIMEOUT))),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5608` `os.getenv(`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5612` `db_statement_timeout=int(os.getenv(ENV_DB_STATEMENT_TIMEOUT, str(DEFAULT_DB_STATEMENT_TIMEOUT))),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5615` `os.getenv(ENV_DB_MAX_PARALLEL_WORKERS_PER_GATHER),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5621` `os.getenv(ENV_ENTITY_TRGM_SIMILARITY_THRESHOLD, str(DEFAULT_ENTITY_TRGM_SIMILARITY_THRESHOLD))`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5624` `os.getenv(ENV_ENTITY_INTRABATCH_MERGE_SIMILARITY, str(DEFAULT_ENTITY_INTRABATCH_MERGE_SIMILARITY))`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5627` `os.getenv(ENV_ENTITY_MERGE_MIN_SIMILARITY, str(DEFAULT_ENTITY_MERGE_MIN_SIMILARITY))`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5629` `model_init_timeout=float(os.getenv(ENV_MODEL_INIT_TIMEOUT, str(DEFAULT_MODEL_INIT_TIMEOUT))),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5631` `worker_enabled=os.getenv(ENV_WORKER_ENABLED, str(DEFAULT_WORKER_ENABLED)).lower() == "true",`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5632` `worker_id=os.getenv(ENV_WORKER_ID) or DEFAULT_WORKER_ID,`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5633` `worker_poll_interval_ms=int(os.getenv(ENV_WORKER_POLL_INTERVAL_MS, str(DEFAULT_WORKER_POLL_INTERVAL_MS))),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5634` `worker_max_retries=int(os.getenv(ENV_WORKER_MAX_RETRIES, str(DEFAULT_WORKER_MAX_RETRIES))),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5636` `os.getenv(`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5641` `worker_http_port=int(os.getenv(ENV_WORKER_HTTP_PORT, str(DEFAULT_WORKER_HTTP_PORT))),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5642` `worker_max_slots=int(os.getenv(ENV_WORKER_MAX_SLOTS, str(DEFAULT_WORKER_MAX_SLOTS))),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5645` `os.getenv(ENV_WORKER_CONSOLIDATION_BANK_PRIORITY, "")`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5649` `os.getenv(ENV_OPERATION_RETENTION_DAYS),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5654` `os.getenv(ENV_OPERATION_CLEANUP_BATCH_SIZE),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5657` `retain_max_concurrent=int(os.getenv(ENV_RETAIN_MAX_CONCURRENT, str(DEFAULT_RETAIN_MAX_CONCURRENT))),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5659` `os.getenv(ENV_RETAIN_SUBBATCH_CONCURRENCY, str(DEFAULT_RETAIN_SUBBATCH_CONCURRENCY))`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5662` `os.getenv(ENV_RETAIN_STORE_MAX_CONCURRENT, str(DEFAULT_RETAIN_STORE_MAX_CONCURRENT))`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5664` `retain_wall_timeout=int(os.getenv(ENV_RETAIN_WALL_TIMEOUT, str(DEFAULT_RETAIN_WALL_TIMEOUT))),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5666` `os.getenv(ENV_CONSOLIDATION_WALL_TIMEOUT, str(DEFAULT_CONSOLIDATION_WALL_TIMEOUT))`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5669` `reflect_max_iterations=int(os.getenv(ENV_REFLECT_MAX_ITERATIONS, str(DEFAULT_REFLECT_MAX_ITERATIONS))),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5670` `reflect_use_decision_model=os.getenv(`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5674` `mental_model_refresh_use_decision_model=os.getenv(`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5678` `mental_model_delta_use_decision_model=os.getenv(`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5684` `os.getenv(ENV_MENTAL_MODEL_DELTA_GATE_MAX_MEMORIES),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5688` `os.getenv(ENV_REFLECT_MAX_CONTEXT_TOKENS, str(DEFAULT_REFLECT_MAX_CONTEXT_TOKENS))`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5690` `reflect_wall_timeout=int(os.getenv(ENV_REFLECT_WALL_TIMEOUT, str(DEFAULT_REFLECT_WALL_TIMEOUT))),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5691` `reflect_mission=os.getenv(ENV_REFLECT_MISSION) or None,`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5693` `os.getenv(ENV_REFLECT_SOURCE_FACTS_MAX_TOKENS, str(DEFAULT_REFLECT_SOURCE_FACTS_MAX_TOKENS))`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5695` `reflect_default_options=json.loads(os.getenv(ENV_REFLECT_DEFAULT_OPTIONS, "").strip() or "null")`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5700` `enable_text_search=os.getenv(ENV_ENABLE_TEXT_SEARCH, str(DEFAULT_ENABLE_TEXT_SEARCH)).lower()`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5702` `enable_temporal_retrieval=os.getenv(`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5706` `enable_graph_retrieval=os.getenv(ENV_ENABLE_GRAPH_RETRIEVAL, str(DEFAULT_ENABLE_GRAPH_RETRIEVAL)).lower()`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5708` `enable_reranking=os.getenv(ENV_ENABLE_RERANKING, str(DEFAULT_ENABLE_RERANKING)).lower()`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5710` `recall_prefer_observations=os.getenv(`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5714` `recall_include_chunks=os.getenv(ENV_RECALL_INCLUDE_CHUNKS, str(DEFAULT_RECALL_INCLUDE_CHUNKS)).lower()`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5716` `recall_max_tokens=int(os.getenv(ENV_RECALL_MAX_TOKENS, str(DEFAULT_RECALL_MAX_TOKENS))),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5718` `os.getenv(ENV_RECALL_CHUNKS_MAX_TOKENS, str(DEFAULT_RECALL_CHUNKS_MAX_TOKENS))`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5721` `os.getenv(ENV_RECALL_BUDGET_FUNCTION, DEFAULT_RECALL_BUDGET_FUNCTION)`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5723` `recall_budget_fixed_low=int(os.getenv(ENV_RECALL_BUDGET_FIXED_LOW, str(DEFAULT_RECALL_BUDGET_FIXED_LOW))),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5724` `recall_budget_fixed_mid=int(os.getenv(ENV_RECALL_BUDGET_FIXED_MID, str(DEFAULT_RECALL_BUDGET_FIXED_MID))),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5726` `os.getenv(ENV_RECALL_BUDGET_FIXED_HIGH, str(DEFAULT_RECALL_BUDGET_FIXED_HIGH))`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5729` `os.getenv(ENV_RECALL_BUDGET_ADAPTIVE_LOW, str(DEFAULT_RECALL_BUDGET_ADAPTIVE_LOW))`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5732` `os.getenv(ENV_RECALL_BUDGET_ADAPTIVE_MID, str(DEFAULT_RECALL_BUDGET_ADAPTIVE_MID))`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5735` `os.getenv(ENV_RECALL_BUDGET_ADAPTIVE_HIGH, str(DEFAULT_RECALL_BUDGET_ADAPTIVE_HIGH))`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5737` `recall_budget_min=int(os.getenv(ENV_RECALL_BUDGET_MIN, str(DEFAULT_RECALL_BUDGET_MIN))),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5738` `recall_budget_max=int(os.getenv(ENV_RECALL_BUDGET_MAX, str(DEFAULT_RECALL_BUDGET_MAX))),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5744` `otel_traces_enabled=os.getenv(ENV_OTEL_TRACES_ENABLED, str(DEFAULT_OTEL_TRACES_ENABLED)).lower()`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5746` `otel_exporter_otlp_endpoint=os.getenv(ENV_OTEL_EXPORTER_OTLP_ENDPOINT) or None,`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5747` `otel_exporter_otlp_headers=os.getenv(ENV_OTEL_EXPORTER_OTLP_HEADERS) or None,`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5748` `otel_service_name=os.getenv(ENV_OTEL_SERVICE_NAME) or None,`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5749` `otel_deployment_environment=os.getenv(ENV_OTEL_DEPLOYMENT_ENVIRONMENT, DEFAULT_OTEL_DEPLOYMENT_ENVIRONMENT),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5750` `metrics_include_bank_id=os.getenv(ENV_METRICS_INCLUDE_BANK_ID, str(DEFAULT_METRICS_INCLUDE_BANK_ID)).lower()`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5752` `metrics_backlog_enabled=os.getenv(ENV_METRICS_BACKLOG_ENABLED, str(DEFAULT_METRICS_BACKLOG_ENABLED)).lower()`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5754` `metrics_include_tenant=os.getenv(ENV_METRICS_INCLUDE_TENANT, str(DEFAULT_METRICS_INCLUDE_TENANT)).lower()`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5757` `loop_watchdog_enabled=os.getenv(ENV_LOOP_WATCHDOG_ENABLED, str(DEFAULT_LOOP_WATCHDOG_ENABLED)).lower()`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5760` `os.getenv(ENV_LOOP_WATCHDOG_STALL_THRESHOLD_MS, str(DEFAULT_LOOP_WATCHDOG_STALL_THRESHOLD_MS))`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5763` `os.getenv(ENV_LOOP_WATCHDOG_POLL_INTERVAL_MS, str(DEFAULT_LOOP_WATCHDOG_POLL_INTERVAL_MS))`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5766` `os.getenv(ENV_DB_ACQUIRE_WARN_THRESHOLD_MS, str(DEFAULT_DB_ACQUIRE_WARN_THRESHOLD_MS))`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5769` `audit_log_enabled=os.getenv(ENV_AUDIT_LOG_ENABLED, str(DEFAULT_AUDIT_LOG_ENABLED)).lower() == "true",`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5771` `a.strip() for a in os.getenv(ENV_AUDIT_LOG_ACTIONS, DEFAULT_AUDIT_LOG_ACTIONS).split(",") if a.strip()`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5774` `os.getenv(ENV_AUDIT_LOG_RETENTION_DAYS, str(DEFAULT_AUDIT_LOG_RETENTION_DAYS))`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5777` `fail_on_extraction_errors=os.getenv(`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5782` `llm_trace_enabled=os.getenv(ENV_LLM_TRACE_ENABLED, str(DEFAULT_LLM_TRACE_ENABLED)).lower() == "true",`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5784` `s.strip() for s in os.getenv(ENV_LLM_TRACE_SCOPES, DEFAULT_LLM_TRACE_SCOPES).split(",") if s.strip()`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5788` `for s in os.getenv(ENV_LLM_TRACE_EXCLUDE_SCOPES, DEFAULT_LLM_TRACE_EXCLUDE_SCOPES).split(",")`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5792` `os.getenv(ENV_LLM_TRACE_RETENTION_DAYS, str(DEFAULT_LLM_TRACE_RETENTION_DAYS))`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5794` `llm_trace_max_chars=int(os.getenv(ENV_LLM_TRACE_MAX_CHARS, str(DEFAULT_LLM_TRACE_MAX_CHARS))),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5795` `llm_trace_delete_with_document=os.getenv(`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5801` `os.getenv(`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5807` `os.getenv(`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5814` `os.getenv(ENV_RETENTION_SWEEP_INTERVAL_SECONDS),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5819` `os.getenv(ENV_VECTOR_INDEX_MIN_ROWS),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5824` `os.getenv(ENV_VECTOR_INDEX_MAINTENANCE_MIN_INTERVAL_SECONDS),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5829` `os.getenv(ENV_OPERATION_CLEANUP_INTERVAL_SECONDS),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5834` `os.getenv(ENV_MAINTENANCE_START_JITTER_SECONDS),`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5838` `webhook_url=os.getenv(ENV_WEBHOOK_URL) or DEFAULT_WEBHOOK_URL,`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5839` `webhook_secret=os.getenv(ENV_WEBHOOK_SECRET) or DEFAULT_WEBHOOK_SECRET,`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5842` `for t in os.getenv(ENV_WEBHOOK_EVENT_TYPES, DEFAULT_WEBHOOK_EVENT_TYPES).split(",")`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5845` `webhook_allowed_hosts=[h.strip() for h in os.getenv(ENV_WEBHOOK_ALLOWED_HOSTS, "").split(",") if h.strip()],`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5853` `h.lower() for h in _parse_str_list(os.getenv(ENV_EXTENSION_PASSTHROUGH_HEADERS, ""))`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5866` `return "https://api.groq.com/openai/v1"`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5868` `return "http://localhost:11434/v1"`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5870` `return "https://ollama.com/v1"`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:5872` `return "http://localhost:1234/v1"`
- `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:6070` `raw = os.getenv(ENV_MIGRATION_ISOLATION, DEFAULT_MIGRATION_ISOLATION).strip().lower()`

- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/storage/__init__.py:155` `for key, value in os.environ.items()`

- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/storage/gcs.py:25` `credentials, _ = google.auth.default(scopes=["https://www.googleapis.com/auth/cloud-platform"])`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/storage/gcs.py:71` `gac = os.environ.pop("GOOGLE_APPLICATION_CREDENTIALS", None)`
- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/storage/gcs.py:76` `os.environ["GOOGLE_APPLICATION_CREDENTIALS"] = gac`

- `sources/hindsight/hindsight-api-slim/hindsight_api/engine/storage/s3.py:34` `if endpoint.startswith("http://"):`

- `sources/hindsight/hindsight-cli/src/config.rs:7` `const DEFAULT_API_URL: &str = "http://localhost:8888";`
- `sources/hindsight/hindsight-cli/src/config.rs:89` `if !api_url.starts_with("http://") && !api_url.starts_with("https://") {`
- `sources/hindsight/hindsight-cli/src/config.rs:91` `"Invalid API URL: {}. Must start with http:// or https://",`
- `sources/hindsight/hindsight-cli/src/config.rs:265` `if !api_url.starts_with("http://") && !api_url.starts_with("https://") {`
- `sources/hindsight/hindsight-cli/src/config.rs:267` `"Invalid API URL: {}. Must start with http:// or https://",`
- `sources/hindsight/hindsight-cli/src/config.rs:289` `let perms = fs::Permissions::from_mode(0o600);`
- `sources/hindsight/hindsight-cli/src/config.rs:437` `save_profile_to_dir(&dir, "prod", "https://api.example.com", Some("hsk_abc")).unwrap();`
- `sources/hindsight/hindsight-cli/src/config.rs:441` `assert_eq!(url, "https://api.example.com");`
- `sources/hindsight/hindsight-cli/src/config.rs:448` `assert_eq!(mode, 0o600);`
- `sources/hindsight/hindsight-cli/src/config.rs:481` `save_profile_to_dir(&dir, "prod", "https://prod.example.com", None).unwrap();`
- `sources/hindsight/hindsight-cli/src/config.rs:482` `save_profile_to_dir(&dir, "dev", "https://dev.example.com", None).unwrap();`
- `sources/hindsight/hindsight-cli/src/config.rs:483` `save_profile_to_dir(&dir, "staging", "https://staging.example.com", None).unwrap();`
- `sources/hindsight/hindsight-cli/src/config.rs:501` `"http://localhost:8888".to_string(),`
- `sources/hindsight/hindsight-cli/src/config.rs:507` `assert_eq!(config.api_url, "http://localhost:8888");`
- `sources/hindsight/hindsight-cli/src/config.rs:514` `"https://api.example.com".to_string(),`
- `sources/hindsight/hindsight-cli/src/config.rs:520` `assert_eq!(config.api_url, "https://api.example.com");`
- `sources/hindsight/hindsight-cli/src/config.rs:532` `assert!(err.contains("Must start with http:// or https://"));`
- `sources/hindsight/hindsight-cli/src/config.rs:565` `parse_config_value(r#"api_url = "http://localhost:8888""#, "api_url"),`
- `sources/hindsight/hindsight-cli/src/config.rs:566` `Some("http://localhost:8888".to_string())`
- `sources/hindsight/hindsight-cli/src/config.rs:573` `parse_config_value("api_url = 'http://localhost:8888'", "api_url"),`
- `sources/hindsight/hindsight-cli/src/config.rs:574` `Some("http://localhost:8888".to_string())`
- `sources/hindsight/hindsight-cli/src/config.rs:581` `parse_config_value("api_url = http://localhost:8888", "api_url"),`
- `sources/hindsight/hindsight-cli/src/config.rs:582` `Some("http://localhost:8888".to_string())`
- `sources/hindsight/hindsight-cli/src/config.rs:589` `parse_config_value("  api_url  =  \"http://localhost:8888\"  ", "api_url"),`
- `sources/hindsight/hindsight-cli/src/config.rs:590` `Some("http://localhost:8888".to_string())`
- `sources/hindsight/hindsight-cli/src/config.rs:623` `api_url: "http://test:8080".to_string(),`
- `sources/hindsight/hindsight-cli/src/config.rs:627` `assert_eq!(config.api_url(), "http://test:8080");`

- `sources/hindsight/hindsight-clients/python/hindsight_client_api/configuration.py:9` `Generated by OpenAPI Generator (https://openapi-generator.tech)`
- `sources/hindsight/hindsight-clients/python/hindsight_client_api/configuration.py:187` `self._base_path = "http://localhost" if host is None else host`

- `sources/hindsight/hindsight-clients/python/hindsight_client_api/models/bank_config_response.py:9` `Generated by OpenAPI Generator (https://openapi-generator.tech)`

- `sources/hindsight/hindsight-clients/python/hindsight_client_api/models/bank_config_update.py:9` `Generated by OpenAPI Generator (https://openapi-generator.tech)`

- `sources/hindsight/hindsight-clients/python/hindsight_client_api/models/bank_template_config.py:9` `Generated by OpenAPI Generator (https://openapi-generator.tech)`

- `sources/hindsight/hindsight-clients/python/hindsight_client_api/models/webhook_http_config.py:9` `Generated by OpenAPI Generator (https://openapi-generator.tech)`

- `sources/hindsight/hindsight-integrations/hermes/__init__.py:113` `Under ''gateway.multiplex_profiles'' ''os.environ'' holds the DEFAULT profile's ''.env'', so a miss`
- `sources/hindsight/hindsight-integrations/hermes/__init__.py:114` `is a miss , never ''os.environ'' (same rule as the daemon's key and base URL in ''embedded.py'').`
- `sources/hindsight/hindsight-integrations/hermes/__init__.py:128` `value = get_secret(name, default)`
- `sources/hindsight/hindsight-integrations/hermes/__init__.py:142` `return config.get("apiKey") or config.get("api_key") or get_secret("HINDSIGHT_API_KEY", "")`
- `sources/hindsight/hindsight-integrations/hermes/__init__.py:419` `for path in (get_hermes_home() / "hindsight" / "config.json", Path.home() / ".hindsight" / "config.json"):`
- `sources/hindsight/hindsight-integrations/hermes/__init__.py:427` `"mode": get_secret("HINDSIGHT_MODE", "") or "cloud",`
- `sources/hindsight/hindsight-integrations/hermes/__init__.py:428` `"apiKey": get_secret("HINDSIGHT_API_KEY", ""),`
- `sources/hindsight/hindsight-integrations/hermes/__init__.py:429` `"timeout": _parse_int_setting(os.environ.get("HINDSIGHT_TIMEOUT"), _DEFAULT_TIMEOUT),`
- `sources/hindsight/hindsight-integrations/hermes/__init__.py:430` `"idle_timeout": _parse_int_setting(os.environ.get("HINDSIGHT_IDLE_TIMEOUT"), _DEFAULT_IDLE_TIMEOUT),`
- `sources/hindsight/hindsight-integrations/hermes/__init__.py:431` `"retain_tags": get_secret("HINDSIGHT_RETAIN_TAGS", "") or "",`
- `sources/hindsight/hindsight-integrations/hermes/__init__.py:432` `"observation_scopes": get_secret("HINDSIGHT_RETAIN_OBSERVATION_SCOPES", "") or "",`
- `sources/hindsight/hindsight-integrations/hermes/__init__.py:443` `"bankId": get_secret("HINDSIGHT_BANK_ID", "") or "hermes",`
- `sources/hindsight/hindsight-integrations/hermes/__init__.py:444` `"budget": os.environ.get("HINDSIGHT_BUDGET", "mid"),`
- `sources/hindsight/hindsight-integrations/hermes/__init__.py:524` `return [str(Path.home() / ".hindsight")]`
- `sources/hindsight/hindsight-integrations/hermes/__init__.py:615` `_cloud_api_key(cfg) or cfg.get("api_url") or get_secret("HINDSIGHT_API_URL", "")`
- `sources/hindsight/hindsight-integrations/hermes/__init__.py:641` `atomic_json_write(config_path, {**read_json_or_empty(config_path), **values}, mode=0o600)`
- `sources/hindsight/hindsight-integrations/hermes/__init__.py:669` `"url": "https://ui.hindsight.vectorize.io",`
- `sources/hindsight/hindsight-integrations/hermes/__init__.py:706` `"description": "Endpoint URL (e.g. http://[ip]:8080/v1)",`
- `sources/hindsight/hindsight-integrations/hermes/__init__.py:891` `return _parse_int_setting(os.environ.get(env_var, env_default) if value is None else value, default)`
- `sources/hindsight/hindsight-integrations/hermes/__init__.py:1291` `if self._agent_context == "primary" and os.environ.get("HERMES_KANBAN_TASK"):`
- `sources/hindsight/hindsight-integrations/hermes/__init__.py:1461` `self._api_url = cfg.get("api_url") or get_secret("HINDSIGHT_API_URL", "") or default_url`
- `sources/hindsight/hindsight-integrations/hermes/__init__.py:1469` `get_secret("HINDSIGHT_BANK_ID", "") or cfg.get("bank_id") or banks.get("bankId") or "hermes"`

- `sources/hindsight/hindsight-integrations/hermes/config_schema.py:44` `default="https://api.hindsight.vectorize.io",`

- `sources/hindsight/hindsight-integrations/hermes/embedded.py:66` `os.environ.setdefault(_PORT_HEALTH_GRACE_ENV, repr(seconds))`
- `sources/hindsight/hindsight-integrations/hermes/embedded.py:136` `"    os.environ.pop(key, None)\n"`
- `sources/hindsight/hindsight-integrations/hermes/embedded.py:169` `for tools in (Path.home() / ".hermes" / "tools", Path(sys.prefix).parent / "tools"):`
- `sources/hindsight/hindsight-integrations/hermes/embedded.py:179` `Workaround for hindsight-embed <= 0.10.2, which copies ''os.environ'' into the daemon process.`
- `sources/hindsight/hindsight-integrations/hermes/embedded.py:186` `Deliberately NOT done by scrubbing ''os.environ'' around ''ensure_running'': that hole would be`
- `sources/hindsight/hindsight-integrations/hermes/embedded.py:194` `env = dict(os.environ)`
- `sources/hindsight/hindsight-integrations/hermes/embedded.py:198` `result = subprocess.run(`
- `sources/hindsight/hindsight-integrations/hermes/embedded.py:206` `except (OSError, subprocess.SubprocessError) as exc:`
- `sources/hindsight/hindsight-integrations/hermes/embedded.py:298` `return Path.home() / ".hindsight" / "profiles" / f"{profile}.env"`
- `sources/hindsight/hindsight-integrations/hermes/embedded.py:327` `scoped = get_secret("HINDSIGHT_API_LLM_API_KEY", "") or get_secret("HINDSIGHT_LLM_API_KEY", "")`
- `sources/hindsight/hindsight-integrations/hermes/embedded.py:396` `base_url = get_secret("HINDSIGHT_API_LLM_BASE_URL", "") or ""`
- `sources/hindsight/hindsight-integrations/hermes/embedded.py:402` `idle_timeout = os.environ.get("HINDSIGHT_IDLE_TIMEOUT")`
- `sources/hindsight/hindsight-integrations/hermes/embedded.py:421` `if url := get_secret(key, ""):`
- `sources/hindsight/hindsight-integrations/hermes/embedded.py:434` `os.chmod(profile_env, 0o600)`
- `sources/hindsight/hindsight-integrations/hermes/embedded.py:435` `fd = os.open(str(profile_env), os.O_WRONLY | os.O_CREAT | os.O_TRUNC, 0o600)`
- `sources/hindsight/hindsight-integrations/hermes/embedded.py:446` `if stat.S_IMODE(profile_env.stat().st_mode) != 0o600:`
- `sources/hindsight/hindsight-integrations/hermes/embedded.py:448` `os.chmod(profile_env, 0o600)`
- `sources/hindsight/hindsight-integrations/hermes/embedded.py:449` `if stat.S_IMODE(profile_env.stat().st_mode) != 0o600:`

- `sources/hindsight/hindsight-integrations/hermes/settings.py:19` `_DEFAULT_API_URL = "https://api.hindsight.vectorize.io"`
- `sources/hindsight/hindsight-integrations/hermes/settings.py:20` `_DEFAULT_LOCAL_URL = "http://localhost:8888"`
- `sources/hindsight/hindsight-integrations/hermes/settings.py:211` `home = Path.home().resolve()`

- `sources/hindsight/hindsight-integrations/hermes/setup.py:73` `prompt = "  LLM endpoint URL (e.g. http://[ip]:8080/v1)" + (`
- `sources/hindsight/hindsight-integrations/hermes/setup.py:79` `provider_config["llm_base_url"] = "https://openrouter.ai/api/v1"`
- `sources/hindsight/hindsight-integrations/hermes/setup.py:144` `print("\n  Get your API key at https://ui.hindsight.vectorize.io\n")`
- `sources/hindsight/hindsight-integrations/hermes/setup.py:145` `existing_key = get_secret("HINDSIGHT_API_KEY", "") or ""`
- `sources/hindsight/hindsight-integrations/hermes/setup.py:185` `api_key=env_writes.get("HINDSIGHT_API_KEY") or os.environ.get("HINDSIGHT_API_KEY", "") or None,`

- `sources/hindsight/hindsight-integrations/hermes/templates.py:21` `"https://raw.githubusercontent.com/vectorize-io/hindsight/main/hindsight-docs/src/data/templates.json"`
- `sources/hindsight/hindsight-integrations/hermes/templates.py:34` `return os.environ.get("HINDSIGHT_TEMPLATES_URL", _DEFAULT_CATALOG_URL)`

</details>

## 5. Reproducibility

`range`: `hindsight-client>=0.10.1,<2`, `hindsight-embed>=0.10.1,<2` (`sources/hindsight/hindsight-integrations/hermes/pyproject.toml:18`). The plugin manifest delegates dependency installation to this pyproject.

Retained lock/checksum files: `sources/hindsight/hindsight-integrations/zed/package-lock.json:1`, `sources/hindsight/hindsight-integrations/eliza/package-lock.json:1`, `sources/hindsight/hindsight-integrations/cloudflare-oauth-proxy/package-lock.json:1`, `sources/hindsight/hindsight-integrations/paperclip/package-lock.json:1`, `sources/hindsight/hindsight-integrations/chat/package-lock.json:1`, `sources/hindsight/hindsight-integrations/openclaw/package-lock.json:1`, `sources/hindsight/hindsight-integrations/zapier/package-lock.json:1`, `sources/hindsight/hindsight-integrations/n8n/package-lock.json:1`, `sources/hindsight/hindsight-integrations/eve/package-lock.json:1`, `sources/hindsight/hindsight-integrations/flowise/package-lock.json:1`, `sources/hindsight/hindsight-integrations/obsidian/package-lock.json:1`, `sources/hindsight/hindsight-integrations/nemoclaw/package-lock.json:1`, `sources/hindsight/hindsight-integrations/ai-sdk/package-lock.json:1`, `sources/hindsight/hindsight-integrations/coding-agents/package-lock.json:1`, `sources/hindsight/hindsight-clients/go/go.sum:1`, `sources/hindsight/hindsight-embed/hindsight_embed/control_center/ui/package-lock.json:1`. Upstream completeness is unknown: fetch.py:15 excludes .lock files, fetch.py:12 excludes build/vendor directories. Absence here is not absence upstream.

- `sources/hindsight/hindsight-integrations/hermes/plugin.yaml:2` `version: 1.2.4`
- `sources/hindsight/hindsight-integrations/hermes/plugin.yaml:4` `# No dependency list here on purpose: Hermes' package manager reads this plugin's own`
- `sources/hindsight/hindsight-integrations/hermes/plugin.yaml:5` `# pyproject.toml (pm/plugin_declarations.py prefers an authored project.dependencies and ignores`
- `sources/hindsight/hindsight-integrations/hermes/plugin.yaml:7` `# manifest deps only until NousResearch/hermes-agent#128408 lands , display, not installation.`

- `sources/hindsight/hindsight-integrations/hermes/pyproject.toml:3` `version = "1.2.4"`
- `sources/hindsight/hindsight-integrations/hermes/pyproject.toml:5` `requires-python = ">=3.11"`
- `sources/hindsight/hindsight-integrations/hermes/pyproject.toml:7` `# Installed into the Hermes venv by 'hermes plugins install' / 'enable' and re-applied after`
- `sources/hindsight/hindsight-integrations/hermes/pyproject.toml:9` `# plugin whose requirements cannot resolve against them.`
- `sources/hindsight/hindsight-integrations/hermes/pyproject.toml:10` `dependencies = [`
- `sources/hindsight/hindsight-integrations/hermes/pyproject.toml:15` `# task"). That breaks local_embedded outright. Anyone who installed between`
- `sources/hindsight/hindsight-integrations/hermes/pyproject.toml:17` `# off it, so keep these at >=0.10.1 rather than widening them back.`
- `sources/hindsight/hindsight-integrations/hermes/pyproject.toml:18` `"hindsight-client>=0.10.1,<2",`
- `sources/hindsight/hindsight-integrations/hermes/pyproject.toml:19` `"hindsight-embed>=0.10.1,<2",`
- `sources/hindsight/hindsight-integrations/hermes/pyproject.toml:22` `# The plugin is loaded from a directory by Hermes, not installed as a distribution.`
- `sources/hindsight/hindsight-integrations/hermes/pyproject.toml:26` `[dependency-groups]`
- `sources/hindsight/hindsight-integrations/hermes/pyproject.toml:27` `dev = ["pytest>=8.0"]`

## 6. Tests

Adapter test files and declaration counts are listed below. There is no adapter forget tool to test. Engine invalidation/document/bank tests are separate; do not use a CLI method name as evidence of a working delete. Archive test asserts the live row and links are removed, the archive row remains, and a derived observation is removed. (`sources/hindsight/hindsight-api-slim/tests/test_memory_curation.py:228`).

Adapter-related declarations: **166**. Python AST test_ functions, fixtures excluded; literal TS/JS it/test, Go Test*, Rust test_* declarations. No parametrization expansion, collection, execution, or coverage measurement. Other rows are engine test candidates selected by mutation names; names alone do not verify forget coverage.

| file | declarations | adapter-related | forget-related declarations |
|---|---:|---|---|
| `sources/hindsight/hindsight-api-slim/tests/test_attachment_lifecycle.py:1` | 13 | no | `test_deleting_the_bank_deletes_its_attachment_blobs` `sources/hindsight/hindsight-api-slim/tests/test_attachment_lifecycle.py:138`; `test_deleting_the_bank_deletes_blobs_stored_under_the_tenantless_layout` `sources/hindsight/hindsight-api-slim/tests/test_attachment_lifecycle.py:166`; `test_renaming_the_bank_moves_its_blobs_so_deleting_it_leaves_nothing` `sources/hindsight/hindsight-api-slim/tests/test_attachment_lifecycle.py:185`; `test_deleting_the_document_reclaims_its_blob` `sources/hindsight/hindsight-api-slim/tests/test_attachment_lifecycle.py:228`; `test_deleting_one_document_leaves_the_others_copy_of_the_same_image` `sources/hindsight/hindsight-api-slim/tests/test_attachment_lifecycle.py:242`; `test_deleting_a_document_leaves_another_documents_own_attachment_alone` `sources/hindsight/hindsight-api-slim/tests/test_attachment_lifecycle.py:259` |
| `sources/hindsight/hindsight-api-slim/tests/test_audit_log.py:1` | 17 | no | `test_destructive_actions_cover_every_delete_route` `sources/hindsight/hindsight-api-slim/tests/test_audit_log.py:516` |
| `sources/hindsight/hindsight-api-slim/tests/test_bank_aliases.py:1` | 21 | no | `test_deleting_an_alias_of_another_bank_is_a_404` `sources/hindsight/hindsight-api-slim/tests/test_bank_aliases.py:146`; `test_deleting_the_bank_takes_its_aliases_with_it` `sources/hindsight/hindsight-api-slim/tests/test_bank_aliases.py:159`; `test_deleting_the_primary_alias_takes_the_display_with_it` `sources/hindsight/hindsight-api-slim/tests/test_bank_aliases.py:316` |
| `sources/hindsight/hindsight-api-slim/tests/test_bank_file_storage.py:1` | 11 | no | `test_object_store_delete_prefix_stops_at_the_prefix` `sources/hindsight/hindsight-api-slim/tests/test_bank_file_storage.py:46`; `test_postgres_delete_prefix_treats_like_wildcards_literally` `sources/hindsight/hindsight-api-slim/tests/test_bank_file_storage.py:61`; `test_deleting_a_document_deletes_its_uploaded_original` `sources/hindsight/hindsight-api-slim/tests/test_bank_file_storage.py:72`; `test_deleting_a_bank_deletes_everything_under_its_prefix` `sources/hindsight/hindsight-api-slim/tests/test_bank_file_storage.py:95` |
| `sources/hindsight/hindsight-api-slim/tests/test_bank_info_cache_invalidation.py:1` | 14 | no | `test_a_deleted_bank_stops_reading_as_existing` `sources/hindsight/hindsight-api-slim/tests/test_bank_info_cache_invalidation.py:147`; `test_a_recall_of_a_bank_deleted_by_another_process_404s` `sources/hindsight/hindsight-api-slim/tests/test_bank_info_cache_invalidation.py:189`; `test_a_sql_recall_of_a_bank_deleted_by_another_process_404s` `sources/hindsight/hindsight-api-slim/tests/test_bank_info_cache_invalidation.py:277`; `test_the_bank_gone_check_404s_a_deleted_bank_and_passes_a_live_one` `sources/hindsight/hindsight-api-slim/tests/test_bank_info_cache_invalidation.py:313`; `test_no_bank_read_route_500s_on_a_bank_deleted_by_another_process` `sources/hindsight/hindsight-api-slim/tests/test_bank_info_cache_invalidation.py:339` |
| `sources/hindsight/hindsight-api-slim/tests/test_bank_lifecycle_deadlock_retry.py:1` | 4 | no | `test_bank_delete_retries_transient_deadlock` `sources/hindsight/hindsight-api-slim/tests/test_bank_lifecycle_deadlock_retry.py:53` |
| `sources/hindsight/hindsight-api-slim/tests/test_bank_stats_cache.py:1` | 13 | no | `test_invalidate_drops_entry` `sources/hindsight/hindsight-api-slim/tests/test_bank_stats_cache.py:217`; `test_invalidate_detaches_in_flight_loader` `sources/hindsight/hindsight-api-slim/tests/test_bank_stats_cache.py:228` |
| `sources/hindsight/hindsight-api-slim/tests/test_bank_stats_cache_invalidation.py:1` | 3 | no | `test_delete_memory_unit_invalidates_stats_cache` `sources/hindsight/hindsight-api-slim/tests/test_bank_stats_cache_invalidation.py:142`; `test_delete_document_invalidates_stats_cache` `sources/hindsight/hindsight-api-slim/tests/test_bank_stats_cache_invalidation.py:170`; `test_clear_observations_invalidates_stats_cache` `sources/hindsight/hindsight-api-slim/tests/test_bank_stats_cache_invalidation.py:196` |
| `sources/hindsight/hindsight-api-slim/tests/test_bank_templates.py:1` | 41 | no | `test_import_handles_resource_deleted_before_write` `sources/hindsight/hindsight-api-slim/tests/test_bank_templates.py:625` |
| `sources/hindsight/hindsight-api-slim/tests/test_bank_vector_index_ddl_lock.py:1` | 4 | no | `test_concurrent_bank_delete_storm_does_not_deadlock` `sources/hindsight/hindsight-api-slim/tests/test_bank_vector_index_ddl_lock.py:106` |
| `sources/hindsight/hindsight-api-slim/tests/test_chunk_storage_delete_ordering.py:1` | 4 | no | `test_delete_chunks_by_ids_predeletes_links_before_chunks` `sources/hindsight/hindsight-api-slim/tests/test_chunk_storage_delete_ordering.py:31`; `test_delete_chunks_by_ids_matches_link_endpoints_through_indexable_joins` `sources/hindsight/hindsight-api-slim/tests/test_chunk_storage_delete_ordering.py:64`; `test_delete_chunks_by_ids_noops_without_chunks` `sources/hindsight/hindsight-api-slim/tests/test_chunk_storage_delete_ordering.py:87`; `test_delete_chunks_by_ids_sweeps_links_on_both_endpoints` `sources/hindsight/hindsight-api-slim/tests/test_chunk_storage_delete_ordering.py:96` |
| `sources/hindsight/hindsight-api-slim/tests/test_consolidation_batch_atomicity.py:1` | 5 | no | `test_failed_create_rolls_back_the_delete_it_was_replacing` `sources/hindsight/hindsight-api-slim/tests/test_consolidation_batch_atomicity.py:134` |
| `sources/hindsight/hindsight-api-slim/tests/test_consolidation_dedup.py:1` | 37 | no | `test_dedup_update_merge_folds_into_twin_and_deletes_updated` `sources/hindsight/hindsight-api-slim/tests/test_consolidation_dedup.py:489`; `test_dedup_create_all_new_sources_deleted_returns_none` `sources/hindsight/hindsight-api-slim/tests/test_consolidation_dedup.py:639`; `test_dedup_update_twin_vanished_does_not_delete_updated` `sources/hindsight/hindsight-api-slim/tests/test_consolidation_dedup.py:659`; `test_dedup_update_all_updated_sources_deleted_skips_fold_and_delete` `sources/hindsight/hindsight-api-slim/tests/test_consolidation_dedup.py:696` |
| `sources/hindsight/hindsight-api-slim/tests/test_consolidation_delete_prompt_llm.py:1` | 1 | no | `test_real_model_emits_a_delete_that_names_its_target` `sources/hindsight/hindsight-api-slim/tests/test_consolidation_delete_prompt_llm.py:36` |
| `sources/hindsight/hindsight-api-slim/tests/test_consolidation_delete_schema.py:1` | 9 | no | `test_delete_accepts_the_id_alias` `sources/hindsight/hindsight-api-slim/tests/test_consolidation_delete_schema.py:65`; `test_delete_still_accepts_the_canonical_name` `sources/hindsight/hindsight-api-slim/tests/test_consolidation_delete_schema.py:72`; `test_delete_naming_no_target_still_fails_closed` `sources/hindsight/hindsight-api-slim/tests/test_consolidation_delete_schema.py:80`; `test_prompt_states_that_deletes_need_an_observation_id` `sources/hindsight/hindsight-api-slim/tests/test_consolidation_delete_schema.py:121`; `test_prompt_shows_a_worked_example_of_a_populated_deletes_array` `sources/hindsight/hindsight-api-slim/tests/test_consolidation_delete_schema.py:127`; `test_supersession_delete_lands_under_either_field_name` `sources/hindsight/hindsight-api-slim/tests/test_consolidation_delete_schema.py:201`; `test_delete_with_no_identifier_fails_the_batch_and_is_reported` `sources/hindsight/hindsight-api-slim/tests/test_consolidation_delete_schema.py:250` |
| `sources/hindsight/hindsight-api-slim/tests/test_consolidation_triage.py:1` | 13 | no | `test_evidence_never_revives_an_observation_the_llm_deleted` `sources/hindsight/hindsight-api-slim/tests/test_consolidation_triage.py:199` |
| `sources/hindsight/hindsight-api-slim/tests/test_consolidation_update_delete_same_observation.py:1` | 1 | no | `test_update_wins_over_delete_of_the_same_observation` `sources/hindsight/hindsight-api-slim/tests/test_consolidation_update_delete_same_observation.py:30` |
| `sources/hindsight/hindsight-api-slim/tests/test_curation_http.py:1` | 5 | no | `test_patch_invalidate_and_revert_over_http` `sources/hindsight/hindsight-api-slim/tests/test_curation_http.py:59` |
| `sources/hindsight/hindsight-api-slim/tests/test_curation_validator_hooks.py:1` | 8 | no | `test_invalidate_reembeds_nothing_and_revert_reembeds_the_restored_text` `sources/hindsight/hindsight-api-slim/tests/test_curation_validator_hooks.py:117`; `test_edit_and_invalidate_reports_the_state_change_and_the_edited_text` `sources/hindsight/hindsight-api-slim/tests/test_curation_validator_hooks.py:142` |
| `sources/hindsight/hindsight-api-slim/tests/test_delete_bank_leaves_no_rows.py:1` | 2 | no | `test_deleting_a_bank_leaves_no_row_in_any_table` `sources/hindsight/hindsight-api-slim/tests/test_delete_bank_leaves_no_rows.py:195` |
| `sources/hindsight/hindsight-api-slim/tests/test_delete_document_concurrency.py:1` | 6 | no | `test_shared_observation_deletes_complete_after_commit_or_rollback` `sources/hindsight/hindsight-api-slim/tests/test_delete_document_concurrency.py:230`; `test_reingest_and_delete_of_observation_sources_both_complete` `sources/hindsight/hindsight-api-slim/tests/test_delete_document_concurrency.py:266`; `test_reingest_and_delete_of_linked_documents_both_complete` `sources/hindsight/hindsight-api-slim/tests/test_delete_document_concurrency.py:313`; `test_other_bank_can_delete_while_first_bank_is_paused` `sources/hindsight/hindsight-api-slim/tests/test_delete_document_concurrency.py:413`; `test_bank_cleanup_and_document_delete_use_same_lock_order` `sources/hindsight/hindsight-api-slim/tests/test_delete_document_concurrency.py:437` |
| `sources/hindsight/hindsight-api-slim/tests/test_delete_missing_bank_store_owned.py:1` | 2 | no | `test_delete_missing_bank_never_asks_the_store` `sources/hindsight/hindsight-api-slim/tests/test_delete_missing_bank_store_owned.py:111`; `test_delete_existing_bank_still_goes_through_the_store` `sources/hindsight/hindsight-api-slim/tests/test_delete_missing_bank_store_owned.py:123` |
| `sources/hindsight/hindsight-api-slim/tests/test_delta_removed_chunks.py:1` | 4 | no | `test_pure_deletion_drops_the_removed_chunks` `sources/hindsight/hindsight-api-slim/tests/test_delta_removed_chunks.py:96`; `test_pure_deletion_only_deletes` `sources/hindsight/hindsight-api-slim/tests/test_delta_removed_chunks.py:119`; `test_middle_deletion_still_drops_the_removed_section` `sources/hindsight/hindsight-api-slim/tests/test_delta_removed_chunks.py:159` |
| `sources/hindsight/hindsight-api-slim/tests/test_delta_retain.py:1` | 23 | no | `test_delta_retain_removed_chunks_delete_facts` `sources/hindsight/hindsight-api-slim/tests/test_delta_retain.py:570` |
| `sources/hindsight/hindsight-api-slim/tests/test_document_history.py:1` | 8 | no | `test_a_delete_files_each_document_under_the_observations_its_own_facts_held` `sources/hindsight/hindsight-api-slim/tests/test_document_history.py:52`; `test_append_invalidation_and_retag_are_each_recorded` `sources/hindsight/hindsight-api-slim/tests/test_document_history.py:148`; `test_deletes_and_clears_are_recorded_on_the_consolidation_they_queue` `sources/hindsight/hindsight-api-slim/tests/test_document_history.py:185` |
| `sources/hindsight/hindsight-api-slim/tests/test_document_tracking.py:1` | 14 | no | `test_document_deletion` `sources/hindsight/hindsight-api-slim/tests/test_document_tracking.py:91`; `test_document_deletion_counts_only_its_own_bank_units` `sources/hindsight/hindsight-api-slim/tests/test_document_tracking.py:143` |
| `sources/hindsight/hindsight-api-slim/tests/test_document_transfer.py:1` | 85 | no | `test_delete_operation_removes_export_archive` `sources/hindsight/hindsight-api-slim/tests/test_document_transfer.py:2353`; `test_purge_expired_export_archives` `sources/hindsight/hindsight-api-slim/tests/test_document_transfer.py:2375`; `test_purge_expired_export_archives_includes_export_bank` `sources/hindsight/hindsight-api-slim/tests/test_document_transfer.py:2413`; `test_purge_expired_export_archives_honours_the_batch_bound` `sources/hindsight/hindsight-api-slim/tests/test_document_transfer.py:2446`; `test_invalidated_facts_survive_a_bank_copy` `sources/hindsight/hindsight-api-slim/tests/test_document_transfer.py:3178` |
| `sources/hindsight/hindsight-api-slim/tests/test_entity_mention_count.py:1` | 16 | no | `test_invalidate_then_revert_restores_the_mention` `sources/hindsight/hindsight-api-slim/tests/test_entity_mention_count.py:343`; `test_deleting_a_document_gives_back_only_its_own_mentions` `sources/hindsight/hindsight-api-slim/tests/test_entity_mention_count.py:408`; `test_racing_deletes_of_one_memory_give_back_its_mention_once` `sources/hindsight/hindsight-api-slim/tests/test_entity_mention_count.py:451` |
| `sources/hindsight/hindsight-api-slim/tests/test_extension_bank_tables.py:1` | 9 | no | `test_delete_bank_sweeps_extension_tables` `sources/hindsight/hindsight-api-slim/tests/test_extension_bank_tables.py:113` |
| `sources/hindsight/hindsight-api-slim/tests/test_file_storage_s3.py:1` | 6 | no | `test_s3_storage_exists_and_delete` `sources/hindsight/hindsight-api-slim/tests/test_file_storage_s3.py:209` |
| `sources/hindsight/hindsight-api-slim/tests/test_fk_violation_is_retryable.py:1` | 2 | no | `test_retain_fk_violation_from_concurrent_delete_is_retryable` `sources/hindsight/hindsight-api-slim/tests/test_fk_violation_is_retryable.py:40` |
| `sources/hindsight/hindsight-api-slim/tests/test_github_copilot_llm.py:1` | 17 | no | `test_timeout_aborts_and_deletes_session` `sources/hindsight/hindsight-api-slim/tests/test_github_copilot_llm.py:285`; `test_runtime_failure_invalidates_shared_client` `sources/hindsight/hindsight-api-slim/tests/test_github_copilot_llm.py:329`; `test_session_cleanup_timeout_invalidates_runtime` `sources/hindsight/hindsight-api-slim/tests/test_github_copilot_llm.py:342`; `test_shared_runtime_invalidation_replaces_dead_client` `sources/hindsight/hindsight-api-slim/tests/test_github_copilot_llm.py:379` |
| `sources/hindsight/hindsight-api-slim/tests/test_graph_maintenance.py:1` | 35 | no | `test_enqueues_units_with_outgoing_link_to_deleted` `sources/hindsight/hindsight-api-slim/tests/test_graph_maintenance.py:183`; `test_excludes_deleted_units_themselves` `sources/hindsight/hindsight-api-slim/tests/test_graph_maintenance.py:204`; `test_delete_document_enqueues_cross_doc_victims` `sources/hindsight/hindsight-api-slim/tests/test_graph_maintenance.py:364`; `test_delete_isolated_document_prunes_its_entities` `sources/hindsight/hindsight-api-slim/tests/test_graph_maintenance.py:389`; `test_dedupes_across_overlapping_deletes` `sources/hindsight/hindsight-api-slim/tests/test_graph_maintenance.py:940`; `test_delete_memory_unit_prunes_the_entity_it_was_holding_up` `sources/hindsight/hindsight-api-slim/tests/test_graph_maintenance.py:961`; `test_delete_memory_unit_reconciles_vector_indexes_for_an_observation` `sources/hindsight/hindsight-api-slim/tests/test_graph_maintenance.py:988`; `test_delta_retain_chunk_delete` `sources/hindsight/hindsight-api-slim/tests/test_graph_maintenance.py:1028`; `test_curation_invalidate` `sources/hindsight/hindsight-api-slim/tests/test_graph_maintenance.py:1099` |
| `sources/hindsight/hindsight-api-slim/tests/test_hnsw_indexes.py:1` | 14 | no | `test_delete_bank_drops_vector_indexes` `sources/hindsight/hindsight-api-slim/tests/test_hnsw_indexes.py:184` |
| `sources/hindsight/hindsight-api-slim/tests/test_http_api_integration.py:1` | 62 | no | `test_document_deletion` `sources/hindsight/hindsight-api-slim/tests/test_http_api_integration.py:485`; `test_document_deletion_with_slashes_in_id` `sources/hindsight/hindsight-api-slim/tests/test_http_api_integration.py:536`; `test_delete_bank` `sources/hindsight/hindsight-api-slim/tests/test_http_api_integration.py:594`; `test_delete_bank_nonexistent` `sources/hindsight/hindsight-api-slim/tests/test_http_api_integration.py:668`; `test_config_only_import_returns_409_when_bank_is_deleted_before_config_write` `sources/hindsight/hindsight-api-slim/tests/test_http_api_integration.py:1953` |
| `sources/hindsight/hindsight-api-slim/tests/test_knowledge_base.py:1` | 95 | no | `test_move_serializes_behind_a_concurrent_delete` `sources/hindsight/hindsight-api-slim/tests/test_knowledge_base.py:1157`; `test_create_serializes_behind_a_concurrent_delete` `sources/hindsight/hindsight-api-slim/tests/test_knowledge_base.py:1192`; `test_delete_folder_cascades` `sources/hindsight/hindsight-api-slim/tests/test_knowledge_base.py:1402`; `test_delete_denied` `sources/hindsight/hindsight-api-slim/tests/test_knowledge_base.py:1881` |
| `sources/hindsight/hindsight-api-slim/tests/test_link_utils.py:1` | 30 | no | `test_matches_delete_total_order` `sources/hindsight/hindsight-api-slim/tests/test_link_utils.py:268` |
| `sources/hindsight/hindsight-api-slim/tests/test_list_time_window.py:1` | 10 | no | `test_memories_window_applies_to_invalidated_archive` `sources/hindsight/hindsight-api-slim/tests/test_list_time_window.py:187` |
| `sources/hindsight/hindsight-api-slim/tests/test_llm_trace.py:1` | 54 | no | `test_deleting_bank_deletes_its_request_history` `sources/hindsight/hindsight-api-slim/tests/test_llm_trace.py:1424`; `test_bank_delete_and_memory_id_patch_lock_rows_in_id_order` `sources/hindsight/hindsight-api-slim/tests/test_llm_trace.py:1446`; `test_deleting_document_deletes_its_request_history_when_enabled` `sources/hindsight/hindsight-api-slim/tests/test_llm_trace.py:1503`; `test_deleting_document_deletes_its_rows_still_buffered` `sources/hindsight/hindsight-api-slim/tests/test_llm_trace.py:1554` |
| `sources/hindsight/hindsight-api-slim/tests/test_maintenance_loop.py:1` | 26 | no | `test_purge_expired_deletes_old_rows_across_schema` `sources/hindsight/hindsight-api-slim/tests/test_maintenance_loop.py:321`; `test_chunked_delete_drains_every_expired_row` `sources/hindsight/hindsight-api-slim/tests/test_maintenance_loop.py:386`; `test_archive_purge_shares_the_prune_batch_bound` `sources/hindsight/hindsight-api-slim/tests/test_maintenance_loop.py:588` |
| `sources/hindsight/hindsight-api-slim/tests/test_mcp_tools.py:1` | 266 | no | `test_delete_multi_bank` `sources/hindsight/hindsight-api-slim/tests/test_mcp_tools.py:1001`; `test_delete_with_bank_id_override` `sources/hindsight/hindsight-api-slim/tests/test_mcp_tools.py:1006`; `test_delete_not_found_multi_bank` `sources/hindsight/hindsight-api-slim/tests/test_mcp_tools.py:1012`; `test_delete_not_found_single_bank` `sources/hindsight/hindsight-api-slim/tests/test_mcp_tools.py:1017`; `test_delete_single_bank` `sources/hindsight/hindsight-api-slim/tests/test_mcp_tools.py:1023`; `test_delete_no_bank_returns_error` `sources/hindsight/hindsight-api-slim/tests/test_mcp_tools.py:1028`; `test_delete_engine_error` `sources/hindsight/hindsight-api-slim/tests/test_mcp_tools.py:1032`; `test_delete_directive` `sources/hindsight/hindsight-api-slim/tests/test_mcp_tools.py:1585`; `test_delete_directive_not_found` `sources/hindsight/hindsight-api-slim/tests/test_mcp_tools.py:1591`; `test_invalidate_memory` `sources/hindsight/hindsight-api-slim/tests/test_mcp_tools.py:1661`; `test_invalidate_memory_restore` `sources/hindsight/hindsight-api-slim/tests/test_mcp_tools.py:1669`; `test_delete_document` `sources/hindsight/hindsight-api-slim/tests/test_mcp_tools.py:1740`; `test_delete_bank` `sources/hindsight/hindsight-api-slim/tests/test_mcp_tools.py:1936`; `test_delete_bank_single_bank` `sources/hindsight/hindsight-api-slim/tests/test_mcp_tools.py:1972`; `test_delete_document_engine_error` `sources/hindsight/hindsight-api-slim/tests/test_mcp_tools.py:2027`; `test_delete_document_single_bank` `sources/hindsight/hindsight-api-slim/tests/test_mcp_tools.py:2034`; `test_delete_node` `sources/hindsight/hindsight-api-slim/tests/test_mcp_tools.py:2457`; `test_delete_node_not_found` `sources/hindsight/hindsight-api-slim/tests/test_mcp_tools.py:2462`; `test_delete_bank_logged_with_audit_disabled` `sources/hindsight/hindsight-api-slim/tests/test_mcp_tools.py:2870` |
| `sources/hindsight/hindsight-api-slim/tests/test_memories_extension.py:1` | 61 | no | `test_deleting_a_bank_drops_its_storage` `sources/hindsight/hindsight-api-slim/tests/test_memories_extension.py:2048`; `test_bulk_delete_removes_store_owned_memories` `sources/hindsight/hindsight-api-slim/tests/test_memories_extension.py:2206`; `test_a_retag_that_deletes_an_observation_requeues_its_sources` `sources/hindsight/hindsight-api-slim/tests/test_memories_extension.py:2696` |
| `sources/hindsight/hindsight-api-slim/tests/test_memory_curation.py:1` | 27 | no | `test_invalidate_moves_to_archive_and_prunes` `sources/hindsight/hindsight-api-slim/tests/test_memory_curation.py:228`; `test_invalidate_idempotent_updates_reason` `sources/hindsight/hindsight-api-slim/tests/test_memory_curation.py:321`; `test_cannot_edit_invalidated_memory` `sources/hindsight/hindsight-api-slim/tests/test_memory_curation.py:717`; `test_recall_excludes_invalidated` `sources/hindsight/hindsight-api-slim/tests/test_memory_curation.py:1049`; `test_invalidate_snapshots_and_revert_restores` `sources/hindsight/hindsight-api-slim/tests/test_memory_curation.py:1148`; `test_revert_skips_permanently_deleted_peer` `sources/hindsight/hindsight-api-slim/tests/test_memory_curation.py:1255` |
| `sources/hindsight/hindsight-api-slim/tests/test_memory_links_to_unit_id_concurrent_delete.py:1` | 1 | no | `test_to_unit_id_survives_concurrent_neighbor_delete` `sources/hindsight/hindsight-api-slim/tests/test_memory_links_to_unit_id_concurrent_delete.py:58` |
| `sources/hindsight/hindsight-api-slim/tests/test_memory_units_updated_at.py:1` | 6 | no | `test_invalidation_reason_change_stamps_updated_at` `sources/hindsight/hindsight-api-slim/tests/test_memory_units_updated_at.py:144` |
| `sources/hindsight/hindsight-api-slim/tests/test_mental_model_retractions.py:1` | 28 | no | `test_invalidating_a_cited_fact_makes_the_page_stale` `sources/hindsight/hindsight-api-slim/tests/test_mental_model_retractions.py:280`; `test_deleting_a_cited_fact_makes_the_page_stale` `sources/hindsight/hindsight-api-slim/tests/test_mental_model_retractions.py:308` |
| `sources/hindsight/hindsight-api-slim/tests/test_observation_history.py:1` | 4 | no | `test_history_is_reclaimed_when_observations_deleted_by_type` `sources/hindsight/hindsight-api-slim/tests/test_observation_history.py:105` |
| `sources/hindsight/hindsight-api-slim/tests/test_observation_invalidation.py:1` | 39 | no | `test_deleting_source_memory_removes_observation` `sources/hindsight/hindsight-api-slim/tests/test_observation_invalidation.py:175`; `test_deleting_source_memory_resets_remaining_source_consolidated_at` `sources/hindsight/hindsight-api-slim/tests/test_observation_invalidation.py:197`; `test_deleting_non_source_memory_leaves_observations_intact` `sources/hindsight/hindsight-api-slim/tests/test_observation_invalidation.py:225`; `test_deleting_sole_source_memory_removes_observation_no_remaining_reset` `sources/hindsight/hindsight-api-slim/tests/test_observation_invalidation.py:251`; `test_deleting_observation_type_memory_does_not_trigger_invalidation` `sources/hindsight/hindsight-api-slim/tests/test_observation_invalidation.py:272`; `test_deleting_document_removes_observations` `sources/hindsight/hindsight-api-slim/tests/test_observation_invalidation.py:303`; `test_chunk_delete_removes_observations_from_outgoing_memories` `sources/hindsight/hindsight-api-slim/tests/test_observation_invalidation.py:498`; `test_chunk_delete_keeps_observations_of_unchanged_chunks` `sources/hindsight/hindsight-api-slim/tests/test_observation_invalidation.py:544`; `test_chunk_delete_sweeps_every_deleted_chunk` `sources/hindsight/hindsight-api-slim/tests/test_observation_invalidation.py:583`; `test_update_tags_invalidates_observations` `sources/hindsight/hindsight-api-slim/tests/test_observation_invalidation.py:925`; `test_update_tags_triggers_consolidation_when_observations_invalidated` `sources/hindsight/hindsight-api-slim/tests/test_observation_invalidation.py:976`; `test_repatching_identical_tags_invalidates_nothing` `sources/hindsight/hindsight-api-slim/tests/test_observation_invalidation.py:1062`; `test_partial_tag_overlap_still_invalidates` `sources/hindsight/hindsight-api-slim/tests/test_observation_invalidation.py:1155`; `test_create_observation_filters_deleted_source_memories` `sources/hindsight/hindsight-api-slim/tests/test_observation_invalidation.py:1343`; `test_create_observation_skipped_when_all_sources_deleted` `sources/hindsight/hindsight-api-slim/tests/test_observation_invalidation.py:1409`; `test_update_observation_skipped_when_all_new_sources_deleted` `sources/hindsight/hindsight-api-slim/tests/test_observation_invalidation.py:1436`; `test_update_observation_skipped_when_source_deleted_after_preflight` `sources/hindsight/hindsight-api-slim/tests/test_observation_invalidation.py:1482` |
| `sources/hindsight/hindsight-api-slim/tests/test_op_cancellation.py:1` | 15 | no | `test_bank_deletion_cascades_to_async_operations` `sources/hindsight/hindsight-api-slim/tests/test_op_cancellation.py:77`; `test_bank_deletion_cascades_to_webhooks` `sources/hindsight/hindsight-api-slim/tests/test_op_cancellation.py:93`; `test_returns_false_when_op_deleted` `sources/hindsight/hindsight-api-slim/tests/test_op_cancellation.py:140`; `test_returns_false_after_bank_cascade_delete` `sources/hindsight/hindsight-api-slim/tests/test_op_cancellation.py:159` |
| `sources/hindsight/hindsight-api-slim/tests/test_operation_status.py:1` | 17 | no | `test_retry_does_not_acknowledge_operation_deleted_by_cleanup` `sources/hindsight/hindsight-api-slim/tests/test_operation_status.py:239`; `test_delete_removes_terminal_operation` `sources/hindsight/hindsight-api-slim/tests/test_operation_status.py:397`; `test_delete_rejects_non_terminal_statuses` `sources/hindsight/hindsight-api-slim/tests/test_operation_status.py:422`; `test_delete_wrong_bank_returns_404` `sources/hindsight/hindsight-api-slim/tests/test_operation_status.py:434` |
| `sources/hindsight/hindsight-api-slim/tests/test_oracle_integration.py:1` | 56 | no | `test_delete_memory` `sources/hindsight/hindsight-api-slim/tests/test_oracle_integration.py:250`; `test_delete_document` `sources/hindsight/hindsight-api-slim/tests/test_oracle_integration.py:275`; `test_semantic_ann_links_delete_chunks` `sources/hindsight/hindsight-api-slim/tests/test_oracle_integration.py:681`; `test_delete_nonexistent_bank` `sources/hindsight/hindsight-api-slim/tests/test_oracle_integration.py:1638`; `test_retain_and_delete_cycle` `sources/hindsight/hindsight-api-slim/tests/test_oracle_integration.py:1644` |
| `sources/hindsight/hindsight-api-slim/tests/test_reflections.py:1` | 27 | no | `test_delete_mental_model` `sources/hindsight/hindsight-api-slim/tests/test_reflections.py:347` |
| `sources/hindsight/hindsight-api-slim/tests/test_repair_bank_vector_indexes.py:1` | 46 | no | `test_index_orphaned_by_a_deleted_bank_is_dropped` `sources/hindsight/hindsight-api-slim/tests/test_repair_bank_vector_indexes.py:406` |
| `sources/hindsight/hindsight-api-slim/tests/test_retain_consolidation_deadlock.py:1` | 2 | no | `test_delete_document_and_consolidation_do_not_deadlock` `sources/hindsight/hindsight-api-slim/tests/test_retain_consolidation_deadlock.py:296` |
| `sources/hindsight/hindsight-api-slim/tests/test_retain_tag_change_observations.py:1` | 7 | no | `test_tags_only_re_retain_invalidates_observations` `sources/hindsight/hindsight-api-slim/tests/test_retain_tag_change_observations.py:120`; `test_tag_change_with_partial_edit_invalidates_unchanged_chunks_observations` `sources/hindsight/hindsight-api-slim/tests/test_retain_tag_change_observations.py:162`; `test_observation_scopes_change_re_retain_invalidates_observations` `sources/hindsight/hindsight-api-slim/tests/test_retain_tag_change_observations.py:213` |
| `sources/hindsight/hindsight-api-slim/tests/test_webhooks.py:1` | 72 | no | `test_http_delete_webhook` `sources/hindsight/hindsight-api-slim/tests/test_webhooks.py:1028`; `test_http_delete_webhook_not_found` `sources/hindsight/hindsight-api-slim/tests/test_webhooks.py:1049`; `test_delete_webhook_targets_resolved_schema` `sources/hindsight/hindsight-api-slim/tests/test_webhooks.py:2294` |
| `sources/hindsight/hindsight-api-slim/tests/test_worker.py:1` | 139 | no | `test_worker_fire_and_forget_nonblocking` `sources/hindsight/hindsight-api-slim/tests/test_worker.py:3414`; `test_pruning_cancelled_child_cancels_parent_before_deletion_and_releases_siblings_later` `sources/hindsight/hindsight-api-slim/tests/test_worker.py:5494`; `test_postgresql_pruning_reconciles_pending_same_bank_parent_before_child_delete` `sources/hindsight/hindsight-api-slim/tests/test_worker.py:5607`; `test_oracle_pruning_reconciles_pending_same_bank_parent_before_child_delete` `sources/hindsight/hindsight-api-slim/tests/test_worker.py:5673` |
| `sources/hindsight/hindsight-cli/tests/cli_integration.rs:1` | 26 | no | `test_bank_create_and_delete` `sources/hindsight/hindsight-cli/tests/cli_integration.rs:141`; `test_mental_model_create_and_delete` `sources/hindsight/hindsight-cli/tests/cli_integration.rs:242`; `test_directive_create_get_update_delete` `sources/hindsight/hindsight-cli/tests/cli_integration.rs:526` |
| `sources/hindsight/hindsight-clients/go/integration_test.go:1` | 15 | no | `TestDeleteBank` `sources/hindsight/hindsight-clients/go/integration_test.go:396` |
| `sources/hindsight/hindsight-clients/python/tests/test_main_operations.py:1` | 37 | no | `test_delete_document` `sources/hindsight/hindsight-clients/python/tests/test_main_operations.py:372`; `test_delete_bank` `sources/hindsight/hindsight-clients/python/tests/test_main_operations.py:679`; `test_adelete_bank` `sources/hindsight/hindsight-clients/python/tests/test_main_operations.py:881` |
| `sources/hindsight/hindsight-clients/typescript/tests/main_operations.test.ts:1` | 39 | no | `delete document` `sources/hindsight/hindsight-clients/typescript/tests/main_operations.test.ts:304`; `delete bank` `sources/hindsight/hindsight-clients/typescript/tests/main_operations.test.ts:400` |
| `sources/hindsight/hindsight-control-plane/tests/app/api/documents/route.test.ts:1` | 10 | no | `DELETE passes the id through to the dataplane intact` `sources/hindsight/hindsight-control-plane/tests/app/api/documents/route.test.ts:113` |
| `sources/hindsight/hindsight-control-plane/tests/components/banks-overview.test.tsx:1` | 8 | no | `deletes the selected banks one at a time, never concurrently` `sources/hindsight/hindsight-control-plane/tests/components/banks-overview.test.tsx:88`; `requires the confirmation word before the delete can run` `sources/hindsight/hindsight-control-plane/tests/components/banks-overview.test.tsx:196` |
| `sources/hindsight/hindsight-control-plane/tests/lib/api.test.ts:1` | 15 | no | `issues DELETE to /api/banks/{bankId}/operations/{opId}` `sources/hindsight/hindsight-control-plane/tests/lib/api.test.ts:194` |
| `sources/hindsight/hindsight-embed/tests/test_control_center.py:1` | 52 | no | `test_delete_named_profile` `sources/hindsight/hindsight-embed/tests/test_control_center.py:187`; `test_delete_default_profile_refused` `sources/hindsight/hindsight-embed/tests/test_control_center.py:194`; `test_delete_profile_over_http` `sources/hindsight/hindsight-embed/tests/test_control_center.py:413`; `test_delete_keeps_profile_when_stop_fails` `sources/hindsight/hindsight-embed/tests/test_control_center.py:471` |
| `sources/hindsight/hindsight-embed/tests/test_profile_integration.py:1` | 11 | no | `test_delete_profile` `sources/hindsight/hindsight-embed/tests/test_profile_integration.py:176` |
| `sources/hindsight/hindsight-embed/tests/test_profile_lock.py:1` | 6 | no | `test_delete_profile_removes_the_lock_owner_sidecar` `sources/hindsight/hindsight-embed/tests/test_profile_lock.py:105` |
| `sources/hindsight/hindsight-embed/tests/test_profile_manager.py:1` | 35 | no | `test_delete_profile` `sources/hindsight/hindsight-embed/tests/test_profile_manager.py:146`; `test_delete_profile_survives_undeletable_log` `sources/hindsight/hindsight-embed/tests/test_profile_manager.py:171`; `test_delete_nonexistent_profile` `sources/hindsight/hindsight-embed/tests/test_profile_manager.py:196`; `test_delete_default_profile_fails` `sources/hindsight/hindsight-embed/tests/test_profile_manager.py:201`; `test_delete_active_profile_clears_active_file` `sources/hindsight/hindsight-embed/tests/test_profile_manager.py:410` |
| `sources/hindsight/hindsight-integrations/coding-agents/src/core/bank-missing-dir.test.ts:1` | 11 | no | `names the repository rather than the deleted leaf directory` `sources/hindsight/hindsight-integrations/coding-agents/src/core/bank-missing-dir.test.ts:29` |
| `sources/hindsight/hindsight-integrations/coding-agents/src/core/git.gitlog.test.ts:1` | 25 | no | `never enumerates or deletes foreign git-log documents from a shared bank` `sources/hindsight/hindsight-integrations/coding-agents/src/core/git.gitlog.test.ts:202` |
| `sources/hindsight/hindsight-integrations/coding-agents/src/core/hindsight.pages.test.ts:1` | 65 | no | `seedPages: a page deleted under us is skipped, not the rest of the sync` `sources/hindsight/hindsight-integrations/coding-agents/src/core/hindsight.pages.test.ts:261`; `an initiative page deleted under us is skipped, not the rest of the folder` `sources/hindsight/hindsight-integrations/coding-agents/src/core/hindsight.pages.test.ts:774`; `configureBank({reset: true}) DELETEs the bank before the import POST` `sources/hindsight/hindsight-integrations/coding-agents/src/core/hindsight.pages.test.ts:1390` |
| `sources/hindsight/hindsight-integrations/coding-agents/src/core/missions.test.ts:1` | 44 | no | `never deletes a strategy the user defined, nor reverts their edits to ours` `sources/hindsight/hindsight-integrations/coding-agents/src/core/missions.test.ts:337` |
| `sources/hindsight/hindsight-integrations/coding-agents/src/installer.test.ts:1` | 194 | no | `uninstall removes our entry and deletes the plugin key when empty` `sources/hindsight/hindsight-integrations/coding-agents/src/installer.test.ts:1688`; `uninstall removes our entry and deletes the plugin key when empty` `sources/hindsight/hindsight-integrations/coding-agents/src/installer.test.ts:1831`; `uninstall removes our entry and deletes the extensions key when empty` `sources/hindsight/hindsight-integrations/coding-agents/src/installer.test.ts:1870`; `uninstall deletes a Grok hooks file left empty` `sources/hindsight/hindsight-integrations/coding-agents/src/installer.test.ts:2271` |
| `sources/hindsight/hindsight-integrations/crewai/tests/test_storage.py:1` | 20 | no | `test_reset_deletes_bank` `sources/hindsight/hindsight-integrations/crewai/tests/test_storage.py:198` |
| `sources/hindsight/hindsight-integrations/devin-desktop/tests/test_global_rules.py:1` | 8 | no | `test_clear_deletes_file_if_only_our_block` `sources/hindsight/hindsight-integrations/devin-desktop/tests/test_global_rules.py:75` |
| `sources/hindsight/hindsight-integrations/devin-desktop/tests/test_managed_block.py:1` | 6 | no | `test_clear_deletes_file_if_only_block` `sources/hindsight/hindsight-integrations/devin-desktop/tests/test_managed_block.py:53` |
| `sources/hindsight/hindsight-integrations/devin-desktop/tests/test_rules.py:1` | 11 | no | `test_clear_deletes_our_file` `sources/hindsight/hindsight-integrations/devin-desktop/tests/test_rules.py:80` |
| `sources/hindsight/hindsight-integrations/eliza/tests/evaluator.test.ts:1` | 13 | no | `does not block the turn on a slow retain in async mode (fire-and-forget)` `sources/hindsight/hindsight-integrations/eliza/tests/evaluator.test.ts:37` |
| `sources/hindsight/hindsight-integrations/github-copilot/tests/test_instructions.py:1` | 6 | no | `test_clear_deletes_if_only_block` `sources/hindsight/hindsight-integrations/github-copilot/tests/test_instructions.py:38` |
| `sources/hindsight/hindsight-integrations/hermes/tests/conftest.py:1` | 0 | yes | none |
| `sources/hindsight/hindsight-integrations/hermes/tests/test_env_settings.py:1` | 4 | yes | none |
| `sources/hindsight/hindsight-integrations/hermes/tests/test_local_runtime.py:1` | 24 | yes | none |
| `sources/hindsight/hindsight-integrations/hermes/tests/test_multibank.py:1` | 48 | yes | none |
| `sources/hindsight/hindsight-integrations/hermes/tests/test_provider.py:1` | 73 | yes | none |
| `sources/hindsight/hindsight-integrations/hermes/tests/test_retain_durability.py:1` | 8 | yes | none |
| `sources/hindsight/hindsight-integrations/hermes/tests/test_settings.py:1` | 9 | yes | none |
| `sources/hindsight/hindsight-integrations/obsidian/tests/client.spec.ts:1` | 13 | no | `deleteDocument encodes segments but preserves path slashes` `sources/hindsight/hindsight-integrations/obsidian/tests/client.spec.ts:47`; `deleteDocument treats an already-missing document as deleted (#5067)` `sources/hindsight/hindsight-integrations/obsidian/tests/client.spec.ts:57`; `deleteDocument still rejects auth, missing-bank and server errors` `sources/hindsight/hindsight-integrations/obsidian/tests/client.spec.ts:63` |
| `sources/hindsight/hindsight-integrations/obsidian/tests/node/cli.spec.ts:1` | 19 | no | `upserts a live file and deletes on unlink` `sources/hindsight/hindsight-integrations/obsidian/tests/node/cli.spec.ts:151`; `refuses a stale --index whose scope changed, preventing cross-target deletes` `sources/hindsight/hindsight-integrations/obsidian/tests/node/cli.spec.ts:329` |
| `sources/hindsight/hindsight-integrations/obsidian/tests/node/e2e-http.spec.ts:1` | 2 | no | `POSTs a retain with the bearer token and correct document id, then DELETEs on prune` `sources/hindsight/hindsight-integrations/obsidian/tests/node/e2e-http.spec.ts:58` |
| `sources/hindsight/hindsight-integrations/obsidian/tests/node/reconcile-integration.spec.ts:1` | 11 | no | `prunes a document when its note is deleted from disk` `sources/hindsight/hindsight-integrations/obsidian/tests/node/reconcile-integration.spec.ts:129`; `handles a rename as delete-old + create-new` `sources/hindsight/hindsight-integrations/obsidian/tests/node/reconcile-integration.spec.ts:140` |
| `sources/hindsight/hindsight-integrations/obsidian/tests/node/watch.spec.ts:1` | 1 | no | `ingests, updates and deletes as files change on disk; ignores non-markdown` `sources/hindsight/hindsight-integrations/obsidian/tests/node/watch.spec.ts:69` |
| `sources/hindsight/hindsight-integrations/obsidian/tests/sync.spec.ts:1` | 16 | no | `deletes a document on note delete (only if previously synced)` `sources/hindsight/hindsight-integrations/obsidian/tests/sync.spec.ts:197`; `rename = delete old document + ingest new path` `sources/hindsight/hindsight-integrations/obsidian/tests/sync.spec.ts:210`; `handleDelete drops the stale entry` `sources/hindsight/hindsight-integrations/obsidian/tests/sync.spec.ts:327` |
| `sources/hindsight/hindsight-integrations/openclaw/src/tools.test.ts:1` | 13 | no | `offers delete only with allowKnowledgeDelete` `sources/hindsight/hindsight-integrations/openclaw/src/tools.test.ts:318` |
| `sources/hindsight/hindsight-integrations/openhands/tests/test_agents_md.py:1` | 6 | no | `test_clear_deletes_if_only_our_block` `sources/hindsight/hindsight-integrations/openhands/tests/test_agents_md.py:40` |
| `sources/hindsight/hindsight-integrations/vapi/tests/test_webhook.py:1` | 20 | no | `test_transcript_retained_fire_and_forget` `sources/hindsight/hindsight-integrations/vapi/tests/test_webhook.py:152` |
| `sources/hindsight/hindsight-integrations/zapier/test/triggers.test.js:1` | 10 | no | `unsubscribes by deleting the registered webhook` `sources/hindsight/hindsight-integrations/zapier/test/triggers.test.js:75` |
| `sources/hindsight/hindsight-integrations/zed/test/rulesFile.test.js:1` | 5 | no | `clear deletes the file if it held only our block` `sources/hindsight/hindsight-integrations/zed/test/rulesFile.test.js:68` |
| `sources/hindsight/hindsight-integrations/zed/test/zedSettings.test.js:1` | 7 | no | `remove deletes our entry and drops an empty context_servers` `sources/hindsight/hindsight-integrations/zed/test/zedSettings.test.js:127` |
| `sources/hindsight/hindsight-system-tests/tests/test_13_delete_document.py:1` | 3 | no | `test_deleting_a_document_removes_only_its_own_facts` `sources/hindsight/hindsight-system-tests/tests/test_13_delete_document.py:55`; `test_a_deleted_fact_stops_answering_recalls` `sources/hindsight/hindsight-system-tests/tests/test_13_delete_document.py:65`; `test_deleting_from_one_bank_leaves_the_same_id_alone_in_another` `sources/hindsight/hindsight-system-tests/tests/test_13_delete_document.py:76` |
| `sources/hindsight/hindsight-system-tests/tests/test_15_memory_curation.py:1` | 6 | no | `test_an_invalidated_memory_stops_answering_recalls` `sources/hindsight/hindsight-system-tests/tests/test_15_memory_curation.py:105`; `test_an_invalidated_memory_keeps_its_paper_trail` `sources/hindsight/hindsight-system-tests/tests/test_15_memory_curation.py:116` |
| `sources/hindsight/hindsight-system-tests/tests/test_22_consolidation_safety_nets.py:1` | 8 | no | `test_a_folding_delete_moves_its_evidence_to_the_survivor` `sources/hindsight/hindsight-system-tests/tests/test_22_consolidation_safety_nets.py:188`; `test_a_plain_delete_retires_its_evidence` `sources/hindsight/hindsight-system-tests/tests/test_22_consolidation_safety_nets.py:216` |
| `sources/hindsight/hindsight-system-tests/tests/test_25_observation_lifecycle.py:1` | 5 | no | `test_deleting_the_source_document_takes_the_observation_with_it` `sources/hindsight/hindsight-system-tests/tests/test_25_observation_lifecycle.py:81` |
| `sources/hindsight/hindsight-system-tests/tests/test_30_mental_models.py:1` | 7 | no | `test_deleting_a_model_leaves_the_facts_alone` `sources/hindsight/hindsight-system-tests/tests/test_30_mental_models.py:111` |
| `sources/hindsight/hindsight-system-tests/tests/test_35_knowledge_pages.py:1` | 10 | no | `test_deleting_a_page_leaves_the_facts_alone` `sources/hindsight/hindsight-system-tests/tests/test_35_knowledge_pages.py:107` |
| `sources/hindsight/hindsight-system-tests/tests/test_43_directives.py:1` | 5 | no | `test_a_deleted_directive_stops_being_sent` `sources/hindsight/hindsight-system-tests/tests/test_43_directives.py:91` |
| `sources/hindsight/hindsight-system-tests/tests/test_53_bank_transfer.py:1` | 9 | no | `test_a_deleted_clone_can_be_cloned_again_under_the_same_id` `sources/hindsight/hindsight-system-tests/tests/test_53_bank_transfer.py:214` |
| `sources/hindsight/hindsight-system-tests/tests/test_74_entity_links.py:1` | 5 | no | `test_deleting_a_document_removes_the_entities_only_it_named` `sources/hindsight/hindsight-system-tests/tests/test_74_entity_links.py:100` |
| `sources/hindsight/hindsight-system-tests/tests/test_75_delete_concurrency.py:1` | 2 | no | `test_concurrent_deletes_of_observation_sources` `sources/hindsight/hindsight-system-tests/tests/test_75_delete_concurrency.py:45`; `test_reingest_races_delete_of_another_observation_source` `sources/hindsight/hindsight-system-tests/tests/test_75_delete_concurrency.py:58` |
| `sources/hindsight/hindsight-system-tests/tests/test_80_bank_isolation.py:1` | 6 | no | `test_deleting_a_shared_document_id_leaves_the_neighbour_untouched` `sources/hindsight/hindsight-system-tests/tests/test_80_bank_isolation.py:120`; `test_deleting_a_whole_bank_leaves_its_neighbour_intact` `sources/hindsight/hindsight-system-tests/tests/test_80_bank_isolation.py:132` |
| `sources/hindsight/hindsight-system-tests/tests/test_95_model_call_traces.py:1` | 3 | no | `test_deleting_a_bank_deletes_its_request_history` `sources/hindsight/hindsight-system-tests/tests/test_95_model_call_traces.py:66` |
| `sources/hindsight/hindsight-system-tests/tests/test_95c_document_delete_traces.py:1` | 1 | no | `test_deleting_a_document_deletes_its_request_history` `sources/hindsight/hindsight-system-tests/tests/test_95c_document_delete_traces.py:46` |

## 7. Corrections to B2

Old values below are copied from the triage row (TRIAGE.md), not reconstructed from memory. A refinement narrows an over-broad field; it need not reverse its original value. Counts are distinct field paths, not the number of factual errors.

- `engine_delete.scope`: `"Named deletion/retraction endpoint or method, not a guarantee of physical erasure. Its reach may overlap an agent tool; lack of a native tool is counted separately."` → Single-memory CLI delete is stub; PATCH invalidation is reversible; document/bank deletion works through different bodies. Evidence: `sources/hindsight/hindsight-cli/src/api.rs:468`, `sources/hindsight/hindsight-api-slim/hindsight_api/api/http.py:5339`.
- `context.mechanism.v`: `"Source expressions below identify retrieval, cached/static returns, lifecycle capture, and limits. A hook definition is not a guarantee it emits nonempty context each turn; branches and configuration apply."` → System prompt identifies bank and budget (`sources/hindsight/hindsight-integrations/hermes/__init__.py:1666`). Recall uses a default 4096-token budget per bank, not a single global character limit (`sources/hindsight/hindsight-integrations/hermes/__init__.py:1569`; `sources/hindsight/hindsight-integrations/hermes/__init__.py:1700`). Prefetch can use reflect or recall and returns nothing on disabled/failed branches; assembly and tool-result bodies are quoted below. Multiple banks can enlarge the total. Evidence: `sources/hindsight/hindsight-integrations/hermes/__init__.py:1666`, `sources/hindsight/hindsight-integrations/hermes/__init__.py:1569`, `sources/hindsight/hindsight-integrations/hermes/__init__.py:1700`.
- `sec.telemetry.v`: `"unknown: no emitter identified in the selected adapter files; external dependencies not cleared"` → Direct access `yes`: configured HTTP client or local daemon/database, subject to API/database auth. Profile configuration is `$HERMES_HOME/hindsight/config.json` (`sources/hindsight/hindsight-integrations/hermes/__init__.py:640`), plus profile `.env`; legacy shared config is a fallback. Do not assert an `instance.json` path or mode from this repo: pg0's credential writer is outside this snapshot. Earlier observed database-password access is a historical run finding, not proof of the current dependency's layout. Accountless local mode `yes` with local LLM/embedding services; their configuration and resource needs remain. Telemetry default and configured outbound targets are recorded below; transitive pg0 behavior is unknown. OTLP tracing is off by default in retained configuration; explicit initialization exports to configured endpoint (`sources/hindsight/hindsight-api-slim/hindsight_api/config.py:2185`; `sources/hindsight/hindsight-api-slim/hindsight_api/tracing.py:198`). Evidence: `sources/hindsight/hindsight-integrations/hermes/__init__.py:640`, `sources/hindsight/hindsight-api-slim/hindsight_api/config.py:2185`, `sources/hindsight/hindsight-api-slim/hindsight_api/tracing.py:198`.
- `repro.decls`: `["hindsight-integrations/hermes/plugin.yaml:5", "hindsight-integrations/hermes/pyproject.toml:3", "hindsight-integrations/hermes/pyproject.toml:5", "hindsight-integrations/hermes/pyproject.toml:10", "hindsight-integrations/hermes/pyproject.toml:18", "hindsight-integrations/hermes/pyproject.toml:19", "hindsight-integrations/hermes/pyproject.toml:27", "package.json:14", "package.json:18", "package.json:20", "package.json:35", "package.json:39", "package.json:40", "package.json:41", "package.json:54", "package.json:55", "package.json:56", "package.json:57", "package.json:58", "package.json:59", "package.json:60", "package.json:61", "package.json:62", "package.json:63", "package.json:64", "pyproject.toml:8", "pyproject.toml:13"]` → `range`: `hindsight-client>=0.10.1,<2`, `hindsight-embed>=0.10.1,<2` (`sources/hindsight/hindsight-integrations/hermes/pyproject.toml:18`). The plugin manifest delegates dependency installation to this pyproject. Evidence: `sources/hindsight/hindsight-integrations/hermes/pyproject.toml:18`.
- `tests.count`: `13852` → 166 adapter-related declarations using the stated parser/scope; B2 counted a different, repository-wide inventory. This is a scope refinement, not a count of executed or passing tests. Adapter test files and declaration counts are listed below. There is no adapter forget tool to test. Engine invalidation/document/bank tests are separate; do not use a CLI method name as evidence of a working delete. Archive test asserts the live row and links are removed, the archive row remains, and a derived observation is removed. (`sources/hindsight/hindsight-api-slim/tests/test_memory_curation.py:228`). Evidence: `sources/hindsight/hindsight-api-slim/tests/test_memory_curation.py:228`.

MATRIX counts these 5 explicit field corrections/refinements. Sections 1-6 provide the body and test inventories. Test-name candidates are not passing tests or measured coverage.
