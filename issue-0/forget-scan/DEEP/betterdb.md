# betterdb: B3 static correctness review

Snapshot `cf54fb5f97afef8b3387ffe91be6272d5da23ea2` (MANIFEST.json row `betterdb`). No plugin execution or passing-test claims.

## 1. Architecture

`provider`: imports MemoryStore in-process, runs async operations on LoopRuntime, and uses Valkey/valkey-search as an external database plus an HTTP embedding function (`sources/betterdb/packages/hermes-memory-py/betterdb_hermes_memory/__init__.py:324`; `sources/betterdb/packages/hermes-memory-py/betterdb_hermes_memory/__init__.py:342`).

Hook/body inventory (no-op/fallback declarations are not active behavior):

| callable | declaration | first executable body line |
|---|---|---|
| `register` | `sources/betterdb/packages/hermes-memory-py/betterdb_hermes_memory/__init__.py:770` | `sources/betterdb/packages/hermes-memory-py/betterdb_hermes_memory/__init__.py:772` |
| `is_available` | `sources/betterdb/packages/hermes-memory-py/betterdb_hermes_memory/__init__.py:231` | `sources/betterdb/packages/hermes-memory-py/betterdb_hermes_memory/__init__.py:235` |
| `get_config_schema` | `sources/betterdb/packages/hermes-memory-py/betterdb_hermes_memory/__init__.py:273` | `sources/betterdb/packages/hermes-memory-py/betterdb_hermes_memory/__init__.py:274` |
| `save_config` | `sources/betterdb/packages/hermes-memory-py/betterdb_hermes_memory/__init__.py:308` | `sources/betterdb/packages/hermes-memory-py/betterdb_hermes_memory/__init__.py:315` |
| `initialize` | `sources/betterdb/packages/hermes-memory-py/betterdb_hermes_memory/__init__.py:323` | `sources/betterdb/packages/hermes-memory-py/betterdb_hermes_memory/__init__.py:324` |
| `queue_prefetch` | `sources/betterdb/packages/hermes-memory-py/betterdb_hermes_memory/__init__.py:408` | `sources/betterdb/packages/hermes-memory-py/betterdb_hermes_memory/__init__.py:414` |
| `prefetch` | `sources/betterdb/packages/hermes-memory-py/betterdb_hermes_memory/__init__.py:455` | `sources/betterdb/packages/hermes-memory-py/betterdb_hermes_memory/__init__.py:462` |
| `on_session_switch` | `sources/betterdb/packages/hermes-memory-py/betterdb_hermes_memory/__init__.py:481` | `sources/betterdb/packages/hermes-memory-py/betterdb_hermes_memory/__init__.py:491` |
| `sync_turn` | `sources/betterdb/packages/hermes-memory-py/betterdb_hermes_memory/__init__.py:511` | `sources/betterdb/packages/hermes-memory-py/betterdb_hermes_memory/__init__.py:521` |
| `on_memory_write` | `sources/betterdb/packages/hermes-memory-py/betterdb_hermes_memory/__init__.py:530` | `sources/betterdb/packages/hermes-memory-py/betterdb_hermes_memory/__init__.py:534` |
| `get_tool_schemas` | `sources/betterdb/packages/hermes-memory-py/betterdb_hermes_memory/__init__.py:575` | `sources/betterdb/packages/hermes-memory-py/betterdb_hermes_memory/__init__.py:576` |
| `handle_tool_call` | `sources/betterdb/packages/hermes-memory-py/betterdb_hermes_memory/__init__.py:614` | `sources/betterdb/packages/hermes-memory-py/betterdb_hermes_memory/__init__.py:615` |
| `shutdown` | `sources/betterdb/packages/hermes-memory-py/betterdb_hermes_memory/__init__.py:673` | `sources/betterdb/packages/hermes-memory-py/betterdb_hermes_memory/__init__.py:674` |

## 2. Forget paths and derived copies

`yes`: betterdb_forget reads the record, verifies agent_id and effective namespace, then calls store.forget (`sources/betterdb/packages/hermes-memory-py/betterdb_hermes_memory/__init__.py:653`; `sources/betterdb/packages/hermes-memory-py/betterdb_hermes_memory/__init__.py:660`). Engine `forget(id)` executes DEL on one memory hash; `forget_by_scope` searches/deletes matching thread/agent/namespace/tag hashes in bounded batches (`sources/betterdb/packages/agent-memory-py/betterdb_agent_memory/memory_store.py:733`; `sources/betterdb/packages/agent-memory-py/betterdb_agent_memory/memory_store.py:777`). Cap exhaustion emits a warning rather than proving full removal (`sources/betterdb/packages/agent-memory-py/betterdb_agent_memory/memory_store.py:787`). `partial`: content/vector hash is deleted, but adapter's cached recall block is not cleared by this handler, and no host-copy cleanup exists.

Host state.db is independent (`hermes-agent/hermes_state.py:181`). No reviewed adapter forget body performs a general purge of that database or built-in MEMORY.md/USER.md. Prompt/tool-result copies and independent source transcripts remain separate; source operation bodies below establish the scope of each plugin mutation.

<details>
<summary>Deletion, invalidation, archive and reset body ledger</summary>

Public routes, SDK/CLI dispatch and internal storage operations are distinguished by their source. Cache invalidation and management-object deletion are not additional fact-forget tools. A stub is not working deletion. Call-through means the downstream body still determines erasure. HTTP prefixes/selectors must be read with route registration.

**operation and route call sites**, body `sources/betterdb/packages/agent-memory/src/MemoryStore.ts:63`; body evidence.

