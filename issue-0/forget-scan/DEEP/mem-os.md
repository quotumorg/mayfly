# mem-os: B3 static correctness review

Snapshot `af8fa31e01078e4d5e38210b61acdf90f6d70581` (MANIFEST.json row `mem-os`). No plugin execution or passing-test claims.

## 1. Architecture

`mcp`, not MemoryProvider. Plugin launches npx @mem-os/sdk@1.6.26 mcp as stdio subprocess (`sources/mem-os/plugin/.mcp.json:4`). TS MCP server registers tools over MemOS engine; Python adapters in the same repository are not the selected entrypoint (`sources/mem-os/src/mcp.ts:266`).

Hook/body inventory (no-op/fallback declarations are not active behavior):

| callable | declaration | first executable body line |
|---|---|---|

## 2. Forget paths and derived copies

`yes`: memos_forget callback awaits memos.forget(id) (`sources/mem-os/src/mcp.ts:280`). Engine forget calls storage.deleteNode, removes in-memory graph node, emits node:deleted and invalidates search cache (`sources/mem-os/src/memory.ts:796`). SQLite deleteNode and edge/embedding/bulk-delete bodies are in the ledger. `partial` for all-copy criterion: live graph/search cleanup is explicit, but generated summaries stored as independent nodes, exported copies and Hermes dialogue are not globally purged by one node ID. No claim that all model-derived content disappears. SQLite enables foreign keys; edge/embedding child tables use ON DELETE CASCADE, and FTS has an AFTER DELETE trigger (`sources/mem-os/src/storage/sqlite.ts:271`; `sources/mem-os/src/storage/sqlite.ts:301`; `sources/mem-os/src/storage/sqlite.ts:318`; `sources/mem-os/src/storage/sqlite.ts:386`). These establish cleanup of those linked copies.

Host state.db is independent (`hermes-agent/hermes_state.py:181`). No reviewed adapter forget body performs a general purge of that database or built-in MEMORY.md/USER.md. Prompt/tool-result copies and independent source transcripts remain separate; source operation bodies below establish the scope of each plugin mutation.

<details>
<summary>Deletion, invalidation, archive and reset body ledger</summary>

Public routes, SDK/CLI dispatch and internal storage operations are distinguished by their source. Cache invalidation and management-object deletion are not additional fact-forget tools. A stub is not working deletion. Call-through means the downstream body still determines erasure. HTTP prefixes/selectors must be read with route registration.

**forget**, body `sources/mem-os/memos/adapters/crewai.py:201`; implementation/call-through.

- `sources/mem-os/memos/adapters/crewai.py:191` `def forget(self, memory_id: str) -> bool:`
- `sources/mem-os/memos/adapters/crewai.py:201` `return self._memos_forget(memory_id)`

**_memos_forget**, body `sources/mem-os/memos/adapters/crewai.py:240`; implementation/call-through.

- `sources/mem-os/memos/adapters/crewai.py:238` `def _memos_forget(self, memory_id: str) -> bool:`
- `sources/mem-os/memos/adapters/crewai.py:240` `try:`

**forget**, body `sources/mem-os/memos/adapters/crewai.py:376`; implementation/call-through.

- `sources/mem-os/memos/adapters/crewai.py:366` `def forget(self, memory_id: str) -> bool:`
- `sources/mem-os/memos/adapters/crewai.py:376` `return self._tool.forget(memory_id)`

**clear**, body `sources/mem-os/memos/adapters/langchain.py:197`; implementation/call-through.

- `sources/mem-os/memos/adapters/langchain.py:195` `def clear(self) -> None:`
- `sources/mem-os/memos/adapters/langchain.py:197` `self._conversation_buffer.clear()`

**forget**, body `sources/mem-os/memos/adapters/langchain.py:239`; implementation/call-through.

- `sources/mem-os/memos/adapters/langchain.py:229` `def forget(self, memory_id: str) -> bool:`
- `sources/mem-os/memos/adapters/langchain.py:239` `try:`

**forget**, body `sources/mem-os/memos/adapters/ollama.py:219`; implementation/call-through.

- `sources/mem-os/memos/adapters/ollama.py:209` `def forget(self, memory_id: str) -> bool:`
- `sources/mem-os/memos/adapters/ollama.py:219` `self._assert_init()`

**_memos_forget**, body `sources/mem-os/memos/adapters/ollama.py:315`; implementation/call-through.

- `sources/mem-os/memos/adapters/ollama.py:313` `def _memos_forget(self, memory_id: str) -> bool:`
- `sources/mem-os/memos/adapters/ollama.py:315` `try:`

**forget_memory**, body `sources/mem-os/memos/server/routes.py:204`; implementation/call-through.

- `sources/mem-os/memos/server/routes.py:201` `@router.post("/forget", summary="Forget (delete) a memory")`
- `sources/mem-os/memos/server/routes.py:202` `async def forget_memory(req: ForgetRequest):`
- `sources/mem-os/memos/server/routes.py:204` `result = _rpc_call("forget", {"id": req.id})`

**operation and route call sites**, body `sources/mem-os/src/adapters/anthropic.ts:410`; body evidence.

- `sources/mem-os/src/adapters/anthropic.ts:409` `*/`
- `sources/mem-os/src/adapters/anthropic.ts:410` `async forget(memoryId: string): Promise<boolean> {`
- `sources/mem-os/src/adapters/anthropic.ts:411` `return this.memos.forget(memoryId);`
- `sources/mem-os/src/adapters/anthropic.ts:412` `}`
- `sources/mem-os/src/adapters/anthropic.ts:413` ``
- `sources/mem-os/src/adapters/anthropic.ts:414` `/**`
- `sources/mem-os/src/adapters/anthropic.ts:415` `* Remove all memories and edges from the underlying MemOS instance.`

**operation and route call sites**, body `sources/mem-os/src/adapters/openai.ts:391`; body evidence.

- `sources/mem-os/src/adapters/openai.ts:390` `*/`
- `sources/mem-os/src/adapters/openai.ts:391` `async forget(memoryId: string): Promise<boolean> {`
- `sources/mem-os/src/adapters/openai.ts:392` `return this.memos.forget(memoryId);`
- `sources/mem-os/src/adapters/openai.ts:393` `}`
- `sources/mem-os/src/adapters/openai.ts:394` ``
- `sources/mem-os/src/adapters/openai.ts:395` `/**`
- `sources/mem-os/src/adapters/openai.ts:396` `* Remove all memories and edges from the underlying MemOS instance.`

**operation and route call sites**, body `sources/mem-os/src/cli-browse.ts:53`; body evidence.

- `sources/mem-os/src/cli-browse.ts:52` `console.log(`
- `sources/mem-os/src/cli-browse.ts:53` `'  page ${page + 1}/${pages} · <n> detail · n next · p prev · s <query> search · s clears · f <n> forget · q quit',`
- `sources/mem-os/src/cli-browse.ts:54` `);`
- `sources/mem-os/src/cli-browse.ts:55` `}`
- `sources/mem-os/src/cli-browse.ts:56` ``
- `sources/mem-os/src/cli-browse.ts:57` `export async function runBrowse(memos: MemOS): Promise<void> {`
- `sources/mem-os/src/cli-browse.ts:114` `await rl.question(`
- `sources/mem-os/src/cli-browse.ts:115` `'  forget ${node.id.slice(0, 8)} "${node.content.slice(0, 40)}"? (y/N) ',`
- `sources/mem-os/src/cli-browse.ts:116` `)`
- `sources/mem-os/src/cli-browse.ts:117` `)`
- `sources/mem-os/src/cli-browse.ts:118` `.trim()`
- `sources/mem-os/src/cli-browse.ts:119` `.toLowerCase();`
- `sources/mem-os/src/cli-browse.ts:120` `if (confirmed === "y" || confirmed === "yes") {`
- `sources/mem-os/src/cli-browse.ts:121` `await memos.forget(node.id);`
- `sources/mem-os/src/cli-browse.ts:122` `console.log("  forgotten.");`
- `sources/mem-os/src/cli-browse.ts:123` `} else {`
- `sources/mem-os/src/cli-browse.ts:124` `console.log("  kept.");`
- `sources/mem-os/src/cli-browse.ts:125` `}`

**operation and route call sites**, body `sources/mem-os/src/cli.ts:275`; body evidence.

