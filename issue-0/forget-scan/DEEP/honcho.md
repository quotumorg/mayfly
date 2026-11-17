# honcho: B3 static correctness review

Snapshot `894b88321872e01702a1aa638ddfdde6ad60f173` (MANIFEST.json row `honcho`). No plugin execution or passing-test claims.

## 1. Architecture

`provider`: in-process manager imports honcho-ai SDK and calls a remote or self-hosted Honcho service. Conclusions, peer card, messages and reasoning context are different objects (`sources/honcho/hermes-plugin-honcho/session_context.py:304`).

Hook/body inventory (no-op/fallback declarations are not active behavior):

| callable | declaration | first executable body line |
|---|---|---|
| `register` | `sources/honcho/hermes-plugin-honcho/__init__.py:1195` | `sources/honcho/hermes-plugin-honcho/__init__.py:1197` |
| `is_available` | `sources/honcho/hermes-plugin-honcho/__init__.py:201` | `sources/honcho/hermes-plugin-honcho/__init__.py:203` |
| `save_config` | `sources/honcho/hermes-plugin-honcho/__init__.py:209` | `sources/honcho/hermes-plugin-honcho/__init__.py:212` |
| `get_config_schema` | `sources/honcho/hermes-plugin-honcho/__init__.py:220` | `sources/honcho/hermes-plugin-honcho/__init__.py:221` |
| `initialize` | `sources/honcho/hermes-plugin-honcho/__init__.py:256` | `sources/honcho/hermes-plugin-honcho/__init__.py:258` |
| `system_prompt_block` | `sources/honcho/hermes-plugin-honcho/__init__.py:489` | `sources/honcho/hermes-plugin-honcho/__init__.py:492` |
| `prefetch` | `sources/honcho/hermes-plugin-honcho/__init__.py:601` | `sources/honcho/hermes-plugin-honcho/__init__.py:605` |
| `queue_prefetch` | `sources/honcho/hermes-plugin-honcho/__init__.py:761` | `sources/honcho/hermes-plugin-honcho/__init__.py:764` |
| `on_turn_start` | `sources/honcho/hermes-plugin-honcho/__init__.py:831` | `sources/honcho/hermes-plugin-honcho/__init__.py:834` |
| `on_session_switch` | `sources/honcho/hermes-plugin-honcho/__init__.py:856` | `sources/honcho/hermes-plugin-honcho/__init__.py:858` |
| `sync_turn` | `sources/honcho/hermes-plugin-honcho/__init__.py:897` | `sources/honcho/hermes-plugin-honcho/__init__.py:905` |
| `on_memory_write` | `sources/honcho/hermes-plugin-honcho/__init__.py:988` | `sources/honcho/hermes-plugin-honcho/__init__.py:993` |
| `on_session_end` | `sources/honcho/hermes-plugin-honcho/__init__.py:1003` | `sources/honcho/hermes-plugin-honcho/__init__.py:1005` |
| `get_tool_schemas` | `sources/honcho/hermes-plugin-honcho/__init__.py:1018` | `sources/honcho/hermes-plugin-honcho/__init__.py:1020` |
| `handle_tool_call` | `sources/honcho/hermes-plugin-honcho/__init__.py:1134` | `sources/honcho/hermes-plugin-honcho/__init__.py:1136` |
| `shutdown` | `sources/honcho/hermes-plugin-honcho/__init__.py:1171` | `sources/honcho/hermes-plugin-honcho/__init__.py:1174` |
| `shutdown` | `sources/honcho/hermes-plugin-honcho/session.py:617` | `sources/honcho/hermes-plugin-honcho/session.py:621` |

## 2. Forget paths and derived copies

`yes`: `honcho_conclude(delete_id=...)` deletes one conclusion for the selected peer through `_manager.delete_conclusion` (`sources/honcho/hermes-plugin-honcho/__init__.py:1119`). Body calls scoped SDK `.delete(conclusion_id)` (`sources/honcho/hermes-plugin-honcho/session_context.py:304`). `honcho_profile(card=...)` replaces a card, not all evidence. Engine conclusion route, document soft deletion, session/workspace deletion and vector cleanup are catalogued below (`sources/honcho/src/routers/conclusions.py:171`; `sources/honcho/src/crud/document.py:1139`). Derived reach `partial`: a conclusion delete is not message/card/session erasure; soft-deleted documents have separate cleanup/reconciliation paths (`sources/honcho/src/crud/document.py:1966`). Workspace conclusion deletion also erases representation-history copies in the same transaction, while live document soft deletion awaits reconciliation (`sources/honcho/src/crud/document.py:1283`; `sources/honcho/src/crud/document.py:1287`). This is stronger than merely hiding a conclusion, but still not erasure of its source messages/card.

Host state.db is independent (`hermes-agent/hermes_state.py:181`). No reviewed adapter forget body performs a general purge of that database or built-in MEMORY.md/USER.md. Prompt/tool-result copies and independent source transcripts remain separate; source operation bodies below establish the scope of each plugin mutation.

<details>
<summary>Deletion, invalidation, archive and reset body ledger</summary>

Public routes, SDK/CLI dispatch and internal storage operations are distinguished by their source. Cache invalidation and management-object deletion are not additional fact-forget tools. A stub is not working deletion. Call-through means the downstream body still determines erasure. HTTP prefixes/selectors must be read with route registration.

**delete_conclusion**, body `sources/honcho/hermes-plugin-honcho/session_context.py:301`; implementation/call-through.

- `sources/honcho/hermes-plugin-honcho/session_context.py:299` `def delete_conclusion(self, session_key: str, conclusion_id: str, peer: str = "user") -> bool:`
- `sources/honcho/hermes-plugin-honcho/session_context.py:304` `"conclusion delete", lambda: self._conclusions_scope(session, target_peer_id).delete(conclusion_id),`

**_delete**, body `sources/honcho/hermes-plugin-honcho/session_context.py:302`; implementation/call-through.

- `sources/honcho/hermes-plugin-honcho/session_context.py:301` `def _delete(session: Any) -> bool:`
- `sources/honcho/hermes-plugin-honcho/session_context.py:304` `"conclusion delete", lambda: self._conclusions_scope(session, target_peer_id).delete(conclusion_id),`

**delete_workspace**, body `sources/honcho/sdks/python/src/honcho/aio.py:583`; implementation/call-through.

- `sources/honcho/sdks/python/src/honcho/aio.py:581` `async def delete_workspace(self, workspace_id: str) -> None:`
- `sources/honcho/sdks/python/src/honcho/aio.py:583` `await self._honcho._async_http_client.delete(routes.workspace(workspace_id))`

**delete**, body `sources/honcho/sdks/python/src/honcho/aio.py:1554`; implementation/call-through.

- `sources/honcho/sdks/python/src/honcho/aio.py:1552` `async def delete(self) -> None:`
- `sources/honcho/sdks/python/src/honcho/aio.py:1555` `await self._session._honcho._async_http_client.delete(`

**delete**, body `sources/honcho/sdks/python/src/honcho/aio.py:2185`; implementation/call-through.

- `sources/honcho/sdks/python/src/honcho/aio.py:2183` `async def delete(self, conclusion_id: str) -> None:`
- `sources/honcho/sdks/python/src/honcho/aio.py:2186` `await self._view._honcho._async_http_client.delete(`

**delete_workspace**, body `sources/honcho/sdks/python/src/honcho/client.py:826`; implementation/call-through.

- `sources/honcho/sdks/python/src/honcho/client.py:811` `@validate_call`
- `sources/honcho/sdks/python/src/honcho/client.py:812` `def delete_workspace(`
- `sources/honcho/sdks/python/src/honcho/client.py:826` `self._http.delete(routes.workspace(workspace_id))`

**delete**, body `sources/honcho/sdks/python/src/honcho/conclusions.py:549`; implementation/call-through.

- `sources/honcho/sdks/python/src/honcho/conclusions.py:542` `def delete(self, conclusion_id: str) -> None:`
- `sources/honcho/sdks/python/src/honcho/conclusions.py:550` `self._honcho._http.delete(routes.conclusion(self.workspace_id, conclusion_id))`

**delete**, body `sources/honcho/sdks/python/src/honcho/http/async_client.py:231`; implementation/call-through.

- `sources/honcho/sdks/python/src/honcho/http/async_client.py:221` `async def delete(`
- `sources/honcho/sdks/python/src/honcho/http/async_client.py:232` `"DELETE", path, body=body, query=query, headers=headers, timeout=timeout`

**delete**, body `sources/honcho/sdks/python/src/honcho/http/client.py:261`; implementation/call-through.

- `sources/honcho/sdks/python/src/honcho/http/client.py:251` `def delete(`
- `sources/honcho/sdks/python/src/honcho/http/client.py:262` `"DELETE", path, body=body, query=query, headers=headers, timeout=timeout`

**delete**, body `sources/honcho/sdks/python/src/honcho/session.py:538`; implementation/call-through.

- `sources/honcho/sdks/python/src/honcho/session.py:525` `def delete(self) -> None:`
- `sources/honcho/sdks/python/src/honcho/session.py:539` `self._honcho._http.delete(routes.session(self.workspace_id, self.id))`

**operation and route call sites**, body `sources/honcho/sdks/typescript/src/client.ts:273`; body evidence.

- `sources/honcho/sdks/typescript/src/client.ts:272` ``
- `sources/honcho/sdks/typescript/src/client.ts:273` `private async _deleteWorkspace(workspaceId: string): Promise<void> {`
- `sources/honcho/sdks/typescript/src/client.ts:274` `await this._http.delete('/${API_VERSION}/workspaces/${workspaceId}')`
- `sources/honcho/sdks/typescript/src/client.ts:275` `}`
- `sources/honcho/sdks/typescript/src/client.ts:276` ``
- `sources/honcho/sdks/typescript/src/client.ts:277` `private async _listWorkspaces(params?: {`
- `sources/honcho/sdks/typescript/src/client.ts:278` `filters?: Record<string, unknown>`
- `sources/honcho/sdks/typescript/src/client.ts:1068` `*/`
- `sources/honcho/sdks/typescript/src/client.ts:1069` `async deleteWorkspace(workspaceId: string): Promise<void> {`
- `sources/honcho/sdks/typescript/src/client.ts:1070` `await this._deleteWorkspace(workspaceId)`
- `sources/honcho/sdks/typescript/src/client.ts:1071` `}`
- `sources/honcho/sdks/typescript/src/client.ts:1072` ``
- `sources/honcho/sdks/typescript/src/client.ts:1073` `/**`
- `sources/honcho/sdks/typescript/src/client.ts:1074` `* Search for messages in the current workspace.`

**operation and route call sites**, body `sources/honcho/sdks/typescript/src/conclusions.ts:258`; body evidence.

- `sources/honcho/sdks/typescript/src/conclusions.ts:257` ``
- `sources/honcho/sdks/typescript/src/conclusions.ts:258` `private async _delete(conclusionId: string): Promise<void> {`
- `sources/honcho/sdks/typescript/src/conclusions.ts:259` `await this._ensureWorkspace()`
- `sources/honcho/sdks/typescript/src/conclusions.ts:260` `await this._http.delete(`
- `sources/honcho/sdks/typescript/src/conclusions.ts:261` `'/${API_VERSION}/workspaces/${this.workspaceId}/conclusions/${conclusionId}'`
- `sources/honcho/sdks/typescript/src/conclusions.ts:262` `)`
- `sources/honcho/sdks/typescript/src/conclusions.ts:263` `}`
- `sources/honcho/sdks/typescript/src/conclusions.ts:264` ``
- `sources/honcho/sdks/typescript/src/conclusions.ts:474` `*/`
- `sources/honcho/sdks/typescript/src/conclusions.ts:475` `async delete(conclusionId: string): Promise<void> {`
- `sources/honcho/sdks/typescript/src/conclusions.ts:476` `await this._delete(conclusionId)`
- `sources/honcho/sdks/typescript/src/conclusions.ts:477` `}`
- `sources/honcho/sdks/typescript/src/conclusions.ts:478` ``
- `sources/honcho/sdks/typescript/src/conclusions.ts:479` `/**`
- `sources/honcho/sdks/typescript/src/conclusions.ts:480` `* Create conclusions in this view.`

**operation and route call sites**, body `sources/honcho/sdks/typescript/src/http/client.ts:208`; body evidence.

- `sources/honcho/sdks/typescript/src/http/client.ts:207` `*/`
- `sources/honcho/sdks/typescript/src/http/client.ts:208` `async delete(path: string, options?: RequestOptions): Promise<void>`
- `sources/honcho/sdks/typescript/src/http/client.ts:209` `async delete<T>(path: string, options?: RequestOptions): Promise<T>`
- `sources/honcho/sdks/typescript/src/http/client.ts:210` `async delete<T = void>(`
- `sources/honcho/sdks/typescript/src/http/client.ts:211` `path: string,`
- `sources/honcho/sdks/typescript/src/http/client.ts:212` `options?: RequestOptions`
- `sources/honcho/sdks/typescript/src/http/client.ts:213` `): Promise<T | void> {`
- `sources/honcho/sdks/typescript/src/http/client.ts:214` `return this.request<T>('DELETE', path, options)`
- `sources/honcho/sdks/typescript/src/http/client.ts:215` `}`
- `sources/honcho/sdks/typescript/src/http/client.ts:216` ``
- `sources/honcho/sdks/typescript/src/http/client.ts:217` `/**`
- `sources/honcho/sdks/typescript/src/http/client.ts:218` `* Make a streaming request that returns a Response object for SSE parsing.`

**operation and route call sites**, body `sources/honcho/sdks/typescript/src/scope.ts:155`; body evidence.

- `sources/honcho/sdks/typescript/src/scope.ts:154` `await this._ensureWorkspace()`
- `sources/honcho/sdks/typescript/src/scope.ts:155` `await this._http.delete('${this._basePath}/sessions/${sessionId}')`
- `sources/honcho/sdks/typescript/src/scope.ts:156` `}`
- `sources/honcho/sdks/typescript/src/scope.ts:157` ``
- `sources/honcho/sdks/typescript/src/scope.ts:158` `private async _listSessions(params?: {`
- `sources/honcho/sdks/typescript/src/scope.ts:159` `page?: number`

**operation and route call sites**, body `sources/honcho/sdks/typescript/src/session.ts:228`; body evidence.