- `sources/betterdb/packages/agent-memory/src/MemoryStore.ts:62` `const RECALL_OVERFETCH = 4;`
- `sources/betterdb/packages/agent-memory/src/MemoryStore.ts:63` `const FORGET_BATCH_SIZE = 500;`
- `sources/betterdb/packages/agent-memory/src/MemoryStore.ts:64` `const FORGET_MAX_BATCHES = 10000;`
- `sources/betterdb/packages/agent-memory/src/MemoryStore.ts:65` `const EVICTION_SCAN_LIMIT = 10000;`
- `sources/betterdb/packages/agent-memory/src/MemoryStore.ts:66` `const CONSOLIDATE_SCAN_LIMIT = 10000;`
- `sources/betterdb/packages/agent-memory/src/MemoryStore.ts:67` `const DEFAULT_SUMMARY_IMPORTANCE = 0.7;`
- `sources/betterdb/packages/agent-memory/src/MemoryStore.ts:68` `const SUMMARY_SOURCE = 'summary';`
- `sources/betterdb/packages/agent-memory/src/MemoryStore.ts:478` `if (this.discovery) {`
- `sources/betterdb/packages/agent-memory/src/MemoryStore.ts:479` `await this.discovery.stop({ deleteHeartbeat: true });`
- `sources/betterdb/packages/agent-memory/src/MemoryStore.ts:480` `}`
- `sources/betterdb/packages/agent-memory/src/MemoryStore.ts:481` `this.captureSession();`
- `sources/betterdb/packages/agent-memory/src/MemoryStore.ts:482` `if (this.sessionExitHandler) {`
- `sources/betterdb/packages/agent-memory/src/MemoryStore.ts:483` `process.removeListener('beforeExit', this.sessionExitHandler);`
- `sources/betterdb/packages/agent-memory/src/MemoryStore.ts:678` ``
- `sources/betterdb/packages/agent-memory/src/MemoryStore.ts:679` `async forget(id: string): Promise<boolean> {`
- `sources/betterdb/packages/agent-memory/src/MemoryStore.ts:680` `await this.ensureAnalyticsStarted();`
- `sources/betterdb/packages/agent-memory/src/MemoryStore.ts:681` `const removed = Number(await this.client.call('DEL', '${this.name}:mem:${id}'));`
- `sources/betterdb/packages/agent-memory/src/MemoryStore.ts:682` `if (removed > 0) {`
- `sources/betterdb/packages/agent-memory/src/MemoryStore.ts:683` `this.telemetry.metrics.items.labels(this.storeLabels).dec(removed);`
- `sources/betterdb/packages/agent-memory/src/MemoryStore.ts:688` ``
- `sources/betterdb/packages/agent-memory/src/MemoryStore.ts:689` `async forgetByScope(scope: MemoryScope & { tags?: string[] }): Promise<number> {`
- `sources/betterdb/packages/agent-memory/src/MemoryStore.ts:690` `await this.ensureAnalyticsStarted();`
- `sources/betterdb/packages/agent-memory/src/MemoryStore.ts:691` `const tags = scope.tags ?? [];`
- `sources/betterdb/packages/agent-memory/src/MemoryStore.ts:692` `const hasFilter =`
- `sources/betterdb/packages/agent-memory/src/MemoryStore.ts:693` `scope.threadId !== undefined ||`
- `sources/betterdb/packages/agent-memory/src/MemoryStore.ts:697` `if (!hasFilter) {`
- `sources/betterdb/packages/agent-memory/src/MemoryStore.ts:698` `throw new Error('forgetByScope requires at least one scope field or tag');`
- `sources/betterdb/packages/agent-memory/src/MemoryStore.ts:699` `}`
- `sources/betterdb/packages/agent-memory/src/MemoryStore.ts:700` ``
- `sources/betterdb/packages/agent-memory/src/MemoryStore.ts:701` `const filter = buildScopeFilter(scope, tags);`
- `sources/betterdb/packages/agent-memory/src/MemoryStore.ts:702` `let deleted = 0;`
- `sources/betterdb/packages/agent-memory/src/MemoryStore.ts:703` `let batch = 0;`
- `sources/betterdb/packages/agent-memory/src/MemoryStore.ts:704` ``
- `sources/betterdb/packages/agent-memory/src/MemoryStore.ts:705` `for (; batch < FORGET_MAX_BATCHES; batch++) {`
- `sources/betterdb/packages/agent-memory/src/MemoryStore.ts:706` `const raw = await this.client.call(`
- `sources/betterdb/packages/agent-memory/src/MemoryStore.ts:707` `'FT.SEARCH',`
- `sources/betterdb/packages/agent-memory/src/MemoryStore.ts:708` `'${this.name}:mem:idx',`
- `sources/betterdb/packages/agent-memory/src/MemoryStore.ts:709` `filter,`
- `sources/betterdb/packages/agent-memory/src/MemoryStore.ts:711` `'0',`
- `sources/betterdb/packages/agent-memory/src/MemoryStore.ts:712` `String(FORGET_BATCH_SIZE),`
- `sources/betterdb/packages/agent-memory/src/MemoryStore.ts:713` `'DIALECT',`
- `sources/betterdb/packages/agent-memory/src/MemoryStore.ts:714` `'2',`
- `sources/betterdb/packages/agent-memory/src/MemoryStore.ts:715` `);`
- `sources/betterdb/packages/agent-memory/src/MemoryStore.ts:716` `const keys = parseFtSearchResponse(raw).map((hit) => hit.key);`
- `sources/betterdb/packages/agent-memory/src/MemoryStore.ts:720` `const removed = Number(await this.client.call('DEL', ...keys));`
- `sources/betterdb/packages/agent-memory/src/MemoryStore.ts:721` `deleted += removed;`
- `sources/betterdb/packages/agent-memory/src/MemoryStore.ts:722` `// Stop when a batch makes no progress (every match was already gone),`
- `sources/betterdb/packages/agent-memory/src/MemoryStore.ts:723` `// so a lagging index that re-lists deleted keys can't loop forever.`
- `sources/betterdb/packages/agent-memory/src/MemoryStore.ts:724` `if (removed === 0) {`
- `sources/betterdb/packages/agent-memory/src/MemoryStore.ts:725` `break;`
- `sources/betterdb/packages/agent-memory/src/MemoryStore.ts:730` `// surface it rather than returning a partial count that reads as complete.`
- `sources/betterdb/packages/agent-memory/src/MemoryStore.ts:731` `if (batch === FORGET_MAX_BATCHES) {`
- `sources/betterdb/packages/agent-memory/src/MemoryStore.ts:732` `console.warn(`
- `sources/betterdb/packages/agent-memory/src/MemoryStore.ts:733` `'forgetByScope hit the ${FORGET_MAX_BATCHES}-batch safety cap for '${this.name}'; ' +`
- `sources/betterdb/packages/agent-memory/src/MemoryStore.ts:734` `'${deleted} memories deleted, but some matches may remain , re-run to continue.',`
- `sources/betterdb/packages/agent-memory/src/MemoryStore.ts:735` `);`
- `sources/betterdb/packages/agent-memory/src/MemoryStore.ts:736` `}`
- `sources/betterdb/packages/agent-memory/src/MemoryStore.ts:737` ``
- `sources/betterdb/packages/agent-memory/src/MemoryStore.ts:738` `if (deleted > 0) {`
- `sources/betterdb/packages/agent-memory/src/MemoryStore.ts:739` `this.telemetry.metrics.items.labels(this.storeLabels).dec(deleted);`
- `sources/betterdb/packages/agent-memory/src/MemoryStore.ts:740` `this.sessionCounts.forgotten += deleted;`
- `sources/betterdb/packages/agent-memory/src/MemoryStore.ts:741` `}`
- `sources/betterdb/packages/agent-memory/src/MemoryStore.ts:742` `return deleted;`
- `sources/betterdb/packages/agent-memory/src/MemoryStore.ts:743` `}`
- `sources/betterdb/packages/agent-memory/src/MemoryStore.ts:744` ``
- `sources/betterdb/packages/agent-memory/src/MemoryStore.ts:745` `private async writeMemory(`
- `sources/betterdb/packages/agent-memory/src/MemoryStore.ts:746` `content: string,`
- `sources/betterdb/packages/agent-memory/src/MemoryStore.ts:876` `span.setAttribute('consolidate.created', 0);`
- `sources/betterdb/packages/agent-memory/src/MemoryStore.ts:877` `span.setAttribute('consolidate.deleted', 0);`
- `sources/betterdb/packages/agent-memory/src/MemoryStore.ts:878` `return { consolidated: 0, created: [], deleted: 0 };`
- `sources/betterdb/packages/agent-memory/src/MemoryStore.ts:879` `}`
- `sources/betterdb/packages/agent-memory/src/MemoryStore.ts:880` ``
- `sources/betterdb/packages/agent-memory/src/MemoryStore.ts:881` `// Write the summary before deleting sources so a failure can never destroy`
- `sources/betterdb/packages/agent-memory/src/MemoryStore.ts:882` `// memories without leaving their consolidated replacement behind. Use the`
- `sources/betterdb/packages/agent-memory/src/MemoryStore.ts:898` ``
- `sources/betterdb/packages/agent-memory/src/MemoryStore.ts:899` `let deleted = 0;`
- `sources/betterdb/packages/agent-memory/src/MemoryStore.ts:900` `if (options.deleteSources !== false) {`
- `sources/betterdb/packages/agent-memory/src/MemoryStore.ts:901` `const keys = candidates.map((item) => '${this.name}:mem:${item.id}');`
- `sources/betterdb/packages/agent-memory/src/MemoryStore.ts:902` `deleted = Number(await this.client.call('DEL', ...keys));`
- `sources/betterdb/packages/agent-memory/src/MemoryStore.ts:903` `if (deleted > 0) {`
- `sources/betterdb/packages/agent-memory/src/MemoryStore.ts:904` `this.telemetry.metrics.items.labels(this.storeLabels).dec(deleted);`
- `sources/betterdb/packages/agent-memory/src/MemoryStore.ts:905` `}`
- `sources/betterdb/packages/agent-memory/src/MemoryStore.ts:906` `}`
- `sources/betterdb/packages/agent-memory/src/MemoryStore.ts:907` ``
- `sources/betterdb/packages/agent-memory/src/MemoryStore.ts:908` `this.telemetry.metrics.consolidations.labels(this.storeLabels).inc();`
- `sources/betterdb/packages/agent-memory/src/MemoryStore.ts:911` `sources: candidates.length,`
- `sources/betterdb/packages/agent-memory/src/MemoryStore.ts:912` `deleted,`
- `sources/betterdb/packages/agent-memory/src/MemoryStore.ts:913` `});`
- `sources/betterdb/packages/agent-memory/src/MemoryStore.ts:914` `span.setAttribute('consolidate.created', 1);`
- `sources/betterdb/packages/agent-memory/src/MemoryStore.ts:915` `span.setAttribute('consolidate.deleted', deleted);`
- `sources/betterdb/packages/agent-memory/src/MemoryStore.ts:916` `return { consolidated: candidates.length, created: [summaryId], deleted };`
- `sources/betterdb/packages/agent-memory/src/MemoryStore.ts:917` `}`
- `sources/betterdb/packages/agent-memory/src/MemoryStore.ts:918` ``
- `sources/betterdb/packages/agent-memory/src/MemoryStore.ts:919` `/**`
- `sources/betterdb/packages/agent-memory/src/MemoryStore.ts:920` `* Write-time fact consolidation: distill the selected source memories into`
- `sources/betterdb/packages/agent-memory/src/MemoryStore.ts:1001` `span.setAttribute('consolidate_facts.created', 0);`
- `sources/betterdb/packages/agent-memory/src/MemoryStore.ts:1002` `span.setAttribute('consolidate_facts.deleted', 0);`
- `sources/betterdb/packages/agent-memory/src/MemoryStore.ts:1003` `return { candidates: 0, facts: 0, created: [], deleted: 0, unmatchedTombstones: [] };`
- `sources/betterdb/packages/agent-memory/src/MemoryStore.ts:1004` `}`
- `sources/betterdb/packages/agent-memory/src/MemoryStore.ts:1005` ``
- `sources/betterdb/packages/agent-memory/src/MemoryStore.ts:1006` `// Load the fact memories already written for this scope so reconciliation`
- `sources/betterdb/packages/agent-memory/src/MemoryStore.ts:1007` `// considers them (not just the current batch): a re-run over the same`
- `sources/betterdb/packages/agent-memory/src/MemoryStore.ts:1038` `// A tombstone that matched no live fact is surfaced, not silently dropped.`
- `sources/betterdb/packages/agent-memory/src/MemoryStore.ts:1039` `const unmatchedTombstones = ops.flatMap((op) =>`
- `sources/betterdb/packages/agent-memory/src/MemoryStore.ts:1040` `op.type === 'unmatched-tombstone' ? [op.subject] : [],`
- `sources/betterdb/packages/agent-memory/src/MemoryStore.ts:1041` `);`
- `sources/betterdb/packages/agent-memory/src/MemoryStore.ts:1042` `if (unmatchedTombstones.length > 0) {`
- `sources/betterdb/packages/agent-memory/src/MemoryStore.ts:1043` `this.telemetry.metrics.factTombstoneUnmatched`
- `sources/betterdb/packages/agent-memory/src/MemoryStore.ts:1044` `.labels(this.storeLabels)`
- `sources/betterdb/packages/agent-memory/src/MemoryStore.ts:1045` `.inc(unmatchedTombstones.length);`
- `sources/betterdb/packages/agent-memory/src/MemoryStore.ts:1046` `}`
- `sources/betterdb/packages/agent-memory/src/MemoryStore.ts:1047` `const curatedBySubject = new Map<string, Fact>();`
- `sources/betterdb/packages/agent-memory/src/MemoryStore.ts:1048` `for (const fact of curated) {`
- `sources/betterdb/packages/agent-memory/src/MemoryStore.ts:1049` `curatedBySubject.set(subjectKey(fact.subject), fact);`
- `sources/betterdb/packages/agent-memory/src/MemoryStore.ts:1051` `span.setAttribute('consolidate_facts.facts', curated.length);`
- `sources/betterdb/packages/agent-memory/src/MemoryStore.ts:1052` `span.setAttribute('consolidate_facts.unmatched_tombstones', unmatchedTombstones.length);`
- `sources/betterdb/packages/agent-memory/src/MemoryStore.ts:1053` ``
- `sources/betterdb/packages/agent-memory/src/MemoryStore.ts:1054` `// Diff the curated set against what is stored. Writing before deleting keeps`
- `sources/betterdb/packages/agent-memory/src/MemoryStore.ts:1055` `// a crash between the two from losing a fact (recall-safe, at worst a`
- `sources/betterdb/packages/agent-memory/src/MemoryStore.ts:1056` `// duplicate), mirroring consolidate()'s write-then-delete ordering.`
- `sources/betterdb/packages/agent-memory/src/MemoryStore.ts:1082` `// rows from a prior concurrent-write race.`
- `sources/betterdb/packages/agent-memory/src/MemoryStore.ts:1083` `const toDelete: string[] = duplicateFactIds.map((id) => '${this.name}:mem:${id}');`
- `sources/betterdb/packages/agent-memory/src/MemoryStore.ts:1084` `for (const [subject, existing] of existingBySubject) {`
- `sources/betterdb/packages/agent-memory/src/MemoryStore.ts:1085` `const curatedFact = curatedBySubject.get(subject);`
- `sources/betterdb/packages/agent-memory/src/MemoryStore.ts:1086` `if (curatedFact === undefined || factContent(curatedFact) !== existing.content) {`
- `sources/betterdb/packages/agent-memory/src/MemoryStore.ts:1087` `toDelete.push('${this.name}:mem:${existing.id}');`
- `sources/betterdb/packages/agent-memory/src/MemoryStore.ts:1088` `}`
- `sources/betterdb/packages/agent-memory/src/MemoryStore.ts:1089` `}`
- `sources/betterdb/packages/agent-memory/src/MemoryStore.ts:1090` `let deleted = 0;`
- `sources/betterdb/packages/agent-memory/src/MemoryStore.ts:1091` `if (toDelete.length > 0) {`
- `sources/betterdb/packages/agent-memory/src/MemoryStore.ts:1092` `deleted = Number(await this.client.call('DEL', ...toDelete));`
- `sources/betterdb/packages/agent-memory/src/MemoryStore.ts:1093` `if (deleted > 0) {`
- `sources/betterdb/packages/agent-memory/src/MemoryStore.ts:1094` `this.telemetry.metrics.items.labels(this.storeLabels).dec(deleted);`
- `sources/betterdb/packages/agent-memory/src/MemoryStore.ts:1095` `}`
- `sources/betterdb/packages/agent-memory/src/MemoryStore.ts:1096` `}`
- `sources/betterdb/packages/agent-memory/src/MemoryStore.ts:1097` ``
- `sources/betterdb/packages/agent-memory/src/MemoryStore.ts:1098` `this.telemetry.metrics.consolidations.labels(this.storeLabels).inc();`
- `sources/betterdb/packages/agent-memory/src/MemoryStore.ts:1099` `span.setAttribute('consolidate_facts.created', created.length);`
- `sources/betterdb/packages/agent-memory/src/MemoryStore.ts:1100` `span.setAttribute('consolidate_facts.deleted', deleted);`
- `sources/betterdb/packages/agent-memory/src/MemoryStore.ts:1101` `return { candidates: candidates.length, facts: curated.length, created, deleted, unmatchedTombstones };`
- `sources/betterdb/packages/agent-memory/src/MemoryStore.ts:1102` `}`
- `sources/betterdb/packages/agent-memory/src/MemoryStore.ts:1103` ``
- `sources/betterdb/packages/agent-memory/src/MemoryStore.ts:1104` `// Load the fact memories already written for this scope (source == factSource),`
- `sources/betterdb/packages/agent-memory/src/MemoryStore.ts:1105` `// so a consolidateFacts run can reconcile against them.`

**operation and route call sites**, body `sources/betterdb/packages/agent-memory/src/discovery.ts:94`; body evidence.