- `sources/mem-os/src/cli.ts:274` `"The server speaks MCP 2026-07-28 with a legacy-2025 fallback and",`
- `sources/mem-os/src/cli.ts:275` `"exposes 14 tools: store/search/retrieve/forget/graph/context,",`
- `sources/mem-os/src/cli.ts:276` `"context_pack (token-budgeted injection), search_temporal,",`
- `sources/mem-os/src/cli.ts:277` `"set_validity, supersede, set_trust, extract_facts, diagnostics,",`
- `sources/mem-os/src/cli.ts:278` `"reindex. All data stays in local SQLite.",`
- `sources/mem-os/src/cli.ts:279` `].join("\n"),`
- `sources/mem-os/src/cli.ts:320` `--scope user:alice[,...]) (--pool event|note|procedure)`
- `sources/mem-os/src/cli.ts:321` `forget <id>             Delete a memory by ID`
- `sources/mem-os/src/cli.ts:322` `summarize               Summarize all memories`
- `sources/mem-os/src/cli.ts:323` `graph                   Print the full memory graph (--mermaid for a`
- `sources/mem-os/src/cli.ts:324` `GitHub-renderable Mermaid diagram)`
- `sources/mem-os/src/cli.ts:325` `browse                  Interactive terminal browser (search, inspect,`
- `sources/mem-os/src/cli.ts:326` `forget memories)`
- `sources/mem-os/src/cli.ts:327` `link <src> <dst>        Link two memories`
- `sources/mem-os/src/cli.ts:328` `count                   Show memory count`
- `sources/mem-os/src/cli.ts:329` `tag <id> <tag> [...]    Add tags to a memory`
- `sources/mem-os/src/cli.ts:330` `untag <id> <tag> [...]  Remove tags from a memory`
- `sources/mem-os/src/cli.ts:340` `digest                  Run consolidation now and print a summary`
- `sources/mem-os/src/cli.ts:341` `consolidate             Offline maintenance pass: merge duplicates, archive`
- `sources/mem-os/src/cli.ts:342` `stale memories, supersede decayed ones (kept as`
- `sources/mem-os/src/cli.ts:343` `history), distill cluster summary notes`
- `sources/mem-os/src/cli.ts:344` `(--dry-run, --no-summarize, --no-decay,`
- `sources/mem-os/src/cli.ts:345` `--decay-half-life <days>, --min-retention <score>,`
- `sources/mem-os/src/cli.ts:353` `doctor                  Health-check the store, embedding config and endpoints`
- `sources/mem-os/src/cli.ts:354` `consolidate             Offline maintenance pass: merge duplicates, archive`
- `sources/mem-os/src/cli.ts:355` `stale memories, supersede decayed ones (kept as`
- `sources/mem-os/src/cli.ts:356` `history), distill cluster summary notes`
- `sources/mem-os/src/cli.ts:357` `(--dry-run, --no-summarize, --no-decay,`
- `sources/mem-os/src/cli.ts:358` `--decay-half-life <days>, --min-retention <score>,`
- `sources/mem-os/src/cli.ts:363` `reindex-embeddings      Re-embed all memories with the configured provider`
- `sources/mem-os/src/cli.ts:364` `(--purge-stale deletes vectors from other models first)`
- `sources/mem-os/src/cli.ts:365` `mcp                     Start the MemOS MCP stdio server`
- `sources/mem-os/src/cli.ts:366` `serve                   Start the HTTP server`
- `sources/mem-os/src/cli.ts:367` `trio [--up]             Show (or launch) the full AI Trio: MemOS + LLM-Guardian + Universal-MCP-Toolkit`
- `sources/mem-os/src/cli.ts:368` `help                    Show this help message`
- `sources/mem-os/src/cli.ts:633` ``
- `sources/mem-os/src/cli.ts:634` `case "forget": {`
- `sources/mem-os/src/cli.ts:635` `const id = args[1];`
- `sources/mem-os/src/cli.ts:636` `if (!id) {`
- `sources/mem-os/src/cli.ts:637` `console.error("Error: ID is required.\n  Usage: memos forget <id>");`
- `sources/mem-os/src/cli.ts:638` `process.exit(1);`
- `sources/mem-os/src/cli.ts:639` `}`
- `sources/mem-os/src/cli.ts:640` `const deleted = await memos.forget(id);`
- `sources/mem-os/src/cli.ts:641` `if (jsonFlag) {`
- `sources/mem-os/src/cli.ts:642` `console.log(JSON.stringify({ deleted, id }));`
- `sources/mem-os/src/cli.ts:643` `} else {`
- `sources/mem-os/src/cli.ts:644` `console.log(`
- `sources/mem-os/src/cli.ts:645` `deleted ? 'Forgot memory: ${id}' : 'Memory not found: ${id}',`
- `sources/mem-os/src/cli.ts:646` `);`
- `sources/mem-os/src/cli.ts:647` `}`
- `sources/mem-os/src/cli.ts:648` `break;`
- `sources/mem-os/src/cli.ts:649` `}`
- `sources/mem-os/src/cli.ts:801` `console.log(`
- `sources/mem-os/src/cli.ts:802` `'[${new Date().toISOString()}] merged ${result.merges.length} · archived ${result.moves.length} · decayed ${result.decayed.length} · notes ${result.clusters.length} · ${usage.packTokens} tok injected (${usage.savedPct}% saved) , ${Date.now() - started} ms',`
- `sources/mem-os/src/cli.ts:803` `);`
- `sources/mem-os/src/cli.ts:804` `} catch (error) {`
- `sources/mem-os/src/cli.ts:805` `console.error(`
- `sources/mem-os/src/cli.ts:806` `"consolidation cycle failed:",`
- `sources/mem-os/src/cli.ts:842` `);`
- `sources/mem-os/src/cli.ts:843` `console.log('  archived: ${result.moves.length} memory(ies)');`
- `sources/mem-os/src/cli.ts:844` `console.log(`
- `sources/mem-os/src/cli.ts:845` `'  decayed:  ${result.decayed.length} superseded (kept as history , queryable via searchTemporal)',`
- `sources/mem-os/src/cli.ts:846` `);`
- `sources/mem-os/src/cli.ts:847` `console.log(`
- `sources/mem-os/src/cli.ts:873` `console.log('  superseded:      ${result.decayed.length} decayed');`
- `sources/mem-os/src/cli.ts:874` `console.log('  archived:        ${result.moves.length} moved aside');`
- `sources/mem-os/src/cli.ts:875` `console.log(`
- `sources/mem-os/src/cli.ts:876` `'  new notes:       ${result.clusters.length} cluster summary(ies)',`
- `sources/mem-os/src/cli.ts:877` `);`
- `sources/mem-os/src/cli.ts:878` `console.log(`
- `sources/mem-os/src/cli.ts:1256` `suggestions.push(`
- `sources/mem-os/src/cli.ts:1257` `"Run 'memos reindex-embeddings --purge-stale' after switching models.",`
- `sources/mem-os/src/cli.ts:1258` `);`
- `sources/mem-os/src/cli.ts:1259` `}`
- `sources/mem-os/src/cli.ts:1260` `if (!jsonOut) {`
- `sources/mem-os/src/cli.ts:1261` `console.log(`
- `sources/mem-os/src/cli.ts:1366` `case "reindex-embeddings": {`
- `sources/mem-os/src/cli.ts:1367` `const purgeStale = args.includes("--purge-stale");`
- `sources/mem-os/src/cli.ts:1368` `try {`
- `sources/mem-os/src/cli.ts:1369` `const result = await memos.reindexEmbeddings({ purgeStale });`
- `sources/mem-os/src/cli.ts:1370` `if (jsonFlag) {`
- `sources/mem-os/src/cli.ts:1371` `console.log(JSON.stringify(result));`
- `sources/mem-os/src/cli.ts:1372` `} else {`
- `sources/mem-os/src/cli.ts:1373` `console.log(`
- `sources/mem-os/src/cli.ts:1374` `'Re-embedded ${result.reembedded} memories with "${result.model}"' +`
- `sources/mem-os/src/cli.ts:1375` `'${result.purged ? ', purged ${result.purged} stale vectors' : ""}' +`
- `sources/mem-os/src/cli.ts:1376` `'${result.failed ? ', ${result.failed} FAILED' : ""}.',`
- `sources/mem-os/src/cli.ts:1377` `);`
- `sources/mem-os/src/cli.ts:1378` `}`
- `sources/mem-os/src/cli.ts:1379` `} catch (err) {`

**operation and route call sites**, body `sources/mem-os/src/graph.ts:203`; body evidence.

- `sources/mem-os/src/graph.ts:202` ``
- `sources/mem-os/src/graph.ts:203` `this.nodes.delete(id);`
- `sources/mem-os/src/graph.ts:204` `this.adjacency.delete(id);`
- `sources/mem-os/src/graph.ts:205` `return true;`
- `sources/mem-os/src/graph.ts:206` `}`
- `sources/mem-os/src/graph.ts:207` ``
- `sources/mem-os/src/graph.ts:208` `/**`
- `sources/mem-os/src/graph.ts:253` ``
- `sources/mem-os/src/graph.ts:254` `this.adjacency.get(edge.sourceId)?.delete(id);`
- `sources/mem-os/src/graph.ts:255` `this.adjacency.get(edge.targetId)?.delete(id);`
- `sources/mem-os/src/graph.ts:256` `this.edges.delete(id);`
- `sources/mem-os/src/graph.ts:257` `return true;`
- `sources/mem-os/src/graph.ts:258` `}`
- `sources/mem-os/src/graph.ts:259` ``
- `sources/mem-os/src/graph.ts:260` `/**`

**operation and route call sites**, body `sources/mem-os/src/index.ts:100`; body evidence.

- `sources/mem-os/src/index.ts:99` `DedupeResult,`
- `sources/mem-os/src/index.ts:100` `ArchiveOptions,`
- `sources/mem-os/src/index.ts:101` `ArchiveMove,`
- `sources/mem-os/src/index.ts:102` `ArchiveResult,`
- `sources/mem-os/src/index.ts:103` `ConsolidateOptions,`
- `sources/mem-os/src/index.ts:104` `ConsolidateResult,`
- `sources/mem-os/src/index.ts:105` `ConversationMessage,`
- `sources/mem-os/src/index.ts:106` `ExtractedFact,`