- `sources/honcho/sdks/typescript/src/session.ts:227` ``
- `sources/honcho/sdks/typescript/src/session.ts:228` `private async _delete(): Promise<SessionResponse> {`
- `sources/honcho/sdks/typescript/src/session.ts:229` `await this._ensureWorkspace()`
- `sources/honcho/sdks/typescript/src/session.ts:230` `return this._http.delete<SessionResponse>(`
- `sources/honcho/sdks/typescript/src/session.ts:231` `'/${API_VERSION}/workspaces/${this.workspaceId}/sessions/${this.id}'`
- `sources/honcho/sdks/typescript/src/session.ts:232` `)`
- `sources/honcho/sdks/typescript/src/session.ts:233` `}`
- `sources/honcho/sdks/typescript/src/session.ts:234` ``
- `sources/honcho/sdks/typescript/src/session.ts:312` `await this._ensureWorkspace()`
- `sources/honcho/sdks/typescript/src/session.ts:313` `await this._http.delete(`
- `sources/honcho/sdks/typescript/src/session.ts:314` `'/${API_VERSION}/workspaces/${this.workspaceId}/sessions/${this.id}/peers',`
- `sources/honcho/sdks/typescript/src/session.ts:315` `{ body: peerIds }`
- `sources/honcho/sdks/typescript/src/session.ts:316` `)`
- `sources/honcho/sdks/typescript/src/session.ts:317` `}`
- `sources/honcho/sdks/typescript/src/session.ts:742` `*/`
- `sources/honcho/sdks/typescript/src/session.ts:743` `async delete(): Promise<void> {`
- `sources/honcho/sdks/typescript/src/session.ts:744` `await this._delete()`
- `sources/honcho/sdks/typescript/src/session.ts:745` `}`
- `sources/honcho/sdks/typescript/src/session.ts:746` ``
- `sources/honcho/sdks/typescript/src/session.ts:747` `/**`
- `sources/honcho/sdks/typescript/src/session.ts:748` `* Clone this session.`

**safe_cache_delete**, body `sources/honcho/src/cache/client.py:567`; implementation/call-through.

- `sources/honcho/src/cache/client.py:565` `async def safe_cache_delete(key: str) -> None:`
- `sources/honcho/src/cache/client.py:575` `await cache.delete(key)`

**invalidate_collection_cache**, body `sources/honcho/src/crud/collection.py:297`; implementation/call-through.

- `sources/honcho/src/crud/collection.py:293` `async def invalidate_collection_cache(`
- `sources/honcho/src/crud/collection.py:297` `await safe_cache_delete(collection_cache_key(workspace_name, observer, observed))`

**delete_document**, body `sources/honcho/src/crud/document.py:1165`; implementation/call-through.

- `sources/honcho/src/crud/document.py:1139` `async def delete_document(`
- `sources/honcho/src/crud/document.py:1170` `models.Document.deleted_at.is_(None),`

**delete_documents**, body `sources/honcho/src/crud/document.py:1209`; implementation/call-through.

- `sources/honcho/src/crud/document.py:1188` `async def delete_documents(`
- `sources/honcho/src/crud/document.py:1217` `models.Document.deleted_at.is_(None),`

**delete_document_by_id**, body `sources/honcho/src/crud/document.py:1276`; implementation/call-through.

- `sources/honcho/src/crud/document.py:1254` `async def delete_document_by_id(`
- `sources/honcho/src/crud/document.py:1281` `models.Document.deleted_at.is_(None),`

**cleanup_soft_deleted_documents**, body `sources/honcho/src/crud/document.py:1988`; implementation/call-through.

- `sources/honcho/src/crud/document.py:1966` `async def cleanup_soft_deleted_documents(`
- `sources/honcho/src/crud/document.py:2008` `.where(models.Document.deleted_at.is_not(None))`
- `sources/honcho/src/crud/document.py:2009` `.where(models.Document.deleted_at < cutoff)`
- `sources/honcho/src/crud/document.py:2014` `stmt.order_by(models.Document.deleted_at)`
- `sources/honcho/src/crud/document.py:2049` `# Delete from external vector store (per namespace) and track successful deletions`
- `sources/honcho/src/crud/document.py:2053` `await external_vector_store.delete_many(namespace, [d.id for d in docs])`

**delete_session_history**, body `sources/honcho/src/crud/dream_history.py:404`; implementation/call-through.

- `sources/honcho/src/crud/dream_history.py:396` `async def delete_session_history(`
- `sources/honcho/src/crud/dream_history.py:404` `tenant_id = models.scoped_tenant_id()`

**delete_workspace_history**, body `sources/honcho/src/crud/dream_history.py:472`; implementation/call-through.

- `sources/honcho/src/crud/dream_history.py:467` `async def delete_workspace_history(db: AsyncSession, workspace_name: str) -> None:`
- `sources/honcho/src/crud/dream_history.py:472` `tenant_id = models.scoped_tenant_id()`

**invalidate_scope_peer_cache**, body `sources/honcho/src/crud/scope.py:853`; implementation/call-through.

- `sources/honcho/src/crud/scope.py:851` `async def invalidate_scope_peer_cache(workspace_name: str, scope_peer: str) -> None:`
- `sources/honcho/src/crud/scope.py:853` `await safe_cache_delete(peer_cache_key(workspace_name, scope_peer))`

**_batch_delete_matching**, body `sources/honcho/src/crud/session.py:628`; implementation/call-through.

- `sources/honcho/src/crud/session.py:610` `async def _batch_delete_matching(`
- `sources/honcho/src/crud/session.py:628` `total_deleted = 0`

**delete_session**, body `sources/honcho/src/crud/session.py:680`; implementation/call-through.

- `sources/honcho/src/crud/session.py:653` `async def delete_session(`
- `sources/honcho/src/crud/session.py:711` `# count are needed to rebuild the IDs we delete from the external store`
- `sources/honcho/src/crud/session.py:726` `# Only delete from external vector store if one exists`
- `sources/honcho/src/crud/session.py:734` `# Try to delete from external vector store (best effort)`
- `sources/honcho/src/crud/session.py:739` `await external_vector_store.delete_many(namespace, vector_ids)`
- `sources/honcho/src/crud/session.py:775` `# Only delete from external vector store if one exists`
- `sources/honcho/src/crud/session.py:788` `# Try to delete from external vector store (best effort, per namespace)`
- `sources/honcho/src/crud/session.py:791` `await external_vector_store.delete_many(namespace, doc_ids)`
- `sources/honcho/src/crud/session.py:846` `await db.delete(honcho_session)`
- `sources/honcho/src/crud/session.py:849` `# Invalidate session cache`

**delete_tenant**, body `sources/honcho/src/crud/tenant.py:197`; implementation/call-through.

- `sources/honcho/src/crud/tenant.py:187` `async def delete_tenant(db: AsyncSession, tenant_id: str) -> None:`
- `sources/honcho/src/crud/tenant.py:198` `await db.delete(tenant)`

**delete_webhook_endpoint**, body `sources/honcho/src/crud/webhook.py:109`; implementation/call-through.

- `sources/honcho/src/crud/webhook.py:95` `async def delete_webhook_endpoint(`
- `sources/honcho/src/crud/webhook.py:121` `await db.delete(endpoint)`

**delete_workspace**, body `sources/honcho/src/crud/workspace.py:366`; implementation/call-through.

- `sources/honcho/src/crud/workspace.py:352` `async def delete_workspace(`
- `sources/honcho/src/crud/workspace.py:530` `await db.delete(honcho_workspace)`
- `sources/honcho/src/crud/workspace.py:542` `await external_vector_store.delete_namespace(message_namespace)`
- `sources/honcho/src/crud/workspace.py:564` `await external_vector_store.delete_namespace(doc_namespace)`
- `sources/honcho/src/crud/workspace.py:580` `await cache.delete_match(workspace_pattern)`

**process_deletion**, body `sources/honcho/src/deriver/consumer.py:271`; implementation/call-through.

- `sources/honcho/src/deriver/consumer.py:254` `async def process_deletion(`
- `sources/honcho/src/deriver/consumer.py:295` `result = await crud.delete_session(`
- `sources/honcho/src/deriver/consumer.py:318` `await crud.delete_document_by_id(`
- `sources/honcho/src/deriver/consumer.py:337` `result = await crud.delete_workspace(`

**create_deletion_record**, body `sources/honcho/src/deriver/enqueue.py:881`; implementation/call-through.

- `sources/honcho/src/deriver/enqueue.py:865` `def create_deletion_record(`
- `sources/honcho/src/deriver/enqueue.py:881` `deletion_payload = create_deletion_payload(`

**enqueue_deletion**, body `sources/honcho/src/deriver/enqueue.py:916`; implementation/call-through.

- `sources/honcho/src/deriver/enqueue.py:896` `async def enqueue_deletion(`
- `sources/honcho/src/deriver/enqueue.py:916` `async def _do_enqueue(session: AsyncSession, should_commit: bool) -> None:`

**_cleanup_soft_deleted_documents_pgvector**, body `sources/honcho/src/reconciler/sync_vectors.py:787`; implementation/call-through.

- `sources/honcho/src/reconciler/sync_vectors.py:778` `async def _cleanup_soft_deleted_documents_pgvector(`
- `sources/honcho/src/reconciler/sync_vectors.py:794` `.where(models.Document.deleted_at.is_not(None))`
- `sources/honcho/src/reconciler/sync_vectors.py:795` `.where(models.Document.deleted_at < cutoff)`

**delete_conclusion**, body `sources/honcho/src/routers/conclusions.py:183`; implementation/call-through.

- `sources/honcho/src/routers/conclusions.py:166` `@router.delete(`
- `sources/honcho/src/routers/conclusions.py:167` `"/{conclusion_id}",`
- `sources/honcho/src/routers/conclusions.py:168` `status_code=204,`
- `sources/honcho/src/routers/conclusions.py:169` `response_model=None,`
- `sources/honcho/src/routers/conclusions.py:170` `)`
- `sources/honcho/src/routers/conclusions.py:171` `async def delete_conclusion(`
- `sources/honcho/src/routers/conclusions.py:184` `await crud.delete_document_by_id(`

**remove_session_from_scope**, body `sources/honcho/src/routers/scopes.py:191`; implementation/call-through.

- `sources/honcho/src/routers/scopes.py:171` `@router.delete(`
- `sources/honcho/src/routers/scopes.py:172` `"/{scope_id}/sessions/{session_id}",`
- `sources/honcho/src/routers/scopes.py:173` `status_code=204,`
- `sources/honcho/src/routers/scopes.py:174` `response_model=None,`
- `sources/honcho/src/routers/scopes.py:175` `dependencies=[Depends(require_auth(workspace_name="workspace_id"))],`
- `sources/honcho/src/routers/scopes.py:176` `)`
- `sources/honcho/src/routers/scopes.py:177` `async def remove_session_from_scope(`
- `sources/honcho/src/routers/scopes.py:191` `await crud.remove_session_from_scope(`

**delete_session**, body `sources/honcho/src/routers/sessions.py:464`; implementation/call-through.

- `sources/honcho/src/routers/sessions.py:444` `@router.delete(`
- `sources/honcho/src/routers/sessions.py:445` `"/{session_id}",`
- `sources/honcho/src/routers/sessions.py:446` `status_code=202,`
- `sources/honcho/src/routers/sessions.py:447` `dependencies=[`
- `sources/honcho/src/routers/sessions.py:448` `Depends(require_auth(workspace_name="workspace_id", session_name="session_id"))`
- `sources/honcho/src/routers/sessions.py:449` `],`
- `sources/honcho/src/routers/sessions.py:450` `)`
- `sources/honcho/src/routers/sessions.py:451` `async def delete_session(`
- `sources/honcho/src/routers/sessions.py:464` `try:`

**delete_tenant**, body `sources/honcho/src/routers/tenants.py:134`; implementation/call-through.

- `sources/honcho/src/routers/tenants.py:131` `@router.delete("/{tenant_id}", status_code=204)`
- `sources/honcho/src/routers/tenants.py:132` `async def delete_tenant(tenant_id: Annotated[str, Path()]) -> None:`
- `sources/honcho/src/routers/tenants.py:139` `async with service_db("tenants.delete") as db:`
- `sources/honcho/src/routers/tenants.py:140` `await tenant_crud.delete_tenant(db, tenant_id)`

**delete_webhook_endpoint**, body `sources/honcho/src/routers/webhooks.py:91`; implementation/call-through.

- `sources/honcho/src/routers/webhooks.py:80` `@router.delete("/{endpoint_id}", response_model=None, status_code=204)`
- `sources/honcho/src/routers/webhooks.py:81` `async def delete_webhook_endpoint(`
- `sources/honcho/src/routers/webhooks.py:94` `await crud.delete_webhook_endpoint(db, workspace_id, endpoint_id)`

**delete_workspace**, body `sources/honcho/src/routers/workspaces.py:146`; implementation/call-through.

- `sources/honcho/src/routers/workspaces.py:126` `@router.delete(`
- `sources/honcho/src/routers/workspaces.py:127` `"/{workspace_id}",`
- `sources/honcho/src/routers/workspaces.py:128` `status_code=202,`
- `sources/honcho/src/routers/workspaces.py:129` `dependencies=[Depends(require_auth(workspace_name="workspace_id"))],`
- `sources/honcho/src/routers/workspaces.py:130` `)`
- `sources/honcho/src/routers/workspaces.py:131` `async def delete_workspace(`
- `sources/honcho/src/routers/workspaces.py:146` `await crud.get_workspace(db, workspace_name=workspace_id)`

**reset**, body `sources/honcho/src/telemetry/langfuse_session.py:230`; implementation/call-through.

- `sources/honcho/src/telemetry/langfuse_session.py:228` `def reset() -> None:`
- `sources/honcho/src/telemetry/langfuse_session.py:230` `with _lock:`

**reset**, body `sources/honcho/src/telemetry/trace_session.py:61`; implementation/call-through.

- `sources/honcho/src/telemetry/trace_session.py:59` `def reset() -> None:`
- `sources/honcho/src/telemetry/trace_session.py:61` `with _lock:`

**_handle_delete_observations**, body `sources/honcho/src/utils/agent_tools.py:2140`; implementation/call-through.

- `sources/honcho/src/utils/agent_tools.py:2136` `async def _handle_delete_observations(`
- `sources/honcho/src/utils/agent_tools.py:2149` `async with ctx.db_lock, tracked_db("tool.delete_observations") as db:`
- `sources/honcho/src/utils/agent_tools.py:2150` `deleted = await crud.delete_documents(`

**create_deletion_payload**, body `sources/honcho/src/utils/queue_payload.py:185`; implementation/call-through.

- `sources/honcho/src/utils/queue_payload.py:180` `def create_deletion_payload(`
- `sources/honcho/src/utils/queue_payload.py:185` `return DeletionPayload(`