- `sources/betterdb/packages/agent-memory/src/discovery.ts:93` ``
- `sources/betterdb/packages/agent-memory/src/discovery.ts:94` `async stop(opts: { deleteHeartbeat: boolean }): Promise<void> {`
- `sources/betterdb/packages/agent-memory/src/discovery.ts:95` `if (this.heartbeatHandle) {`
- `sources/betterdb/packages/agent-memory/src/discovery.ts:96` `clearInterval(this.heartbeatHandle);`
- `sources/betterdb/packages/agent-memory/src/discovery.ts:97` `this.heartbeatHandle = null;`
- `sources/betterdb/packages/agent-memory/src/discovery.ts:98` `}`
- `sources/betterdb/packages/agent-memory/src/discovery.ts:103` `}`
- `sources/betterdb/packages/agent-memory/src/discovery.ts:104` `if (!opts.deleteHeartbeat) {`
- `sources/betterdb/packages/agent-memory/src/discovery.ts:105` `return;`
- `sources/betterdb/packages/agent-memory/src/discovery.ts:106` `}`
- `sources/betterdb/packages/agent-memory/src/discovery.ts:107` `try {`
- `sources/betterdb/packages/agent-memory/src/discovery.ts:108` `await this.client.call('DEL', this.heartbeatKey);`

**operation and route call sites**, body `sources/betterdb/packages/agent-memory/src/reconcileFacts.ts:40`; body evidence.

- `sources/betterdb/packages/agent-memory/src/reconcileFacts.ts:39` `| { type: 'update'; subject: string; fact: Fact }`
- `sources/betterdb/packages/agent-memory/src/reconcileFacts.ts:40` `| { type: 'delete'; subject: string }`
- `sources/betterdb/packages/agent-memory/src/reconcileFacts.ts:41` `| { type: 'noop'; subject: string }`
- `sources/betterdb/packages/agent-memory/src/reconcileFacts.ts:42` `// A tombstone whose subject matched no prior fact. It stores nothing (like a`
- `sources/betterdb/packages/agent-memory/src/reconcileFacts.ts:43` `// noop) but is surfaced separately so the caller can log it instead of losing`
- `sources/betterdb/packages/agent-memory/src/reconcileFacts.ts:44` `// it silently , a model that mislabels a live fact as a tombstone shows up here.`
- `sources/betterdb/packages/agent-memory/src/reconcileFacts.ts:45` `| { type: 'unmatched-tombstone'; subject: string };`
- `sources/betterdb/packages/agent-memory/src/reconcileFacts.ts:46` ``
- `sources/betterdb/packages/agent-memory/src/reconcileFacts.ts:47` `/**`
- `sources/betterdb/packages/agent-memory/src/reconcileFacts.ts:48` `* Match key for a subject: case- and whitespace-insensitive, so "Dashboard theme"`
- `sources/betterdb/packages/agent-memory/src/reconcileFacts.ts:49` `* and "dashboard theme" reconcile to the same fact. Only the match key is folded;`
- `sources/betterdb/packages/agent-memory/src/reconcileFacts.ts:77` `// claim to lose against).`
- `sources/betterdb/packages/agent-memory/src/reconcileFacts.ts:78` `function isStaleTombstone(tombstone: Fact, prior: Fact): boolean {`
- `sources/betterdb/packages/agent-memory/src/reconcileFacts.ts:79` `const tombstoneDate = factDate(tombstone);`
- `sources/betterdb/packages/agent-memory/src/reconcileFacts.ts:80` `const priorDate = factDate(prior);`
- `sources/betterdb/packages/agent-memory/src/reconcileFacts.ts:81` `return tombstoneDate !== undefined && priorDate !== undefined && tombstoneDate < priorDate;`
- `sources/betterdb/packages/agent-memory/src/reconcileFacts.ts:82` `}`
- `sources/betterdb/packages/agent-memory/src/reconcileFacts.ts:83` ``
- `sources/betterdb/packages/agent-memory/src/reconcileFacts.ts:84` `/**`
- `sources/betterdb/packages/agent-memory/src/reconcileFacts.ts:85` `* Reconcile a batch of 'incoming' facts against the 'existing' curated set,`
- `sources/betterdb/packages/agent-memory/src/reconcileFacts.ts:106` `const prior = bySubject.get(key);`
- `sources/betterdb/packages/agent-memory/src/reconcileFacts.ts:107` `if (fact.tombstone === true) {`
- `sources/betterdb/packages/agent-memory/src/reconcileFacts.ts:108` `if (prior === undefined) {`
- `sources/betterdb/packages/agent-memory/src/reconcileFacts.ts:109` `// No live fact to retract: surface it rather than silently swallow it.`
- `sources/betterdb/packages/agent-memory/src/reconcileFacts.ts:110` `ops.push({ type: 'unmatched-tombstone', subject: fact.subject });`
- `sources/betterdb/packages/agent-memory/src/reconcileFacts.ts:111` `} else if (isStaleTombstone(fact, prior)) {`
- `sources/betterdb/packages/agent-memory/src/reconcileFacts.ts:112` `ops.push({ type: 'noop', subject: fact.subject });`
- `sources/betterdb/packages/agent-memory/src/reconcileFacts.ts:113` `} else {`
- `sources/betterdb/packages/agent-memory/src/reconcileFacts.ts:114` `ops.push({ type: 'delete', subject: fact.subject });`
- `sources/betterdb/packages/agent-memory/src/reconcileFacts.ts:115` `bySubject.delete(key);`
- `sources/betterdb/packages/agent-memory/src/reconcileFacts.ts:116` `}`
- `sources/betterdb/packages/agent-memory/src/reconcileFacts.ts:117` `continue;`
- `sources/betterdb/packages/agent-memory/src/reconcileFacts.ts:118` `}`
- `sources/betterdb/packages/agent-memory/src/reconcileFacts.ts:119` `if (prior === undefined) {`
- `sources/betterdb/packages/agent-memory/src/reconcileFacts.ts:162` `bySubject.set(subjectKey(op.subject), op.fact);`
- `sources/betterdb/packages/agent-memory/src/reconcileFacts.ts:163` `} else if (op.type === 'delete') {`
- `sources/betterdb/packages/agent-memory/src/reconcileFacts.ts:164` `bySubject.delete(subjectKey(op.subject));`
- `sources/betterdb/packages/agent-memory/src/reconcileFacts.ts:165` `}`
- `sources/betterdb/packages/agent-memory/src/reconcileFacts.ts:166` `// 'noop' and 'unmatched-tombstone' change nothing in the curated set.`
- `sources/betterdb/packages/agent-memory/src/reconcileFacts.ts:167` `}`
- `sources/betterdb/packages/agent-memory/src/reconcileFacts.ts:168` `return [...bySubject.values()];`

**operation and route call sites**, body `sources/betterdb/packages/agent-memory/src/telemetry.ts:34`; body evidence.

- `sources/betterdb/packages/agent-memory/src/telemetry.ts:33` `consolidations: Counter<string>;`
- `sources/betterdb/packages/agent-memory/src/telemetry.ts:34` `factTombstoneUnmatched: Counter<string>;`
- `sources/betterdb/packages/agent-memory/src/telemetry.ts:35` `}`
- `sources/betterdb/packages/agent-memory/src/telemetry.ts:36` ``
- `sources/betterdb/packages/agent-memory/src/telemetry.ts:37` `export interface MemoryTelemetry {`
- `sources/betterdb/packages/agent-memory/src/telemetry.ts:38` `tracer: Tracer;`
- `sources/betterdb/packages/agent-memory/src/telemetry.ts:121` `}),`
- `sources/betterdb/packages/agent-memory/src/telemetry.ts:122` `factTombstoneUnmatched: getOrCreateCounter(registry, {`
- `sources/betterdb/packages/agent-memory/src/telemetry.ts:123` `name: '${prefix}_fact_tombstone_unmatched_total',`
- `sources/betterdb/packages/agent-memory/src/telemetry.ts:124` `help: 'Fact tombstones that matched no live fact (surfaced, not silently dropped)',`
- `sources/betterdb/packages/agent-memory/src/telemetry.ts:125` `labelNames,`
- `sources/betterdb/packages/agent-memory/src/telemetry.ts:126` `}),`
- `sources/betterdb/packages/agent-memory/src/telemetry.ts:127` `},`
- `sources/betterdb/packages/agent-memory/src/telemetry.ts:128` `};`

**operation and route call sites**, body `sources/betterdb/packages/agent-memory/src/types.ts:83`; body evidence.

- `sources/betterdb/packages/agent-memory/src/types.ts:82` `summarize: (items: MemoryItem[]) => Promise<string>;`
- `sources/betterdb/packages/agent-memory/src/types.ts:83` `deleteSources?: boolean;`
- `sources/betterdb/packages/agent-memory/src/types.ts:84` `summaryImportance?: number;`
- `sources/betterdb/packages/agent-memory/src/types.ts:85` `tags?: string[];`
- `sources/betterdb/packages/agent-memory/src/types.ts:86` `}`
- `sources/betterdb/packages/agent-memory/src/types.ts:87` ``
- `sources/betterdb/packages/agent-memory/src/types.ts:90` `created: string[];`
- `sources/betterdb/packages/agent-memory/src/types.ts:91` `deleted: number;`
- `sources/betterdb/packages/agent-memory/src/types.ts:92` `}`
- `sources/betterdb/packages/agent-memory/src/types.ts:93` ``
- `sources/betterdb/packages/agent-memory/src/types.ts:94` `/**`
- `sources/betterdb/packages/agent-memory/src/types.ts:95` `* An atomic, durable fact distilled from one or more memories. 'subject' is a`
- `sources/betterdb/packages/agent-memory/src/types.ts:104` `date?: string;`
- `sources/betterdb/packages/agent-memory/src/types.ts:105` `tombstone?: boolean;`
- `sources/betterdb/packages/agent-memory/src/types.ts:106` `}`
- `sources/betterdb/packages/agent-memory/src/types.ts:107` ``
- `sources/betterdb/packages/agent-memory/src/types.ts:108` `/**`
- `sources/betterdb/packages/agent-memory/src/types.ts:109` `* Caller-provided LLM seam that distills a batch of source memories into atomic`
- `sources/betterdb/packages/agent-memory/src/types.ts:133` `created: string[];`
- `sources/betterdb/packages/agent-memory/src/types.ts:134` `/** Prior fact memories deleted because a run superseded or retracted their subject. */`
- `sources/betterdb/packages/agent-memory/src/types.ts:135` `deleted: number;`
- `sources/betterdb/packages/agent-memory/src/types.ts:136` `/**`
- `sources/betterdb/packages/agent-memory/src/types.ts:137` `* Subjects the extractor tombstoned but that matched no live fact, so nothing`
- `sources/betterdb/packages/agent-memory/src/types.ts:138` `* was retracted. Surfaced instead of silently dropped: a model that mislabels a`
- `sources/betterdb/packages/agent-memory/src/types.ts:139` `* current fact as a tombstone (a known failure mode) shows up here rather than`
- `sources/betterdb/packages/agent-memory/src/types.ts:141` `*/`
- `sources/betterdb/packages/agent-memory/src/types.ts:142` `unmatchedTombstones: string[];`
- `sources/betterdb/packages/agent-memory/src/types.ts:143` `}`
- `sources/betterdb/packages/agent-memory/src/types.ts:144` ``
- `sources/betterdb/packages/agent-memory/src/types.ts:145` `export interface MemoryListOptions extends MemoryScope {`
- `sources/betterdb/packages/agent-memory/src/types.ts:146` `tags?: string[];`

**forget**, body `sources/betterdb/packages/agent-memory-py/betterdb_agent_memory/memory_store.py:732`; implementation/call-through.

- `sources/betterdb/packages/agent-memory-py/betterdb_agent_memory/memory_store.py:731` `async def forget(self, id: str) -> bool:`
- `sources/betterdb/packages/agent-memory-py/betterdb_agent_memory/memory_store.py:733` `removed = int(await self._client.execute_command("DEL", f"{self._name}:mem:{id}"))`

**forget_by_scope**, body `sources/betterdb/packages/agent-memory-py/betterdb_agent_memory/memory_store.py:747`; implementation/call-through.