**operation and route call sites**, body `sources/mem-os/src/mcp.ts:267`; body evidence.

- `sources/mem-os/src/mcp.ts:266` `server.registerTool(`
- `sources/mem-os/src/mcp.ts:267` `"memos_forget",`
- `sources/mem-os/src/mcp.ts:268` `{`
- `sources/mem-os/src/mcp.ts:269` `title: "Forget Memory",`
- `sources/mem-os/src/mcp.ts:270` `description: "Delete one memory by ID.",`
- `sources/mem-os/src/mcp.ts:271` `inputSchema: z.object({`
- `sources/mem-os/src/mcp.ts:272` `id: z.string().describe("Memory ID."),`
- `sources/mem-os/src/mcp.ts:273` `}),`
- `sources/mem-os/src/mcp.ts:274` `outputSchema: z.object({`
- `sources/mem-os/src/mcp.ts:275` `id: z.string(),`
- `sources/mem-os/src/mcp.ts:276` `deleted: z.boolean(),`
- `sources/mem-os/src/mcp.ts:277` `}),`
- `sources/mem-os/src/mcp.ts:278` `},`
- `sources/mem-os/src/mcp.ts:279` `async ({ id }) => {`
- `sources/mem-os/src/mcp.ts:280` `const deleted = await memos.forget(id);`
- `sources/mem-os/src/mcp.ts:281` `return {`
- `sources/mem-os/src/mcp.ts:282` `content: [`
- `sources/mem-os/src/mcp.ts:283` `{`
- `sources/mem-os/src/mcp.ts:284` `type: "text" as const,`
- `sources/mem-os/src/mcp.ts:285` `text: deleted ? 'Forgot memory ${id}.' : 'Memory not found: ${id}.',`
- `sources/mem-os/src/mcp.ts:286` `},`
- `sources/mem-os/src/mcp.ts:287` `],`
- `sources/mem-os/src/mcp.ts:288` `structuredContent: { id, deleted },`
- `sources/mem-os/src/mcp.ts:289` `};`
- `sources/mem-os/src/mcp.ts:290` `},`
- `sources/mem-os/src/mcp.ts:291` `);`
- `sources/mem-os/src/mcp.ts:292` ``
- `sources/mem-os/src/mcp.ts:627` `"model. Required after switching models. Can be slow on large " +`
- `sources/mem-os/src/mcp.ts:628` `"stores; purgeStale also deletes vectors from other models.",`
- `sources/mem-os/src/mcp.ts:629` `inputSchema: z.object({`
- `sources/mem-os/src/mcp.ts:630` `purgeStale: z.boolean().optional(),`
- `sources/mem-os/src/mcp.ts:631` `}),`
- `sources/mem-os/src/mcp.ts:632` `outputSchema: z.object({`
- `sources/mem-os/src/mcp.ts:633` `reembedded: z.number(),`
- `sources/mem-os/src/mcp.ts:634` `purged: z.number(),`
- `sources/mem-os/src/mcp.ts:635` `failed: z.number(),`
- `sources/mem-os/src/mcp.ts:636` `model: z.string(),`
- `sources/mem-os/src/mcp.ts:637` `}),`
- `sources/mem-os/src/mcp.ts:638` `},`
- `sources/mem-os/src/mcp.ts:639` `async ({ purgeStale }) => {`
- `sources/mem-os/src/mcp.ts:640` `const summary = await memos.reindexEmbeddings({ purgeStale });`
- `sources/mem-os/src/mcp.ts:641` `return {`
- `sources/mem-os/src/mcp.ts:642` `content: [`
- `sources/mem-os/src/mcp.ts:643` `{`
- `sources/mem-os/src/mcp.ts:644` `type: "text" as const,`
- `sources/mem-os/src/mcp.ts:645` `text: 'Re-embedded ${summary.reembedded} memories (purged ${summary.purged}, failed ${summary.failed}).',`
- `sources/mem-os/src/mcp.ts:646` `},`
- `sources/mem-os/src/mcp.ts:647` `],`
- `sources/mem-os/src/mcp.ts:648` `structuredContent: summary,`
- `sources/mem-os/src/mcp.ts:649` `};`
- `sources/mem-os/src/mcp.ts:658` `"Offline maintenance pass ('dreaming'): merge near-duplicates, " +`
- `sources/mem-os/src/mcp.ts:659` `"archive stale low-importance memories, supersede decayed ones " +`
- `sources/mem-os/src/mcp.ts:660` `"(marked historical , never deleted), and distill cluster summary " +`
- `sources/mem-os/src/mcp.ts:661` `"notes into the note pool. Run periodically or while idle.",`
- `sources/mem-os/src/mcp.ts:662` `inputSchema: z.object({`
- `sources/mem-os/src/mcp.ts:663` `namespace: z.string().optional(),`
- `sources/mem-os/src/mcp.ts:664` `dryRun: z`
- `sources/mem-os/src/mcp.ts:674` `.optional()`
- `sources/mem-os/src/mcp.ts:675` `.describe("Run decay-based forgetting. Default true."),`
- `sources/mem-os/src/mcp.ts:676` `decayHalfLifeDays: z`
- `sources/mem-os/src/mcp.ts:677` `.number()`
- `sources/mem-os/src/mcp.ts:678` `.optional()`
- `sources/mem-os/src/mcp.ts:679` `.describe("Recency half-life in days for the retention score."),`
- `sources/mem-os/src/mcp.ts:706` `});`
- `sources/mem-os/src/mcp.ts:707` `const text = 'Consolidated: ${report.merges.length} merged, ${report.moves.length} archived, ${report.decayed.length} decayed, ${report.clusters.length} note(s)${report.dryRun ? " (dry run , nothing changed)" : ""}.';`
- `sources/mem-os/src/mcp.ts:708` `return {`
- `sources/mem-os/src/mcp.ts:709` `content: [{ type: "text" as const, text }],`
- `sources/mem-os/src/mcp.ts:710` `structuredContent: { report },`
- `sources/mem-os/src/mcp.ts:711` `};`
- `sources/mem-os/src/mcp.ts:1021` `{`
- `sources/mem-os/src/mcp.ts:1022` `name: "memos_forget",`
- `sources/mem-os/src/mcp.ts:1023` `description: "Delete one memory by ID.",`
- `sources/mem-os/src/mcp.ts:1024` `inputSchema: z.toJSONSchema(z.object({ id: z.string() })) as Record<`
- `sources/mem-os/src/mcp.ts:1025` `string,`
- `sources/mem-os/src/mcp.ts:1026` `unknown`
- `sources/mem-os/src/mcp.ts:1027` `>,`
- `sources/mem-os/src/mcp.ts:1028` `outputSchema: z.toJSONSchema(`
- `sources/mem-os/src/mcp.ts:1029` `z.object({ id: z.string(), deleted: z.boolean() }),`
- `sources/mem-os/src/mcp.ts:1030` `) as Record<string, unknown>,`
- `sources/mem-os/src/mcp.ts:1031` `},`
- `sources/mem-os/src/mcp.ts:1032` `{`
- `sources/mem-os/src/mcp.ts:1033` `name: "memos_graph",`
- `sources/mem-os/src/mcp.ts:1169` `"model. Required after switching models. Can be slow on large " +`
- `sources/mem-os/src/mcp.ts:1170` `"stores; purgeStale also deletes vectors from other models.",`
- `sources/mem-os/src/mcp.ts:1171` `inputSchema: z.toJSONSchema(`
- `sources/mem-os/src/mcp.ts:1172` `z.object({ purgeStale: z.boolean().optional() }),`
- `sources/mem-os/src/mcp.ts:1173` `) as Record<string, unknown>,`
- `sources/mem-os/src/mcp.ts:1174` `outputSchema: z.toJSONSchema(`
- `sources/mem-os/src/mcp.ts:1175` `z.object({`
- `sources/mem-os/src/mcp.ts:1176` `reembedded: z.number(),`
- `sources/mem-os/src/mcp.ts:1177` `purged: z.number(),`
- `sources/mem-os/src/mcp.ts:1178` `failed: z.number(),`
- `sources/mem-os/src/mcp.ts:1179` `model: z.string(),`
- `sources/mem-os/src/mcp.ts:1180` `}),`
- `sources/mem-os/src/mcp.ts:1181` `) as Record<string, unknown>,`
- `sources/mem-os/src/mcp.ts:1186` `"Offline maintenance pass ('dreaming'): merge near-duplicates, " +`
- `sources/mem-os/src/mcp.ts:1187` `"archive stale low-importance memories, supersede decayed ones " +`
- `sources/mem-os/src/mcp.ts:1188` `"(marked historical , never deleted), and distill cluster summary " +`
- `sources/mem-os/src/mcp.ts:1189` `"notes into the note pool. Run periodically or while idle.",`
- `sources/mem-os/src/mcp.ts:1190` `inputSchema: z.toJSONSchema(`
- `sources/mem-os/src/mcp.ts:1191` `z.object({`
- `sources/mem-os/src/mcp.ts:1192` `namespace: z.string().optional(),`