**delete_many**, body `sources/honcho/src/vector_store/__init__.py:170`; stub.

- `sources/honcho/src/vector_store/__init__.py:161` `@abstractmethod`
- `sources/honcho/src/vector_store/__init__.py:162` `async def delete_many(self, namespace: str, ids: list[str]) -> None:`
- `sources/honcho/src/vector_store/__init__.py:170` `...`

**delete_namespace**, body `sources/honcho/src/vector_store/__init__.py:180`; stub.

- `sources/honcho/src/vector_store/__init__.py:172` `@abstractmethod`
- `sources/honcho/src/vector_store/__init__.py:173` `async def delete_namespace(self, namespace: str) -> None:`
- `sources/honcho/src/vector_store/__init__.py:180` `...`

**delete_many**, body `sources/honcho/src/vector_store/chroma.py:445`; implementation/call-through.

- `sources/honcho/src/vector_store/chroma.py:437` `async def delete_many(self, namespace: str, ids: list[str]) -> None:`
- `sources/honcho/src/vector_store/chroma.py:460` `await asyncio.to_thread(collection.delete, ids=batch)`
- `sources/honcho/src/vector_store/chroma.py:465` `"ChromaDB unavailable for delete from namespace %s: %s",`

**delete_namespace**, body `sources/honcho/src/vector_store/chroma.py:484`; implementation/call-through.

- `sources/honcho/src/vector_store/chroma.py:477` `async def delete_namespace(self, namespace: str) -> None:`
- `sources/honcho/src/vector_store/chroma.py:487` `client.delete_collection, self._collection_name(namespace)`

**delete_many**, body `sources/honcho/src/vector_store/lancedb.py:348`; implementation/call-through.

- `sources/honcho/src/vector_store/lancedb.py:340` `async def delete_many(self, namespace: str, ids: list[str]) -> None:`
- `sources/honcho/src/vector_store/lancedb.py:360` `await table.delete(f"id IN ({in_clause})")`

**delete_namespace**, body `sources/honcho/src/vector_store/lancedb.py:375`; implementation/call-through.

- `sources/honcho/src/vector_store/lancedb.py:368` `async def delete_namespace(self, namespace: str) -> None:`
- `sources/honcho/src/vector_store/lancedb.py:375` `try:`

**delete_many**, body `sources/honcho/src/vector_store/qdrant.py:145`; implementation/call-through.

- `sources/honcho/src/vector_store/qdrant.py:144` `async def delete_many(self, namespace: str, ids: list[str]) -> None:`
- `sources/honcho/src/vector_store/qdrant.py:150` `await self._client.delete(`

**delete_namespace**, body `sources/honcho/src/vector_store/qdrant.py:163`; implementation/call-through.

- `sources/honcho/src/vector_store/qdrant.py:162` `async def delete_namespace(self, namespace: str) -> None:`
- `sources/honcho/src/vector_store/qdrant.py:164` `await self._client.delete_collection(namespace)`

**delete_many**, body `sources/honcho/src/vector_store/turbopuffer.py:296`; implementation/call-through.

- `sources/honcho/src/vector_store/turbopuffer.py:288` `async def delete_many(self, namespace: str, ids: list[str]) -> None:`
- `sources/honcho/src/vector_store/turbopuffer.py:308` `"Turbopuffer unavailable for delete from namespace %s (%s after retries)",`

**delete_namespace**, body `sources/honcho/src/vector_store/turbopuffer.py:328`; implementation/call-through.

- `sources/honcho/src/vector_store/turbopuffer.py:321` `async def delete_namespace(self, namespace: str) -> None:`
- `sources/honcho/src/vector_store/turbopuffer.py:331` `await ns.delete_all()`

</details>

## 3. What reaches the agent

System prompt depends on recall mode; tools mode skips automatic context (`sources/honcho/hermes-plugin-honcho/__init__.py:494`; `sources/honcho/hermes-plugin-honcho/__init__.py:605`). Prefetch joins base context and pending dialectic then applies `_truncate_to_budget`; synchronous recall has a separate path (`sources/honcho/hermes-plugin-honcho/__init__.py:611`; `sources/honcho/hermes-plugin-honcho/__init__.py:645`). Exact budget and response assembly functions below. It does not guarantee nonempty context every turn. context_tokens defaults None (uncapped); when configured, truncation approximates four characters/token and appends an ellipsis suffix, not an exact tokenizer bound (`sources/honcho/hermes-plugin-honcho/client.py:368`; `sources/honcho/hermes-plugin-honcho/__init__.py:754`; `sources/honcho/hermes-plugin-honcho/__init__.py:759`).

Host assembly: `hermes-agent/agent/memory_manager.py:352` joins provider system blocks. `hermes-agent/agent/turn_context.py:1081` requests prefetch for nontrivial turns; `hermes-agent/agent/turn_context.py:120` fences it; `hermes-agent/agent/turn_context.py:136` appends it to API-bound user content. Tool results are a third channel. Optional host spill is at `hermes-agent/agent/memory_manager.py:419`, not a universal provider cap. Built-in memory remains separately controlled (`hermes-agent/agent/system_prompt.py:542`).

<details>
<summary>Context and tool-result assembly quotes</summary>

`_format_first_turn_context`, body `sources/honcho/hermes-plugin-honcho/__init__.py:472`:

- `sources/honcho/hermes-plugin-honcho/__init__.py:480` `suppressed.append(f"{name} ({len(value)}B)")`
- `sources/honcho/hermes-plugin-honcho/__init__.py:482` `parts.append(f"## {header}\n{value}")`
- `sources/honcho/hermes-plugin-honcho/__init__.py:487` `return "\n\n".join(parts)`

`system_prompt_block`, body `sources/honcho/hermes-plugin-honcho/__init__.py:492`:

- `sources/honcho/hermes-plugin-honcho/__init__.py:493` `return ""`
- `sources/honcho/hermes-plugin-honcho/__init__.py:494` `return _PROMPT_HEADERS.get(self._recall_mode, _PROMPT_HEADERS["hybrid"])`

`prefetch`, body `sources/honcho/hermes-plugin-honcho/__init__.py:605`:

- `sources/honcho/hermes-plugin-honcho/__init__.py:606` `return self._log_injection("cron-or-tools-mode")`
- `sources/honcho/hermes-plugin-honcho/__init__.py:611` `payload = "\n\n".join(part for part in (notice, prefetch_sync(self, query)) if part)`
- `sources/honcho/hermes-plugin-honcho/__init__.py:612` `return self._log_injection("injected" if payload else "recall-sync-empty", payload)`
- `sources/honcho/hermes-plugin-honcho/__init__.py:623` `self._init_thread.join(timeout=max(0.0, first_turn_base_deadline - time.monotonic()))`
- `sources/honcho/hermes-plugin-honcho/__init__.py:626` `return self._log_injection("session-not-ready", self._pop_auth_notice() or self._pop_peer_notice())`
- `sources/honcho/hermes-plugin-honcho/__init__.py:631` `return self._log_injection("trivial-prompt", self._truncate_to_budget(ready) if ready else "")`
- `sources/honcho/hermes-plugin-honcho/__init__.py:637` `parts.append(self._fetch_base_context_layer(query, first_turn_base_deadline))`
- `sources/honcho/hermes-plugin-honcho/__init__.py:641` `parts.append(self._consume_pending_dialectic())`
- `sources/honcho/hermes-plugin-honcho/__init__.py:644` `return self._log_injection("fetched-but-empty")`
- `sources/honcho/hermes-plugin-honcho/__init__.py:645` `return self._log_injection("injected", self._truncate_to_budget("\n\n".join(parts)))`

`_truncate_to_budget`, body `sources/honcho/hermes-plugin-honcho/__init__.py:752`:

- `sources/honcho/hermes-plugin-honcho/__init__.py:753` `return text`
- `sources/honcho/hermes-plugin-honcho/__init__.py:754` `budget_chars = self._config.context_tokens * 4`
- `sources/honcho/hermes-plugin-honcho/__init__.py:755` `if len(text) <= budget_chars:`
- `sources/honcho/hermes-plugin-honcho/__init__.py:756` `return text`
- `sources/honcho/hermes-plugin-honcho/__init__.py:757` `truncated = text[:budget_chars]`
- `sources/honcho/hermes-plugin-honcho/__init__.py:759` `return (truncated[:last_space] if last_space > budget_chars * 0.8 else truncated) + " …"`

`queue_prefetch`, body `sources/honcho/hermes-plugin-honcho/__init__.py:764`:



`get_tool_schemas`, body `sources/honcho/hermes-plugin-honcho/__init__.py:1020`:

- `sources/honcho/hermes-plugin-honcho/__init__.py:1021` `return []`
- `sources/honcho/hermes-plugin-honcho/__init__.py:1022` `return list(ALL_TOOL_SCHEMAS)`

`_tool_profile`, body `sources/honcho/hermes-plugin-honcho/__init__.py:1048`:

- `sources/honcho/hermes-plugin-honcho/__init__.py:1051` `return refusal`
- `sources/honcho/hermes-plugin-honcho/__init__.py:1054` `return tool_error("Failed to update peer card.")`
- `sources/honcho/hermes-plugin-honcho/__init__.py:1055` `return json.dumps({"result": f"Peer card updated ({len(result)} facts).", "card": result})`
- `sources/honcho/hermes-plugin-honcho/__init__.py:1057` `return json.dumps({"result": card} if card else self._empty_profile_hint(peer))`

`_tool_search`, body `sources/honcho/hermes-plugin-honcho/__init__.py:1060`:

- `sources/honcho/hermes-plugin-honcho/__init__.py:1061` `return tool_error("Missing required parameter: query")`
- `sources/honcho/hermes-plugin-honcho/__init__.py:1062` `max_tokens = min(int(args.get("max_tokens", 800)), 2000)`
- `sources/honcho/hermes-plugin-honcho/__init__.py:1063` `result = self._manager.search_context(self._session_key, query, max_tokens=max_tokens, peer=args.get("peer", "user"))`
- `sources/honcho/hermes-plugin-honcho/__init__.py:1064` `return json.dumps({"result": result or "No relevant context found."})`

`_tool_reasoning`, body `sources/honcho/hermes-plugin-honcho/__init__.py:1067`:

- `sources/honcho/hermes-plugin-honcho/__init__.py:1070` `return tool_error("Missing required parameter: query")`
- `sources/honcho/hermes-plugin-honcho/__init__.py:1082` `return tool_error(`
- `sources/honcho/hermes-plugin-honcho/__init__.py:1090` `return json.dumps({"result": result or "No result from Honcho."})`

`_tool_context`, body `sources/honcho/hermes-plugin-honcho/__init__.py:1093`:

- `sources/honcho/hermes-plugin-honcho/__init__.py:1095` `return json.dumps({"result": "No context available yet."})`
- `sources/honcho/hermes-plugin-honcho/__init__.py:1100` `parts.append("## Recent messages\n" + "\n".join(f"  [{m['role']}] {m['content'][:200]}" for m in recent[-5:]))`
- `sources/honcho/hermes-plugin-honcho/__init__.py:1101` `return json.dumps({"result": "\n\n".join(parts) or "No context available."})`

`_tool_conclude`, body `sources/honcho/hermes-plugin-honcho/__init__.py:1104`:

- `sources/honcho/hermes-plugin-honcho/__init__.py:1109` `return tool_error("Exactly one of conclusion, delete_id, or list must be provided.")`
- `sources/honcho/hermes-plugin-honcho/__init__.py:1112` `return tool_error("query is only valid when list is true.")`
- `sources/honcho/hermes-plugin-honcho/__init__.py:1115` `return json.dumps({"conclusions": self._manager.list_conclusions(self._session_key, query=query or None, peer=peer)})`
- `sources/honcho/hermes-plugin-honcho/__init__.py:1117` `return refusal`
- `sources/honcho/hermes-plugin-honcho/__init__.py:1120` `return json.dumps({"result": f"Conclusion {delete_id} deleted."})`
- `sources/honcho/hermes-plugin-honcho/__init__.py:1121` `return tool_error(f"Failed to delete conclusion {delete_id}.")`
- `sources/honcho/hermes-plugin-honcho/__init__.py:1123` `return json.dumps({"result": f"Conclusion saved for {peer}: {conclusion}"})`
- `sources/honcho/hermes-plugin-honcho/__init__.py:1124` `return tool_error("Failed to save conclusion.")`

`handle_tool_call`, body `sources/honcho/hermes-plugin-honcho/__init__.py:1136`:

- `sources/honcho/hermes-plugin-honcho/__init__.py:1139` `return tool_error("Honcho is not active (cron context).")`
- `sources/honcho/hermes-plugin-honcho/__init__.py:1142` `return tool_error("Honcho session is still initializing; try again shortly.")`
- `sources/honcho/hermes-plugin-honcho/__init__.py:1145` `return tool_error(f"Honcho memory authentication failed: {self._init_auth_failure}")`
- `sources/honcho/hermes-plugin-honcho/__init__.py:1147` `return tool_error(self._peer_failure_text())`
- `sources/honcho/hermes-plugin-honcho/__init__.py:1148` `return tool_error("Honcho session could not be initialized.")`
- `sources/honcho/hermes-plugin-honcho/__init__.py:1150` `return tool_error("Honcho is not active for this session.")`
- `sources/honcho/hermes-plugin-honcho/__init__.py:1152` `return tool_error(f"Unknown tool: {tool_name}")`
- `sources/honcho/hermes-plugin-honcho/__init__.py:1154` `return handler(self, args)`
- `sources/honcho/hermes-plugin-honcho/__init__.py:1158` `return tool_error(f"Honcho memory authentication failed: {e}")`
- `sources/honcho/hermes-plugin-honcho/__init__.py:1161` `return tool_error(f"Honcho {tool_name} failed: {e}")`

`prefetch_sync`, body `sources/honcho/hermes-plugin-honcho/recall_sync.py:16`:

- `sources/honcho/hermes-plugin-honcho/recall_sync.py:17` `budget = timeout if timeout is not None and math.isfinite(timeout) and timeout > 0 else 5.0`
- `sources/honcho/hermes-plugin-honcho/recall_sync.py:18` `deadline = time.monotonic() + budget`
- `sources/honcho/hermes-plugin-honcho/recall_sync.py:20` `return ""`
- `sources/honcho/hermes-plugin-honcho/recall_sync.py:21` `# No lock wait can extend the request budget. Keep the slot occupied even after`
- `sources/honcho/hermes-plugin-honcho/recall_sync.py:24` `return ""`
- `sources/honcho/hermes-plugin-honcho/recall_sync.py:29` `return ""`
- `sources/honcho/hermes-plugin-honcho/recall_sync.py:35` `provider._init_thread.join(timeout=max(0.0, deadline - time.monotonic()))`
- `sources/honcho/hermes-plugin-honcho/recall_sync.py:37` `return provider._pop_auth_notice() or provider._pop_peer_notice()`
- `sources/honcho/hermes-plugin-honcho/recall_sync.py:41` `return ""`
- `sources/honcho/hermes-plugin-honcho/recall_sync.py:49` `return ""`
- `sources/honcho/hermes-plugin-honcho/recall_sync.py:57` `return (cancelled.is_set() or generation is not provider._recall_generation`
- `sources/honcho/hermes-plugin-honcho/recall_sync.py:89` `results.append(result)`
- `sources/honcho/hermes-plugin-honcho/recall_sync.py:98` `worker.join(timeout=max(0.0, deadline - time.monotonic()))`
- `sources/honcho/hermes-plugin-honcho/recall_sync.py:103` `return ""`
- `sources/honcho/hermes-plugin-honcho/recall_sync.py:114` `return provider._truncate_to_budget("\n\n".join(part for part in parts if part and part.strip()))`
- `sources/honcho/hermes-plugin-honcho/recall_sync.py:116` `# Thread.join may return before a coarse host clock crosses the deadline.`

`prefetch_context`, body `sources/honcho/hermes-plugin-honcho/session.py:635`:



`_fetch_peer_context`, body `sources/honcho/hermes-plugin-honcho/session_context.py:95`:

- `sources/honcho/hermes-plugin-honcho/session_context.py:116` `return {"representation": representation, "card": card}`

`get_prefetch_context`, body `sources/honcho/hermes-plugin-honcho/session_context.py:129`:

- `sources/honcho/hermes-plugin-honcho/session_context.py:131` `return {}`
- `sources/honcho/hermes-plugin-honcho/session_context.py:153` `result["card"] = "\n".join(self._normalize_card(getattr(ctx, "peer_card", None)))`
- `sources/honcho/hermes-plugin-honcho/session_context.py:178` `return result`

`get_session_context`, body `sources/honcho/hermes-plugin-honcho/session_context.py:183`:

- `sources/honcho/hermes-plugin-honcho/session_context.py:185` `return {}`
- `sources/honcho/hermes-plugin-honcho/session_context.py:189` `return self._fetch_peer_context(peer_id, target=peer_id)`
- `sources/honcho/hermes-plugin-honcho/session_context.py:206` `result["card"] = "\n".join(ctx.peer_card)`
- `sources/honcho/hermes-plugin-honcho/session_context.py:209` `{"role": getattr(m, "peer_id", "unknown"), "content": (m.content or "")[:500]}`
- `sources/honcho/hermes-plugin-honcho/session_context.py:212` `return result`
- `sources/honcho/hermes-plugin-honcho/session_context.py:213` `return self._guarded(_fetch, {}, logging.DEBUG, "Session context fetch failed: %s")`

`search_context`, body `sources/honcho/hermes-plugin-honcho/session_context.py:228`:

- `sources/honcho/hermes-plugin-honcho/session_context.py:229` `q = (query or "").strip()[:4000]  # Honcho caps query length for the embedding model.`
- `sources/honcho/hermes-plugin-honcho/session_context.py:231` `return ""`
- `sources/honcho/hermes-plugin-honcho/session_context.py:233` `char_budget = max(200, int(max_tokens) * 4)`
- `sources/honcho/hermes-plugin-honcho/session_context.py:234` `limit = max(3, min(20, char_budget // 300))`
- `sources/honcho/hermes-plugin-honcho/session_context.py:236` `"message search", lambda: self.honcho.search(q, filters={"peer_perspective": peer_id}, limit=limit),`
- `sources/honcho/hermes-plugin-honcho/session_context.py:245` `"peer search", lambda: self._peer_ref(peer_id).search(q, limit=limit),`
- `sources/honcho/hermes-plugin-honcho/session_context.py:249` `return ""`
- `sources/honcho/hermes-plugin-honcho/session_context.py:259` `entry = f"[{who}{f' · {sess}' if sess else ''}] {content[:1200]}"`
- `sources/honcho/hermes-plugin-honcho/session_context.py:261` `remaining = char_budget - len("\n\n".join(lines)) - (2 if lines else 0)`
- `sources/honcho/hermes-plugin-honcho/session_context.py:265` `entry = entry[:remaining].rstrip()`
- `sources/honcho/hermes-plugin-honcho/session_context.py:268` `lines.append(entry)`
- `sources/honcho/hermes-plugin-honcho/session_context.py:271` `return "\n\n".join(lines)`

</details>

## 4. Data access boundaries

Direct access `yes`, with the same Honcho credentials or self-hosted service access. Credentials/configuration are profile-dependent; source ledger lists the local JSON and `.env` paths. Accountless local mode `yes` in self-hosted configuration with local model services; cloud requires an account/key. No conclusion scope check is an OS boundary (`sources/honcho/hermes-plugin-honcho/session_context.py:302`). Telemetry defaults and configurable model endpoints are not equivalent to a network trace.

Boundary model: same OS user, terminal, mounted filesystem and allowed network. Direct reachability means another client can use the same file/endpoint and available credentials, not unauthenticated access or encryption bypass. Hermes setup persists declared secrets through `hermes-agent/hermes_cli/memory_setup.py:321` and `hermes-agent/hermes_cli/config.py:2697`. These are possible configured paths; no live credentials were read.

Host credential-file detail: `get_hermes_home()/.env` is resolved at `hermes-agent/hermes_cli/config.py:473`. New files are secured to 0600 at `hermes-agent/hermes_cli/config.py:506`; the writer preserves an existing mode at `hermes-agent/hermes_cli/config.py:2544` and `:2562`. Therefore 0600 is not an unconditional mode claim for every existing profile.

<details>
<summary>Paths, environment, permissions, egress, telemetry and process-launch evidence</summary>

- `sources/honcho/hermes-plugin-honcho/__init__.py:218` `atomic_json_write(config_path, {**existing, **values}, mode=0o600)`
- `sources/honcho/hermes-plugin-honcho/__init__.py:500` `explicit = os.environ.get("HONCHO_INJECTION_LOG")`
- `sources/honcho/hermes-plugin-honcho/__init__.py:505` `enabled = _as_flag(os.environ.get("HONCHO_LOGGING"), default=False)`
- `sources/honcho/hermes-plugin-honcho/__init__.py:525` `fd = os.open(path, os.O_WRONLY | os.O_CREAT | os.O_APPEND, 0o600)`

- `sources/honcho/hermes-plugin-honcho/attribution.py:5` `starts in Hermes lands in web analytics as its own source instead of "Direct"::`
- `sources/honcho/hermes-plugin-honcho/attribution.py:7` `https://app.honcho.dev/?utm_source=hermes&utm_medium=plugin&utm_content=apikey-prompt`
- `sources/honcho/hermes-plugin-honcho/attribution.py:20` `HONCHO_APP_URL = "https://app.honcho.dev"`
- `sources/honcho/hermes-plugin-honcho/attribution.py:21` `HONCHO_DOCS_URL = "https://docs.honcho.dev/v3/guides/integrations/hermes"`

- `sources/honcho/hermes-plugin-honcho/cli.py:158` `atomic_json_write(path, out, mode=0o600)`
- `sources/honcho/hermes-plugin-honcho/cli.py:194` `return cfg_get(cfg, "hosts", HOST, default={}), bool(cfg.get("apiKey") or os.environ.get("HONCHO_API_KEY"))`
- `sources/honcho/hermes-plugin-honcho/cli.py:202` `key = (block.get("apiKey") or cfg.get("apiKey", "") or (os.environ.get("HONCHO_API_KEY", "") if env else ""))`
- `sources/honcho/hermes-plugin-honcho/cli.py:206` `or (os.environ.get("HONCHO_BASE_URL", "") if env else "") or "").strip()`
- `sources/honcho/hermes-plugin-honcho/cli.py:691` `if new_url := _prompt("Base URL", default=cfg.get("baseUrl") or "http://localhost:8000"):`
- `sources/honcho/hermes-plugin-honcho/cli.py:918` `("peerName", "Your name (user peer)", current_peer or os.getenv("USER", "user")),`
- `sources/honcho/hermes-plugin-honcho/cli.py:1047` `global_path = Path.home() / ".honcho" / "config.json"`
- `sources/honcho/hermes-plugin-honcho/cli.py:1113` `def _state_db_path() -> Path:`
- `sources/honcho/hermes-plugin-honcho/cli.py:1124` `def _seen_gateway_accounts(db_path: Path) -> list[dict]:`
- `sources/honcho/hermes-plugin-honcho/cli.py:1130` `if not db_path.exists():`
- `sources/honcho/hermes-plugin-honcho/cli.py:1145` `with closing(sqlite3.connect(f"file:{db_path}?mode=ro", uri=True)) as conn:`
- `sources/honcho/hermes-plugin-honcho/cli.py:1459` `accounts = _seen_gateway_accounts(_state_db_path())`
- `sources/honcho/hermes-plugin-honcho/cli.py:1736` `candidates = (d / name for name in names for d in (Path(os.getcwd()), Path.home() / ".openclaw"))`

- `sources/honcho/hermes-plugin-honcho/client.py:83` `explicit = (get_secret("HERMES_HONCHO_HOST", "") or "").strip()`
- `sources/honcho/hermes-plugin-honcho/client.py:108` `return Path.home() / ".honcho" / "config.json"`
- `sources/honcho/hermes-plugin-honcho/client.py:266` `return (get_secret("HONCHO_BASE_URL", "") or "").strip() or (get_secret("HONCHO_URL", "") or "").strip() or None`
- `sources/honcho/hermes-plugin-honcho/client.py:272` `api_key = look.pick("apiKey") or get_secret("HONCHO_API_KEY")`
- `sources/honcho/hermes-plugin-honcho/client.py:291` `"timeout": _resolve_optional_float(*look.vals("timeout", "requestTimeout"), os.environ.get("HONCHO_TIMEOUT")),`
- `sources/honcho/hermes-plugin-honcho/client.py:428` `api_key = get_secret("HONCHO_API_KEY")`
- `sources/honcho/hermes-plugin-honcho/client.py:432` `environment=get_secret("HONCHO_ENVIRONMENT", "") or "production",`
- `sources/honcho/hermes-plugin-honcho/client.py:433` `timeout=_resolve_optional_float(os.environ.get("HONCHO_TIMEOUT")),`
- `sources/honcho/hermes-plugin-honcho/client.py:468` `root = subprocess.run(["git", "rev-parse", "--show-toplevel"], capture_output=True, text=True, encoding='utf-8',`
- `sources/honcho/hermes-plugin-honcho/client.py:469` `errors='replace', cwd=cwd, timeout=5, stdin=subprocess.DEVNULL)`
- `sources/honcho/hermes-plugin-honcho/client.py:470` `except (OSError, subprocess.TimeoutExpired):`
- `sources/honcho/hermes-plugin-honcho/client.py:676` `def telemetry_headers() -> dict[str, str]:`
- `sources/honcho/hermes-plugin-honcho/client.py:677` `"""Client-identity headers for telemetry attribution, e.g.::`
- `sources/honcho/hermes-plugin-honcho/client.py:683` `by the API's telemetry middleware; nothing else reads them, so they must never raise.`
- `sources/honcho/hermes-plugin-honcho/client.py:751` `"timeout": timeout, "default_headers": telemetry_headers()}`

- `sources/honcho/hermes-plugin-honcho/client_cache.py:65` `basis = _fingerprint_basis(block, lambda: block.get("apiKey") or raw.get("apiKey") or get_secret("HONCHO_API_KEY") or "")`
- `sources/honcho/hermes-plugin-honcho/client_cache.py:170` `timeout = _resolve_optional_float(os.environ.get("HONCHO_TIMEOUT"))`

- `sources/honcho/hermes-plugin-honcho/config_panel.py:73` `atomic_json_write(path, cfg, mode=0o600)`

- `sources/honcho/hermes-plugin-honcho/config_schema.py:40` `docs_url="https://docs.honcho.dev/v3/guides/integrations/hermes?utm_source=hermes&utm_medium=plugin&utm_content=config-panel",`
- `sources/honcho/hermes-plugin-honcho/config_schema.py:46` `aliases=("base_url",), env_fallbacks=("HONCHO_BASE_URL",), placeholder="https://… (self-hosted)",`

- `sources/honcho/hermes-plugin-honcho/oauth.py:310` `atomic_json_write(path, raw, mode=0o600)`

- `sources/honcho/hermes-plugin-honcho/oauth_flow.py:35` `LOOPBACK_REDIRECT_URI = f"http://{LOOPBACK_HOST}:{LOOPBACK_PORT}/callback"`
- `sources/honcho/hermes-plugin-honcho/oauth_flow.py:40` `_CLOUD_DASHBOARD = "https://app.honcho.dev"`
- `sources/honcho/hermes-plugin-honcho/oauth_flow.py:41` `_CLOUD_TOKEN_URL = "https://api.honcho.dev/oauth/token"`
- `sources/honcho/hermes-plugin-honcho/oauth_flow.py:42` `_LOCAL_DASHBOARD = "http://localhost:3000"`
- `sources/honcho/hermes-plugin-honcho/oauth_flow.py:43` `_LOCAL_TOKEN_URL = "http://localhost:8000/oauth/token"`
- `sources/honcho/hermes-plugin-honcho/oauth_flow.py:52` `return "~/" + str(p.relative_to(Path.home()))`
- `sources/honcho/hermes-plugin-honcho/oauth_flow.py:82` `dashboard = os.environ.get("HONCHO_OAUTH_DASHBOARD", _LOCAL_DASHBOARD if is_local else _CLOUD_DASHBOARD).rstrip("/")`
- `sources/honcho/hermes-plugin-honcho/oauth_flow.py:83` `token_url = os.environ.get("HONCHO_OAUTH_TOKEN_URL", default_token)`
- `sources/honcho/hermes-plugin-honcho/oauth_flow.py:86` `authorize_url=os.environ.get("HONCHO_OAUTH_AUTHORIZE_URL", f"{dashboard}/authorize"),`
- `sources/honcho/hermes-plugin-honcho/oauth_flow.py:88` `client_id=os.environ.get("HONCHO_OAUTH_CLIENT_ID", _DEFAULT_CLIENT_ID),`
- `sources/honcho/hermes-plugin-honcho/oauth_flow.py:89` `scope=os.environ.get("HONCHO_OAUTH_SCOPE", "write"),`
- `sources/honcho/hermes-plugin-honcho/oauth_flow.py:90` `device_authorization_url=os.environ.get("HONCHO_OAUTH_DEVICE_AUTH_URL", default_device),`
- `sources/honcho/hermes-plugin-honcho/oauth_flow.py:220` `redirect_uri = f"http://{LOOPBACK_HOST}:{server.server_address[1]}/callback"`