- `sources/betterdb/packages/agent-memory-py/betterdb_agent_memory/memory_store.py:739` `async def forget_by_scope(`
- `sources/betterdb/packages/agent-memory-py/betterdb_agent_memory/memory_store.py:777` `removed = int(await self._client.execute_command("DEL", *keys))`

**_is_stale_tombstone**, body `sources/betterdb/packages/agent-memory-py/betterdb_agent_memory/reconcile_facts.py:107`; implementation/call-through.

- `sources/betterdb/packages/agent-memory-py/betterdb_agent_memory/reconcile_facts.py:103` `def _is_stale_tombstone(tombstone: Fact, prior: Fact) -> bool:`
- `sources/betterdb/packages/agent-memory-py/betterdb_agent_memory/reconcile_facts.py:107` `tombstone_date = _fact_date(tombstone)`
- `sources/betterdb/packages/agent-memory-py/betterdb_agent_memory/reconcile_facts.py:109` `return tombstone_date is not None and prior_date is not None and tombstone_date < prior_date`

</details>

## 3. What reaches the agent

Prefetch consumes and clears the cached block only for the matching session (`sources/betterdb/packages/hermes-memory-py/betterdb_hermes_memory/__init__.py:466`). Queue-prefetch assembly/limits are quoted below. Explicit recall defaults k=8 and returns JSON IDs/content/scores (`sources/betterdb/packages/hermes-memory-py/betterdb_hermes_memory/__init__.py:638`; `sources/betterdb/packages/hermes-memory-py/betterdb_hermes_memory/__init__.py:645`). Do not describe this as synchronous fresh search every turn.

Host assembly: `hermes-agent/agent/memory_manager.py:352` joins provider system blocks. `hermes-agent/agent/turn_context.py:1081` requests prefetch for nontrivial turns; `hermes-agent/agent/turn_context.py:120` fences it; `hermes-agent/agent/turn_context.py:136` appends it to API-bound user content. Tool results are a third channel. Optional host spill is at `hermes-agent/agent/memory_manager.py:419`, not a universal provider cap. Built-in memory remains separately controlled (`hermes-agent/agent/system_prompt.py:542`).

<details>
<summary>Context and tool-result assembly quotes</summary>

`queue_prefetch`, body `sources/betterdb/packages/hermes-memory-py/betterdb_hermes_memory/__init__.py:414`:



`prefetch`, body `sources/betterdb/packages/hermes-memory-py/betterdb_hermes_memory/__init__.py:462`:

- `sources/betterdb/packages/hermes-memory-py/betterdb_hermes_memory/__init__.py:476` `return block`

`get_tool_schemas`, body `sources/betterdb/packages/hermes-memory-py/betterdb_hermes_memory/__init__.py:576`:

- `sources/betterdb/packages/hermes-memory-py/betterdb_hermes_memory/__init__.py:576` `return [`

`handle_tool_call`, body `sources/betterdb/packages/hermes-memory-py/betterdb_hermes_memory/__init__.py:615`:

- `sources/betterdb/packages/hermes-memory-py/betterdb_hermes_memory/__init__.py:615` `from tools.registry import tool_error  # type: ignore`
- `sources/betterdb/packages/hermes-memory-py/betterdb_hermes_memory/__init__.py:618` `return tool_error("BetterDB memory is not initialized")`
- `sources/betterdb/packages/hermes-memory-py/betterdb_hermes_memory/__init__.py:633` `return json.dumps({"id": mem_id, "status": "remembered"})`
- `sources/betterdb/packages/hermes-memory-py/betterdb_hermes_memory/__init__.py:648` `return json.dumps({"results": results, "count": len(results)})`
- `sources/betterdb/packages/hermes-memory-py/betterdb_hermes_memory/__init__.py:655` `return json.dumps({"removed": False, "status": "not found"})`
- `sources/betterdb/packages/hermes-memory-py/betterdb_hermes_memory/__init__.py:659` `return tool_error("refusing to delete a memory from another scope")`
- `sources/betterdb/packages/hermes-memory-py/betterdb_hermes_memory/__init__.py:661` `return json.dumps({"removed": bool(removed)})`
- `sources/betterdb/packages/hermes-memory-py/betterdb_hermes_memory/__init__.py:662` `return tool_error(f"Unknown tool: {tool_name}")`
- `sources/betterdb/packages/hermes-memory-py/betterdb_hermes_memory/__init__.py:664` `return tool_error(f"Missing required argument: {exc}")`
- `sources/betterdb/packages/hermes-memory-py/betterdb_hermes_memory/__init__.py:667` `return tool_error("BetterDB memory timed out after 10s")`
- `sources/betterdb/packages/hermes-memory-py/betterdb_hermes_memory/__init__.py:669` `return tool_error(str(exc))`

</details>

## 4. Data access boundaries

Direct Valkey access `yes` with the configured connection. URL/password use scoped BETTERDB_MEMORY_URL/BETTERDB_MEMORY_PASSWORD and may live in profile .env; nonsecret connection settings are in Hermes config (`sources/betterdb/packages/hermes-memory-py/betterdb_hermes_memory/__init__.py:143`; `sources/betterdb/packages/hermes-memory-py/betterdb_hermes_memory/__init__.py:148`). Telemetry defaults `off` in adapter config (`sources/betterdb/packages/hermes-memory-py/betterdb_hermes_memory/__init__.py:158`). Accountless local mode `yes` with local Valkey and local embeddings; the adapter checks that the endpoint was explicitly configured (`sources/betterdb/packages/hermes-memory-py/betterdb_hermes_memory/__init__.py:255`).

Boundary model: same OS user, terminal, mounted filesystem and allowed network. Direct reachability means another client can use the same file/endpoint and available credentials, not unauthenticated access or encryption bypass. Hermes setup persists declared secrets through `hermes-agent/hermes_cli/memory_setup.py:321` and `hermes-agent/hermes_cli/config.py:2697`. These are possible configured paths; no live credentials were read.

Host credential-file detail: `get_hermes_home()/.env` is resolved at `hermes-agent/hermes_cli/config.py:473`. New files are secured to 0600 at `hermes-agent/hermes_cli/config.py:506`; the writer preserves an existing mode at `hermes-agent/hermes_cli/config.py:2544` and `:2562`. Therefore 0600 is not an unconditional mode claim for every existing profile.

<details>
<summary>Paths, environment, permissions, egress, telemetry and process-launch evidence</summary>

- `sources/betterdb/packages/agent-memory/src/telemetry.ts:1` `import { trace, type Tracer } from '@opentelemetry/api';`
- `sources/betterdb/packages/agent-memory/src/telemetry.ts:18` `export interface MemoryTelemetryOptions {`
- `sources/betterdb/packages/agent-memory/src/telemetry.ts:37` `export interface MemoryTelemetry {`
- `sources/betterdb/packages/agent-memory/src/telemetry.ts:67` `export function createMemoryTelemetry(options: MemoryTelemetryOptions = {}): MemoryTelemetry {`

- `sources/betterdb/packages/agent-memory-py/betterdb_agent_memory/telemetry.py:6` `from opentelemetry import trace`
- `sources/betterdb/packages/agent-memory-py/betterdb_agent_memory/telemetry.py:7` `from opentelemetry.trace import Tracer`
- `sources/betterdb/packages/agent-memory-py/betterdb_agent_memory/telemetry.py:79` `class MemoryTelemetryOptions:`
- `sources/betterdb/packages/agent-memory-py/betterdb_agent_memory/telemetry.py:100` `class MemoryTelemetry:`
- `sources/betterdb/packages/agent-memory-py/betterdb_agent_memory/telemetry.py:105` `def create_memory_telemetry(`
- `sources/betterdb/packages/agent-memory-py/betterdb_agent_memory/telemetry.py:106` `options: MemoryTelemetryOptions | None = None,`
- `sources/betterdb/packages/agent-memory-py/betterdb_agent_memory/telemetry.py:107` `) -> MemoryTelemetry:`
- `sources/betterdb/packages/agent-memory-py/betterdb_agent_memory/telemetry.py:108` `opts = options or MemoryTelemetryOptions()`
- `sources/betterdb/packages/agent-memory-py/betterdb_agent_memory/telemetry.py:178` `return MemoryTelemetry(tracer=tracer, metrics=metrics)`

- `sources/betterdb/packages/hermes-memory-py/betterdb_hermes_memory/__init__.py:55` `def _get_secret(name: str, default: str = "") -> str:`
- `sources/betterdb/packages/hermes-memory-py/betterdb_hermes_memory/__init__.py:60` `''os.environ'' , that is the standalone / test case. A failure raised BY the`
- `sources/betterdb/packages/hermes-memory-py/betterdb_hermes_memory/__init__.py:70` `return os.environ.get(name, default) or default`
- `sources/betterdb/packages/hermes-memory-py/betterdb_hermes_memory/__init__.py:71` `return get_secret(name, default) or default`
- `sources/betterdb/packages/hermes-memory-py/betterdb_hermes_memory/__init__.py:143` `self.url: str = _get_secret("BETTERDB_MEMORY_URL", "") or str(block.get("url", "") or "")`
- `sources/betterdb/packages/hermes-memory-py/betterdb_hermes_memory/__init__.py:148` `self.password: str = _get_secret("BETTERDB_MEMORY_PASSWORD", "")`
- `sources/betterdb/packages/hermes-memory-py/betterdb_hermes_memory/__init__.py:158` `self.telemetry: bool = _as_bool(block.get("telemetry"), False)`
- `sources/betterdb/packages/hermes-memory-py/betterdb_hermes_memory/__init__.py:169` `api_key=_get_secret("BETTERDB_MEMORY_EMBEDDINGS_API_KEY", ""),`
- `sources/betterdb/packages/hermes-memory-py/betterdb_hermes_memory/__init__.py:265` `"Refusing to send embeddings input over plaintext http:// to a remote host "`
- `sources/betterdb/packages/hermes-memory-py/betterdb_hermes_memory/__init__.py:266` `"(it would leak the text, and any API key, in the clear). Use an https:// "`
- `sources/betterdb/packages/hermes-memory-py/betterdb_hermes_memory/__init__.py:292` `{"key": "telemetry",`
- `sources/betterdb/packages/hermes-memory-py/betterdb_hermes_memory/__init__.py:293` `"description": "Enable BetterDB usage analytics in the memory SDK (opt-in, off by default)",`
- `sources/betterdb/packages/hermes-memory-py/betterdb_hermes_memory/__init__.py:305` `"url": "https://platform.openai.com/api-keys"},`
- `sources/betterdb/packages/hermes-memory-py/betterdb_hermes_memory/__init__.py:351` `analytics=self._config.telemetry,`

- `sources/betterdb/packages/hermes-memory-py/betterdb_hermes_memory/embeddings.py:34` `DEFAULT_EMBEDDINGS_BASE_URL = "https://api.openai.com/v1"`
- `sources/betterdb/packages/hermes-memory-py/betterdb_hermes_memory/embeddings.py:51` `hostnames. Plaintext http:// to such a host never leaves a trusted network, so`
- `sources/betterdb/packages/hermes-memory-py/betterdb_hermes_memory/embeddings.py:86` `"""True when request input would ride plaintext http:// to a remote host.`
- `sources/betterdb/packages/hermes-memory-py/betterdb_hermes_memory/embeddings.py:88` `Plaintext http:// exposes both the embedded text and any Bearer credential`
- `sources/betterdb/packages/hermes-memory-py/betterdb_hermes_memory/embeddings.py:90` `is set , a keyless endpoint still leaks the user's input. http:// to a local`
- `sources/betterdb/packages/hermes-memory-py/betterdb_hermes_memory/embeddings.py:92` `hostname) stays on the machine/LAN and is allowed; https:// is always fine.`
- `sources/betterdb/packages/hermes-memory-py/betterdb_hermes_memory/embeddings.py:95` `if not url.lower().startswith("http://"):`
- `sources/betterdb/packages/hermes-memory-py/betterdb_hermes_memory/embeddings.py:128` `"refusing to send embeddings input over plaintext http:// to a remote "`
- `sources/betterdb/packages/hermes-memory-py/betterdb_hermes_memory/embeddings.py:129` `"host; use https:// or a local endpoint (localhost / a private address)")`
- `sources/betterdb/packages/hermes-memory-py/betterdb_hermes_memory/embeddings.py:140` `plaintext = self._config.base_url.strip().lower().startswith("http://")`