**operation and route call sites**, body `sources/mem-os/src/memory.ts:69`; body evidence.

- `sources/mem-os/src/memory.ts:68` `DecaySuperseded,`
- `sources/mem-os/src/memory.ts:69` `DecayForgettingOptions,`
- `sources/mem-os/src/memory.ts:70` `DecayForgetResult,`
- `sources/mem-os/src/memory.ts:71` `ArchiveOptions,`
- `sources/mem-os/src/memory.ts:72` `ArchiveMove,`
- `sources/mem-os/src/memory.ts:73` `ArchiveResult,`
- `sources/mem-os/src/memory.ts:74` `ConsolidateOptions,`
- `sources/mem-os/src/memory.ts:75` `ConsolidateResult,`
- `sources/mem-os/src/memory.ts:76` `SummarizeClusterOptions,`
- `sources/mem-os/src/memory.ts:77` `SummarizeClusterResult,`
- `sources/mem-os/src/memory.ts:601` `this.scheduleEmbedding(node);`
- `sources/mem-os/src/memory.ts:602` `this.invalidateSearchCache();`
- `sources/mem-os/src/memory.ts:603` ``
- `sources/mem-os/src/memory.ts:604` `// Auto-link , batch save edges for performance`
- `sources/mem-os/src/memory.ts:605` `const links: MemoryEdge[] = [];`
- `sources/mem-os/src/memory.ts:606` `if (this.config.autoLinkThreshold > 0) {`
- `sources/mem-os/src/memory.ts:683` `for (const [k, v] of this.searchCache) {`
- `sources/mem-os/src/memory.ts:684` `if (v.expiry <= now) this.searchCache.delete(k);`
- `sources/mem-os/src/memory.ts:685` `}`
- `sources/mem-os/src/memory.ts:686` `while (this.searchCache.size > this.searchCacheMaxEntries) {`
- `sources/mem-os/src/memory.ts:687` `const oldest = this.searchCache.keys().next().value;`
- `sources/mem-os/src/memory.ts:688` `if (oldest === undefined) break;`
- `sources/mem-os/src/memory.ts:689` `this.searchCache.delete(oldest);`
- `sources/mem-os/src/memory.ts:690` `}`
- `sources/mem-os/src/memory.ts:691` `}`
- `sources/mem-os/src/memory.ts:692` `const results =`
- `sources/mem-os/src/memory.ts:693` `filter.query && this.embeddingProvider`
- `sources/mem-os/src/memory.ts:792` `*/`
- `sources/mem-os/src/memory.ts:793` `async forget(id: string): Promise<boolean> {`
- `sources/mem-os/src/memory.ts:794` `this.assertInit();`
- `sources/mem-os/src/memory.ts:795` ``
- `sources/mem-os/src/memory.ts:796` `const deleted = await this.storage.deleteNode(id);`
- `sources/mem-os/src/memory.ts:797` `if (deleted) {`
- `sources/mem-os/src/memory.ts:798` `this.graph.removeNode(id);`
- `sources/mem-os/src/memory.ts:799` `this.emit("node:deleted", id);`
- `sources/mem-os/src/memory.ts:800` `this.invalidateSearchCache();`
- `sources/mem-os/src/memory.ts:801` `}`
- `sources/mem-os/src/memory.ts:802` `return deleted;`
- `sources/mem-os/src/memory.ts:803` `}`
- `sources/mem-os/src/memory.ts:804` ``
- `sources/mem-os/src/memory.ts:805` `/**`
- `sources/mem-os/src/memory.ts:806` `* Generate a summary of all stored memories.`
- `sources/mem-os/src/memory.ts:1447` `this.emit("node:updated", node);`
- `sources/mem-os/src/memory.ts:1448` `this.invalidateSearchCache();`
- `sources/mem-os/src/memory.ts:1449` `}`
- `sources/mem-os/src/memory.ts:1450` `return node;`
- `sources/mem-os/src/memory.ts:1451` `}`
- `sources/mem-os/src/memory.ts:1452` ``
- `sources/mem-os/src/memory.ts:1499` `this.assertInit();`
- `sources/mem-os/src/memory.ts:1500` `await this.storage.deleteAllNodes();`
- `sources/mem-os/src/memory.ts:1501` `this.graph.clear();`
- `sources/mem-os/src/memory.ts:1502` `}`
- `sources/mem-os/src/memory.ts:1503` ``
- `sources/mem-os/src/memory.ts:1504` `/**`
- `sources/mem-os/src/memory.ts:1870` `for (const removedId of removedIds) {`
- `sources/mem-os/src/memory.ts:1871` `await this.forget(removedId);`
- `sources/mem-os/src/memory.ts:1872` `}`
- `sources/mem-os/src/memory.ts:1873` `}`
- `sources/mem-os/src/memory.ts:1874` `}`
- `sources/mem-os/src/memory.ts:1875` ``
- `sources/mem-os/src/memory.ts:1888` `*/`
- `sources/mem-os/src/memory.ts:1889` `async archive(opts: ArchiveOptions = {}): Promise<ArchiveResult> {`
- `sources/mem-os/src/memory.ts:1890` `this.assertInit();`
- `sources/mem-os/src/memory.ts:1891` `const start = Date.now();`
- `sources/mem-os/src/memory.ts:1892` `const afterDays = opts.afterDays ?? 90;`
- `sources/mem-os/src/memory.ts:1893` `const importanceBelow = opts.importanceBelow ?? 0.3;`
- `sources/mem-os/src/memory.ts:1901` `const all = await this.storage.queryNodes({ namespace, limit: 10_000 });`
- `sources/mem-os/src/memory.ts:1902` `const moves: ArchiveMove[] = [];`
- `sources/mem-os/src/memory.ts:1903` `for (const { node } of all) {`
- `sources/mem-os/src/memory.ts:1904` `if (node.importance >= importanceBelow) continue;`
- `sources/mem-os/src/memory.ts:1905` `if (node.lastAccessed > cutoff) continue;`
- `sources/mem-os/src/memory.ts:1906` `if (node.expiresAt !== null) continue; // already on a TTL clock`
- `sources/mem-os/src/memory.ts:1908` `if (!dryRun) {`
- `sources/mem-os/src/memory.ts:1909` `await this.storage.setNodeNamespace(node.id, "archived");`
- `sources/mem-os/src/memory.ts:1910` `this.graph.updateNode({ ...node, namespace: "archived" });`
- `sources/mem-os/src/memory.ts:1911` `}`
- `sources/mem-os/src/memory.ts:1912` `}`
- `sources/mem-os/src/memory.ts:1913` ``
- `sources/mem-os/src/memory.ts:1914` `return { moves, dryRun, durationMs: Date.now() - start };`
- `sources/mem-os/src/memory.ts:2108` `});`
- `sources/mem-os/src/memory.ts:2109` `const archiveResult = await this.archive({`
- `sources/mem-os/src/memory.ts:2110` `afterDays: opts.archiveAfterDays ?? 90,`
- `sources/mem-os/src/memory.ts:2111` `importanceBelow: opts.archiveImportanceBelow ?? 0.3,`
- `sources/mem-os/src/memory.ts:2112` `namespace,`
- `sources/mem-os/src/memory.ts:2113` `dryRun,`
- `sources/mem-os/src/memory.ts:2114` `});`
- `sources/mem-os/src/memory.ts:2115` `const summarizeResult = summarize`
- `sources/mem-os/src/memory.ts:2130` `? { superseded: [] as DecaySuperseded[], dryRun, durationMs: 0 }`
- `sources/mem-os/src/memory.ts:2131` `: await this.forgetByDecay({`
- `sources/mem-os/src/memory.ts:2132` `namespace,`
- `sources/mem-os/src/memory.ts:2133` `halfLifeDays: opts.decayHalfLifeDays,`
- `sources/mem-os/src/memory.ts:2134` `minScore: opts.minRetentionScore,`
- `sources/mem-os/src/memory.ts:2135` `olderThanDays: opts.olderThanDays,`
- `sources/mem-os/src/memory.ts:2140` `merges: dedupeResult.merges,`
- `sources/mem-os/src/memory.ts:2141` `moves: archiveResult.moves,`
- `sources/mem-os/src/memory.ts:2142` `clusters: summarizeResult.clusters,`
- `sources/mem-os/src/memory.ts:2143` `decayed: decayResult.superseded,`
- `sources/mem-os/src/memory.ts:2144` `dryRun,`
- `sources/mem-os/src/memory.ts:2145` `durationMs: Date.now() - start,`
- `sources/mem-os/src/memory.ts:2162` `*/`
- `sources/mem-os/src/memory.ts:2163` `async forgetByDecay(`
- `sources/mem-os/src/memory.ts:2164` `opts: DecayForgettingOptions = {},`
- `sources/mem-os/src/memory.ts:2165` `): Promise<DecayForgetResult> {`
- `sources/mem-os/src/memory.ts:2166` `this.assertInit();`
- `sources/mem-os/src/memory.ts:2167` `const start = Date.now();`
- `sources/mem-os/src/memory.ts:2168` `const namespace = opts.namespace ?? "default";`
- `sources/mem-os/src/memory.ts:2169` `const halfLifeMs = (opts.halfLifeDays ?? 60) * 24 * 60 * 60 * 1000;`
- `sources/mem-os/src/memory.ts:2205` `if (!dryRun && superseded.length > 0) {`
- `sources/mem-os/src/memory.ts:2206` `this.invalidateSearchCache();`
- `sources/mem-os/src/memory.ts:2207` `}`
- `sources/mem-os/src/memory.ts:2208` ``
- `sources/mem-os/src/memory.ts:2209` `return { superseded, dryRun, durationMs: Date.now() - start };`
- `sources/mem-os/src/memory.ts:2210` `}`
- `sources/mem-os/src/memory.ts:2239` `/** Clear the search result cache. Called on any mutation. */`
- `sources/mem-os/src/memory.ts:2240` `private invalidateSearchCache(): void {`
- `sources/mem-os/src/memory.ts:2241` `this.searchCache.clear();`
- `sources/mem-os/src/memory.ts:2242` `}`
- `sources/mem-os/src/memory.ts:2243` ``
- `sources/mem-os/src/memory.ts:2244` `// ---------------------------------------------------------------------------`
- `sources/mem-os/src/memory.ts:2272` `*/`
- `sources/mem-os/src/memory.ts:2273` `async reindexEmbeddings(opts: { purgeStale?: boolean } = {}): Promise<{`
- `sources/mem-os/src/memory.ts:2274` `reembedded: number;`
- `sources/mem-os/src/memory.ts:2275` `purged: number;`
- `sources/mem-os/src/memory.ts:2276` `failed: number;`
- `sources/mem-os/src/memory.ts:2277` `model: string;`
- `sources/mem-os/src/memory.ts:2278` `}> {`
- `sources/mem-os/src/memory.ts:2279` `this.assertInit();`
- `sources/mem-os/src/memory.ts:2286` ``
- `sources/mem-os/src/memory.ts:2287` `let purged = 0;`
- `sources/mem-os/src/memory.ts:2288` `if (opts.purgeStale && this.storage.deleteEmbeddingsByModel) {`
- `sources/mem-os/src/memory.ts:2289` `const counts = this.storage.getEmbeddingModelCounts`
- `sources/mem-os/src/memory.ts:2290` `? await this.storage.getEmbeddingModelCounts()`
- `sources/mem-os/src/memory.ts:2291` `: [];`
- `sources/mem-os/src/memory.ts:2292` `for (const { model, count } of counts) {`
- `sources/mem-os/src/memory.ts:2293` `if (model !== provider.model) {`
- `sources/mem-os/src/memory.ts:2294` `purged += await this.storage.deleteEmbeddingsByModel(model);`
- `sources/mem-os/src/memory.ts:2295` `void count;`
- `sources/mem-os/src/memory.ts:2296` `}`
- `sources/mem-os/src/memory.ts:2297` `}`
- `sources/mem-os/src/memory.ts:2298` `}`
- `sources/mem-os/src/memory.ts:2345` ``
- `sources/mem-os/src/memory.ts:2346` `return { reembedded, purged, failed, model: provider.model };`
- `sources/mem-os/src/memory.ts:2347` `}`
- `sources/mem-os/src/memory.ts:2348` ``
- `sources/mem-os/src/memory.ts:2349` `/**`
- `sources/mem-os/src/memory.ts:2350` `* Per-node embedding status, plus the queue's running/pending counters.`
- `sources/mem-os/src/memory.ts:2864` `const victim = nodes[0];`
- `sources/mem-os/src/memory.ts:2865` `await this.forget(victim.id);`
- `sources/mem-os/src/memory.ts:2866` `this.emit("eviction", victim);`
- `sources/mem-os/src/memory.ts:2867` `}`
- `sources/mem-os/src/memory.ts:2868` `}`
- `sources/mem-os/src/memory.ts:2869` ``
- `sources/mem-os/src/memory.ts:2961` `if (updated) this.graph.addNode(updated);`
- `sources/mem-os/src/memory.ts:2962` `this.invalidateSearchCache();`
- `sources/mem-os/src/memory.ts:2963` `this.emit("memory:reinforced", {`
- `sources/mem-os/src/memory.ts:2964` `nodeId: bestNode.id,`
- `sources/mem-os/src/memory.ts:2965` `confidence: update.confidence,`
- `sources/mem-os/src/memory.ts:2966` `evidenceCount: update.evidenceCount,`
- `sources/mem-os/src/memory.ts:2981` `await this.supersede(bestNode.id);`
- `sources/mem-os/src/memory.ts:2982` `this.invalidateSearchCache();`
- `sources/mem-os/src/memory.ts:2983` `this.emit("memory:superseded", {`
- `sources/mem-os/src/memory.ts:2984` `nodeId: bestNode.id,`
- `sources/mem-os/src/memory.ts:2985` `by: content,`
- `sources/mem-os/src/memory.ts:2986` `});`