- `sources/honcho/src/config.py:22` `if not os.getenv("PYTHON_DOTENV_DISABLED"):`
- `sources/honcho/src/config.py:50` `if config_path == "config.toml" and os.getenv("HONCHO_CONFIG_TOML_DISABLED"):`
- `sources/honcho/src/config.py:499` `return os.getenv(env_name)`
- `sources/honcho/src/config.py:670` `"TELEMETRY": "telemetry",`
- `sources/honcho/src/config.py:1368` `class TelemetrySettings(HonchoSettings):`
- `sources/honcho/src/config.py:1369` `"""CloudEvents telemetry settings for analytics.`
- `sources/honcho/src/config.py:1372` `structured events to an analytics backend.`
- `sources/honcho/src/config.py:1375` `model_config = SettingsConfigDict(env_prefix="TELEMETRY_", extra="ignore")  # pyright: ignore`
- `sources/honcho/src/config.py:1639` `LANCEDB_PATH: str = "./lancedb_data"`
- `sources/honcho/src/config.py:1642` `QDRANT_URL: str = "http://localhost:6333"`
- `sources/honcho/src/config.py:1710` `VENDOR_CDN_BASE: str = "https://cdn.jsdelivr.net/npm"`
- `sources/honcho/src/config.py:1741` `"http://localhost",`
- `sources/honcho/src/config.py:1742` `"http://127.0.0.1:8000",`
- `sources/honcho/src/config.py:1743` `"https://api.honcho.dev",`
- `sources/honcho/src/config.py:1780` `TELEMETRY: TelemetrySettings = Field(default_factory=TelemetrySettings)`
- `sources/honcho/src/config.py:1823` `if "NAMESPACE" not in self.TELEMETRY.model_fields_set:`
- `sources/honcho/src/config.py:1824` `self.TELEMETRY.NAMESPACE = self.NAMESPACE`

- `sources/honcho/src/telemetry/__init__.py:2` `Telemetry module for Honcho.`
- `sources/honcho/src/telemetry/__init__.py:4` `This module consolidates all telemetry, metrics, and observability functionality:`
- `sources/honcho/src/telemetry/__init__.py:7` `- CloudEvents: Structured events for analytics (push-based)`
- `sources/honcho/src/telemetry/__init__.py:15` `from src.telemetry.events import emit`
- `sources/honcho/src/telemetry/__init__.py:16` `from src.telemetry.prometheus import (`
- `sources/honcho/src/telemetry/__init__.py:24` `"initialize_telemetry_async",`
- `sources/honcho/src/telemetry/__init__.py:28` `"shutdown_telemetry",`
- `sources/honcho/src/telemetry/__init__.py:32` `async def initialize_telemetry_async() -> None:`
- `sources/honcho/src/telemetry/__init__.py:34` `Initialize async telemetry systems based on configuration.`
- `sources/honcho/src/telemetry/__init__.py:38` `- CloudEvents emitter (if TELEMETRY_ENABLED=true)`
- `sources/honcho/src/telemetry/__init__.py:44` `from src.telemetry.events import initialize_telemetry_events`
- `sources/honcho/src/telemetry/__init__.py:48` `if settings.TELEMETRY.ENABLED:`
- `sources/honcho/src/telemetry/__init__.py:49` `await initialize_telemetry_events()`
- `sources/honcho/src/telemetry/__init__.py:52` `async def shutdown_telemetry() -> None:`
- `sources/honcho/src/telemetry/__init__.py:54` `Shutdown all telemetry systems gracefully.`
- `sources/honcho/src/telemetry/__init__.py:61` `from src.telemetry.events import shutdown_telemetry_events`
- `sources/honcho/src/telemetry/__init__.py:62` `from src.telemetry.logging import flush_langfuse`
- `sources/honcho/src/telemetry/__init__.py:68` `await shutdown_telemetry_events()`

- `sources/honcho/src/telemetry/client_context.py:1` `"""Per-request client identity for telemetry enrichment.`

- `sources/honcho/src/telemetry/emitter.py:4` `This module provides a TelemetryEmitter class that:`
- `sources/honcho/src/telemetry/emitter.py:23` `from src.telemetry.client_context import client_context_body`
- `sources/honcho/src/telemetry/emitter.py:24` `from src.telemetry.tenant import TENANTLESS_CATEGORIES, current_tenant_id`
- `sources/honcho/src/telemetry/emitter.py:27` `from src.telemetry.events.base import BaseEvent`
- `sources/honcho/src/telemetry/emitter.py:38` `reads it straight from 'settings.TELEMETRY.HIGH_VOLUME_SAMPLE_RATE',`
- `sources/honcho/src/telemetry/emitter.py:76` `class TelemetryEmitter:`
- `sources/honcho/src/telemetry/emitter.py:84` `emitter = TelemetryEmitter(`
- `sources/honcho/src/telemetry/emitter.py:85` `endpoint="https://telemetry.example.com/v1/events",`
- `sources/honcho/src/telemetry/emitter.py:126` `"""Initialize the telemetry emitter.`
- `sources/honcho/src/telemetry/emitter.py:165` `logger.info("Telemetry emitter disabled (no endpoint or disabled)")`
- `sources/honcho/src/telemetry/emitter.py:183` `from src.telemetry.prometheus.metrics import prometheus_metrics`
- `sources/honcho/src/telemetry/emitter.py:185` `prometheus_metrics.initialize_telemetry_dropped_metrics(`
- `sources/honcho/src/telemetry/emitter.py:192` `logger.info("Telemetry emitter started, endpoint: %s", self.endpoint)`
- `sources/honcho/src/telemetry/emitter.py:231` `logger.info("Telemetry emitter shutdown complete")`
- `sources/honcho/src/telemetry/emitter.py:247` `from src.telemetry.prometheus.metrics import prometheus_metrics`
- `sources/honcho/src/telemetry/emitter.py:276` `settings.TELEMETRY.HIGH_VOLUME_SAMPLE_RATE,`
- `sources/honcho/src/telemetry/emitter.py:279` `prometheus_metrics.record_telemetry_event_sampled_out(`
- `sources/honcho/src/telemetry/emitter.py:289` `namespace = settings.TELEMETRY.NAMESPACE`
- `sources/honcho/src/telemetry/emitter.py:300` `"dataschema": f"https://honcho.dev/schemas/{event.event_type()}/v{event.schema_version()}",`
- `sources/honcho/src/telemetry/emitter.py:322` `prometheus_metrics.record_telemetry_event_untenanted(`
- `sources/honcho/src/telemetry/emitter.py:344` `prometheus_metrics.record_telemetry_event_dropped(`
- `sources/honcho/src/telemetry/emitter.py:350` `prometheus_metrics.record_telemetry_event_emitted(event_type=event.event_type())`
- `sources/honcho/src/telemetry/emitter.py:351` `prometheus_metrics.set_telemetry_buffer_size(size=buffer_size)`
- `sources/honcho/src/telemetry/emitter.py:362` `"Telemetry buffer at %.0f%% capacity (%d/%d events)",`
- `sources/honcho/src/telemetry/emitter.py:420` `from src.telemetry.prometheus.metrics import prometheus_metrics`
- `sources/honcho/src/telemetry/emitter.py:428` `prometheus_metrics.record_telemetry_event_dropped(`
- `sources/honcho/src/telemetry/emitter.py:526` `_emitter: TelemetryEmitter | None = None`
- `sources/honcho/src/telemetry/emitter.py:529` `def get_emitter() -> TelemetryEmitter | None:`
- `sources/honcho/src/telemetry/emitter.py:530` `"""Get the global telemetry emitter instance."""`
- `sources/honcho/src/telemetry/emitter.py:543` `) -> TelemetryEmitter:`
- `sources/honcho/src/telemetry/emitter.py:544` `"""Initialize and start the global telemetry emitter.`
- `sources/honcho/src/telemetry/emitter.py:561` `_emitter = TelemetryEmitter(`
- `sources/honcho/src/telemetry/emitter.py:576` `"""Shutdown the global telemetry emitter."""`
- `sources/honcho/src/telemetry/emitter.py:586` `_trace_emitter: TelemetryEmitter | None = None`
- `sources/honcho/src/telemetry/emitter.py:589` `def get_trace_emitter() -> TelemetryEmitter | None:`
- `sources/honcho/src/telemetry/emitter.py:603` `) -> TelemetryEmitter:`
- `sources/honcho/src/telemetry/emitter.py:611` `_trace_emitter = TelemetryEmitter(`

- `sources/honcho/src/telemetry/events/__init__.py:2` `CloudEvents telemetry event system for Honcho.`
- `sources/honcho/src/telemetry/events/__init__.py:5` `- emit(): Queue an event for emission to the telemetry backend`
- `sources/honcho/src/telemetry/events/__init__.py:43` `from src.telemetry.events import emit, RepresentationCompletedEvent`
- `sources/honcho/src/telemetry/events/__init__.py:64` `from src.telemetry.events.agent import (`
- `sources/honcho/src/telemetry/events/__init__.py:72` `from src.telemetry.events.api import (`
- `sources/honcho/src/telemetry/events/__init__.py:77` `from src.telemetry.events.base import BaseEvent, generate_event_id`
- `sources/honcho/src/telemetry/events/__init__.py:78` `from src.telemetry.events.deletion import DeletionCompletedEvent`
- `sources/honcho/src/telemetry/events/__init__.py:79` `from src.telemetry.events.dialectic import DialecticCompletedEvent`
- `sources/honcho/src/telemetry/events/__init__.py:80` `from src.telemetry.events.dream import (`
- `sources/honcho/src/telemetry/events/__init__.py:84` `from src.telemetry.events.llm import (`
- `sources/honcho/src/telemetry/events/__init__.py:90` `from src.telemetry.events.reconciliation import (`
- `sources/honcho/src/telemetry/events/__init__.py:95` `from src.telemetry.events.representation import RepresentationCompletedEvent`
- `sources/honcho/src/telemetry/events/__init__.py:96` `from src.telemetry.events.resource import (`
- `sources/honcho/src/telemetry/events/__init__.py:102` `from src.telemetry.events.trace import (`
- `sources/honcho/src/telemetry/events/__init__.py:154` `"initialize_telemetry_events",`
- `sources/honcho/src/telemetry/events/__init__.py:155` `"shutdown_telemetry_events",`
- `sources/honcho/src/telemetry/events/__init__.py:230` `"""Queue an event for emission to the telemetry backend.`
- `sources/honcho/src/telemetry/events/__init__.py:232` `This is the main entry point for emitting telemetry events.`
- `sources/honcho/src/telemetry/events/__init__.py:235` `If telemetry is disabled or not initialized, this is a no-op.`
- `sources/honcho/src/telemetry/events/__init__.py:236` `Telemetry failures are caught and logged to Sentry to avoid blocking operations.`
- `sources/honcho/src/telemetry/events/__init__.py:242` `from src.telemetry.events import emit`
- `sources/honcho/src/telemetry/events/__init__.py:250` `from src.telemetry.emitter import get_emitter`
- `sources/honcho/src/telemetry/events/__init__.py:254` `logger.debug("Telemetry emitter not initialized, dropping event")`
- `sources/honcho/src/telemetry/events/__init__.py:264` `"Failed to emit telemetry event %s: %s",`
- `sources/honcho/src/telemetry/events/__init__.py:275` `Best-effort , swallows failures so telemetry never breaks the LLM path.`
- `sources/honcho/src/telemetry/events/__init__.py:278` `from src.telemetry.emitter import get_trace_emitter`
- `sources/honcho/src/telemetry/events/__init__.py:285` `except Exception:  # pragma: no cover - best-effort telemetry`
- `sources/honcho/src/telemetry/events/__init__.py:291` `async def initialize_telemetry_events() -> None:`
- `sources/honcho/src/telemetry/events/__init__.py:292` `"""Initialize the telemetry events system based on configuration.`
- `sources/honcho/src/telemetry/events/__init__.py:297` `This is typically called from initialize_telemetry() in the main`
- `sources/honcho/src/telemetry/events/__init__.py:298` `telemetry module.`
- `sources/honcho/src/telemetry/events/__init__.py:301` `from src.telemetry.emitter import initialize_emitter`
- `sources/honcho/src/telemetry/events/__init__.py:303` `if not settings.TELEMETRY.ENABLED:`
- `sources/honcho/src/telemetry/events/__init__.py:304` `logger.info("CloudEvents telemetry disabled")`
- `sources/honcho/src/telemetry/events/__init__.py:313` `from src.telemetry.langfuse_exporter import LangfuseExporter`
- `sources/honcho/src/telemetry/events/__init__.py:319` `endpoint=settings.TELEMETRY.ENDPOINT,`
- `sources/honcho/src/telemetry/events/__init__.py:320` `headers=settings.TELEMETRY.HEADERS,`
- `sources/honcho/src/telemetry/events/__init__.py:321` `batch_size=settings.TELEMETRY.BATCH_SIZE,`
- `sources/honcho/src/telemetry/events/__init__.py:322` `flush_interval_seconds=settings.TELEMETRY.FLUSH_INTERVAL_SECONDS,`
- `sources/honcho/src/telemetry/events/__init__.py:323` `flush_threshold=settings.TELEMETRY.FLUSH_THRESHOLD,`
- `sources/honcho/src/telemetry/events/__init__.py:324` `max_retries=settings.TELEMETRY.MAX_RETRIES,`
- `sources/honcho/src/telemetry/events/__init__.py:325` `max_buffer_size=settings.TELEMETRY.MAX_BUFFER_SIZE,`
- `sources/honcho/src/telemetry/events/__init__.py:330` `"CloudEvents telemetry initialized, endpoint: %s", settings.TELEMETRY.ENDPOINT`
- `sources/honcho/src/telemetry/events/__init__.py:334` `if settings.TELEMETRY.TRACE_PAYLOADS_ENABLED:`
- `sources/honcho/src/telemetry/events/__init__.py:336` `from src.telemetry.emitter import initialize_trace_emitter`
- `sources/honcho/src/telemetry/events/__init__.py:337` `from src.telemetry.trace_exporter import TraceExporter`
- `sources/honcho/src/telemetry/events/__init__.py:340` `endpoint=settings.TELEMETRY.ENDPOINT,`
- `sources/honcho/src/telemetry/events/__init__.py:341` `headers=settings.TELEMETRY.HEADERS,`
- `sources/honcho/src/telemetry/events/__init__.py:342` `batch_size=settings.TELEMETRY.BATCH_SIZE,`
- `sources/honcho/src/telemetry/events/__init__.py:343` `flush_interval_seconds=settings.TELEMETRY.FLUSH_INTERVAL_SECONDS,`
- `sources/honcho/src/telemetry/events/__init__.py:344` `flush_threshold=settings.TELEMETRY.FLUSH_THRESHOLD,`
- `sources/honcho/src/telemetry/events/__init__.py:345` `max_retries=settings.TELEMETRY.MAX_RETRIES,`
- `sources/honcho/src/telemetry/events/__init__.py:346` `max_buffer_size=settings.TELEMETRY.MAX_BUFFER_SIZE,`
- `sources/honcho/src/telemetry/events/__init__.py:351` `"Payload tracing initialized, endpoint: %s", settings.TELEMETRY.ENDPOINT`
- `sources/honcho/src/telemetry/events/__init__.py:355` `async def shutdown_telemetry_events() -> None:`
- `sources/honcho/src/telemetry/events/__init__.py:356` `"""Shutdown the telemetry events system.`
- `sources/honcho/src/telemetry/events/__init__.py:364` `from src.telemetry import langfuse_session, trace_session`
- `sources/honcho/src/telemetry/events/__init__.py:365` `from src.telemetry.emitter import shutdown_emitter, shutdown_trace_emitter`
- `sources/honcho/src/telemetry/events/__init__.py:373` `logger.info("CloudEvents telemetry shutdown complete")`