- `sources/betterdb/packages/hermes-memory-py/plugin-catalog/betterdb.yaml:2` `repo: https://github.com/BetterDB-inc/monitor`
- `sources/betterdb/packages/hermes-memory-py/plugin-catalog/betterdb.yaml:6` `description: "BetterDB long-term memory provider , semantic recall on Valkey / valkey-search via the betterdb-agent-memory SDK. Completed turns are embedded and stored in a vector index; the next turn recalls the most relevant memories with a composite similarity/recency/importance score. Explicit betterdb_remember / betterdb_recall / betterdb_forget tools. Disclosure , turn text and tool-stored facts are embedded through an OpenAI-compatible embeddings endpoint you configure (embeddings_base_url, default api.openai.com) and written to the Valkey instance you point it at. Opt-in, off by default: a discovery marker (set discovery=true to let BetterDB Monitor enumerate the tier) and SDK usage analytics (set telemetry=true)."`
- `sources/betterdb/packages/hermes-memory-py/plugin-catalog/betterdb.yaml:11` `docs_url: https://github.com/BetterDB-inc/monitor/tree/master/packages/hermes-memory-py#readme`

- `sources/betterdb/packages/hermes-memory-py/pyproject.toml:29` `Homepage = "https://github.com/BetterDB-inc/monitor/tree/master/packages/hermes-memory-py"`
- `sources/betterdb/packages/hermes-memory-py/pyproject.toml:30` `Repository = "https://github.com/BetterDB-inc/monitor"`
- `sources/betterdb/packages/hermes-memory-py/pyproject.toml:31` `Issues = "https://github.com/BetterDB-inc/monitor/issues"`

</details>

## 5. Reproducibility

`range`: betterdb-agent-memory>=0.7.0,<0.8.0, valkey>=6.0,<7.0, httpx>=0.27,<1.0 (`sources/betterdb/packages/hermes-memory-py/pyproject.toml:23`). Server/module version is outside those Python package pins.

Retained lock/checksum files: `sources/betterdb/pnpm-lock.yaml:1`, `sources/betterdb/packages/semantic-cache/examples/llamaindex/package-lock.json:1`, `sources/betterdb/packages/semantic-cache/examples/langgraph/package-lock.json:1`, `sources/betterdb/packages/semantic-cache/examples/vercel-ai-sdk/package-lock.json:1`, `sources/betterdb/packages/semantic-cache/examples/monitor-proposals/package-lock.json:1`, `sources/betterdb/packages/semantic-cache/examples/langchain/package-lock.json:1`, `sources/betterdb/packages/semantic-cache/examples/batch-check/package-lock.json:1`, `sources/betterdb/packages/semantic-cache/examples/basic/package-lock.json:1`, `sources/betterdb/packages/semantic-cache/examples/embedding-cache/package-lock.json:1`, `sources/betterdb/packages/semantic-cache/examples/cost-tracking/package-lock.json:1`, `sources/betterdb/packages/semantic-cache/examples/multimodal/package-lock.json:1`, `sources/betterdb/packages/semantic-cache/examples/judge/pnpm-lock.yaml:1`, `sources/betterdb/packages/semantic-cache/examples/anthropic/package-lock.json:1`, `sources/betterdb/packages/semantic-cache/examples/rerank/package-lock.json:1`, `sources/betterdb/packages/semantic-cache/examples/threshold-tuning/package-lock.json:1`, `sources/betterdb/packages/semantic-cache/examples/openai/package-lock.json:1`, `sources/betterdb/packages/semantic-cache/examples/openai-responses/package-lock.json:1`, `sources/betterdb/packages/agent-cache/examples/llamaindex/package-lock.json:1`, `sources/betterdb/packages/agent-cache/examples/langgraph/package-lock.json:1`, `sources/betterdb/packages/agent-cache/examples/vercel-ai-sdk/package-lock.json:1`, `sources/betterdb/packages/agent-cache/examples/monitor-proposals/package-lock.json:1`, `sources/betterdb/packages/agent-cache/examples/langchain/package-lock.json:1`, `sources/betterdb/packages/agent-cache/examples/anthropic/package-lock.json:1`, `sources/betterdb/packages/agent-cache/examples/openai/package-lock.json:1`. Upstream completeness is unknown: fetch.py:15 excludes .lock files, fetch.py:12 excludes build/vendor directories. Absence here is not absence upstream.

- `sources/betterdb/packages/hermes-memory-py/plugin.yaml:2` `version: "0.1.0"`

- `sources/betterdb/packages/hermes-memory-py/pyproject.toml:7` `version = "0.1.0"`
- `sources/betterdb/packages/hermes-memory-py/pyproject.toml:12` `# Matches Hermes's own range (">=3.11,<3.15"). betterdb-agent-memory declares`
- `sources/betterdb/packages/hermes-memory-py/pyproject.toml:13` `# ">=3.11" (no ceiling) so it resolves cleanly across this whole range.`
- `sources/betterdb/packages/hermes-memory-py/pyproject.toml:14` `requires-python = ">=3.11,<3.15"`
- `sources/betterdb/packages/hermes-memory-py/pyproject.toml:22` `dependencies = [`
- `sources/betterdb/packages/hermes-memory-py/pyproject.toml:23` `"betterdb-agent-memory>=0.7.0,<0.8.0",`
- `sources/betterdb/packages/hermes-memory-py/pyproject.toml:24` `"valkey>=6.0,<7.0",`
- `sources/betterdb/packages/hermes-memory-py/pyproject.toml:25` `"httpx>=0.27,<1.0",`
- `sources/betterdb/packages/hermes-memory-py/pyproject.toml:33` `[project.optional-dependencies]`
- `sources/betterdb/packages/hermes-memory-py/pyproject.toml:35` `"pytest>=8.0.0",`

## 6. Tests

Adapter tests in packages/hermes-memory-py/tests/test_provider.py cover the tool/ownership contract; SDK tests cover DEL/scoped deletion. Counted independently below. Adapter tool-cycle test includes forget; foreign memory refusal is separately tested. (`sources/betterdb/packages/hermes-memory-py/tests/test_provider.py:391`).

Adapter-related declarations: **44**. Python AST test_ functions, fixtures excluded; literal TS/JS it/test, Go Test*, Rust test_* declarations. No parametrization expansion, collection, execution, or coverage measurement. Other rows are engine test candidates selected by mutation names; names alone do not verify forget coverage.