**operation and route call sites**, body `sources/mem-os/src/repl.ts:144`; body evidence.

- `sources/mem-os/src/repl.ts:143` ``
- `sources/mem-os/src/repl.ts:144` `case "forget": {`
- `sources/mem-os/src/repl.ts:145` `const id = args.trim();`
- `sources/mem-os/src/repl.ts:146` `if (!id) {`
- `sources/mem-os/src/repl.ts:147` `out.push("Usage: forget <id>");`
- `sources/mem-os/src/repl.ts:148` `break;`
- `sources/mem-os/src/repl.ts:149` `}`
- `sources/mem-os/src/repl.ts:150` `const deleted = await memos.forget(id);`
- `sources/mem-os/src/repl.ts:151` `out.push(`
- `sources/mem-os/src/repl.ts:152` `deleted ? 'Forgot memory: ${id}' : 'Memory not found: ${id}',`
- `sources/mem-os/src/repl.ts:153` `);`
- `sources/mem-os/src/repl.ts:154` `break;`
- `sources/mem-os/src/repl.ts:155` `}`
- `sources/mem-os/src/repl.ts:156` ``
- `sources/mem-os/src/repl.ts:286` `"semantic <query>",`
- `sources/mem-os/src/repl.ts:287` `"forget <id>",`
- `sources/mem-os/src/repl.ts:288` `"summarize",`
- `sources/mem-os/src/repl.ts:289` `"graph",`
- `sources/mem-os/src/repl.ts:290` `"count",`
- `sources/mem-os/src/repl.ts:291` `"context-pack <query> [budget=N]",`

**operation and route call sites**, body `sources/mem-os/src/storage/sqlite.ts:301`; body evidence.