- `sources/honcho/src/telemetry/events/agent.py:2` `Agent events for Honcho telemetry.`
- `sources/honcho/src/telemetry/events/agent.py:6` `correlate with parent events via run_id for detailed analytics.`
- `sources/honcho/src/telemetry/events/agent.py:20` `from src.telemetry.events.base import BaseEvent, CustomInstructionsFields`
- `sources/honcho/src/telemetry/events/agent.py:82` `Tracks conclusion creation with level breakdown for analytics.`
- `sources/honcho/src/telemetry/events/agent.py:121` `Tracks conclusion deletion for memory consolidation analytics.`
- `sources/honcho/src/telemetry/events/agent.py:234` `"'provider_input_tokens' would only churn analytics queries."`
- `sources/honcho/src/telemetry/events/agent.py:277` `about specific tools, with a lightweight per-call telemetry record that`
- `sources/honcho/src/telemetry/events/agent.py:301` `description="Provider-supplied tool call id (e.g. Anthropic's toolu_*) when available; lets analytics cross-reference provider logs",`

- `sources/honcho/src/telemetry/events/api.py:2` `API events for Honcho telemetry.`
- `sources/honcho/src/telemetry/events/api.py:11` `from src.telemetry.events.base import BaseEvent`

- `sources/honcho/src/telemetry/events/base.py:2` `Base event classes and idempotency key generation for CloudEvents telemetry.`
- `sources/honcho/src/telemetry/events/base.py:5` `- BaseEvent: Abstract base class for all Honcho telemetry events`
- `sources/honcho/src/telemetry/events/base.py:50` `"""Base class for all Honcho telemetry events.`
- `sources/honcho/src/telemetry/events/base.py:128` `"""Custom-instruction telemetry shared by the agent completion events."""`

- `sources/honcho/src/telemetry/events/deletion.py:2` `Deletion events for Honcho telemetry.`
- `sources/honcho/src/telemetry/events/deletion.py:12` `from src.telemetry.events.base import BaseEvent`

- `sources/honcho/src/telemetry/events/dialectic.py:2` `Dialectic events for Honcho telemetry.`
- `sources/honcho/src/telemetry/events/dialectic.py:13` `from src.telemetry.events.base import BaseEvent, CustomInstructionsFields`
- `sources/honcho/src/telemetry/events/dialectic.py:24` `for detailed analytics.`

- `sources/honcho/src/telemetry/events/dream.py:2` `Dream events for Honcho telemetry.`
- `sources/honcho/src/telemetry/events/dream.py:6` `with run_id correlation for analytics queries.`
- `sources/honcho/src/telemetry/events/dream.py:13` `from src.telemetry.events.base import BaseEvent, CustomInstructionsFields`
- `sources/honcho/src/telemetry/events/dream.py:93` `"to preserve the two-gate scheduler semantics in analytics."`

- `sources/honcho/src/telemetry/events/llm.py:1` `"""LLM-call events for Honcho telemetry.`
- `sources/honcho/src/telemetry/events/llm.py:10` `'settings.TELEMETRY.HIGH_VOLUME_SAMPLE_RATE' sampler so per-iteration emission`
- `sources/honcho/src/telemetry/events/llm.py:22` `from src.telemetry.events.base import BaseEvent`
- `sources/honcho/src/telemetry/events/llm.py:30` `first , keeps the analytics taxonomy stable.`
- `sources/honcho/src/telemetry/events/llm.py:71` `description="Parent category for analytics joins: 'representation' | 'dialectic' | 'dream' | 'summary'",`
- `sources/honcho/src/telemetry/events/llm.py:144` `description="1-indexed iteration within an agentic tool loop. Passed explicitly via LLMTelemetryContext , NOT read from set_current_iteration (that fires after the LLM call)",`
- `sources/honcho/src/telemetry/events/llm.py:159` `requires adding a value here first , keeps the analytics taxonomy stable`
- `sources/honcho/src/telemetry/events/llm.py:211` `description="Parent category for analytics joins (e.g. 'dialectic', 'representation')",`

- `sources/honcho/src/telemetry/events/reconciliation.py:2` `Reconciliation events for Honcho telemetry.`
- `sources/honcho/src/telemetry/events/reconciliation.py:14` `from src.telemetry.events.base import BaseEvent`

- `sources/honcho/src/telemetry/events/representation.py:2` `Representation events for Honcho telemetry.`
- `sources/honcho/src/telemetry/events/representation.py:13` `from src.telemetry.events.base import BaseEvent, CustomInstructionsFields`

- `sources/honcho/src/telemetry/events/resource.py:2` `Resource creation events for Honcho telemetry.`
- `sources/honcho/src/telemetry/events/resource.py:14` `from src.telemetry.events.base import BaseEvent`

- `sources/honcho/src/telemetry/events/trace.py:27` `from src.telemetry.events.base import BaseEvent`

- `sources/honcho/src/telemetry/langfuse_exporter.py:29` `Best-effort throughout: every export is wrapped so telemetry can never break the`
- `sources/honcho/src/telemetry/langfuse_exporter.py:42` `from src.telemetry import langfuse_session`
- `sources/honcho/src/telemetry/langfuse_exporter.py:43` `from src.telemetry.tenant import current_tenant_id`
- `sources/honcho/src/telemetry/langfuse_exporter.py:62` `except Exception:  # pragma: no cover - best-effort telemetry`
- `sources/honcho/src/telemetry/langfuse_exporter.py:70` `except Exception:  # pragma: no cover - best-effort telemetry`
- `sources/honcho/src/telemetry/langfuse_exporter.py:78` `except Exception:  # pragma: no cover - best-effort telemetry`

- `sources/honcho/src/telemetry/langfuse_session.py:224` `except Exception:  # pragma: no cover - best-effort telemetry`

- `sources/honcho/src/telemetry/logging.py:18` `from src.telemetry.metrics_collector import append_metrics_to_file`

- `sources/honcho/src/telemetry/prometheus/__init__.py:2` `Prometheus telemetry module.`
- `sources/honcho/src/telemetry/prometheus/__init__.py:10` `from src.telemetry.prometheus.metrics import (`

- `sources/honcho/src/telemetry/prometheus/metrics.py:25` `from src.telemetry.tenant import TENANTLESS_CATEGORIES, current_tenant_id`
- `sources/honcho/src/telemetry/prometheus/metrics.py:201` `telemetry_events_emitted_counter = NamespacedCounter(`
- `sources/honcho/src/telemetry/prometheus/metrics.py:202` `"telemetry_events_emitted",`
- `sources/honcho/src/telemetry/prometheus/metrics.py:207` `telemetry_events_sampled_out_counter = NamespacedCounter(`
- `sources/honcho/src/telemetry/prometheus/metrics.py:208` `"telemetry_events_sampled_out",`
- `sources/honcho/src/telemetry/prometheus/metrics.py:213` `telemetry_events_dropped_counter = NamespacedCounter(`
- `sources/honcho/src/telemetry/prometheus/metrics.py:214` `"telemetry_events_dropped",`
- `sources/honcho/src/telemetry/prometheus/metrics.py:224` `telemetry_events_untenanted_counter = NamespacedCounter(`
- `sources/honcho/src/telemetry/prometheus/metrics.py:225` `"telemetry_events_untenanted",`
- `sources/honcho/src/telemetry/prometheus/metrics.py:230` `telemetry_buffer_size_gauge = NamespacedGauge(`
- `sources/honcho/src/telemetry/prometheus/metrics.py:231` `"telemetry_buffer_size",`
- `sources/honcho/src/telemetry/prometheus/metrics.py:592` `def record_telemetry_event_emitted(self, *, event_type: str) -> None:`
- `sources/honcho/src/telemetry/prometheus/metrics.py:594` `telemetry_events_emitted_counter.labels(type=event_type).inc()`
- `sources/honcho/src/telemetry/prometheus/metrics.py:596` `self._handle_metric_error("record_telemetry_event_emitted", e)`
- `sources/honcho/src/telemetry/prometheus/metrics.py:598` `def record_telemetry_event_sampled_out(self, *, event_type: str) -> None:`
- `sources/honcho/src/telemetry/prometheus/metrics.py:600` `telemetry_events_sampled_out_counter.labels(type=event_type).inc()`
- `sources/honcho/src/telemetry/prometheus/metrics.py:602` `self._handle_metric_error("record_telemetry_event_sampled_out", e)`
- `sources/honcho/src/telemetry/prometheus/metrics.py:604` `def record_telemetry_event_dropped(self, *, reason: str) -> None:`
- `sources/honcho/src/telemetry/prometheus/metrics.py:607` `telemetry_events_dropped_counter.labels(reason=reason).inc()`
- `sources/honcho/src/telemetry/prometheus/metrics.py:609` `self._handle_metric_error("record_telemetry_event_dropped", e)`
- `sources/honcho/src/telemetry/prometheus/metrics.py:611` `def record_telemetry_event_untenanted(self, *, event_type: str) -> None:`
- `sources/honcho/src/telemetry/prometheus/metrics.py:613` `telemetry_events_untenanted_counter.labels(type=event_type).inc()`
- `sources/honcho/src/telemetry/prometheus/metrics.py:615` `self._handle_metric_error("record_telemetry_event_untenanted", e)`
- `sources/honcho/src/telemetry/prometheus/metrics.py:630` `def initialize_telemetry_dropped_metrics(self, *, reasons: list[str]) -> None:`
- `sources/honcho/src/telemetry/prometheus/metrics.py:631` `"""Pre-create telemetry_events_dropped ''(namespace, reason)'' children at 0.`
- `sources/honcho/src/telemetry/prometheus/metrics.py:651` `self._touch(telemetry_events_dropped_counter, reason=reason)`
- `sources/honcho/src/telemetry/prometheus/metrics.py:707` `from src.telemetry.events import ALL_EVENT_TYPES, HIGH_VOLUME_EVENT_TYPES`
- `sources/honcho/src/telemetry/prometheus/metrics.py:716` `self._touch(telemetry_events_emitted_counter, type=event_type)`
- `sources/honcho/src/telemetry/prometheus/metrics.py:718` `self._touch(telemetry_events_sampled_out_counter, type=event_type)`
- `sources/honcho/src/telemetry/prometheus/metrics.py:726` `from src.telemetry.events.base import BaseEvent`
- `sources/honcho/src/telemetry/prometheus/metrics.py:734` `self._touch(telemetry_events_untenanted_counter, type=event_type_value)`
- `sources/honcho/src/telemetry/prometheus/metrics.py:735` `self.set_telemetry_buffer_size(size=0)`
- `sources/honcho/src/telemetry/prometheus/metrics.py:811` `def set_telemetry_buffer_size(self, *, size: int) -> None:`
- `sources/honcho/src/telemetry/prometheus/metrics.py:813` `telemetry_buffer_size_gauge.labels().set(size)`
- `sources/honcho/src/telemetry/prometheus/metrics.py:815` `self._handle_metric_error("set_telemetry_buffer_size", e)`

- `sources/honcho/src/telemetry/sentry.py:20` `from src.telemetry.tenant import current_tenant_id`

- `sources/honcho/src/telemetry/tenant.py:1` `"""Ambient tenant identity for telemetry surfaces.`
- `sources/honcho/src/telemetry/tenant.py:6` `Telemetry reads it here rather than threading a tenant through every event class and`
- `sources/honcho/src/telemetry/tenant.py:11` `telemetry stays byte-identical to pre-tenancy output, where the ''namespace'' (the`

- `sources/honcho/src/telemetry/trace_exporter.py:4` `'TELEMETRY.TRACE_PAYLOADS_ENABLED' is on. For each captured call it emits:`
- `sources/honcho/src/telemetry/trace_exporter.py:28` `from src.telemetry import trace_session`
- `sources/honcho/src/telemetry/trace_exporter.py:29` `from src.telemetry.events import emit_trace`
- `sources/honcho/src/telemetry/trace_exporter.py:30` `from src.telemetry.events.trace import LLMCallTracedEvent, TraceContentEvent`
- `sources/honcho/src/telemetry/trace_exporter.py:41` `if not settings.TELEMETRY.TRACE_PAYLOADS_ENABLED:`
- `sources/honcho/src/telemetry/trace_exporter.py:43` `purposes = settings.TELEMETRY.TRACE_PURPOSES`

- `sources/honcho/src/telemetry/trace_session.py:67` `from src.telemetry import prometheus_metrics`
- `sources/honcho/src/telemetry/trace_session.py:69` `prometheus_metrics.record_telemetry_event_dropped(`
- `sources/honcho/src/telemetry/trace_session.py:72` `except Exception:  # pragma: no cover - best-effort telemetry`

</details>

## 5. Reproducibility

Manifest installs unpinned `honcho-ai` (`sources/honcho/hermes-plugin-honcho/plugin.yaml:5`); installing the Python package instead applies range honcho-ai>=2.2.0,<3 (`sources/honcho/hermes-plugin-honcho/pyproject.toml:11`). Self-hosted server and SDK are not guaranteed to match the repository snapshot.