| file | declarations | adapter-related | forget-related declarations |
|---|---:|---|---|
| `sources/betterdb/apps/api/src/auth/auth-secret.spec.ts:1` | 15 | no | `warns that sessions are invalidated when a stored secret is too short` `sources/betterdb/apps/api/src/auth/auth-secret.spec.ts:144` |
| `sources/betterdb/apps/api/src/common/services/__tests__/multi-connection-poller.spec.ts:1` | 9 | no | `forgets the version when the connection is removed` `sources/betterdb/apps/api/src/common/services/__tests__/multi-connection-poller.spec.ts:122` |
| `sources/betterdb/apps/api/src/connections/__tests__/connection-registry.external.spec.ts:1` | 17 | no | `forgets a discovered address once it is registered` `sources/betterdb/apps/api/src/connections/__tests__/connection-registry.external.spec.ts:160`; `forgets the pushed address when the registered host was edited` `sources/betterdb/apps/api/src/connections/__tests__/connection-registry.external.spec.ts:168`; `forgets a discovered address registered as a direct connection` `sources/betterdb/apps/api/src/connections/__tests__/connection-registry.external.spec.ts:199` |
| `sources/betterdb/apps/api/src/connections/__tests__/connection-registry.membership.spec.ts:1` | 48 | no | `disconnects and forgets a managed child when persisting it fails` `sources/betterdb/apps/api/src/connections/__tests__/connection-registry.membership.spec.ts:167`; `cascades seed removal: deletes auto children, detaches adopted ones` `sources/betterdb/apps/api/src/connections/__tests__/connection-registry.membership.spec.ts:562` |
| `sources/betterdb/apps/api/src/cve/__tests__/cve-scan.service.spec.ts:1` | 68 | no | `deletes the stored scan result of a removed connection` `sources/betterdb/apps/api/src/cve/__tests__/cve-scan.service.spec.ts:1807`; `logs a warning instead of rejecting when the delete fails` `sources/betterdb/apps/api/src/cve/__tests__/cve-scan.service.spec.ts:1846` |
| `sources/betterdb/apps/api/src/database/parsers/vector-index.parser.spec.ts:1` | 40 | no | `should parse num_deleted_docs and total_indexing_time when present` `sources/betterdb/apps/api/src/database/parsers/vector-index.parser.spec.ts:93`; `should default num_deleted_docs and total_indexing_time to 0 when absent` `sources/betterdb/apps/api/src/database/parsers/vector-index.parser.spec.ts:106` |
| `sources/betterdb/apps/api/src/database/ssh/__tests__/ssh-tunnel.service.spec.ts:1` | 22 | no | `closes a tunnel and forgets it` `sources/betterdb/apps/api/src/database/ssh/__tests__/ssh-tunnel.service.spec.ts:195` |
| `sources/betterdb/apps/api/src/external-metrics/__tests__/discovered-instances.store.spec.ts:1` | 18 | no | `purges entries unseen for 24 h` `sources/betterdb/apps/api/src/external-metrics/__tests__/discovered-instances.store.spec.ts:65`; `forget removes the entry without dismissing it` `sources/betterdb/apps/api/src/external-metrics/__tests__/discovered-instances.store.spec.ts:82`; `forgetPushed removes the entry and keeps the pushed address from reappearing` `sources/betterdb/apps/api/src/external-metrics/__tests__/discovered-instances.store.spec.ts:90`; `forget clears a lingering dismissal` `sources/betterdb/apps/api/src/external-metrics/__tests__/discovered-instances.store.spec.ts:100` |
| `sources/betterdb/apps/api/src/metric-forecasting/__tests__/metric-forecasting.service.spec.ts:1` | 35 | no | `update invalidates forecast cache` `sources/betterdb/apps/api/src/metric-forecasting/__tests__/metric-forecasting.service.spec.ts:380` |
| `sources/betterdb/apps/api/src/migration/__tests__/type-handlers.spec.ts:1` | 15 | no | `should handle deleted key gracefully and skip write` `sources/betterdb/apps/api/src/migration/__tests__/type-handlers.spec.ts:72` |
| `sources/betterdb/apps/api/src/monitor/__tests__/monitor-support-probe.spec.ts:1` | 15 | no | `probes again after invalidate(connectionId)` `sources/betterdb/apps/api/src/monitor/__tests__/monitor-support-probe.spec.ts:230` |
| `sources/betterdb/apps/api/src/monitor/__tests__/monitor.controller.spec.ts:1` | 47 | no | `deleteSchedule returns { deleted: true } when the scheduler accepts` `sources/betterdb/apps/api/src/monitor/__tests__/monitor.controller.spec.ts:540`; `deleteSchedule throws NotFound when the scheduler reports no row` `sources/betterdb/apps/api/src/monitor/__tests__/monitor.controller.spec.ts:546` |
| `sources/betterdb/apps/api/src/otel-telemetry/__tests__/series-expiry.spec.ts:1` | 11 | no | `forgets expired series on prune and tracks them again once re-observed` `sources/betterdb/apps/api/src/otel-telemetry/__tests__/series-expiry.spec.ts:88` |
| `sources/betterdb/apps/api/src/prometheus/staleness.spec.ts:1` | 16 | no | `forgets a connection and reports its label` `sources/betterdb/apps/api/src/prometheus/staleness.spec.ts:98` |
| `sources/betterdb/apps/api/src/storage/adapters/__tests__/cache-proposals.spec.ts:1` | 17 | no | `preserves payload shape for semantic invalidate` `sources/betterdb/apps/api/src/storage/adapters/__tests__/cache-proposals.spec.ts:346`; `preserves payload shape for agent invalidate by tool` `sources/betterdb/apps/api/src/storage/adapters/__tests__/cache-proposals.spec.ts:370` |
| `sources/betterdb/apps/api/src/storage/adapters/__tests__/capture-sessions.spec.ts:1` | 14 | no | `pruneOldCaptureSessions deletes ended sessions older than the cutoff` `sources/betterdb/apps/api/src/storage/adapters/__tests__/capture-sessions.spec.ts:181`; `pruneOldCaptureChunks deletes chunks whose lastTs is below the cutoff` `sources/betterdb/apps/api/src/storage/adapters/__tests__/capture-sessions.spec.ts:215`; `pruneOldCaptureTriggers deletes terminal triggers older than cutoff and preserves active ones` `sources/betterdb/apps/api/src/storage/adapters/__tests__/capture-sessions.spec.ts:246`; `pruneOldScheduledCaptures deletes disabled rows older than cutoff and preserves enabled ones` `sources/betterdb/apps/api/src/storage/adapters/__tests__/capture-sessions.spec.ts:276` |
| `sources/betterdb/apps/api/src/storage/adapters/__tests__/chunked-delete.spec.ts:1` | 2 | no | `deletes across multiple chunks and reports the full count` `sources/betterdb/apps/api/src/storage/adapters/__tests__/chunked-delete.spec.ts:5` |
| `sources/betterdb/apps/api/src/storage/adapters/__tests__/cve-storage.spec.ts:1` | 32 | no | `deletes every scan row for a removed connection and leaves other connections alone` `sources/betterdb/apps/api/src/storage/adapters/__tests__/cve-storage.spec.ts:273`; `treats deleting the scan of a connection with no stored scan as a no-op` `sources/betterdb/apps/api/src/storage/adapters/__tests__/cve-storage.spec.ts:286` |
| `sources/betterdb/apps/api/src/storage/adapters/__tests__/memory-proposal-integrity.spec.ts:1` | 14 | no | `records that partial deletion is unknown rather than claiming a rollback` `sources/betterdb/apps/api/src/storage/adapters/__tests__/memory-proposal-integrity.spec.ts:190` |
| `sources/betterdb/apps/api/src/storage/adapters/__tests__/webhook-storage.spec.ts:1` | 25 | no | `should delete webhook` `sources/betterdb/apps/api/src/storage/adapters/__tests__/webhook-storage.spec.ts:136`; `should cascade delete deliveries when webhook deleted` `sources/betterdb/apps/api/src/storage/adapters/__tests__/webhook-storage.spec.ts:153` |
| `sources/betterdb/apps/api/src/topology/__tests__/topology-auto-registration.service.spec.ts:1` | 24 | no | `purges retired auto children older than the retention window` `sources/betterdb/apps/api/src/topology/__tests__/topology-auto-registration.service.spec.ts:220` |
| `sources/betterdb/apps/api/src/workspace/member.service.spec.ts:1` | 30 | no | `keeps the sole owner for a retry when deleting their accounts fails` `sources/betterdb/apps/api/src/workspace/member.service.spec.ts:275` |
| `sources/betterdb/apps/api/test/api-webhooks.e2e-spec.ts:1` | 23 | no | `should delete webhook` `sources/betterdb/apps/api/test/api-webhooks.e2e-spec.ts:368` |
| `sources/betterdb/apps/api/test/connections-api.e2e-spec.ts:1` | 17 | no | `should not allow deleting env-default connection` `sources/betterdb/apps/api/test/connections-api.e2e-spec.ts:258` |
| `sources/betterdb/apps/api/test/cve-storage-postgres.e2e-spec.ts:1` | 14 | no | `deletes every scan row for a removed connection and leaves other connections alone` `sources/betterdb/apps/api/test/cve-storage-postgres.e2e-spec.ts:292` |
| `sources/betterdb/apps/web/src/api/client.test.ts:1` | 20 | no | `resolves undefined for 204 No Content responses (e.g. webhook delete)` `sources/betterdb/apps/web/src/api/client.test.ts:79` |
| `sources/betterdb/apps/web/src/components/connection-selector/autoRegisterCopy.test.ts:1` | 9 | no | `warns that deleting a seed deletes its auto nodes and keeps adopted ones` `sources/betterdb/apps/web/src/components/connection-selector/autoRegisterCopy.test.ts:16`; `warns that a deleted auto node comes back` `sources/betterdb/apps/web/src/components/connection-selector/autoRegisterCopy.test.ts:22` |
| `sources/betterdb/apps/web/src/components/pages/cache-proposals/PendingCard.test.tsx:1` | 16 | no | `renders semantic invalidate body with monospace filter block` `sources/betterdb/apps/web/src/components/pages/cache-proposals/PendingCard.test.tsx:121`; `renders agent invalidate body with filter_kind label` `sources/betterdb/apps/web/src/components/pages/cache-proposals/PendingCard.test.tsx:126`; `hides Edit button on invalidate cards` `sources/betterdb/apps/web/src/components/pages/cache-proposals/PendingCard.test.tsx:132` |
| `sources/betterdb/apps/web/src/components/pages/security/scan-error.test.ts:1` | 12 | no | `replaces a deleted connection with a message the reader can act on` `sources/betterdb/apps/web/src/components/pages/security/scan-error.test.ts:13` |
| `sources/betterdb/apps/web/src/pages/BulkDelete.test.tsx:1` | 5 | no | `ignores batching/pacing knobs (count, batchPauseMs, confirmDeleteAll)` `sources/betterdb/apps/web/src/pages/BulkDelete.test.tsx:10` |
| `sources/betterdb/apps/web/src/pages/MonitorSession.test.tsx:1` | 10 | no | `invalidates the session query after a successful stop` `sources/betterdb/apps/web/src/pages/MonitorSession.test.tsx:183` |
| `sources/betterdb/apps/web/src/pages/ScalingReadiness.test.tsx:1` | 13 | no | `invalidates the previous connection settings when switching with a dirty edit` `sources/betterdb/apps/web/src/pages/ScalingReadiness.test.tsx:208` |
| `sources/betterdb/packages/agent-cache/src/__tests__/AgentCache.cluster.integration.test.ts:1` | 8 | no | `invalidateByModel() removes keys across all nodes` `sources/betterdb/packages/agent-cache/src/__tests__/AgentCache.cluster.integration.test.ts:192`; `invalidateByTool() removes keys across all nodes` `sources/betterdb/packages/agent-cache/src/__tests__/AgentCache.cluster.integration.test.ts:244` |
| `sources/betterdb/packages/agent-cache/src/__tests__/LlmCache.test.ts:1` | 19 | no | `deletes corrupt entry and returns miss on invalid JSON` `sources/betterdb/packages/agent-cache/src/__tests__/LlmCache.test.ts:120`; `deletes only matching model entries using batched pipeline` `sources/betterdb/packages/agent-cache/src/__tests__/LlmCache.test.ts:391` |
| `sources/betterdb/packages/agent-cache/src/__tests__/SessionStore.test.ts:1` | 22 | no | `scans and deletes all thread keys` `sources/betterdb/packages/agent-cache/src/__tests__/SessionStore.test.ts:193`; `escapes glob metacharacters in threadId during invalidation scan` `sources/betterdb/packages/agent-cache/src/__tests__/SessionStore.test.ts:215` |
| `sources/betterdb/packages/agent-cache/src/__tests__/ToolCache.test.ts:1` | 24 | no | `deletes corrupt entry and returns miss on invalid JSON` `sources/betterdb/packages/agent-cache/src/__tests__/ToolCache.test.ts:113`; `scans and deletes correct pattern` `sources/betterdb/packages/agent-cache/src/__tests__/ToolCache.test.ts:244`; `escapes glob metacharacters in toolName during invalidation scan` `sources/betterdb/packages/agent-cache/src/__tests__/ToolCache.test.ts:262`; `deletes specific key` `sources/betterdb/packages/agent-cache/src/__tests__/ToolCache.test.ts:281` |
| `sources/betterdb/packages/agent-cache/src/__tests__/adapters.test.ts:1` | 28 | no | `deleteThread() calls session.destroyThread` `sources/betterdb/packages/agent-cache/src/__tests__/adapters.test.ts:683` |
| `sources/betterdb/packages/agent-cache/src/__tests__/analytics.test.ts:1` | 17 | no | `emits a snapshot fire-and-forget on frozen serverless (AWS Lambda)` `sources/betterdb/packages/agent-cache/src/__tests__/analytics.test.ts:268` |
| `sources/betterdb/packages/agent-cache/src/__tests__/discovery.test.ts:1` | 13 | no | `publishes tool_ttl_adjust, invalidate_by_tool, tool_effectiveness capabilities` `sources/betterdb/packages/agent-cache/src/__tests__/discovery.test.ts:125`; `stop({ deleteHeartbeat: true }) deletes the heartbeat key without touching the registry` `sources/betterdb/packages/agent-cache/src/__tests__/discovery.test.ts:333` |
| `sources/betterdb/packages/agent-cache-py/tests/test_discovery.py:1` | 19 | no | `test_stop_delete_heartbeat_removes_heartbeat_key_not_registry` `sources/betterdb/packages/agent-cache-py/tests/test_discovery.py:354`; `test_stop_no_delete_heartbeat_leaves_key_intact` `sources/betterdb/packages/agent-cache-py/tests/test_discovery.py:370` |
| `sources/betterdb/packages/agent-cache-py/tests/test_llm_cache.py:1` | 16 | no | `test_check_corrupt_entry_returns_miss_and_deletes` `sources/betterdb/packages/agent-cache-py/tests/test_llm_cache.py:81`; `test_clear_deletes_only_llm_keys` `sources/betterdb/packages/agent-cache-py/tests/test_llm_cache.py:192`; `test_invalidate_by_model_deletes_matching_entries` `sources/betterdb/packages/agent-cache-py/tests/test_llm_cache.py:220`; `test_invalidate_by_model_no_keys_returns_zero` `sources/betterdb/packages/agent-cache-py/tests/test_llm_cache.py:239` |
| `sources/betterdb/packages/agent-cache-py/tests/test_session_store.py:1` | 32 | no | `test_delete_returns_true_when_key_existed` `sources/betterdb/packages/agent-cache-py/tests/test_session_store.py:165`; `test_delete_returns_false_when_key_absent` `sources/betterdb/packages/agent-cache-py/tests/test_session_store.py:175`; `test_destroy_thread_deletes_all_keys_and_returns_count` `sources/betterdb/packages/agent-cache-py/tests/test_session_store.py:341` |
| `sources/betterdb/packages/agent-cache-py/tests/test_tool_cache.py:1` | 22 | no | `test_check_corrupt_returns_miss_and_deletes` `sources/betterdb/packages/agent-cache-py/tests/test_tool_cache.py:76`; `test_invalidate_specific_entry` `sources/betterdb/packages/agent-cache-py/tests/test_tool_cache.py:150`; `test_invalidate_returns_false_when_key_absent` `sources/betterdb/packages/agent-cache-py/tests/test_tool_cache.py:160`; `test_refresh_policies_removes_deleted_entries` `sources/betterdb/packages/agent-cache-py/tests/test_tool_cache.py:223` |
| `sources/betterdb/packages/agent-memory/src/__tests__/MemoryStore.consolidate.test.ts:1` | 12 | no | `summarizes matching candidates, writes a summary, deletes sources, returns counts` `sources/betterdb/packages/agent-memory/src/__tests__/MemoryStore.consolidate.test.ts:62`; `keeps sources when deleteSources is false` `sources/betterdb/packages/agent-memory/src/__tests__/MemoryStore.consolidate.test.ts:158`; `defaults summaryImportance to 0.7 and deletes sources by default` `sources/betterdb/packages/agent-memory/src/__tests__/MemoryStore.consolidate.test.ts:211` |
| `sources/betterdb/packages/agent-memory/src/__tests__/MemoryStore.consolidateFacts.test.ts:1` | 23 | no | `writes fact memories additively without deleting the source memories` `sources/betterdb/packages/agent-memory/src/__tests__/MemoryStore.consolidateFacts.test.ts:188`; `supersedes a stored fact: a newer dated statement deletes the prior memory and writes the new one` `sources/betterdb/packages/agent-memory/src/__tests__/MemoryStore.consolidateFacts.test.ts:465`; `retracts a stored fact across runs: a tombstone deletes the prior memory and writes nothing` `sources/betterdb/packages/agent-memory/src/__tests__/MemoryStore.consolidateFacts.test.ts:593` |
| `sources/betterdb/packages/agent-memory/src/__tests__/MemoryStore.eviction.test.ts:1` | 13 | no | `counts only actual removals when the index lists already-deleted keys` `sources/betterdb/packages/agent-memory/src/__tests__/MemoryStore.eviction.test.ts:110` |
| `sources/betterdb/packages/agent-memory/src/__tests__/MemoryStore.forget.test.ts:1` | 8 | no | `throws when no scope field or tag is given (prevents mass delete)` `sources/betterdb/packages/agent-memory/src/__tests__/MemoryStore.forget.test.ts:67`; `warns when the batch safety cap is hit so a partial delete is not silent` `sources/betterdb/packages/agent-memory/src/__tests__/MemoryStore.forget.test.ts:105` |
| `sources/betterdb/packages/agent-memory/src/__tests__/MemoryStore.reinforce.test.ts:1` | 4 | no | `does not resurrect a recalled hit whose hash was already deleted` `sources/betterdb/packages/agent-memory/src/__tests__/MemoryStore.reinforce.test.ts:44` |
| `sources/betterdb/packages/agent-memory/src/__tests__/MemoryStore.spans.test.ts:1` | 3 | no | `emits a consolidate span with candidate/created/deleted counts` `sources/betterdb/packages/agent-memory/src/__tests__/MemoryStore.spans.test.ts:101` |
| `sources/betterdb/packages/agent-memory/src/__tests__/discovery.test.ts:1` | 9 | no | `deletes the heartbeat key on stop when asked` `sources/betterdb/packages/agent-memory/src/__tests__/discovery.test.ts:89`; `waits for an in-flight heartbeat tick before deleting, so no write lands after stop` `sources/betterdb/packages/agent-memory/src/__tests__/discovery.test.ts:131` |
| `sources/betterdb/packages/agent-memory/src/__tests__/reconcileFacts.test.ts:1` | 17 | no | `retracts on an empty-string-date tombstone (dateless, still deletes)` `sources/betterdb/packages/agent-memory/src/__tests__/reconcileFacts.test.ts:122`; `deletes on a tombstone but noops on a stale (older-dated) tombstone` `sources/betterdb/packages/agent-memory/src/__tests__/reconcileFacts.test.ts:131`; `produces the curated set after add/update/delete` `sources/betterdb/packages/agent-memory/src/__tests__/reconcileFacts.test.ts:201` |
| `sources/betterdb/packages/agent-memory-py/tests/test_discovery.py:1` | 9 | no | `test_deletes_heartbeat_key_on_stop_when_asked` `sources/betterdb/packages/agent-memory-py/tests/test_discovery.py:99`; `test_waits_for_in_flight_tick_before_deleting` `sources/betterdb/packages/agent-memory-py/tests/test_discovery.py:136` |
| `sources/betterdb/packages/agent-memory-py/tests/test_memory_store_consolidate.py:1` | 11 | no | `test_summarizes_writes_summary_deletes_sources_returns_counts` `sources/betterdb/packages/agent-memory-py/tests/test_memory_store_consolidate.py:59`; `test_keeps_sources_when_delete_sources_false` `sources/betterdb/packages/agent-memory-py/tests/test_memory_store_consolidate.py:133`; `test_defaults_summary_importance_to_point_seven_and_deletes_by_default` `sources/betterdb/packages/agent-memory-py/tests/test_memory_store_consolidate.py:180` |
| `sources/betterdb/packages/agent-memory-py/tests/test_memory_store_consolidate_facts.py:1` | 23 | no | `test_writes_fact_memories_additively_without_deleting_sources` `sources/betterdb/packages/agent-memory-py/tests/test_memory_store_consolidate_facts.py:142`; `test_supersedes_a_stored_fact_deleting_prior_and_writing_new` `sources/betterdb/packages/agent-memory-py/tests/test_memory_store_consolidate_facts.py:363` |
| `sources/betterdb/packages/agent-memory-py/tests/test_memory_store_forget.py:1` | 8 | no | `test_forget_dels_hash_and_reports_it_existed` `sources/betterdb/packages/agent-memory-py/tests/test_memory_store_forget.py:15`; `test_forget_returns_false_when_absent` `sources/betterdb/packages/agent-memory-py/tests/test_memory_store_forget.py:23`; `test_forget_by_scope_searches_dels_matches_returns_count` `sources/betterdb/packages/agent-memory-py/tests/test_memory_store_forget.py:30`; `test_forget_by_scope_escapes_glob_chars` `sources/betterdb/packages/agent-memory-py/tests/test_memory_store_forget.py:56`; `test_forget_by_scope_throws_without_scope_or_tag` `sources/betterdb/packages/agent-memory-py/tests/test_memory_store_forget.py:67`; `test_forget_by_scope_returns_zero_and_no_del_when_nothing_matches` `sources/betterdb/packages/agent-memory-py/tests/test_memory_store_forget.py:74`; `test_forget_by_scope_paginates_across_batches` `sources/betterdb/packages/agent-memory-py/tests/test_memory_store_forget.py:82`; `test_forget_by_scope_warns_when_batch_safety_cap_is_hit` `sources/betterdb/packages/agent-memory-py/tests/test_memory_store_forget.py:109` |
| `sources/betterdb/packages/agent-memory-py/tests/test_memory_store_reinforce.py:1` | 4 | no | `test_does_not_resurrect_recalled_hit_whose_hash_was_deleted` `sources/betterdb/packages/agent-memory-py/tests/test_memory_store_reinforce.py:50` |
| `sources/betterdb/packages/agent-memory-py/tests/test_reconcile_facts.py:1` | 17 | no | `test_deletes_on_a_tombstone_but_noops_on_a_stale_older_dated_tombstone` `sources/betterdb/packages/agent-memory-py/tests/test_reconcile_facts.py:112`; `test_apply_ops_produces_the_curated_set_after_add_update_delete` `sources/betterdb/packages/agent-memory-py/tests/test_reconcile_facts.py:170` |
| `sources/betterdb/packages/hermes-memory-py/tests/conftest.py:1` | 0 | yes | none |
| `sources/betterdb/packages/hermes-memory-py/tests/test_provider.py:1` | 44 | yes | `test_tool_remember_recall_forget` `sources/betterdb/packages/hermes-memory-py/tests/test_provider.py:391`; `test_tool_forget_refuses_foreign_agent` `sources/betterdb/packages/hermes-memory-py/tests/test_provider.py:738`; `test_tool_forget_refuses_foreign_namespace` `sources/betterdb/packages/hermes-memory-py/tests/test_provider.py:746`; `test_tool_forget_missing_id_reports_not_found` `sources/betterdb/packages/hermes-memory-py/tests/test_provider.py:754` |
| `sources/betterdb/packages/retrieval/src/__tests__/Retriever.integration.test.ts:1` | 9 | no | `removes a document with delete` `sources/betterdb/packages/retrieval/src/__tests__/Retriever.integration.test.ts:182` |
| `sources/betterdb/packages/retrieval-py/tests/test_discovery.py:1` | 5 | no | `test_unregister_deletes_own_marker` `sources/betterdb/packages/retrieval-py/tests/test_discovery.py:64` |
| `sources/betterdb/packages/retrieval-py/tests/test_upsert_delete.py:1` | 14 | no | `test_delete_dels_derived_keys` `sources/betterdb/packages/retrieval-py/tests/test_upsert_delete.py:244`; `test_delete_empty_list` `sources/betterdb/packages/retrieval-py/tests/test_upsert_delete.py:253` |
| `sources/betterdb/packages/semantic-cache/src/__tests__/SemanticCache.cluster.integration.test.ts:1` | 5 | no | `invalidate deletes entries scattered across every shard` `sources/betterdb/packages/semantic-cache/src/__tests__/SemanticCache.cluster.integration.test.ts:162` |
| `sources/betterdb/packages/semantic-cache/src/__tests__/SemanticCache.integration.test.ts:1` | 13 | no | `invalidate() deletes matching entries and reports truncation status` `sources/betterdb/packages/semantic-cache/src/__tests__/SemanticCache.integration.test.ts:185` |
| `sources/betterdb/packages/semantic-cache/src/__tests__/cluster-deletes.test.ts:1` | 5 | no | `deletes scanned keys one at a time instead of in one multi-key DEL` `sources/betterdb/packages/semantic-cache/src/__tests__/cluster-deletes.test.ts:102` |
| `sources/betterdb/packages/semantic-cache/src/__tests__/discovery.test.ts:1` | 21 | no | `stop({ deleteHeartbeat: true }) deletes the heartbeat key` `sources/betterdb/packages/semantic-cache/src/__tests__/discovery.test.ts:416`; `stop({ deleteHeartbeat: false }) leaves the heartbeat key to expire naturally` `sources/betterdb/packages/semantic-cache/src/__tests__/discovery.test.ts:432` |
| `sources/betterdb/packages/semantic-cache/src/__tests__/features.test.ts:1` | 16 | no | `calls invalidate with @model:{...} filter` `sources/betterdb/packages/semantic-cache/src/__tests__/features.test.ts:93`; `calls invalidate with @category:{...} filter` `sources/betterdb/packages/semantic-cache/src/__tests__/features.test.ts:121` |
| `sources/betterdb/packages/semantic-cache-py/tests/adapters/test_langgraph.py:1` | 3 | no | `test_adelete_removes_entries_across_multiple_pages` `sources/betterdb/packages/semantic-cache-py/tests/adapters/test_langgraph.py:41`; `test_adelete_raises_on_valkey_error` `sources/betterdb/packages/semantic-cache-py/tests/adapters/test_langgraph.py:88`; `test_adelete_empty_namespace_does_nothing` `sources/betterdb/packages/semantic-cache-py/tests/adapters/test_langgraph.py:99` |
| `sources/betterdb/packages/semantic-cache-py/tests/test_discovery.py:1` | 21 | no | `test_stop_delete_heartbeat_true_deletes_key` `sources/betterdb/packages/semantic-cache-py/tests/test_discovery.py:344`; `test_stop_delete_heartbeat_false_leaves_key` `sources/betterdb/packages/semantic-cache-py/tests/test_discovery.py:355` |
| `sources/betterdb/packages/semantic-cache-py/tests/test_features.py:1` | 14 | no | `test_invalidate_by_model_calls_ft_search_with_model_filter` `sources/betterdb/packages/semantic-cache-py/tests/test_features.py:43`; `test_invalidate_by_category_calls_ft_search_with_category_filter` `sources/betterdb/packages/semantic-cache-py/tests/test_features.py:57` |
| `sources/betterdb/packages/semantic-cache-py/tests/test_semantic_cache.py:1` | 22 | no | `test_flush_drops_index_and_deletes_keys` `sources/betterdb/packages/semantic-cache-py/tests/test_semantic_cache.py:309` |
| `sources/betterdb/proprietary/anomaly-detection/__tests__/anomaly.service.spec.ts:1` | 282 | no | `re-emits after a dismissed event is purged from the cache, outage still live` `sources/betterdb/proprietary/anomaly-detection/__tests__/anomaly.service.spec.ts:4078`; `still emits the unchanged stale-twin WARNING with FORGET advice` `sources/betterdb/proprietary/anomaly-detection/__tests__/anomaly.service.spec.ts:4795` |
| `sources/betterdb/proprietary/anomaly-detection/__tests__/cob-pressure-detector.spec.ts:1` | 32 | no | `forgets pressure older than the coincidence memory window` `sources/betterdb/proprietary/anomaly-detection/__tests__/cob-pressure-detector.spec.ts:425` |
| `sources/betterdb/proprietary/anomaly-detection/__tests__/ghost-membership-history.spec.ts:1` | 14 | no | `stays silent when the id returns faster than the FORGET blacklist window` `sources/betterdb/proprietary/anomaly-detection/__tests__/ghost-membership-history.spec.ts:131`; `treats a wiped view as a local CLUSTER RESET, not a mass FORGET` `sources/betterdb/proprietary/anomaly-detection/__tests__/ghost-membership-history.spec.ts:150`; `forgets a departure once the retention window lapses, so a later return is a fresh join` `sources/betterdb/proprietary/anomaly-detection/__tests__/ghost-membership-history.spec.ts:172` |
| `sources/betterdb/proprietary/anomaly-detection/__tests__/orphaned-slot-keys-detector.spec.ts:1` | 13 | no | `does not fire on a delete/expiry-heavy keyspace with no leak` `sources/betterdb/proprietary/anomaly-detection/__tests__/orphaned-slot-keys-detector.spec.ts:93` |
| `sources/betterdb/proprietary/bulk-delete/__tests__/bulk-delete-engine.spec.ts:1` | 14 | no | `rejects a catch-all pattern unless confirmDeleteAll is set` `sources/betterdb/proprietary/bulk-delete/__tests__/bulk-delete-engine.spec.ts:60`; `execute unlinks each batch and totals the deletions` `sources/betterdb/proprietary/bulk-delete/__tests__/bulk-delete-engine.spec.ts:116`; `stops at maxKeys and marks the result truncated (never over-deletes)` `sources/betterdb/proprietary/bulk-delete/__tests__/bulk-delete-engine.spec.ts:131` |
| `sources/betterdb/proprietary/bulk-delete/__tests__/bulk-delete.service.spec.ts:1` | 12 | no | `preview runs as a dry-run job and reports matches without deleting` `sources/betterdb/proprietary/bulk-delete/__tests__/bulk-delete.service.spec.ts:85`; `execute deletes matching keys and persists an audit record` `sources/betterdb/proprietary/bulk-delete/__tests__/bulk-delete.service.spec.ts:103` |
| `sources/betterdb/proprietary/bulk-delete/__tests__/valkey-scan-target.spec.ts:1` | 5 | no | `deletes per key via a pipeline (never a cross-slot variadic UNLINK)` `sources/betterdb/proprietary/bulk-delete/__tests__/valkey-scan-target.spec.ts:48` |
| `sources/betterdb/proprietary/cache-proposals/__tests__/cache-apply.dispatcher.spec.ts:1` | 16 | no | `semantic invalidate parses FT.SEARCH RETURN 0 response without skipping keys` `sources/betterdb/proprietary/cache-proposals/__tests__/cache-apply.dispatcher.spec.ts:154`; `agent invalidate by key_prefix scopes the SCAN pattern to the cache namespace` `sources/betterdb/proprietary/cache-proposals/__tests__/cache-apply.dispatcher.spec.ts:172`; `agent invalidate scans cache.prefix namespace, not cache.name` `sources/betterdb/proprietary/cache-proposals/__tests__/cache-apply.dispatcher.spec.ts:204`; `semantic invalidate queries {prefix}:idx , the index SemanticCache creates` `sources/betterdb/proprietary/cache-proposals/__tests__/cache-apply.dispatcher.spec.ts:237`; `semantic invalidate forwards filter_expression from the payload verbatim to FT.SEARCH` `sources/betterdb/proprietary/cache-proposals/__tests__/cache-apply.dispatcher.spec.ts:257`; `semantic invalidate with a non-default prefix queries {prefix}:idx` `sources/betterdb/proprietary/cache-proposals/__tests__/cache-apply.dispatcher.spec.ts:275` |
| `sources/betterdb/proprietary/cache-proposals/__tests__/cache-approval.service.spec.ts:1` | 13 | no | `rejects edits on invalidate proposals` `sources/betterdb/proprietary/cache-proposals/__tests__/cache-approval.service.spec.ts:219` |
| `sources/betterdb/proprietary/cloud-auth/demo-mode.guard.spec.ts:1` | 8 | no | `throws NotFoundException on demo host DELETE /monitor/sessions/:id` `sources/betterdb/proprietary/cloud-auth/demo-mode.guard.spec.ts:57` |
| `sources/betterdb/proprietary/memory-proposals/__tests__/apply-sweep-race.spec.ts:1` | 7 | no | `abandons a forget that outruns the bound` `sources/betterdb/proprietary/memory-proposals/__tests__/apply-sweep-race.spec.ts:128`; `leaves no timer behind when the forget finishes first` `sources/betterdb/proprietary/memory-proposals/__tests__/apply-sweep-race.spec.ts:141` |
| `sources/betterdb/proprietary/memory-proposals/__tests__/memory-apply.dispatcher.spec.ts:1` | 4 | no | `forgets a single memory by id` `sources/betterdb/proprietary/memory-proposals/__tests__/memory-apply.dispatcher.spec.ts:50`; `does not leave an unhandled rejection when an abandoned forget fails later` `sources/betterdb/proprietary/memory-proposals/__tests__/memory-apply.dispatcher.spec.ts:70`; `forgets by scope and tags, returning the removed count` `sources/betterdb/proprietary/memory-proposals/__tests__/memory-apply.dispatcher.spec.ts:105` |
| `sources/betterdb/proprietary/memory-proposals/__tests__/memory-proposal.service.spec.ts:1` | 15 | no | `does not mark failed when the forget succeeds but finalize bookkeeping throws` `sources/betterdb/proprietary/memory-proposals/__tests__/memory-proposal.service.spec.ts:179`; `claims the proposal so a concurrent approve cannot dispatch the forget twice` `sources/betterdb/proprietary/memory-proposals/__tests__/memory-proposal.service.spec.ts:205` |