- `sources/mem-os/src/storage/sqlite.ts:300` `id          TEXT PRIMARY KEY,`
- `sources/mem-os/src/storage/sqlite.ts:301` `source_id   TEXT NOT NULL REFERENCES nodes(id) ON DELETE CASCADE,`
- `sources/mem-os/src/storage/sqlite.ts:302` `target_id   TEXT NOT NULL REFERENCES nodes(id) ON DELETE CASCADE,`
- `sources/mem-os/src/storage/sqlite.ts:303` `relation    TEXT NOT NULL DEFAULT 'relates_to',`
- `sources/mem-os/src/storage/sqlite.ts:304` `weight      REAL NOT NULL DEFAULT 0.5,`
- `sources/mem-os/src/storage/sqlite.ts:305` `metadata    TEXT NOT NULL DEFAULT '{}',`
- `sources/mem-os/src/storage/sqlite.ts:306` `created_at  INTEGER NOT NULL,`
- `sources/mem-os/src/storage/sqlite.ts:317` `CREATE TABLE IF NOT EXISTS embeddings (`
- `sources/mem-os/src/storage/sqlite.ts:318` `node_id    TEXT PRIMARY KEY REFERENCES nodes(id) ON DELETE CASCADE,`
- `sources/mem-os/src/storage/sqlite.ts:319` `vector     BLOB NOT NULL,`
- `sources/mem-os/src/storage/sqlite.ts:320` `dimensions INTEGER NOT NULL,`
- `sources/mem-os/src/storage/sqlite.ts:321` `model      TEXT NOT NULL,`
- `sources/mem-os/src/storage/sqlite.ts:322` `updated_at INTEGER NOT NULL`
- `sources/mem-os/src/storage/sqlite.ts:385` ``
- `sources/mem-os/src/storage/sqlite.ts:386` `CREATE TRIGGER IF NOT EXISTS nodes_ad AFTER DELETE ON nodes BEGIN`
- `sources/mem-os/src/storage/sqlite.ts:387` `INSERT INTO nodes_fts(nodes_fts, rowid, content, summary)`
- `sources/mem-os/src/storage/sqlite.ts:388` `VALUES ('delete', old.rowid, old.content, old.summary);`
- `sources/mem-os/src/storage/sqlite.ts:389` `END;`
- `sources/mem-os/src/storage/sqlite.ts:390` ``
- `sources/mem-os/src/storage/sqlite.ts:391` `CREATE TRIGGER IF NOT EXISTS nodes_au AFTER UPDATE ON nodes BEGIN`
- `sources/mem-os/src/storage/sqlite.ts:392` `INSERT INTO nodes_fts(nodes_fts, rowid, content, summary)`
- `sources/mem-os/src/storage/sqlite.ts:393` `VALUES ('delete', old.rowid, old.content, old.summary);`
- `sources/mem-os/src/storage/sqlite.ts:394` `INSERT INTO nodes_fts(rowid, content, summary)`
- `sources/mem-os/src/storage/sqlite.ts:395` `VALUES (new.rowid, new.content, new.summary);`
- `sources/mem-os/src/storage/sqlite.ts:396` `END;`
- `sources/mem-os/src/storage/sqlite.ts:397` `');`
- `sources/mem-os/src/storage/sqlite.ts:403` `CREATE TABLE IF NOT EXISTS node_tags (`
- `sources/mem-os/src/storage/sqlite.ts:404` `node_id TEXT NOT NULL REFERENCES nodes(id) ON DELETE CASCADE,`
- `sources/mem-os/src/storage/sqlite.ts:405` `tag     TEXT NOT NULL,`
- `sources/mem-os/src/storage/sqlite.ts:406` `PRIMARY KEY (node_id, tag)`
- `sources/mem-os/src/storage/sqlite.ts:407` `);`
- `sources/mem-os/src/storage/sqlite.ts:408` `CREATE INDEX IF NOT EXISTS idx_node_tags_tag ON node_tags(tag);`
- `sources/mem-os/src/storage/sqlite.ts:463` `private writeNodeTags(nodeId: string, tags: string[]): void {`
- `sources/mem-os/src/storage/sqlite.ts:464` `this.db.prepare("DELETE FROM node_tags WHERE node_id = ?").run(nodeId);`
- `sources/mem-os/src/storage/sqlite.ts:465` `if (tags.length === 0) return;`
- `sources/mem-os/src/storage/sqlite.ts:466` `const insert = this.db.prepare(`
- `sources/mem-os/src/storage/sqlite.ts:467` `"INSERT OR IGNORE INTO node_tags (node_id, tag) VALUES (?, ?)",`
- `sources/mem-os/src/storage/sqlite.ts:468` `);`
- `sources/mem-os/src/storage/sqlite.ts:621` `const id = row.id as string;`
- `sources/mem-os/src/storage/sqlite.ts:622` `const result = this.db.prepare("DELETE FROM nodes WHERE id = ?").run(id);`
- `sources/mem-os/src/storage/sqlite.ts:623` `if (result.changes > 0) return id;`
- `sources/mem-os/src/storage/sqlite.ts:624` `return null;`
- `sources/mem-os/src/storage/sqlite.ts:625` `}`
- `sources/mem-os/src/storage/sqlite.ts:626` ``
- `sources/mem-os/src/storage/sqlite.ts:687` ``
- `sources/mem-os/src/storage/sqlite.ts:688` `async deleteNode(id: string): Promise<boolean> {`
- `sources/mem-os/src/storage/sqlite.ts:689` `const result = this.db.prepare("DELETE FROM nodes WHERE id = ?").run(id);`
- `sources/mem-os/src/storage/sqlite.ts:690` `return result.changes > 0;`
- `sources/mem-os/src/storage/sqlite.ts:691` `}`
- `sources/mem-os/src/storage/sqlite.ts:692` ``
- `sources/mem-os/src/storage/sqlite.ts:693` `// -----------------------------------------------------------------------`
- `sources/mem-os/src/storage/sqlite.ts:769` ``
- `sources/mem-os/src/storage/sqlite.ts:770` `async deleteEdge(id: string): Promise<boolean> {`
- `sources/mem-os/src/storage/sqlite.ts:771` `const result = this.db.prepare("DELETE FROM edges WHERE id = ?").run(id);`
- `sources/mem-os/src/storage/sqlite.ts:772` `return result.changes > 0;`
- `sources/mem-os/src/storage/sqlite.ts:773` `}`
- `sources/mem-os/src/storage/sqlite.ts:774` ``
- `sources/mem-os/src/storage/sqlite.ts:775` `// -----------------------------------------------------------------------`
- `sources/mem-os/src/storage/sqlite.ts:1186` `const node = await this.getNode(nodeId);`
- `sources/mem-os/src/storage/sqlite.ts:1187` `if (!node) continue; // node deleted between the two queries`
- `sources/mem-os/src/storage/sqlite.ts:1188` `results.push({`
- `sources/mem-os/src/storage/sqlite.ts:1189` `node,`
- `sources/mem-os/src/storage/sqlite.ts:1190` `score,`
- `sources/mem-os/src/storage/sqlite.ts:1191` `scores: { semantic: score },`
- `sources/mem-os/src/storage/sqlite.ts:1207` ``
- `sources/mem-os/src/storage/sqlite.ts:1208` `async deleteEmbeddingsByModel(model: string): Promise<number> {`
- `sources/mem-os/src/storage/sqlite.ts:1209` `const info = this.db`
- `sources/mem-os/src/storage/sqlite.ts:1210` `.prepare('DELETE FROM embeddings WHERE model = ?')`
- `sources/mem-os/src/storage/sqlite.ts:1211` `.run(model);`
- `sources/mem-os/src/storage/sqlite.ts:1212` `this.vectorCache.clear();`
- `sources/mem-os/src/storage/sqlite.ts:1213` `return info.changes;`
- `sources/mem-os/src/storage/sqlite.ts:1214` `}`
- `sources/mem-os/src/storage/sqlite.ts:1283` `.prepare(`
- `sources/mem-os/src/storage/sqlite.ts:1284` `"DELETE FROM nodes WHERE expires_at IS NOT NULL AND expires_at <= ?",`
- `sources/mem-os/src/storage/sqlite.ts:1285` `)`
- `sources/mem-os/src/storage/sqlite.ts:1286` `.run(now);`
- `sources/mem-os/src/storage/sqlite.ts:1287` `return result.changes;`
- `sources/mem-os/src/storage/sqlite.ts:1288` `}`
- `sources/mem-os/src/storage/sqlite.ts:1310` ``
- `sources/mem-os/src/storage/sqlite.ts:1311` `async deleteAllNodes(): Promise<void> {`
- `sources/mem-os/src/storage/sqlite.ts:1312` `this.vectorCache.clear();`
- `sources/mem-os/src/storage/sqlite.ts:1313` `this.db.exec("DELETE FROM embeddings");`
- `sources/mem-os/src/storage/sqlite.ts:1314` `this.db.exec("DELETE FROM edges");`
- `sources/mem-os/src/storage/sqlite.ts:1315` `this.db.exec("DELETE FROM nodes");`
- `sources/mem-os/src/storage/sqlite.ts:1316` `// Rebuild FTS index`
- `sources/mem-os/src/storage/sqlite.ts:1317` `this.db.exec("INSERT INTO nodes_fts(nodes_fts) VALUES('rebuild')");`
- `sources/mem-os/src/storage/sqlite.ts:1318` `}`
- `sources/mem-os/src/storage/sqlite.ts:1319` ``
- `sources/mem-os/src/storage/sqlite.ts:1399` `const oldest = this.vectorCache.keys().next().value;`
- `sources/mem-os/src/storage/sqlite.ts:1400` `if (oldest !== undefined) this.vectorCache.delete(oldest);`
- `sources/mem-os/src/storage/sqlite.ts:1401` `}`
- `sources/mem-os/src/storage/sqlite.ts:1402` `this.vectorCache.set(nodeId, vector);`
- `sources/mem-os/src/storage/sqlite.ts:1403` `}`
- `sources/mem-os/src/storage/sqlite.ts:1404` ``

**operation and route call sites**, body `sources/mem-os/src/types.ts:720`; body evidence.

