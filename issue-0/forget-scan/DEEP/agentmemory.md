# agentmemory: B3 static correctness review

Snapshot `28ffc538b81b265e2f649354ed9f88b0c12ae414` (MANIFEST.json row `agentmemory`). No plugin execution or passing-test claims.

## 1. Architecture

`provider`: Python adapter calls a separate AgentMemory service over HTTP; TypeScript engine registers iii functions and exposes HTTP triggers. It is not an in-process Python store (`sources/agentmemory/integrations/hermes/__init__.py:402`; `sources/agentmemory/src/functions/remember.ts:267`).

Hook/body inventory (no-op/fallback declarations are not active behavior):

| callable | declaration | first executable body line |
|---|---|---|
| `register` | `sources/agentmemory/integrations/hermes/__init__.py:669` | `sources/agentmemory/integrations/hermes/__init__.py:670` |
| `is_available` | `sources/agentmemory/integrations/hermes/__init__.py:256` | `sources/agentmemory/integrations/hermes/__init__.py:257` |
| `initialize` | `sources/agentmemory/integrations/hermes/__init__.py:269` | `sources/agentmemory/integrations/hermes/__init__.py:270` |
| `on_turn_start` | `sources/agentmemory/integrations/hermes/__init__.py:353` | `sources/agentmemory/integrations/hermes/__init__.py:354` |
| `get_config_schema` | `sources/agentmemory/integrations/hermes/__init__.py:358` | `sources/agentmemory/integrations/hermes/__init__.py:359` |
| `save_config` | `sources/agentmemory/integrations/hermes/__init__.py:381` | `sources/agentmemory/integrations/hermes/__init__.py:382` |
| `system_prompt_block` | `sources/agentmemory/integrations/hermes/__init__.py:389` | `sources/agentmemory/integrations/hermes/__init__.py:390` |
| `prefetch` | `sources/agentmemory/integrations/hermes/__init__.py:392` | `sources/agentmemory/integrations/hermes/__init__.py:393` |
| `queue_prefetch` | `sources/agentmemory/integrations/hermes/__init__.py:432` | `sources/agentmemory/integrations/hermes/__init__.py:433` |
| `get_tool_schemas` | `sources/agentmemory/integrations/hermes/__init__.py:437` | `sources/agentmemory/integrations/hermes/__init__.py:438` |
| `handle_tool_call` | `sources/agentmemory/integrations/hermes/__init__.py:491` | `sources/agentmemory/integrations/hermes/__init__.py:492` |
| `sync_turn` | `sources/agentmemory/integrations/hermes/__init__.py:533` | `sources/agentmemory/integrations/hermes/__init__.py:535` |
| `on_session_end` | `sources/agentmemory/integrations/hermes/__init__.py:556` | `sources/agentmemory/integrations/hermes/__init__.py:557` |
| `on_session_switch` | `sources/agentmemory/integrations/hermes/__init__.py:566` | `sources/agentmemory/integrations/hermes/__init__.py:567` |
| `on_pre_compress` | `sources/agentmemory/integrations/hermes/__init__.py:579` | `sources/agentmemory/integrations/hermes/__init__.py:580` |
| `on_memory_write` | `sources/agentmemory/integrations/hermes/__init__.py:599` | `sources/agentmemory/integrations/hermes/__init__.py:600` |
| `shutdown` | `sources/agentmemory/integrations/hermes/__init__.py:661` | `sources/agentmemory/integrations/hermes/__init__.py:662` |
| `is_available` | `sources/agentmemory/integrations/hermes/__init__.py:76` | `sources/agentmemory/integrations/hermes/__init__.py:76` |
| `initialize` | `sources/agentmemory/integrations/hermes/__init__.py:78` | `sources/agentmemory/integrations/hermes/__init__.py:78` |
| `get_tool_schemas` | `sources/agentmemory/integrations/hermes/__init__.py:80` | `sources/agentmemory/integrations/hermes/__init__.py:80` |
| `handle_tool_call` | `sources/agentmemory/integrations/hermes/__init__.py:82` | `sources/agentmemory/integrations/hermes/__init__.py:82` |
| `get_config_schema` | `sources/agentmemory/integrations/hermes/__init__.py:83` | `sources/agentmemory/integrations/hermes/__init__.py:83` |
| `save_config` | `sources/agentmemory/integrations/hermes/__init__.py:84` | `sources/agentmemory/integrations/hermes/__init__.py:84` |
| `system_prompt_block` | `sources/agentmemory/integrations/hermes/__init__.py:85` | `sources/agentmemory/integrations/hermes/__init__.py:85` |
| `prefetch` | `sources/agentmemory/integrations/hermes/__init__.py:86` | `sources/agentmemory/integrations/hermes/__init__.py:86` |
| `queue_prefetch` | `sources/agentmemory/integrations/hermes/__init__.py:87` | `sources/agentmemory/integrations/hermes/__init__.py:87` |
| `sync_turn` | `sources/agentmemory/integrations/hermes/__init__.py:88` | `sources/agentmemory/integrations/hermes/__init__.py:88` |
| `on_session_end` | `sources/agentmemory/integrations/hermes/__init__.py:89` | `sources/agentmemory/integrations/hermes/__init__.py:89` |
| `on_pre_compress` | `sources/agentmemory/integrations/hermes/__init__.py:90` | `sources/agentmemory/integrations/hermes/__init__.py:90` |
| `on_memory_write` | `sources/agentmemory/integrations/hermes/__init__.py:91` | `sources/agentmemory/integrations/hermes/__init__.py:91` |
| `shutdown` | `sources/agentmemory/integrations/hermes/__init__.py:92` | `sources/agentmemory/integrations/hermes/__init__.py:92` |

## 2. Forget paths and derived copies

`none`: `handle_tool_call` accepts exactly memory_recall, memory_search, memory_save (`sources/agentmemory/integrations/hermes/__init__.py:492`). Engine `yes`: `mem::forget` supports a memory ID, observation/session cleanup, and selected session summary handling; API forwards to it (`sources/agentmemory/src/triggers/api.ts:1759`). Memory deletion deletes KV row, facets, access log, index content, and flushes the index (`sources/agentmemory/src/functions/remember.ts:344`; `sources/agentmemory/src/functions/remember.ts:356`; `sources/agentmemory/src/functions/remember.ts:422`). `partial`: a fact ID does not imply deletion of the raw conversation observation or every derived summary. Session deletion is separately selected (`sources/agentmemory/src/functions/remember.ts:388`). No host purge.

Host state.db is independent (`hermes-agent/hermes_state.py:181`). No reviewed adapter forget body performs a general purge of that database or built-in MEMORY.md/USER.md. Prompt/tool-result copies and independent source transcripts remain separate; source operation bodies below establish the scope of each plugin mutation.

<details>
<summary>Deletion, invalidation, archive and reset body ledger</summary>

Public routes, SDK/CLI dispatch and internal storage operations are distinguished by their source. Cache invalidation and management-object deletion are not additional fact-forget tools. A stub is not working deletion. Call-through means the downstream body still determines erasure. HTTP prefixes/selectors must be read with route registration.

**operation and route call sites**, body `sources/agentmemory/src/capture/client.ts:38`; body evidence.

- `sources/agentmemory/src/capture/client.ts:37` `pending.add(safe);`
- `sources/agentmemory/src/capture/client.ts:38` `void safe.finally(() => pending.delete(safe));`
- `sources/agentmemory/src/capture/client.ts:39` `return safe;`
- `sources/agentmemory/src/capture/client.ts:40` `}`
- `sources/agentmemory/src/capture/client.ts:41` `function replay() {`
- `sources/agentmemory/src/capture/client.ts:42` `if (stopped || draining) return draining;`

**operation and route call sites**, body `sources/agentmemory/src/capture/event-record.ts:16`; body evidence.

- `sources/agentmemory/src/capture/event-record.ts:15` `attempts: number;`
- `sources/agentmemory/src/capture/event-record.ts:16` `state?: "deleted";`
- `sources/agentmemory/src/capture/event-record.ts:17` `deletedAt?: string;`
- `sources/agentmemory/src/capture/event-record.ts:18` `}`
- `sources/agentmemory/src/capture/event-record.ts:19` ``
- `sources/agentmemory/src/capture/event-record.ts:20` `export function captureEventShard(key: string): string {`
- `sources/agentmemory/src/capture/event-record.ts:21` `return key.slice(4, 4 + EVENT_SHARD_CHARS);`
- `sources/agentmemory/src/capture/event-record.ts:31` ``
- `sources/agentmemory/src/capture/event-record.ts:32` `export async function markCaptureEventDeleted(`
- `sources/agentmemory/src/capture/event-record.ts:33` `kv: StateKV,`
- `sources/agentmemory/src/capture/event-record.ts:34` `observation: { id: string; sessionId?: string; captureKey?: unknown } | null | undefined,`
- `sources/agentmemory/src/capture/event-record.ts:35` `): Promise<void> {`
- `sources/agentmemory/src/capture/event-record.ts:36` `const key = observation?.captureKey;`
- `sources/agentmemory/src/capture/event-record.ts:39` `const existing = await kv.get<CompletedEvent>(scope, key);`
- `sources/agentmemory/src/capture/event-record.ts:40` `if (existing?.state === "deleted") return;`
- `sources/agentmemory/src/capture/event-record.ts:41` `const now = new Date().toISOString();`
- `sources/agentmemory/src/capture/event-record.ts:42` `const marker: CompletedEvent = {`
- `sources/agentmemory/src/capture/event-record.ts:43` `key,`
- `sources/agentmemory/src/capture/event-record.ts:44` `eventId: existing?.eventId ?? "",`
- `sources/agentmemory/src/capture/event-record.ts:50` `attempts: existing?.attempts ?? 0,`
- `sources/agentmemory/src/capture/event-record.ts:51` `state: "deleted",`
- `sources/agentmemory/src/capture/event-record.ts:52` `deletedAt: now,`
- `sources/agentmemory/src/capture/event-record.ts:53` `};`
- `sources/agentmemory/src/capture/event-record.ts:54` `await kv.set(scope, key, marker);`
- `sources/agentmemory/src/capture/event-record.ts:55` `}`

**operation and route call sites**, body `sources/agentmemory/src/capture/prompt-ledger.ts:59`; body evidence.

- `sources/agentmemory/src/capture/prompt-ledger.ts:58` `for (const key of keys.slice(0, Math.max(0, keys.length - PROMPT_LEDGER_MAX_KEYS))) {`
- `sources/agentmemory/src/capture/prompt-ledger.ts:59` `delete ledger.prompts[key];`
- `sources/agentmemory/src/capture/prompt-ledger.ts:60` `}`
- `sources/agentmemory/src/capture/prompt-ledger.ts:61` `ledger.updatedAt = new Date().toISOString();`
- `sources/agentmemory/src/capture/prompt-ledger.ts:62` `await kv.set(KV.capturePrompts, ledgerKey(ledger.sessionId), ledger);`
- `sources/agentmemory/src/capture/prompt-ledger.ts:63` `}`
- `sources/agentmemory/src/capture/prompt-ledger.ts:74` `if (current.live.includes(ref)) return;`
- `sources/agentmemory/src/capture/prompt-ledger.ts:75` `delete ledger.prompts[key];`
- `sources/agentmemory/src/capture/prompt-ledger.ts:76` `ledger.prompts[key] = { live: bounded([...current.live, ref]), claimed: current.claimed };`
- `sources/agentmemory/src/capture/prompt-ledger.ts:77` `await save(kv, ledger);`
- `sources/agentmemory/src/capture/prompt-ledger.ts:78` `}`
- `sources/agentmemory/src/capture/prompt-ledger.ts:79` ``
- `sources/agentmemory/src/capture/prompt-ledger.ts:101` `if (i >= maxLedgers || Date.parse(ledger.updatedAt) < cutoffMs) {`
- `sources/agentmemory/src/capture/prompt-ledger.ts:102` `await kv.delete(KV.capturePrompts, ledgerKey(ledger.sessionId)).catch(() => {});`
- `sources/agentmemory/src/capture/prompt-ledger.ts:103` `removed++;`
- `sources/agentmemory/src/capture/prompt-ledger.ts:104` `}`
- `sources/agentmemory/src/capture/prompt-ledger.ts:105` `}`
- `sources/agentmemory/src/capture/prompt-ledger.ts:106` `return removed;`

**operation and route call sites**, body `sources/agentmemory/src/capture/spool.ts:636`; body evidence.

- `sources/agentmemory/src/capture/spool.ts:635` `if (result.error) s.lastDrainError = result.error;`
- `sources/agentmemory/src/capture/spool.ts:636` `else delete s.lastDrainError;`
- `sources/agentmemory/src/capture/spool.ts:637` `});`
- `sources/agentmemory/src/capture/spool.ts:638` `return result;`
- `sources/agentmemory/src/capture/spool.ts:639` `} finally {`
- `sources/agentmemory/src/capture/spool.ts:640` `releaseLock(paths.drainLock);`

**operation and route call sites**, body `sources/agentmemory/src/cli/compose-project.ts:89`; body evidence.

- `sources/agentmemory/src/cli/compose-project.ts:88` `"AGENTMEMORY_VECTOR_BACKFILL", "AGENTMEMORY_SLOTS", "AGENTMEMORY_PROVIDER", "AGENTMEMORY_VERBOSE",`
- `sources/agentmemory/src/cli/compose-project.ts:89` `"OBSIDIAN_AUTO_EXPORT", "AUTO_FORGET_ENABLED", "AUTO_FORGET_INTERVAL_MS", "LESSON_DECAY_ENABLED",`
- `sources/agentmemory/src/cli/compose-project.ts:90` `"INSIGHT_DECAY_ENABLED", "REBUILD_EMBED_BATCH_SIZE", "RERANK_ENABLED",`
- `sources/agentmemory/src/cli/compose-project.ts:91` `"AGENTMEMORY_LOCAL_DISPATCH",`
- `sources/agentmemory/src/cli/compose-project.ts:92` `]);`
- `sources/agentmemory/src/cli/compose-project.ts:93` ``
- `sources/agentmemory/src/cli/compose-project.ts:351` `'          - ${yamlString('http://127.0.0.1:${viewerPort}')}',`
- `sources/agentmemory/src/cli/compose-project.ts:352` `"        allowed_methods: [GET, POST, PUT, DELETE, OPTIONS]",`
- `sources/agentmemory/src/cli/compose-project.ts:353` `);`
- `sources/agentmemory/src/cli/compose-project.ts:354` ``
- `sources/agentmemory/src/cli/compose-project.ts:355` `lines.push(`
- `sources/agentmemory/src/cli/compose-project.ts:356` `"  agentmemory:",`

**operation and route call sites**, body `sources/agentmemory/src/cli/connect/dsh.ts:85`; body evidence.

- `sources/agentmemory/src/cli/connect/dsh.ts:84` `} else {`
- `sources/agentmemory/src/cli/connect/dsh.ts:85` `entries.delete(j--);`
- `sources/agentmemory/src/cli/connect/dsh.ts:86` `removed = true;`
- `sources/agentmemory/src/cli/connect/dsh.ts:87` `}`
- `sources/agentmemory/src/cli/connect/dsh.ts:88` `}`
- `sources/agentmemory/src/cli/connect/dsh.ts:89` `if (removed && entries.items.length === 0) operations.delete(i--);`
- `sources/agentmemory/src/cli/connect/dsh.ts:90` `}`
- `sources/agentmemory/src/cli/connect/dsh.ts:91` `if (!replaced) operations.items.push(...addition.items);`
- `sources/agentmemory/src/cli/connect/dsh.ts:92` `operations.flow = false;`
- `sources/agentmemory/src/cli/connect/dsh.ts:93` `return doc.toString();`

**operation and route call sites**, body `sources/agentmemory/src/cli/connect/omp.ts:68`; body evidence.

- `sources/agentmemory/src/cli/connect/omp.ts:67` `if (userOwnsMcp) {`
- `sources/agentmemory/src/cli/connect/omp.ts:68` `delete extra[".mcp.json"];`
- `sources/agentmemory/src/cli/connect/omp.ts:69` `p.log.info("Preserved the existing AgentMemory MCP server in OMP's mcp.json.");`
- `sources/agentmemory/src/cli/connect/omp.ts:70` `} else if (opts.withMcp) extra[".mcp.json"] = JSON.stringify({ mcpServers: { agentmemory: nativeMcpEntry(root) } }, null, 2) + "\n";`
- `sources/agentmemory/src/cli/connect/omp.ts:71` `}`
- `sources/agentmemory/src/cli/connect/omp.ts:72` `const bundle = prepareNativeBundle(join(root, "integrations/omp"), target, "omp-extension", ["package.json", "dist/index.mjs"], opts.force, join(root, "plugin/skills"), extra);`

**operation and route call sites**, body `sources/agentmemory/src/cli/connect/openclaw.ts:88`; body evidence.

- `sources/agentmemory/src/cli/connect/openclaw.ts:87` `if ((key === "AGENTMEMORY_URL" && value === "${AGENTMEMORY_URL:-http://localhost:3111}") ||`
- `sources/agentmemory/src/cli/connect/openclaw.ts:88` `(key === "AGENTMEMORY_SECRET" && value === "${AGENTMEMORY_SECRET:-}")) delete env[key];`
- `sources/agentmemory/src/cli/connect/openclaw.ts:89` `if (key === "AGENTMEMORY_TOOLS" && value === "${AGENTMEMORY_TOOLS:-all}") env[key] = "all";`
- `sources/agentmemory/src/cli/connect/openclaw.ts:90` `}`
- `sources/agentmemory/src/cli/connect/openclaw.ts:91` `servers.agentmemory = { ...migrated, ...runtime, transport: "stdio", env };`
- `sources/agentmemory/src/cli/connect/openclaw.ts:92` `} else p.log.info("Preserved the existing AgentMemory MCP entry. After moving the installed package, review its settings and reconnect with --with-mcp --force to refresh its runtime path.");`
- `sources/agentmemory/src/cli/connect/openclaw.ts:93` `if (migrateLegacy) delete next.mcpServers;`
- `sources/agentmemory/src/cli/connect/openclaw.ts:94` `next.mcp = { ...mcp, servers };`
- `sources/agentmemory/src/cli/connect/openclaw.ts:95` `}`
- `sources/agentmemory/src/cli/connect/openclaw.ts:96` `entries.agentmemory = { ...entry, enabled: true, hooks: { ...hooks, allowConversationAccess: true } };`
- `sources/agentmemory/src/cli/connect/openclaw.ts:97` `plugins.entries = entries;`

**operation and route call sites**, body `sources/agentmemory/src/cli/engine-install.ts:58`; body evidence.

- `sources/agentmemory/src/cli/engine-install.ts:57` ``
- `sources/agentmemory/src/cli/engine-install.ts:58` `export function iiiArchivePath(binDir: string): string {`
- `sources/agentmemory/src/cli/engine-install.ts:59` `return join(binDir, ".iii-download.tar.gz");`
- `sources/agentmemory/src/cli/engine-install.ts:60` `}`
- `sources/agentmemory/src/cli/engine-install.ts:61` ``
- `sources/agentmemory/src/cli/engine-install.ts:62` `export function iiiDownloadShellCommand(): string {`
- `sources/agentmemory/src/cli/engine-install.ts:76` `const verify = sha256`
- `sources/agentmemory/src/cli/engine-install.ts:77` `? ' && printf '%s  %s\\n' "${sha256}" "$archive" | { sha256sum -c - 2>/dev/null || shasum -a 256 -c -; }'`
- `sources/agentmemory/src/cli/engine-install.ts:78` `: "";`
- `sources/agentmemory/src/cli/engine-install.ts:79` `return 'mkdir -p "${binDir}" && archive="$(mktemp "${binDir}/.iii-download.XXXXXX")" && { curl -fL --connect-timeout ${III_INSTALL_CONNECT_TIMEOUT_S} ${curlRetryFlags()} -o "$archive" "${releaseUrl}"${verify} && tar -xzf "$archive" -C "${binDir}"; code=$?; rm -f "$archive"; (exit $code); } && chmod +x "${binPath}"';`
- `sources/agentmemory/src/cli/engine-install.ts:80` `}`
- `sources/agentmemory/src/cli/engine-install.ts:81` ``
- `sources/agentmemory/src/cli/engine-install.ts:82` `export function describeInstallFailure(result: {`
- `sources/agentmemory/src/cli/engine-install.ts:83` `error?: Error & { code?: string };`
- `sources/agentmemory/src/cli/engine-install.ts:105` ``
- `sources/agentmemory/src/cli/engine-install.ts:106` `export function installIiiArchive(opts: {`
- `sources/agentmemory/src/cli/engine-install.ts:107` `sh: string;`
- `sources/agentmemory/src/cli/engine-install.ts:108` `releaseUrl: string;`
- `sources/agentmemory/src/cli/engine-install.ts:109` `version: string;`
- `sources/agentmemory/src/cli/engine-install.ts:110` `asset: string;`
- `sources/agentmemory/src/cli/engine-install.ts:131` ``
- `sources/agentmemory/src/cli/engine-install.ts:132` `const archive = iiiArchivePath(opts.binDir);`
- `sources/agentmemory/src/cli/engine-install.ts:133` `const download = run(iiiDownloadShellCommand(), [opts.releaseUrl, opts.binDir, archive]);`
- `sources/agentmemory/src/cli/engine-install.ts:134` `if (download.status !== 0) {`
- `sources/agentmemory/src/cli/engine-install.ts:135` `return { ok: false, reason: "download", detail: describeInstallFailure(download) };`
- `sources/agentmemory/src/cli/engine-install.ts:136` `}`
- `sources/agentmemory/src/cli/engine-install.ts:137` ``
- `sources/agentmemory/src/cli/engine-install.ts:139` `try {`
- `sources/agentmemory/src/cli/engine-install.ts:140` `actual = sha256File(archive);`
- `sources/agentmemory/src/cli/engine-install.ts:141` `} catch (err) {`
- `sources/agentmemory/src/cli/engine-install.ts:142` `rmSync(archive, { force: true });`
- `sources/agentmemory/src/cli/engine-install.ts:143` `return { ok: false, reason: "download", detail: err instanceof Error ? err.message : String(err) };`
- `sources/agentmemory/src/cli/engine-install.ts:144` `}`
- `sources/agentmemory/src/cli/engine-install.ts:145` `if (actual !== expected) {`
- `sources/agentmemory/src/cli/engine-install.ts:146` `rmSync(archive, { force: true });`
- `sources/agentmemory/src/cli/engine-install.ts:147` `return {`
- `sources/agentmemory/src/cli/engine-install.ts:148` `ok: false,`
- `sources/agentmemory/src/cli/engine-install.ts:149` `reason: "checksum",`
- `sources/agentmemory/src/cli/engine-install.ts:150` `detail: 'SHA-256 mismatch for ${opts.asset}: expected ${expected}, got ${actual}. The download was deleted and nothing was installed',`
- `sources/agentmemory/src/cli/engine-install.ts:151` `};`
- `sources/agentmemory/src/cli/engine-install.ts:152` `}`
- `sources/agentmemory/src/cli/engine-install.ts:153` ``
- `sources/agentmemory/src/cli/engine-install.ts:154` `const extract = run(iiiExtractShellCommand(), [archive, opts.binDir, opts.binPath]);`
- `sources/agentmemory/src/cli/engine-install.ts:155` `if (extract.status !== 0) {`
- `sources/agentmemory/src/cli/engine-install.ts:156` `return { ok: false, reason: "extract", detail: describeInstallFailure(extract) };`
- `sources/agentmemory/src/cli/engine-install.ts:157` `}`
- `sources/agentmemory/src/cli/engine-install.ts:158` `return { ok: true };`
- `sources/agentmemory/src/cli/engine-install.ts:172` ``
- `sources/agentmemory/src/cli/engine-install.ts:173` `export function iiiZipArchivePath(binDir: string): string {`
- `sources/agentmemory/src/cli/engine-install.ts:174` `return join(binDir, '.iii-download-${process.pid}-${randomBytes(4).toString("hex")}.zip');`
- `sources/agentmemory/src/cli/engine-install.ts:175` `}`
- `sources/agentmemory/src/cli/engine-install.ts:176` ``
- `sources/agentmemory/src/cli/engine-install.ts:177` `function removeQuietly(path: string): void {`
- `sources/agentmemory/src/cli/engine-install.ts:244` `export function windowsZipExtractCommands(`
- `sources/agentmemory/src/cli/engine-install.ts:245` `archive: string,`
- `sources/agentmemory/src/cli/engine-install.ts:246` `binDir: string,`
- `sources/agentmemory/src/cli/engine-install.ts:247` `env: NodeJS.ProcessEnv = process.env,`
- `sources/agentmemory/src/cli/engine-install.ts:248` `): { command: string; args: string[] }[] {`
- `sources/agentmemory/src/cli/engine-install.ts:249` `const systemRoot = env["SystemRoot"] || env["SYSTEMROOT"] || "C:\\Windows";`
- `sources/agentmemory/src/cli/engine-install.ts:251` `return [`
- `sources/agentmemory/src/cli/engine-install.ts:252` `{ command: join(system32, "tar.exe"), args: ["-xf", archive, "-C", binDir] },`
- `sources/agentmemory/src/cli/engine-install.ts:253` `{`
- `sources/agentmemory/src/cli/engine-install.ts:254` `command: join(system32, "WindowsPowerShell", "v1.0", "powershell.exe"),`
- `sources/agentmemory/src/cli/engine-install.ts:255` `args: [`
- `sources/agentmemory/src/cli/engine-install.ts:256` `"-NoProfile",`
- `sources/agentmemory/src/cli/engine-install.ts:258` `"-Command",`
- `sources/agentmemory/src/cli/engine-install.ts:259` `'Expand-Archive -LiteralPath ${powershellLiteral(archive)} -DestinationPath ${powershellLiteral(binDir)} -Force',`
- `sources/agentmemory/src/cli/engine-install.ts:260` `],`
- `sources/agentmemory/src/cli/engine-install.ts:261` `},`
- `sources/agentmemory/src/cli/engine-install.ts:262` `];`
- `sources/agentmemory/src/cli/engine-install.ts:263` `}`
- `sources/agentmemory/src/cli/engine-install.ts:376` `const download = opts.download ?? ((url: string, dest: string) => downloadToFile(url, dest, opts.env ?? process.env, run));`
- `sources/agentmemory/src/cli/engine-install.ts:377` `const archive = iiiZipArchivePath(opts.binDir);`
- `sources/agentmemory/src/cli/engine-install.ts:378` ``
- `sources/agentmemory/src/cli/engine-install.ts:379` `try {`
- `sources/agentmemory/src/cli/engine-install.ts:380` `mkdirSync(opts.binDir, { recursive: true });`
- `sources/agentmemory/src/cli/engine-install.ts:381` `removeStaleInstallFiles(opts.binDir);`
- `sources/agentmemory/src/cli/engine-install.ts:382` `await download(opts.releaseUrl, archive);`
- `sources/agentmemory/src/cli/engine-install.ts:383` `} catch (err) {`
- `sources/agentmemory/src/cli/engine-install.ts:384` `removeQuietly(archive);`
- `sources/agentmemory/src/cli/engine-install.ts:385` `return { ok: false, reason: "download", detail: err instanceof Error ? err.message : String(err) };`
- `sources/agentmemory/src/cli/engine-install.ts:386` `}`
- `sources/agentmemory/src/cli/engine-install.ts:387` ``
- `sources/agentmemory/src/cli/engine-install.ts:388` `let actual: string;`
- `sources/agentmemory/src/cli/engine-install.ts:389` `try {`
- `sources/agentmemory/src/cli/engine-install.ts:390` `actual = sha256File(archive);`
- `sources/agentmemory/src/cli/engine-install.ts:391` `} catch (err) {`
- `sources/agentmemory/src/cli/engine-install.ts:392` `removeQuietly(archive);`
- `sources/agentmemory/src/cli/engine-install.ts:393` `return { ok: false, reason: "download", detail: err instanceof Error ? err.message : String(err) };`
- `sources/agentmemory/src/cli/engine-install.ts:394` `}`
- `sources/agentmemory/src/cli/engine-install.ts:395` `if (actual !== expected) {`
- `sources/agentmemory/src/cli/engine-install.ts:396` `removeQuietly(archive);`
- `sources/agentmemory/src/cli/engine-install.ts:397` `return {`
- `sources/agentmemory/src/cli/engine-install.ts:398` `ok: false,`
- `sources/agentmemory/src/cli/engine-install.ts:399` `reason: "checksum",`
- `sources/agentmemory/src/cli/engine-install.ts:400` `detail: 'SHA-256 mismatch for ${opts.asset}: expected ${expected}, got ${actual}. The download was deleted and nothing was installed',`
- `sources/agentmemory/src/cli/engine-install.ts:401` `};`
- `sources/agentmemory/src/cli/engine-install.ts:402` `}`
- `sources/agentmemory/src/cli/engine-install.ts:403` ``
- `sources/agentmemory/src/cli/engine-install.ts:404` `const staging = join(opts.binDir, '.iii-staging-${process.pid}-${randomBytes(4).toString("hex")}');`
- `sources/agentmemory/src/cli/engine-install.ts:407` `try {`
- `sources/agentmemory/src/cli/engine-install.ts:408` `for (const step of windowsZipExtractCommands(archive, staging, opts.env)) {`
- `sources/agentmemory/src/cli/engine-install.ts:409` `removeQuietly(staging);`
- `sources/agentmemory/src/cli/engine-install.ts:410` `mkdirSync(staging, { recursive: true });`
- `sources/agentmemory/src/cli/engine-install.ts:411` `const result = await run(step.command, step.args);`
- `sources/agentmemory/src/cli/engine-install.ts:412` `if (result.status === 0 && existsSync(staged)) {`
- `sources/agentmemory/src/cli/engine-install.ts:420` `}`
- `sources/agentmemory/src/cli/engine-install.ts:421` `failures.push('${step.command}: ${result.status === 0 ? 'finished but ${basename(opts.binPath)} is missing from the archive' : describeInstallFailure(result)}');`
- `sources/agentmemory/src/cli/engine-install.ts:422` `}`
- `sources/agentmemory/src/cli/engine-install.ts:423` `return { ok: false, reason: "extract", detail: failures.join("; ") };`
- `sources/agentmemory/src/cli/engine-install.ts:424` `} catch (err) {`
- `sources/agentmemory/src/cli/engine-install.ts:425` `failures.push(err instanceof Error ? err.message : String(err));`
- `sources/agentmemory/src/cli/engine-install.ts:428` `removeQuietly(staging);`
- `sources/agentmemory/src/cli/engine-install.ts:429` `removeQuietly(archive);`
- `sources/agentmemory/src/cli/engine-install.ts:430` `}`
- `sources/agentmemory/src/cli/engine-install.ts:431` `}`

**operation and route call sites**, body `sources/agentmemory/src/cli/onboarding.ts:248`; body evidence.

- `sources/agentmemory/src/cli/onboarding.ts:247` `} else {`
- `sources/agentmemory/src/cli/onboarding.ts:248` `lines.push('  ! Could not remove AGENTMEMORY_LLM_PROVIDER from ${envPath}. Delete that line to keep LLM features off.');`
- `sources/agentmemory/src/cli/onboarding.ts:249` `}`
- `sources/agentmemory/src/cli/onboarding.ts:250` `p.note(lines.join("\n"), "ready");`
- `sources/agentmemory/src/cli/onboarding.ts:251` ``
- `sources/agentmemory/src/cli/onboarding.ts:252` `if (agents.length > 0) {`
- `sources/agentmemory/src/cli/onboarding.ts:259` `export function setLlmProviderInEnv(envPath: string | null, provider: string | null): boolean {`
- `sources/agentmemory/src/cli/onboarding.ts:260` `if (provider === null) delete process.env["AGENTMEMORY_LLM_PROVIDER"];`
- `sources/agentmemory/src/cli/onboarding.ts:261` `if (!envPath || !existsSync(envPath)) return provider === null;`
- `sources/agentmemory/src/cli/onboarding.ts:262` `try {`
- `sources/agentmemory/src/cli/onboarding.ts:263` `const current = readFileSync(envPath, "utf-8");`
- `sources/agentmemory/src/cli/onboarding.ts:264` `const kept = current`

**operation and route call sites**, body `sources/agentmemory/src/cli/remove-plan.ts:172`; body evidence.

- `sources/agentmemory/src/cli/remove-plan.ts:171` `id: "pidfile",`
- `sources/agentmemory/src/cli/remove-plan.ts:172` `description: "Delete pidfile",`
- `sources/agentmemory/src/cli/remove-plan.ts:173` `path: pidfilePath(runtimeDir),`
- `sources/agentmemory/src/cli/remove-plan.ts:174` `alwaysAsk: false,`
- `sources/agentmemory/src/cli/remove-plan.ts:175` `applicable: pathExists(pidfilePath(runtimeDir)),`
- `sources/agentmemory/src/cli/remove-plan.ts:176` `sizeBytes: safeSize(pidfilePath(runtimeDir)),`
- `sources/agentmemory/src/cli/remove-plan.ts:180` `id: "worker-pidfile",`
- `sources/agentmemory/src/cli/remove-plan.ts:181` `description: "Delete worker pidfile",`
- `sources/agentmemory/src/cli/remove-plan.ts:182` `path: workerPidfilePath(runtimeDir),`
- `sources/agentmemory/src/cli/remove-plan.ts:183` `alwaysAsk: false,`
- `sources/agentmemory/src/cli/remove-plan.ts:184` `applicable: pathExists(workerPidfilePath(runtimeDir)),`
- `sources/agentmemory/src/cli/remove-plan.ts:185` `sizeBytes: safeSize(workerPidfilePath(runtimeDir)),`
- `sources/agentmemory/src/cli/remove-plan.ts:189` `id: "engine-state",`
- `sources/agentmemory/src/cli/remove-plan.ts:190` `description: "Delete engine-state.json",`
- `sources/agentmemory/src/cli/remove-plan.ts:191` `path: enginePath(runtimeDir),`
- `sources/agentmemory/src/cli/remove-plan.ts:192` `alwaysAsk: false,`
- `sources/agentmemory/src/cli/remove-plan.ts:193` `applicable:`
- `sources/agentmemory/src/cli/remove-plan.ts:194` `!options.preserveRuntimeState && pathExists(enginePath(runtimeDir)),`
- `sources/agentmemory/src/cli/remove-plan.ts:201` `id: "env",`
- `sources/agentmemory/src/cli/remove-plan.ts:202` `description: "Delete .env (your API keys) , will ask separately",`
- `sources/agentmemory/src/cli/remove-plan.ts:203` `path: envPath(home),`
- `sources/agentmemory/src/cli/remove-plan.ts:204` `alwaysAsk: true,`
- `sources/agentmemory/src/cli/remove-plan.ts:205` `applicable: !options.keepData && pathExists(envPath(home)),`
- `sources/agentmemory/src/cli/remove-plan.ts:206` `sizeBytes: safeSize(envPath(home)),`
- `sources/agentmemory/src/cli/remove-plan.ts:210` `id: "preferences",`
- `sources/agentmemory/src/cli/remove-plan.ts:211` `description: "Delete preferences.json",`
- `sources/agentmemory/src/cli/remove-plan.ts:212` `path: preferencesPath(home),`
- `sources/agentmemory/src/cli/remove-plan.ts:213` `alwaysAsk: false,`
- `sources/agentmemory/src/cli/remove-plan.ts:214` `applicable: !options.keepData && pathExists(preferencesPath(home)),`
- `sources/agentmemory/src/cli/remove-plan.ts:215` `sizeBytes: safeSize(preferencesPath(home)),`
- `sources/agentmemory/src/cli/remove-plan.ts:219` `id: "backups",`
- `sources/agentmemory/src/cli/remove-plan.ts:220` `description: "Delete backups/ directory (connect manifest + backups)",`
- `sources/agentmemory/src/cli/remove-plan.ts:221` `path: backupsDir(home),`
- `sources/agentmemory/src/cli/remove-plan.ts:222` `alwaysAsk: false,`
- `sources/agentmemory/src/cli/remove-plan.ts:223` `applicable: !options.keepData && pathExists(backupsDir(home)),`
- `sources/agentmemory/src/cli/remove-plan.ts:224` `sizeBytes: -1,`
- `sources/agentmemory/src/cli/remove-plan.ts:228` `id: "runtime-config",`
- `sources/agentmemory/src/cli/remove-plan.ts:229` `description: "Delete generated iii-config.runtime.yaml",`
- `sources/agentmemory/src/cli/remove-plan.ts:230` `path: runtimeConfigPath(dataDir),`
- `sources/agentmemory/src/cli/remove-plan.ts:231` `alwaysAsk: false,`
- `sources/agentmemory/src/cli/remove-plan.ts:232` `applicable: pathExists(runtimeConfigPath(dataDir)),`
- `sources/agentmemory/src/cli/remove-plan.ts:233` `sizeBytes: safeSize(runtimeConfigPath(dataDir)),`
- `sources/agentmemory/src/cli/remove-plan.ts:258` `id: "private-bin-iii",`
- `sources/agentmemory/src/cli/remove-plan.ts:259` `description: 'Delete ~/.agentmemory/bin/iii (agentmemory's private install)',`
- `sources/agentmemory/src/cli/remove-plan.ts:260` `path: privIii,`
- `sources/agentmemory/src/cli/remove-plan.ts:261` `alwaysAsk: false,`
- `sources/agentmemory/src/cli/remove-plan.ts:262` `applicable: true,`
- `sources/agentmemory/src/cli/remove-plan.ts:263` `sizeBytes: safeSize(privIii),`
- `sources/agentmemory/src/cli/remove-plan.ts:277` `description: matches`
- `sources/agentmemory/src/cli/remove-plan.ts:278` `? 'Delete ~/.local/bin/iii (legacy install location, matches pinned v${pinnedVersion})'`
- `sources/agentmemory/src/cli/remove-plan.ts:279` `: 'Delete ~/.local/bin/iii (legacy install location, version ${localBinIiiVersion ?? "unknown"} != pinned v${pinnedVersion}) , will ask',`
- `sources/agentmemory/src/cli/remove-plan.ts:280` `path: legacyIii,`
- `sources/agentmemory/src/cli/remove-plan.ts:281` `alwaysAsk: !matches,`
- `sources/agentmemory/src/cli/remove-plan.ts:282` `applicable: true,`
- `sources/agentmemory/src/cli/remove-plan.ts:283` `sizeBytes: safeSize(legacyIii),`
- `sources/agentmemory/src/cli/remove-plan.ts:290` `id: "data-dir",`
- `sources/agentmemory/src/cli/remove-plan.ts:291` `description: 'Delete memory data directory (${dataDir}) , will ask separately',`
- `sources/agentmemory/src/cli/remove-plan.ts:292` `path: dataDir,`
- `sources/agentmemory/src/cli/remove-plan.ts:293` `alwaysAsk: true,`
- `sources/agentmemory/src/cli/remove-plan.ts:294` `applicable: !options.keepData && pathExists(dataDir),`
- `sources/agentmemory/src/cli/remove-plan.ts:295` `sizeBytes: -1,`

**operation and route call sites**, body `sources/agentmemory/src/cli.ts:61`; body evidence.

- `sources/agentmemory/src/cli.ts:60` `import { embeddingStatusLabel } from "./cli/embedding-status.js";`
- `sources/agentmemory/src/cli.ts:61` `import { expectedIiiSha256, iiiManualInstallCommand, installIiiArchive, installIiiZip } from "./cli/engine-install.js";`
- `sources/agentmemory/src/cli.ts:62` `import {`
- `sources/agentmemory/src/cli.ts:63` `runtimeConfigPath,`
- `sources/agentmemory/src/cli.ts:64` `redactCredentialUrls,`
- `sources/agentmemory/src/cli.ts:65` `} from "./cli/engine-launch.js";`
- `sources/agentmemory/src/cli.ts:976` `spinner.stop('${label} ${pc.green("✓")}');`
- `sources/agentmemory/src/cli.ts:977` `iiiProbeScan?.delete(binPath);`
- `sources/agentmemory/src/cli.ts:978` `return { ok: true, binPath };`
- `sources/agentmemory/src/cli.ts:979` `}`
- `sources/agentmemory/src/cli.ts:980` `spinner.stop('${label} ${pc.red("✗")}');`
- `sources/agentmemory/src/cli.ts:981` `p.log.warn(`
- `sources/agentmemory/src/cli.ts:1006` `spinner.start(label);`
- `sources/agentmemory/src/cli.ts:1007` `const outcome = installIiiArchive({`
- `sources/agentmemory/src/cli.ts:1008` `sh: shBin,`
- `sources/agentmemory/src/cli.ts:1009` `releaseUrl,`
- `sources/agentmemory/src/cli.ts:1010` `version: IIPINNED_VERSION,`
- `sources/agentmemory/src/cli.ts:1011` `asset: asset ?? "",`
- `sources/agentmemory/src/cli.ts:1025` `? ""`
- `sources/agentmemory/src/cli.ts:1026` `: 'Compare the archive against ${releaseUrl.replace(/\.(tar\.gz|zip)$/, ".sha256")} before extracting.\n';`
- `sources/agentmemory/src/cli.ts:1027` `p.log.warn(`
- `sources/agentmemory/src/cli.ts:1028` `'${heading}: ${outcome.detail.slice(0, 300)}\n' +`
- `sources/agentmemory/src/cli.ts:1029` `'Install it manually, then re-run agentmemory:\n' +`
- `sources/agentmemory/src/cli.ts:1030` `'  ${iiiManualInstallCommand(releaseUrl, binDir, binPath, sha256)}\n' +`
- `sources/agentmemory/src/cli.ts:1035` `spinner.stop('${label} ${pc.green("✓")}');`
- `sources/agentmemory/src/cli.ts:1036` `iiiProbeScan?.delete(binPath);`
- `sources/agentmemory/src/cli.ts:1037` `return { ok: true, binPath };`
- `sources/agentmemory/src/cli.ts:1038` `}`
- `sources/agentmemory/src/cli.ts:1039` ``
- `sources/agentmemory/src/cli.ts:1040` `type StartupFailure = {`
- `sources/agentmemory/src/cli.ts:2854` `'Viewer:        ${c.url(getViewerUrl())}',`
- `sources/agentmemory/src/cli.ts:2855` `'Clean up with: ${c.dim('curl -X DELETE "${base}/agentmemory/sessions?project=${demoProject}"')}',`
- `sources/agentmemory/src/cli.ts:2856` `];`
- `sources/agentmemory/src/cli.ts:2857` ``
- `sources/agentmemory/src/cli.ts:2858` `p.note(lines.join("\n"), "demo complete");`
- `sources/agentmemory/src/cli.ts:2859` `p.log.success("agentmemory is working. Point your agent at it and get back to coding.");`
- `sources/agentmemory/src/cli.ts:3082` `if (pidfilePid) enginePids.add(pidfilePid);`
- `sources/agentmemory/src/cli.ts:3083` `if (workerPid) enginePids.delete(workerPid);`
- `sources/agentmemory/src/cli.ts:3084` ``
- `sources/agentmemory/src/cli.ts:3085` `const foreign = [...enginePids]`
- `sources/agentmemory/src/cli.ts:3086` `.map((pid) => ({ pid, comm: pidCommand(pid) }))`
- `sources/agentmemory/src/cli.ts:3087` `.filter(({ comm }) => isForeignPortHolder(comm));`
- `sources/agentmemory/src/cli.ts:3553` ``
- `sources/agentmemory/src/cli.ts:3554` `function safeDelete(path: string): { ok: boolean; message: string } {`
- `sources/agentmemory/src/cli.ts:3555` `try {`
- `sources/agentmemory/src/cli.ts:3556` `if (!existsSync(path)) return { ok: true, message: 'not present (${path})' };`
- `sources/agentmemory/src/cli.ts:3557` `const st = statSync(path);`
- `sources/agentmemory/src/cli.ts:3558` `if (st.isDirectory()) {`
- `sources/agentmemory/src/cli.ts:3562` `}`
- `sources/agentmemory/src/cli.ts:3563` `return { ok: true, message: 'deleted ${path}' };`
- `sources/agentmemory/src/cli.ts:3564` `} catch (err) {`
- `sources/agentmemory/src/cli.ts:3565` `return {`
- `sources/agentmemory/src/cli.ts:3566` `ok: false,`
- `sources/agentmemory/src/cli.ts:3567` `message: 'failed ${path}: ${err instanceof Error ? err.message : String(err)}',`
- `sources/agentmemory/src/cli.ts:3623` `const proceed = await p.confirm({`
- `sources/agentmemory/src/cli.ts:3624` `message: "Proceed with these deletions?",`
- `sources/agentmemory/src/cli.ts:3625` `initialValue: false,`
- `sources/agentmemory/src/cli.ts:3626` `});`
- `sources/agentmemory/src/cli.ts:3627` `if (p.isCancel(proceed) || proceed !== true) {`
- `sources/agentmemory/src/cli.ts:3628` `p.cancel("Cancelled. Nothing was deleted.");`
- `sources/agentmemory/src/cli.ts:3629` `return;`
- `sources/agentmemory/src/cli.ts:3630` `}`
- `sources/agentmemory/src/cli.ts:3631` `const sure = await p.confirm({`
- `sources/agentmemory/src/cli.ts:3632` `message: "This is irreversible. Continue?",`
- `sources/agentmemory/src/cli.ts:3635` `if (p.isCancel(sure) || sure !== true) {`
- `sources/agentmemory/src/cli.ts:3636` `p.cancel("Cancelled. Nothing was deleted.");`
- `sources/agentmemory/src/cli.ts:3637` `return;`
- `sources/agentmemory/src/cli.ts:3638` `}`
- `sources/agentmemory/src/cli.ts:3639` `}`
- `sources/agentmemory/src/cli.ts:3640` ``
- `sources/agentmemory/src/cli.ts:3646` `const ok = await p.confirm({`
- `sources/agentmemory/src/cli.ts:3647` `message: '${item.description} , really delete${item.path ? ' ${item.path}' : ""}?',`
- `sources/agentmemory/src/cli.ts:3648` `initialValue: false,`
- `sources/agentmemory/src/cli.ts:3649` `});`
- `sources/agentmemory/src/cli.ts:3650` `if (p.isCancel(ok) || ok !== true) {`
- `sources/agentmemory/src/cli.ts:3651` `p.log.info('skipped: ${item.id}');`
- `sources/agentmemory/src/cli.ts:3670` `if (!item.path) continue;`
- `sources/agentmemory/src/cli.ts:3671` `const r = safeDelete(item.path);`
- `sources/agentmemory/src/cli.ts:3672` `if (r.ok) p.log.success(r.message);`
- `sources/agentmemory/src/cli.ts:3673` `else p.log.error(r.message);`
- `sources/agentmemory/src/cli.ts:3674` `}`
- `sources/agentmemory/src/cli.ts:3675` ``

**operation and route call sites**, body `sources/agentmemory/src/functions/access-tracker.ts:93`; body evidence.

- `sources/agentmemory/src/functions/access-tracker.ts:92` ``
- `sources/agentmemory/src/functions/access-tracker.ts:93` `export async function deleteAccessLog(`
- `sources/agentmemory/src/functions/access-tracker.ts:94` `kv: StateKV,`
- `sources/agentmemory/src/functions/access-tracker.ts:95` `memoryId: string,`
- `sources/agentmemory/src/functions/access-tracker.ts:96` `): Promise<void> {`
- `sources/agentmemory/src/functions/access-tracker.ts:97` `if (!memoryId) return;`
- `sources/agentmemory/src/functions/access-tracker.ts:99` `await withKeyedLock('mem:access:${memoryId}', async () => {`
- `sources/agentmemory/src/functions/access-tracker.ts:100` `await kv.delete(KV.accessLog, memoryId);`
- `sources/agentmemory/src/functions/access-tracker.ts:101` `});`
- `sources/agentmemory/src/functions/access-tracker.ts:102` `} catch {}`
- `sources/agentmemory/src/functions/access-tracker.ts:103` `}`
- `sources/agentmemory/src/functions/access-tracker.ts:104` ``

**operation and route call sites**, body `sources/agentmemory/src/functions/audit.ts:14`; body evidence.

- `sources/agentmemory/src/functions/audit.ts:13` `const AUDIT_MIGRATION_COPY_CONCURRENCY = 32;`
- `sources/agentmemory/src/functions/audit.ts:14` `const AUDIT_MIGRATION_DELETE_CONCURRENCY = 8;`
- `sources/agentmemory/src/functions/audit.ts:15` `const AUDIT_MONTHS_LOCK = KV.auditMonths;`
- `sources/agentmemory/src/functions/audit.ts:16` `const AUDIT_MIGRATION_LOCK = "mem:audit:migration-lock";`
- `sources/agentmemory/src/functions/audit.ts:17` `export const AUDIT_MIGRATION_STATE_KEY = "migration";`
- `sources/agentmemory/src/functions/audit.ts:18` `const AUDIT_MIGRATION_DELETE_WAIT_INTERVALS = 2;`
- `sources/agentmemory/src/functions/audit.ts:19` `const STATE_STORE_DIR_NAME = "state_store.db";`
- `sources/agentmemory/src/functions/audit.ts:20` ``
- `sources/agentmemory/src/functions/audit.ts:21` `function sleep(ms: number): Promise<void> {`
- `sources/agentmemory/src/functions/audit.ts:22` `return new Promise((resolve) => setTimeout(resolve, ms));`
- `sources/agentmemory/src/functions/audit.ts:87` ``
- `sources/agentmemory/src/functions/audit.ts:88` `async function auditMigrationDeleteDelayMs(): Promise<number> {`
- `sources/agentmemory/src/functions/audit.ts:89` `const dataDir = getEnvVar("AGENTMEMORY_DATA_DIR");`
- `sources/agentmemory/src/functions/audit.ts:90` `if (!dataDir) return ENGINE_DEFAULT_SAVE_INTERVAL_MS * AUDIT_MIGRATION_DELETE_WAIT_INTERVALS;`
- `sources/agentmemory/src/functions/audit.ts:91` `const configTexts: string[] = [];`
- `sources/agentmemory/src/functions/audit.ts:92` `for (const path of engineStateConfigPaths(dataDir, runtimeConfigPath(dataDir))) {`
- `sources/agentmemory/src/functions/audit.ts:93` `try {`
- `sources/agentmemory/src/functions/audit.ts:94` `configTexts.push(await readFile(path, "utf-8"));`
- `sources/agentmemory/src/functions/audit.ts:96` `}`
- `sources/agentmemory/src/functions/audit.ts:97` `return engineSaveIntervalMs(configTexts) * AUDIT_MIGRATION_DELETE_WAIT_INTERVALS;`
- `sources/agentmemory/src/functions/audit.ts:98` `}`
- `sources/agentmemory/src/functions/audit.ts:99` ``
- `sources/agentmemory/src/functions/audit.ts:100` `async function readAuditMigrationState(kv: StateKV): Promise<AuditMigrationState | null> {`
- `sources/agentmemory/src/functions/audit.ts:101` `return kv.get<AuditMigrationState>(KV.auditMonths, AUDIT_MIGRATION_STATE_KEY);`
- `sources/agentmemory/src/functions/audit.ts:315` `probe?: () => Promise<LegacyAuditProbe>;`
- `sources/agentmemory/src/functions/audit.ts:316` `deleteDelayMs?: () => Promise<number>;`
- `sources/agentmemory/src/functions/audit.ts:317` `}`
- `sources/agentmemory/src/functions/audit.ts:318` ``
- `sources/agentmemory/src/functions/audit.ts:319` `export async function startAuditMigration(`
- `sources/agentmemory/src/functions/audit.ts:320` `kv: StateKV,`
- `sources/agentmemory/src/functions/audit.ts:325` `deps.probe ?? (onRedis ? () => probeLegacyAuditScope(kv) : probeLegacyAuditFile);`
- `sources/agentmemory/src/functions/audit.ts:326` `const runDeleteDelayMs =`
- `sources/agentmemory/src/functions/audit.ts:327` `deps.deleteDelayMs ?? (onRedis ? async () => 0 : auditMigrationDeleteDelayMs);`
- `sources/agentmemory/src/functions/audit.ts:328` ``
- `sources/agentmemory/src/functions/audit.ts:329` `const marker = await readAuditMigrationState(kv);`
- `sources/agentmemory/src/functions/audit.ts:330` `if (marker?.status === "done") return;`
- `sources/agentmemory/src/functions/audit.ts:331` ``
- `sources/agentmemory/src/functions/audit.ts:339` `migrated: marker?.migrated ?? 0,`
- `sources/agentmemory/src/functions/audit.ts:340` `purged: marker?.purged ?? 0,`
- `sources/agentmemory/src/functions/audit.ts:341` `summaryWritten: marker?.summaryWritten ?? true,`
- `sources/agentmemory/src/functions/audit.ts:342` `checkedAt: new Date().toISOString(),`
- `sources/agentmemory/src/functions/audit.ts:343` `});`
- `sources/agentmemory/src/functions/audit.ts:344` `return;`
- `sources/agentmemory/src/functions/audit.ts:352` `migrated: marker?.migrated ?? 0,`
- `sources/agentmemory/src/functions/audit.ts:353` `purged: marker?.purged ?? 0,`
- `sources/agentmemory/src/functions/audit.ts:354` `summaryWritten: marker?.summaryWritten ?? false,`
- `sources/agentmemory/src/functions/audit.ts:355` `checkedAt: new Date().toISOString(),`
- `sources/agentmemory/src/functions/audit.ts:356` `});`
- `sources/agentmemory/src/functions/audit.ts:357` `logger.warn("legacy audit log left frozen; migration only reads it locally and under a size cap", {`
- `sources/agentmemory/src/functions/audit.ts:364` `let migrated = marker?.migrated ?? 0;`
- `sources/agentmemory/src/functions/audit.ts:365` `let purged = marker?.purged ?? 0;`
- `sources/agentmemory/src/functions/audit.ts:366` ``
- `sources/agentmemory/src/functions/audit.ts:367` `if (marker?.status !== "copied") {`
- `sources/agentmemory/src/functions/audit.ts:368` `let legacy: AuditEntry[];`
- `sources/agentmemory/src/functions/audit.ts:369` `try {`
- `sources/agentmemory/src/functions/audit.ts:382` `migrated,`
- `sources/agentmemory/src/functions/audit.ts:383` `purged,`
- `sources/agentmemory/src/functions/audit.ts:384` `summaryWritten: marker?.summaryWritten ?? true,`
- `sources/agentmemory/src/functions/audit.ts:385` `checkedAt: new Date().toISOString(),`
- `sources/agentmemory/src/functions/audit.ts:386` `});`
- `sources/agentmemory/src/functions/audit.ts:387` `return;`
- `sources/agentmemory/src/functions/audit.ts:396` `const kept = deduped.filter((entry) => entry.operation !== "index_persist");`
- `sources/agentmemory/src/functions/audit.ts:397` `purged = deduped.length - kept.length;`
- `sources/agentmemory/src/functions/audit.ts:398` `migrated = kept.length;`
- `sources/agentmemory/src/functions/audit.ts:399` ``
- `sources/agentmemory/src/functions/audit.ts:400` `try {`
- `sources/agentmemory/src/functions/audit.ts:401` `const months = new Set<string>();`
- `sources/agentmemory/src/functions/audit.ts:420` `migrated,`
- `sources/agentmemory/src/functions/audit.ts:421` `purged,`
- `sources/agentmemory/src/functions/audit.ts:422` `summaryWritten: false,`
- `sources/agentmemory/src/functions/audit.ts:423` `checkedAt: new Date().toISOString(),`
- `sources/agentmemory/src/functions/audit.ts:424` `});`
- `sources/agentmemory/src/functions/audit.ts:425` `}`
- `sources/agentmemory/src/functions/audit.ts:426` ``
- `sources/agentmemory/src/functions/audit.ts:427` `await sleep(await runDeleteDelayMs());`
- `sources/agentmemory/src/functions/audit.ts:428` ``
- `sources/agentmemory/src/functions/audit.ts:429` `let remaining: AuditEntry[];`
- `sources/agentmemory/src/functions/audit.ts:430` `try {`
- `sources/agentmemory/src/functions/audit.ts:431` `remaining = await collectAll<AuditEntry>(kv, KV.audit);`
- `sources/agentmemory/src/functions/audit.ts:433` `logger.warn(`
- `sources/agentmemory/src/functions/audit.ts:434` `"audit migration could not re-read the legacy audit log before deleting; it resumes on the next boot",`
- `sources/agentmemory/src/functions/audit.ts:435` `{ error: err instanceof Error ? err.message : String(err) },`
- `sources/agentmemory/src/functions/audit.ts:436` `);`
- `sources/agentmemory/src/functions/audit.ts:437` `return;`
- `sources/agentmemory/src/functions/audit.ts:438` `}`
- `sources/agentmemory/src/functions/audit.ts:442` `remaining.slice().reverse(),`
- `sources/agentmemory/src/functions/audit.ts:443` `AUDIT_MIGRATION_DELETE_CONCURRENCY,`
- `sources/agentmemory/src/functions/audit.ts:444` `async (entry) => {`
- `sources/agentmemory/src/functions/audit.ts:445` `await kv.delete(KV.audit, entry.id);`
- `sources/agentmemory/src/functions/audit.ts:446` `},`
- `sources/agentmemory/src/functions/audit.ts:447` `);`
- `sources/agentmemory/src/functions/audit.ts:448` `} catch (err) {`
- `sources/agentmemory/src/functions/audit.ts:449` `logger.warn(`
- `sources/agentmemory/src/functions/audit.ts:450` `"audit migration stopped before finishing the delete pass; it resumes on the next boot",`
- `sources/agentmemory/src/functions/audit.ts:451` `{ error: err instanceof Error ? err.message : String(err) },`
- `sources/agentmemory/src/functions/audit.ts:452` `);`
- `sources/agentmemory/src/functions/audit.ts:453` `return;`
- `sources/agentmemory/src/functions/audit.ts:454` `}`
- `sources/agentmemory/src/functions/audit.ts:458` `if (current?.summaryWritten) return;`
- `sources/agentmemory/src/functions/audit.ts:459` `if (purged > 0) {`
- `sources/agentmemory/src/functions/audit.ts:460` `await safeAudit(kv, "audit_migrate", AUDIT_MIGRATE_FUNCTION_ID, [], {`
- `sources/agentmemory/src/functions/audit.ts:461` `reason: "index_persist rows predate opt-in auditing",`
- `sources/agentmemory/src/functions/audit.ts:462` `purged,`
- `sources/agentmemory/src/functions/audit.ts:463` `migrated,`
- `sources/agentmemory/src/functions/audit.ts:464` `});`
- `sources/agentmemory/src/functions/audit.ts:465` `}`
- `sources/agentmemory/src/functions/audit.ts:466` `await writeAuditMigrationState(kv, {`
- `sources/agentmemory/src/functions/audit.ts:470` `migrated,`
- `sources/agentmemory/src/functions/audit.ts:471` `purged,`
- `sources/agentmemory/src/functions/audit.ts:472` `summaryWritten: true,`
- `sources/agentmemory/src/functions/audit.ts:473` `checkedAt: new Date().toISOString(),`
- `sources/agentmemory/src/functions/audit.ts:474` `});`
- `sources/agentmemory/src/functions/audit.ts:475` `});`
- `sources/agentmemory/src/functions/audit.ts:476` ``
- `sources/agentmemory/src/functions/audit.ts:477` `logger.info("audit log moved to monthly scopes", { migrated, purged });`
- `sources/agentmemory/src/functions/audit.ts:478` `}`
- `sources/agentmemory/src/functions/audit.ts:479` ``
- `sources/agentmemory/src/functions/audit.ts:480` `export async function runAuditRetentionSweep(`
- `sources/agentmemory/src/functions/audit.ts:481` `kv: StateKV,`
- `sources/agentmemory/src/functions/audit.ts:498` `for (const row of rows.slice().reverse()) {`
- `sources/agentmemory/src/functions/audit.ts:499` `await kv.delete(KV.auditMonth(month), row.id);`
- `sources/agentmemory/src/functions/audit.ts:500` `droppedRows++;`
- `sources/agentmemory/src/functions/audit.ts:501` `}`
- `sources/agentmemory/src/functions/audit.ts:502` `}`
- `sources/agentmemory/src/functions/audit.ts:503` ``

**operation and route call sites**, body `sources/agentmemory/src/functions/auto-forget.ts:2`; body evidence.

- `sources/agentmemory/src/functions/auto-forget.ts:1` `import type { IIIClient } from "iii-sdk";`
- `sources/agentmemory/src/functions/auto-forget.ts:2` `import { markCaptureEventDeleted } from "../capture/event-record.js";`
- `sources/agentmemory/src/functions/auto-forget.ts:3` `import type { Memory, CompressedObservation, Session } from "../types.js";`
- `sources/agentmemory/src/functions/auto-forget.ts:4` `import { KV } from "../state/schema.js";`
- `sources/agentmemory/src/functions/auto-forget.ts:5` `import { StateKV, collectAll } from "../state/kv.js";`
- `sources/agentmemory/src/functions/auto-forget.ts:6` `import { supersededAt } from "../state/memory-utils.js";`
- `sources/agentmemory/src/functions/auto-forget.ts:9` `import { recordAudit } from "./audit.js";`
- `sources/agentmemory/src/functions/auto-forget.ts:10` `import { deleteAccessLog } from "./access-tracker.js";`
- `sources/agentmemory/src/functions/auto-forget.ts:11` `import { deleteFacetsForTargets } from "./facets.js";`
- `sources/agentmemory/src/functions/auto-forget.ts:12` `import { getSearchIndex, vectorIndexRemove, flushIndexSave } from "./search.js";`
- `sources/agentmemory/src/functions/auto-forget.ts:13` `import { logger } from "../logger.js";`
- `sources/agentmemory/src/functions/auto-forget.ts:14` ``
- `sources/agentmemory/src/functions/auto-forget.ts:15` `const MS_PER_DAY = 24 * 60 * 60 * 1000;`
- `sources/agentmemory/src/functions/auto-forget.ts:17` ``
- `sources/agentmemory/src/functions/auto-forget.ts:18` `interface AutoForgetResult {`
- `sources/agentmemory/src/functions/auto-forget.ts:19` `ttlExpired: string[];`
- `sources/agentmemory/src/functions/auto-forget.ts:20` `contradictions: Array<{`
- `sources/agentmemory/src/functions/auto-forget.ts:21` `memoryA: string;`
- `sources/agentmemory/src/functions/auto-forget.ts:22` `memoryB: string;`
- `sources/agentmemory/src/functions/auto-forget.ts:28` ``
- `sources/agentmemory/src/functions/auto-forget.ts:29` `export function registerAutoForgetFunction(sdk: IIIClient, kv: StateKV): void {`
- `sources/agentmemory/src/functions/auto-forget.ts:30` `sdk.registerFunction("mem::auto-forget",`
- `sources/agentmemory/src/functions/auto-forget.ts:31` `async (data: { dryRun?: boolean }): Promise<AutoForgetResult> => {`
- `sources/agentmemory/src/functions/auto-forget.ts:32` `const dryRun = data?.dryRun ?? false;`
- `sources/agentmemory/src/functions/auto-forget.ts:33` `const now = Date.now();`
- `sources/agentmemory/src/functions/auto-forget.ts:34` `const { decrementImageRef } = await import("./image-refs.js");`
- `sources/agentmemory/src/functions/auto-forget.ts:35` ``
- `sources/agentmemory/src/functions/auto-forget.ts:36` `const result: AutoForgetResult = {`
- `sources/agentmemory/src/functions/auto-forget.ts:37` `ttlExpired: [],`
- `sources/agentmemory/src/functions/auto-forget.ts:38` `contradictions: [],`
- `sources/agentmemory/src/functions/auto-forget.ts:39` `lowValueObs: [],`
- `sources/agentmemory/src/functions/auto-forget.ts:40` `dryRun,`
- `sources/agentmemory/src/functions/auto-forget.ts:43` `const memories = await collectAll<Memory>(kv, KV.memories);`
- `sources/agentmemory/src/functions/auto-forget.ts:44` `const deletedIds = new Set<string>();`
- `sources/agentmemory/src/functions/auto-forget.ts:45` `const facetIds = new Set<string>();`
- `sources/agentmemory/src/functions/auto-forget.ts:46` `try {`
- `sources/agentmemory/src/functions/auto-forget.ts:47` `for (const mem of memories) {`
- `sources/agentmemory/src/functions/auto-forget.ts:48` `if (mem.forgetAfter) {`
- `sources/agentmemory/src/functions/auto-forget.ts:49` `const expiry = new Date(mem.forgetAfter).getTime();`
- `sources/agentmemory/src/functions/auto-forget.ts:50` `if (now > expiry) {`
- `sources/agentmemory/src/functions/auto-forget.ts:51` `result.ttlExpired.push(mem.id);`
- `sources/agentmemory/src/functions/auto-forget.ts:52` `deletedIds.add(mem.id);`
- `sources/agentmemory/src/functions/auto-forget.ts:53` `if (!dryRun) {`
- `sources/agentmemory/src/functions/auto-forget.ts:54` `if (mem.imageRef) {`
- `sources/agentmemory/src/functions/auto-forget.ts:55` `await decrementImageRef(kv, sdk, mem.imageRef);`
- `sources/agentmemory/src/functions/auto-forget.ts:56` `}`
- `sources/agentmemory/src/functions/auto-forget.ts:57` `await kv.delete(KV.memories, mem.id);`
- `sources/agentmemory/src/functions/auto-forget.ts:58` `facetIds.add(mem.id);`
- `sources/agentmemory/src/functions/auto-forget.ts:59` `await recordAudit(kv, "delete", "mem::auto-forget", [mem.id], {`
- `sources/agentmemory/src/functions/auto-forget.ts:60` `resource: "memory",`
- `sources/agentmemory/src/functions/auto-forget.ts:61` `reason: "auto-forget TTL",`
- `sources/agentmemory/src/functions/auto-forget.ts:62` `timestamp: mem.forgetAfter,`
- `sources/agentmemory/src/functions/auto-forget.ts:63` `});`
- `sources/agentmemory/src/functions/auto-forget.ts:64` `await deleteAccessLog(kv, mem.id);`
- `sources/agentmemory/src/functions/auto-forget.ts:65` `getSearchIndex().remove(mem.id);`
- `sources/agentmemory/src/functions/auto-forget.ts:66` `vectorIndexRemove(mem.id);`
- `sources/agentmemory/src/functions/auto-forget.ts:67` `}`
- `sources/agentmemory/src/functions/auto-forget.ts:68` `}`
- `sources/agentmemory/src/functions/auto-forget.ts:71` `} finally {`
- `sources/agentmemory/src/functions/auto-forget.ts:72` `await deleteFacetsForTargets(kv, facetIds);`
- `sources/agentmemory/src/functions/auto-forget.ts:73` `}`
- `sources/agentmemory/src/functions/auto-forget.ts:74` ``
- `sources/agentmemory/src/functions/auto-forget.ts:75` `const latestMemories = memories`
- `sources/agentmemory/src/functions/auto-forget.ts:76` `.filter((m) => m.isLatest !== false && !deletedIds.has(m.id))`
- `sources/agentmemory/src/functions/auto-forget.ts:77` `.sort(`
- `sources/agentmemory/src/functions/auto-forget.ts:78` `(a, b) =>`
- `sources/agentmemory/src/functions/auto-forget.ts:79` `new Date(b.createdAt).getTime() - new Date(a.createdAt).getTime(),`
- `sources/agentmemory/src/functions/auto-forget.ts:80` `)`
- `sources/agentmemory/src/functions/auto-forget.ts:146` `await kv.set(KV.memories, older.id, older);`
- `sources/agentmemory/src/functions/auto-forget.ts:147` `await recordAudit(kv, "forget", "mem::auto-forget", [older.id], {`
- `sources/agentmemory/src/functions/auto-forget.ts:148` `resource: "memory",`
- `sources/agentmemory/src/functions/auto-forget.ts:149` `reason: "auto-forget contradiction",`
- `sources/agentmemory/src/functions/auto-forget.ts:150` `olderId: older.id,`
- `sources/agentmemory/src/functions/auto-forget.ts:151` `similarity: sim,`
- `sources/agentmemory/src/functions/auto-forget.ts:152` `});`
- `sources/agentmemory/src/functions/auto-forget.ts:153` `}`
- `sources/agentmemory/src/functions/auto-forget.ts:185` `if (!(currentAge > 180 * MS_PER_DAY && (current.importance ?? 5) <= 2)) return null;`
- `sources/agentmemory/src/functions/auto-forget.ts:186` `await markCaptureEventDeleted(kv, current);`
- `sources/agentmemory/src/functions/auto-forget.ts:187` `await kv.delete(KV.observations(sessions[i].id), obs.id);`
- `sources/agentmemory/src/functions/auto-forget.ts:188` `getSearchIndex().remove(obs.id);`
- `sources/agentmemory/src/functions/auto-forget.ts:189` `vectorIndexRemove(obs.id);`
- `sources/agentmemory/src/functions/auto-forget.ts:190` `return current;`
- `sources/agentmemory/src/functions/auto-forget.ts:191` `});`
- `sources/agentmemory/src/functions/auto-forget.ts:192` `} catch {`
- `sources/agentmemory/src/functions/auto-forget.ts:193` `logger.warn("Auto-forget observation cleanup failed");`
- `sources/agentmemory/src/functions/auto-forget.ts:194` `}`
- `sources/agentmemory/src/functions/auto-forget.ts:195` `if (removed) {`
- `sources/agentmemory/src/functions/auto-forget.ts:196` `await unindexObservationSession(kv, obs.id).catch(() => {});`
- `sources/agentmemory/src/functions/auto-forget.ts:197` `if (removed.imageData) await decrementImageRef(kv, sdk, removed.imageData);`
- `sources/agentmemory/src/functions/auto-forget.ts:200` `}`
- `sources/agentmemory/src/functions/auto-forget.ts:201` `await recordAudit(kv, "delete", "mem::auto-forget", [obs.id], {`
- `sources/agentmemory/src/functions/auto-forget.ts:202` `resource: "observation",`
- `sources/agentmemory/src/functions/auto-forget.ts:203` `reason: "auto-forget low-value observation",`
- `sources/agentmemory/src/functions/auto-forget.ts:204` `sessionId: sessions[i].id,`
- `sources/agentmemory/src/functions/auto-forget.ts:205` `timestamp: obs.timestamp,`
- `sources/agentmemory/src/functions/auto-forget.ts:206` `});`
- `sources/agentmemory/src/functions/auto-forget.ts:207` `}`
- `sources/agentmemory/src/functions/auto-forget.ts:216` ``
- `sources/agentmemory/src/functions/auto-forget.ts:217` `logger.info("Auto-forget complete", {`
- `sources/agentmemory/src/functions/auto-forget.ts:218` `ttlExpired: result.ttlExpired.length,`
- `sources/agentmemory/src/functions/auto-forget.ts:219` `contradictions: result.contradictions.length,`
- `sources/agentmemory/src/functions/auto-forget.ts:220` `lowValueObs: result.lowValueObs.length,`
- `sources/agentmemory/src/functions/auto-forget.ts:221` `dryRun,`

**operation and route call sites**, body `sources/agentmemory/src/functions/capture.ts:238`; body evidence.

- `sources/agentmemory/src/functions/capture.ts:237` `if (rec.eventSource === "client") await kv.set(eventScope(rec.key), rec.key, done);`
- `sources/agentmemory/src/functions/capture.ts:238` `await kv.delete(KV.captureInbox, rec.key);`
- `sources/agentmemory/src/functions/capture.ts:239` `if (inboxSize !== null) inboxSize = Math.max(0, inboxSize - 1);`
- `sources/agentmemory/src/functions/capture.ts:240` `counters.completed++;`
- `sources/agentmemory/src/functions/capture.ts:241` `}`
- `sources/agentmemory/src/functions/capture.ts:242` ``
- `sources/agentmemory/src/functions/capture.ts:283` `const dead: InboxRecord = { ...rec, status: "dead", updatedAt: now.toISOString(), deadAt: now.toISOString(), lastError: message };`
- `sources/agentmemory/src/functions/capture.ts:284` `delete dead.nextAttemptAt;`
- `sources/agentmemory/src/functions/capture.ts:285` `await kv.set(KV.captureInbox, rec.key, dead);`
- `sources/agentmemory/src/functions/capture.ts:286` `counters.deadLettered++;`
- `sources/agentmemory/src/functions/capture.ts:287` `logger.warn("capture moved to dead letters", { eventId: rec.eventId, attempts: rec.attempts, error: message });`
- `sources/agentmemory/src/functions/capture.ts:288` `return { status: "rejected", eventId: rec.eventId, error: message, retryable: false, observationId: rec.observationId };`
- `sources/agentmemory/src/functions/capture.ts:306` `const done = await kv.get<CompletedEvent>(eventScope(key), key);`
- `sources/agentmemory/src/functions/capture.ts:307` `if (done?.state === "deleted") {`
- `sources/agentmemory/src/functions/capture.ts:308` `counters.duplicates++;`
- `sources/agentmemory/src/functions/capture.ts:309` `return { status: "duplicate", state: "completed", eventId, observationId: done.observationId, deduplicated: true };`
- `sources/agentmemory/src/functions/capture.ts:310` `}`
- `sources/agentmemory/src/functions/capture.ts:311` `if (done) {`
- `sources/agentmemory/src/functions/capture.ts:362` `if (!force && rec.status === "retrying" && rec.nextAttemptAt && Date.parse(rec.nextAttemptAt) > Date.now()) return "waiting";`
- `sources/agentmemory/src/functions/capture.ts:363` `if (rec.eventSource === "client" && (await kv.get<CompletedEvent>(eventScope(key), key))?.state === "deleted") {`
- `sources/agentmemory/src/functions/capture.ts:364` `await kv.delete(KV.captureInbox, key);`
- `sources/agentmemory/src/functions/capture.ts:365` `if (inboxSize !== null) inboxSize = Math.max(0, inboxSize - 1);`
- `sources/agentmemory/src/functions/capture.ts:366` `counters.duplicates++;`
- `sources/agentmemory/src/functions/capture.ts:367` `return "gone";`
- `sources/agentmemory/src/functions/capture.ts:368` `}`
- `sources/agentmemory/src/functions/capture.ts:418` `const ev = sorted[j]!;`
- `sources/agentmemory/src/functions/capture.ts:419` `if (j >= perShard || Date.parse(ev.deletedAt ?? ev.completedAt) < cutoff) {`
- `sources/agentmemory/src/functions/capture.ts:420` `await kv.delete(scope, ev.key).catch(() => {});`
- `sources/agentmemory/src/functions/capture.ts:421` `removed++;`
- `sources/agentmemory/src/functions/capture.ts:422` `}`
- `sources/agentmemory/src/functions/capture.ts:423` `}`
- `sources/agentmemory/src/functions/capture.ts:424` `}`
- `sources/agentmemory/src/functions/capture.ts:429` `if (j >= policy.deadMax || Date.parse(rec.deadAt ?? rec.updatedAt) < cutoff) {`
- `sources/agentmemory/src/functions/capture.ts:430` `await kv.delete(KV.captureInbox, rec.key).catch(() => {});`
- `sources/agentmemory/src/functions/capture.ts:431` `removed++;`
- `sources/agentmemory/src/functions/capture.ts:432` `}`
- `sources/agentmemory/src/functions/capture.ts:433` `}`
- `sources/agentmemory/src/functions/capture.ts:434` `removed += await prunePromptLedgers(kv, cutoff, policy.eventsMax);`

**operation and route call sites**, body `sources/agentmemory/src/functions/cascade.ts:107`; body evidence.

- `sources/agentmemory/src/functions/cascade.ts:106` `offset, reason: graphReason!, updatedAt: new Date().toISOString() });`
- `sources/agentmemory/src/functions/cascade.ts:107` `} else await kv.delete(KV.graphControl, progressKey);`
- `sources/agentmemory/src/functions/cascade.ts:108` `} catch { continueBatches = false; progressSaved = false; graphDeferred = true; graphReason = "Graph cascade progress could not be saved; retry after storage recovery."; }`
- `sources/agentmemory/src/functions/cascade.ts:109` `if (continueBatches) await yieldToEventLoop();`
- `sources/agentmemory/src/functions/cascade.ts:110` `}`
- `sources/agentmemory/src/functions/cascade.ts:111` `} else {`
- `sources/agentmemory/src/functions/cascade.ts:112` `await kv.delete(KV.graphControl, progressKey).catch(() => {});`
- `sources/agentmemory/src/functions/cascade.ts:113` `}`
- `sources/agentmemory/src/functions/cascade.ts:114` ``
- `sources/agentmemory/src/functions/cascade.ts:115` `const supersededConcepts = new Set(`
- `sources/agentmemory/src/functions/cascade.ts:116` `(superseded.concepts ?? []).map((c) => c.toLowerCase()),`

**operation and route call sites**, body `sources/agentmemory/src/functions/compress.ts:255`; body evidence.

- `sources/agentmemory/src/functions/compress.ts:254` `const task: ActiveCompression = { attempt, promise: compress(data, attempt, deadline).finally(() => {`
- `sources/agentmemory/src/functions/compress.ts:255` `if (active.get(key) === task) active.delete(key);`
- `sources/agentmemory/src/functions/compress.ts:256` `}) };`
- `sources/agentmemory/src/functions/compress.ts:257` `active.set(key, task);`
- `sources/agentmemory/src/functions/compress.ts:258` `return task.promise;`
- `sources/agentmemory/src/functions/compress.ts:259` `}`

**operation and route call sites**, body `sources/agentmemory/src/functions/compression-recovery.ts:74`; body evidence.

- `sources/agentmemory/src/functions/compression-recovery.ts:73` `const request = input && typeof input === "object" && !Array.isArray(input) ? { ...input } : input;`
- `sources/agentmemory/src/functions/compression-recovery.ts:74` `if (request && typeof request === "object") delete (request as Record<string, unknown>)._caller_worker_id;`
- `sources/agentmemory/src/functions/compression-recovery.ts:75` `const scope = parseCompressionScope(request);`
- `sources/agentmemory/src/functions/compression-recovery.ts:76` `const after = readCursor(scope);`
- `sources/agentmemory/src/functions/compression-recovery.ts:77` `const sessions = scope.sessionId`
- `sources/agentmemory/src/functions/compression-recovery.ts:78` `? [await kv.get<Session>(KV.sessions, scope.sessionId)].filter((s): s is Session => Boolean(s))`

**operation and route call sites**, body `sources/agentmemory/src/functions/consolidate.ts:266`; body evidence.

- `sources/agentmemory/src/functions/consolidate.ts:265` `await kv.set(KV.memories, previous.id, previous);`
- `sources/agentmemory/src/functions/consolidate.ts:266` `await kv.delete(KV.memories, memory.id);`
- `sources/agentmemory/src/functions/consolidate.ts:267` `throw error;`
- `sources/agentmemory/src/functions/consolidate.ts:268` `}`
- `sources/agentmemory/src/functions/consolidate.ts:269` `}`
- `sources/agentmemory/src/functions/consolidate.ts:270` `await safeAudit(kv, previous ? "evolve" : "remember", "mem::consolidate", [memory.id], {`

**operation and route call sites**, body `sources/agentmemory/src/functions/context.ts:128`; body evidence.

- `sources/agentmemory/src/functions/context.ts:127` `const relevantLessons = lessons`
- `sources/agentmemory/src/functions/context.ts:128` `.filter((l) => !l.deleted && (!l.project || l.project === data.project))`
- `sources/agentmemory/src/functions/context.ts:129` `.sort((a, b) => {`
- `sources/agentmemory/src/functions/context.ts:130` `const scoreA = (a.project === data.project ? 1.5 : 1) * a.confidence;`
- `sources/agentmemory/src/functions/context.ts:131` `const scoreB = (b.project === data.project ? 1.5 : 1) * b.confidence;`
- `sources/agentmemory/src/functions/context.ts:132` `return scoreB - scoreA;`

**operation and route call sites**, body `sources/agentmemory/src/functions/dedup.ts:52`; body evidence.

- `sources/agentmemory/src/functions/dedup.ts:51` `if (Date.now() > entry.expiresAt) {`
- `sources/agentmemory/src/functions/dedup.ts:52` `this.entries.delete(hash);`
- `sources/agentmemory/src/functions/dedup.ts:53` `return false;`
- `sources/agentmemory/src/functions/dedup.ts:54` `}`
- `sources/agentmemory/src/functions/dedup.ts:55` `return true;`
- `sources/agentmemory/src/functions/dedup.ts:56` `}`
- `sources/agentmemory/src/functions/dedup.ts:64` `for (const [key, entry] of this.entries) {`
- `sources/agentmemory/src/functions/dedup.ts:65` `if (now > entry.expiresAt) this.entries.delete(key);`
- `sources/agentmemory/src/functions/dedup.ts:66` `}`
- `sources/agentmemory/src/functions/dedup.ts:67` `}`
- `sources/agentmemory/src/functions/dedup.ts:68` ``
- `sources/agentmemory/src/functions/dedup.ts:69` `stop(): void {`

**operation and route call sites**, body `sources/agentmemory/src/functions/diagnostics.ts:409`; body evidence.

- `sources/agentmemory/src/functions/diagnostics.ts:408` `const lessons = await kv.list<Lesson>(KV.lessons);`
- `sources/agentmemory/src/functions/diagnostics.ts:409` `const live = lessons.filter((l) => !l.deleted);`
- `sources/agentmemory/src/functions/diagnostics.ts:410` `let lessonIssues = 0;`
- `sources/agentmemory/src/functions/diagnostics.ts:411` `for (const l of live) {`
- `sources/agentmemory/src/functions/diagnostics.ts:412` `// Number.isFinite rejects NaN / Infinity / non-numbers; a`
- `sources/agentmemory/src/functions/diagnostics.ts:413` `// corrupted row passing those would silently survive the < / >`
- `sources/agentmemory/src/functions/diagnostics.ts:436` `status: "pass",`
- `sources/agentmemory/src/functions/diagnostics.ts:437` `message: 'All ${live.length} lessons are healthy (${lessons.length - live.length} tombstoned)',`
- `sources/agentmemory/src/functions/diagnostics.ts:438` `fixable: false,`
- `sources/agentmemory/src/functions/diagnostics.ts:439` `});`
- `sources/agentmemory/src/functions/diagnostics.ts:440` `}`
- `sources/agentmemory/src/functions/diagnostics.ts:441` `}`
- `sources/agentmemory/src/functions/diagnostics.ts:633` `status: "warn",`
- `sources/agentmemory/src/functions/diagnostics.ts:634` `message: '${orphans.length} facets point at deleted targets',`
- `sources/agentmemory/src/functions/diagnostics.ts:635` `fixable: true,`
- `sources/agentmemory/src/functions/diagnostics.ts:636` `}`
- `sources/agentmemory/src/functions/diagnostics.ts:637` `: {`
- `sources/agentmemory/src/functions/diagnostics.ts:638` `name: "facets-ok",`
- `sources/agentmemory/src/functions/diagnostics.ts:864` `details.push(`
- `sources/agentmemory/src/functions/diagnostics.ts:865` `'[dry-run] Would delete orphaned lease ${lease.id}',`
- `sources/agentmemory/src/functions/diagnostics.ts:866` `);`
- `sources/agentmemory/src/functions/diagnostics.ts:867` `fixed++;`
- `sources/agentmemory/src/functions/diagnostics.ts:868` `continue;`
- `sources/agentmemory/src/functions/diagnostics.ts:869` `}`
- `sources/agentmemory/src/functions/diagnostics.ts:870` `await kv.delete(KV.leases, lease.id);`
- `sources/agentmemory/src/functions/diagnostics.ts:871` `await recordAudit(kv, "heal", "mem::heal", [lease.id], {`
- `sources/agentmemory/src/functions/diagnostics.ts:872` `entityType: "lease",`
- `sources/agentmemory/src/functions/diagnostics.ts:873` `reason: "orphaned-lease",`
- `sources/agentmemory/src/functions/diagnostics.ts:874` `action: "delete",`
- `sources/agentmemory/src/functions/diagnostics.ts:875` `});`
- `sources/agentmemory/src/functions/diagnostics.ts:876` `details.push('Deleted orphaned lease ${lease.id}');`
- `sources/agentmemory/src/functions/diagnostics.ts:877` `fixed++;`
- `sources/agentmemory/src/functions/diagnostics.ts:878` `}`
- `sources/agentmemory/src/functions/diagnostics.ts:879` `}`
- `sources/agentmemory/src/functions/diagnostics.ts:880` `}`
- `sources/agentmemory/src/functions/diagnostics.ts:967` `) {`
- `sources/agentmemory/src/functions/diagnostics.ts:968` `await kv.delete(KV.actionEdges, edge.id);`
- `sources/agentmemory/src/functions/diagnostics.ts:969` `await recordAudit(kv, "heal", "mem::heal", [edge.id], {`
- `sources/agentmemory/src/functions/diagnostics.ts:970` `entityType: "actionEdge",`
- `sources/agentmemory/src/functions/diagnostics.ts:971` `reason: "sketch-gc-discard",`
- `sources/agentmemory/src/functions/diagnostics.ts:972` `action: "delete",`
- `sources/agentmemory/src/functions/diagnostics.ts:973` `});`
- `sources/agentmemory/src/functions/diagnostics.ts:974` `}`
- `sources/agentmemory/src/functions/diagnostics.ts:975` `}`
- `sources/agentmemory/src/functions/diagnostics.ts:976` `for (const actionId of fresh.actionIds) {`
- `sources/agentmemory/src/functions/diagnostics.ts:977` `await kv.delete(KV.actions, actionId);`
- `sources/agentmemory/src/functions/diagnostics.ts:978` `await recordAudit(kv, "heal", "mem::heal", [actionId], {`
- `sources/agentmemory/src/functions/diagnostics.ts:979` `entityType: "action",`
- `sources/agentmemory/src/functions/diagnostics.ts:980` `reason: "sketch-gc-discard",`
- `sources/agentmemory/src/functions/diagnostics.ts:981` `action: "delete",`
- `sources/agentmemory/src/functions/diagnostics.ts:982` `});`
- `sources/agentmemory/src/functions/diagnostics.ts:983` `}`
- `sources/agentmemory/src/functions/diagnostics.ts:984` ``
- `sources/agentmemory/src/functions/diagnostics.ts:985` `fresh.status = "discarded";`
- `sources/agentmemory/src/functions/diagnostics.ts:1017` `details.push(`
- `sources/agentmemory/src/functions/diagnostics.ts:1018` `'[dry-run] Would delete expired signal ${signal.id}',`
- `sources/agentmemory/src/functions/diagnostics.ts:1019` `);`
- `sources/agentmemory/src/functions/diagnostics.ts:1020` `fixed++;`
- `sources/agentmemory/src/functions/diagnostics.ts:1021` `continue;`
- `sources/agentmemory/src/functions/diagnostics.ts:1022` `}`
- `sources/agentmemory/src/functions/diagnostics.ts:1023` `await kv.delete(KV.signals, signal.id);`
- `sources/agentmemory/src/functions/diagnostics.ts:1024` `await recordAudit(kv, "heal", "mem::heal", [signal.id], {`
- `sources/agentmemory/src/functions/diagnostics.ts:1025` `entityType: "signal",`
- `sources/agentmemory/src/functions/diagnostics.ts:1026` `reason: "expired-signal",`
- `sources/agentmemory/src/functions/diagnostics.ts:1027` `action: "delete",`
- `sources/agentmemory/src/functions/diagnostics.ts:1028` `});`
- `sources/agentmemory/src/functions/diagnostics.ts:1029` `details.push('Deleted expired signal ${signal.id}');`
- `sources/agentmemory/src/functions/diagnostics.ts:1030` `fixed++;`
- `sources/agentmemory/src/functions/diagnostics.ts:1031` `}`
- `sources/agentmemory/src/functions/diagnostics.ts:1032` `}`
- `sources/agentmemory/src/functions/diagnostics.ts:1033` `}`
- `sources/agentmemory/src/functions/diagnostics.ts:1085` `for (const facet of await findOrphanFacets(kv)) {`
- `sources/agentmemory/src/functions/diagnostics.ts:1086` `const label = 'facet ${facet.dimension}:${facet.value} on deleted ${facet.targetType} ${facet.targetId}';`
- `sources/agentmemory/src/functions/diagnostics.ts:1087` `if (dryRun) {`
- `sources/agentmemory/src/functions/diagnostics.ts:1088` `details.push('[dry-run] Would remove ${label}');`
- `sources/agentmemory/src/functions/diagnostics.ts:1089` `fixed++;`
- `sources/agentmemory/src/functions/diagnostics.ts:1090` `continue;`
- `sources/agentmemory/src/functions/diagnostics.ts:1095` `}`
- `sources/agentmemory/src/functions/diagnostics.ts:1096` `await kv.delete(KV.facets, facet.id);`
- `sources/agentmemory/src/functions/diagnostics.ts:1097` `await recordAudit(kv, "heal", "mem::heal", [facet.id], {`
- `sources/agentmemory/src/functions/diagnostics.ts:1098` `entityType: "facet",`
- `sources/agentmemory/src/functions/diagnostics.ts:1099` `reason: "orphan-facet",`
- `sources/agentmemory/src/functions/diagnostics.ts:1100` `targetId: facet.targetId,`
- `sources/agentmemory/src/functions/diagnostics.ts:1101` `action: "delete",`
- `sources/agentmemory/src/functions/diagnostics.ts:1102` `});`
- `sources/agentmemory/src/functions/diagnostics.ts:1103` `details.push('Removed ${label}');`
- `sources/agentmemory/src/functions/diagnostics.ts:1104` `fixed++;`
- `sources/agentmemory/src/functions/diagnostics.ts:1105` `}`

**operation and route call sites**, body `sources/agentmemory/src/functions/evict.ts:2`; body evidence.

- `sources/agentmemory/src/functions/evict.ts:1` `import type { IIIClient } from "iii-sdk";`
- `sources/agentmemory/src/functions/evict.ts:2` `import { markCaptureEventDeleted } from "../capture/event-record.js";`
- `sources/agentmemory/src/functions/evict.ts:3` `import type {`
- `sources/agentmemory/src/functions/evict.ts:4` `Session,`
- `sources/agentmemory/src/functions/evict.ts:5` `CompressedObservation,`
- `sources/agentmemory/src/functions/evict.ts:6` `RawObservation,`
- `sources/agentmemory/src/functions/evict.ts:16` `import { recordAudit } from "./audit.js";`
- `sources/agentmemory/src/functions/evict.ts:17` `import { deleteAccessLog } from "./access-tracker.js";`
- `sources/agentmemory/src/functions/evict.ts:18` `import { deleteFacetsForTargets } from "./facets.js";`
- `sources/agentmemory/src/functions/evict.ts:19` `import { getSearchIndex, vectorIndexRemove, flushIndexSave } from "./search.js";`
- `sources/agentmemory/src/functions/evict.ts:20` `import { logger } from "../logger.js";`
- `sources/agentmemory/src/functions/evict.ts:21` ``
- `sources/agentmemory/src/functions/evict.ts:22` `interface EvictionConfig {`
- `sources/agentmemory/src/functions/evict.ts:126` `if (!current || !eligible(current)) return null;`
- `sources/agentmemory/src/functions/evict.ts:127` `await markCaptureEventDeleted(kv, current);`
- `sources/agentmemory/src/functions/evict.ts:128` `await kv.delete(KV.observations(sessionId), selected.id);`
- `sources/agentmemory/src/functions/evict.ts:129` `getSearchIndex().remove(selected.id);`
- `sources/agentmemory/src/functions/evict.ts:130` `vectorIndexRemove(selected.id);`
- `sources/agentmemory/src/functions/evict.ts:131` `indexCleaned = true;`
- `sources/agentmemory/src/functions/evict.ts:132` `return current;`
- `sources/agentmemory/src/functions/evict.ts:200` `if (JSON.stringify(currentObservations) !== observationsRevision) return false;`
- `sources/agentmemory/src/functions/evict.ts:201` `await kv.delete(KV.sessions, session.id);`
- `sources/agentmemory/src/functions/evict.ts:202` `await kv.delete(KV.summaryRepairs, session.id);`
- `sources/agentmemory/src/functions/evict.ts:203` `return true;`
- `sources/agentmemory/src/functions/evict.ts:204` `});`
- `sources/agentmemory/src/functions/evict.ts:205` `if (!removed) continue;`
- `sources/agentmemory/src/functions/evict.ts:206` `stats.staleSessions++;`
- `sources/agentmemory/src/functions/evict.ts:207` `} catch (err) {`
- `sources/agentmemory/src/functions/evict.ts:208` `logger.warn("Eviction delete failed", {`
- `sources/agentmemory/src/functions/evict.ts:209` `resource: "session",`
- `sources/agentmemory/src/functions/evict.ts:210` `id: session.id,`
- `sources/agentmemory/src/functions/evict.ts:211` `error: err instanceof Error ? err.message : String(err),`
- `sources/agentmemory/src/functions/evict.ts:212` `});`
- `sources/agentmemory/src/functions/evict.ts:222` `).catch(() => {});`
- `sources/agentmemory/src/functions/evict.ts:223` `await recordAudit(kv, "delete", "mem::evict", [session.id], {`
- `sources/agentmemory/src/functions/evict.ts:224` `resource: "session",`
- `sources/agentmemory/src/functions/evict.ts:225` `reason: recovered`
- `sources/agentmemory/src/functions/evict.ts:226` `? "stale_session_recovered_then_evicted"`
- `sources/agentmemory/src/functions/evict.ts:227` `: "stale_session_without_summary",`
- `sources/agentmemory/src/functions/evict.ts:264` `} catch (err) {`
- `sources/agentmemory/src/functions/evict.ts:265` `logger.warn("Eviction delete failed", {`
- `sources/agentmemory/src/functions/evict.ts:266` `resource: "observation",`
- `sources/agentmemory/src/functions/evict.ts:267` `id: o.id,`
- `sources/agentmemory/src/functions/evict.ts:268` `sessionId: session.id,`
- `sources/agentmemory/src/functions/evict.ts:269` `error: err instanceof Error ? err.message : String(err),`
- `sources/agentmemory/src/functions/evict.ts:275` `if (removed.imageRef && removed.imageRef !== removed.imageData) await decrementImageRef(kv, sdk, removed.imageRef);`
- `sources/agentmemory/src/functions/evict.ts:276` `await recordAudit(kv, "delete", "mem::evict", [o.id], {`
- `sources/agentmemory/src/functions/evict.ts:277` `resource: "observation",`
- `sources/agentmemory/src/functions/evict.ts:278` `reason: "low_importance_old_observation",`
- `sources/agentmemory/src/functions/evict.ts:279` `sessionId: session.id,`
- `sources/agentmemory/src/functions/evict.ts:280` `dryRun,`
- `sources/agentmemory/src/functions/evict.ts:312` `} catch (err) {`
- `sources/agentmemory/src/functions/evict.ts:313` `logger.warn("Eviction delete failed", {`
- `sources/agentmemory/src/functions/evict.ts:314` `resource: "observation",`
- `sources/agentmemory/src/functions/evict.ts:315` `id: o.id,`
- `sources/agentmemory/src/functions/evict.ts:316` `sessionId: o.sessionId,`
- `sources/agentmemory/src/functions/evict.ts:317` `error: err instanceof Error ? err.message : String(err),`
- `sources/agentmemory/src/functions/evict.ts:323` `if (removed.imageRef && removed.imageRef !== removed.imageData) await decrementImageRef(kv, sdk, removed.imageRef);`
- `sources/agentmemory/src/functions/evict.ts:324` `await recordAudit(kv, "delete", "mem::evict", [o.id], {`
- `sources/agentmemory/src/functions/evict.ts:325` `resource: "observation",`
- `sources/agentmemory/src/functions/evict.ts:326` `reason: "project_observation_cap",`
- `sources/agentmemory/src/functions/evict.ts:327` `sessionId: o.sessionId,`
- `sources/agentmemory/src/functions/evict.ts:328` `dryRun,`
- `sources/agentmemory/src/functions/evict.ts:336` `const evictedMemIds = new Set<string>();`
- `sources/agentmemory/src/functions/evict.ts:337` `const deletedMemIds = new Set<string>();`
- `sources/agentmemory/src/functions/evict.ts:338` `try {`
- `sources/agentmemory/src/functions/evict.ts:339` `for (const mem of memories) {`
- `sources/agentmemory/src/functions/evict.ts:340` `if (mem.forgetAfter) {`
- `sources/agentmemory/src/functions/evict.ts:341` `const expiry = new Date(mem.forgetAfter).getTime();`
- `sources/agentmemory/src/functions/evict.ts:342` `if (now > expiry) {`
- `sources/agentmemory/src/functions/evict.ts:343` `if (dryRun) {`
- `sources/agentmemory/src/functions/evict.ts:344` `stats.expiredMemories++;`
- `sources/agentmemory/src/functions/evict.ts:345` `evictedMemIds.add(mem.id);`
- `sources/agentmemory/src/functions/evict.ts:347` `try {`
- `sources/agentmemory/src/functions/evict.ts:348` `await kv.delete(KV.memories, mem.id);`
- `sources/agentmemory/src/functions/evict.ts:349` `deletedMemIds.add(mem.id);`
- `sources/agentmemory/src/functions/evict.ts:350` `stats.expiredMemories++;`
- `sources/agentmemory/src/functions/evict.ts:351` `evictedMemIds.add(mem.id);`
- `sources/agentmemory/src/functions/evict.ts:352` `} catch (err) {`
- `sources/agentmemory/src/functions/evict.ts:353` `logger.warn("Eviction delete failed", {`
- `sources/agentmemory/src/functions/evict.ts:354` `resource: "memory",`
- `sources/agentmemory/src/functions/evict.ts:355` `id: mem.id,`
- `sources/agentmemory/src/functions/evict.ts:356` `reason: "expired_memory",`
- `sources/agentmemory/src/functions/evict.ts:357` `error: err instanceof Error ? err.message : String(err),`
- `sources/agentmemory/src/functions/evict.ts:366` `}`
- `sources/agentmemory/src/functions/evict.ts:367` `await recordAudit(kv, "delete", "mem::evict", [mem.id], {`
- `sources/agentmemory/src/functions/evict.ts:368` `resource: "memory",`
- `sources/agentmemory/src/functions/evict.ts:369` `reason: "expired_memory",`
- `sources/agentmemory/src/functions/evict.ts:370` `dryRun,`
- `sources/agentmemory/src/functions/evict.ts:371` `});`
- `sources/agentmemory/src/functions/evict.ts:372` `await deleteAccessLog(kv, mem.id);`
- `sources/agentmemory/src/functions/evict.ts:373` `}`
- `sources/agentmemory/src/functions/evict.ts:374` `}`
- `sources/agentmemory/src/functions/evict.ts:375` `}`
- `sources/agentmemory/src/functions/evict.ts:376` ``
- `sources/agentmemory/src/functions/evict.ts:387` `try {`
- `sources/agentmemory/src/functions/evict.ts:388` `await kv.delete(KV.memories, mem.id);`
- `sources/agentmemory/src/functions/evict.ts:389` `deletedMemIds.add(mem.id);`
- `sources/agentmemory/src/functions/evict.ts:390` `stats.nonLatestMemories++;`
- `sources/agentmemory/src/functions/evict.ts:391` `} catch (err) {`
- `sources/agentmemory/src/functions/evict.ts:392` `logger.warn("Eviction delete failed", {`
- `sources/agentmemory/src/functions/evict.ts:393` `resource: "memory",`
- `sources/agentmemory/src/functions/evict.ts:394` `id: mem.id,`
- `sources/agentmemory/src/functions/evict.ts:395` `reason: "old_non_latest_memory",`
- `sources/agentmemory/src/functions/evict.ts:396` `error: err instanceof Error ? err.message : String(err),`
- `sources/agentmemory/src/functions/evict.ts:405` `}`
- `sources/agentmemory/src/functions/evict.ts:406` `await recordAudit(kv, "delete", "mem::evict", [mem.id], {`
- `sources/agentmemory/src/functions/evict.ts:407` `resource: "memory",`
- `sources/agentmemory/src/functions/evict.ts:408` `reason: "old_non_latest_memory",`
- `sources/agentmemory/src/functions/evict.ts:409` `dryRun,`
- `sources/agentmemory/src/functions/evict.ts:410` `});`
- `sources/agentmemory/src/functions/evict.ts:411` `await deleteAccessLog(kv, mem.id);`
- `sources/agentmemory/src/functions/evict.ts:412` `}`
- `sources/agentmemory/src/functions/evict.ts:413` `}`
- `sources/agentmemory/src/functions/evict.ts:414` `}`
- `sources/agentmemory/src/functions/evict.ts:415` `}`
- `sources/agentmemory/src/functions/evict.ts:416` `} finally {`
- `sources/agentmemory/src/functions/evict.ts:417` `await deleteFacetsForTargets(kv, deletedMemIds);`
- `sources/agentmemory/src/functions/evict.ts:418` `if (indexCleaned) await flushIndexSave();`
- `sources/agentmemory/src/functions/evict.ts:419` `}`
- `sources/agentmemory/src/functions/evict.ts:420` ``
- `sources/agentmemory/src/functions/evict.ts:421` `logger.info("Eviction complete", { stats });`

**operation and route call sites**, body `sources/agentmemory/src/functions/export-import.ts:5`; body evidence.

- `sources/agentmemory/src/functions/export-import.ts:4` `import type { IIIClient } from "iii-sdk";`
- `sources/agentmemory/src/functions/export-import.ts:5` `import { markCaptureEventDeleted } from "../capture/event-record.js";`
- `sources/agentmemory/src/functions/export-import.ts:6` `import type {`
- `sources/agentmemory/src/functions/export-import.ts:7` `Session,`
- `sources/agentmemory/src/functions/export-import.ts:8` `CompressedObservation,`
- `sources/agentmemory/src/functions/export-import.ts:9` `Memory,`
- `sources/agentmemory/src/functions/export-import.ts:493` `// multiplies in-flight deletes to chunk-size squared.`
- `sources/agentmemory/src/functions/export-import.ts:494` `const obsDeletes: Array<{ sessionId: string; obsId: string; captureKey?: string }> = [];`
- `sources/agentmemory/src/functions/export-import.ts:495` `await runChunked(obsBySession, async ({ session, obs }) => {`
- `sources/agentmemory/src/functions/export-import.ts:496` `await kv.delete(KV.sessions, session.id);`
- `sources/agentmemory/src/functions/export-import.ts:497` `await kv.delete(KV.summaryRepairs, session.id);`
- `sources/agentmemory/src/functions/export-import.ts:498` `await removeSessionFromProjectIndex(`
- `sources/agentmemory/src/functions/export-import.ts:499` `kv,`
- `sources/agentmemory/src/functions/export-import.ts:500` `session.project,`
- `sources/agentmemory/src/functions/export-import.ts:501` `session.id,`
- `sources/agentmemory/src/functions/export-import.ts:503` `for (const o of obs) {`
- `sources/agentmemory/src/functions/export-import.ts:504` `obsDeletes.push({ sessionId: session.id, obsId: o.id, captureKey: o.captureKey });`
- `sources/agentmemory/src/functions/export-import.ts:505` `}`
- `sources/agentmemory/src/functions/export-import.ts:506` `});`
- `sources/agentmemory/src/functions/export-import.ts:507` `await runChunked(obsDeletes, async (d) => {`
- `sources/agentmemory/src/functions/export-import.ts:508` `await markCaptureEventDeleted(kv, { id: d.obsId, sessionId: d.sessionId, captureKey: d.captureKey });`
- `sources/agentmemory/src/functions/export-import.ts:509` `await kv.delete(KV.observations(d.sessionId), d.obsId);`
- `sources/agentmemory/src/functions/export-import.ts:510` `await unindexObservationSession(kv, d.obsId).catch(() => {});`
- `sources/agentmemory/src/functions/export-import.ts:511` `});`
- `sources/agentmemory/src/functions/export-import.ts:512` `await runChunked(memories, (m) =>`
- `sources/agentmemory/src/functions/export-import.ts:513` `kv.delete(KV.memories, m.id),`
- `sources/agentmemory/src/functions/export-import.ts:514` `);`
- `sources/agentmemory/src/functions/export-import.ts:515` `await runChunked(summaries, (s) => kv.delete(KV.summaries, s.sessionId));`
- `sources/agentmemory/src/functions/export-import.ts:516` `await runChunked(actions, (a) =>`
- `sources/agentmemory/src/functions/export-import.ts:517` `kv.delete(KV.actions, a.id),`
- `sources/agentmemory/src/functions/export-import.ts:518` `);`
- `sources/agentmemory/src/functions/export-import.ts:519` `await runChunked(actionEdges, (e) =>`
- `sources/agentmemory/src/functions/export-import.ts:520` `kv.delete(KV.actionEdges, e.id),`
- `sources/agentmemory/src/functions/export-import.ts:521` `);`
- `sources/agentmemory/src/functions/export-import.ts:522` `await runChunked(routines, (r) =>`
- `sources/agentmemory/src/functions/export-import.ts:523` `kv.delete(KV.routines, r.id),`
- `sources/agentmemory/src/functions/export-import.ts:524` `);`
- `sources/agentmemory/src/functions/export-import.ts:525` `await runChunked(signals, (s) =>`
- `sources/agentmemory/src/functions/export-import.ts:526` `kv.delete(KV.signals, s.id),`
- `sources/agentmemory/src/functions/export-import.ts:527` `);`
- `sources/agentmemory/src/functions/export-import.ts:528` `await runChunked(checkpoints, (c) =>`
- `sources/agentmemory/src/functions/export-import.ts:529` `kv.delete(KV.checkpoints, c.id),`
- `sources/agentmemory/src/functions/export-import.ts:530` `);`
- `sources/agentmemory/src/functions/export-import.ts:531` `await runChunked(sentinels, (s) =>`
- `sources/agentmemory/src/functions/export-import.ts:532` `kv.delete(KV.sentinels, s.id),`
- `sources/agentmemory/src/functions/export-import.ts:533` `);`
- `sources/agentmemory/src/functions/export-import.ts:534` `await runChunked(sketches, (s) =>`
- `sources/agentmemory/src/functions/export-import.ts:535` `kv.delete(KV.sketches, s.id),`
- `sources/agentmemory/src/functions/export-import.ts:536` `);`
- `sources/agentmemory/src/functions/export-import.ts:537` `await runChunked(crystals, (c) =>`
- `sources/agentmemory/src/functions/export-import.ts:538` `kv.delete(KV.crystals, c.id),`
- `sources/agentmemory/src/functions/export-import.ts:539` `);`
- `sources/agentmemory/src/functions/export-import.ts:540` `await runChunked(facets, (f) =>`
- `sources/agentmemory/src/functions/export-import.ts:541` `kv.delete(KV.facets, f.id),`
- `sources/agentmemory/src/functions/export-import.ts:542` `);`
- `sources/agentmemory/src/functions/export-import.ts:543` `await runChunked(lessons, (l) =>`
- `sources/agentmemory/src/functions/export-import.ts:544` `kv.delete(KV.lessons, l.id),`
- `sources/agentmemory/src/functions/export-import.ts:545` `);`
- `sources/agentmemory/src/functions/export-import.ts:546` `resetLessonIndex();`
- `sources/agentmemory/src/functions/export-import.ts:547` `await runChunked(insights, (i) =>`
- `sources/agentmemory/src/functions/export-import.ts:548` `kv.delete(KV.insights, i.id),`
- `sources/agentmemory/src/functions/export-import.ts:549` `);`
- `sources/agentmemory/src/functions/export-import.ts:550` `await runChunked(semanticRows, (s) =>`
- `sources/agentmemory/src/functions/export-import.ts:551` `kv.delete(KV.semantic, s.id),`
- `sources/agentmemory/src/functions/export-import.ts:552` `);`
- `sources/agentmemory/src/functions/export-import.ts:553` `await runChunked(proceduralRows, (p) =>`
- `sources/agentmemory/src/functions/export-import.ts:554` `kv.delete(KV.procedural, p.id),`
- `sources/agentmemory/src/functions/export-import.ts:555` `);`
- `sources/agentmemory/src/functions/export-import.ts:556` `await runChunked(profiles, (profile) =>`
- `sources/agentmemory/src/functions/export-import.ts:557` `kv.delete(KV.profiles, profile.project),`
- `sources/agentmemory/src/functions/export-import.ts:558` `);`
- `sources/agentmemory/src/functions/export-import.ts:559` `await runChunked(accessLogs, (a) =>`
- `sources/agentmemory/src/functions/export-import.ts:560` `kv.delete(KV.accessLog, a.memoryId),`
- `sources/agentmemory/src/functions/export-import.ts:561` `);`
- `sources/agentmemory/src/functions/export-import.ts:562` `}`
- `sources/agentmemory/src/functions/export-import.ts:563` ``
- `sources/agentmemory/src/functions/export-import.ts:564` `await runChunked(importData.sessions, async (session) => {`

**operation and route call sites**, body `sources/agentmemory/src/functions/facets.ts:25`; body evidence.

- `sources/agentmemory/src/functions/facets.ts:24` `const lesson = await kv.get<Lesson>(KV.lessons, id);`
- `sources/agentmemory/src/functions/facets.ts:25` `if (lesson && !lesson.deleted) return { observation, record: true };`
- `sources/agentmemory/src/functions/facets.ts:26` `}`
- `sources/agentmemory/src/functions/facets.ts:27` `const [mem, sem] = await Promise.all([`
- `sources/agentmemory/src/functions/facets.ts:28` `kv.get(KV.memories, id),`
- `sources/agentmemory/src/functions/facets.ts:29` `kv.get(KV.semantic, id),`
- `sources/agentmemory/src/functions/facets.ts:39` ``
- `sources/agentmemory/src/functions/facets.ts:40` `export async function deleteFacetsForTargets(`
- `sources/agentmemory/src/functions/facets.ts:41` `kv: StateKV,`
- `sources/agentmemory/src/functions/facets.ts:42` `targetIds: Iterable<string>,`
- `sources/agentmemory/src/functions/facets.ts:43` `options?: { targetTypes?: readonly string[] },`
- `sources/agentmemory/src/functions/facets.ts:44` `): Promise<void> {`
- `sources/agentmemory/src/functions/facets.ts:45` `const ids = new Set(targetIds);`
- `sources/agentmemory/src/functions/facets.ts:46` `ids.delete("");`
- `sources/agentmemory/src/functions/facets.ts:47` `if (ids.size === 0) return;`
- `sources/agentmemory/src/functions/facets.ts:48` `const targetTypes = new Set(options?.targetTypes ?? ["memory"]);`
- `sources/agentmemory/src/functions/facets.ts:49` `try {`
- `sources/agentmemory/src/functions/facets.ts:50` `const live = new Map<string, TargetLiveness>();`
- `sources/agentmemory/src/functions/facets.ts:74` `);`
- `sources/agentmemory/src/functions/facets.ts:75` `await Promise.all(matches.map((f) => kv.delete(KV.facets, f.id)));`
- `sources/agentmemory/src/functions/facets.ts:76` `} catch (err) {`
- `sources/agentmemory/src/functions/facets.ts:77` `logger.warn("Facet cleanup failed", {`
- `sources/agentmemory/src/functions/facets.ts:78` `targetIds: ids.size,`
- `sources/agentmemory/src/functions/facets.ts:79` `error: err instanceof Error ? err.message : String(err),`
- `sources/agentmemory/src/functions/facets.ts:236` `for (const f of matches) {`
- `sources/agentmemory/src/functions/facets.ts:237` `await kv.delete(KV.facets, f.id);`
- `sources/agentmemory/src/functions/facets.ts:238` `}`
- `sources/agentmemory/src/functions/facets.ts:239` ``
- `sources/agentmemory/src/functions/facets.ts:240` `return { success: true, removed: matches.length };`
- `sources/agentmemory/src/functions/facets.ts:241` `},`

**operation and route call sites**, body `sources/agentmemory/src/functions/governance.ts:7`; body evidence.

- `sources/agentmemory/src/functions/governance.ts:6` `import { recordAudit, safeAudit, queryAudit } from "./audit.js";`
- `sources/agentmemory/src/functions/governance.ts:7` `import { deleteAccessLog } from "./access-tracker.js";`
- `sources/agentmemory/src/functions/governance.ts:8` `import { deleteFacetsForTargets } from "./facets.js";`
- `sources/agentmemory/src/functions/governance.ts:9` `import { getSearchIndex, vectorIndexRemove, flushIndexSave } from "./search.js";`
- `sources/agentmemory/src/functions/governance.ts:10` `import { logger } from "../logger.js";`
- `sources/agentmemory/src/functions/governance.ts:11` `const BULK_FILTER_KEYS = ["type", "dateFrom", "dateTo", "project", "qualityBelow"];`
- `sources/agentmemory/src/functions/governance.ts:12` `const BULK_CONTROL_KEYS = ["dryRun", "reason"];`
- `sources/agentmemory/src/functions/governance.ts:13` ``
- `sources/agentmemory/src/functions/governance.ts:14` `type DeleteFailure = { id: string; error: string };`
- `sources/agentmemory/src/functions/governance.ts:15` ``
- `sources/agentmemory/src/functions/governance.ts:16` `function errorMessage(err: unknown): string {`
- `sources/agentmemory/src/functions/governance.ts:17` `return err instanceof Error ? err.message : String(err);`
- `sources/agentmemory/src/functions/governance.ts:18` `}`
- `sources/agentmemory/src/functions/governance.ts:19` ``
- `sources/agentmemory/src/functions/governance.ts:20` `async function deleteMemoryRecord(`
- `sources/agentmemory/src/functions/governance.ts:21` `kv: StateKV,`
- `sources/agentmemory/src/functions/governance.ts:22` `id: string,`
- `sources/agentmemory/src/functions/governance.ts:23` `): Promise<"deleted" | "missing"> {`
- `sources/agentmemory/src/functions/governance.ts:24` `const mem = await kv.get<Memory>(KV.memories, id);`
- `sources/agentmemory/src/functions/governance.ts:25` `if (!mem) return "missing";`
- `sources/agentmemory/src/functions/governance.ts:26` `await kv.delete(KV.memories, id);`
- `sources/agentmemory/src/functions/governance.ts:27` `await deleteAccessLog(kv, id);`
- `sources/agentmemory/src/functions/governance.ts:28` `getSearchIndex().remove(id);`
- `sources/agentmemory/src/functions/governance.ts:29` `vectorIndexRemove(id);`
- `sources/agentmemory/src/functions/governance.ts:30` `return "deleted";`
- `sources/agentmemory/src/functions/governance.ts:31` `}`
- `sources/agentmemory/src/functions/governance.ts:32` ``
- `sources/agentmemory/src/functions/governance.ts:33` `export function registerGovernanceFunction(sdk: IIIClient, kv: StateKV): void {`
- `sources/agentmemory/src/functions/governance.ts:34` `sdk.registerFunction("mem::governance-delete",`
- `sources/agentmemory/src/functions/governance.ts:35` `async (data: { memoryIds: string[]; reason?: string }) => {`
- `sources/agentmemory/src/functions/governance.ts:36` `if (`
- `sources/agentmemory/src/functions/governance.ts:37` `!data.memoryIds ||`
- `sources/agentmemory/src/functions/governance.ts:38` `!Array.isArray(data.memoryIds) ||`
- `sources/agentmemory/src/functions/governance.ts:43` ``
- `sources/agentmemory/src/functions/governance.ts:44` `const deletedIds: string[] = [];`
- `sources/agentmemory/src/functions/governance.ts:45` `const notFound: string[] = [];`
- `sources/agentmemory/src/functions/governance.ts:46` `const failures: DeleteFailure[] = [];`
- `sources/agentmemory/src/functions/governance.ts:47` `for (const id of data.memoryIds) {`
- `sources/agentmemory/src/functions/governance.ts:48` `try {`
- `sources/agentmemory/src/functions/governance.ts:49` `if ((await deleteMemoryRecord(kv, id)) === "deleted") {`
- `sources/agentmemory/src/functions/governance.ts:50` `deletedIds.push(id);`
- `sources/agentmemory/src/functions/governance.ts:51` `} else {`
- `sources/agentmemory/src/functions/governance.ts:52` `notFound.push(id);`
- `sources/agentmemory/src/functions/governance.ts:53` `}`
- `sources/agentmemory/src/functions/governance.ts:54` `} catch (err) {`
- `sources/agentmemory/src/functions/governance.ts:55` `logger.warn("Governance delete failed", {`
- `sources/agentmemory/src/functions/governance.ts:56` `memoryId: id,`
- `sources/agentmemory/src/functions/governance.ts:57` `error: errorMessage(err),`
- `sources/agentmemory/src/functions/governance.ts:58` `});`
- `sources/agentmemory/src/functions/governance.ts:59` `failures.push({ id, error: "delete_failed" });`
- `sources/agentmemory/src/functions/governance.ts:60` `}`
- `sources/agentmemory/src/functions/governance.ts:61` `}`
- `sources/agentmemory/src/functions/governance.ts:62` ``
- `sources/agentmemory/src/functions/governance.ts:63` `// Sweeping notFound ids too lets a retried delete clean up after a`
- `sources/agentmemory/src/functions/governance.ts:65` `// still resolve in either memory scope.`
- `sources/agentmemory/src/functions/governance.ts:66` `await deleteFacetsForTargets(kv, [...deletedIds, ...notFound]);`
- `sources/agentmemory/src/functions/governance.ts:67` ``
- `sources/agentmemory/src/functions/governance.ts:68` `const deleted = deletedIds.length;`
- `sources/agentmemory/src/functions/governance.ts:69` `if (deleted > 0) await flushIndexSave();`
- `sources/agentmemory/src/functions/governance.ts:70` ``
- `sources/agentmemory/src/functions/governance.ts:71` `if (deleted > 0 || failures.length > 0) {`
- `sources/agentmemory/src/functions/governance.ts:72` `await recordAudit(`
- `sources/agentmemory/src/functions/governance.ts:73` `kv,`
- `sources/agentmemory/src/functions/governance.ts:74` `"delete",`
- `sources/agentmemory/src/functions/governance.ts:75` `"mem::governance-delete",`
- `sources/agentmemory/src/functions/governance.ts:76` `deletedIds,`
- `sources/agentmemory/src/functions/governance.ts:77` `{`
- `sources/agentmemory/src/functions/governance.ts:78` `reason: data.reason || "manual deletion",`
- `sources/agentmemory/src/functions/governance.ts:79` `deleted,`
- `sources/agentmemory/src/functions/governance.ts:80` `requested: data.memoryIds.length,`
- `sources/agentmemory/src/functions/governance.ts:81` `notFound: notFound.length,`
- `sources/agentmemory/src/functions/governance.ts:82` `failed: failures.length,`
- `sources/agentmemory/src/functions/governance.ts:83` `failures: failures.length > 0 ? failures : undefined,`
- `sources/agentmemory/src/functions/governance.ts:87` ``
- `sources/agentmemory/src/functions/governance.ts:88` `logger.info("Governance delete", {`
- `sources/agentmemory/src/functions/governance.ts:89` `requested: data.memoryIds.length,`
- `sources/agentmemory/src/functions/governance.ts:90` `deleted,`
- `sources/agentmemory/src/functions/governance.ts:91` `notFound: notFound.length,`
- `sources/agentmemory/src/functions/governance.ts:92` `failed: failures.length,`
- `sources/agentmemory/src/functions/governance.ts:93` `});`
- `sources/agentmemory/src/functions/governance.ts:94` `return {`
- `sources/agentmemory/src/functions/governance.ts:95` `success: failures.length === 0,`
- `sources/agentmemory/src/functions/governance.ts:96` `deleted,`
- `sources/agentmemory/src/functions/governance.ts:97` `total: data.memoryIds.length,`
- `sources/agentmemory/src/functions/governance.ts:98` `notFound,`
- `sources/agentmemory/src/functions/governance.ts:99` `failed: failures.length,`
- `sources/agentmemory/src/functions/governance.ts:100` `failures: failures.length > 0 ? failures : undefined,`
- `sources/agentmemory/src/functions/governance.ts:113` `success: false,`
- `sources/agentmemory/src/functions/governance.ts:114` `error: 'Unsupported bulk delete filter: ${unsupported.join(", ")}. Supported filters: ${BULK_FILTER_KEYS.join(", ")}',`
- `sources/agentmemory/src/functions/governance.ts:115` `};`
- `sources/agentmemory/src/functions/governance.ts:116` `}`
- `sources/agentmemory/src/functions/governance.ts:117` `if (`
- `sources/agentmemory/src/functions/governance.ts:118` `data.project !== undefined &&`
- `sources/agentmemory/src/functions/governance.ts:132` `success: false,`
- `sources/agentmemory/src/functions/governance.ts:133` `error: 'At least one filter is required for bulk delete (${BULK_FILTER_KEYS.join(", ")})',`
- `sources/agentmemory/src/functions/governance.ts:134` `};`
- `sources/agentmemory/src/functions/governance.ts:135` `}`
- `sources/agentmemory/src/functions/governance.ts:136` ``
- `sources/agentmemory/src/functions/governance.ts:137` `const memories = await collectAll<Memory>(kv, KV.memories);`
- `sources/agentmemory/src/functions/governance.ts:171` `dryRun: true,`
- `sources/agentmemory/src/functions/governance.ts:172` `wouldDelete: candidates.length,`
- `sources/agentmemory/src/functions/governance.ts:173` `ids: candidates.map((m) => m.id),`
- `sources/agentmemory/src/functions/governance.ts:174` `};`
- `sources/agentmemory/src/functions/governance.ts:175` `}`
- `sources/agentmemory/src/functions/governance.ts:176` ``
- `sources/agentmemory/src/functions/governance.ts:179` `const notFound: string[] = [];`
- `sources/agentmemory/src/functions/governance.ts:180` `const failures: DeleteFailure[] = [];`
- `sources/agentmemory/src/functions/governance.ts:181` `for (let i = 0; i < candidates.length; i += BATCH_SIZE) {`
- `sources/agentmemory/src/functions/governance.ts:182` `const batch = candidates.slice(i, i + BATCH_SIZE);`
- `sources/agentmemory/src/functions/governance.ts:183` `const results = await Promise.allSettled(`
- `sources/agentmemory/src/functions/governance.ts:184` `batch.map((mem) => deleteMemoryRecord(kv, mem.id)),`
- `sources/agentmemory/src/functions/governance.ts:185` `);`
- `sources/agentmemory/src/functions/governance.ts:186` `results.forEach((result, j) => {`
- `sources/agentmemory/src/functions/governance.ts:187` `const mem = batch[j];`
- `sources/agentmemory/src/functions/governance.ts:188` `if (result.status === "rejected") {`
- `sources/agentmemory/src/functions/governance.ts:189` `logger.warn("Governance bulk delete failed", {`
- `sources/agentmemory/src/functions/governance.ts:190` `memoryId: mem.id,`
- `sources/agentmemory/src/functions/governance.ts:191` `error: errorMessage(result.reason),`
- `sources/agentmemory/src/functions/governance.ts:192` `});`
- `sources/agentmemory/src/functions/governance.ts:193` `failures.push({ id: mem.id, error: "delete_failed" });`
- `sources/agentmemory/src/functions/governance.ts:194` `} else if (result.value === "deleted") {`
- `sources/agentmemory/src/functions/governance.ts:195` `successfulIds.push(mem.id);`
- `sources/agentmemory/src/functions/governance.ts:196` `} else {`
- `sources/agentmemory/src/functions/governance.ts:197` `notFound.push(mem.id);`
- `sources/agentmemory/src/functions/governance.ts:198` `}`
- `sources/agentmemory/src/functions/governance.ts:201` ``
- `sources/agentmemory/src/functions/governance.ts:202` `await deleteFacetsForTargets(kv, [...successfulIds, ...notFound]);`
- `sources/agentmemory/src/functions/governance.ts:203` ``
- `sources/agentmemory/src/functions/governance.ts:204` `if (successfulIds.length > 0) await flushIndexSave();`
- `sources/agentmemory/src/functions/governance.ts:205` ``
- `sources/agentmemory/src/functions/governance.ts:206` `if (successfulIds.length > 0 || failures.length > 0) {`
- `sources/agentmemory/src/functions/governance.ts:208` `kv,`
- `sources/agentmemory/src/functions/governance.ts:209` `"delete",`
- `sources/agentmemory/src/functions/governance.ts:210` `"mem::governance-bulk",`
- `sources/agentmemory/src/functions/governance.ts:211` `successfulIds,`
- `sources/agentmemory/src/functions/governance.ts:212` `{`
- `sources/agentmemory/src/functions/governance.ts:213` `filter: data,`
- `sources/agentmemory/src/functions/governance.ts:214` `deleted: successfulIds.length,`
- `sources/agentmemory/src/functions/governance.ts:215` `notFound: notFound.length,`
- `sources/agentmemory/src/functions/governance.ts:216` `failed: failures.length,`
- `sources/agentmemory/src/functions/governance.ts:217` `failures: failures.length > 0 ? failures : undefined,`
- `sources/agentmemory/src/functions/governance.ts:218` `},`
- `sources/agentmemory/src/functions/governance.ts:221` ``
- `sources/agentmemory/src/functions/governance.ts:222` `logger.info("Governance bulk delete", {`
- `sources/agentmemory/src/functions/governance.ts:223` `deleted: successfulIds.length,`
- `sources/agentmemory/src/functions/governance.ts:224` `notFound: notFound.length,`
- `sources/agentmemory/src/functions/governance.ts:225` `failed: failures.length,`
- `sources/agentmemory/src/functions/governance.ts:226` `});`
- `sources/agentmemory/src/functions/governance.ts:227` `return {`
- `sources/agentmemory/src/functions/governance.ts:228` `success: failures.length === 0,`
- `sources/agentmemory/src/functions/governance.ts:229` `deleted: successfulIds.length,`
- `sources/agentmemory/src/functions/governance.ts:230` `notFound,`
- `sources/agentmemory/src/functions/governance.ts:231` `failed: failures.length,`
- `sources/agentmemory/src/functions/governance.ts:232` `failures: failures.length > 0 ? failures : undefined,`
- `sources/agentmemory/src/functions/governance.ts:233` `};`

**operation and route call sites**, body `sources/agentmemory/src/functions/graph-retrieval.ts:174`; body evidence.

- `sources/agentmemory/src/functions/graph-retrieval.ts:173` `if (!evidence || !matchesSearchLayer(id, evidence.observation.sessionId, layer, evidence.layer)) continue;`
- `sources/agentmemory/src/functions/graph-retrieval.ts:174` `results.delete(id);`
- `sources/agentmemory/src/functions/graph-retrieval.ts:175` `results.set(id, { obsId: id, sessionId: evidence.observation.sessionId, score, pathLength,`
- `sources/agentmemory/src/functions/graph-retrieval.ts:176` `graphContext: '[graph evidence] ${evidence.observation.title.slice(0, 512)}' });`
- `sources/agentmemory/src/functions/graph-retrieval.ts:177` `}`
- `sources/agentmemory/src/functions/graph-retrieval.ts:178` `}`

**operation and route call sites**, body `sources/agentmemory/src/functions/graph.ts:221`; body evidence.

- `sources/agentmemory/src/functions/graph.ts:220` `const evicted = snap.topNodes.pop();`
- `sources/agentmemory/src/functions/graph.ts:221` `if (evicted) delete snap.topDegrees[evicted.id];`
- `sources/agentmemory/src/functions/graph.ts:222` `snap.topNodes.push(node);`
- `sources/agentmemory/src/functions/graph.ts:223` `snap.topDegrees[node.id] = next;`
- `sources/agentmemory/src/functions/graph.ts:224` `snap.topNodes.sort(`
- `sources/agentmemory/src/functions/graph.ts:225` `(a, b) =>`
- `sources/agentmemory/src/functions/graph.ts:250` `for (const id of incoming) {`
- `sources/agentmemory/src/functions/graph.ts:251` `merged.delete(id);`
- `sources/agentmemory/src/functions/graph.ts:252` `merged.add(id);`
- `sources/agentmemory/src/functions/graph.ts:253` `}`
- `sources/agentmemory/src/functions/graph.ts:254` `return [...merged].slice(-MAX_GRAPH_SOURCE_OBSERVATIONS);`
- `sources/agentmemory/src/functions/graph.ts:255` `}`
- `sources/agentmemory/src/functions/graph.ts:709` `if (scope === undefined) {`
- `sources/agentmemory/src/functions/graph.ts:710` `delete result.total;`
- `sources/agentmemory/src/functions/graph.ts:711` `result.nextOffset = null;`
- `sources/agentmemory/src/functions/graph.ts:712` `}`
- `sources/agentmemory/src/functions/graph.ts:713` `return result;`
- `sources/agentmemory/src/functions/graph.ts:714` `}`

**operation and route call sites**, body `sources/agentmemory/src/functions/image-quota-cleanup.ts:6`; body evidence.

- `sources/agentmemory/src/functions/image-quota-cleanup.ts:5` `import { join } from "node:path";`
- `sources/agentmemory/src/functions/image-quota-cleanup.ts:6` `import { IMAGES_DIR, getMaxBytes, deleteImage } from "../utils/image-store.js";`
- `sources/agentmemory/src/functions/image-quota-cleanup.ts:7` `import { withKeyedLock } from "../state/keyed-mutex.js";`
- `sources/agentmemory/src/functions/image-quota-cleanup.ts:8` `import { logger } from "../logger.js";`
- `sources/agentmemory/src/functions/image-quota-cleanup.ts:9` ``
- `sources/agentmemory/src/functions/image-quota-cleanup.ts:10` `const GRACE_PERIOD_MS = 30_000;`
- `sources/agentmemory/src/functions/image-quota-cleanup.ts:75` ``
- `sources/agentmemory/src/functions/image-quota-cleanup.ts:76` `const { deletedBytes } = await deleteImage(f.filePath);`
- `sources/agentmemory/src/functions/image-quota-cleanup.ts:77` `if (deletedBytes > 0) {`
- `sources/agentmemory/src/functions/image-quota-cleanup.ts:78` `sdk.trigger({`
- `sources/agentmemory/src/functions/image-quota-cleanup.ts:79` `function_id: "mem::disk-size-delta",`
- `sources/agentmemory/src/functions/image-quota-cleanup.ts:80` `payload: { deltaBytes: -deletedBytes },`
- `sources/agentmemory/src/functions/image-quota-cleanup.ts:81` `action: TriggerAction.Void(),`
- `sources/agentmemory/src/functions/image-quota-cleanup.ts:82` `});`
- `sources/agentmemory/src/functions/image-quota-cleanup.ts:83` `totalToFree -= deletedBytes;`
- `sources/agentmemory/src/functions/image-quota-cleanup.ts:84` `freedBytes += deletedBytes;`
- `sources/agentmemory/src/functions/image-quota-cleanup.ts:85` `evicted++;`
- `sources/agentmemory/src/functions/image-quota-cleanup.ts:86` `}`
- `sources/agentmemory/src/functions/image-quota-cleanup.ts:87` `});`
- `sources/agentmemory/src/functions/image-quota-cleanup.ts:88` `}`

**operation and route call sites**, body `sources/agentmemory/src/functions/image-refs.ts:4`; body evidence.

- `sources/agentmemory/src/functions/image-refs.ts:3` `import { StateKV } from "../state/kv.js";`
- `sources/agentmemory/src/functions/image-refs.ts:4` `import { deleteImage, touchImage } from "../utils/image-store.js";`
- `sources/agentmemory/src/functions/image-refs.ts:5` `import { withKeyedLock } from "../state/keyed-mutex.js";`
- `sources/agentmemory/src/functions/image-refs.ts:6` ``
- `sources/agentmemory/src/functions/image-refs.ts:7` `export async function getImageRefCount(kv: StateKV, filePath: string): Promise<number> {`
- `sources/agentmemory/src/functions/image-refs.ts:8` `const count = await kv.get<number>(KV.imageRefs, filePath);`
- `sources/agentmemory/src/functions/image-refs.ts:23` `if (current <= 1) {`
- `sources/agentmemory/src/functions/image-refs.ts:24` `await kv.delete(KV.imageEmbeddings, filePath);`
- `sources/agentmemory/src/functions/image-refs.ts:25` `await kv.delete(KV.imageRefs, filePath);`
- `sources/agentmemory/src/functions/image-refs.ts:26` `const { deletedBytes } = await deleteImage(filePath);`
- `sources/agentmemory/src/functions/image-refs.ts:27` `if (deletedBytes > 0) {`
- `sources/agentmemory/src/functions/image-refs.ts:28` `sdk.trigger({`
- `sources/agentmemory/src/functions/image-refs.ts:29` `function_id: "mem::disk-size-delta",`
- `sources/agentmemory/src/functions/image-refs.ts:30` `payload: { deltaBytes: -deletedBytes },`
- `sources/agentmemory/src/functions/image-refs.ts:31` `action: TriggerAction.Void(),`
- `sources/agentmemory/src/functions/image-refs.ts:32` `});`
- `sources/agentmemory/src/functions/image-refs.ts:33` `}`
- `sources/agentmemory/src/functions/image-refs.ts:34` `} else {`

**operation and route call sites**, body `sources/agentmemory/src/functions/lessons.ts:8`; body evidence.

- `sources/agentmemory/src/functions/lessons.ts:7` `import { recordAudit } from "./audit.js";`
- `sources/agentmemory/src/functions/lessons.ts:8` `import { deleteFacetsForTargets } from "./facets.js";`
- `sources/agentmemory/src/functions/lessons.ts:9` ``
- `sources/agentmemory/src/functions/lessons.ts:10` `// Dedicated BM25 index for lessons, with the full records cached`
- `sources/agentmemory/src/functions/lessons.ts:11` `// alongside it. Recall previously listed every lesson from KV and`
- `sources/agentmemory/src/functions/lessons.ts:12` `// substring-matched per query , O(corpus) per call with no term`
- `sources/agentmemory/src/functions/lessons.ts:41` `for (const l of all) {`
- `sources/agentmemory/src/functions/lessons.ts:42` `if (!l.deleted) {`
- `sources/agentmemory/src/functions/lessons.ts:43` `idx.add(lessonToObservation(l), "lesson");`
- `sources/agentmemory/src/functions/lessons.ts:44` `lessonRecords.set(l.id, l);`
- `sources/agentmemory/src/functions/lessons.ts:45` `}`
- `sources/agentmemory/src/functions/lessons.ts:46` `}`
- `sources/agentmemory/src/functions/lessons.ts:108` ``
- `sources/agentmemory/src/functions/lessons.ts:109` `if (existing && !existing.deleted) {`
- `sources/agentmemory/src/functions/lessons.ts:110` `reinforceLesson(existing);`
- `sources/agentmemory/src/functions/lessons.ts:111` `let indexedTextChanged = false;`
- `sources/agentmemory/src/functions/lessons.ts:112` `if (data.context && !existing.context) {`
- `sources/agentmemory/src/functions/lessons.ts:113` `existing.context = data.context;`
- `sources/agentmemory/src/functions/lessons.ts:200` `const l = lessonRecords.get(hits[i].obsId);`
- `sources/agentmemory/src/functions/lessons.ts:201` `if (!l || l.deleted || l.confidence < minConfidence) continue;`
- `sources/agentmemory/src/functions/lessons.ts:202` `if (data.project && l.project !== data.project) continue;`
- `sources/agentmemory/src/functions/lessons.ts:203` ``
- `sources/agentmemory/src/functions/lessons.ts:204` `const relevance = maxHit > 0 ? hits[i].score / maxHit : 0;`
- `sources/agentmemory/src/functions/lessons.ts:205` `const daysSinceReinforced = l.lastReinforcedAt`
- `sources/agentmemory/src/functions/lessons.ts:248` `lessons = lessons.filter(`
- `sources/agentmemory/src/functions/lessons.ts:249` `(l) => !l.deleted && l.confidence >= minConfidence,`
- `sources/agentmemory/src/functions/lessons.ts:250` `);`
- `sources/agentmemory/src/functions/lessons.ts:251` ``
- `sources/agentmemory/src/functions/lessons.ts:252` `if (data.project) {`
- `sources/agentmemory/src/functions/lessons.ts:253` `lessons = lessons.filter((l) => l.project === data.project);`
- `sources/agentmemory/src/functions/lessons.ts:271` `const lesson = await kv.get<Lesson>(KV.lessons, data.lessonId);`
- `sources/agentmemory/src/functions/lessons.ts:272` `if (!lesson || lesson.deleted) {`
- `sources/agentmemory/src/functions/lessons.ts:273` `return { success: false, error: "lesson not found" };`
- `sources/agentmemory/src/functions/lessons.ts:274` `}`
- `sources/agentmemory/src/functions/lessons.ts:275` ``
- `sources/agentmemory/src/functions/lessons.ts:276` `reinforceLesson(lesson);`
- `sources/agentmemory/src/functions/lessons.ts:291` ``
- `sources/agentmemory/src/functions/lessons.ts:292` `sdk.registerFunction("mem::lesson-delete",`
- `sources/agentmemory/src/functions/lessons.ts:293` `async (data: { lessonId: string }) => {`
- `sources/agentmemory/src/functions/lessons.ts:294` `if (!data.lessonId) {`
- `sources/agentmemory/src/functions/lessons.ts:295` `return { success: false, error: "lessonId is required" };`
- `sources/agentmemory/src/functions/lessons.ts:296` `}`
- `sources/agentmemory/src/functions/lessons.ts:298` `const lesson = await kv.get<Lesson>(KV.lessons, data.lessonId);`
- `sources/agentmemory/src/functions/lessons.ts:299` `if (!lesson || lesson.deleted) {`
- `sources/agentmemory/src/functions/lessons.ts:300` `return { success: false, error: "lesson not found" };`
- `sources/agentmemory/src/functions/lessons.ts:301` `}`
- `sources/agentmemory/src/functions/lessons.ts:302` ``
- `sources/agentmemory/src/functions/lessons.ts:303` `lesson.deleted = true;`
- `sources/agentmemory/src/functions/lessons.ts:304` `lesson.updatedAt = new Date().toISOString();`
- `sources/agentmemory/src/functions/lessons.ts:305` ``
- `sources/agentmemory/src/functions/lessons.ts:306` `await kv.set(KV.lessons, lesson.id, lesson);`
- `sources/agentmemory/src/functions/lessons.ts:307` `lessonRecords.delete(lesson.id);`
- `sources/agentmemory/src/functions/lessons.ts:308` `if (lessonIndex) lessonIndex.remove(lesson.id);`
- `sources/agentmemory/src/functions/lessons.ts:309` `noteLessonMutation();`
- `sources/agentmemory/src/functions/lessons.ts:310` `await deleteFacetsForTargets(kv, [lesson.id]);`
- `sources/agentmemory/src/functions/lessons.ts:311` ``
- `sources/agentmemory/src/functions/lessons.ts:312` `try {`
- `sources/agentmemory/src/functions/lessons.ts:313` `await recordAudit(kv, "lesson_delete", "mem::lesson-delete", [`
- `sources/agentmemory/src/functions/lessons.ts:314` `lesson.id,`
- `sources/agentmemory/src/functions/lessons.ts:315` `]);`
- `sources/agentmemory/src/functions/lessons.ts:316` `} catch {}`
- `sources/agentmemory/src/functions/lessons.ts:317` ``
- `sources/agentmemory/src/functions/lessons.ts:325` `let decayed = 0;`
- `sources/agentmemory/src/functions/lessons.ts:326` `let softDeleted = 0;`
- `sources/agentmemory/src/functions/lessons.ts:327` `const now = Date.now();`
- `sources/agentmemory/src/functions/lessons.ts:328` `const timestamp = new Date().toISOString();`
- `sources/agentmemory/src/functions/lessons.ts:329` `const dirty: Lesson[] = [];`
- `sources/agentmemory/src/functions/lessons.ts:330` `const auditEvents: Array<{`
- `sources/agentmemory/src/functions/lessons.ts:331` `id: string;`
- `sources/agentmemory/src/functions/lessons.ts:332` `action: "decay" | "soft-delete";`
- `sources/agentmemory/src/functions/lessons.ts:333` `beforeConfidence: number;`
- `sources/agentmemory/src/functions/lessons.ts:334` `afterConfidence: number;`
- `sources/agentmemory/src/functions/lessons.ts:335` `beforeDeleted: boolean;`
- `sources/agentmemory/src/functions/lessons.ts:336` `afterDeleted: boolean;`
- `sources/agentmemory/src/functions/lessons.ts:337` `}> = [];`
- `sources/agentmemory/src/functions/lessons.ts:338` ``
- `sources/agentmemory/src/functions/lessons.ts:339` `for (const lesson of lessons) {`
- `sources/agentmemory/src/functions/lessons.ts:340` `if (lesson.deleted) continue;`
- `sources/agentmemory/src/functions/lessons.ts:341` ``
- `sources/agentmemory/src/functions/lessons.ts:342` `const baseline = lesson.lastDecayedAt || lesson.lastReinforcedAt || lesson.createdAt;`
- `sources/agentmemory/src/functions/lessons.ts:343` `const weeksSinceBaseline =`
- `sources/agentmemory/src/functions/lessons.ts:344` `(now - new Date(baseline).getTime()) / (1000 * 60 * 60 * 24 * 7);`
- `sources/agentmemory/src/functions/lessons.ts:352` `const beforeConfidence = lesson.confidence;`
- `sources/agentmemory/src/functions/lessons.ts:353` `const beforeDeleted = !!lesson.deleted;`
- `sources/agentmemory/src/functions/lessons.ts:354` `lesson.confidence = Math.round(newConfidence * 1000) / 1000;`
- `sources/agentmemory/src/functions/lessons.ts:355` `lesson.lastDecayedAt = timestamp;`
- `sources/agentmemory/src/functions/lessons.ts:356` `lesson.updatedAt = timestamp;`
- `sources/agentmemory/src/functions/lessons.ts:357` ``
- `sources/agentmemory/src/functions/lessons.ts:358` `if (lesson.confidence <= 0.1 && lesson.reinforcements === 0) {`
- `sources/agentmemory/src/functions/lessons.ts:359` `lesson.deleted = true;`
- `sources/agentmemory/src/functions/lessons.ts:360` `softDeleted++;`
- `sources/agentmemory/src/functions/lessons.ts:361` `} else {`
- `sources/agentmemory/src/functions/lessons.ts:362` `decayed++;`
- `sources/agentmemory/src/functions/lessons.ts:363` `}`
- `sources/agentmemory/src/functions/lessons.ts:364` ``
- `sources/agentmemory/src/functions/lessons.ts:367` `id: lesson.id,`
- `sources/agentmemory/src/functions/lessons.ts:368` `action: lesson.deleted ? "soft-delete" : "decay",`
- `sources/agentmemory/src/functions/lessons.ts:369` `beforeConfidence,`
- `sources/agentmemory/src/functions/lessons.ts:370` `afterConfidence: lesson.confidence,`
- `sources/agentmemory/src/functions/lessons.ts:371` `beforeDeleted,`
- `sources/agentmemory/src/functions/lessons.ts:372` `afterDeleted: !!lesson.deleted,`
- `sources/agentmemory/src/functions/lessons.ts:373` `});`
- `sources/agentmemory/src/functions/lessons.ts:374` `}`
- `sources/agentmemory/src/functions/lessons.ts:375` `}`
- `sources/agentmemory/src/functions/lessons.ts:376` ``
- `sources/agentmemory/src/functions/lessons.ts:377` `await Promise.all(dirty.map((l) => kv.set(KV.lessons, l.id, l)));`
- `sources/agentmemory/src/functions/lessons.ts:378` `const softDeletedIds: string[] = [];`
- `sources/agentmemory/src/functions/lessons.ts:379` `for (const l of dirty) {`
- `sources/agentmemory/src/functions/lessons.ts:380` `if (l.deleted) {`
- `sources/agentmemory/src/functions/lessons.ts:381` `softDeletedIds.push(l.id);`
- `sources/agentmemory/src/functions/lessons.ts:382` `lessonRecords.delete(l.id);`
- `sources/agentmemory/src/functions/lessons.ts:383` `if (lessonIndex) lessonIndex.remove(l.id);`
- `sources/agentmemory/src/functions/lessons.ts:384` `} else {`
- `sources/agentmemory/src/functions/lessons.ts:385` `lessonRecords.set(l.id, l);`
- `sources/agentmemory/src/functions/lessons.ts:386` `}`
- `sources/agentmemory/src/functions/lessons.ts:388` `if (dirty.length > 0) noteLessonMutation();`
- `sources/agentmemory/src/functions/lessons.ts:389` `if (softDeletedIds.length > 0) {`
- `sources/agentmemory/src/functions/lessons.ts:390` `await deleteFacetsForTargets(kv, softDeletedIds);`
- `sources/agentmemory/src/functions/lessons.ts:391` `}`
- `sources/agentmemory/src/functions/lessons.ts:392` `await Promise.all(`
- `sources/agentmemory/src/functions/lessons.ts:393` `auditEvents.map((event) =>`
- `sources/agentmemory/src/functions/lessons.ts:394` `recordAudit(kv, "lesson_strengthen", "mem::lesson-decay-sweep", [event.id], {`
- `sources/agentmemory/src/functions/lessons.ts:399` `confidence: event.beforeConfidence,`
- `sources/agentmemory/src/functions/lessons.ts:400` `deleted: event.beforeDeleted,`
- `sources/agentmemory/src/functions/lessons.ts:401` `},`
- `sources/agentmemory/src/functions/lessons.ts:402` `after: {`
- `sources/agentmemory/src/functions/lessons.ts:403` `confidence: event.afterConfidence,`
- `sources/agentmemory/src/functions/lessons.ts:404` `deleted: event.afterDeleted,`
- `sources/agentmemory/src/functions/lessons.ts:405` `},`
- `sources/agentmemory/src/functions/lessons.ts:406` `}),`
- `sources/agentmemory/src/functions/lessons.ts:407` `),`
- `sources/agentmemory/src/functions/lessons.ts:408` `);`
- `sources/agentmemory/src/functions/lessons.ts:409` ``
- `sources/agentmemory/src/functions/lessons.ts:410` `return { success: true, decayed, softDeleted, total: lessons.length };`
- `sources/agentmemory/src/functions/lessons.ts:411` `},`
- `sources/agentmemory/src/functions/lessons.ts:412` `);`
- `sources/agentmemory/src/functions/lessons.ts:413` `}`

**operation and route call sites**, body `sources/agentmemory/src/functions/mesh.ts:387`; body evidence.

- `sources/agentmemory/src/functions/mesh.ts:386` `}`
- `sources/agentmemory/src/functions/mesh.ts:387` `await kv.delete(KV.mesh, data.peerId);`
- `sources/agentmemory/src/functions/mesh.ts:388` `await recordAudit(kv, "mesh_sync", "mem::mesh-remove", [data.peerId], {`
- `sources/agentmemory/src/functions/mesh.ts:389` `action: "mesh.remove",`
- `sources/agentmemory/src/functions/mesh.ts:390` `});`
- `sources/agentmemory/src/functions/mesh.ts:391` `return { success: true };`

**operation and route call sites**, body `sources/agentmemory/src/functions/observation-source-budget.ts:46`; body evidence.

- `sources/agentmemory/src/functions/observation-source-budget.ts:45` `if (payloadByteLength(metadata) > perSource) {`
- `sources/agentmemory/src/functions/observation-source-budget.ts:46` `delete row.source;`
- `sources/agentmemory/src/functions/observation-source-budget.ts:47` `sourceOmitted++;`
- `sources/agentmemory/src/functions/observation-source-budget.ts:48` `continue;`
- `sources/agentmemory/src/functions/observation-source-budget.ts:49` `}`
- `sources/agentmemory/src/functions/observation-source-budget.ts:50` `row.source = normalizeObservationSource(source, perSource);`

**operation and route call sites**, body `sources/agentmemory/src/functions/obsidian-export.ts:318`; body evidence.

- `sources/agentmemory/src/functions/obsidian-export.ts:317` `for (const l of lessons.filter(`
- `sources/agentmemory/src/functions/obsidian-export.ts:318` `(l): l is Lesson & { id: string } => hasExportId(l) && !l.deleted,`
- `sources/agentmemory/src/functions/obsidian-export.ts:319` `)) {`
- `sources/agentmemory/src/functions/obsidian-export.ts:320` `const filename = '${sanitize(l.id)}.md';`
- `sources/agentmemory/src/functions/obsidian-export.ts:321` `const filepath = join(dirs.lessons, filename);`
- `sources/agentmemory/src/functions/obsidian-export.ts:322` `try {`

**operation and route call sites**, body `sources/agentmemory/src/functions/recent-searches-sweep.ts:31`; body evidence.

- `sources/agentmemory/src/functions/recent-searches-sweep.ts:30` `try {`
- `sources/agentmemory/src/functions/recent-searches-sweep.ts:31` `await kv.delete(KV.recentSearches, row.sessionId);`
- `sources/agentmemory/src/functions/recent-searches-sweep.ts:32` `swept++;`
- `sources/agentmemory/src/functions/recent-searches-sweep.ts:33` `} catch (err) {`
- `sources/agentmemory/src/functions/recent-searches-sweep.ts:34` `logger.warn("recent-searches sweep delete failed", {`
- `sources/agentmemory/src/functions/recent-searches-sweep.ts:35` `sessionId: row.sessionId,`
- `sources/agentmemory/src/functions/recent-searches-sweep.ts:36` `error: err instanceof Error ? err.message : String(err),`
- `sources/agentmemory/src/functions/recent-searches-sweep.ts:37` `});`
- `sources/agentmemory/src/functions/recent-searches-sweep.ts:38` `}`

**operation and route call sites**, body `sources/agentmemory/src/functions/reflect.ts:140`; body evidence.

- `sources/agentmemory/src/functions/reflect.ts:139` `const current = await kv.get<Record<string, unknown>>(scope, id);`
- `sources/agentmemory/src/functions/reflect.ts:140` `return !!current && current[field] === text && current.deleted !== true;`
- `sources/agentmemory/src/functions/reflect.ts:141` `}))).every(Boolean);`
- `sources/agentmemory/src/functions/reflect.ts:142` `}`
- `sources/agentmemory/src/functions/reflect.ts:143` ``
- `sources/agentmemory/src/functions/reflect.ts:144` `function reinforceInsight(insight: Insight): void {`
- `sources/agentmemory/src/functions/reflect.ts:368` ``
- `sources/agentmemory/src/functions/reflect.ts:369` `let activeLessons = lessons.filter((l) => !l.deleted);`
- `sources/agentmemory/src/functions/reflect.ts:370` `let scopedCrystals = crystals;`
- `sources/agentmemory/src/functions/reflect.ts:371` `if (data?.project) {`
- `sources/agentmemory/src/functions/reflect.ts:372` `activeLessons = activeLessons.filter((l) => l.project === data.project);`
- `sources/agentmemory/src/functions/reflect.ts:373` `scopedCrystals = crystals.filter((c) => c.project === data.project);`
- `sources/agentmemory/src/functions/reflect.ts:493` `};`
- `sources/agentmemory/src/functions/reflect.ts:494` `if (existing && !existing.deleted) {`
- `sources/agentmemory/src/functions/reflect.ts:495` `const insight = {`
- `sources/agentmemory/src/functions/reflect.ts:496` `...existing, ...evidence, title,`
- `sources/agentmemory/src/functions/reflect.ts:497` `sourceConceptCluster: conceptNames, tags: conceptNames,`
- `sources/agentmemory/src/functions/reflect.ts:498` `};`
- `sources/agentmemory/src/functions/reflect.ts:575` `items = items.filter(`
- `sources/agentmemory/src/functions/reflect.ts:576` `(i) => !i.deleted && i.confidence >= minConfidence,`
- `sources/agentmemory/src/functions/reflect.ts:577` `);`
- `sources/agentmemory/src/functions/reflect.ts:578` ``
- `sources/agentmemory/src/functions/reflect.ts:579` `if (data?.project) {`
- `sources/agentmemory/src/functions/reflect.ts:580` `items = items.filter((i) => i.project === data.project);`
- `sources/agentmemory/src/functions/reflect.ts:607` `items = items.filter(`
- `sources/agentmemory/src/functions/reflect.ts:608` `(i) => !i.deleted && i.confidence >= minConfidence,`
- `sources/agentmemory/src/functions/reflect.ts:609` `);`
- `sources/agentmemory/src/functions/reflect.ts:610` ``
- `sources/agentmemory/src/functions/reflect.ts:611` `if (data.project) {`
- `sources/agentmemory/src/functions/reflect.ts:612` `items = items.filter((i) => i.project === data.project);`
- `sources/agentmemory/src/functions/reflect.ts:659` `let decayed = 0;`
- `sources/agentmemory/src/functions/reflect.ts:660` `let softDeleted = 0;`
- `sources/agentmemory/src/functions/reflect.ts:661` `const now = Date.now();`
- `sources/agentmemory/src/functions/reflect.ts:662` `const timestamp = new Date().toISOString();`
- `sources/agentmemory/src/functions/reflect.ts:663` `const dirty: Insight[] = [];`
- `sources/agentmemory/src/functions/reflect.ts:664` ``
- `sources/agentmemory/src/functions/reflect.ts:667` `const current = await kv.get<Insight>(KV.insights, item.id);`
- `sources/agentmemory/src/functions/reflect.ts:668` `if (!current || current.deleted) return;`
- `sources/agentmemory/src/functions/reflect.ts:669` `const insight = { ...current };`
- `sources/agentmemory/src/functions/reflect.ts:670` `const baseline = Math.max(...[`
- `sources/agentmemory/src/functions/reflect.ts:671` `insight.createdAt, insight.lastDecayedAt, insight.lastReinforcedAt,`
- `sources/agentmemory/src/functions/reflect.ts:672` `].filter((date): date is string => !!date).map((date) => new Date(date).getTime()));`
- `sources/agentmemory/src/functions/reflect.ts:680` `insight.updatedAt = timestamp;`
- `sources/agentmemory/src/functions/reflect.ts:681` `if (insight.confidence <= 0.1 && insight.reinforcements === 0) insight.deleted = true;`
- `sources/agentmemory/src/functions/reflect.ts:682` `await kv.set(KV.insights, insight.id, insight);`
- `sources/agentmemory/src/functions/reflect.ts:683` `if (insight.deleted) softDeleted++;`
- `sources/agentmemory/src/functions/reflect.ts:684` `else decayed++;`
- `sources/agentmemory/src/functions/reflect.ts:685` `dirty.push(insight);`
- `sources/agentmemory/src/functions/reflect.ts:686` `});`
- `sources/agentmemory/src/functions/reflect.ts:687` `}`
- `sources/agentmemory/src/functions/reflect.ts:691` `decayed,`
- `sources/agentmemory/src/functions/reflect.ts:692` `softDeleted,`
- `sources/agentmemory/src/functions/reflect.ts:693` `total: items.length,`
- `sources/agentmemory/src/functions/reflect.ts:694` `timestamp,`
- `sources/agentmemory/src/functions/reflect.ts:695` `});`
- `sources/agentmemory/src/functions/reflect.ts:696` ``
- `sources/agentmemory/src/functions/reflect.ts:697` `return { success: true, decayed, softDeleted, total: items.length };`
- `sources/agentmemory/src/functions/reflect.ts:698` `},`
- `sources/agentmemory/src/functions/reflect.ts:699` `);`
- `sources/agentmemory/src/functions/reflect.ts:700` `}`

**operation and route call sites**, body `sources/agentmemory/src/functions/remember.ts:2`; body evidence.

- `sources/agentmemory/src/functions/remember.ts:1` `import { TriggerAction, type IIIClient } from "iii-sdk";`
- `sources/agentmemory/src/functions/remember.ts:2` `import { markCaptureEventDeleted } from "../capture/event-record.js";`
- `sources/agentmemory/src/functions/remember.ts:3` `import type { Memory, Session } from "../types.js";`
- `sources/agentmemory/src/functions/remember.ts:4` `import { KV, generateId, jaccardSimilarity } from "../state/schema.js";`
- `sources/agentmemory/src/functions/remember.ts:5` `import { StateKV, collectAll } from "../state/kv.js";`
- `sources/agentmemory/src/functions/remember.ts:6` `import { withKeyedLock } from "../state/keyed-mutex.js";`
- `sources/agentmemory/src/functions/remember.ts:9` `import { memoryToObservation, observationEmbedText, supersededAt } from "../state/memory-utils.js";`
- `sources/agentmemory/src/functions/remember.ts:10` `import { deleteAccessLog } from "./access-tracker.js";`
- `sources/agentmemory/src/functions/remember.ts:11` `import { deleteFacetsForTargets } from "./facets.js";`
- `sources/agentmemory/src/functions/remember.ts:12` `import { recordAudit } from "./audit.js";`
- `sources/agentmemory/src/functions/remember.ts:13` `import { getSearchIndex, getVectorIndex, isMemoryIndexReady, scheduleIndexSave, vectorIndexAddGuarded, vectorIndexRemove, flushIndexSave } from "./search.js";`
- `sources/agentmemory/src/functions/remember.ts:14` `import { getAgentId } from "../config.js";`
- `sources/agentmemory/src/functions/remember.ts:15` `import { logger } from "../logger.js";`
- `sources/agentmemory/src/functions/remember.ts:181` `if (data.ttlDays && typeof data.ttlDays === "number" && data.ttlDays > 0) {`
- `sources/agentmemory/src/functions/remember.ts:182` `memory.forgetAfter = new Date(Date.now() + data.ttlDays * 86400000).toISOString();`
- `sources/agentmemory/src/functions/remember.ts:183` `}`
- `sources/agentmemory/src/functions/remember.ts:184` ``
- `sources/agentmemory/src/functions/remember.ts:185` `if (supersededMemory) {`
- `sources/agentmemory/src/functions/remember.ts:186` `supersededMemory.isLatest = false;`
- `sources/agentmemory/src/functions/remember.ts:266` ``
- `sources/agentmemory/src/functions/remember.ts:267` `sdk.registerFunction("mem::forget",`
- `sources/agentmemory/src/functions/remember.ts:268` `async (data: {`
- `sources/agentmemory/src/functions/remember.ts:269` `sessionId?: string;`
- `sources/agentmemory/src/functions/remember.ts:270` `observationIds?: string[];`
- `sources/agentmemory/src/functions/remember.ts:271` `memoryId?: string;`
- `sources/agentmemory/src/functions/remember.ts:273` `type ObservationRef = { id?: string; imageData?: string; imageRef?: string; captureKey?: string };`
- `sources/agentmemory/src/functions/remember.ts:274` `let deleted = 0;`
- `sources/agentmemory/src/functions/remember.ts:275` `const deletedMemoryIds: string[] = [];`
- `sources/agentmemory/src/functions/remember.ts:276` `const deletedObservationIds: string[] = [];`
- `sources/agentmemory/src/functions/remember.ts:277` `const notFound: string[] = [];`
- `sources/agentmemory/src/functions/remember.ts:278` `const failures: Array<{ id: string; error: string }> = [];`
- `sources/agentmemory/src/functions/remember.ts:279` `let deletedSession = false;`
- `sources/agentmemory/src/functions/remember.ts:280` `let deletedSummary = false;`
- `sources/agentmemory/src/functions/remember.ts:281` `let indexCleaned = false;`
- `sources/agentmemory/src/functions/remember.ts:282` `const { decrementImageRef } = await import("./image-refs.js");`
- `sources/agentmemory/src/functions/remember.ts:283` ``
- `sources/agentmemory/src/functions/remember.ts:284` `const recordFailure = (id: string, err: unknown) => {`
- `sources/agentmemory/src/functions/remember.ts:285` `logger.warn("Forget failed", {`
- `sources/agentmemory/src/functions/remember.ts:286` `id,`
- `sources/agentmemory/src/functions/remember.ts:287` `error: err instanceof Error ? err.message : String(err),`
- `sources/agentmemory/src/functions/remember.ts:288` `});`
- `sources/agentmemory/src/functions/remember.ts:289` `failures.push({ id, error: "delete_failed" });`
- `sources/agentmemory/src/functions/remember.ts:290` `};`
- `sources/agentmemory/src/functions/remember.ts:291` ``
- `sources/agentmemory/src/functions/remember.ts:292` `const cleanupFailures: Array<{ id: string; error: string }> = [];`
- `sources/agentmemory/src/functions/remember.ts:293` `const cleanup = async (id: string, steps: () => Promise<void>) => {`
- `sources/agentmemory/src/functions/remember.ts:296` `} catch (err) {`
- `sources/agentmemory/src/functions/remember.ts:297` `logger.warn("Forget cleanup failed", {`
- `sources/agentmemory/src/functions/remember.ts:298` `id,`
- `sources/agentmemory/src/functions/remember.ts:299` `error: err instanceof Error ? err.message : String(err),`
- `sources/agentmemory/src/functions/remember.ts:300` `});`
- `sources/agentmemory/src/functions/remember.ts:301` `cleanupFailures.push({ id, error: "cleanup_failed" });`
- `sources/agentmemory/src/functions/remember.ts:306` `try {`
- `sources/agentmemory/src/functions/remember.ts:307` `if (await remove()) deleted++;`
- `sources/agentmemory/src/functions/remember.ts:308` `else notFound.push(id);`
- `sources/agentmemory/src/functions/remember.ts:309` `} catch (err) {`
- `sources/agentmemory/src/functions/remember.ts:310` `recordFailure(id, err);`
- `sources/agentmemory/src/functions/remember.ts:311` `}`
- `sources/agentmemory/src/functions/remember.ts:313` ``
- `sources/agentmemory/src/functions/remember.ts:314` `const forgetObservation = async (`
- `sources/agentmemory/src/functions/remember.ts:315` `sessionId: string,`
- `sources/agentmemory/src/functions/remember.ts:316` `obsId: string,`
- `sources/agentmemory/src/functions/remember.ts:317` `known?: ObservationRef,`
- `sources/agentmemory/src/functions/remember.ts:318` `): Promise<boolean> => {`
- `sources/agentmemory/src/functions/remember.ts:322` `if (!obs) return false;`
- `sources/agentmemory/src/functions/remember.ts:323` `await markCaptureEventDeleted(kv, { ...obs, id: obsId, sessionId });`
- `sources/agentmemory/src/functions/remember.ts:324` `await kv.delete(KV.observations(sessionId), obsId);`
- `sources/agentmemory/src/functions/remember.ts:325` `deletedObservationIds.push(obsId);`
- `sources/agentmemory/src/functions/remember.ts:326` `getSearchIndex().remove(obsId);`
- `sources/agentmemory/src/functions/remember.ts:327` `vectorIndexRemove(obsId);`
- `sources/agentmemory/src/functions/remember.ts:328` `indexCleaned = true;`
- `sources/agentmemory/src/functions/remember.ts:329` `await unindexObservationSession(kv, obsId).catch(() => {});`
- `sources/agentmemory/src/functions/remember.ts:343` `const current = await kv.get<Memory>(KV.memories, memoryId);`
- `sources/agentmemory/src/functions/remember.ts:344` `if (current) await kv.delete(KV.memories, memoryId);`
- `sources/agentmemory/src/functions/remember.ts:345` `return current;`
- `sources/agentmemory/src/functions/remember.ts:346` `});`
- `sources/agentmemory/src/functions/remember.ts:347` `if (!mem) {`
- `sources/agentmemory/src/functions/remember.ts:348` `await deleteFacetsForTargets(kv, [memoryId]);`
- `sources/agentmemory/src/functions/remember.ts:349` `return false;`
- `sources/agentmemory/src/functions/remember.ts:350` `}`
- `sources/agentmemory/src/functions/remember.ts:351` `deletedMemoryIds.push(memoryId);`
- `sources/agentmemory/src/functions/remember.ts:352` `getSearchIndex().remove(memoryId);`
- `sources/agentmemory/src/functions/remember.ts:353` `vectorIndexRemove(memoryId);`
- `sources/agentmemory/src/functions/remember.ts:354` `indexCleaned = true;`
- `sources/agentmemory/src/functions/remember.ts:355` `await cleanup(memoryId, async () => {`
- `sources/agentmemory/src/functions/remember.ts:356` `await deleteAccessLog(kv, memoryId);`
- `sources/agentmemory/src/functions/remember.ts:357` `await deleteFacetsForTargets(kv, [memoryId]);`
- `sources/agentmemory/src/functions/remember.ts:358` `if (mem.imageRef) await decrementImageRef(kv, sdk, mem.imageRef);`
- `sources/agentmemory/src/functions/remember.ts:359` `});`
- `sources/agentmemory/src/functions/remember.ts:360` `return true;`
- `sources/agentmemory/src/functions/remember.ts:361` `});`
- `sources/agentmemory/src/functions/remember.ts:371` `await attempt(obsId, () =>`
- `sources/agentmemory/src/functions/remember.ts:372` `withKeyedLock('obs:${sessionId}', () => forgetObservation(sessionId, obsId)),`
- `sources/agentmemory/src/functions/remember.ts:373` `);`
- `sources/agentmemory/src/functions/remember.ts:374` `}`
- `sources/agentmemory/src/functions/remember.ts:375` `}`
- `sources/agentmemory/src/functions/remember.ts:376` ``
- `sources/agentmemory/src/functions/remember.ts:387` `for (const obs of observations) {`
- `sources/agentmemory/src/functions/remember.ts:388` `await attempt(obs.id, () => forgetObservation(sessionId, obs.id, obs));`
- `sources/agentmemory/src/functions/remember.ts:389` `}`
- `sources/agentmemory/src/functions/remember.ts:390` `await attempt(sessionId, async () => {`
- `sources/agentmemory/src/functions/remember.ts:391` `const session = await kv.get<Session>(KV.sessions, sessionId);`
- `sources/agentmemory/src/functions/remember.ts:392` `if (!session) return false;`
- `sources/agentmemory/src/functions/remember.ts:393` `await kv.delete(KV.sessions, sessionId);`
- `sources/agentmemory/src/functions/remember.ts:394` `await kv.delete(KV.summaryRepairs, sessionId);`
- `sources/agentmemory/src/functions/remember.ts:395` `await removeSessionFromProjectIndex(`
- `sources/agentmemory/src/functions/remember.ts:396` `kv,`
- `sources/agentmemory/src/functions/remember.ts:397` `session.project,`
- `sources/agentmemory/src/functions/remember.ts:398` `sessionId,`
- `sources/agentmemory/src/functions/remember.ts:399` `).catch(() => {});`
- `sources/agentmemory/src/functions/remember.ts:400` `deletedSession = true;`
- `sources/agentmemory/src/functions/remember.ts:401` `return true;`
- `sources/agentmemory/src/functions/remember.ts:402` `});`
- `sources/agentmemory/src/functions/remember.ts:403` `try {`
- `sources/agentmemory/src/functions/remember.ts:404` `const summary = await kv.get(KV.summaries, sessionId);`
- `sources/agentmemory/src/functions/remember.ts:405` `if (summary) {`
- `sources/agentmemory/src/functions/remember.ts:406` `await kv.delete(KV.summaries, sessionId);`
- `sources/agentmemory/src/functions/remember.ts:407` `deletedSummary = true;`
- `sources/agentmemory/src/functions/remember.ts:408` `deleted++;`
- `sources/agentmemory/src/functions/remember.ts:409` `}`
- `sources/agentmemory/src/functions/remember.ts:410` `} catch (err) {`
- `sources/agentmemory/src/functions/remember.ts:411` `recordFailure('summary:${sessionId}', err);`
- `sources/agentmemory/src/functions/remember.ts:412` `}`
- `sources/agentmemory/src/functions/remember.ts:415` ``
- `sources/agentmemory/src/functions/remember.ts:416` `if (deletedObservationIds.length > 0) {`
- `sources/agentmemory/src/functions/remember.ts:417` `await deleteFacetsForTargets(kv, deletedObservationIds, {`
- `sources/agentmemory/src/functions/remember.ts:418` `targetTypes: ["memory", "observation"],`
- `sources/agentmemory/src/functions/remember.ts:419` `});`
- `sources/agentmemory/src/functions/remember.ts:420` `}`
- `sources/agentmemory/src/functions/remember.ts:421` ``
- `sources/agentmemory/src/functions/remember.ts:422` `if (deleted > 0 || indexCleaned) await flushIndexSave();`
- `sources/agentmemory/src/functions/remember.ts:423` ``
- `sources/agentmemory/src/functions/remember.ts:424` `if (deleted > 0 || failures.length > 0) {`
- `sources/agentmemory/src/functions/remember.ts:425` `await recordAudit(`
- `sources/agentmemory/src/functions/remember.ts:426` `kv,`
- `sources/agentmemory/src/functions/remember.ts:427` `"forget",`
- `sources/agentmemory/src/functions/remember.ts:428` `"mem::forget",`
- `sources/agentmemory/src/functions/remember.ts:429` `[...deletedMemoryIds, ...deletedObservationIds],`
- `sources/agentmemory/src/functions/remember.ts:430` `{`
- `sources/agentmemory/src/functions/remember.ts:431` `sessionId: data.sessionId,`
- `sources/agentmemory/src/functions/remember.ts:432` `deleted,`
- `sources/agentmemory/src/functions/remember.ts:433` `memoriesDeleted: deletedMemoryIds.length,`
- `sources/agentmemory/src/functions/remember.ts:434` `observationsDeleted: deletedObservationIds.length,`
- `sources/agentmemory/src/functions/remember.ts:435` `sessionDeleted: deletedSession,`
- `sources/agentmemory/src/functions/remember.ts:436` `summaryDeleted: deletedSummary,`
- `sources/agentmemory/src/functions/remember.ts:437` `notFound: notFound.length,`
- `sources/agentmemory/src/functions/remember.ts:438` `failed: failures.length,`
- `sources/agentmemory/src/functions/remember.ts:439` `failures: failures.length > 0 ? failures : undefined,`
- `sources/agentmemory/src/functions/remember.ts:440` `cleanupFailed: cleanupFailures.length,`
- `sources/agentmemory/src/functions/remember.ts:441` `cleanupFailures: cleanupFailures.length > 0 ? cleanupFailures : undefined,`
- `sources/agentmemory/src/functions/remember.ts:442` `reason: "user-initiated forget",`
- `sources/agentmemory/src/functions/remember.ts:443` `},`
- `sources/agentmemory/src/functions/remember.ts:444` `);`
- `sources/agentmemory/src/functions/remember.ts:445` `}`
- `sources/agentmemory/src/functions/remember.ts:446` ``
- `sources/agentmemory/src/functions/remember.ts:447` `logger.info("Memory forgotten", {`
- `sources/agentmemory/src/functions/remember.ts:448` `deleted,`
- `sources/agentmemory/src/functions/remember.ts:449` `notFound: notFound.length,`
- `sources/agentmemory/src/functions/remember.ts:450` `failed: failures.length,`
- `sources/agentmemory/src/functions/remember.ts:451` `});`
- `sources/agentmemory/src/functions/remember.ts:452` `return {`
- `sources/agentmemory/src/functions/remember.ts:453` `success: failures.length === 0,`
- `sources/agentmemory/src/functions/remember.ts:454` `deleted,`
- `sources/agentmemory/src/functions/remember.ts:455` `notFound,`
- `sources/agentmemory/src/functions/remember.ts:456` `failed: failures.length,`
- `sources/agentmemory/src/functions/remember.ts:457` `failures: failures.length > 0 ? failures : undefined,`
- `sources/agentmemory/src/functions/remember.ts:458` `cleanupFailed: cleanupFailures.length,`

**operation and route call sites**, body `sources/agentmemory/src/functions/retention.ts:14`; body evidence.

- `sources/agentmemory/src/functions/retention.ts:13` `emptyAccessLog,`
- `sources/agentmemory/src/functions/retention.ts:14` `deleteAccessLog,`
- `sources/agentmemory/src/functions/retention.ts:15` `normalizeAccessLog,`
- `sources/agentmemory/src/functions/retention.ts:16` `} from "./access-tracker.js";`
- `sources/agentmemory/src/functions/retention.ts:17` `import { recordAudit, runAuditRetentionSweep } from "./audit.js";`
- `sources/agentmemory/src/functions/retention.ts:18` `import { deleteFacetsForTargets } from "./facets.js";`
- `sources/agentmemory/src/functions/retention.ts:19` `import { getSearchIndex, vectorIndexRemove, flushIndexSave } from "./search.js";`
- `sources/agentmemory/src/functions/retention.ts:20` `import { logger } from "../logger.js";`
- `sources/agentmemory/src/functions/retention.ts:21` `import { getAuditRetentionMonths } from "../config.js";`
- `sources/agentmemory/src/functions/retention.ts:22` `const DEFAULT_DECAY: DecayConfig = {`
- `sources/agentmemory/src/functions/retention.ts:378` `}`
- `sources/agentmemory/src/functions/retention.ts:379` `await kv.delete(scope, candidate.memoryId);`
- `sources/agentmemory/src/functions/retention.ts:380` `evictedIds.push(candidate.memoryId);`
- `sources/agentmemory/src/functions/retention.ts:381` `evicted++;`
- `sources/agentmemory/src/functions/retention.ts:382` `if (resolvedSource === "semantic") evictedSemantic++;`
- `sources/agentmemory/src/functions/retention.ts:383` `else evictedEpisodic++;`
- `sources/agentmemory/src/functions/retention.ts:384` `await kv.delete(KV.retentionScores, candidate.memoryId);`
- `sources/agentmemory/src/functions/retention.ts:385` `await deleteAccessLog(kv, candidate.memoryId);`
- `sources/agentmemory/src/functions/retention.ts:386` `getSearchIndex().remove(candidate.memoryId);`
- `sources/agentmemory/src/functions/retention.ts:387` `vectorIndexRemove(candidate.memoryId);`
- `sources/agentmemory/src/functions/retention.ts:388` `} catch {`
- `sources/agentmemory/src/functions/retention.ts:389` `continue;`
- `sources/agentmemory/src/functions/retention.ts:392` ``
- `sources/agentmemory/src/functions/retention.ts:393` `await deleteFacetsForTargets(kv, [...evictedIds, ...missingIds]);`
- `sources/agentmemory/src/functions/retention.ts:394` ``
- `sources/agentmemory/src/functions/retention.ts:395` `// Retention eviction is a structural delete path that removes`
- `sources/agentmemory/src/functions/retention.ts:396` `// memories, retention scores, and access logs, so it needs to`
- `sources/agentmemory/src/functions/retention.ts:397` `// emit an audit record per the repo's audit-coverage policy (see`
- `sources/agentmemory/src/functions/retention.ts:402` `await flushIndexSave();`
- `sources/agentmemory/src/functions/retention.ts:403` `await recordAudit(kv, "delete", "mem::retention-evict", evictedIds, {`
- `sources/agentmemory/src/functions/retention.ts:404` `threshold,`
- `sources/agentmemory/src/functions/retention.ts:405` `evicted,`
- `sources/agentmemory/src/functions/retention.ts:406` `evictedEpisodic,`
- `sources/agentmemory/src/functions/retention.ts:407` `evictedSemantic,`

**operation and route call sites**, body `sources/agentmemory/src/functions/search.ts:151`; body evidence.

- `sources/agentmemory/src/functions/search.ts:150` `export function vectorIndexRemove(id: string): void {`
- `sources/agentmemory/src/functions/search.ts:151` `getSearchIndex().invalidateRevision(id);`
- `sources/agentmemory/src/functions/search.ts:152` `vectorIndex?.remove(id);`
- `sources/agentmemory/src/functions/search.ts:153` `}`
- `sources/agentmemory/src/functions/search.ts:154` ``
- `sources/agentmemory/src/functions/search.ts:155` `// Persistence sync hook. Without this, index removals only live in`

**operation and route call sites**, body `sources/agentmemory/src/functions/session-finalize.ts:53`; body evidence.

- `sources/agentmemory/src/functions/session-finalize.ts:52` `if (!(sessionId in due)) return;`
- `sources/agentmemory/src/functions/session-finalize.ts:53` `delete due[sessionId];`
- `sources/agentmemory/src/functions/session-finalize.ts:54` `await kv.set(KV.config, FINALIZE_DUE_KEY, due);`
- `sources/agentmemory/src/functions/session-finalize.ts:55` `});`
- `sources/agentmemory/src/functions/session-finalize.ts:56` `}`
- `sources/agentmemory/src/functions/session-finalize.ts:57` ``

**operation and route call sites**, body `sources/agentmemory/src/functions/signals.ts:188`; body evidence.

- `sources/agentmemory/src/functions/signals.ts:187` `if (sig.expiresAt && new Date(sig.expiresAt).getTime() <= now) {`
- `sources/agentmemory/src/functions/signals.ts:188` `await recordAudit(kv, "delete", "mem::signal-cleanup", [sig.id], {`
- `sources/agentmemory/src/functions/signals.ts:189` `action: "delete",`
- `sources/agentmemory/src/functions/signals.ts:190` `resource: "Signal",`
- `sources/agentmemory/src/functions/signals.ts:191` `before: sig,`
- `sources/agentmemory/src/functions/signals.ts:192` `});`
- `sources/agentmemory/src/functions/signals.ts:193` `await kv.delete(KV.signals, sig.id);`
- `sources/agentmemory/src/functions/signals.ts:194` `removed++;`
- `sources/agentmemory/src/functions/signals.ts:195` `}`
- `sources/agentmemory/src/functions/signals.ts:196` `}`
- `sources/agentmemory/src/functions/signals.ts:197` ``

**operation and route call sites**, body `sources/agentmemory/src/functions/sketches.ts:151`; body evidence.

- `sources/agentmemory/src/functions/sketches.ts:150` `if (action) {`
- `sources/agentmemory/src/functions/sketches.ts:151` `delete action.sketchId;`
- `sources/agentmemory/src/functions/sketches.ts:152` `if (data.project) {`
- `sources/agentmemory/src/functions/sketches.ts:153` `action.project = data.project;`
- `sources/agentmemory/src/functions/sketches.ts:154` `}`
- `sources/agentmemory/src/functions/sketches.ts:155` `action.updatedAt = new Date().toISOString();`
- `sources/agentmemory/src/functions/sketches.ts:200` `) {`
- `sources/agentmemory/src/functions/sketches.ts:201` `await kv.delete(KV.actionEdges, edge.id);`
- `sources/agentmemory/src/functions/sketches.ts:202` `await safeAudit(kv, "sketch_discard", "mem::sketch-discard", [edge.id], {`
- `sources/agentmemory/src/functions/sketches.ts:203` `action: "discard.edge",`
- `sources/agentmemory/src/functions/sketches.ts:204` `sketchId: sketch.id,`
- `sources/agentmemory/src/functions/sketches.ts:205` `});`
- `sources/agentmemory/src/functions/sketches.ts:209` `for (const actionId of sketch.actionIds) {`
- `sources/agentmemory/src/functions/sketches.ts:210` `await kv.delete(KV.actions, actionId);`
- `sources/agentmemory/src/functions/sketches.ts:211` `await safeAudit(kv, "sketch_discard", "mem::sketch-discard", [actionId], {`
- `sources/agentmemory/src/functions/sketches.ts:212` `action: "discard.action",`
- `sources/agentmemory/src/functions/sketches.ts:213` `sketchId: sketch.id,`
- `sources/agentmemory/src/functions/sketches.ts:214` `});`
- `sources/agentmemory/src/functions/sketches.ts:285` `) {`
- `sources/agentmemory/src/functions/sketches.ts:286` `await kv.delete(KV.actionEdges, edge.id);`
- `sources/agentmemory/src/functions/sketches.ts:287` `await safeAudit(kv, "sketch_discard", "mem::sketch-gc", [edge.id], {`
- `sources/agentmemory/src/functions/sketches.ts:288` `action: "gc.edge",`
- `sources/agentmemory/src/functions/sketches.ts:289` `sketchId: current.id,`
- `sources/agentmemory/src/functions/sketches.ts:290` `});`
- `sources/agentmemory/src/functions/sketches.ts:294` `for (const actionId of current.actionIds) {`
- `sources/agentmemory/src/functions/sketches.ts:295` `await kv.delete(KV.actions, actionId);`
- `sources/agentmemory/src/functions/sketches.ts:296` `await safeAudit(kv, "sketch_discard", "mem::sketch-gc", [actionId], {`
- `sources/agentmemory/src/functions/sketches.ts:297` `action: "gc.action",`
- `sources/agentmemory/src/functions/sketches.ts:298` `sketchId: current.id,`
- `sources/agentmemory/src/functions/sketches.ts:299` `});`

**operation and route call sites**, body `sources/agentmemory/src/functions/slots.ts:342`; body evidence.

- `sources/agentmemory/src/functions/slots.ts:341` `sdk.registerFunction(`
- `sources/agentmemory/src/functions/slots.ts:342` `"mem::slot-delete",`
- `sources/agentmemory/src/functions/slots.ts:343` `async (data: { label?: string }) => {`
- `sources/agentmemory/src/functions/slots.ts:344` `const label = validateLabel(data?.label);`
- `sources/agentmemory/src/functions/slots.ts:345` `if (!label) return { success: false, error: "label required" };`
- `sources/agentmemory/src/functions/slots.ts:346` `return withKeyedLock('slot:${label}', async () => {`
- `sources/agentmemory/src/functions/slots.ts:349` `if (slot.readOnly) return { success: false, error: "slot is read-only" };`
- `sources/agentmemory/src/functions/slots.ts:350` `await kv.delete(scopeKv(scope), label);`
- `sources/agentmemory/src/functions/slots.ts:351` `await recordAudit(kv, "slot_delete", "mem::slot-delete", [label], {`
- `sources/agentmemory/src/functions/slots.ts:352` `scope,`
- `sources/agentmemory/src/functions/slots.ts:353` `size: slot.content.length,`
- `sources/agentmemory/src/functions/slots.ts:354` `});`
- `sources/agentmemory/src/functions/slots.ts:355` `return { success: true };`

**operation and route call sites**, body `sources/agentmemory/src/functions/smart-search.ts:309`; body evidence.

- `sources/agentmemory/src/functions/smart-search.ts:308` `.finally(() => {`
- `sources/agentmemory/src/functions/smart-search.ts:309` `pendingFollowups.delete(detection);`
- `sources/agentmemory/src/functions/smart-search.ts:310` `});`
- `sources/agentmemory/src/functions/smart-search.ts:311` `pendingFollowups.add(detection);`
- `sources/agentmemory/src/functions/smart-search.ts:312` `}`
- `sources/agentmemory/src/functions/smart-search.ts:313` ``

**operation and route call sites**, body `sources/agentmemory/src/functions/snapshot.ts:191`; body evidence.

- `sources/agentmemory/src/functions/snapshot.ts:190` `const content = await gitExec(snapshotDir, ["show", '${data.commitHash}:state.json'], Infinity);`
- `sources/agentmemory/src/functions/snapshot.ts:191` `const archive = JSON.parse(content) as {`
- `sources/agentmemory/src/functions/snapshot.ts:192` `sessions?: Array<{ id: string } & Record<string, unknown>>;`
- `sources/agentmemory/src/functions/snapshot.ts:193` `memories?: Memory[];`
- `sources/agentmemory/src/functions/snapshot.ts:194` `graphNodes?: GraphNode[];`
- `sources/agentmemory/src/functions/snapshot.ts:195` `graphEdges?: GraphEdge[];`
- `sources/agentmemory/src/functions/snapshot.ts:198` `};`
- `sources/agentmemory/src/functions/snapshot.ts:199` `validateGraphRecords(archive.graphNodes ?? [], archive.graphEdges ?? []);`
- `sources/agentmemory/src/functions/snapshot.ts:200` `const memoryIds = new Set<string>();`
- `sources/agentmemory/src/functions/snapshot.ts:201` `const observationSessions = new Map<string, string>();`
- `sources/agentmemory/src/functions/snapshot.ts:202` `let restoreError: unknown;`
- `sources/agentmemory/src/functions/snapshot.ts:203` `try {`
- `sources/agentmemory/src/functions/snapshot.ts:204` `if (archive.sessions) {`
- `sources/agentmemory/src/functions/snapshot.ts:205` `for (const session of archive.sessions) {`
- `sources/agentmemory/src/functions/snapshot.ts:206` `await withKeyedLock('obs:${session.id}', () =>`
- `sources/agentmemory/src/functions/snapshot.ts:207` `kv.set(KV.sessions, session.id, session),`
- `sources/agentmemory/src/functions/snapshot.ts:208` `);`
- `sources/agentmemory/src/functions/snapshot.ts:209` `const project = session.project;`
- `sources/agentmemory/src/functions/snapshot.ts:220` `}`
- `sources/agentmemory/src/functions/snapshot.ts:221` `for (const memory of archive.memories ?? []) {`
- `sources/agentmemory/src/functions/snapshot.ts:222` `await kv.set(KV.memories, memory.id, memory);`
- `sources/agentmemory/src/functions/snapshot.ts:223` `getSearchIndex().remove(memory.id);`
- `sources/agentmemory/src/functions/snapshot.ts:224` `vectorIndexRemove(memory.id);`
- `sources/agentmemory/src/functions/snapshot.ts:225` `memoryIds.add(memory.id);`
- `sources/agentmemory/src/functions/snapshot.ts:226` `}`
- `sources/agentmemory/src/functions/snapshot.ts:227` `for (const [sessionId, observations] of Object.entries(archive.observations ?? {})) {`
- `sources/agentmemory/src/functions/snapshot.ts:228` `await withKeyedLock('obs:${sessionId}', async () => {`
- `sources/agentmemory/src/functions/snapshot.ts:229` `for (const observation of observations) {`
- `sources/agentmemory/src/functions/snapshot.ts:230` `await kv.set(KV.observations(sessionId), observation.id, observation);`
- `sources/agentmemory/src/functions/snapshot.ts:231` `getSearchIndex().remove(observation.id);`
- `sources/agentmemory/src/functions/snapshot.ts:236` `}`
- `sources/agentmemory/src/functions/snapshot.ts:237` `if (archive.graphNodes?.length || archive.graphEdges?.length) {`
- `sources/agentmemory/src/functions/snapshot.ts:238` `await persistGraphRecords(kv, archive.graphNodes ?? [], archive.graphEdges ?? [], { mode: "merge" });`
- `sources/agentmemory/src/functions/snapshot.ts:239` `}`
- `sources/agentmemory/src/functions/snapshot.ts:240` `for (const log of archive.accessLogs ?? []) {`
- `sources/agentmemory/src/functions/snapshot.ts:241` `await kv.set(KV.accessLog, log.memoryId, log);`
- `sources/agentmemory/src/functions/snapshot.ts:242` `}`
- `sources/agentmemory/src/functions/snapshot.ts:243` `} catch (err) {`
- `sources/agentmemory/src/functions/snapshot.ts:244` `restoreError = err;`
- `sources/agentmemory/src/functions/snapshot.ts:280` `commitHash: data.commitHash,`
- `sources/agentmemory/src/functions/snapshot.ts:281` `sessions: archive.sessions?.length || 0,`
- `sources/agentmemory/src/functions/snapshot.ts:282` `memories: archive.memories?.length || 0,`
- `sources/agentmemory/src/functions/snapshot.ts:283` `graphNodes: archive.graphNodes?.length || 0,`
- `sources/agentmemory/src/functions/snapshot.ts:284` `});`
- `sources/agentmemory/src/functions/snapshot.ts:285` ``
- `sources/agentmemory/src/functions/snapshot.ts:286` `logger.info("Snapshot restored", {`
- `sources/agentmemory/src/functions/snapshot.ts:287` `commitHash: data.commitHash,`

**operation and route call sites**, body `sources/agentmemory/src/functions/status.ts:477`; body evidence.

- `sources/agentmemory/src/functions/status.ts:476` `? "Safe to ignore. To remove it, run redis-cli -u \"$AGENTMEMORY_REDIS_URL\" DEL state:mem:audit, then restart agentmemory."`
- `sources/agentmemory/src/functions/status.ts:477` `: "Safe to ignore. To remove it, stop agentmemory, delete mem%3Aaudit.bin from the state store directory, then start it again.",`
- `sources/agentmemory/src/functions/status.ts:478` `});`
- `sources/agentmemory/src/functions/status.ts:479` `}`
- `sources/agentmemory/src/functions/status.ts:480` ``
- `sources/agentmemory/src/functions/status.ts:481` `if (input.capture === null) {`

**operation and route call sites**, body `sources/agentmemory/src/functions/summary-recovery.ts:74`; body evidence.

- `sources/agentmemory/src/functions/summary-recovery.ts:73` `if (!session) {`
- `sources/agentmemory/src/functions/summary-recovery.ts:74` `await kv.delete(KV.summaryRepairs, sessionId);`
- `sources/agentmemory/src/functions/summary-recovery.ts:75` `return;`
- `sources/agentmemory/src/functions/summary-recovery.ts:76` `}`
- `sources/agentmemory/src/functions/summary-recovery.ts:77` `const rows = await kv.list<StoredObservation>(KV.observations(sessionId));`
- `sources/agentmemory/src/functions/summary-recovery.ts:78` `if (session.id === sessionId && session.status !== "active" && !rows.some(needsCompression) &&`
- `sources/agentmemory/src/functions/summary-recovery.ts:80` `await kv.get<SessionSummary>(KV.summaries, sessionId)))) {`
- `sources/agentmemory/src/functions/summary-recovery.ts:81` `await kv.delete(KV.summaryRepairs, sessionId);`
- `sources/agentmemory/src/functions/summary-recovery.ts:82` `return;`
- `sources/agentmemory/src/functions/summary-recovery.ts:83` `}`
- `sources/agentmemory/src/functions/summary-recovery.ts:84` `} catch {`
- `sources/agentmemory/src/functions/summary-recovery.ts:85` `lastError = "recovery_failed";`
- `sources/agentmemory/src/functions/summary-recovery.ts:99` `catch (error) { logger.warn("Could not persist summary recovery progress", { sessionId: request.sessionId, error: error instanceof Error ? error.message : String(error) }); }`
- `sources/agentmemory/src/functions/summary-recovery.ts:100` `finally { active.delete(request.sessionId); }`
- `sources/agentmemory/src/functions/summary-recovery.ts:101` `});`
- `sources/agentmemory/src/functions/summary-recovery.ts:102` `sdk.registerFunction("mem::summary-recover", async (data?: { now?: number }) => {`
- `sources/agentmemory/src/functions/summary-recovery.ts:103` `const now = typeof data?.now === "number" && Number.isFinite(data.now) ? data.now : Date.now();`
- `sources/agentmemory/src/functions/summary-recovery.ts:104` `const pending = await kv.list<SessionSummaryRepair>(KV.summaryRepairs);`

**operation and route call sites**, body `sources/agentmemory/src/functions/temporal-graph.ts:227`; body evidence.

- `sources/agentmemory/src/functions/temporal-graph.ts:226` `};`
- `sources/agentmemory/src/functions/temporal-graph.ts:227` `if (merged.aliases.length === 0) delete (merged as any).aliases;`
- `sources/agentmemory/src/functions/temporal-graph.ts:228` `await persistGraphRecords(kv, [merged], []);`
- `sources/agentmemory/src/functions/temporal-graph.ts:229` `node.id = existing.id;`
- `sources/agentmemory/src/functions/temporal-graph.ts:230` `idRemap.set(oldId, existing.id);`
- `sources/agentmemory/src/functions/temporal-graph.ts:231` `} else {`

**operation and route call sites**, body `sources/agentmemory/src/functions/working-memory.ts:75`; body evidence.

- `sources/agentmemory/src/functions/working-memory.ts:74` `if (!data?.id) return { success: false, error: "id is required" };`
- `sources/agentmemory/src/functions/working-memory.ts:75` `await kv.delete(CORE_SCOPE, data.id);`
- `sources/agentmemory/src/functions/working-memory.ts:76` ``
- `sources/agentmemory/src/functions/working-memory.ts:77` `try {`
- `sources/agentmemory/src/functions/working-memory.ts:78` `await recordAudit(kv, "core_remove", "mem::core-remove", [data.id], {});`
- `sources/agentmemory/src/functions/working-memory.ts:79` `} catch {}`
- `sources/agentmemory/src/functions/working-memory.ts:234` `await kv.set(KV.memories, archivalMemory.id, archivalMemory);`
- `sources/agentmemory/src/functions/working-memory.ts:235` `await kv.delete(CORE_SCOPE, entry.id);`
- `sources/agentmemory/src/functions/working-memory.ts:236` ``
- `sources/agentmemory/src/functions/working-memory.ts:237` `totalTokens -= tokens;`
- `sources/agentmemory/src/functions/working-memory.ts:238` `paged++;`
- `sources/agentmemory/src/functions/working-memory.ts:239` `pagedIds.push(entry.id);`

**operation and route call sites**, body `sources/agentmemory/src/hooks/_endpoint.ts:32`; body evidence.

- `sources/agentmemory/src/hooks/_endpoint.ts:31` `const explicit = hookSetting("AGENTMEMORY_SECRET", env);`
- `sources/agentmemory/src/hooks/_endpoint.ts:32` `if (explicit === undefined) delete secretEnv["AGENTMEMORY_SECRET"];`
- `sources/agentmemory/src/hooks/_endpoint.ts:33` `else secretEnv["AGENTMEMORY_SECRET"] = explicit;`
- `sources/agentmemory/src/hooks/_endpoint.ts:34` `return resolveClientSecret(url, secretEnv);`
- `sources/agentmemory/src/hooks/_endpoint.ts:35` `}`
- `sources/agentmemory/src/hooks/_endpoint.ts:36` ``

**operation and route call sites**, body `sources/agentmemory/src/hooks/cursor/workspace.ts:153`; body evidence.

- `sources/agentmemory/src/hooks/cursor/workspace.ts:152` `const previous = loadSessionCache();`
- `sources/agentmemory/src/hooks/cursor/workspace.ts:153` `delete previous[sessionId];`
- `sources/agentmemory/src/hooks/cursor/workspace.ts:154` `const cache = pruneSessionCache({ [sessionId]: { project, cwd, updatedAt: new Date(now).toISOString() }, ...previous }, now);`
- `sources/agentmemory/src/hooks/cursor/workspace.ts:155` `const tmp = '${SESSION_CACHE_PATH}.${process.pid}.${Date.now()}.tmp';`
- `sources/agentmemory/src/hooks/cursor/workspace.ts:156` `writeFileSync(tmp, JSON.stringify(cache, null, 2));`
- `sources/agentmemory/src/hooks/cursor/workspace.ts:157` `renameSync(tmp, SESSION_CACHE_PATH);`

**operation and route call sites**, body `sources/agentmemory/src/index.ts:81`; body evidence.

- `sources/agentmemory/src/index.ts:80` `import { registerProfileFunction } from "./functions/profile.js";`
- `sources/agentmemory/src/index.ts:81` `import { registerAutoForgetFunction } from "./functions/auto-forget.js";`
- `sources/agentmemory/src/index.ts:82` `import { registerExportImportFunction } from "./functions/export-import.js";`
- `sources/agentmemory/src/index.ts:83` `import { registerEnrichFunction } from "./functions/enrich.js";`
- `sources/agentmemory/src/index.ts:84` `import { registerClaudeBridgeFunction } from "./functions/claude-bridge.js";`
- `sources/agentmemory/src/index.ts:85` `import { registerGraphFunction } from "./functions/graph.js";`
- `sources/agentmemory/src/index.ts:332` `registerProfileFunction(sdk, kv);`
- `sources/agentmemory/src/index.ts:333` `registerAutoForgetFunction(sdk, kv);`
- `sources/agentmemory/src/index.ts:334` `registerExportImportFunction(sdk, kv);`
- `sources/agentmemory/src/index.ts:335` `registerEnrichFunction(sdk, kv);`
- `sources/agentmemory/src/index.ts:336` ``
- `sources/agentmemory/src/index.ts:337` `const claudeBridgeConfig = loadClaudeBridgeConfig();`
- `sources/agentmemory/src/index.ts:690` ``
- `sources/agentmemory/src/index.ts:691` `const autoForgetIntervalMs = parseInt(process.env.AUTO_FORGET_INTERVAL_MS || "3600000", 10);`
- `sources/agentmemory/src/index.ts:692` `const consolidationIntervalMs = getConsolidationIntervalMs();`
- `sources/agentmemory/src/index.ts:693` ``
- `sources/agentmemory/src/index.ts:694` `if (process.env.AUTO_FORGET_ENABLED !== "false") {`
- `sources/agentmemory/src/index.ts:695` `const autoForgetTimer = setInterval(async () => {`
- `sources/agentmemory/src/index.ts:696` `try {`
- `sources/agentmemory/src/index.ts:697` `await sdk.trigger({ function_id: "mem::auto-forget", payload: { dryRun: false } });`
- `sources/agentmemory/src/index.ts:698` `} catch {}`
- `sources/agentmemory/src/index.ts:699` `}, autoForgetIntervalMs);`
- `sources/agentmemory/src/index.ts:700` `autoForgetTimer.unref();`
- `sources/agentmemory/src/index.ts:701` `bootLog('Auto-forget: enabled (every ${autoForgetIntervalMs / 60000}m)');`
- `sources/agentmemory/src/index.ts:702` `}`
- `sources/agentmemory/src/index.ts:703` ``
- `sources/agentmemory/src/index.ts:704` `const auditRetentionMonths = getAuditRetentionMonths();`
- `sources/agentmemory/src/index.ts:705` `if (auditRetentionMonths > 0) {`

**operation and route call sites**, body `sources/agentmemory/src/integrations/dsh.ts:141`; body evidence.

- `sources/agentmemory/src/integrations/dsh.ts:140` `enqueue(state, () => post("/session/end", { sessionId: id }));`
- `sources/agentmemory/src/integrations/dsh.ts:141` `return state.tail.finally(() => states.delete(id));`
- `sources/agentmemory/src/integrations/dsh.ts:142` `}`
- `sources/agentmemory/src/integrations/dsh.ts:143` ``
- `sources/agentmemory/src/integrations/dsh.ts:144` `ctx.on("session/created", sessionState);`
- `sources/agentmemory/src/integrations/dsh.ts:145` `ctx.on("session/event", (session, event) => {`
- `sources/agentmemory/src/integrations/dsh.ts:164` `const call = state.calls.get(id);`
- `sources/agentmemory/src/integrations/dsh.ts:165` `state.calls.delete(id);`
- `sources/agentmemory/src/integrations/dsh.ts:166` `if (call) observe(session, state, event, (message.isError || data.error) ? "post_tool_failure" : "post_tool_use", {`
- `sources/agentmemory/src/integrations/dsh.ts:167` `tool_name: call.name, tool_input: call.input, tool_output: safe(message.content, captureOutputMax()), call_id: id,`
- `sources/agentmemory/src/integrations/dsh.ts:168` `});`
- `sources/agentmemory/src/integrations/dsh.ts:169` `} else if (event.type === "compaction/summary") {`

**operation and route call sites**, body `sources/agentmemory/src/mcp/in-memory-kv.ts:36`; body evidence.

- `sources/agentmemory/src/mcp/in-memory-kv.ts:35` ``
- `sources/agentmemory/src/mcp/in-memory-kv.ts:36` `async delete(scope: string, key: string): Promise<void> {`
- `sources/agentmemory/src/mcp/in-memory-kv.ts:37` `this.store.get(scope)?.delete(key);`
- `sources/agentmemory/src/mcp/in-memory-kv.ts:38` `}`
- `sources/agentmemory/src/mcp/in-memory-kv.ts:39` ``
- `sources/agentmemory/src/mcp/in-memory-kv.ts:40` `async list<T = unknown>(scope: string): Promise<T[]> {`
- `sources/agentmemory/src/mcp/in-memory-kv.ts:41` `const entries = this.store.get(scope);`

**operation and route call sites**, body `sources/agentmemory/src/mcp/rest-proxy.ts:133`; body evidence.

- `sources/agentmemory/src/mcp/rest-proxy.ts:132` ``
- `sources/agentmemory/src/mcp/rest-proxy.ts:133` `export function invalidateHandle(): void {`
- `sources/agentmemory/src/mcp/rest-proxy.ts:134` `cached = null;`
- `sources/agentmemory/src/mcp/rest-proxy.ts:135` `cachedUrl = null;`
- `sources/agentmemory/src/mcp/rest-proxy.ts:136` `cachedAt = 0;`
- `sources/agentmemory/src/mcp/rest-proxy.ts:137` `}`
- `sources/agentmemory/src/mcp/rest-proxy.ts:141` `const url = baseUrl();`
- `sources/agentmemory/src/mcp/rest-proxy.ts:142` `if (cached && (cachedUrl !== url || (cached.mode === "local" && isProxyRequired()))) invalidateHandle();`
- `sources/agentmemory/src/mcp/rest-proxy.ts:143` `if (cached) {`
- `sources/agentmemory/src/mcp/rest-proxy.ts:144` `if (cached.mode === "local" && now - cachedAt >= LOCAL_MODE_TTL_MS) {`
- `sources/agentmemory/src/mcp/rest-proxy.ts:145` `cached = null;`
- `sources/agentmemory/src/mcp/rest-proxy.ts:146` `cachedAt = 0;`

**operation and route call sites**, body `sources/agentmemory/src/mcp/server.ts:645`; body evidence.

- `sources/agentmemory/src/mcp/server.ts:644` ``
- `sources/agentmemory/src/mcp/server.ts:645` `case "memory_governance_delete": {`
- `sources/agentmemory/src/mcp/server.ts:646` `if (typeof args.memoryIds !== "string") {`
- `sources/agentmemory/src/mcp/server.ts:647` `return {`
- `sources/agentmemory/src/mcp/server.ts:648` `status_code: 400,`
- `sources/agentmemory/src/mcp/server.ts:649` `body: { error: "memoryIds is required" },`
- `sources/agentmemory/src/mcp/server.ts:656` `try {`
- `sources/agentmemory/src/mcp/server.ts:657` `const result = await sdk.trigger({ function_id: "mem::governance-delete", payload: {`
- `sources/agentmemory/src/mcp/server.ts:658` `memoryIds: ids,`
- `sources/agentmemory/src/mcp/server.ts:659` `reason: args.reason as string,`
- `sources/agentmemory/src/mcp/server.ts:660` `} });`
- `sources/agentmemory/src/mcp/server.ts:661` `return {`
- `sources/agentmemory/src/mcp/server.ts:672` `body: {`
- `sources/agentmemory/src/mcp/server.ts:673` `content: [{ type: "text", text: "Governance delete failed" }],`
- `sources/agentmemory/src/mcp/server.ts:674` `isError: true,`
- `sources/agentmemory/src/mcp/server.ts:675` `},`
- `sources/agentmemory/src/mcp/server.ts:676` `};`
- `sources/agentmemory/src/mcp/server.ts:677` `}`
- `sources/agentmemory/src/mcp/server.ts:1184` ``
- `sources/agentmemory/src/mcp/server.ts:1185` `case "memory_lesson_delete": {`
- `sources/agentmemory/src/mcp/server.ts:1186` `if (typeof args.lessonId !== "string" || !args.lessonId.trim()) {`
- `sources/agentmemory/src/mcp/server.ts:1187` `return { status_code: 400, body: { error: "lessonId is required" } };`
- `sources/agentmemory/src/mcp/server.ts:1188` `}`
- `sources/agentmemory/src/mcp/server.ts:1189` `const lessonDeleteResult = await sdk.trigger({ function_id: "mem::lesson-delete", payload: {`
- `sources/agentmemory/src/mcp/server.ts:1190` `lessonId: args.lessonId.trim(),`
- `sources/agentmemory/src/mcp/server.ts:1191` `} });`
- `sources/agentmemory/src/mcp/server.ts:1192` `return { status_code: 200, body: { content: [{ type: "text", text: JSON.stringify(lessonDeleteResult, null, 2) }] } };`
- `sources/agentmemory/src/mcp/server.ts:1193` `}`
- `sources/agentmemory/src/mcp/server.ts:1194` ``
- `sources/agentmemory/src/mcp/server.ts:1195` `case "memory_reflect": {`
- `sources/agentmemory/src/mcp/server.ts:1196` `const reflectResult = await sdk.trigger({ function_id: "mem::reflect", payload: {`
- `sources/agentmemory/src/mcp/server.ts:1282` ``
- `sources/agentmemory/src/mcp/server.ts:1283` `case "memory_slot_delete": {`
- `sources/agentmemory/src/mcp/server.ts:1284` `const label = asNonEmptyString(args.label);`
- `sources/agentmemory/src/mcp/server.ts:1285` `if (!label) return { status_code: 400, body: { error: "label required" } };`
- `sources/agentmemory/src/mcp/server.ts:1286` `const result = await sdk.trigger({ function_id: "mem::slot-delete", payload: { label } });`
- `sources/agentmemory/src/mcp/server.ts:1287` `return {`
- `sources/agentmemory/src/mcp/server.ts:1288` `status_code: 200,`
- `sources/agentmemory/src/mcp/server.ts:1289` `body: { content: [{ type: "text", text: JSON.stringify(result, null, 2) }] },`
- `sources/agentmemory/src/mcp/server.ts:1290` `};`

**operation and route call sites**, body `sources/agentmemory/src/mcp/standalone.ts:20`; body evidence.

- `sources/agentmemory/src/mcp/standalone.ts:19` `resolveHandle,`
- `sources/agentmemory/src/mcp/standalone.ts:20` `invalidateHandle,`
- `sources/agentmemory/src/mcp/standalone.ts:21` `isProxyRequired,`
- `sources/agentmemory/src/mcp/standalone.ts:22` `ProxyCallError,`
- `sources/agentmemory/src/mcp/standalone.ts:23` `type Handle,`
- `sources/agentmemory/src/mcp/standalone.ts:24` `type ProxyHandle,`
- `sources/agentmemory/src/mcp/standalone.ts:33` `"memory_audit",`
- `sources/agentmemory/src/mcp/standalone.ts:34` `"memory_governance_delete",`
- `sources/agentmemory/src/mcp/standalone.ts:35` `]);`
- `sources/agentmemory/src/mcp/standalone.ts:36` ``
- `sources/agentmemory/src/mcp/standalone.ts:37` `const SUPPORTED_PROTOCOL_VERSIONS = [`
- `sources/agentmemory/src/mcp/standalone.ts:38` `"2025-11-25",`
- `sources/agentmemory/src/mcp/standalone.ts:209` `}`
- `sources/agentmemory/src/mcp/standalone.ts:210` `case "memory_governance_delete": {`
- `sources/agentmemory/src/mcp/standalone.ts:211` `const ids = normalizeList(args["memoryIds"]);`
- `sources/agentmemory/src/mcp/standalone.ts:212` `if (ids.length === 0) throw new Error("memoryIds is required");`
- `sources/agentmemory/src/mcp/standalone.ts:213` `v.memoryIds = ids;`
- `sources/agentmemory/src/mcp/standalone.ts:214` `v.reason = (args["reason"] as string) || "plugin skill request";`
- `sources/agentmemory/src/mcp/standalone.ts:297` `}`
- `sources/agentmemory/src/mcp/standalone.ts:298` `case "memory_governance_delete": {`
- `sources/agentmemory/src/mcp/standalone.ts:299` `const result = await handle.call("/agentmemory/governance/memories", {`
- `sources/agentmemory/src/mcp/standalone.ts:300` `method: "DELETE",`
- `sources/agentmemory/src/mcp/standalone.ts:301` `body: JSON.stringify({ memoryIds: v.memoryIds, reason: v.reason }),`
- `sources/agentmemory/src/mcp/standalone.ts:302` `});`
- `sources/agentmemory/src/mcp/standalone.ts:303` `return textResponse(result);`
- `sources/agentmemory/src/mcp/standalone.ts:304` `}`
- `sources/agentmemory/src/mcp/standalone.ts:439` ``
- `sources/agentmemory/src/mcp/standalone.ts:440` `case "memory_governance_delete": {`
- `sources/agentmemory/src/mcp/standalone.ts:441` `let deleted = 0;`
- `sources/agentmemory/src/mcp/standalone.ts:442` `for (const id of v.memoryIds || []) {`
- `sources/agentmemory/src/mcp/standalone.ts:443` `const existing = await kvInstance.get("mem:memories", id);`
- `sources/agentmemory/src/mcp/standalone.ts:444` `if (existing) {`
- `sources/agentmemory/src/mcp/standalone.ts:445` `await kvInstance.delete("mem:memories", id);`
- `sources/agentmemory/src/mcp/standalone.ts:446` `deleted++;`
- `sources/agentmemory/src/mcp/standalone.ts:447` `}`
- `sources/agentmemory/src/mcp/standalone.ts:448` `}`
- `sources/agentmemory/src/mcp/standalone.ts:449` `const persisted = kvInstance.persist();`
- `sources/agentmemory/src/mcp/standalone.ts:450` `return textResponse({`
- `sources/agentmemory/src/mcp/standalone.ts:451` `deleted,`
- `sources/agentmemory/src/mcp/standalone.ts:452` `requested: (v.memoryIds || []).length,`
- `sources/agentmemory/src/mcp/standalone.ts:453` `reason: v.reason,`
- `sources/agentmemory/src/mcp/standalone.ts:454` `...localFallbackNotice(persisted),`
- `sources/agentmemory/src/mcp/standalone.ts:455` `});`
- `sources/agentmemory/src/mcp/standalone.ts:510` `);`
- `sources/agentmemory/src/mcp/standalone.ts:511` `invalidateHandle();`
- `sources/agentmemory/src/mcp/standalone.ts:512` `throw err;`
- `sources/agentmemory/src/mcp/standalone.ts:513` `}`
- `sources/agentmemory/src/mcp/standalone.ts:514` `}`
- `sources/agentmemory/src/mcp/standalone.ts:515` `throw new Error(`
- `sources/agentmemory/src/mcp/standalone.ts:525` `if (toolName === "memory_sessions" || isAuthenticationError(err) || isProxyRequired()) {`
- `sources/agentmemory/src/mcp/standalone.ts:526` `invalidateHandle();`
- `sources/agentmemory/src/mcp/standalone.ts:527` `throw err;`
- `sources/agentmemory/src/mcp/standalone.ts:528` `}`
- `sources/agentmemory/src/mcp/standalone.ts:529` `process.stderr.write(`
- `sources/agentmemory/src/mcp/standalone.ts:530` `'[@agentmemory/mcp] proxy call failed for ${toolName}: ${err instanceof Error ? err.message : String(err)}; invalidating handle and falling back to local KV\n',`
- `sources/agentmemory/src/mcp/standalone.ts:531` `);`
- `sources/agentmemory/src/mcp/standalone.ts:532` `invalidateHandle();`
- `sources/agentmemory/src/mcp/standalone.ts:533` `}`
- `sources/agentmemory/src/mcp/standalone.ts:534` `}`
- `sources/agentmemory/src/mcp/standalone.ts:535` `return handleLocal(validated, kvInstance);`
- `sources/agentmemory/src/mcp/standalone.ts:536` `}`
- `sources/agentmemory/src/mcp/standalone.ts:574` `} catch (err) {`
- `sources/agentmemory/src/mcp/standalone.ts:575` `invalidateHandle();`
- `sources/agentmemory/src/mcp/standalone.ts:576` `if (isAuthenticationError(err) || isProxyRequired()) throw err;`
- `sources/agentmemory/src/mcp/standalone.ts:577` `process.stderr.write(`
- `sources/agentmemory/src/mcp/standalone.ts:578` `'[@agentmemory/mcp] tools/list proxy failed: ${err instanceof Error ? err.message : String(err)}; falling back to local list\n',`
- `sources/agentmemory/src/mcp/standalone.ts:579` `);`

**operation and route call sites**, body `sources/agentmemory/src/mcp/tools-registry.ts:49`; body evidence.

- `sources/agentmemory/src/mcp/tools-registry.ts:48` `"memory_claude_bridge_sync",`
- `sources/agentmemory/src/mcp/tools-registry.ts:49` `"memory_governance_delete",`
- `sources/agentmemory/src/mcp/tools-registry.ts:50` `"memory_heal",`
- `sources/agentmemory/src/mcp/tools-registry.ts:51` `"memory_lesson_delete",`
- `sources/agentmemory/src/mcp/tools-registry.ts:52` `"memory_obsidian_export",`
- `sources/agentmemory/src/mcp/tools-registry.ts:53` `"memory_slot_replace",`
- `sources/agentmemory/src/mcp/tools-registry.ts:54` `"memory_slot_delete",`
- `sources/agentmemory/src/mcp/tools-registry.ts:55` `]);`
- `sources/agentmemory/src/mcp/tools-registry.ts:56` ``
- `sources/agentmemory/src/mcp/tools-registry.ts:57` `const IDEMPOTENT_WRITE_TOOLS = new Set([`
- `sources/agentmemory/src/mcp/tools-registry.ts:58` `"memory_governance_delete",`
- `sources/agentmemory/src/mcp/tools-registry.ts:59` `"memory_lesson_delete",`
- `sources/agentmemory/src/mcp/tools-registry.ts:60` `"memory_slot_replace",`
- `sources/agentmemory/src/mcp/tools-registry.ts:61` `"memory_slot_delete",`
- `sources/agentmemory/src/mcp/tools-registry.ts:62` `"memory_claude_bridge_sync",`
- `sources/agentmemory/src/mcp/tools-registry.ts:63` `]);`
- `sources/agentmemory/src/mcp/tools-registry.ts:64` ``
- `sources/agentmemory/src/mcp/tools-registry.ts:65` `const OPEN_WORLD_TOOLS = new Set(["memory_mesh_sync"]);`
- `sources/agentmemory/src/mcp/tools-registry.ts:457` `{`
- `sources/agentmemory/src/mcp/tools-registry.ts:458` `name: "memory_governance_delete",`
- `sources/agentmemory/src/mcp/tools-registry.ts:459` `description: "Delete specific memories with audit trail.",`
- `sources/agentmemory/src/mcp/tools-registry.ts:460` `inputSchema: {`
- `sources/agentmemory/src/mcp/tools-registry.ts:461` `type: "object",`
- `sources/agentmemory/src/mcp/tools-registry.ts:462` `properties: {`
- `sources/agentmemory/src/mcp/tools-registry.ts:463` `memoryIds: {`
- `sources/agentmemory/src/mcp/tools-registry.ts:464` `type: "string",`
- `sources/agentmemory/src/mcp/tools-registry.ts:465` `description: "Comma-separated memory IDs to delete",`
- `sources/agentmemory/src/mcp/tools-registry.ts:466` `},`
- `sources/agentmemory/src/mcp/tools-registry.ts:467` `reason: { type: "string", description: "Reason for deletion" },`
- `sources/agentmemory/src/mcp/tools-registry.ts:468` `},`
- `sources/agentmemory/src/mcp/tools-registry.ts:469` `required: ["memoryIds"],`
- `sources/agentmemory/src/mcp/tools-registry.ts:470` `},`
- `sources/agentmemory/src/mcp/tools-registry.ts:471` `},`
- `sources/agentmemory/src/mcp/tools-registry.ts:932` `{`
- `sources/agentmemory/src/mcp/tools-registry.ts:933` `name: "memory_lesson_delete",`
- `sources/agentmemory/src/mcp/tools-registry.ts:934` `description:`
- `sources/agentmemory/src/mcp/tools-registry.ts:935` `"Soft-delete a lesson by id. Deleted lessons are excluded from recall and list; re-saving the same content creates a fresh lesson.",`
- `sources/agentmemory/src/mcp/tools-registry.ts:936` `inputSchema: {`
- `sources/agentmemory/src/mcp/tools-registry.ts:937` `type: "object",`
- `sources/agentmemory/src/mcp/tools-registry.ts:938` `properties: {`
- `sources/agentmemory/src/mcp/tools-registry.ts:939` `lessonId: { type: "string", description: "The lesson id (lsn_...)" },`
- `sources/agentmemory/src/mcp/tools-registry.ts:1057` `{`
- `sources/agentmemory/src/mcp/tools-registry.ts:1058` `name: "memory_slot_delete",`
- `sources/agentmemory/src/mcp/tools-registry.ts:1059` `description: "Delete a slot. Seeded default slots can be deleted unless marked readOnly.",`
- `sources/agentmemory/src/mcp/tools-registry.ts:1060` `inputSchema: {`
- `sources/agentmemory/src/mcp/tools-registry.ts:1061` `type: "object",`
- `sources/agentmemory/src/mcp/tools-registry.ts:1062` `properties: {`
- `sources/agentmemory/src/mcp/tools-registry.ts:1063` `label: { type: "string", description: "Slot label" },`

**operation and route call sites**, body `sources/agentmemory/src/mcp/transport.ts:278`; body evidence.

- `sources/agentmemory/src/mcp/transport.ts:277` `if (activeStdioTransports === 0) {`
- `sources/agentmemory/src/mcp/transport.ts:278` `delete process.env["AGENTMEMORY_MCP_STDIO__"];`
- `sources/agentmemory/src/mcp/transport.ts:279` `}`
- `sources/agentmemory/src/mcp/transport.ts:280` `}`
- `sources/agentmemory/src/mcp/transport.ts:281` `parser = null;`
- `sources/agentmemory/src/mcp/transport.ts:282` `},`

**operation and route call sites**, body `sources/agentmemory/src/providers/agent-sdk.ts:84`; body evidence.

- `sources/agentmemory/src/providers/agent-sdk.ts:83` `if (sdkOriginalEnv === undefined) {`
- `sources/agentmemory/src/providers/agent-sdk.ts:84` `delete process.env.AGENTMEMORY_SDK_CHILD`
- `sources/agentmemory/src/providers/agent-sdk.ts:85` `} else {`
- `sources/agentmemory/src/providers/agent-sdk.ts:86` `process.env.AGENTMEMORY_SDK_CHILD = sdkOriginalEnv`
- `sources/agentmemory/src/providers/agent-sdk.ts:87` `}`
- `sources/agentmemory/src/providers/agent-sdk.ts:88` `sdkOriginalEnv = undefined`

**operation and route call sites**, body `sources/agentmemory/src/shutdown.ts:18`; body evidence.

- `sources/agentmemory/src/shutdown.ts:17` `return () => {`
- `sources/agentmemory/src/shutdown.ts:18` `if (startupGlobal[STARTUP_CLEANUP] === state) delete startupGlobal[STARTUP_CLEANUP];`
- `sources/agentmemory/src/shutdown.ts:19` `};`
- `sources/agentmemory/src/shutdown.ts:20` `}`
- `sources/agentmemory/src/shutdown.ts:21` ``
- `sources/agentmemory/src/shutdown.ts:22` `export async function runStartupCleanup(exitCode: number): Promise<number> {`

**operation and route call sites**, body `sources/agentmemory/src/state/graph-consistency.ts:27`; body evidence.

- `sources/agentmemory/src/state/graph-consistency.ts:26` `if (key === "current" && (scope === KV.graphSnapshot || scope.startsWith('${KV.graphSnapshot}:generation:'))) {`
- `sources/agentmemory/src/state/graph-consistency.ts:27` `checks.get(kv)?.delete(scope);`
- `sources/agentmemory/src/state/graph-consistency.ts:28` `}`
- `sources/agentmemory/src/state/graph-consistency.ts:29` `}`
- `sources/agentmemory/src/state/graph-consistency.ts:30` ``
- `sources/agentmemory/src/state/graph-consistency.ts:31` `function snapshotCheck(kv: StateKV, scope: string): SnapshotCheck {`
- `sources/agentmemory/src/state/graph-consistency.ts:68` `for (const write of pending.writes) {`
- `sources/agentmemory/src/state/graph-consistency.ts:69` `if (write.value === null) await kv.delete(write.scope + suffix, write.key);`
- `sources/agentmemory/src/state/graph-consistency.ts:70` `else await kv.set(write.scope + suffix, write.key, write.value);`
- `sources/agentmemory/src/state/graph-consistency.ts:71` `}`
- `sources/agentmemory/src/state/graph-consistency.ts:72` `await kv.set(snapshot, "current", pending.snapshot);`
- `sources/agentmemory/src/state/graph-consistency.ts:73` `}`

**operation and route call sites**, body `sources/agentmemory/src/state/graph-generation.ts:89`; body evidence.

- `sources/agentmemory/src/state/graph-generation.ts:88` `function graphView(kv: StateKV, id: string | null): StateKV {`
- `sources/agentmemory/src/state/graph-generation.ts:89` `const methods = new Set(["get", "set", "update", "delete", "list", "listKeys", "getMany", "iterate", "listPage"]);`
- `sources/agentmemory/src/state/graph-generation.ts:90` `const view = new Proxy(kv, {`
- `sources/agentmemory/src/state/graph-generation.ts:91` `get(target, property, receiver) {`
- `sources/agentmemory/src/state/graph-generation.ts:92` `const value = Reflect.get(target, property, receiver);`
- `sources/agentmemory/src/state/graph-generation.ts:93` `if (typeof property !== "string" || typeof value !== "function" || !methods.has(property)) return value;`

**operation and route call sites**, body `sources/agentmemory/src/state/graph-ingestion.ts:49`; body evidence.

- `sources/agentmemory/src/state/graph-ingestion.ts:48` `if (next) Object.defineProperty(counts, type, { value: next, writable: true, enumerable: true, configurable: true });`
- `sources/agentmemory/src/state/graph-ingestion.ts:49` `else delete counts[type];`
- `sources/agentmemory/src/state/graph-ingestion.ts:50` `}`
- `sources/agentmemory/src/state/graph-ingestion.ts:51` ``
- `sources/agentmemory/src/state/graph-ingestion.ts:52` `export async function persistGraphRecords(`
- `sources/agentmemory/src/state/graph-ingestion.ts:53` `state: StateKV,`
- `sources/agentmemory/src/state/graph-ingestion.ts:105` `if (canonical) ids.add(canonical);`
- `sources/agentmemory/src/state/graph-ingestion.ts:106` `if (add) ids.add(id); else ids.delete(id);`
- `sources/agentmemory/src/state/graph-ingestion.ts:107` `const members = [...ids];`
- `sources/agentmemory/src/state/graph-ingestion.ts:108` `set(KV.graphMembership, memberKey, members.length ? { ids: members } : null);`
- `sources/agentmemory/src/state/graph-ingestion.ts:109` `set(scope, key, members.at(-1) ?? null);`
- `sources/agentmemory/src/state/graph-ingestion.ts:110` `};`
- `sources/agentmemory/src/state/graph-ingestion.ts:115` `const ids = new Set(saved?.ids ?? []);`
- `sources/agentmemory/src/state/graph-ingestion.ts:116` `if (add) ids.add(edge.id); else ids.delete(edge.id);`
- `sources/agentmemory/src/state/graph-ingestion.ts:117` `set(KV.graphMembership, key, ids.size ? { ids: [...ids] } : null);`
- `sources/agentmemory/src/state/graph-ingestion.ts:118` `}`
- `sources/agentmemory/src/state/graph-ingestion.ts:119` `};`
- `sources/agentmemory/src/state/graph-ingestion.ts:120` `for (const item of records.slice(offset, offset + BATCH_SIZE)) {`
- `sources/agentmemory/src/state/graph-ingestion.ts:155` `if (item.edge) {`
- `sources/agentmemory/src/state/graph-ingestion.ts:156` `topEdges.delete(row.id);`
- `sources/agentmemory/src/state/graph-ingestion.ts:157` `if (old) await updateAdjacency(old as GraphEdge, false);`
- `sources/agentmemory/src/state/graph-ingestion.ts:158` `if (newLive) await updateAdjacency(row as GraphEdge, true);`
- `sources/agentmemory/src/state/graph-ingestion.ts:159` `if (oldLive) {`
- `sources/agentmemory/src/state/graph-ingestion.ts:160` `await adjustDegree((old as GraphEdge).sourceNodeId, -1);`
- `sources/agentmemory/src/state/graph-ingestion.ts:169` `} else {`
- `sources/agentmemory/src/state/graph-ingestion.ts:170` `candidates.delete(row.id);`
- `sources/agentmemory/src/state/graph-ingestion.ts:171` `if (newLive) candidates.set(row.id, row as GraphNode);`
- `sources/agentmemory/src/state/graph-ingestion.ts:172` `else if (next.stats.totalNodes > TOP_N) next.sampleIncomplete = true;`
- `sources/agentmemory/src/state/graph-ingestion.ts:173` `const value = await degree(row.id);`
- `sources/agentmemory/src/state/graph-ingestion.ts:174` `set(KV.graphNodeDegree, row.id, value);`
- `sources/agentmemory/src/state/graph-ingestion.ts:211` `for (const write of writes.values()) {`
- `sources/agentmemory/src/state/graph-ingestion.ts:212` `if (write.value === null) await target.delete(write.scope, write.key);`
- `sources/agentmemory/src/state/graph-ingestion.ts:213` `else await target.set(write.scope, write.key, write.value);`
- `sources/agentmemory/src/state/graph-ingestion.ts:214` `}`
- `sources/agentmemory/src/state/graph-ingestion.ts:215` `await target.set(KV.graphSnapshot, "current", next);`
- `sources/agentmemory/src/state/graph-ingestion.ts:216` `return next;`

**operation and route call sites**, body `sources/agentmemory/src/state/graph-retrieval-index.ts:139`; body evidence.

- `sources/agentmemory/src/state/graph-retrieval-index.ts:138` `incomplete = true;`
- `sources/agentmemory/src/state/graph-retrieval-index.ts:139` `if ("properties" in row && Object.keys(row.properties).length) delete row.properties[Object.keys(row.properties).at(-1)!];`
- `sources/agentmemory/src/state/graph-retrieval-index.ts:140` `else if ("aliases" in row && row.aliases?.length) row.aliases.pop();`
- `sources/agentmemory/src/state/graph-retrieval-index.ts:141` `else if ("context" in row && row.context) delete row.context;`
- `sources/agentmemory/src/state/graph-retrieval-index.ts:142` `else if (row.sourceObservationIds.length) row.sourceObservationIds.pop();`
- `sources/agentmemory/src/state/graph-retrieval-index.ts:143` `else return { row: null, postings: [], incomplete: true };`
- `sources/agentmemory/src/state/graph-retrieval-index.ts:144` `}`
- `sources/agentmemory/src/state/graph-retrieval-index.ts:145` `const postings = new Set<string>([token(countKey(kind), "")]);`
- `sources/agentmemory/src/state/graph-retrieval-index.ts:232` `next[field][type] = (next[field][type] ?? 0) + amount;`
- `sources/agentmemory/src/state/graph-retrieval-index.ts:233` `if (!next[field][type]) delete next[field][type];`
- `sources/agentmemory/src/state/graph-retrieval-index.ts:234` `if (type === "other") next.otherTypes += amount;`
- `sources/agentmemory/src/state/graph-retrieval-index.ts:235` `}`
- `sources/agentmemory/src/state/graph-retrieval-index.ts:236` `}`
- `sources/agentmemory/src/state/graph-retrieval-index.ts:237` `validateControl(next);`
- `sources/agentmemory/src/state/graph-retrieval-index.ts:253` `for (const write of journal.writes) {`
- `sources/agentmemory/src/state/graph-retrieval-index.ts:254` `if (write.value === null) await kv.delete(KV.graphRetrievalData, write.key);`
- `sources/agentmemory/src/state/graph-retrieval-index.ts:255` `else await kv.set(KV.graphRetrievalData, write.key, write.value);`
- `sources/agentmemory/src/state/graph-retrieval-index.ts:256` `}`
- `sources/agentmemory/src/state/graph-retrieval-index.ts:257` `await kv.set(KV.graphRetrievalControl, CONTROL_KEY, journal.next);`
- `sources/agentmemory/src/state/graph-retrieval-index.ts:258` `}`
- `sources/agentmemory/src/state/graph-retrieval-index.ts:316` `if (next === undefined) return false;`
- `sources/agentmemory/src/state/graph-retrieval-index.ts:317` `if (next === null) await withoutGraphRecovery(() => kv.delete(scope, id));`
- `sources/agentmemory/src/state/graph-retrieval-index.ts:318` `else await withoutGraphRecovery(() => kv.set(scope, id, next));`
- `sources/agentmemory/src/state/graph-retrieval-index.ts:319` `return true;`
- `sources/agentmemory/src/state/graph-retrieval-index.ts:320` `}));`
- `sources/agentmemory/src/state/graph-retrieval-index.ts:321` `}`

**operation and route call sites**, body `sources/agentmemory/src/state/index-persistence.ts:452`; body evidence.

- `sources/agentmemory/src/state/index-persistence.ts:451` `if (this.recoveryError || this.backfillRevision !== revision || !isCovered()) return;`
- `sources/agentmemory/src/state/index-persistence.ts:452` `await this.kv.delete(KV.bm25Index, BACKFILL_MARKER_KEY);`
- `sources/agentmemory/src/state/index-persistence.ts:453` `cleared = true;`
- `sources/agentmemory/src/state/index-persistence.ts:454` `});`
- `sources/agentmemory/src/state/index-persistence.ts:455` `if (!cleared) return false;`
- `sources/agentmemory/src/state/index-persistence.ts:456` `if (this.backfillRevision !== revision || !isCovered()) {`
- `sources/agentmemory/src/state/index-persistence.ts:472` `return this.queueBackfillChange(async () => {`
- `sources/agentmemory/src/state/index-persistence.ts:473` `if (this.vector) await this.kv.delete(KV.bm25Index, BACKFILL_MARKER_KEY);`
- `sources/agentmemory/src/state/index-persistence.ts:474` `});`
- `sources/agentmemory/src/state/index-persistence.ts:475` `}`
- `sources/agentmemory/src/state/index-persistence.ts:476` ``
- `sources/agentmemory/src/state/index-persistence.ts:477` `async flushPendingLog(): Promise<void> {`
- `sources/agentmemory/src/state/index-persistence.ts:504` `this.pendingLog.set(key, q);`
- `sources/agentmemory/src/state/index-persistence.ts:505` `this.acknowledgedLog.delete(key);`
- `sources/agentmemory/src/state/index-persistence.ts:506` `this.enqueueLog(key, async () => {`
- `sources/agentmemory/src/state/index-persistence.ts:507` `await this.kv.set<PendingVectorRow>(KV.vectorPendingLog, key, row);`
- `sources/agentmemory/src/state/index-persistence.ts:508` `if (this.pendingLog.get(key) === q) this.acknowledgedLog.set(key, q);`
- `sources/agentmemory/src/state/index-persistence.ts:509` `}).then(`
- `sources/agentmemory/src/state/index-persistence.ts:528` `void tail.then(() => {`
- `sources/agentmemory/src/state/index-persistence.ts:529` `if (this.logChains.get(key) === tail) this.logChains.delete(key);`
- `sources/agentmemory/src/state/index-persistence.ts:530` `});`
- `sources/agentmemory/src/state/index-persistence.ts:531` `return run;`
- `sources/agentmemory/src/state/index-persistence.ts:532` `}`
- `sources/agentmemory/src/state/index-persistence.ts:533` ``
- `sources/agentmemory/src/state/index-persistence.ts:544` ``
- `sources/agentmemory/src/state/index-persistence.ts:545` `private deletePendingKey(key: string, q: number | undefined): Promise<void> {`
- `sources/agentmemory/src/state/index-persistence.ts:546` `return this.enqueueLog(key, async () => {`
- `sources/agentmemory/src/state/index-persistence.ts:547` `if (this.pendingLog.get(key) !== q) return;`
- `sources/agentmemory/src/state/index-persistence.ts:548` `await this.kv.delete(KV.vectorPendingLog, key);`
- `sources/agentmemory/src/state/index-persistence.ts:549` `if (this.pendingLog.get(key) === q) {`
- `sources/agentmemory/src/state/index-persistence.ts:550` `this.pendingLog.delete(key);`
- `sources/agentmemory/src/state/index-persistence.ts:551` `this.acknowledgedLog.delete(key);`
- `sources/agentmemory/src/state/index-persistence.ts:552` `}`
- `sources/agentmemory/src/state/index-persistence.ts:553` `});`
- `sources/agentmemory/src/state/index-persistence.ts:554` `}`
- `sources/agentmemory/src/state/index-persistence.ts:555` ``
- `sources/agentmemory/src/state/index-persistence.ts:558` `if (clearSeq !== undefined ? clearSeq <= coveredSeq : this.pendingLogUnknown) {`
- `sources/agentmemory/src/state/index-persistence.ts:559` `await this.deletePendingKey(PENDING_CLEAR_KEY, clearSeq);`
- `sources/agentmemory/src/state/index-persistence.ts:560` `}`
- `sources/agentmemory/src/state/index-persistence.ts:561` `const covered = [...this.pendingLog].filter(([key, q]) => key !== PENDING_CLEAR_KEY && q <= coveredSeq);`
- `sources/agentmemory/src/state/index-persistence.ts:562` `const failures = await inBatches(covered, WRITE_CONCURRENCY, ([key, q]) => this.deletePendingKey(key, q));`
- `sources/agentmemory/src/state/index-persistence.ts:563` `if (failures.length > 0) {`
- `sources/agentmemory/src/state/index-persistence.ts:564` `throw new Error('${failures.length} of ${covered.length} pending vector log deletes failed: ${errorMessage(failures[0])}');`
- `sources/agentmemory/src/state/index-persistence.ts:565` `}`
- `sources/agentmemory/src/state/index-persistence.ts:566` `this.pendingLogUnknown = false;`
- `sources/agentmemory/src/state/index-persistence.ts:567` `}`
- `sources/agentmemory/src/state/index-persistence.ts:568` ``
- `sources/agentmemory/src/state/index-persistence.ts:631` `if (bucket === undefined) return;`
- `sources/agentmemory/src/state/index-persistence.ts:632` `this.bucketOfId.delete(id);`
- `sources/agentmemory/src/state/index-persistence.ts:633` `const remaining = (this.bucketCounts.get(bucket) ?? 1) - 1;`
- `sources/agentmemory/src/state/index-persistence.ts:634` `if (remaining > 0) this.bucketCounts.set(bucket, remaining);`
- `sources/agentmemory/src/state/index-persistence.ts:635` `else this.bucketCounts.delete(bucket);`
- `sources/agentmemory/src/state/index-persistence.ts:636` `}`
- `sources/agentmemory/src/state/index-persistence.ts:637` ``
- `sources/agentmemory/src/state/index-persistence.ts:638` `private clearTimer(): void {`
- `sources/agentmemory/src/state/index-persistence.ts:639` `if (this.timer) {`
- `sources/agentmemory/src/state/index-persistence.ts:722` `if (bucket !== undefined) {`
- `sources/agentmemory/src/state/index-persistence.ts:723` `await this.kv.delete(vectorBucketScope(bucket), id);`
- `sources/agentmemory/src/state/index-persistence.ts:724` `this.releaseBucket(id);`
- `sources/agentmemory/src/state/index-persistence.ts:725` `}`
- `sources/agentmemory/src/state/index-persistence.ts:726` `}`
- `sources/agentmemory/src/state/index-persistence.ts:727` `} catch (err) {`
- `sources/agentmemory/src/state/index-persistence.ts:844` `if (manifest && Array.isArray(manifest.shards)) {`
- `sources/agentmemory/src/state/index-persistence.ts:845` `await this.deleteShards(manifest.shards.filter(isValidShardDescriptor), "legacy_vector_cleanup");`
- `sources/agentmemory/src/state/index-persistence.ts:846` `}`
- `sources/agentmemory/src/state/index-persistence.ts:847` `await this.deleteKey(KV.bm25Index, LEGACY_VECTOR_MANIFEST_KEY, "legacy_vector_cleanup");`
- `sources/agentmemory/src/state/index-persistence.ts:848` `await this.deleteKey(KV.bm25Index, LEGACY_VECTOR_KEY, "legacy_vector_cleanup");`
- `sources/agentmemory/src/state/index-persistence.ts:849` `}`
- `sources/agentmemory/src/state/index-persistence.ts:850` ``
- `sources/agentmemory/src/state/index-persistence.ts:851` `private async removeLegacyVectorSnapshotIfPresent(): Promise<void> {`
- `sources/agentmemory/src/state/index-persistence.ts:852` `const manifest = await this.readIndexValue<IndexShardManifest>(KV.bm25Index, LEGACY_VECTOR_MANIFEST_KEY, "vector", "manifest");`
- `sources/agentmemory/src/state/index-persistence.ts:866` `if (manifest && Array.isArray(manifest.shards)) {`
- `sources/agentmemory/src/state/index-persistence.ts:867` `await this.deleteShards(manifest.shards.filter(isValidShardDescriptor), "legacy_bm25_cleanup");`
- `sources/agentmemory/src/state/index-persistence.ts:868` `}`
- `sources/agentmemory/src/state/index-persistence.ts:869` `if (manifest != null) await this.deleteKey(KV.bm25Index, LEGACY_BM25_MANIFEST_KEY, "legacy_bm25_cleanup");`
- `sources/agentmemory/src/state/index-persistence.ts:870` `if (legacy != null) await this.deleteKey(KV.bm25Index, LEGACY_BM25_KEY, "legacy_bm25_cleanup");`
- `sources/agentmemory/src/state/index-persistence.ts:871` `}`
- `sources/agentmemory/src/state/index-persistence.ts:872` ``
- `sources/agentmemory/src/state/index-persistence.ts:873` `private async finishLegacyCleanup(): Promise<void> {`
- `sources/agentmemory/src/state/index-persistence.ts:874` `await this.removeLegacyBm25Snapshot();`
- `sources/agentmemory/src/state/index-persistence.ts:894` `const failures = await inBatches(orphans, WRITE_CONCURRENCY, async ([scope]) => {`
- `sources/agentmemory/src/state/index-persistence.ts:895` `if (!(await this.deleteKey(scope, LEGACY_SHARD_KEY, "orphan_generation_sweep"))) throw new Error(scope);`
- `sources/agentmemory/src/state/index-persistence.ts:896` `});`
- `sources/agentmemory/src/state/index-persistence.ts:897` `const summary = {`
- `sources/agentmemory/src/state/index-persistence.ts:898` `generations: new Set(orphans.map(([, generation]) => generation)).size,`
- `sources/agentmemory/src/state/index-persistence.ts:899` `shards: orphans.length,`
- `sources/agentmemory/src/state/index-persistence.ts:967` ``
- `sources/agentmemory/src/state/index-persistence.ts:968` `private async deleteKey(scope: string, key: string, reason: string): Promise<boolean> {`
- `sources/agentmemory/src/state/index-persistence.ts:969` `let result = "deleted";`
- `sources/agentmemory/src/state/index-persistence.ts:970` `let error: string | undefined;`
- `sources/agentmemory/src/state/index-persistence.ts:971` `try {`
- `sources/agentmemory/src/state/index-persistence.ts:972` `await this.kv.delete(scope, key);`
- `sources/agentmemory/src/state/index-persistence.ts:973` `} catch (err) {`
- `sources/agentmemory/src/state/index-persistence.ts:974` `result = "failed";`
- `sources/agentmemory/src/state/index-persistence.ts:975` `error = errorMessage(err);`
- `sources/agentmemory/src/state/index-persistence.ts:976` `}`
- `sources/agentmemory/src/state/index-persistence.ts:977` `await this.auditIndexPersistence("delete", [statePath(scope, key)], {`
- `sources/agentmemory/src/state/index-persistence.ts:978` `scope,`
- `sources/agentmemory/src/state/index-persistence.ts:979` `key,`
- `sources/agentmemory/src/state/index-persistence.ts:980` `reason,`
- `sources/agentmemory/src/state/index-persistence.ts:981` `result,`
- `sources/agentmemory/src/state/index-persistence.ts:986` ``
- `sources/agentmemory/src/state/index-persistence.ts:987` `private async deleteShards(shards: IndexShardManifest["shards"], reason: string): Promise<void> {`
- `sources/agentmemory/src/state/index-persistence.ts:988` `await inBatches(shards, WRITE_CONCURRENCY, async (shard) => {`
- `sources/agentmemory/src/state/index-persistence.ts:989` `await this.deleteKey(shard.scope, shard.key, reason);`
- `sources/agentmemory/src/state/index-persistence.ts:990` `});`
- `sources/agentmemory/src/state/index-persistence.ts:991` `}`
- `sources/agentmemory/src/state/index-persistence.ts:992` ``
- `sources/agentmemory/src/state/index-persistence.ts:993` `private async loadShardedData(legacyKey: string, manifestKey: string, label: string): Promise<string | null> {`

**operation and route call sites**, body `sources/agentmemory/src/state/keyed-mutex.ts:40`; body evidence.

- `sources/agentmemory/src/state/keyed-mutex.ts:39` `cleanup.then(() => {`
- `sources/agentmemory/src/state/keyed-mutex.ts:40` `if (locks.get(key) === cleanup) locks.delete(key);`
- `sources/agentmemory/src/state/keyed-mutex.ts:41` `});`
- `sources/agentmemory/src/state/keyed-mutex.ts:42` `const flush = async () => {`
- `sources/agentmemory/src/state/keyed-mutex.ts:43` `await cleanup;`
- `sources/agentmemory/src/state/keyed-mutex.ts:44` `const errors: unknown[] = [];`

**operation and route call sites**, body `sources/agentmemory/src/state/kv.ts:46`; body evidence.

- `sources/agentmemory/src/state/kv.ts:45` `if (path === '') current = null`
- `sources/agentmemory/src/state/kv.ts:46` `else if (isPlainObject(current)) delete current[path]`
- `sources/agentmemory/src/state/kv.ts:47` `} else if (op.type === 'merge' && isPlainObject(op.value)) {`
- `sources/agentmemory/src/state/kv.ts:48` `if (path === '') {`
- `sources/agentmemory/src/state/kv.ts:49` `if (isPlainObject(current)) Object.assign(current, op.value)`
- `sources/agentmemory/src/state/kv.ts:50` `} else {`
- `sources/agentmemory/src/state/kv.ts:190` ``
- `sources/agentmemory/src/state/kv.ts:191` `async delete(scope: string, key: string): Promise<void> {`
- `sources/agentmemory/src/state/kv.ts:192` `await this.mutate(scope, key, async (resolvedScope) => {`
- `sources/agentmemory/src/state/kv.ts:193` `await this.sdk.trigger<{ scope: string; key: string }, void>({`
- `sources/agentmemory/src/state/kv.ts:194` `function_id: 'state::delete',`
- `sources/agentmemory/src/state/kv.ts:195` `payload: { scope: resolvedScope, key },`
- `sources/agentmemory/src/state/kv.ts:196` `})`
- `sources/agentmemory/src/state/kv.ts:197` `})`
- `sources/agentmemory/src/state/kv.ts:198` `}`

**operation and route call sites**, body `sources/agentmemory/src/state/obs-index.ts:26`; body evidence.

- `sources/agentmemory/src/state/obs-index.ts:25` `): Promise<void> {`
- `sources/agentmemory/src/state/obs-index.ts:26` `await kv.delete(KV.obsSessionIndex(shardFor(obsId)), obsId);`
- `sources/agentmemory/src/state/obs-index.ts:27` `}`
- `sources/agentmemory/src/state/obs-index.ts:28` ``
- `sources/agentmemory/src/state/obs-index.ts:29` `export async function lookupObservationSession(`
- `sources/agentmemory/src/state/obs-index.ts:30` `kv: StateKV,`

**operation and route call sites**, body `sources/agentmemory/src/state/search-index.ts:37`; body evidence.

- `sources/agentmemory/src/state/search-index.ts:36` `add(obs: CompressedObservation, layer: SearchResultLayer = "observation"): void {`
- `sources/agentmemory/src/state/search-index.ts:37` `this.invalidateAbsentRevision(obs.id);`
- `sources/agentmemory/src/state/search-index.ts:38` `if (this.entries.has(obs.id)) this.remove(obs.id);`
- `sources/agentmemory/src/state/search-index.ts:39` `const terms = this.extractTerms(obs);`
- `sources/agentmemory/src/state/search-index.ts:40` `const termFreq = new Map<string, number>();`
- `sources/agentmemory/src/state/search-index.ts:41` `let termCount = 0;`
- `sources/agentmemory/src/state/search-index.ts:93` `slot.readers--;`
- `sources/agentmemory/src/state/search-index.ts:94` `if (slot.readers === 0 && this.absentRevisions.get(id) === slot) this.absentRevisions.delete(id);`
- `sources/agentmemory/src/state/search-index.ts:95` `},`
- `sources/agentmemory/src/state/search-index.ts:96` `};`
- `sources/agentmemory/src/state/search-index.ts:97` `}`
- `sources/agentmemory/src/state/search-index.ts:98` ``
- `sources/agentmemory/src/state/search-index.ts:99` `private invalidateAbsentRevision(id: string): void {`
- `sources/agentmemory/src/state/search-index.ts:100` `const slot = this.absentRevisions.get(id);`
- `sources/agentmemory/src/state/search-index.ts:101` `if (slot) slot.token = {};`
- `sources/agentmemory/src/state/search-index.ts:102` `}`
- `sources/agentmemory/src/state/search-index.ts:103` ``
- `sources/agentmemory/src/state/search-index.ts:104` `invalidateRevision(id: string): void {`
- `sources/agentmemory/src/state/search-index.ts:105` `this.invalidateAbsentRevision(id);`
- `sources/agentmemory/src/state/search-index.ts:106` `const entry = this.entries.get(id);`
- `sources/agentmemory/src/state/search-index.ts:107` `if (entry) this.entries.set(id, { ...entry });`
- `sources/agentmemory/src/state/search-index.ts:108` `}`
- `sources/agentmemory/src/state/search-index.ts:109` ``
- `sources/agentmemory/src/state/search-index.ts:152` `remove(id: string): void {`
- `sources/agentmemory/src/state/search-index.ts:153` `this.invalidateAbsentRevision(id);`
- `sources/agentmemory/src/state/search-index.ts:154` `const entry = this.entries.get(id);`
- `sources/agentmemory/src/state/search-index.ts:155` `if (!entry) return;`
- `sources/agentmemory/src/state/search-index.ts:156` ``
- `sources/agentmemory/src/state/search-index.ts:157` `const termFreq = this.docTermCounts.get(id);`
- `sources/agentmemory/src/state/search-index.ts:161` `if (postingList) {`
- `sources/agentmemory/src/state/search-index.ts:162` `postingList.delete(id);`
- `sources/agentmemory/src/state/search-index.ts:163` `if (postingList.size === 0) {`
- `sources/agentmemory/src/state/search-index.ts:164` `this.invertedIndex.delete(term);`
- `sources/agentmemory/src/state/search-index.ts:165` `}`
- `sources/agentmemory/src/state/search-index.ts:166` `}`
- `sources/agentmemory/src/state/search-index.ts:167` `}`
- `sources/agentmemory/src/state/search-index.ts:168` `this.docTermCounts.delete(id);`
- `sources/agentmemory/src/state/search-index.ts:169` `}`
- `sources/agentmemory/src/state/search-index.ts:170` ``
- `sources/agentmemory/src/state/search-index.ts:171` `this.totalDocLength = Math.max(0, this.totalDocLength - entry.termCount);`
- `sources/agentmemory/src/state/search-index.ts:172` `this.entries.delete(id);`
- `sources/agentmemory/src/state/search-index.ts:173` `this.sortedTerms = null;`
- `sources/agentmemory/src/state/search-index.ts:174` `}`
- `sources/agentmemory/src/state/search-index.ts:175` ``
- `sources/agentmemory/src/state/search-index.ts:176` `search(`

**operation and route call sites**, body `sources/agentmemory/src/state/session-index.ts:101`; body evidence.

- `sources/agentmemory/src/state/session-index.ts:100` `recordRebuildChange(project, (changes) => {`
- `sources/agentmemory/src/state/session-index.ts:101` `changes.removed.delete(entry.id);`
- `sources/agentmemory/src/state/session-index.ts:102` `changes.added.set(entry.id, entry);`
- `sources/agentmemory/src/state/session-index.ts:103` `});`
- `sources/agentmemory/src/state/session-index.ts:104` `try {`
- `sources/agentmemory/src/state/session-index.ts:105` `await kv.set(KV.projectSessionsIndex, project, capWithAgentFairness(merged));`
- `sources/agentmemory/src/state/session-index.ts:106` `} catch (err) {`
- `sources/agentmemory/src/state/session-index.ts:107` `await kv.delete(KV.projectSessionsIndex, project).catch(() => {});`
- `sources/agentmemory/src/state/session-index.ts:108` `throw err;`
- `sources/agentmemory/src/state/session-index.ts:109` `}`
- `sources/agentmemory/src/state/session-index.ts:110` `});`
- `sources/agentmemory/src/state/session-index.ts:111` `}`
- `sources/agentmemory/src/state/session-index.ts:119` `recordRebuildChange(project, (changes) => {`
- `sources/agentmemory/src/state/session-index.ts:120` `changes.added.delete(sessionId);`
- `sources/agentmemory/src/state/session-index.ts:121` `changes.removed.add(sessionId);`
- `sources/agentmemory/src/state/session-index.ts:122` `});`
- `sources/agentmemory/src/state/session-index.ts:123` `const existing = await getProjectSessionIndex(kv, project);`
- `sources/agentmemory/src/state/session-index.ts:124` `if (!existing) return;`
- `sources/agentmemory/src/state/session-index.ts:189` `} finally {`
- `sources/agentmemory/src/state/session-index.ts:190` `activeRebuilds.delete(changesSinceListing);`
- `sources/agentmemory/src/state/session-index.ts:191` `}`
- `sources/agentmemory/src/state/session-index.ts:192` `}`
- `sources/agentmemory/src/state/session-index.ts:193` ``
- `sources/agentmemory/src/state/session-index.ts:194` `const SESSION_INDEX_GENERATION_KEY = "session-index-generation";`

**operation and route call sites**, body `sources/agentmemory/src/state/vector-index.ts:53`; body evidence.

- `sources/agentmemory/src/state/vector-index.ts:52` `remove(obsId: string): void {`
- `sources/agentmemory/src/state/vector-index.ts:53` `if (this.vectors.delete(obsId)) {`
- `sources/agentmemory/src/state/vector-index.ts:54` `this.changes.set(obsId, false);`
- `sources/agentmemory/src/state/vector-index.ts:55` `this.listener?.(obsId, null);`
- `sources/agentmemory/src/state/vector-index.ts:56` `}`
- `sources/agentmemory/src/state/vector-index.ts:57` `}`

**operation and route call sites**, body `sources/agentmemory/src/state/viewer-counts.ts:44`; body evidence.

- `sources/agentmemory/src/state/viewer-counts.ts:43` ``
- `sources/agentmemory/src/state/viewer-counts.ts:44` `export function invalidateViewerCounts(): void {`
- `sources/agentmemory/src/state/viewer-counts.ts:45` `counts = null;`
- `sources/agentmemory/src/state/viewer-counts.ts:46` `if (loading) changedWhileLoading = true;`
- `sources/agentmemory/src/state/viewer-counts.ts:47` `}`
- `sources/agentmemory/src/state/viewer-counts.ts:48` ``
- `sources/agentmemory/src/state/viewer-counts.ts:54` `function isLiveLesson(lesson: Lesson | null | undefined): boolean {`
- `sources/agentmemory/src/state/viewer-counts.ts:55` `return Boolean(lesson) && lesson!.deleted !== true;`
- `sources/agentmemory/src/state/viewer-counts.ts:56` `}`
- `sources/agentmemory/src/state/viewer-counts.ts:57` ``
- `sources/agentmemory/src/state/viewer-counts.ts:58` `async function listOrEmpty<T>(kv: StateKV, scope: string): Promise<T[]> {`
- `sources/agentmemory/src/state/viewer-counts.ts:59` `const rows = await kv.list<T>(scope).catch(() => [] as T[]);`

**operation and route call sites**, body `sources/agentmemory/src/telemetry/setup.ts:33`; body evidence.

- `sources/agentmemory/src/telemetry/setup.ts:32` `vectorSearchTotal: Counter;`
- `sources/agentmemory/src/telemetry/setup.ts:33` `autoForgetTotal: Counter;`
- `sources/agentmemory/src/telemetry/setup.ts:34` `profileGenerated: Counter;`
- `sources/agentmemory/src/telemetry/setup.ts:35` `claudeBridgeSync: Counter;`
- `sources/agentmemory/src/telemetry/setup.ts:36` `graphExtraction: Counter;`
- `sources/agentmemory/src/telemetry/setup.ts:37` `consolidationRun: Counter;`
- `sources/agentmemory/src/telemetry/setup.ts:40` `snapshotCreate: Counter;`
- `sources/agentmemory/src/telemetry/setup.ts:41` `governanceDelete: Counter;`
- `sources/agentmemory/src/telemetry/setup.ts:42` `smartSearchFollowupWithinWindow: Counter;`
- `sources/agentmemory/src/telemetry/setup.ts:43` `readerFailureWithEvidence: Counter;`
- `sources/agentmemory/src/telemetry/setup.ts:44` `}`
- `sources/agentmemory/src/telemetry/setup.ts:45` ``
- `sources/agentmemory/src/telemetry/setup.ts:76` `["vectorSearchTotal", "vector_search.total"],`
- `sources/agentmemory/src/telemetry/setup.ts:77` `["autoForgetTotal", "auto_forget.total"],`
- `sources/agentmemory/src/telemetry/setup.ts:78` `["profileGenerated", "profile.generated"],`
- `sources/agentmemory/src/telemetry/setup.ts:79` `["claudeBridgeSync", "claude_bridge.sync"],`
- `sources/agentmemory/src/telemetry/setup.ts:80` `["graphExtraction", "graph.extraction"],`
- `sources/agentmemory/src/telemetry/setup.ts:81` `["consolidationRun", "consolidation.run"],`
- `sources/agentmemory/src/telemetry/setup.ts:84` `["snapshotCreate", "snapshot.create"],`
- `sources/agentmemory/src/telemetry/setup.ts:85` `["governanceDelete", "governance.delete"],`
- `sources/agentmemory/src/telemetry/setup.ts:86` `["smartSearchFollowupWithinWindow", "smart_search.followup_within_window_total"],`
- `sources/agentmemory/src/telemetry/setup.ts:87` `["readerFailureWithEvidence", "reader_failure_with_evidence_total"],`
- `sources/agentmemory/src/telemetry/setup.ts:88` `];`
- `sources/agentmemory/src/telemetry/setup.ts:89` ``

**operation and route call sites**, body `sources/agentmemory/src/triggers/api.ts:1743`; body evidence.

- `sources/agentmemory/src/triggers/api.ts:1742` ``
- `sources/agentmemory/src/triggers/api.ts:1743` `sdk.registerFunction("api::forget",`
- `sources/agentmemory/src/triggers/api.ts:1744` `async (`
- `sources/agentmemory/src/triggers/api.ts:1745` `req: HttpRequest<{`
- `sources/agentmemory/src/triggers/api.ts:1746` `sessionId?: string;`
- `sources/agentmemory/src/triggers/api.ts:1747` `observationIds?: string[];`
- `sources/agentmemory/src/triggers/api.ts:1758` `}`
- `sources/agentmemory/src/triggers/api.ts:1759` `const result = await sdk.trigger({ function_id: "mem::forget", payload: req.body });`
- `sources/agentmemory/src/triggers/api.ts:1760` `return { status_code: 200, body: result };`
- `sources/agentmemory/src/triggers/api.ts:1761` `},`
- `sources/agentmemory/src/triggers/api.ts:1762` `);`
- `sources/agentmemory/src/triggers/api.ts:1763` `sdk.registerTrigger({`
- `sources/agentmemory/src/triggers/api.ts:1764` `type: "http",`
- `sources/agentmemory/src/triggers/api.ts:1765` `function_id: "api::forget",`
- `sources/agentmemory/src/triggers/api.ts:1766` `config: { api_path: "/agentmemory/forget", http_method: "POST" },`
- `sources/agentmemory/src/triggers/api.ts:1767` `});`
- `sources/agentmemory/src/triggers/api.ts:1768` ``
- `sources/agentmemory/src/triggers/api.ts:1769` `sdk.registerFunction("api::consolidate",`
- `sources/agentmemory/src/triggers/api.ts:1770` `async (`
- `sources/agentmemory/src/triggers/api.ts:2152` ``
- `sources/agentmemory/src/triggers/api.ts:2153` `sdk.registerFunction("api::auto-forget",`
- `sources/agentmemory/src/triggers/api.ts:2154` `async (req: HttpRequest<{ dryRun?: boolean }>): Promise<Response> => {`
- `sources/agentmemory/src/triggers/api.ts:2155` `const authErr = checkAuth(req, secret);`
- `sources/agentmemory/src/triggers/api.ts:2156` `if (authErr) return authErr;`
- `sources/agentmemory/src/triggers/api.ts:2157` `const dryRun =`
- `sources/agentmemory/src/triggers/api.ts:2158` `req.query_params?.["dryRun"] === "true" || req.body?.dryRun === true;`
- `sources/agentmemory/src/triggers/api.ts:2159` `const result = await sdk.trigger({ function_id: "mem::auto-forget", payload: { dryRun } });`
- `sources/agentmemory/src/triggers/api.ts:2160` `return { status_code: 200, body: result };`
- `sources/agentmemory/src/triggers/api.ts:2161` `},`
- `sources/agentmemory/src/triggers/api.ts:2162` `);`
- `sources/agentmemory/src/triggers/api.ts:2163` `sdk.registerTrigger({`
- `sources/agentmemory/src/triggers/api.ts:2164` `type: "http",`
- `sources/agentmemory/src/triggers/api.ts:2165` `function_id: "api::auto-forget",`
- `sources/agentmemory/src/triggers/api.ts:2166` `config: { api_path: "/agentmemory/auto-forget", http_method: "POST" },`
- `sources/agentmemory/src/triggers/api.ts:2167` `});`
- `sources/agentmemory/src/triggers/api.ts:2168` ``
- `sources/agentmemory/src/triggers/api.ts:2169` `sdk.registerFunction("api::claude-bridge-read",`
- `sources/agentmemory/src/triggers/api.ts:2170` `async (req: HttpRequest): Promise<Response> => {`
- `sources/agentmemory/src/triggers/api.ts:2628` ``
- `sources/agentmemory/src/triggers/api.ts:2629` `sdk.registerFunction("api::governance-delete",`
- `sources/agentmemory/src/triggers/api.ts:2630` `async (`
- `sources/agentmemory/src/triggers/api.ts:2631` `req: HttpRequest<{ memoryIds: string[]; reason?: string }>,`
- `sources/agentmemory/src/triggers/api.ts:2632` `): Promise<Response> => {`
- `sources/agentmemory/src/triggers/api.ts:2633` `const authErr = checkAuth(req, secret);`
- `sources/agentmemory/src/triggers/api.ts:2640` `}`
- `sources/agentmemory/src/triggers/api.ts:2641` `const result = await sdk.trigger({ function_id: "mem::governance-delete", payload: req.body });`
- `sources/agentmemory/src/triggers/api.ts:2642` `return { status_code: 200, body: result };`
- `sources/agentmemory/src/triggers/api.ts:2643` `},`
- `sources/agentmemory/src/triggers/api.ts:2644` `);`
- `sources/agentmemory/src/triggers/api.ts:2645` `sdk.registerTrigger({`
- `sources/agentmemory/src/triggers/api.ts:2646` `type: "http",`
- `sources/agentmemory/src/triggers/api.ts:2647` `function_id: "api::governance-delete",`
- `sources/agentmemory/src/triggers/api.ts:2648` `config: {`
- `sources/agentmemory/src/triggers/api.ts:2649` `api_path: "/agentmemory/governance/memories",`
- `sources/agentmemory/src/triggers/api.ts:2650` `http_method: "DELETE",`
- `sources/agentmemory/src/triggers/api.ts:2651` `},`
- `sources/agentmemory/src/triggers/api.ts:2652` `});`
- `sources/agentmemory/src/triggers/api.ts:2653` ``
- `sources/agentmemory/src/triggers/api.ts:2654` `sdk.registerFunction("api::governance-bulk",`
- `sources/agentmemory/src/triggers/api.ts:2674` `config: {`
- `sources/agentmemory/src/triggers/api.ts:2675` `api_path: "/agentmemory/governance/bulk-delete",`
- `sources/agentmemory/src/triggers/api.ts:2676` `http_method: "POST",`
- `sources/agentmemory/src/triggers/api.ts:2677` `},`
- `sources/agentmemory/src/triggers/api.ts:2678` `});`
- `sources/agentmemory/src/triggers/api.ts:2679` ``
- `sources/agentmemory/src/triggers/api.ts:3170` ``
- `sources/agentmemory/src/triggers/api.ts:3171` `sdk.registerFunction("api::slot-delete", async (req: HttpRequest): Promise<Response> => {`
- `sources/agentmemory/src/triggers/api.ts:3172` `const authErr = checkAuth(req, secret);`
- `sources/agentmemory/src/triggers/api.ts:3173` `if (authErr) return authErr;`
- `sources/agentmemory/src/triggers/api.ts:3174` `if (!isSlotsEnabled()) return slotsDisabledResponse();`
- `sources/agentmemory/src/triggers/api.ts:3175` `const label = asNonEmptyString(req.query_params?.["label"]);`
- `sources/agentmemory/src/triggers/api.ts:3176` `if (!label) return { status_code: 400, body: { error: "label query param required" } };`
- `sources/agentmemory/src/triggers/api.ts:3177` `const result = await sdk.trigger({ function_id: "mem::slot-delete", payload: { label } });`
- `sources/agentmemory/src/triggers/api.ts:3178` `const resp = result as { success?: boolean; error?: string };`
- `sources/agentmemory/src/triggers/api.ts:3179` `if (resp?.success === false) {`
- `sources/agentmemory/src/triggers/api.ts:3180` `return { status_code: resp.error?.includes("not found") ? 404 : 400, body: resp };`
- `sources/agentmemory/src/triggers/api.ts:3181` `}`
- `sources/agentmemory/src/triggers/api.ts:3185` `type: "http",`
- `sources/agentmemory/src/triggers/api.ts:3186` `function_id: "api::slot-delete",`
- `sources/agentmemory/src/triggers/api.ts:3187` `config: { api_path: "/agentmemory/slot", http_method: "DELETE" },`
- `sources/agentmemory/src/triggers/api.ts:3188` `});`
- `sources/agentmemory/src/triggers/api.ts:3189` ``
- `sources/agentmemory/src/triggers/api.ts:3190` `sdk.registerFunction("api::slot-reflect", async (req: HttpRequest): Promise<Response> => {`
- `sources/agentmemory/src/triggers/api.ts:3191` `const authErr = checkAuth(req, secret);`
- `sources/agentmemory/src/triggers/api.ts:4228` `const lesson = await kv.get<Lesson>(KV.lessons, lessonId);`
- `sources/agentmemory/src/triggers/api.ts:4229` `if (!lesson || lesson.deleted) return { status_code: 404, body: { error: "lesson not found" } };`
- `sources/agentmemory/src/triggers/api.ts:4230` `if (lesson.project !== undefined && lesson.project !== project) {`
- `sources/agentmemory/src/triggers/api.ts:4231` `return { status_code: 404, body: { error: "lesson not found" } };`
- `sources/agentmemory/src/triggers/api.ts:4232` `}`
- `sources/agentmemory/src/triggers/api.ts:4233` `return { status_code: 200, body: { success: true, lesson } };`
- `sources/agentmemory/src/triggers/api.ts:4246` ``
- `sources/agentmemory/src/triggers/api.ts:4247` `sdk.registerFunction("api::lesson-delete",  async (req: HttpRequest) => {`
- `sources/agentmemory/src/triggers/api.ts:4248` `const denied = checkAuth(req, secret);`
- `sources/agentmemory/src/triggers/api.ts:4249` `if (denied) return denied;`
- `sources/agentmemory/src/triggers/api.ts:4250` `const body = req.body as Record<string, unknown>;`
- `sources/agentmemory/src/triggers/api.ts:4251` `const lessonId = typeof body?.lessonId === "string" ? body.lessonId.trim() : "";`
- `sources/agentmemory/src/triggers/api.ts:4252` `if (!lessonId) return { status_code: 400, body: { error: "lessonId is required" } };`
- `sources/agentmemory/src/triggers/api.ts:4253` `const result = await sdk.trigger({ function_id: "mem::lesson-delete", payload: { lessonId } });`
- `sources/agentmemory/src/triggers/api.ts:4254` `const resp = result as { success?: boolean; error?: string };`
- `sources/agentmemory/src/triggers/api.ts:4255` `if (resp?.success === false && resp.error === "lesson not found") {`
- `sources/agentmemory/src/triggers/api.ts:4256` `return { status_code: 404, body: { error: "lesson not found" } };`
- `sources/agentmemory/src/triggers/api.ts:4257` `}`
- `sources/agentmemory/src/triggers/api.ts:4259` `});`
- `sources/agentmemory/src/triggers/api.ts:4260` `sdk.registerTrigger({ type: "http", function_id: "api::lesson-delete", config: { api_path: "/agentmemory/lessons/delete", http_method: "POST" } });`
- `sources/agentmemory/src/triggers/api.ts:4261` ``
- `sources/agentmemory/src/triggers/api.ts:4262` `sdk.registerFunction("api::obsidian-export", async (req: HttpRequest) => {`
- `sources/agentmemory/src/triggers/api.ts:4263` `const denied = checkAuth(req, secret);`
- `sources/agentmemory/src/triggers/api.ts:4264` `if (denied) return denied;`

**operation and route call sites**, body `sources/agentmemory/src/triggers/events.ts:119`; body evidence.

- `sources/agentmemory/src/triggers/events.ts:118` `}) => {`
- `sources/agentmemory/src/triggers/events.ts:119` `noteSessionChange(payload.old_value, isStateDelete(payload) ? null : payload.new_value);`
- `sources/agentmemory/src/triggers/events.ts:120` `if (isOutOfAgentScope(payload.new_value ?? payload.old_value)) {`
- `sources/agentmemory/src/triggers/events.ts:121` `return { emitted: false };`
- `sources/agentmemory/src/triggers/events.ts:122` `}`
- `sources/agentmemory/src/triggers/events.ts:123` `if (isStateDelete(payload)) {`
- `sources/agentmemory/src/triggers/events.ts:124` `await sendViewerEvent(sdk, 'session-deleted-${payload.key}-${Date.now()}', "session.deleted", {`
- `sources/agentmemory/src/triggers/events.ts:125` `sessionId: payload.key,`
- `sources/agentmemory/src/triggers/events.ts:126` `});`
- `sources/agentmemory/src/triggers/events.ts:127` `return { emitted: true };`
- `sources/agentmemory/src/triggers/events.ts:128` `}`
- `sources/agentmemory/src/triggers/events.ts:161` `}) => {`
- `sources/agentmemory/src/triggers/events.ts:162` `const deleted = isStateDelete(payload);`
- `sources/agentmemory/src/triggers/events.ts:163` `noteMemoryChange(payload.old_value, deleted ? null : payload.new_value);`
- `sources/agentmemory/src/triggers/events.ts:164` `if (isOutOfAgentScope(payload.new_value ?? payload.old_value)) {`
- `sources/agentmemory/src/triggers/events.ts:165` `return { emitted: false };`
- `sources/agentmemory/src/triggers/events.ts:166` `}`
- `sources/agentmemory/src/triggers/events.ts:167` `const memory = payload.new_value;`
- `sources/agentmemory/src/triggers/events.ts:168` `const created = !deleted && !payload.old_value;`
- `sources/agentmemory/src/triggers/events.ts:169` `let type = "memory.updated";`
- `sources/agentmemory/src/triggers/events.ts:170` `if (deleted) type = "memory.deleted";`
- `sources/agentmemory/src/triggers/events.ts:171` `else if (created) type = "memory.created";`
- `sources/agentmemory/src/triggers/events.ts:172` `await sendViewerEvent(`
- `sources/agentmemory/src/triggers/events.ts:173` `sdk,`
- `sources/agentmemory/src/triggers/events.ts:174` `'${type.replace(".", "-")}-${payload.key}-${Date.now()}',`
- `sources/agentmemory/src/triggers/events.ts:175` `type,`
- `sources/agentmemory/src/triggers/events.ts:176` `deleted`
- `sources/agentmemory/src/triggers/events.ts:177` `? { memoryId: payload.key, isLatest: payload.old_value?.isLatest }`
- `sources/agentmemory/src/triggers/events.ts:178` `: {`
- `sources/agentmemory/src/triggers/events.ts:179` `memoryId: payload.key,`
- `sources/agentmemory/src/triggers/events.ts:180` `type: memory?.type,`
- `sources/agentmemory/src/triggers/events.ts:196` ``
- `sources/agentmemory/src/triggers/events.ts:197` `export function isStateDelete(payload: { event_type: string; new_value?: unknown }): boolean {`
- `sources/agentmemory/src/triggers/events.ts:198` `return payload.event_type === "state:deleted" || !payload.new_value;`
- `sources/agentmemory/src/triggers/events.ts:199` `}`
- `sources/agentmemory/src/triggers/events.ts:200` ``
- `sources/agentmemory/src/triggers/events.ts:201` `export function isOutOfAgentScope(record: { agentId?: string } | undefined): boolean {`
- `sources/agentmemory/src/triggers/events.ts:202` `return isAgentScopeIsolated() && record?.agentId !== getAgentId();`

**operation and route call sites**, body `sources/agentmemory/src/triggers/viewer-streams.ts:31`; body evidence.

- `sources/agentmemory/src/triggers/viewer-streams.ts:30` `getViewerCounts,`
- `sources/agentmemory/src/triggers/viewer-streams.ts:31` `invalidateViewerCounts,`
- `sources/agentmemory/src/triggers/viewer-streams.ts:32` `noteGraphStats,`
- `sources/agentmemory/src/triggers/viewer-streams.ts:33` `readViewerGraphMetadata,`
- `sources/agentmemory/src/triggers/viewer-streams.ts:34` `noteRowChange,`
- `sources/agentmemory/src/triggers/viewer-streams.ts:35` `setViewerCountsEmitter,`
- `sources/agentmemory/src/triggers/viewer-streams.ts:42` `import { buildConfigFlags, checkAuth, createConsolidationStatusReader, createStatusReporter } from "./api.js";`
- `sources/agentmemory/src/triggers/viewer-streams.ts:43` `import { isStateDelete, sendViewerEvent } from "./events.js";`
- `sources/agentmemory/src/triggers/viewer-streams.ts:44` `import { setAuditRecordedListener } from "../functions/audit.js";`
- `sources/agentmemory/src/triggers/viewer-streams.ts:45` ``
- `sources/agentmemory/src/triggers/viewer-streams.ts:46` `type Response = {`
- `sources/agentmemory/src/triggers/viewer-streams.ts:47` `status_code: number;`
- `sources/agentmemory/src/triggers/viewer-streams.ts:123` `sdk.registerFunction(options.functionId, async (payload: StatePayload<T>) => {`
- `sources/agentmemory/src/triggers/viewer-streams.ts:124` `const deleted = isStateDelete(payload);`
- `sources/agentmemory/src/triggers/viewer-streams.ts:125` `const row = deleted ? null : payload.new_value ?? null;`
- `sources/agentmemory/src/triggers/viewer-streams.ts:126` `if (options.countKind) noteRowChange(options.countKind, payload.old_value, row);`
- `sources/agentmemory/src/triggers/viewer-streams.ts:127` `let type = '${options.entity}.updated';`
- `sources/agentmemory/src/triggers/viewer-streams.ts:128` `if (deleted) type = '${options.entity}.deleted';`
- `sources/agentmemory/src/triggers/viewer-streams.ts:129` `else if (!payload.old_value) type = '${options.entity}.created';`
- `sources/agentmemory/src/triggers/viewer-streams.ts:130` `await sendViewerEvent(sdk, eventId(options.entity, payload.key), type, {`
- `sources/agentmemory/src/triggers/viewer-streams.ts:131` `id: payload.key,`
- `sources/agentmemory/src/triggers/viewer-streams.ts:132` `[options.entity]: row,`
- `sources/agentmemory/src/triggers/viewer-streams.ts:159` `} catch (error) {`
- `sources/agentmemory/src/triggers/viewer-streams.ts:160` `invalidateViewerCounts();`
- `sources/agentmemory/src/triggers/viewer-streams.ts:161` `lastGraphSnapshotEventAt = Number.NEGATIVE_INFINITY;`
- `sources/agentmemory/src/triggers/viewer-streams.ts:162` `await sendViewerEvent(sdk, eventId("graph", "invalidation"), "graph.changed", {`
- `sources/agentmemory/src/triggers/viewer-streams.ts:163` `source: "items", coveredBySnapshot: false, ...(reset ? { reset: true } : {}),`
- `sources/agentmemory/src/triggers/viewer-streams.ts:164` `updatedAt: new Date().toISOString(),`
- `sources/agentmemory/src/triggers/viewer-streams.ts:165` `}).catch(() => {});`
- `sources/agentmemory/src/triggers/viewer-streams.ts:166` `throw error;`
- `sources/agentmemory/src/triggers/viewer-streams.ts:185` `if (!await isLegacyGraphState(kv)) return { emitted: false };`
- `sources/agentmemory/src/triggers/viewer-streams.ts:186` `const snapshot = isStateDelete(payload) ? null : payload.new_value ?? null;`
- `sources/agentmemory/src/triggers/viewer-streams.ts:187` `if (snapshot?.pendingMutation) return { emitted: false };`
- `sources/agentmemory/src/triggers/viewer-streams.ts:188` `await publishGraphMetadata();`
- `sources/agentmemory/src/triggers/viewer-streams.ts:189` `return { emitted: true };`
- `sources/agentmemory/src/triggers/viewer-streams.ts:190` `}),`
- `sources/agentmemory/src/triggers/viewer-streams.ts:250` `async (payload: StatePayload<ConsolidationRunRecord>) => {`
- `sources/agentmemory/src/triggers/viewer-streams.ts:251` `if (payload.key !== CONSOLIDATION_LAST_RUN_KEY || isStateDelete(payload)) {`
- `sources/agentmemory/src/triggers/viewer-streams.ts:252` `return { emitted: false };`
- `sources/agentmemory/src/triggers/viewer-streams.ts:253` `}`
- `sources/agentmemory/src/triggers/viewer-streams.ts:254` `const lastRun = payload.new_value ?? null;`
- `sources/agentmemory/src/triggers/viewer-streams.ts:255` `await sendViewerEvent(sdk, eventId("consolidation", "run"), "consolidation.run", {`
- `sources/agentmemory/src/triggers/viewer-streams.ts:270` `async (payload: StatePayload<HealthSnapshot>) => {`
- `sources/agentmemory/src/triggers/viewer-streams.ts:271` `if (payload.key !== HEALTH_LATEST_KEY || isStateDelete(payload)) {`
- `sources/agentmemory/src/triggers/viewer-streams.ts:272` `return { emitted: false };`
- `sources/agentmemory/src/triggers/viewer-streams.ts:273` `}`
- `sources/agentmemory/src/triggers/viewer-streams.ts:274` `await sendViewerEvent(`
- `sources/agentmemory/src/triggers/viewer-streams.ts:275` `sdk,`

**operation and route call sites**, body `sources/agentmemory/src/types.ts:156`; body evidence.

- `sources/agentmemory/src/types.ts:155` `isLatest: boolean;`
- `sources/agentmemory/src/types.ts:156` `forgetAfter?: string;`
- `sources/agentmemory/src/types.ts:157` `imageRef?: string;`
- `sources/agentmemory/src/types.ts:158` `imageData?: string;`
- `sources/agentmemory/src/types.ts:159` `agentId?: string;`
- `sources/agentmemory/src/types.ts:160` `project?: string;`
- `sources/agentmemory/src/types.ts:695` `| "remember"`
- `sources/agentmemory/src/types.ts:696` `| "forget"`
- `sources/agentmemory/src/types.ts:697` `| "evolve"`
- `sources/agentmemory/src/types.ts:698` `| "consolidate"`
- `sources/agentmemory/src/types.ts:699` `| "share"`
- `sources/agentmemory/src/types.ts:700` `| "delete"`
- `sources/agentmemory/src/types.ts:701` `| "import"`
- `sources/agentmemory/src/types.ts:702` `| "export"`
- `sources/agentmemory/src/types.ts:703` `| "action_create"`
- `sources/agentmemory/src/types.ts:704` `| "action_update"`
- `sources/agentmemory/src/types.ts:726` `| "lesson_strengthen"`
- `sources/agentmemory/src/types.ts:727` `| "lesson_delete"`
- `sources/agentmemory/src/types.ts:728` `| "obsidian_export"`
- `sources/agentmemory/src/types.ts:729` `| "reflect"`
- `sources/agentmemory/src/types.ts:730` `| "insight_search"`
- `sources/agentmemory/src/types.ts:731` `| "skill_extract"`
- `sources/agentmemory/src/types.ts:738` `| "slot_create"`
- `sources/agentmemory/src/types.ts:739` `| "slot_delete"`
- `sources/agentmemory/src/types.ts:740` `| "slot_reflect"`
- `sources/agentmemory/src/types.ts:741` `| "graph_compact"`
- `sources/agentmemory/src/types.ts:742` `| "graph_index_migrate"`
- `sources/agentmemory/src/types.ts:743` `| "audit_migrate"`
- `sources/agentmemory/src/types.ts:760` `migrated: number;`
- `sources/agentmemory/src/types.ts:761` `purged: number;`
- `sources/agentmemory/src/types.ts:762` `summaryWritten: boolean;`
- `sources/agentmemory/src/types.ts:763` `checkedAt: string;`
- `sources/agentmemory/src/types.ts:764` `}`
- `sources/agentmemory/src/types.ts:765` ``
- `sources/agentmemory/src/types.ts:968` `decayRate: number;`
- `sources/agentmemory/src/types.ts:969` `deleted?: boolean;`
- `sources/agentmemory/src/types.ts:970` `}`
- `sources/agentmemory/src/types.ts:971` ``
- `sources/agentmemory/src/types.ts:972` `export interface Insight {`
- `sources/agentmemory/src/types.ts:973` `id: string;`
- `sources/agentmemory/src/types.ts:989` `decayRate: number;`
- `sources/agentmemory/src/types.ts:990` `deleted?: boolean;`
- `sources/agentmemory/src/types.ts:991` `}`
- `sources/agentmemory/src/types.ts:992` ``
- `sources/agentmemory/src/types.ts:993` `export interface DiagnosticCheck {`
- `sources/agentmemory/src/types.ts:994` `name: string;`

**operation and route call sites**, body `sources/agentmemory/src/utils/image-store.ts:63`; body evidence.

- `sources/agentmemory/src/utils/image-store.ts:62` ``
- `sources/agentmemory/src/utils/image-store.ts:63` `export async function deleteImage(filePath: string | undefined): Promise<{ deletedBytes: number }> {`
- `sources/agentmemory/src/utils/image-store.ts:64` `if (!filePath) return { deletedBytes: 0 };`
- `sources/agentmemory/src/utils/image-store.ts:65` `if (!isManagedImagePath(filePath)) return { deletedBytes: 0 };`
- `sources/agentmemory/src/utils/image-store.ts:66` `try {`
- `sources/agentmemory/src/utils/image-store.ts:67` `if (existsSync(filePath)) {`
- `sources/agentmemory/src/utils/image-store.ts:68` `const s = await stat(filePath);`
- `sources/agentmemory/src/utils/image-store.ts:69` `const size = s.size;`
- `sources/agentmemory/src/utils/image-store.ts:70` `await unlink(filePath);`
- `sources/agentmemory/src/utils/image-store.ts:71` `return { deletedBytes: size };`
- `sources/agentmemory/src/utils/image-store.ts:72` `}`
- `sources/agentmemory/src/utils/image-store.ts:73` `} catch (err) {`
- `sources/agentmemory/src/utils/image-store.ts:74` `console.error("[agentmemory] Failed to delete image context:", err);`
- `sources/agentmemory/src/utils/image-store.ts:75` `}`
- `sources/agentmemory/src/utils/image-store.ts:76` `return { deletedBytes: 0 };`
- `sources/agentmemory/src/utils/image-store.ts:77` `}`
- `sources/agentmemory/src/utils/image-store.ts:78` ``
- `sources/agentmemory/src/utils/image-store.ts:79` `/** Touch an image file to update its mtime (marking it as recently used for LRU eviction) */`
- `sources/agentmemory/src/utils/image-store.ts:80` `export async function touchImage(filePath: string): Promise<void> {`

**operation and route call sites**, body `sources/agentmemory/src/viewer/live-events.ts:98`; body evidence.

- `sources/agentmemory/src/viewer/live-events.ts:97` `private remove(writer: LiveEventWriter, dropped: boolean): void {`
- `sources/agentmemory/src/viewer/live-events.ts:98` `if (!this.writers.delete(writer)) return;`
- `sources/agentmemory/src/viewer/live-events.ts:99` `const onClose = this.detach.get(writer);`
- `sources/agentmemory/src/viewer/live-events.ts:100` `this.detach.delete(writer);`
- `sources/agentmemory/src/viewer/live-events.ts:101` `if (onClose) {`
- `sources/agentmemory/src/viewer/live-events.ts:102` `writer.off("close", onClose);`
- `sources/agentmemory/src/viewer/live-events.ts:103` `writer.off("error", onClose);`
- `sources/agentmemory/src/viewer/live-events.ts:104` `}`
- `sources/agentmemory/src/viewer/live-events.ts:122` `eventListeners.add(listener);`
- `sources/agentmemory/src/viewer/live-events.ts:123` `return () => eventListeners.delete(listener);`
- `sources/agentmemory/src/viewer/live-events.ts:124` `}`
- `sources/agentmemory/src/viewer/live-events.ts:125` `export function getLiveEventsStats(): LiveEventsStats {`
- `sources/agentmemory/src/viewer/live-events.ts:126` `return liveEvents.stats();`
- `sources/agentmemory/src/viewer/live-events.ts:127` `}`

**operation and route call sites**, body `sources/agentmemory/src/viewer/server.ts:151`; body evidence.

- `sources/agentmemory/src/viewer/server.ts:150` `"Access-Control-Allow-Origin": allowed,`
- `sources/agentmemory/src/viewer/server.ts:151` `"Access-Control-Allow-Methods": "GET, POST, DELETE, OPTIONS",`
- `sources/agentmemory/src/viewer/server.ts:152` `"Access-Control-Allow-Headers": "Content-Type, Authorization",`
- `sources/agentmemory/src/viewer/server.ts:153` `Vary: "Origin",`
- `sources/agentmemory/src/viewer/server.ts:154` `};`
- `sources/agentmemory/src/viewer/server.ts:155` `}`
- `sources/agentmemory/src/viewer/server.ts:477` `let body: string | undefined;`
- `sources/agentmemory/src/viewer/server.ts:478` `if (method === "POST" || method === "PUT" || method === "DELETE" || method === "PATCH") {`
- `sources/agentmemory/src/viewer/server.ts:479` `body = await readBody(req);`
- `sources/agentmemory/src/viewer/server.ts:480` `}`
- `sources/agentmemory/src/viewer/server.ts:481` ``
- `sources/agentmemory/src/viewer/server.ts:482` `const controller = new AbortController();`

</details>

## 3. What reaches the agent

System instructions describe recall/save and scoping (`sources/agentmemory/integrations/hermes/__init__.py:390`). Prefetch smart-search requests five results, reads narratives, clips titles to 200 and narratives to 1000, and returns at most 6000 characters (`sources/agentmemory/integrations/hermes/__init__.py:402`; `sources/agentmemory/integrations/hermes/__init__.py:429`). Tools return service JSON; recall/search limit validated 1..100 (`sources/agentmemory/integrations/hermes/__init__.py:521`).

Host assembly: `hermes-agent/agent/memory_manager.py:352` joins provider system blocks. `hermes-agent/agent/turn_context.py:1081` requests prefetch for nontrivial turns; `hermes-agent/agent/turn_context.py:120` fences it; `hermes-agent/agent/turn_context.py:136` appends it to API-bound user content. Tool results are a third channel. Optional host spill is at `hermes-agent/agent/memory_manager.py:419`, not a universal provider cap. Built-in memory remains separately controlled (`hermes-agent/agent/system_prompt.py:542`).

<details>
<summary>Context and tool-result assembly quotes</summary>

`system_prompt_block`, body `sources/agentmemory/integrations/hermes/__init__.py:390`:

- `sources/agentmemory/integrations/hermes/__init__.py:390` `return "Use memory_recall to retrieve relevant saved context and memory_save to retain durable facts. Memory is scoped to this Hermes profile and author by default."`

`prefetch`, body `sources/agentmemory/integrations/hermes/__init__.py:393`:

- `sources/agentmemory/integrations/hermes/__init__.py:396` `if cached and cached[:3] == (session_id, self._agent_id, query):`
- `sources/agentmemory/integrations/hermes/__init__.py:399` `return cached[3]`
- `sources/agentmemory/integrations/hermes/__init__.py:401` `return ""`
- `sources/agentmemory/integrations/hermes/__init__.py:402` `result = self._request("smart-search", {"query": query, "limit": 5, "project": self._project, "agentId": self._agent_id, "includeLessons": False})`
- `sources/agentmemory/integrations/hermes/__init__.py:405` `return ""`
- `sources/agentmemory/integrations/hermes/__init__.py:407` `for item in rows[:5]:`
- `sources/agentmemory/integrations/hermes/__init__.py:414` `ids.append(reference)`
- `sources/agentmemory/integrations/hermes/__init__.py:419` `return ""`
- `sources/agentmemory/integrations/hermes/__init__.py:421` `for item in rows[:5]:`
- `sources/agentmemory/integrations/hermes/__init__.py:429` `lines.append(f"- {title[:200]}: {narrative[:1000]}")`
- `sources/agentmemory/integrations/hermes/__init__.py:430` `return "\n".join(lines)[:6000]`

`queue_prefetch`, body `sources/agentmemory/integrations/hermes/__init__.py:433`:



`get_tool_schemas`, body `sources/agentmemory/integrations/hermes/__init__.py:438`:

- `sources/agentmemory/integrations/hermes/__init__.py:438` `return [`
- `sources/agentmemory/integrations/hermes/__init__.py:449` `"limit": {"type": "integer", "description": "Max results", "default": 10},`
- `sources/agentmemory/integrations/hermes/__init__.py:484` `"limit": {"type": "integer", "default": 5},`

`handle_tool_call`, body `sources/agentmemory/integrations/hermes/__init__.py:492`:

- `sources/agentmemory/integrations/hermes/__init__.py:493` `return json.dumps({"success": False, "error": f"Unknown tool: {name}"})`
- `sources/agentmemory/integrations/hermes/__init__.py:495` `return json.dumps({"success": False, "error": "Tool arguments must be an object."})`
- `sources/agentmemory/integrations/hermes/__init__.py:498` `return json.dumps({"success": False, "error": f"{key} must be a non-empty string."})`
- `sources/agentmemory/integrations/hermes/__init__.py:503` `return json.dumps({"success": False, "error": f"{field} must be a non-empty string."})`
- `sources/agentmemory/integrations/hermes/__init__.py:506` `return json.dumps({"success": False, "error": "agentId must be at most 128 characters and cannot be a wildcard."})`
- `sources/agentmemory/integrations/hermes/__init__.py:510` `return json.dumps({"success": False, "error": "Invalid memory type."})`
- `sources/agentmemory/integrations/hermes/__init__.py:517` `return json.dumps({"success": False, "error": f"{field} must be CSV text or an array of strings."})`
- `sources/agentmemory/integrations/hermes/__init__.py:521` `limit = args.get("limit", 10 if name == "memory_recall" else 5)`
- `sources/agentmemory/integrations/hermes/__init__.py:522` `if type(limit) is not int or not 1 <= limit <= 100:`
- `sources/agentmemory/integrations/hermes/__init__.py:523` `return json.dumps({"success": False, "error": "limit must be an integer between 1 and 100."})`
- `sources/agentmemory/integrations/hermes/__init__.py:524` `payload["limit"] = limit`
- `sources/agentmemory/integrations/hermes/__init__.py:527` `return json.dumps({"success": False, "error": "Invalid targetLayer."})`
- `sources/agentmemory/integrations/hermes/__init__.py:531` `return json.dumps(self._request(path, payload))`

`get_tool_schemas`, body `sources/agentmemory/integrations/hermes/__init__.py:80`:



`handle_tool_call`, body `sources/agentmemory/integrations/hermes/__init__.py:82`:



`system_prompt_block`, body `sources/agentmemory/integrations/hermes/__init__.py:85`:

- `sources/agentmemory/integrations/hermes/__init__.py:85` `def system_prompt_block(self) -> str: return ""`

`prefetch`, body `sources/agentmemory/integrations/hermes/__init__.py:86`:

- `sources/agentmemory/integrations/hermes/__init__.py:86` `def prefetch(self, query: str, **kwargs: Any) -> str: return ""`

`queue_prefetch`, body `sources/agentmemory/integrations/hermes/__init__.py:87`:



</details>

## 4. Data access boundaries

Direct access `yes`, via HTTP or iii functions with required service credentials; plugin tool absence does not remove the engine function. AGENTMEMORY_URL, AGENTMEMORY_SECRET, AGENTMEMORY_PROJECT_NAME and AGENTMEMORY_AGENT_ID are configured separately (`sources/agentmemory/integrations/hermes/plugin.yaml:7`). Local-service accountless mode `yes` with compatible local model configuration; service secret is optional, not a hosted-account requirement. Storage/child-process/env/telemetry implementation sites are below.

Boundary model: same OS user, terminal, mounted filesystem and allowed network. Direct reachability means another client can use the same file/endpoint and available credentials, not unauthenticated access or encryption bypass. Hermes setup persists declared secrets through `hermes-agent/hermes_cli/memory_setup.py:321` and `hermes-agent/hermes_cli/config.py:2697`. These are possible configured paths; no live credentials were read.

Host credential-file detail: `get_hermes_home()/.env` is resolved at `hermes-agent/hermes_cli/config.py:473`. New files are secured to 0600 at `hermes-agent/hermes_cli/config.py:506`; the writer preserves an existing mode at `hermes-agent/hermes_cli/config.py:2544` and `:2562`. Therefore 0600 is not an unconditional mode claim for every existing profile.

<details>
<summary>Paths, environment, permissions, egress, telemetry and process-launch evidence</summary>

- `sources/agentmemory/integrations/hermes/__init__.py:36` `out = subprocess.run(`
- `sources/agentmemory/integrations/hermes/__init__.py:39` `stdin=subprocess.DEVNULL,`
- `sources/agentmemory/integrations/hermes/__init__.py:95` `DEFAULT_BASE_URL = "http://localhost:3111"`
- `sources/agentmemory/integrations/hermes/__init__.py:149` `if os.environ.get("AGENTMEMORY_REQUIRE_HTTPS") == "1":`
- `sources/agentmemory/integrations/hermes/__init__.py:173` `return get_secret(name)`
- `sources/agentmemory/integrations/hermes/__init__.py:180` `return Path(os.environ.get("HERMES_HOME") or Path.home() / ".hermes")`

- `sources/agentmemory/integrations/hermes/plugin.yaml:5` `homepage: "https://github.com/rohitg00/agentmemory"`

- `sources/agentmemory/src/cli/engine-config.ts:23` `export function engineSaveIntervalMs(configTexts: readonly string[], env: NodeJS.ProcessEnv = process.env): number {`

- `sources/agentmemory/src/config.ts:24` `const DATA_DIR = process.env["AGENTMEMORY_DATA_DIR"] || join(homedir(), ".agentmemory");`
- `sources/agentmemory/src/config.ts:25` `const ENV_FILE = process.env["AGENTMEMORY_ENV_FILE"] || join(homedir(), ".agentmemory", ".env");`
- `sources/agentmemory/src/config.ts:122` `const beforeHydration = { ...process.env };`
- `sources/agentmemory/src/config.ts:124` `if (process.env[k] === undefined) {`
- `sources/agentmemory/src/config.ts:125` `process.env[k] = v;`
- `sources/agentmemory/src/config.ts:134` `return hydrated !== undefined && process.env[key] === hydrated;`
- `sources/agentmemory/src/config.ts:347` `return { ...fileEnv, ...process.env, ...overrides } as Record<string, string>;`
- `sources/agentmemory/src/config.ts:615` `const raw = parseInt(process.env.CONSOLIDATION_INTERVAL_MS || String(CONSOLIDATION_INTERVAL_DEFAULT_MS), 10);`

</details>

## 5. Reproducibility

No engine version is installed by the Hermes manifest; a separately provisioned server is `unpinned` from the adapter's perspective (`sources/agentmemory/integrations/hermes/plugin.yaml:1`). Source snapshot pins are not service deployment pins.

Retained lock/checksum files: none in snapshot. Upstream completeness is unknown: fetch.py:15 excludes .lock files, fetch.py:12 excludes build/vendor directories. Absence here is not absence upstream.

- `sources/agentmemory/integrations/hermes/plugin.yaml:2` `version: 0.9.33`

## 6. Tests

Hermes adapter tests are separate from `mem::forget` tests. The scoped inventory below counts both groups independently; a direct iii/HTTP test is not an agent-tool test. Engine mem::forget audit tests are separate from Hermes dispatch tests. (`sources/agentmemory/test/remember-forget-audit.test.ts:67`).

Adapter-related declarations: **55**. Python AST test_ functions, fixtures excluded; literal TS/JS it/test, Go Test*, Rust test_* declarations. No parametrization expansion, collection, execution, or coverage measurement. Other rows are engine test candidates selected by mutation names; names alone do not verify forget coverage.

| file | declarations | adapter-related | forget-related declarations |
|---|---:|---|---|
| `sources/agentmemory/test/access-tracker.test.ts:1` | 17 | no | `deleteAccessLog removes the target entry and leaves siblings intact` `sources/agentmemory/test/access-tracker.test.ts:141`; `deleteAccessLog is a no-op for unknown ids and empty ids` `sources/agentmemory/test/access-tracker.test.ts:155`; `deleteAccessLog swallows kv.delete errors` `sources/agentmemory/test/access-tracker.test.ts:168` |
| `sources/agentmemory/test/api-lessons-audit.test.ts:1` | 8 | no | `returns a project-less lesson and hides a deleted one` `sources/agentmemory/test/api-lessons-audit.test.ts:51` |
| `sources/agentmemory/test/api-record-list-paging.test.ts:1` | 7 | no | `keeps paging when every record selected for a page was deleted before its read` `sources/agentmemory/test/api-record-list-paging.test.ts:120` |
| `sources/agentmemory/test/audit-retention.test.ts:1` | 29 | no | `does not report the legacy scope frozen once its rows are copied and only the delete pass remains` `sources/agentmemory/test/audit-retention.test.ts:216`; `moves legacy rows into their month scopes and purges index_persist noise` `sources/agentmemory/test/audit-retention.test.ts:334`; `records no summary row when there is nothing to purge` `sources/agentmemory/test/audit-retention.test.ts:391`; `waits before deleting so the legacy rows survive until the delay elapses` `sources/agentmemory/test/audit-retention.test.ts:473`; `deletes legacy rows in reverse insertion order` `sources/agentmemory/test/audit-retention.test.ts:509`; `resumes after a crash mid-delete without double-counting or double-writing the summary` `sources/agentmemory/test/audit-retention.test.ts:540`; `deletes dropped rows in reverse insertion order` `sources/agentmemory/test/audit-retention.test.ts:673` |
| `sources/agentmemory/test/auto-forget.test.ts:1` | 9 | no | `detects and deletes TTL-expired memories` `sources/agentmemory/test/auto-forget.test.ts:82`; `dryRun mode identifies but does not delete anything` `sources/agentmemory/test/auto-forget.test.ts:193` |
| `sources/agentmemory/test/capture-deleted.test.ts:1` | 6 | no | `keeps a forgotten observation deleted when its event is replayed` `sources/agentmemory/test/capture-deleted.test.ts:75`; `keeps it deleted when the event record was lost in a crash before the delete` `sources/agentmemory/test/capture-deleted.test.ts:93`; `drops a stranded inbox entry for a deleted observation instead of storing it again` `sources/agentmemory/test/capture-deleted.test.ts:108`; `still stores an observation lost in a crash when nobody deleted it` `sources/agentmemory/test/capture-deleted.test.ts:132`; `keeps the deleted marker for the retention window counted from the delete` `sources/agentmemory/test/capture-deleted.test.ts:143` |
| `sources/agentmemory/test/cli-lifecycle-safety.test.ts:1` | 3 | no | `refuses an instance-scoped global uninstall before deleting shared assets` `sources/agentmemory/test/cli-lifecycle-safety.test.ts:158` |
| `sources/agentmemory/test/compression-recovery.test.ts:1` | 20 | no | `does not restart a joining retry when its replacement is deleted` `sources/agentmemory/test/compression-recovery.test.ts:240`; `does not restore a vector after deletion while embedding is pending` `sources/agentmemory/test/compression-recovery.test.ts:307` |
| `sources/agentmemory/test/connect-dsh.test.ts:1` | 16 | no | `removes duplicate managed entries without deleting unrelated empty operations` `sources/agentmemory/test/connect-dsh.test.ts:252` |
| `sources/agentmemory/test/connect-hermes.test.ts:1` | 9 | yes | none |
| `sources/agentmemory/test/context-lessons.test.ts:1` | 9 | no | `excludes deleted lessons` `sources/agentmemory/test/context-lessons.test.ts:153` |
| `sources/agentmemory/test/deploy-onnx-prune.test.ts:1` | 3 | no | `fails before deleting anything when the target binding is missing` `sources/agentmemory/test/deploy-onnx-prune.test.ts:47`; `rejects an unrecognized binary layout without deleting it` `sources/agentmemory/test/deploy-onnx-prune.test.ts:69` |
| `sources/agentmemory/test/diagnostic-followup-rate.test.ts:1` | 10 | no | `recent-searches sweep deletes rows older than 24h, keeps fresh ones` `sources/agentmemory/test/diagnostic-followup-rate.test.ts:221` |
| `sources/agentmemory/test/diagnostics.test.ts:1` | 40 | no | `diagnose reports facets whose target was deleted` `sources/agentmemory/test/diagnostics.test.ts:520`; `heal removes only facets whose target was deleted` `sources/agentmemory/test/diagnostics.test.ts:531`; `heal keeps a facet whose target comes back before the delete` `sources/agentmemory/test/diagnostics.test.ts:555`; `deletes orphaned lease` `sources/agentmemory/test/diagnostics.test.ts:645` |
| `sources/agentmemory/test/directory-review-privacy.test.ts:1` | 26 | no | `marks tools that delete or overwrite data as destructive` `sources/agentmemory/test/directory-review-privacy.test.ts:198` |
| `sources/agentmemory/test/evict.test.ts:1` | 13 | no | `runs session recovery before deleting a stale observed session` `sources/agentmemory/test/evict.test.ts:142`; `does not delete a session restarted while eviction waits for publication` `sources/agentmemory/test/evict.test.ts:267` |
| `sources/agentmemory/test/facet-delete-cascade.test.ts:1` | 11 | no | `lesson-delete removes facets that point at the deleted lesson` `sources/agentmemory/test/facet-delete-cascade.test.ts:123`; `lesson-decay-sweep removes facets only for soft-deleted lessons` `sources/agentmemory/test/facet-delete-cascade.test.ts:139`; `forget with observationIds removes facets for the forgotten observations` `sources/agentmemory/test/facet-delete-cascade.test.ts:165`; `whole-session forget removes facets for every forgotten observation` `sources/agentmemory/test/facet-delete-cascade.test.ts:190`; `deleteFacetsForTargets keeps facets whose target is still live` `sources/agentmemory/test/facet-delete-cascade.test.ts:216`; `deleteFacetsForTargets keeps facets of a live replay-imported lesson (lesson_ id)` `sources/agentmemory/test/facet-delete-cascade.test.ts:235`; `deleteFacetsForTargets keeps facets when a lesson-prefixed id still names a live memory` `sources/agentmemory/test/facet-delete-cascade.test.ts:253`; `deleteFacetsForTargets judges a shared obs_ id per facet type` `sources/agentmemory/test/facet-delete-cascade.test.ts:262`; `deleteFacetsForTargets removes facets of a soft-deleted replay-imported lesson` `sources/agentmemory/test/facet-delete-cascade.test.ts:272`; `deleteFacetsForTargets leaves observation facets alone unless the caller opts in` `sources/agentmemory/test/facet-delete-cascade.test.ts:289`; `deleteFacetsForTargets removes observation facets for a deleted observation when opted in` `sources/agentmemory/test/facet-delete-cascade.test.ts:301` |
| `sources/agentmemory/test/fs-watcher.test.ts:1` | 18 | no | `emits changeKind=file_delete when a watched file is removed` `sources/agentmemory/test/fs-watcher.test.ts:80` |
| `sources/agentmemory/test/governance.test.ts:1` | 22 | no | `governance-delete removes specified memories` `sources/agentmemory/test/governance.test.ts:96`; `governance-delete handles non-existent IDs gracefully` `sources/agentmemory/test/governance.test.ts:110`; `governance-bulk deletes by type filter` `sources/agentmemory/test/governance.test.ts:123`; `governance-delete removes facets pointing at the deleted memory` `sources/agentmemory/test/governance.test.ts:152`; `governance-delete sweeps facets for already-missing memories` `sources/agentmemory/test/governance.test.ts:165`; `governance-delete keeps facets on live semantic records` `sources/agentmemory/test/governance.test.ts:183`; `governance-bulk removes facets for every deleted memory` `sources/agentmemory/test/governance.test.ts:196`; `governance-delete removes the memory from the search index` `sources/agentmemory/test/governance.test.ts:252`; `governance-delete flushes persistence immediately` `sources/agentmemory/test/governance.test.ts:265`; `governance-delete skips persistence flush when nothing was deleted` `sources/agentmemory/test/governance.test.ts:278`; `governance-bulk removes deleted memories from the search index` `sources/agentmemory/test/governance.test.ts:289`; `governance-delete writes no audit row when no id existed` `sources/agentmemory/test/governance.test.ts:348`; `governance-delete audits only the ids it deleted` `sources/agentmemory/test/governance.test.ts:358`; `governance-delete reports a failed delete separately` `sources/agentmemory/test/governance.test.ts:371`; `governance-bulk does not count or audit a candidate that vanished before delete` `sources/agentmemory/test/governance.test.ts:449` |
| `sources/agentmemory/test/graph-index-migration.test.ts:1` | 17 | no | `keeps current writer updates, deletes and late inserts while discovery is running` `sources/agentmemory/test/graph-index-migration.test.ts:186` |
| `sources/agentmemory/test/graph-ingestion.test.ts:1` | 25 | no | `does not cache old clean state while an invalidated journal write is still in flight` `sources/agentmemory/test/graph-ingestion.test.ts:286` |
| `sources/agentmemory/test/graph-query.test.ts:1` | 13 | no | `refreshes after writes, updates, deletion and stale markers without a TTL` `sources/agentmemory/test/graph-query.test.ts:144` |
| `sources/agentmemory/test/graph-retrieval.test.ts:1` | 32 | no | `rechecks canonical evidence before exposing a result after forget` `sources/agentmemory/test/graph-retrieval.test.ts:539` |
| `sources/agentmemory/test/hermes-live-model.py:1` | 0 | yes | none |
| `sources/agentmemory/test/hermes-live-preflight.test.ts:1` | 2 | yes | none |
| `sources/agentmemory/test/hermes-native-e2e.py:1` | 0 | yes | none |
| `sources/agentmemory/test/hermes-plugin.test.ts:1` | 2 | yes | none |
| `sources/agentmemory/test/hermes-provider-contracts.py:1` | 42 | yes | `test_external_evolution_conflict_never_deletes_or_creates_a_competing_record` `sources/agentmemory/test/hermes-provider-contracts.py:266`; `test_same_returned_id_is_never_deleted` `sources/agentmemory/test/hermes-provider-contracts.py:282`; `test_prefetch_cache_is_scoped_one_shot_and_invalidated_by_writes` `sources/agentmemory/test/hermes-provider-contracts.py:335` |
| `sources/agentmemory/test/index-persistence.test.ts:1` | 33 | no | `removes deleted vectors from their bucket` `sources/agentmemory/test/index-persistence.test.ts:168`; `a delete touches only the bucket that owns the id, in a multi-bucket store` `sources/agentmemory/test/index-persistence.test.ts:185`; `clearing the index deletes every persisted vector` `sources/agentmemory/test/index-persistence.test.ts:201`; `deletes nothing when a manifest read fails` `sources/agentmemory/test/index-persistence.test.ts:662`; `deletes nothing when a manifest names no generation` `sources/agentmemory/test/index-persistence.test.ts:678`; `deletes nothing when a legacy-prefixed scope does not parse` `sources/agentmemory/test/index-persistence.test.ts:689`; `deletes nothing when scopes cannot be listed, and sweeps on the next boot` `sources/agentmemory/test/index-persistence.test.ts:701`; `records no completion when a delete fails, and finishes on the next boot` `sources/agentmemory/test/index-persistence.test.ts:719`; `runs once: a later boot neither lists scopes nor deletes` `sources/agentmemory/test/index-persistence.test.ts:738`; `audits each sweep delete and the summary when index auditing is on` `sources/agentmemory/test/index-persistence.test.ts:758` |
| `sources/agentmemory/test/kv-paged.test.ts:1` | 9 | no | `listPage asc order and deleted cursor keys keep paging` `sources/agentmemory/test/kv-paged.test.ts:117` |
| `sources/agentmemory/test/lesson-index-recall.test.ts:1` | 5 | no | `stops returning deleted lessons` `sources/agentmemory/test/lesson-index-recall.test.ts:87` |
| `sources/agentmemory/test/lessons.test.ts:1` | 33 | no | `soft-deletes low-confidence unreinforced lessons` `sources/agentmemory/test/lessons.test.ts:341`; `soft-deletes an existing lesson` `sources/agentmemory/test/lessons.test.ts:382`; `excludes a soft-deleted lesson from recall and list` `sources/agentmemory/test/lessons.test.ts:399`; `returns not found for an already-deleted lesson` `sources/agentmemory/test/lessons.test.ts:418`; `creates a fresh lesson when deleted content is re-saved` `sources/agentmemory/test/lessons.test.ts:451`; `records a lesson_delete audit row` `sources/agentmemory/test/lessons.test.ts:467` |
| `sources/agentmemory/test/mcp-standalone-proxy.test.ts:1` | 21 | no | `proxies memory_governance_delete to the DELETE REST endpoint` `sources/agentmemory/test/mcp-standalone-proxy.test.ts:149`; `invalidates the handle on proxy failure, so the next call re-probes` `sources/agentmemory/test/mcp-standalone-proxy.test.ts:258` |
| `sources/agentmemory/test/mcp-standalone.test.ts:1` | 35 | no | `get/set/list/delete operations work` `sources/agentmemory/test/mcp-standalone.test.ts:100`; `memory_smart_search rejects empty query to prevent match-all in forget flow` `sources/agentmemory/test/mcp-standalone.test.ts:300`; `memory_governance_delete removes memories by id array` `sources/agentmemory/test/mcp-standalone.test.ts:396`; `memory_governance_delete accepts CSV-string memoryIds too` `sources/agentmemory/test/mcp-standalone.test.ts:424`; `memory_governance_delete throws when memoryIds is missing or empty` `sources/agentmemory/test/mcp-standalone.test.ts:440`; `memory_governance_delete silently skips unknown ids` `sources/agentmemory/test/mcp-standalone.test.ts:450`; `memory_governance_delete marks the result local` `sources/agentmemory/test/mcp-standalone.test.ts:528`; `memory_governance_delete says the change is lost on exit when there is no local file` `sources/agentmemory/test/mcp-standalone.test.ts:544` |
| `sources/agentmemory/test/mesh-export-project-scope.test.ts:1` | 10 | no | `does not skip surviving rows when a row is deleted between pages` `sources/agentmemory/test/mesh-export-project-scope.test.ts:133` |
| `sources/agentmemory/test/multimodal.test.ts:1` | 13 | no | `should increment and decrement ref counts correctly with deletion parity` `sources/agentmemory/test/multimodal.test.ts:279`; `deleteImage should return deletedBytes after successful deletion` `sources/agentmemory/test/multimodal.test.ts:359`; `deleteImage should return 0 for non-existent files` `sources/agentmemory/test/multimodal.test.ts:372` |
| `sources/agentmemory/test/observation-delete-race.test.ts:1` | 1 | no | `invalidates a delayed vector job when project-cap eviction deletes its observation` `sources/agentmemory/test/observation-delete-race.test.ts:91` |
| `sources/agentmemory/test/observation-source.test.ts:1` | 20 | no | `rejects an oversized replacement before deleting or writing any stored records` `sources/agentmemory/test/observation-source.test.ts:282` |
| `sources/agentmemory/test/obsidian-export.test.ts:1` | 16 | no | `skips deleted lessons` `sources/agentmemory/test/obsidian-export.test.ts:266` |
| `sources/agentmemory/test/opencode-auto-context.test.ts:1` | 10 | no | `session.deleted clears the cache to avoid stale entries` `sources/agentmemory/test/opencode-auto-context.test.ts:33` |
| `sources/agentmemory/test/opencode-plugin-v2.test.ts:1` | 26 | no | `session.deleted fecha a sessao e libera as chaves` `sources/agentmemory/test/opencode-plugin-v2.test.ts:397` |
| `sources/agentmemory/test/reflect.test.ts:1` | 29 | no | `lists all non-deleted insights sorted by confidence` `sources/agentmemory/test/reflect.test.ts:602`; `soft-deletes low-confidence unreinforced insights` `sources/agentmemory/test/reflect.test.ts:664` |
| `sources/agentmemory/test/remember-forget-audit.test.ts:1` | 16 | no | `does not emit an audit row when nothing is deleted` `sources/agentmemory/test/remember-forget-audit.test.ts:116`; `returns deleted: 0 for a nonexistent memoryId (lesson id)` `sources/agentmemory/test/remember-forget-audit.test.ts:130`; `reports a failed delete separately and keeps it out of the deleted count` `sources/agentmemory/test/remember-forget-audit.test.ts:419` |
| `sources/agentmemory/test/remember-lock-scope.test.ts:1` | 7 | no | `waits for the mem:remember lock before deleting a memory` `sources/agentmemory/test/remember-lock-scope.test.ts:261` |
| `sources/agentmemory/test/retention.test.ts:1` | 16 | no | `dry-run eviction shows candidates without deleting` `sources/agentmemory/test/retention.test.ts:211`; `mem::retention-evict deletes semantic memories from mem:semantic, not mem:memories` `sources/agentmemory/test/retention.test.ts:300` |
| `sources/agentmemory/test/search-index-write-race.test.ts:1` | 8 | no | `keeps newer keywords/vectors and deletion while accepting unaffected rows from the same old batch` `sources/agentmemory/test/search-index-write-race.test.ts:57`; `rejects an absent-ID completion after delete/add/delete returns the ID to absence` `sources/agentmemory/test/search-index-write-race.test.ts:106` |
| `sources/agentmemory/test/session-end-triggers-graph.test.ts:1` | 12 | no | `event::session::stopped uses TriggerAction.Void for fire-and-forget` `sources/agentmemory/test/session-end-triggers-graph.test.ts:19` |
| `sources/agentmemory/test/session-index-self-heal.test.ts:1` | 6 | no | `deletes nothing when an observation scope read fails` `sources/agentmemory/test/session-index-self-heal.test.ts:146`; `deletes nothing when a later collection read fails` `sources/agentmemory/test/session-index-self-heal.test.ts:179` |
| `sources/agentmemory/test/session-index.test.ts:1` | 24 | no | `skips index entries whose session record was since deleted` `sources/agentmemory/test/session-index.test.ts:518` |
| `sources/agentmemory/test/session-sweep.test.ts:1` | 15 | no | `writes no audit entry and deletes nothing when no session is stale` `sources/agentmemory/test/session-sweep.test.ts:263` |
| `sources/agentmemory/test/sketches.test.ts:1` | 29 | no | `discards sketch and deletes actions and edges` `sources/agentmemory/test/sketches.test.ts:324`; `deletes actions and edges of expired sketches` `sources/agentmemory/test/sketches.test.ts:525` |
| `sources/agentmemory/test/slots.test.ts:1` | 14 | no | `delete removes the slot` `sources/agentmemory/test/slots.test.ts:120` |
| `sources/agentmemory/test/state-mutation-order.test.ts:1` | 5 | no | `preserves set and native update responses while delete resolves without a value` `sources/agentmemory/test/state-mutation-order.test.ts:169` |
| `sources/agentmemory/test/temporal-graph.test.ts:1` | 11 | no | `keeps delimiter-safe key membership correct across direct updates, stale records, historical copies and deletion` `sources/agentmemory/test/temporal-graph.test.ts:442` |
| `sources/agentmemory/test/vector-pending-log.test.ts:1` | 14 | no | `requires the latest same-key write acknowledgment and invalidates an earlier success` `sources/agentmemory/test/vector-pending-log.test.ts:160` |
| `sources/agentmemory/test/viewer-counts-reset.test.ts:1` | 3 | no | `does not retain stale counts when invalidated during both load attempts` `sources/agentmemory/test/viewer-counts-reset.test.ts:59` |
| `sources/agentmemory/test/viewer-dashboard-reliability.test.ts:1` | 24 | no | `treats a soft-deleted lesson as removed and patches other row kinds by id` `sources/agentmemory/test/viewer-dashboard-reliability.test.ts:154`; `applies session updates, activity counts and deletes` `sources/agentmemory/test/viewer-dashboard-reliability.test.ts:167` |
| `sources/agentmemory/test/viewer-generation-deltas.test.ts:1` | 8 | no | `uses bounded index metadata and invalidates the viewer instead of reading snapshot payloads` `sources/agentmemory/test/viewer-generation-deltas.test.ts:56`; `invalidates after metadata failure and refreshes counts after reset` `sources/agentmemory/test/viewer-generation-deltas.test.ts:90` |
| `sources/agentmemory/test/viewer-graph-reset.test.ts:1` | 12 | no | `retries a reset-invalidated snapshot once to recover startup metadata` `sources/agentmemory/test/viewer-graph-reset.test.ts:141`; `stops retrying when another reset invalidates the second snapshot` `sources/agentmemory/test/viewer-graph-reset.test.ts:157`; `invalidates old queries and selection even when the reset delta requires a reload` `sources/agentmemory/test/viewer-graph-reset.test.ts:188`; `drops old snapshot totals when a reset could only send an item invalidation` `sources/agentmemory/test/viewer-graph-reset.test.ts:195` |
| `sources/agentmemory/test/viewer-live-events.test.ts:1` | 5 | no | `pushes session.deleted when a session is removed` `sources/agentmemory/test/viewer-live-events.test.ts:51`; `pushes memory.created, memory.updated and memory.deleted with the full row` `sources/agentmemory/test/viewer-live-events.test.ts:81` |
| `sources/agentmemory/test/viewer-memories-page.test.ts:1` | 3 | no | `routes edits through evolve and forgets through governance delete, with no per-row delete` `sources/agentmemory/test/viewer-memories-page.test.ts:67` |
| `sources/agentmemory/test/viewer-streams.test.ts:1` | 12 | no | `pushes created, updated and deleted rows for lessons, actions, crystals, semantic, procedural and audit` `sources/agentmemory/test/viewer-streams.test.ts:106`; `invalidates legacy snapshot changes without forwarding unverified graph rows` `sources/agentmemory/test/viewer-streams.test.ts:155` |
| `sources/agentmemory/test/working-memory.test.ts:1` | 8 | no | `core-remove deletes entry` `sources/agentmemory/test/working-memory.test.ts:72` |

## 7. Corrections to B2

Old values below are copied from the triage row (TRIAGE.md), not reconstructed from memory. A refinement narrows an over-broad field; it need not reverse its original value. Counts are distinct field paths, not the number of factual errors.

- `engine_delete.scope`: `"Named deletion/retraction endpoint or method, not a guarantee of physical erasure. Its reach may overlap an agent tool; lack of a native tool is counted separately."` → mem::forget deletes selected memory/observation/session targets; one fact ID leaves independent raw observations. Evidence: `sources/agentmemory/src/functions/remember.ts:344`, `sources/agentmemory/src/functions/remember.ts:388`.
- `context.mechanism.v`: `"Source expressions below identify retrieval, cached/static returns, lifecycle capture, and limits. A hook definition is not a guarantee it emits nonempty context each turn; branches and configuration apply."` → System instructions describe recall/save and scoping (`sources/agentmemory/integrations/hermes/__init__.py:390`). Prefetch smart-search requests five results, reads narratives, clips titles to 200 and narratives to 1000, and returns at most 6000 characters (`sources/agentmemory/integrations/hermes/__init__.py:402`; `sources/agentmemory/integrations/hermes/__init__.py:429`). Tools return service JSON; recall/search limit validated 1..100 (`sources/agentmemory/integrations/hermes/__init__.py:521`). Evidence: `sources/agentmemory/integrations/hermes/__init__.py:390`, `sources/agentmemory/integrations/hermes/__init__.py:402`, `sources/agentmemory/integrations/hermes/__init__.py:429`, `sources/agentmemory/integrations/hermes/__init__.py:521`.
- `sec.telemetry.v`: `"unknown: no emitter identified in the selected adapter files; external dependencies not cleared"` → Direct access `yes`, via HTTP or iii functions with required service credentials; plugin tool absence does not remove the engine function. AGENTMEMORY_URL, AGENTMEMORY_SECRET, AGENTMEMORY_PROJECT_NAME and AGENTMEMORY_AGENT_ID are configured separately (`sources/agentmemory/integrations/hermes/plugin.yaml:7`). Local-service accountless mode `yes` with compatible local model configuration; service secret is optional, not a hosted-account requirement. Storage/child-process/env/telemetry implementation sites are below. Evidence: `sources/agentmemory/integrations/hermes/plugin.yaml:7`.
- `repro.decls`: `["integrations/hermes/plugin.yaml:5", "integrations/hermes/plugin.yaml:7", "integrations/hermes/plugin.yaml:10", "integrations/hermes/plugin.yaml:13", "integrations/hermes/plugin.yaml:16", "integrations/hermes/plugin.yaml:20", "integrations/hermes/plugin.yaml:21", "integrations/hermes/plugin.yaml:22", "integrations/hermes/plugin.yaml:23", "integrations/hermes/plugin.yaml:24", "integrations/hermes/plugin.yaml:25", "integrations/hermes/plugin.yaml:26", "integrations/hermes/plugin.yaml:27", "package.json:3", "package.json:83", "package.json:87", "package.json:89", "package.json:101", "package.json:106"]` → No engine version is installed by the Hermes manifest; a separately provisioned server is `unpinned` from the adapter's perspective (`sources/agentmemory/integrations/hermes/plugin.yaml:1`). Source snapshot pins are not service deployment pins. Evidence: `sources/agentmemory/integrations/hermes/plugin.yaml:1`.
- `tests.count`: `3745` → 55 adapter-related declarations using the stated parser/scope; B2 counted a different, repository-wide inventory. This is a scope refinement, not a count of executed or passing tests. Hermes adapter tests are separate from `mem::forget` tests. The scoped inventory below counts both groups independently; a direct iii/HTTP test is not an agent-tool test. Engine mem::forget audit tests are separate from Hermes dispatch tests. (`sources/agentmemory/test/remember-forget-audit.test.ts:67`). Evidence: `sources/agentmemory/test/remember-forget-audit.test.ts:67`.

MATRIX counts these 5 explicit field corrections/refinements. Sections 1-6 provide the body and test inventories. Test-name candidates are not passing tests or measured coverage.