## 7. Corrections to B2

Old values below are copied from the triage row (TRIAGE.md), not reconstructed from memory. A refinement narrows an over-broad field; it need not reverse its original value. Counts are distinct field paths, not the number of factual errors.

- `sec.telemetry.v`: `"Hermes adapter defaults telemetry to false; inspect referenced configuration and SDK for the opt-in path."` → Adapter analytics defaults false; SDK receives that value. Evidence: `sources/betterdb/packages/hermes-memory-py/betterdb_hermes_memory/__init__.py:158`, `sources/betterdb/packages/hermes-memory-py/betterdb_hermes_memory/__init__.py:351`.
- `engine_delete.scope`: `"Named deletion/retraction endpoint or method, not a guarantee of physical erasure. Its reach may overlap an agent tool; lack of a native tool is counted separately."` → `yes`: betterdb_forget reads the record, verifies agent_id and effective namespace, then calls store.forget (`sources/betterdb/packages/hermes-memory-py/betterdb_hermes_memory/__init__.py:653`; `sources/betterdb/packages/hermes-memory-py/betterdb_hermes_memory/__init__.py:660`). Engine `forget(id)` executes DEL on one memory hash; `forget_by_scope` searches/deletes matching thread/agent/namespace/tag hashes in bounded batches (`sources/betterdb/packages/agent-memory-py/betterdb_agent_memory/memory_store.py:733`; `sources/betterdb/packages/agent-memory-py/betterdb_agent_memory/memory_store.py:777`). Cap exhaustion emits a warning rather than proving full removal (`sources/betterdb/packages/agent-memory-py/betterdb_agent_memory/memory_store.py:787`). `partial`: content/vector hash is deleted, but adapter's cached recall block is not cleared by this handler, and no host-copy cleanup exists. Evidence: `sources/betterdb/packages/hermes-memory-py/betterdb_hermes_memory/__init__.py:653`, `sources/betterdb/packages/hermes-memory-py/betterdb_hermes_memory/__init__.py:660`, `sources/betterdb/packages/agent-memory-py/betterdb_agent_memory/memory_store.py:733`, `sources/betterdb/packages/agent-memory-py/betterdb_agent_memory/memory_store.py:777`, `sources/betterdb/packages/agent-memory-py/betterdb_agent_memory/memory_store.py:787`.
- `context.mechanism.v`: `"Source expressions below identify retrieval, cached/static returns, lifecycle capture, and limits. A hook definition is not a guarantee it emits nonempty context each turn; branches and configuration apply."` → Prefetch consumes and clears the cached block only for the matching session (`sources/betterdb/packages/hermes-memory-py/betterdb_hermes_memory/__init__.py:466`). Queue-prefetch assembly/limits are quoted below. Explicit recall defaults k=8 and returns JSON IDs/content/scores (`sources/betterdb/packages/hermes-memory-py/betterdb_hermes_memory/__init__.py:638`; `sources/betterdb/packages/hermes-memory-py/betterdb_hermes_memory/__init__.py:645`). Do not describe this as synchronous fresh search every turn. Evidence: `sources/betterdb/packages/hermes-memory-py/betterdb_hermes_memory/__init__.py:466`, `sources/betterdb/packages/hermes-memory-py/betterdb_hermes_memory/__init__.py:638`, `sources/betterdb/packages/hermes-memory-py/betterdb_hermes_memory/__init__.py:645`.
- `repro.decls`: `["package.json:3", "package.json:22", "package.json:23", "package.json:26", "package.json:27", "package.json:53", "package.json:60", "package.json:74", "package.json:75", "package.json:76", "package.json:79", "package.json:82", "package.json:85", "package.json:86", "package.json:87", "package.json:89", "package.json:90", "package.json:91", "package.json:92", "package.json:93", "package.json:94", "package.json:96", "package.json:99", "package.json:101", "package.json:102", "package.json:103", "package.json:104", "package.json:105", "package.json:106", "package.json:107", "package.json:108", "package.json:109", "package.json:110", "package.json:112", "package.json:113", "package.json:115", "package.json:116", "package.json:118", "package.json:121", "package.json:123", "package.json:125", "package.json:126", "package.json:127", "package.json:128", "packages/hermes-memory-py/pyproject.toml:7", "packages/hermes-memory-py/pyproject.toml:12", "packages/hermes-memory-py/pyproject.toml:14", "packages/hermes-memory-py/pyproject.toml:22", "packages/hermes-memory-py/pyproject.toml:23", "packages/hermes-memory-py/pyproject.toml:24", "packages/hermes-memory-py/pyproject.toml:25", "packages/hermes-memory-py/pyproject.toml:29", "packages/hermes-memory-py/pyproject.toml:30", "packages/hermes-memory-py/pyproject.toml:31", "packages/hermes-memory-py/pyproject.toml:33", "packages/hermes-memory-py/pyproject.toml:35"]` → `range`: betterdb-agent-memory>=0.7.0,<0.8.0, valkey>=6.0,<7.0, httpx>=0.27,<1.0 (`sources/betterdb/packages/hermes-memory-py/pyproject.toml:23`). Server/module version is outside those Python package pins. Evidence: `sources/betterdb/packages/hermes-memory-py/pyproject.toml:23`.
- `tests.count`: `6744` → 44 adapter-related declarations using the stated parser/scope; B2 counted a different, repository-wide inventory. This is a scope refinement, not a count of executed or passing tests. Adapter tests in packages/hermes-memory-py/tests/test_provider.py cover the tool/ownership contract; SDK tests cover DEL/scoped deletion. Counted independently below. Adapter tool-cycle test includes forget; foreign memory refusal is separately tested. (`sources/betterdb/packages/hermes-memory-py/tests/test_provider.py:391`). Evidence: `sources/betterdb/packages/hermes-memory-py/tests/test_provider.py:391`.

MATRIX counts these 5 explicit field corrections/refinements. Sections 1-6 provide the body and test inventories. Test-name candidates are not passing tests or measured coverage.