Retained lock/checksum files: none in snapshot. Upstream completeness is unknown: fetch.py:15 excludes .lock files, fetch.py:12 excludes build/vendor directories. Absence here is not absence upstream.

- `sources/honcho/hermes-plugin-honcho/plugin.yaml:2` `version: 1.2.0`
- `sources/honcho/hermes-plugin-honcho/plugin.yaml:4` `pip_dependencies:`

- `sources/honcho/hermes-plugin-honcho/pyproject.toml:3` `version = "1.2.0"`
- `sources/honcho/hermes-plugin-honcho/pyproject.toml:5` `requires-python = ">=3.11"`
- `sources/honcho/hermes-plugin-honcho/pyproject.toml:7` `# Installed into the Hermes venv by 'hermes plugins install' / 'enable' and re-applied after`
- `sources/honcho/hermes-plugin-honcho/pyproject.toml:9` `# and refuses a plugin whose requirements cannot resolve against them.`
- `sources/honcho/hermes-plugin-honcho/pyproject.toml:10` `dependencies = [`
- `sources/honcho/hermes-plugin-honcho/pyproject.toml:11` `"honcho-ai>=2.2.0,<3",`
- `sources/honcho/hermes-plugin-honcho/pyproject.toml:12` `"httpx>=0.27,<1",`
- `sources/honcho/hermes-plugin-honcho/pyproject.toml:15` `# No dependency groups or extras here: Hermes locks this file as a member of a uv workspace whose`
- `sources/honcho/hermes-plugin-honcho/pyproject.toml:17` `# hermes-agent fails every install and update (#1485). The test environment is tests/pyproject.toml.`

## 6. Tests

Adapter refusal and engine conclusion deletion are different coverage; exact files/counts are in the inventory. Previous runtime failures are not reproduced here and are not inferred from static tests. Engine conclusion delete is tested. Adapter a2a tests cover refusal rather than successful deletion; no verified positive adapter deletion assertion is credited here. (`sources/honcho/tests/routes/test_conclusions.py:585`).

Adapter-related declarations: **566**. Python AST test_ functions, fixtures excluded; literal TS/JS it/test, Go Test*, Rust test_* declarations. No parametrization expansion, collection, execution, or coverage measurement. Other rows are engine test candidates selected by mutation names; names alone do not verify forget coverage.

| file | declarations | adapter-related | forget-related declarations |
|---|---:|---|---|
| `sources/honcho/examples/crewai/python/tests/test_storage.py:1` | 7 | no | `test_delete_update_and_discovery_methods` `sources/honcho/examples/crewai/python/tests/test_storage.py:151` |
| `sources/honcho/hermes-plugin-honcho/tests/conftest.py:1` | 0 | yes | none |
| `sources/honcho/hermes-plugin-honcho/tests/pyproject.toml:1` | 0 | yes | none |
| `sources/honcho/hermes-plugin-honcho/tests/test_a2a_sessions.py:1` | 13 | yes | `test_conclude_and_delete_are_refused` `sources/honcho/hermes-plugin-honcho/tests/test_a2a_sessions.py:115` |
| `sources/honcho/hermes-plugin-honcho/tests/test_async_memory.py:1` | 49 | yes | none |
| `sources/honcho/hermes-plugin-honcho/tests/test_attribution.py:1` | 9 | yes | none |
| `sources/honcho/hermes-plugin-honcho/tests/test_auth_recovery.py:1` | 68 | yes | none |
| `sources/honcho/hermes-plugin-honcho/tests/test_cli.py:1` | 40 | yes | none |
| `sources/honcho/hermes-plugin-honcho/tests/test_client.py:1` | 54 | yes | none |
| `sources/honcho/hermes-plugin-honcho/tests/test_client_identity_isolation.py:1` | 10 | yes | none |
| `sources/honcho/hermes-plugin-honcho/tests/test_empty_profile_hint.py:1` | 3 | yes | none |
| `sources/honcho/hermes-plugin-honcho/tests/test_honcho_cli_peers.py:1` | 5 | yes | none |
| `sources/honcho/hermes-plugin-honcho/tests/test_honcho_client_concurrency.py:1` | 3 | yes | none |
| `sources/honcho/hermes-plugin-honcho/tests/test_honcho_client_config.py:1` | 16 | yes | none |
| `sources/honcho/hermes-plugin-honcho/tests/test_honcho_config_schema.py:1` | 6 | yes | none |
| `sources/honcho/hermes-plugin-honcho/tests/test_honcho_session_cache_bounds.py:1` | 23 | yes | none |
| `sources/honcho/hermes-plugin-honcho/tests/test_honcho_session_context.py:1` | 3 | yes | none |
| `sources/honcho/hermes-plugin-honcho/tests/test_honcho_startup_fail_open.py:1` | 19 | yes | none |
| `sources/honcho/hermes-plugin-honcho/tests/test_honcho_summary_sanitize.py:1` | 8 | yes | none |
| `sources/honcho/hermes-plugin-honcho/tests/test_identity_signature.py:1` | 3 | yes | none |
| `sources/honcho/hermes-plugin-honcho/tests/test_network_isolation.py:1` | 5 | yes | none |
| `sources/honcho/hermes-plugin-honcho/tests/test_oauth.py:1` | 16 | yes | none |
| `sources/honcho/hermes-plugin-honcho/tests/test_oauth_flow.py:1` | 18 | yes | none |
| `sources/honcho/hermes-plugin-honcho/tests/test_peers_map.py:1` | 17 | yes | none |
| `sources/honcho/hermes-plugin-honcho/tests/test_pin_peer_name.py:1` | 28 | yes | none |
| `sources/honcho/hermes-plugin-honcho/tests/test_profile_seams.py:1` | 2 | yes | none |
| `sources/honcho/hermes-plugin-honcho/tests/test_query_rewrite.py:1` | 10 | yes | none |
| `sources/honcho/hermes-plugin-honcho/tests/test_recall_sync.py:1` | 4 | yes | none |
| `sources/honcho/hermes-plugin-honcho/tests/test_resume_catch_up.py:1` | 9 | yes | none |
| `sources/honcho/hermes-plugin-honcho/tests/test_save_messages.py:1` | 9 | yes | none |
| `sources/honcho/hermes-plugin-honcho/tests/test_session.py:1` | 69 | yes | none |
| `sources/honcho/hermes-plugin-honcho/tests/test_session_ai_peer_prefix.py:1` | 10 | yes | none |
| `sources/honcho/hermes-plugin-honcho/tests/test_session_routing_integration.py:1` | 5 | yes | none |
| `sources/honcho/hermes-plugin-honcho/tests/test_shutdown.py:1` | 10 | yes | none |
| `sources/honcho/hermes-plugin-honcho/tests/test_telemetry_headers.py:1` | 6 | yes | none |
| `sources/honcho/hermes-plugin-honcho/tests/test_turn_author_peers.py:1` | 16 | yes | none |
| `sources/honcho/honcho-cli/tests/test_commands.py:1` | 36 | no | `test_workspace_delete_aborts_on_no` `sources/honcho/honcho-cli/tests/test_commands.py:152`; `test_session_delete_aborts_on_no` `sources/honcho/honcho-cli/tests/test_commands.py:163` |
| `sources/honcho/kilo-honcho/tests/context-injection.test.js:1` | 19 | no | `session.deleted removes the session` `sources/honcho/kilo-honcho/tests/context-injection.test.js:436` |
| `sources/honcho/sdks/typescript/__tests__/client.test.ts:1` | 28 | no | `deleteWorkspace accepts deletion` `sources/honcho/sdks/typescript/__tests__/client.test.ts:191` |
| `sources/honcho/sdks/typescript/__tests__/conclusions.test.ts:1` | 40 | no | `delete removes conclusion` `sources/honcho/sdks/typescript/__tests__/conclusions.test.ts:482` |
| `sources/honcho/sdks/typescript/__tests__/http-client.test.ts:1` | 69 | no | `delete() uses DELETE method` `sources/honcho/sdks/typescript/__tests__/http-client.test.ts:447` |
| `sources/honcho/sdks/typescript/__tests__/session.test.ts:1` | 47 | no | `delete removes session` `sources/honcho/sdks/typescript/__tests__/session.test.ts:222` |
| `sources/honcho/tests/alembic/test_deleted_at_index_partial.py:1` | 4 | no | `test_full_deleted_at_index_becomes_partial` `sources/honcho/tests/alembic/test_deleted_at_index_partial.py:33`; `test_partial_deleted_at_index_is_kept` `sources/honcho/tests/alembic/test_deleted_at_index_partial.py:48` |
| `sources/honcho/tests/crud/test_conclusion_source_messages.py:1` | 9 | no | `test_citation_edges_follow_message_and_document_deletion` `sources/honcho/tests/crud/test_conclusion_source_messages.py:102` |
| `sources/honcho/tests/crud/test_document.py:1` | 35 | no | `test_query_documents_excludes_soft_deleted` `sources/honcho/tests/crud/test_document.py:151`; `test_delete_document_success` `sources/honcho/tests/crud/test_document.py:884`; `test_delete_document_not_found` `sources/honcho/tests/crud/test_document.py:929` |
| `sources/honcho/tests/crud/test_dream_history.py:1` | 19 | no | `test_deleting_the_session_deletes_its_removed_copies` `sources/honcho/tests/crud/test_dream_history.py:321`; `test_a_conclusion_deleted_through_the_api_is_unavailable` `sources/honcho/tests/crud/test_dream_history.py:346`; `test_deleting_a_restored_conclusion_erases_the_copy_a_dream_kept` `sources/honcho/tests/crud/test_dream_history.py:378`; `test_deleting_the_workspace_deletes_its_history` `sources/honcho/tests/crud/test_dream_history.py:422`; `test_prune_deletes_old_runs_and_interrupts_stale_ones` `sources/honcho/tests/crud/test_dream_history.py:452` |
| `sources/honcho/tests/crud/test_dream_history_lock_order.py:1` | 1 | no | `test_session_and_workspace_deletion_do_not_deadlock` `sources/honcho/tests/crud/test_dream_history_lock_order.py:17` |
| `sources/honcho/tests/crud/test_get_or_create_retry_invalidation.py:1` | 3 | no | `test_peer_retry_still_invalidates_mutated_peer` `sources/honcho/tests/crud/test_get_or_create_retry_invalidation.py:76`; `test_scope_retry_still_invalidates_mutated_scope` `sources/honcho/tests/crud/test_get_or_create_retry_invalidation.py:123`; `test_peer_no_race_does_not_invalidate_unchanged_peer` `sources/honcho/tests/crud/test_get_or_create_retry_invalidation.py:179` |
| `sources/honcho/tests/crud/test_representation_manager.py:1` | 29 | no | `test_query_documents_recent_excludes_soft_deleted` `sources/honcho/tests/crud/test_representation_manager.py:96`; `test_query_documents_most_derived_excludes_soft_deleted` `sources/honcho/tests/crud/test_representation_manager.py:178` |
| `sources/honcho/tests/crud/test_session.py:1` | 19 | no | `test_queue_items_scoped_to_deleting_tenant` `sources/honcho/tests/crud/test_session.py:753`; `test_delete_session_targets_only_the_deleting_tenants_namespace` `sources/honcho/tests/crud/test_session.py:832` |
| `sources/honcho/tests/crud/test_workspace.py:1` | 22 | no | `test_delete_workspace_not_found` `sources/honcho/tests/crud/test_workspace.py:50`; `test_delete_workspace_cascade_peers` `sources/honcho/tests/crud/test_workspace.py:56`; `test_delete_workspace_succeeds_with_active_sessions` `sources/honcho/tests/crud/test_workspace.py:111`; `test_delete_workspace_cascade_messages` `sources/honcho/tests/crud/test_workspace.py:135`; `test_delete_workspace_cascade_collections` `sources/honcho/tests/crud/test_workspace.py:189`; `test_delete_workspace_cascade_documents` `sources/honcho/tests/crud/test_workspace.py:223`; `test_delete_workspace_cascade_session_peers` `sources/honcho/tests/crud/test_workspace.py:279`; `test_delete_workspace_cascade_webhooks` `sources/honcho/tests/crud/test_workspace.py:326`; `test_delete_workspace_cascade_queue_items` `sources/honcho/tests/crud/test_workspace.py:359`; `test_delete_workspace_cascade_active_queue_sessions` `sources/honcho/tests/crud/test_workspace.py:404`; `test_delete_workspace_returns_deleted_workspace` `sources/honcho/tests/crud/test_workspace.py:446`; `test_delete_workspace_complex_cascade` `sources/honcho/tests/crud/test_workspace.py:472`; `test_delete_workspace_allows_inactive_sessions` `sources/honcho/tests/crud/test_workspace.py:584`; `test_check_no_active_sessions_passes_after_deletion` `sources/honcho/tests/crud/test_workspace.py:618`; `test_cleanup_scoped_to_deleting_tenant` `sources/honcho/tests/crud/test_workspace.py:663`; `test_queue_items_scoped_to_deleting_tenant` `sources/honcho/tests/crud/test_workspace.py:748`; `test_delete_workspace_targets_only_the_deleting_tenants_namespace` `sources/honcho/tests/crud/test_workspace.py:858` |
| `sources/honcho/tests/deriver/test_queue_item_batch_concurrency.py:1` | 7 | no | `test_the_delete_guard_keeps_a_unit_whose_next_item_arrived_mid_recompute` `sources/honcho/tests/deriver/test_queue_item_batch_concurrency.py:117` |
| `sources/honcho/tests/deriver/test_queue_item_batches.py:1` | 27 | no | `test_deleting_an_unprocessed_item_shrinks_the_row` `sources/honcho/tests/deriver/test_queue_item_batches.py:278`; `test_deleting_the_last_unprocessed_item_removes_the_row` `sources/honcho/tests/deriver/test_queue_item_batches.py:301`; `test_deleting_a_processed_item_leaves_the_row_alone` `sources/honcho/tests/deriver/test_queue_item_batches.py:319`; `test_a_deletion_unit_ignores_the_tokens_of_its_message` `sources/honcho/tests/deriver/test_queue_item_batches.py:376` |
| `sources/honcho/tests/deriver/test_scope_backfill.py:1` | 21 | no | `test_removal_and_session_deletion_do_not_deadlock` `sources/honcho/tests/deriver/test_scope_backfill.py:963` |
| `sources/honcho/tests/dreamer/test_work_list.py:1` | 14 | no | `test_query_ignores_deleted_and_other_levels` `sources/honcho/tests/dreamer/test_work_list.py:446` |
| `sources/honcho/tests/integration/test_pgvector_peer_liveness.py:1` | 9 | no | `test_committed_parent_deletion_is_seen_on_next_read` `sources/honcho/tests/integration/test_pgvector_peer_liveness.py:515` |
| `sources/honcho/tests/integration/test_telemetry.py:1` | 24 | no | `test_conclusions_deleted_event` `sources/honcho/tests/integration/test_telemetry.py:628`; `test_deletion_event` `sources/honcho/tests/integration/test_telemetry.py:719` |
| `sources/honcho/tests/reconciler/test_queue_cleanup.py:1` | 1 | no | `test_cleanup_queue_items_executes_delete_and_commit` `sources/honcho/tests/reconciler/test_queue_cleanup.py:11` |
| `sources/honcho/tests/reconciler/test_sync_vectors_enqueue_gate.py:1` | 6 | no | `test_soft_deleted_document_is_work_only_after_grace` `sources/honcho/tests/reconciler/test_sync_vectors_enqueue_gate.py:73` |
| `sources/honcho/tests/routes/test_conclusions.py:1` | 41 | no | `test_delete_conclusion_success` `sources/honcho/tests/routes/test_conclusions.py:585`; `test_delete_conclusion_not_found` `sources/honcho/tests/routes/test_conclusions.py:644`; `test_get_conclusion_soft_deleted` `sources/honcho/tests/routes/test_conclusions.py:804` |
| `sources/honcho/tests/routes/test_dreams.py:1` | 6 | no | `test_deleting_a_removed_conclusion_through_the_api_erases_its_copy` `sources/honcho/tests/routes/test_dreams.py:183` |
| `sources/honcho/tests/routes/test_scopes.py:1` | 46 | no | `test_session_list_ignores_deleted_scope` `sources/honcho/tests/routes/test_scopes.py:1413` |
| `sources/honcho/tests/routes/test_sessions.py:1` | 46 | no | `test_update_delete_metadata` `sources/honcho/tests/routes/test_sessions.py:384`; `test_delete_session` `sources/honcho/tests/routes/test_sessions.py:431` |
| `sources/honcho/tests/routes/test_tenants.py:1` | 31 | no | `test_delete_empty_tenant` `sources/honcho/tests/routes/test_tenants.py:313`; `test_delete_default_tenant_forbidden` `sources/honcho/tests/routes/test_tenants.py:330`; `test_delete_missing_tenant` `sources/honcho/tests/routes/test_tenants.py:338`; `test_delete_non_empty_tenant_conflict` `sources/honcho/tests/routes/test_tenants.py:347` |
| `sources/honcho/tests/routes/test_webhooks.py:1` | 10 | no | `test_delete_webhook_endpoint` `sources/honcho/tests/routes/test_webhooks.py:99`; `test_delete_webhook_endpoint_not_found` `sources/honcho/tests/routes/test_webhooks.py:129` |
| `sources/honcho/tests/routes/test_workspaces.py:1` | 32 | no | `test_delete_workspace` `sources/honcho/tests/routes/test_workspaces.py:458`; `test_delete_nonexistent_workspace` `sources/honcho/tests/routes/test_workspaces.py:480`; `test_delete_workspace_with_peers` `sources/honcho/tests/routes/test_workspaces.py:491`; `test_delete_workspace_with_sessions` `sources/honcho/tests/routes/test_workspaces.py:516`; `test_delete_workspace_with_messages` `sources/honcho/tests/routes/test_workspaces.py:543`; `test_delete_workspace_with_webhooks` `sources/honcho/tests/routes/test_workspaces.py:593`; `test_delete_workspace_cascade` `sources/honcho/tests/routes/test_workspaces.py:620`; `test_delete_workspace_returns_accepted` `sources/honcho/tests/routes/test_workspaces.py:682`; `test_delete_workspace_blocked_by_sessions_returns_409` `sources/honcho/tests/routes/test_workspaces.py:700`; `test_delete_workspace_after_session_deletion` `sources/honcho/tests/routes/test_workspaces.py:724` |
| `sources/honcho/tests/sdk/test_conclusions.py:1` | 16 | no | `test_observation_create_then_delete` `sources/honcho/tests/sdk/test_conclusions.py:339` |
| `sources/honcho/tests/sdk/test_session.py:1` | 14 | no | `test_session_delete` `sources/honcho/tests/sdk/test_session.py:463` |
| `sources/honcho/tests/telemetry/test_events.py:1` | 65 | no | `test_failed_deletion_with_error` `sources/honcho/tests/telemetry/test_events.py:776` |
| `sources/honcho/tests/test_cache_key_namespace.py:1` | 11 | no | `test_tenant_scope_raises_without_tenant_for_delete` `sources/honcho/tests/test_cache_key_namespace.py:152`; `test_tenant_scope_raises_without_tenant_for_delete_many` `sources/honcho/tests/test_cache_key_namespace.py:163` |
| `sources/honcho/tests/utils/test_agent_tools.py:1` | 93 | no | `test_delete_valid_observation` `sources/honcho/tests/utils/test_agent_tools.py:972`; `test_delete_invalid_id_handled_gracefully` `sources/honcho/tests/utils/test_agent_tools.py:995`; `test_malformed_observation_ids_rejected_without_deleting` `sources/honcho/tests/utils/test_agent_tools.py:1011`; `test_delete_batch_emits_levels_for_successful_only` `sources/honcho/tests/utils/test_agent_tools.py:1041` |
| `sources/honcho/tests/utils/test_retryable_errors.py:1` | 9 | no | `test_connection_invalidated_is_retryable` `sources/honcho/tests/utils/test_retryable_errors.py:68` |
| `sources/honcho/tests/vector_store/test_chroma.py:1` | 28 | no | `test_delete_many_short_circuits_on_empty` `sources/honcho/tests/vector_store/test_chroma.py:367`; `test_delete_many_noop_when_collection_missing` `sources/honcho/tests/vector_store/test_chroma.py:377`; `test_delete_many_raises_vector_store_error_on_transport_failure` `sources/honcho/tests/vector_store/test_chroma.py:386`; `test_delete_namespace_noop_when_collection_missing` `sources/honcho/tests/vector_store/test_chroma.py:398` |
| `sources/honcho/tests/vector_store/test_qdrant.py:1` | 14 | no | `test_delete_many_raises_vector_store_error_on_failure` `sources/honcho/tests/vector_store/test_qdrant.py:225` |
| `sources/honcho/tests/vector_store/test_tenant_namespace.py:1` | 14 | no | `test_a_deleted_tenant_recreated_with_another_key_resolves_the_new_one` `sources/honcho/tests/vector_store/test_tenant_namespace.py:339` |