- `sources/mem-os/src/types.ts:719` ``
- `sources/mem-os/src/types.ts:720` `/** Delete a node and all connected edges. */`
- `sources/mem-os/src/types.ts:721` `deleteNode(id: string): Promise<boolean>;`
- `sources/mem-os/src/types.ts:722` ``
- `sources/mem-os/src/types.ts:723` `/** Persist a new edge. Returns the created edge. */`
- `sources/mem-os/src/types.ts:724` `saveEdge(edge: MemoryEdge): Promise<MemoryEdge>;`
- `sources/mem-os/src/types.ts:725` ``
- `sources/mem-os/src/types.ts:734` ``
- `sources/mem-os/src/types.ts:735` `/** Delete an edge by ID. */`
- `sources/mem-os/src/types.ts:736` `deleteEdge(id: string): Promise<boolean>;`
- `sources/mem-os/src/types.ts:737` ``
- `sources/mem-os/src/types.ts:738` `/** Query nodes with filters. */`
- `sources/mem-os/src/types.ts:739` `queryNodes(filter: SearchFilter): Promise<ScoredMemory[]>;`
- `sources/mem-os/src/types.ts:740` ``
- `sources/mem-os/src/types.ts:768` ``
- `sources/mem-os/src/types.ts:769` `/** Delete all embedding rows stored by the given model. Returns the count. */`
- `sources/mem-os/src/types.ts:770` `deleteEmbeddingsByModel?(model: string): Promise<number>;`
- `sources/mem-os/src/types.ts:771` ``
- `sources/mem-os/src/types.ts:772` `/** Query edges, optionally filtered by source/target. */`
- `sources/mem-os/src/types.ts:773` `queryEdges(filter?: {`
- `sources/mem-os/src/types.ts:774` `sourceId?: string;`
- `sources/mem-os/src/types.ts:802` ``
- `sources/mem-os/src/types.ts:803` `/** Delete expired nodes and return the count of deleted nodes. */`
- `sources/mem-os/src/types.ts:804` `sweepExpired(): Promise<number>;`
- `sources/mem-os/src/types.ts:805` ``
- `sources/mem-os/src/types.ts:806` `/** Get all nodes with a specific tag. */`
- `sources/mem-os/src/types.ts:807` `queryNodesByTag(tag: string): Promise<MemoryNode[]>;`
- `sources/mem-os/src/types.ts:808` ``
- `sources/mem-os/src/types.ts:809` `/** Delete all nodes and edges (batch operation). */`
- `sources/mem-os/src/types.ts:810` `deleteAllNodes(): Promise<void>;`
- `sources/mem-os/src/types.ts:811` ``
- `sources/mem-os/src/types.ts:812` `/** Close connections and release resources. */`
- `sources/mem-os/src/types.ts:813` `close(): Promise<void>;`
- `sources/mem-os/src/types.ts:814` `}`
- `sources/mem-os/src/types.ts:1167` `| "node:updated"`
- `sources/mem-os/src/types.ts:1168` `| "node:deleted"`
- `sources/mem-os/src/types.ts:1169` `| "edge:created"`
- `sources/mem-os/src/types.ts:1170` `| "edge:deleted"`
- `sources/mem-os/src/types.ts:1171` `| "link:auto"`
- `sources/mem-os/src/types.ts:1172` `| "eviction"`
- `sources/mem-os/src/types.ts:1173` `| "ttl:expired"`
- `sources/mem-os/src/types.ts:1174` `| "embedding:queued"`
- `sources/mem-os/src/types.ts:1198` `namespace?: string;`
- `sources/mem-os/src/types.ts:1199` `/** If true, do not delete or merge anything. Default false. */`
- `sources/mem-os/src/types.ts:1200` `dryRun?: boolean;`
- `sources/mem-os/src/types.ts:1201` `/** Cap on how many pairs to compare. Default 10_000. */`
- `sources/mem-os/src/types.ts:1202` `maxPairs?: number;`
- `sources/mem-os/src/types.ts:1203` `}`
- `sources/mem-os/src/types.ts:1225` ``
- `sources/mem-os/src/types.ts:1226` `/** Options for {@link MemOS.archive}. */`
- `sources/mem-os/src/types.ts:1227` `export interface ArchiveOptions {`
- `sources/mem-os/src/types.ts:1228` `/** Move memories older than this many days. Default 90. */`
- `sources/mem-os/src/types.ts:1229` `afterDays?: number;`
- `sources/mem-os/src/types.ts:1230` `/** Cap on importance for archival. Default 0.3. */`
- `sources/mem-os/src/types.ts:1231` `importanceBelow?: number;`
- `sources/mem-os/src/types.ts:1237` ``
- `sources/mem-os/src/types.ts:1238` `/** A single archive operation. */`
- `sources/mem-os/src/types.ts:1239` `export interface ArchiveMove {`
- `sources/mem-os/src/types.ts:1240` `/** The id of the moved memory. */`
- `sources/mem-os/src/types.ts:1241` `id: string;`
- `sources/mem-os/src/types.ts:1242` `/** Reason it was archived (e.g. "stale_and_low_importance"). */`
- `sources/mem-os/src/types.ts:1243` `reason: string;`
- `sources/mem-os/src/types.ts:1244` `}`
- `sources/mem-os/src/types.ts:1245` ``
- `sources/mem-os/src/types.ts:1246` `/** Result returned by {@link MemOS.archive}. */`
- `sources/mem-os/src/types.ts:1247` `export interface ArchiveResult {`
- `sources/mem-os/src/types.ts:1248` `moves: ArchiveMove[];`
- `sources/mem-os/src/types.ts:1249` `dryRun: boolean;`
- `sources/mem-os/src/types.ts:1250` `durationMs: number;`
- `sources/mem-os/src/types.ts:1251` `}`
- `sources/mem-os/src/types.ts:1252` ``
- `sources/mem-os/src/types.ts:1286` `/** Move memories older than this many days. Default 90. */`
- `sources/mem-os/src/types.ts:1287` `archiveAfterDays?: number;`
- `sources/mem-os/src/types.ts:1288` `/** Cap on importance for archival. Default 0.3. */`
- `sources/mem-os/src/types.ts:1289` `archiveImportanceBelow?: number;`
- `sources/mem-os/src/types.ts:1290` `/** Only consider memories in this namespace. Default "default". */`
- `sources/mem-os/src/types.ts:1291` `namespace?: string;`
- `sources/mem-os/src/types.ts:1292` `/** If true, do not actually mutate anything. Default false. */`
- `sources/mem-os/src/types.ts:1293` `dryRun?: boolean;`
- `sources/mem-os/src/types.ts:1294` `/** Generate cluster summaries in addition to dedupe + archive. Default true. */`
- `sources/mem-os/src/types.ts:1295` `summarize?: boolean;`
- `sources/mem-os/src/types.ts:1296` `/** Minimum cluster size for summarization. Default 3. */`
- `sources/mem-os/src/types.ts:1297` `minClusterSize?: number;`
- `sources/mem-os/src/types.ts:1298` `/** Cosine threshold for cluster membership. Default 0.78. */`
- `sources/mem-os/src/types.ts:1325` ``
- `sources/mem-os/src/types.ts:1326` `/** Options for {@link MemOS.forgetByDecay}. */`
- `sources/mem-os/src/types.ts:1327` `export interface DecayForgettingOptions {`
- `sources/mem-os/src/types.ts:1328` `/** Only consider memories in this namespace. Default "default". */`
- `sources/mem-os/src/types.ts:1329` `namespace?: string;`
- `sources/mem-os/src/types.ts:1330` `/** Exponential half-life (days) for the recency term. Default 60. */`
- `sources/mem-os/src/types.ts:1331` `halfLifeDays?: number;`
- `sources/mem-os/src/types.ts:1339` ``
- `sources/mem-os/src/types.ts:1340` `/** Result returned by {@link MemOS.forgetByDecay}. */`
- `sources/mem-os/src/types.ts:1341` `export interface DecayForgetResult {`
- `sources/mem-os/src/types.ts:1342` `superseded: DecaySuperseded[];`
- `sources/mem-os/src/types.ts:1343` `dryRun: boolean;`
- `sources/mem-os/src/types.ts:1344` `durationMs: number;`
- `sources/mem-os/src/types.ts:1345` `}`
- `sources/mem-os/src/types.ts:1376` `merges: DedupeMerge[];`
- `sources/mem-os/src/types.ts:1377` `moves: ArchiveMove[];`
- `sources/mem-os/src/types.ts:1378` `clusters: ClusterSummary[];`
- `sources/mem-os/src/types.ts:1379` `/** Memories superseded by decay-based forgetting. */`
- `sources/mem-os/src/types.ts:1380` `decayed: DecaySuperseded[];`
- `sources/mem-os/src/types.ts:1381` `dryRun: boolean;`
- `sources/mem-os/src/types.ts:1382` `durationMs: number;`
- `sources/mem-os/src/types.ts:1383` `}`

</details>

## 3. What reaches the agent

No provider system block/prefetch hooks. Agent receives MCP tool descriptions and text plus structuredContent results; forget returns id/deleted and human text (`sources/mem-os/src/mcp.ts:281`; `sources/mem-os/src/mcp.ts:288`). Context-pack/search tools are explicit calls with their own budgets, not automatic native Hermes prefetch.

This is not a native MemoryProvider; provider prefetch rules do not establish its behavior.

<details>
<summary>Context and tool-result assembly quotes</summary>

</details>

## 4. Data access boundaries

Direct SQLite/SDK access `yes`; engine config chooses dbPath and optional cipher key from MEMOS_KEY (`sources/mem-os/src/memory.ts:361`; `sources/mem-os/src/memory.ts:382`). If encryption is configured, direct plaintext access needs that key. Accountless local mode `yes` with local embeddings/default local engine. MCP env config is empty and child can inherit environment (`sources/mem-os/plugin/.mcp.json:6`). No mandatory account credential; optional model/API keys and database paths below. Token-savings metrics are local process telemetry, not by themselves outbound analytics.

Boundary model: same OS user, terminal, mounted filesystem and allowed network. Direct reachability means another client can use the same file/endpoint and available credentials, not unauthenticated access or encryption bypass. Hermes setup persists declared secrets through `hermes-agent/hermes_cli/memory_setup.py:321` and `hermes-agent/hermes_cli/config.py:2697`. These are possible configured paths; no live credentials were read.

Host credential-file detail: `get_hermes_home()/.env` is resolved at `hermes-agent/hermes_cli/config.py:473`. New files are secured to 0600 at `hermes-agent/hermes_cli/config.py:506`; the writer preserves an existing mode at `hermes-agent/hermes_cli/config.py:2544` and `:2562`. Therefore 0600 is not an unconditional mode claim for every existing profile.

<details>
<summary>Paths, environment, permissions, egress, telemetry and process-launch evidence</summary>