## 7. Corrections to B2

Old values below are copied from the triage row (TRIAGE.md), not reconstructed from memory. A refinement narrows an over-broad field; it need not reverse its original value. Counts are distinct field paths, not the number of factual errors.

- `engine_delete.e`: `["hermes-plugin-honcho/session_context.py:299"]` → That is an adapter SDK call, not the server body. Server conclusion/document routes and CRUD are separately identified. Evidence: `sources/honcho/hermes-plugin-honcho/session_context.py:304`, `sources/honcho/src/routers/conclusions.py:171`.
- `context.bounds_status`: `"Source expressions include recall and other output bounds. They are not interchangeable: token vs character limits and static block vs tool output must be read at their cited call sites. No single total bound is established unless explicitly stated."` → Default context_tokens=None is uncapped; configured cap is approximate characters plus suffix. Evidence: `sources/honcho/hermes-plugin-honcho/client.py:368`, `sources/honcho/hermes-plugin-honcho/__init__.py:754`.
- `engine_delete.scope`: `"Named deletion/retraction endpoint or method, not a guarantee of physical erasure. Its reach may overlap an agent tool; lack of a native tool is counted separately."` → `yes`: `honcho_conclude(delete_id=...)` deletes one conclusion for the selected peer through `_manager.delete_conclusion` (`sources/honcho/hermes-plugin-honcho/__init__.py:1119`). Body calls scoped SDK `.delete(conclusion_id)` (`sources/honcho/hermes-plugin-honcho/session_context.py:304`). `honcho_profile(card=...)` replaces a card, not all evidence. Engine conclusion route, document soft deletion, session/workspace deletion and vector cleanup are catalogued below (`sources/honcho/src/routers/conclusions.py:171`; `sources/honcho/src/crud/document.py:1139`). Derived reach `partial`: a conclusion delete is not message/card/session erasure; soft-deleted documents have separate cleanup/reconciliation paths (`sources/honcho/src/crud/document.py:1966`). Workspace conclusion deletion also erases representation-history copies in the same transaction, while live document soft deletion awaits reconciliation (`sources/honcho/src/crud/document.py:1283`; `sources/honcho/src/crud/document.py:1287`). This is stronger than merely hiding a conclusion, but still not erasure of its source messages/card. Evidence: `sources/honcho/hermes-plugin-honcho/__init__.py:1119`, `sources/honcho/hermes-plugin-honcho/session_context.py:304`, `sources/honcho/src/routers/conclusions.py:171`, `sources/honcho/src/crud/document.py:1139`, `sources/honcho/src/crud/document.py:1966`, `sources/honcho/src/crud/document.py:1283`, `sources/honcho/src/crud/document.py:1287`.
- `context.mechanism.v`: `"Source expressions below identify retrieval, cached/static returns, lifecycle capture, and limits. A hook definition is not a guarantee it emits nonempty context each turn; branches and configuration apply."` → System prompt depends on recall mode; tools mode skips automatic context (`sources/honcho/hermes-plugin-honcho/__init__.py:494`; `sources/honcho/hermes-plugin-honcho/__init__.py:605`). Prefetch joins base context and pending dialectic then applies `_truncate_to_budget`; synchronous recall has a separate path (`sources/honcho/hermes-plugin-honcho/__init__.py:611`; `sources/honcho/hermes-plugin-honcho/__init__.py:645`). Exact budget and response assembly functions below. It does not guarantee nonempty context every turn. context_tokens defaults None (uncapped); when configured, truncation approximates four characters/token and appends an ellipsis suffix, not an exact tokenizer bound (`sources/honcho/hermes-plugin-honcho/client.py:368`; `sources/honcho/hermes-plugin-honcho/__init__.py:754`; `sources/honcho/hermes-plugin-honcho/__init__.py:759`). Evidence: `sources/honcho/hermes-plugin-honcho/__init__.py:494`, `sources/honcho/hermes-plugin-honcho/__init__.py:605`, `sources/honcho/hermes-plugin-honcho/__init__.py:611`, `sources/honcho/hermes-plugin-honcho/__init__.py:645`, `sources/honcho/hermes-plugin-honcho/client.py:368`, `sources/honcho/hermes-plugin-honcho/__init__.py:754`, `sources/honcho/hermes-plugin-honcho/__init__.py:759`.
- `sec.telemetry.v`: `"inspect listed emitters/configuration"` → Direct access `yes`, with the same Honcho credentials or self-hosted service access. Credentials/configuration are profile-dependent; source ledger lists the local JSON and `.env` paths. Accountless local mode `yes` in self-hosted configuration with local model services; cloud requires an account/key. No conclusion scope check is an OS boundary (`sources/honcho/hermes-plugin-honcho/session_context.py:302`). Telemetry defaults and configurable model endpoints are not equivalent to a network trace. Evidence: `sources/honcho/hermes-plugin-honcho/session_context.py:302`.
- `repro.decls`: `["hermes-plugin-honcho/plugin.yaml:4", "hermes-plugin-honcho/plugin.yaml:5", "hermes-plugin-honcho/plugin.yaml:7", "hermes-plugin-honcho/pyproject.toml:3", "hermes-plugin-honcho/pyproject.toml:5", "hermes-plugin-honcho/pyproject.toml:10", "hermes-plugin-honcho/pyproject.toml:11", "hermes-plugin-honcho/pyproject.toml:12", "pyproject.toml:3", "pyproject.toml:9", "pyproject.toml:10", "pyproject.toml:11", "pyproject.toml:12", "pyproject.toml:13", "pyproject.toml:14", "pyproject.toml:15", "pyproject.toml:16", "pyproject.toml:17", "pyproject.toml:18", "pyproject.toml:19", "pyproject.toml:20", "pyproject.toml:21", "pyproject.toml:22", "pyproject.toml:23", "pyproject.toml:24", "pyproject.toml:25", "pyproject.toml:26", "pyproject.toml:27", "pyproject.toml:28", "pyproject.toml:29", "pyproject.toml:30", "pyproject.toml:31", "pyproject.toml:32", "pyproject.toml:33", "pyproject.toml:34", "pyproject.toml:35", "pyproject.toml:36", "pyproject.toml:37", "pyproject.toml:38", "pyproject.toml:39", "pyproject.toml:41", "pyproject.toml:46", "pyproject.toml:49", "pyproject.toml:50", "pyproject.toml:53", "pyproject.toml:56", "pyproject.toml:60", "pyproject.toml:61", "pyproject.toml:62", "pyproject.toml:63", "pyproject.toml:64", "pyproject.toml:65", "pyproject.toml:66", "pyproject.toml:67", "pyproject.toml:68", "pyproject.toml:69", "pyproject.toml:71", "pyproject.toml:72", "pyproject.toml:73", "pyproject.toml:74", "pyproject.toml:75", "pyproject.toml:76", "pyproject.toml:77", "pyproject.toml:78", "pyproject.toml:82"]` → Manifest installs unpinned `honcho-ai` (`sources/honcho/hermes-plugin-honcho/plugin.yaml:5`); installing the Python package instead applies range honcho-ai>=2.2.0,<3 (`sources/honcho/hermes-plugin-honcho/pyproject.toml:11`). Self-hosted server and SDK are not guaranteed to match the repository snapshot. Evidence: `sources/honcho/hermes-plugin-honcho/plugin.yaml:5`.
- `tests.count`: `4613` → 566 adapter-related declarations using the stated parser/scope; B2 counted a different, repository-wide inventory. This is a scope refinement, not a count of executed or passing tests. Adapter refusal and engine conclusion deletion are different coverage; exact files/counts are in the inventory. Previous runtime failures are not reproduced here and are not inferred from static tests. Engine conclusion delete is tested. Adapter a2a tests cover refusal rather than successful deletion; no verified positive adapter deletion assertion is credited here. (`sources/honcho/tests/routes/test_conclusions.py:585`). Evidence: `sources/honcho/tests/routes/test_conclusions.py:585`.

MATRIX counts these 7 explicit field corrections/refinements. Sections 1-6 provide the body and test inventories. Test-name candidates are not passing tests or measured coverage.