- `sources/mem-os/plugin/.claude-plugin/plugin.json:5` `"description": "Local-first persistent memory: store durable facts, recall them in any session. Zero cloud, zero telemetry, SQLite-backed.",`
- `sources/mem-os/plugin/.claude-plugin/plugin.json:10` `"homepage": "https://github.com/Markgatcha/memos",`

- `sources/mem-os/plugin/mcp.json:2` `"$schema": "https://agent-plugins.org/schemas/1.0.0/mcp.schema.json",`

- `sources/mem-os/plugin/plugin.json:2` `"$schema": "https://agent-plugins.org/schemas/1.0.0/plugin.schema.json",`
- `sources/mem-os/plugin/plugin.json:5` `"description": "Local-first persistent memory: store durable facts, recall them in any session. Zero cloud, zero telemetry, SQLite-backed.",`
- `sources/mem-os/plugin/plugin.json:10` `"homepage": "https://github.com/Markgatcha/memos",`
- `sources/mem-os/plugin/plugin.json:11` `"repository": "https://github.com/Markgatcha/memos",`

- `sources/mem-os/src/mcp.ts:718` `title: "Token Savings Telemetry",`
- `sources/mem-os/src/mcp.ts:720` `"Lifetime context-pack token telemetry for this server process: " +`
- `sources/mem-os/src/mcp.ts:1209` `"Lifetime context-pack token telemetry for this server process: " +`

- `sources/mem-os/src/storage/sqlite.ts:34` `const home = homedir() || process.env.USERPROFILE || process.env.HOME || ".";`

</details>

## 5. Reproducibility

Exact MCP launcher `@mem-os/sdk@1.6.26` is `pinned` (`sources/mem-os/plugin/.mcp.json:5`). Other plugin/mcp.json launchers and Python package versions are separate surfaces and are listed below; repository dependencies can still be ranged.

Retained lock/checksum files: `sources/mem-os/package-lock.json:1`, `sources/mem-os/website/package-lock.json:1`. Upstream completeness is unknown: fetch.py:15 excludes .lock files, fetch.py:12 excludes build/vendor directories. Absence here is not absence upstream.

- `sources/mem-os/plugin/.mcp.json:5` `"args": ["-y", "@mem-os/sdk@1.6.26", "mcp"],`

## 6. Tests

MCP protocol suite has 12 literal test declarations; memos_forget end-to-end test exists (`sources/mem-os/__tests__/mcp-protocol.test.ts:327`). Engine forget test in memos.test.ts is separate (`sources/mem-os/__tests__/memos.test.ts:261`). Count is declarations, not a test run.

Adapter-related declarations: **12**. Python AST test_ functions, fixtures excluded; literal TS/JS it/test, Go Test*, Rust test_* declarations. No parametrization expansion, collection, execution, or coverage measurement. Other rows are engine test candidates selected by mutation names; names alone do not verify forget coverage.

| file | declarations | adapter-related | forget-related declarations |
|---|---:|---|---|
| `sources/mem-os/__tests__/consolidation.test.ts:1` | 7 | no | `dry-run does not delete or modify anything` `sources/mem-os/__tests__/consolidation.test.ts:76` |
| `sources/mem-os/__tests__/mcp-protocol.test.ts:1` | 12 | yes | `memos_forget end-to-end: deletes a memory` `sources/mem-os/__tests__/mcp-protocol.test.ts:327` |
| `sources/mem-os/__tests__/memos.test.ts:1` | 35 | no | `forget removes memory` `sources/mem-os/__tests__/memos.test.ts:261` |

## 7. Corrections to B2

Old values below are copied from the triage row (TRIAGE.md), not reconstructed from memory. A refinement narrows an over-broad field; it need not reverse its original value. Counts are distinct field paths, not the number of factual errors.

- `sec.paths`: `["memos/server/main.py:38", "memos/server/main.py:125", "memos/server/main.py:277", "memos/server/main.py:283", "memos/server/main.py:338"]` → Selected launcher is TypeScript MCP; its dbPath and MEMOS_KEY cipher configuration determine this integration, not Python server defaults. Evidence: `sources/mem-os/plugin/.mcp.json:5`, `sources/mem-os/src/memory.ts:382`.
- `engine_delete.scope`: `"Named deletion/retraction endpoint or method, not a guarantee of physical erasure. Its reach may overlap an agent tool; lack of a native tool is counted separately."` → `yes`: memos_forget callback awaits memos.forget(id) (`sources/mem-os/src/mcp.ts:280`). Engine forget calls storage.deleteNode, removes in-memory graph node, emits node:deleted and invalidates search cache (`sources/mem-os/src/memory.ts:796`). SQLite deleteNode and edge/embedding/bulk-delete bodies are in the ledger. `partial` for all-copy criterion: live graph/search cleanup is explicit, but generated summaries stored as independent nodes, exported copies and Hermes dialogue are not globally purged by one node ID. No claim that all model-derived content disappears. SQLite enables foreign keys; edge/embedding child tables use ON DELETE CASCADE, and FTS has an AFTER DELETE trigger (`sources/mem-os/src/storage/sqlite.ts:271`; `sources/mem-os/src/storage/sqlite.ts:301`; `sources/mem-os/src/storage/sqlite.ts:318`; `sources/mem-os/src/storage/sqlite.ts:386`). These establish cleanup of those linked copies. Evidence: `sources/mem-os/src/mcp.ts:280`, `sources/mem-os/src/memory.ts:796`, `sources/mem-os/src/storage/sqlite.ts:271`, `sources/mem-os/src/storage/sqlite.ts:301`, `sources/mem-os/src/storage/sqlite.ts:318`, `sources/mem-os/src/storage/sqlite.ts:386`.
- `context.mechanism.v`: `"No native per-turn injection established by this adapter. MCP tools return requested results; skills provide instructions. External shim implementation is unknown where absent."` → No provider system block/prefetch hooks. Agent receives MCP tool descriptions and text plus structuredContent results; forget returns id/deleted and human text (`sources/mem-os/src/mcp.ts:281`; `sources/mem-os/src/mcp.ts:288`). Context-pack/search tools are explicit calls with their own budgets, not automatic native Hermes prefetch. Evidence: `sources/mem-os/src/mcp.ts:281`, `sources/mem-os/src/mcp.ts:288`.
- `sec.telemetry.v`: `"inspect listed emitters/configuration"` → Direct SQLite/SDK access `yes`; engine config chooses dbPath and optional cipher key from MEMOS_KEY (`sources/mem-os/src/memory.ts:361`; `sources/mem-os/src/memory.ts:382`). If encryption is configured, direct plaintext access needs that key. Accountless local mode `yes` with local embeddings/default local engine. MCP env config is empty and child can inherit environment (`sources/mem-os/plugin/.mcp.json:6`). No mandatory account credential; optional model/API keys and database paths below. Token-savings metrics are local process telemetry, not by themselves outbound analytics. Evidence: `sources/mem-os/src/memory.ts:361`, `sources/mem-os/src/memory.ts:382`, `sources/mem-os/plugin/.mcp.json:6`.
- `repro.decls`: `["package.json:3", "package.json:7", "package.json:10", "package.json:13", "package.json:114", "package.json:119", "package.json:123", "package.json:132", "package.json:137", "package.json:144", "package.json:150", "package.json:161", "package.json:162", "package.json:163", "package.json:165", "plugin/.claude-plugin/plugin.json:4", "plugin/.claude-plugin/plugin.json:10", "plugin/.mcp.json:5", "plugin/plugin.json:4", "plugin/plugin.json:10", "plugin/plugin.json:11", "pyproject.toml:9", "pyproject.toml:13", "pyproject.toml:43", "pyproject.toml:44", "pyproject.toml:45", "pyproject.toml:46", "pyproject.toml:49", "pyproject.toml:51", "pyproject.toml:52", "pyproject.toml:55", "pyproject.toml:61", "pyproject.toml:62", "pyproject.toml:63", "pyproject.toml:64", "pyproject.toml:65", "pyproject.toml:70", "pyproject.toml:71", "pyproject.toml:73", "pyproject.toml:108", "pyproject.toml:143"]` → Exact MCP launcher `@mem-os/sdk@1.6.26` is `pinned` (`sources/mem-os/plugin/.mcp.json:5`). Other plugin/mcp.json launchers and Python package versions are separate surfaces and are listed below; repository dependencies can still be ranged. Evidence: `sources/mem-os/plugin/.mcp.json:5`.
- `tests.count`: `324` → 12 adapter-related declarations using the stated parser/scope; B2 counted a different, repository-wide inventory. This is a scope refinement, not a count of executed or passing tests. MCP protocol suite has 12 literal test declarations; memos_forget end-to-end test exists (`sources/mem-os/__tests__/mcp-protocol.test.ts:327`). Engine forget test in memos.test.ts is separate (`sources/mem-os/__tests__/memos.test.ts:261`). Count is declarations, not a test run. Evidence: `sources/mem-os/__tests__/mcp-protocol.test.ts:327`, `sources/mem-os/__tests__/memos.test.ts:261`.

MATRIX counts these 6 explicit field corrections/refinements. Sections 1-6 provide the body and test inventories. Test-name candidates are not passing tests or measured coverage.
