# mnemos: B3 static correctness review

Snapshot `8ca8e76ce4da54d28e4b74d247b26db2fdea1238` (MANIFEST.json row `mnemos`). No plugin execution or passing-test claims.

## 1. Architecture

`provider`: catalogue wrapper installs mnemos-continuity; Python provider uses a local runtime/SQLite store scoped to the Hermes profile, with optional encoder/model facilities (`sources/mnemos/hermes-plugin/pyproject.toml:12`; `sources/mnemos/mnemos/integrations/hermes/provider.py:288`).

Hook/body inventory (no-op/fallback declarations are not active behavior):

| callable | declaration | first executable body line |
|---|---|---|
| `register` | `sources/mnemos/hermes-plugin/__init__.py:10` | `sources/mnemos/hermes-plugin/__init__.py:11` |
| `is_available` | `sources/mnemos/mnemos/integrations/hermes/provider.py:263` | `sources/mnemos/mnemos/integrations/hermes/provider.py:264` |
| `initialize` | `sources/mnemos/mnemos/integrations/hermes/provider.py:267` | `sources/mnemos/mnemos/integrations/hermes/provider.py:268` |
| `system_prompt_block` | `sources/mnemos/mnemos/integrations/hermes/provider.py:311` | `sources/mnemos/mnemos/integrations/hermes/provider.py:312` |
| `prefetch` | `sources/mnemos/mnemos/integrations/hermes/provider.py:321` | `sources/mnemos/mnemos/integrations/hermes/provider.py:322` |
| `queue_prefetch` | `sources/mnemos/mnemos/integrations/hermes/provider.py:353` | `sources/mnemos/mnemos/integrations/hermes/provider.py:354` |
| `sync_turn` | `sources/mnemos/mnemos/integrations/hermes/provider.py:372` | `sources/mnemos/mnemos/integrations/hermes/provider.py:372` |
| `get_tool_schemas` | `sources/mnemos/mnemos/integrations/hermes/provider.py:383` | `sources/mnemos/mnemos/integrations/hermes/provider.py:384` |
| `handle_tool_call` | `sources/mnemos/mnemos/integrations/hermes/provider.py:391` | `sources/mnemos/mnemos/integrations/hermes/provider.py:392` |
| `on_turn_start` | `sources/mnemos/mnemos/integrations/hermes/provider.py:403` | `sources/mnemos/mnemos/integrations/hermes/provider.py:404` |
| `on_session_end` | `sources/mnemos/mnemos/integrations/hermes/provider.py:417` | `sources/mnemos/mnemos/integrations/hermes/provider.py:419` |
| `on_session_switch` | `sources/mnemos/mnemos/integrations/hermes/provider.py:427` | `sources/mnemos/mnemos/integrations/hermes/provider.py:435` |
| `on_pre_compress` | `sources/mnemos/mnemos/integrations/hermes/provider.py:451` | `sources/mnemos/mnemos/integrations/hermes/provider.py:453` |
| `on_delegation` | `sources/mnemos/mnemos/integrations/hermes/provider.py:456` | `sources/mnemos/mnemos/integrations/hermes/provider.py:456` |
| `on_memory_write` | `sources/mnemos/mnemos/integrations/hermes/provider.py:461` | `sources/mnemos/mnemos/integrations/hermes/provider.py:470` |
| `get_config_schema` | `sources/mnemos/mnemos/integrations/hermes/provider.py:508` | `sources/mnemos/mnemos/integrations/hermes/provider.py:509` |
| `save_config` | `sources/mnemos/mnemos/integrations/hermes/provider.py:541` | `sources/mnemos/mnemos/integrations/hermes/provider.py:542` |
| `shutdown` | `sources/mnemos/mnemos/integrations/hermes/provider.py:545` | `sources/mnemos/mnemos/integrations/hermes/provider.py:546` |

## 2. Forget paths and derived copies

`yes` for logical forgetting, correcting B2 update_only: `mnemos_correct(action='forget'/'archive'/'remove'/'delete')` dispatches runtime.correct and clears prefetch cache (`sources/mnemos/mnemos/integrations/hermes/provider.py:587`; `sources/mnemos/mnemos/integrations/hermes/provider.py:592`). Runtime routes forget actions to _forget_pair, retiring note/engram together and handling linked lessons (`sources/mnemos/mnemos/simple_runtime.py:6647`; `sources/mnemos/mnemos/simple_runtime.py:6692`). CLI/runtime forget_memory uses the same archive path (`sources/mnemos/mnemos/simple_runtime.py:6981`). `partial`: archived records/version history remain; this is retrieval exclusion, not row erasure. Scope and lineage handling do not purge host state.

Host state.db is independent (`hermes-agent/hermes_state.py:181`). No reviewed adapter forget body performs a general purge of that database or built-in MEMORY.md/USER.md. Prompt/tool-result copies and independent source transcripts remain separate; source operation bodies below establish the scope of each plugin mutation.

<details>
<summary>Deletion, invalidation, archive and reset body ledger</summary>

Public routes, SDK/CLI dispatch and internal storage operations are distinguished by their source. Cache invalidation and management-object deletion are not additional fact-forget tools. A stub is not working deletion. Call-through means the downstream body still determines erasure. HTTP prefixes/selectors must be read with route registration.

**_cmd_forget**, body `sources/mnemos/mnemos/cli.py:2396`; implementation/call-through.

- `sources/mnemos/mnemos/cli.py:2394` `def _cmd_forget(args: argparse.Namespace) -> int:`
- `sources/mnemos/mnemos/cli.py:2396` `return _change_memory(args, correcting=False)`

**_cmd_repair_archive_rows**, body `sources/mnemos/mnemos/cli.py:3347`; implementation/call-through.

- `sources/mnemos/mnemos/cli.py:3342` `def _cmd_repair_archive_rows(args: argparse.Namespace) -> int:`
- `sources/mnemos/mnemos/cli.py:3349` `plan = runtime.repair_archive_rows(write=args.write)`
- `sources/mnemos/mnemos/cli.py:3362` `print(f"Archive rows in {runtime.db_path}")`
- `sources/mnemos/mnemos/cli.py:3364` `_repair_row(found["rows"], "rows in the archive")`
- `sources/mnemos/mnemos/cli.py:3365` `_repair_row(found["archived"], "for memories in the archive", "stay")`
- `sources/mnemos/mnemos/cli.py:3366` `_repair_row(len(found["repeats"]), "repeating a memory no longer archived, word for word", "go")`
- `sources/mnemos/mnemos/cli.py:3370` `len(found["history"]), "for a memory no longer archived, in other words", "stay, as history"`

**operation and route call sites**, body `sources/mnemos/mnemos/companion/page/engine/post.js:96`; body evidence.

- `sources/mnemos/mnemos/companion/page/engine/post.js:95` `const { gl } = this;`
- `sources/mnemos/mnemos/companion/page/engine/post.js:96` `for (const t of [this.accum, ...this.down, ...this.up]) { gl.deleteFramebuffer(t.fbo); gl.deleteTexture(t.tex); }`
- `sources/mnemos/mnemos/companion/page/engine/post.js:97` `for (const p of [this.downP, this.upP, this.finalP]) gl.deleteProgram(p.p);   // a window resized many times made many`
- `sources/mnemos/mnemos/companion/page/engine/post.js:98` `}`
- `sources/mnemos/mnemos/companion/page/engine/post.js:99` ``
- `sources/mnemos/mnemos/companion/page/engine/post.js:100` `clear() {`
- `sources/mnemos/mnemos/companion/page/engine/post.js:101` `const { gl } = this;`

**operation and route call sites**, body `sources/mnemos/mnemos/companion/page/engine/tissue.js:237`; body evidence.

- `sources/mnemos/mnemos/companion/page/engine/tissue.js:236` `const { gl } = this;`
- `sources/mnemos/mnemos/companion/page/engine/tissue.js:237` `gl.deleteBuffer(this.buf); gl.deleteTexture(this.stateTex);   // the program is shared`
- `sources/mnemos/mnemos/companion/page/engine/tissue.js:238` `for (const v of new Set([this.vao, ...(this.vaos || [])])) gl.deleteVertexArray(v);`
- `sources/mnemos/mnemos/companion/page/engine/tissue.js:239` `this.count = 0;`
- `sources/mnemos/mnemos/companion/page/engine/tissue.js:240` `}`
- `sources/mnemos/mnemos/companion/page/engine/tissue.js:241` ``
- `sources/mnemos/mnemos/companion/page/engine/tissue.js:242` `// written in place: thousands of calls a frame, so no small arrays for the collector to sweep up`

**operation and route call sites**, body `sources/mnemos/mnemos/companion/page/mind/mind.js:635`; body evidence.

- `sources/mnemos/mnemos/companion/page/mind/mind.js:634` `const drop = (e) => {`
- `sources/mnemos/mnemos/companion/page/mind/mind.js:635` `touches.delete(e.pointerId);`
- `sources/mnemos/mnemos/companion/page/mind/mind.js:636` `if (touches.size < 2) pinch = null;`
- `sources/mnemos/mnemos/companion/page/mind/mind.js:637` `if (dragging && e?.type === 'pointerup' && Math.hypot(e.clientX - dragging.x0, e.clientY - dragging.y0) < 6 && performance.now() - dragging.t0 < 450) {`
- `sources/mnemos/mnemos/companion/page/mind/mind.js:638` `const c = pick(e.clientX, e.clientY);`
- `sources/mnemos/mnemos/companion/page/mind/mind.js:639` `if (c >= 0 && memAt.has(D.cells[c].fullId)) openRoom({ tab: 'memories', memory: D.cells[c].fullId });`
- `sources/mnemos/mnemos/companion/page/mind/mind.js:1039` `if (s !== gl.TIMEOUT_EXPIRED) {`
- `sources/mnemos/mnemos/companion/page/mind/mind.js:1040` `gl.deleteSync(meterSync); meterSync = null;`
- `sources/mnemos/mnemos/companion/page/mind/mind.js:1041` `if (s !== gl.WAIT_FAILED && meterOf === t) {`
- `sources/mnemos/mnemos/companion/page/mind/mind.js:1042` `gl.bindBuffer(gl.PIXEL_PACK_BUFFER, meterBuf);`
- `sources/mnemos/mnemos/companion/page/mind/mind.js:1043` `gl.getBufferSubData(gl.PIXEL_PACK_BUFFER, 0, meterPx);`
- `sources/mnemos/mnemos/companion/page/mind/mind.js:1044` `gl.bindBuffer(gl.PIXEL_PACK_BUFFER, null);`
- `sources/mnemos/mnemos/companion/page/mind/mind.js:1082` `if (!gl.getParameter(gpuT.GPU_DISJOINT_EXT)) (gpuMs[label] ||= []).push(gl.getQueryParameter(q, gl.QUERY_RESULT) / 1e6);`
- `sources/mnemos/mnemos/companion/page/mind/mind.js:1083` `gl.deleteQuery(q);`
- `sources/mnemos/mnemos/companion/page/mind/mind.js:1084` `return false;`
- `sources/mnemos/mnemos/companion/page/mind/mind.js:1085` `});`
- `sources/mnemos/mnemos/companion/page/mind/mind.js:1086` `}`
- `sources/mnemos/mnemos/companion/page/mind/mind.js:1087` ``

**operation and route call sites**, body `sources/mnemos/mnemos/companion/page/shared/regions.js:80`; body evidence.

- `sources/mnemos/mnemos/companion/page/shared/regions.js:79` `r.words = r.scored.filter(([w]) => w !== pick?.[0]).slice(0, 3).map(([w]) => (forms.get(w).proper / forms.get(w).all > 0.4 ? shown(w) : w));`
- `sources/mnemos/mnemos/companion/page/shared/regions.js:80` `delete r.scored;`
- `sources/mnemos/mnemos/companion/page/shared/regions.js:81` `}`
- `sources/mnemos/mnemos/companion/page/shared/regions.js:82` `return regions;`
- `sources/mnemos/mnemos/companion/page/shared/regions.js:83` `}`

**operation and route call sites**, body `sources/mnemos/mnemos/companion/page/shared/room.js:264`; body evidence.

- `sources/mnemos/mnemos/companion/page/shared/room.js:263` `if (!text) return;`
- `sources/mnemos/mnemos/companion/page/shared/room.js:264` `this.sending = 'shared-new'; this.notices.delete('shared-new'); this.render();`
- `sources/mnemos/mnemos/companion/page/shared/room.js:265` `let answer;`
- `sources/mnemos/mnemos/companion/page/shared/room.js:266` `try { answer = await this.hooks.send({ command: 'shared', action: 'add', text }); }`
- `sources/mnemos/mnemos/companion/page/shared/room.js:267` `catch { answer = { error: SHARED.failed }; }`
- `sources/mnemos/mnemos/companion/page/shared/room.js:268` `this.sending = null;`
- `sources/mnemos/mnemos/companion/page/shared/room.js:269` `if (answer?.result) this.drafts.delete('shared-new');`
- `sources/mnemos/mnemos/companion/page/shared/room.js:270` `else this.notices.set('shared-new', answer?.error || SHARED.failed);`
- `sources/mnemos/mnemos/companion/page/shared/room.js:271` `this.render(); this.find('[data-draft="shared-new"]')?.focus();`
- `sources/mnemos/mnemos/companion/page/shared/room.js:272` `}`
- `sources/mnemos/mnemos/companion/page/shared/room.js:273` ``
- `sources/mnemos/mnemos/companion/page/shared/room.js:456` `if (!text || this.sending || !this.hooks.send) return;`
- `sources/mnemos/mnemos/companion/page/shared/room.js:457` `this.sending = key; this.notices.delete(key); this.render();`
- `sources/mnemos/mnemos/companion/page/shared/room.js:458` `let answer;`
- `sources/mnemos/mnemos/companion/page/shared/room.js:459` `try { answer = await this.hooks.send(key === 'new' ? { command: 'note', text } : { command: 'reply', note_id: key, text }); }`
- `sources/mnemos/mnemos/companion/page/shared/room.js:460` `catch { answer = { error: 'It couldn’t be sent just now. Is 'mnemos log' still running?' }; }`
- `sources/mnemos/mnemos/companion/page/shared/room.js:461` `this.sending = null;`
- `sources/mnemos/mnemos/companion/page/shared/room.js:462` `if (answer?.result) { this.drafts.delete(key); this.opened.delete(key); }`
- `sources/mnemos/mnemos/companion/page/shared/room.js:463` `else this.notices.set(key, answer?.error || 'It couldn’t be sent.');`
- `sources/mnemos/mnemos/companion/page/shared/room.js:464` `this.render();`
- `sources/mnemos/mnemos/companion/page/shared/room.js:465` `}`
- `sources/mnemos/mnemos/companion/page/shared/room.js:466` ``
- `sources/mnemos/mnemos/companion/page/shared/room.js:483` `if (showing && !this.readTimers.has(id)) this.readTimers.set(id, setTimeout(() => this.markSeen(id), SEEN_AFTER));`
- `sources/mnemos/mnemos/companion/page/shared/room.js:484` `if (!showing && this.readTimers.has(id)) { clearTimeout(this.readTimers.get(id)); this.readTimers.delete(id); }`
- `sources/mnemos/mnemos/companion/page/shared/room.js:485` `}`
- `sources/mnemos/mnemos/companion/page/shared/room.js:486` `}, { root, threshold: [0, 0.2, 0.4, 0.6, 0.8, 1] });`
- `sources/mnemos/mnemos/companion/page/shared/room.js:487` `notes.forEach((n) => this.reading.observe(n));`
- `sources/mnemos/mnemos/companion/page/shared/room.js:488` `}`
- `sources/mnemos/mnemos/companion/page/shared/room.js:489` `markSeen(id) {`
- `sources/mnemos/mnemos/companion/page/shared/room.js:490` `this.readTimers.delete(id);`
- `sources/mnemos/mnemos/companion/page/shared/room.js:491` `if (!this.present || document.visibilityState !== 'visible' || this.seen.has(id)) return;`
- `sources/mnemos/mnemos/companion/page/shared/room.js:492` `this.seen.add(id); this.toMark.add(id);`
- `sources/mnemos/mnemos/companion/page/shared/room.js:493` `clearTimeout(this.marking);`
- `sources/mnemos/mnemos/companion/page/shared/room.js:494` `this.marking = setTimeout(() => {`
- `sources/mnemos/mnemos/companion/page/shared/room.js:496` `// not said? then it's still unread, and the next look tries again`
- `sources/mnemos/mnemos/companion/page/shared/room.js:497` `this.hooks.read(ids).catch(() => ids.forEach((i) => this.seen.delete(i)));`
- `sources/mnemos/mnemos/companion/page/shared/room.js:498` `}, 300);`
- `sources/mnemos/mnemos/companion/page/shared/room.js:499` `}`
- `sources/mnemos/mnemos/companion/page/shared/room.js:500` `hoursHTML() {`
- `sources/mnemos/mnemos/companion/page/shared/room.js:501` `const { life } = this.ctx();`
- `sources/mnemos/mnemos/companion/page/shared/room.js:524` `}`
- `sources/mnemos/mnemos/companion/page/shared/room.js:525` `if (mode === 'forget') {`
- `sources/mnemos/mnemos/companion/page/shared/room.js:526` `return '<div class="confirm" role="alertdialog" aria-labelledby="let-go-h"><div class="set-title" id="let-go-h">Let this memory go?</div>`
- `sources/mnemos/mnemos/companion/page/shared/room.js:527` `<p class="set-d">${shared ? SHARED.withdraw : 'It won't come back by its words, its meaning or its id. It's kept in the archive, not deleted, and ${esc(who.name)} will know, once, that you let a memory go, though not what it said.'}</p>`
- `sources/mnemos/mnemos/companion/page/shared/room.js:528` `<div class="acts"><span class="hint${notice ? ' flag' : ''}" aria-live="polite">${esc(notice || '')}</span>`
- `sources/mnemos/mnemos/companion/page/shared/room.js:529` `<button class="cancel" type="button" data-mem-cancel="1">Keep it</button>`
- `sources/mnemos/mnemos/companion/page/shared/room.js:530` `<button class="send danger" type="button" data-forget-yes="${esc(m.id)}"${this.sending === key ? ' disabled' : ''}>${this.sending === key ? 'Letting go…' : 'Let it go'}</button></div></div>';`
- `sources/mnemos/mnemos/companion/page/shared/room.js:531` `}`
- `sources/mnemos/mnemos/companion/page/shared/room.js:532` `return '<div class="mem-acts">${!shared || m.editable ? '<button class="reply-open" type="button" data-mem-act="correct" data-id="${esc(m.id)}">Correct it</button>' : ''}`
- `sources/mnemos/mnemos/companion/page/shared/room.js:533` `<button class="reply-open" type="button" data-mem-act="forget" data-id="${esc(m.id)}">Let it go</button></div>';`
- `sources/mnemos/mnemos/companion/page/shared/room.js:534` `}`
- `sources/mnemos/mnemos/companion/page/shared/room.js:535` `async memChange(id, request) {`
- `sources/mnemos/mnemos/companion/page/shared/room.js:536` `if (this.s.tab === 'shared') request = { ...request, command: 'shared', action: request.command };`
- `sources/mnemos/mnemos/companion/page/shared/room.js:537` `const key = 'fix:${id}';`
- `sources/mnemos/mnemos/companion/page/shared/room.js:538` `if (this.sending) return;`
- `sources/mnemos/mnemos/companion/page/shared/room.js:539` `this.sending = key; this.notices.delete(key); this.render();`
- `sources/mnemos/mnemos/companion/page/shared/room.js:540` `let answer;`
- `sources/mnemos/mnemos/companion/page/shared/room.js:541` `try { answer = await this.hooks.send(request); }`
- `sources/mnemos/mnemos/companion/page/shared/room.js:542` `catch { answer = { error: 'It couldn’t be changed just now. Is 'mnemos log' still running?' }; }`
- `sources/mnemos/mnemos/companion/page/shared/room.js:543` `this.sending = null;`
- `sources/mnemos/mnemos/companion/page/shared/room.js:544` `if (answer?.result) {`
- `sources/mnemos/mnemos/companion/page/shared/room.js:545` `this.editing = null; this.drafts.delete(key);`
- `sources/mnemos/mnemos/companion/page/shared/room.js:546` `const next = request.command === 'correct' ? answer.result.id || answer.result.memory?.id : null;`
- `sources/mnemos/mnemos/companion/page/shared/room.js:547` `const { byId } = this.ctx();`
- `sources/mnemos/mnemos/companion/page/shared/room.js:548` `return this.go({ tab: this.s.tab, memory: next && byId.has(next) ? next : null });`
- `sources/mnemos/mnemos/companion/page/shared/room.js:549` `}`
- `sources/mnemos/mnemos/companion/page/shared/room.js:587` `<dt>Its memory</dt><dd>${esc(who.store || '')}${life.memory?.storeBytes ? ' · ${(life.memory.storeBytes / 1e6).toFixed(0)} MB' : ''}</dd>`
- `sources/mnemos/mnemos/companion/page/shared/room.js:588` `<dt>Memories</dt><dd>${(life.memory?.count || 0).toLocaleString('en-US')} it can reach${life.memory?.states?.archived ? ', ${life.memory.states.archived.toLocaleString('en-US')} let go' : ''}</dd>`
- `sources/mnemos/mnemos/companion/page/shared/room.js:589` `<dt>Agent</dt><dd>${esc(who.agentId || '')}${who.model ? ' · signs as ${esc(who.name)}' : ''}</dd>`
- `sources/mnemos/mnemos/companion/page/shared/room.js:590` `<dt>You</dt><dd>${who.person ? esc(who.person) : 'not named: start the page with --person-name, or set MNEMOS_PERSON_NAME'}</dd>`
- `sources/mnemos/mnemos/companion/page/shared/room.js:591` `</dl>`
- `sources/mnemos/mnemos/companion/page/shared/room.js:592` `<p class="set-d set-foot">On this machine only. Nothing here leaves it${life.jev?.on && life.jev?.key ? ', except what Jev is sent, above' : ''}.</p>`
- `sources/mnemos/mnemos/companion/page/shared/room.js:618` `if (this.sending || !this.hooks.send) return;`
- `sources/mnemos/mnemos/companion/page/shared/room.js:619` `this.sending = 'jev'; this.notices.delete('jev'); this.render();`
- `sources/mnemos/mnemos/companion/page/shared/room.js:620` `let answer;`
- `sources/mnemos/mnemos/companion/page/shared/room.js:621` `try { answer = await this.hooks.send({ command: 'jev', ...request }); }`
- `sources/mnemos/mnemos/companion/page/shared/room.js:622` `catch { answer = { error: 'It couldn’t be changed just now. Is 'mnemos log' still running?' }; }`
- `sources/mnemos/mnemos/companion/page/shared/room.js:623` `this.sending = null;`
- `sources/mnemos/mnemos/companion/page/shared/room.js:645` `if (!h || this.sending || !this.hooks.send) return;`
- `sources/mnemos/mnemos/companion/page/shared/room.js:646` `this.sending = 'hours'; this.notices.delete('hours'); this.render();`
- `sources/mnemos/mnemos/companion/page/shared/room.js:647` `let answer;`
- `sources/mnemos/mnemos/companion/page/shared/room.js:648` `try { answer = await this.hooks.send({ command: 'hours', on: h.on, mine: h.mine, ours: h.ours, dream: h.dream }); }`
- `sources/mnemos/mnemos/companion/page/shared/room.js:649` `catch { answer = { error: 'It couldn’t be changed just now. Is 'mnemos log' still running?' }; }`
- `sources/mnemos/mnemos/companion/page/shared/room.js:650` `this.sending = null;`
- `sources/mnemos/mnemos/companion/page/shared/room.js:671` `const id = fold.dataset.fold, open = !this.whole.has(id);`
- `sources/mnemos/mnemos/companion/page/shared/room.js:672` `if (open) this.whole.add(id); else this.whole.delete(id);`
- `sources/mnemos/mnemos/companion/page/shared/room.js:673` `const text = this.texts.get(id) || '';`
- `sources/mnemos/mnemos/companion/page/shared/room.js:674` `fold.previousElementSibling.textContent = open ? text : opening(text);`
- `sources/mnemos/mnemos/companion/page/shared/room.js:675` `fold.setAttribute('aria-expanded', open); fold.textContent = open ? 'Show less' : 'Read the whole entry';`
- `sources/mnemos/mnemos/companion/page/shared/room.js:676` `return;`
- `sources/mnemos/mnemos/companion/page/shared/room.js:679` `if (open) {`
- `sources/mnemos/mnemos/companion/page/shared/room.js:680` `this.opened.add(open.dataset.reply); this.closed.delete(open.dataset.reply); this.render();`
- `sources/mnemos/mnemos/companion/page/shared/room.js:681` `return this.find('[data-draft="${CSS.escape(open.dataset.reply)}"]')?.focus();`
- `sources/mnemos/mnemos/companion/page/shared/room.js:682` `}`
- `sources/mnemos/mnemos/companion/page/shared/room.js:683` `const cancel = e.target.closest('[data-cancel]');`
- `sources/mnemos/mnemos/companion/page/shared/room.js:684` `if (cancel) { const k = cancel.dataset.cancel; this.opened.delete(k); this.closed.add(k); this.drafts.delete(k); this.notices.delete(k); return this.render(); }`
- `sources/mnemos/mnemos/companion/page/shared/room.js:685` `const send = e.target.closest('[data-send]'); if (send) return this.send(send.dataset.send);`
- `sources/mnemos/mnemos/companion/page/shared/room.js:686` `const act = e.target.closest('[data-mem-act]');`
- `sources/mnemos/mnemos/companion/page/shared/room.js:687` `if (act) {`
- `sources/mnemos/mnemos/companion/page/shared/room.js:688` `this.editing = { id: act.dataset.id, mode: act.dataset.memAct }; this.render();`
- `sources/mnemos/mnemos/companion/page/shared/room.js:690` `}`
- `sources/mnemos/mnemos/companion/page/shared/room.js:691` `if (e.target.closest('[data-mem-cancel]')) { const id = this.editing?.id; if (id) this.drafts.delete('fix:${id}'); this.editing = null; this.render(); if (this.s.tab === 'shared' && id) this.find('[data-mem-act][data-id="${CSS.escape(id)}"]')?.focus(); return; }`
- `sources/mnemos/mnemos/companion/page/shared/room.js:692` `const fix = e.target.closest('[data-fix-save]');`
- `sources/mnemos/mnemos/companion/page/shared/room.js:693` `if (fix) { const id = fix.dataset.fixSave; return this.memChange(id, { command: 'correct', memory_id: id, text: (this.drafts.get('fix:${id}') || '').trim() }); }`
- `sources/mnemos/mnemos/companion/page/shared/room.js:694` `const yes = e.target.closest('[data-forget-yes]');`
- `sources/mnemos/mnemos/companion/page/shared/room.js:695` `if (yes) { const id = yes.dataset.forgetYes; return this.memChange(id, { command: 'forget', memory_id: id, confirmed: true }); }`
- `sources/mnemos/mnemos/companion/page/shared/room.js:696` `const tellSw = e.target.closest('[data-tell]');`
- `sources/mnemos/mnemos/companion/page/shared/room.js:697` `if (tellSw) { const { who, life } = this.ctx(); return setTelling(who.agentId, tellSw.dataset.tell === 'on', life.notes).then(() => this.render()); }`
- `sources/mnemos/mnemos/companion/page/shared/room.js:698` `const sw = e.target.closest('[data-hours-on]');`
- `sources/mnemos/mnemos/companion/page/shared/room.js:699` `if (sw) { this.hours = { ...this.currentHours(), on: sw.dataset.hoursOn === 'on' }; this.notices.delete('hours'); return this.render(); }`
- `sources/mnemos/mnemos/companion/page/shared/room.js:700` `if (e.target.closest('[data-hours-reset]')) { this.hours = null; this.notices.delete('hours'); return this.render(); }`
- `sources/mnemos/mnemos/companion/page/shared/room.js:701` `if (e.target.closest('[data-hours-save]')) return this.saveHours();`
- `sources/mnemos/mnemos/companion/page/shared/room.js:702` `const jevSw = e.target.closest('[data-jev-on]');`
- `sources/mnemos/mnemos/companion/page/shared/room.js:703` `if (jevSw) return this.jevSend({ on: jevSw.dataset.jevOn === 'on' });`
- `sources/mnemos/mnemos/companion/page/shared/room.js:704` `if (e.target.closest('[data-jev-remove]')) return this.jevSend({ remove_key: true });`
- `sources/mnemos/mnemos/companion/page/shared/room.js:758` `// keep open disclosures open across the live redraws`
- `sources/mnemos/mnemos/companion/page/shared/room.js:759` `body.addEventListener('toggle', (e) => { const d = e.target; if (d.dataset?.key) { if (d.open) this.whole.add(d.dataset.key); else this.whole.delete(d.dataset.key); } }, true);`
- `sources/mnemos/mnemos/companion/page/shared/room.js:760` `const touch = (e) => { const m = e.target.closest('[data-mem]'); this.hooks.hover?.(m ? m.dataset.mem : null); };`
- `sources/mnemos/mnemos/companion/page/shared/room.js:761` `body.addEventListener('pointerover', touch);`
- `sources/mnemos/mnemos/companion/page/shared/room.js:762` `body.addEventListener('focusin', touch);`
- `sources/mnemos/mnemos/companion/page/shared/room.js:763` `body.addEventListener('pointerleave', () => this.hooks.hover?.(null));`

**operation and route call sites**, body `sources/mnemos/mnemos/companion/page/shared/specimens.js:105`; body evidence.

- `sources/mnemos/mnemos/companion/page/shared/specimens.js:104` `const ctx = canvas.getContext('2d');`
- `sources/mnemos/mnemos/companion/page/shared/specimens.js:105` `if (again) engraved.delete(model.cells[i].id);   // opened afresh, it's drawn afresh`
- `sources/mnemos/mnemos/companion/page/shared/specimens.js:106` `let hover = null, start = instant || engraved.has(model.cells[i].id) || reduce ? -1e9 : performance.now(), raf = 0;`
- `sources/mnemos/mnemos/companion/page/shared/specimens.js:107` `engraved.add(model.cells[i].id);`
- `sources/mnemos/mnemos/companion/page/shared/specimens.js:108` `const draw = (now) => {`
- `sources/mnemos/mnemos/companion/page/shared/specimens.js:109` `const progress = Math.max(0, Math.min(1, (now - start) / ENGRAVE));`

**operation and route call sites**, body `sources/mnemos/mnemos/companion/page/shared/words.js:178`; body evidence.

- `sources/mnemos/mnemos/companion/page/shared/words.js:177` `none: 'No shared memories match.', more: 'Load more',`
- `sources/mnemos/mnemos/companion/page/shared/words.js:178` `correction: 'Your earlier words stay in the archive.',`
- `sources/mnemos/mnemos/companion/page/shared/words.js:179` `withdraw: "It won't come back by its words, its meaning or its id. It's kept in the archive, never deleted.",`
- `sources/mnemos/mnemos/companion/page/shared/words.js:180` `failed: "Shared memory couldn't be changed just now.",`
- `sources/mnemos/mnemos/companion/page/shared/words.js:181` `};`

**forget**, body `sources/mnemos/mnemos/companion/server.py:190`; implementation/call-through.

- `sources/mnemos/mnemos/companion/server.py:189` `def forget(self) -> None:`
- `sources/mnemos/mnemos/companion/server.py:190` `with self.lock:`

**correct**, body `sources/mnemos/mnemos/interface/person.py:352`; implementation/call-through.

- `sources/mnemos/mnemos/interface/person.py:351` `def correct(self, request: Mapping[str, Any]) -> Any:`
- `sources/mnemos/mnemos/interface/person.py:352` `memory_id, text = _words(request, "memory_id"), _words(request, "text")`

**forget**, body `sources/mnemos/mnemos/interface/person.py:365`; implementation/call-through.

- `sources/mnemos/mnemos/interface/person.py:364` `def forget(self, request: Mapping[str, Any]) -> Any:`
- `sources/mnemos/mnemos/interface/person.py:370` `return writer.forget_memory(memory_id, person=self._as_person())`

**_config_invalidate**, body `sources/mnemos/mnemos/mcp_server.py:524`; implementation/call-through.

- `sources/mnemos/mnemos/mcp_server.py:522` `def _config_invalidate():`
- `sources/mnemos/mnemos/mcp_server.py:524` `global _config`

**mnemos_forget**, body `sources/mnemos/mnemos/mcp_server.py:1456`; implementation/call-through.

- `sources/mnemos/mnemos/mcp_server.py:1446` `@mcp.tool()`
- `sources/mnemos/mnemos/mcp_server.py:1447` `def mnemos_forget(engram_id: str) -> str:`
- `sources/mnemos/mnemos/mcp_server.py:1464` `_store.archive_engram(engram, reason="user_requested")  # type: ignore`
- `sources/mnemos/mnemos/mcp_server.py:1465` `return f"Archived: {engram_id}\n  Content was: {engram.content[:100]}..."`

**correct**, body `sources/mnemos/mnemos/shared_memory.py:178`; implementation/call-through.

- `sources/mnemos/mnemos/shared_memory.py:177` `def correct(target_id, correction, *, action, agent_id, model):`
- `sources/mnemos/mnemos/shared_memory.py:178` `return _correct(target_id, correction, action=action, agent_id=agent_id, model=model)`

**_archive_rows**, body `sources/mnemos/mnemos/simple_runtime.py:7944`; implementation/call-through.

- `sources/mnemos/mnemos/simple_runtime.py:7940` `def _archive_rows(rows: int) -> int:`
- `sources/mnemos/mnemos/simple_runtime.py:7944` `return max(0, min(rows, RECALL_RESULTS, RECALL_ROWS_WITH_ARCHIVE - rows))`

**_format_archived**, body `sources/mnemos/mnemos/simple_runtime.py:8260`; implementation/call-through.

- `sources/mnemos/mnemos/simple_runtime.py:8257` `def _format_archived(engram: Engram, query: str = "") -> str:`
- `sources/mnemos/mnemos/simple_runtime.py:8260` `return (`

**repair_archive_rows**, body `sources/mnemos/mnemos/simple_runtime.py:2201`; implementation/call-through.

- `sources/mnemos/mnemos/simple_runtime.py:2189` `def repair_archive_rows(self, *, write: bool = False) -> dict[str, Any]:`
- `sources/mnemos/mnemos/simple_runtime.py:2214` `"AS same FROM archive a LEFT JOIN engrams e ON e.id = a.id ORDER BY a.id"`
- `sources/mnemos/mnemos/simple_runtime.py:2217` `"rows": len(rows), "archived": 0, "gone": 0,`
- `sources/mnemos/mnemos/simple_runtime.py:2223` `elif row["state"] == "archived":`
- `sources/mnemos/mnemos/simple_runtime.py:2224` `found["archived"] += 1`
- `sources/mnemos/mnemos/simple_runtime.py:2235` `# repeats a memory that is not archived.`
- `sources/mnemos/mnemos/simple_runtime.py:2237` `"DELETE FROM archive WHERE id = ? AND EXISTS (SELECT 1 FROM engrams e "`
- `sources/mnemos/mnemos/simple_runtime.py:2238` `"WHERE e.id = archive.id AND e.state != 'archived' "`
- `sources/mnemos/mnemos/simple_runtime.py:2239` `"AND e.content = archive.content "`
- `sources/mnemos/mnemos/simple_runtime.py:2240` `"AND e.content_at_encoding = archive.content_at_encoding)",`
- `sources/mnemos/mnemos/simple_runtime.py:2246` `"archive-rows", plan, write=write, find=find,`

**_forget_pair**, body `sources/mnemos/mnemos/simple_runtime.py:6097`; implementation/call-through.

- `sources/mnemos/mnemos/simple_runtime.py:6076` `def _forget_pair(`
- `sources/mnemos/mnemos/simple_runtime.py:6097` `assert self._store is not None`

**correct**, body `sources/mnemos/mnemos/simple_runtime.py:6637`; implementation/call-through.

- `sources/mnemos/mnemos/simple_runtime.py:6591` `@_notice_when_older`
- `sources/mnemos/mnemos/simple_runtime.py:6592` `@_traced("correct")`
- `sources/mnemos/mnemos/simple_runtime.py:6593` `def correct(`
- `sources/mnemos/mnemos/simple_runtime.py:6650` `return "Correction needs replacement text or a forget/archive action."`
- `sources/mnemos/mnemos/simple_runtime.py:6697` `lines = [f"Nothing was archived: {label.lower()} {current} was already archived."]`
- `sources/mnemos/mnemos/simple_runtime.py:6699` `lines = [f"Archived continuity note {current}."]`
- `sources/mnemos/mnemos/simple_runtime.py:6701` `lines.append(f"Its memory {', '.join(retired_memories)} was archived with it.")`
- `sources/mnemos/mnemos/simple_runtime.py:6703` `lines = [f"Archived memory {current}."]`
- `sources/mnemos/mnemos/simple_runtime.py:6705` `lines.append(f"Its continuity note {', '.join(retired_notes)} was archived with it.")`
- `sources/mnemos/mnemos/simple_runtime.py:6725` `f"Archived memory {current} and captured correction {result['engram_id']}.",`
- `sources/mnemos/mnemos/simple_runtime.py:6741` `# archived a note about a reading.`
- `sources/mnemos/mnemos/simple_runtime.py:6784` `lines = [f"Archived closest continuity note {match['id']}."]`
- `sources/mnemos/mnemos/simple_runtime.py:6786` `lines.append(f"Its memory {', '.join(retired_memories)} was archived with it.")`
- `sources/mnemos/mnemos/simple_runtime.py:6821` `lines = [f"Archived closest matching memory {engram.id}."]`
- `sources/mnemos/mnemos/simple_runtime.py:6823` `lines.append(f"Its continuity note {', '.join(retired_notes)} was archived with it.")`
- `sources/mnemos/mnemos/simple_runtime.py:6828` `return "Nothing was archived: give the ID as target_id, or a query that names it."`
- `sources/mnemos/mnemos/simple_runtime.py:6830` `f'Nothing was archived: no note or memory matched "{search_text}" '`

**forget_memory**, body `sources/mnemos/mnemos/simple_runtime.py:6973`; implementation/call-through.

- `sources/mnemos/mnemos/simple_runtime.py:6955` `@_traced("correct")`
- `sources/mnemos/mnemos/simple_runtime.py:6956` `def forget_memory(`
- `sources/mnemos/mnemos/simple_runtime.py:6989` `raise NothingChanged(f"Memory {engram.id} is already archived.")`
- `sources/mnemos/mnemos/simple_runtime.py:6992` `"state": "archived",`
- `sources/mnemos/mnemos/simple_runtime.py:6993` `"forgotten": utc_iso(self._store.archived_at(engram.id)),`

**bulk_archive**, body `sources/mnemos/mnemos/store/archive.py:40`; implementation/call-through.

- `sources/mnemos/mnemos/store/archive.py:23` `def bulk_archive(`
- `sources/mnemos/mnemos/store/archive.py:40` `stats = {"archived": 0, "not_found": 0, "already_archived": 0}`
- `sources/mnemos/mnemos/store/archive.py:45` `elif engram.state == "archived":`
- `sources/mnemos/mnemos/store/archive.py:46` `stats["already_archived"] += 1`
- `sources/mnemos/mnemos/store/archive.py:48` `store.archive_engram(engram, reason=reason)`
- `sources/mnemos/mnemos/store/archive.py:49` `stats["archived"] += 1`

**get_archive_stats**, body `sources/mnemos/mnemos/store/archive.py:113`; implementation/call-through.

- `sources/mnemos/mnemos/store/archive.py:101` `def get_archive_stats(store: EngramStore) -> dict[str, Any]:`
- `sources/mnemos/mnemos/store/archive.py:115` `"""SELECT COUNT(*) AS total, MIN(archived_at) AS oldest,`
- `sources/mnemos/mnemos/store/archive.py:116` `MAX(archived_at) AS newest FROM archive"""`
- `sources/mnemos/mnemos/store/archive.py:119` `row["archive_reason"]: row["count"]`
- `sources/mnemos/mnemos/store/archive.py:121` `"SELECT archive_reason, COUNT(*) AS count FROM archive GROUP BY archive_reason"`
- `sources/mnemos/mnemos/store/archive.py:127` `"SELECT kind, COUNT(*) AS count FROM archive GROUP BY kind"`
- `sources/mnemos/mnemos/store/archive.py:131` `"total_archived": summary["total"],`
- `sources/mnemos/mnemos/store/archive.py:134` `"oldest_archived": summary["oldest"],`
- `sources/mnemos/mnemos/store/archive.py:135` `"newest_archived": summary["newest"],`

**delete_engram**, body `sources/mnemos/mnemos/store/sqlite_store.py:2435`; implementation/call-through.

- `sources/mnemos/mnemos/store/sqlite_store.py:2433` `def delete_engram(self, engram_id: str) -> None:`
- `sources/mnemos/mnemos/store/sqlite_store.py:2436` `conn.execute("DELETE FROM engrams WHERE id = ?", (engram_id,))`
- `sources/mnemos/mnemos/store/sqlite_store.py:2437` `conn.execute("DELETE FROM engrams_fts WHERE id = ?", (engram_id,))`
- `sources/mnemos/mnemos/store/sqlite_store.py:2439` `"DELETE FROM connections WHERE source_id = ? OR target_id = ?",`
- `sources/mnemos/mnemos/store/sqlite_store.py:2442` `conn.execute("DELETE FROM versions WHERE engram_id = ?", (engram_id,))`

**archive_engram**, body `sources/mnemos/mnemos/store/sqlite_store.py:2883`; implementation/call-through.

- `sources/mnemos/mnemos/store/sqlite_store.py:2881` `def archive_engram(self, engram: Engram, reason: str = "low_accessibility") -> None:`
- `sources/mnemos/mnemos/store/sqlite_store.py:2887` `"INSERT OR REPLACE INTO archive "`
- `sources/mnemos/mnemos/store/sqlite_store.py:2889` `"archived_at, archive_reason, final_accessibility) "`
- `sources/mnemos/mnemos/store/sqlite_store.py:2903` `conn.execute("UPDATE engrams SET state = 'archived' WHERE id = ?", (engram.id,))`
- `sources/mnemos/mnemos/store/sqlite_store.py:2904` `conn.execute("DELETE FROM engrams_fts WHERE id = ?", (engram.id,))`
- `sources/mnemos/mnemos/store/sqlite_store.py:2905` `# An unanswered reflection request about an archived memory is a dead`
- `sources/mnemos/mnemos/store/sqlite_store.py:2909` `"DELETE FROM reflection_queue WHERE target_id = ? AND answered_at IS NULL",`

**search_archive**, body `sources/mnemos/mnemos/store/sqlite_store.py:2915`; implementation/call-through.

- `sources/mnemos/mnemos/store/sqlite_store.py:2913` `def search_archive(self, query: str, limit: int = 20) -> list[dict]:`
- `sources/mnemos/mnemos/store/sqlite_store.py:2917` `"SELECT * FROM archive WHERE content LIKE ? OR content_at_encoding LIKE ? "`

**archive_reason**, body `sources/mnemos/mnemos/store/sqlite_store.py:2925`; implementation/call-through.

- `sources/mnemos/mnemos/store/sqlite_store.py:2923` `def archive_reason(self, engram_id: str) -> str | None:`
- `sources/mnemos/mnemos/store/sqlite_store.py:2926` `"SELECT a.archive_reason FROM archive a JOIN engrams e ON e.id = a.id "`
- `sources/mnemos/mnemos/store/sqlite_store.py:2927` `"WHERE a.id = ? AND e.state = 'archived'",`

**archived_at**, body `sources/mnemos/mnemos/store/sqlite_store.py:2935`; implementation/call-through.

- `sources/mnemos/mnemos/store/sqlite_store.py:2932` `def archived_at(self, engram_id: str) -> str | None:`
- `sources/mnemos/mnemos/store/sqlite_store.py:2936` `"SELECT a.archived_at FROM archive a JOIN engrams e ON e.id = a.id "`
- `sources/mnemos/mnemos/store/sqlite_store.py:2937` `"WHERE a.id = ? AND e.state = 'archived'",`

**archive_hypomnema_entry**, body `sources/mnemos/mnemos/store/sqlite_store.py:4256`; implementation/call-through.

- `sources/mnemos/mnemos/store/sqlite_store.py:4240` `def archive_hypomnema_entry(`
- `sources/mnemos/mnemos/store/sqlite_store.py:4257` `raise ValueError("Archive reason cannot be empty")`
- `sources/mnemos/mnemos/store/sqlite_store.py:4274` `"reason": f"archived: {reason.strip()}",`

**purge_stale_reflections**, body `sources/mnemos/mnemos/store/sqlite_store.py:4941`; implementation/call-through.

- `sources/mnemos/mnemos/store/sqlite_store.py:4930` `def purge_stale_reflections(self) -> int:`
- `sources/mnemos/mnemos/store/sqlite_store.py:4944` `DELETE FROM reflection_queue`
- `sources/mnemos/mnemos/store/sqlite_store.py:4946` `AND target_id NOT IN (SELECT id FROM engrams WHERE state != 'archived')`

**purge_retired_reflections**, body `sources/mnemos/mnemos/store/sqlite_store.py:4962`; implementation/call-through.

- `sources/mnemos/mnemos/store/sqlite_store.py:4951` `def purge_retired_reflections(self) -> int:`
- `sources/mnemos/mnemos/store/sqlite_store.py:4973` `f"DELETE FROM reflection_queue WHERE answered_at IS NULL AND kind IN ({kinds})",`

</details>

## 3. What reaches the agent

System wake context takes up to four results and truncates only if max_wake_chars is nonzero (`sources/mnemos/mnemos/integrations/hermes/provider.py:315`). Default max_wake_chars=0 means no explicit wake-character bound; recall defaults max_context_chars=2200 (`sources/mnemos/mnemos/integrations/hermes/scope.py:211`). Prefetch builds a cue block and uses that bound; cache is revision-sensitive (`sources/mnemos/mnemos/integrations/hermes/provider.py:339`). Tool results are separately limited JSON envelopes (`sources/mnemos/mnemos/integrations/hermes/provider.py:397`).

Host assembly: `hermes-agent/agent/memory_manager.py:352` joins provider system blocks. `hermes-agent/agent/turn_context.py:1081` requests prefetch for nontrivial turns; `hermes-agent/agent/turn_context.py:120` fences it; `hermes-agent/agent/turn_context.py:136` appends it to API-bound user content. Tool results are a third channel. Optional host spill is at `hermes-agent/agent/memory_manager.py:419`, not a universal provider cap. Built-in memory remains separately controlled (`hermes-agent/agent/system_prompt.py:542`).

<details>
<summary>Context and tool-result assembly quotes</summary>

`system_prompt_block`, body `sources/mnemos/mnemos/integrations/hermes/provider.py:312`:

- `sources/mnemos/mnemos/integrations/hermes/provider.py:315` `packet = self._runtime.context(max_results=min(4, self._config.max_recall_results))`
- `sources/mnemos/mnemos/integrations/hermes/provider.py:317` `wake_limit = self._config.max_wake_chars`
- `sources/mnemos/mnemos/integrations/hermes/provider.py:318` `return _truncate(body, wake_limit) if wake_limit else body`

`prefetch`, body `sources/mnemos/mnemos/integrations/hermes/provider.py:322`:

- `sources/mnemos/mnemos/integrations/hermes/provider.py:326` `return ""`
- `sources/mnemos/mnemos/integrations/hermes/provider.py:329` `return ""`
- `sources/mnemos/mnemos/integrations/hermes/provider.py:334` `return cached[2]`
- `sources/mnemos/mnemos/integrations/hermes/provider.py:335` `recalled = self._runtime._recall_own(clean, max_results=self._config.max_recall_results)`
- `sources/mnemos/mnemos/integrations/hermes/provider.py:337` `return ""`
- `sources/mnemos/mnemos/integrations/hermes/provider.py:339` `context = _truncate(f"{_CUE_HEADING}\n{_CUE_NOTE}\n\n{rows}", self._config.max_context_chars)`
- `sources/mnemos/mnemos/integrations/hermes/provider.py:350` `return context`

`queue_prefetch`, body `sources/mnemos/mnemos/integrations/hermes/provider.py:354`:



`get_tool_schemas`, body `sources/mnemos/mnemos/integrations/hermes/provider.py:384`:

- `sources/mnemos/mnemos/integrations/hermes/provider.py:384` `return [`

`handle_tool_call`, body `sources/mnemos/mnemos/integrations/hermes/provider.py:392`:

- `sources/mnemos/mnemos/integrations/hermes/provider.py:394` `return _json_error(f"Unknown Mnemos tool: {tool_name}")`
- `sources/mnemos/mnemos/integrations/hermes/provider.py:397` `return _json_ok(result=limit_output(tool(self, args or {})))`
- `sources/mnemos/mnemos/integrations/hermes/provider.py:400` `return _json_error(str(exc), tool=tool_name)`

`_tool_capture`, body `sources/mnemos/mnemos/integrations/hermes/provider.py:563`:

- `sources/mnemos/mnemos/integrations/hermes/provider.py:563` `return self._capture(`
- `sources/mnemos/mnemos/integrations/hermes/provider.py:564` `text_arg("content", _cleaned(args, "content"), MAX_CAPTURE_CHARS, required=True),`
- `sources/mnemos/mnemos/integrations/hermes/provider.py:565` `text_arg("context", _cleaned(args, "context"), MAX_CONTEXT_CHARS),`
- `sources/mnemos/mnemos/integrations/hermes/provider.py:567` `impact=text_arg("impact", _words(args, "impact"), MAX_REFLECTION_CHARS),`
- `sources/mnemos/mnemos/integrations/hermes/provider.py:568` `signed_as=text_arg("signed_as", _words(args, "signed_as"), MAX_ID_CHARS),`

`_tool_recall`, body `sources/mnemos/mnemos/integrations/hermes/provider.py:574`:

- `sources/mnemos/mnemos/integrations/hermes/provider.py:576` `if args.get("max_results") is not None:`
- `sources/mnemos/mnemos/integrations/hermes/provider.py:577` `options["max_results"] = self._bounded_int(args.get("max_results"))`
- `sources/mnemos/mnemos/integrations/hermes/provider.py:578` `return self._runtime.recall(`
- `sources/mnemos/mnemos/integrations/hermes/provider.py:579` `text_arg("query", _cleaned(args, "query"), MAX_QUERY_CHARS, required=not standing),`

`_tool_correct`, body `sources/mnemos/mnemos/integrations/hermes/provider.py:587`:

- `sources/mnemos/mnemos/integrations/hermes/provider.py:588` `return self._runtime.correct(`
- `sources/mnemos/mnemos/integrations/hermes/provider.py:589` `correction=text_arg("correction", _cleaned(args, "correction"), MAX_CAPTURE_CHARS),`
- `sources/mnemos/mnemos/integrations/hermes/provider.py:590` `target_id=text_arg("target_id", _cleaned(args, "target_id"), MAX_ID_CHARS),`
- `sources/mnemos/mnemos/integrations/hermes/provider.py:591` `query=text_arg("query", _cleaned(args, "query"), MAX_QUERY_CHARS),`
- `sources/mnemos/mnemos/integrations/hermes/provider.py:593` `impact=text_arg("impact", _words(args, "impact"), MAX_REFLECTION_CHARS),`
- `sources/mnemos/mnemos/integrations/hermes/provider.py:594` `signed_as=text_arg("signed_as", _words(args, "signed_as"), MAX_ID_CHARS),`

`_tool_handoff`, body `sources/mnemos/mnemos/integrations/hermes/provider.py:599`:

- `sources/mnemos/mnemos/integrations/hermes/provider.py:599` `return self._runtime.handoff(`
- `sources/mnemos/mnemos/integrations/hermes/provider.py:600` `text_arg("text", _words(args, "text"), MAX_HANDOFF_CHARS, required=True),`
- `sources/mnemos/mnemos/integrations/hermes/provider.py:601` `signed_as=text_arg("signed_as", _words(args, "signed_as"), MAX_ID_CHARS),`

`_tool_journal`, body `sources/mnemos/mnemos/integrations/hermes/provider.py:605`:

- `sources/mnemos/mnemos/integrations/hermes/provider.py:605` `return self._runtime.journal(`
- `sources/mnemos/mnemos/integrations/hermes/provider.py:606` `text_arg("text", _words(args, "text"), MAX_JOURNAL_CHARS),`
- `sources/mnemos/mnemos/integrations/hermes/provider.py:607` `signed_as=text_arg("signed_as", _words(args, "signed_as"), MAX_ID_CHARS),`
- `sources/mnemos/mnemos/integrations/hermes/provider.py:608` `mood=text_arg("mood", _words(args, "mood"), MAX_MOOD_CHARS),`

`_tool_note`, body `sources/mnemos/mnemos/integrations/hermes/provider.py:612`:

- `sources/mnemos/mnemos/integrations/hermes/provider.py:612` `return self._runtime.note(`
- `sources/mnemos/mnemos/integrations/hermes/provider.py:613` `text_arg("text", _words(args, "text"), MAX_NOTE_CHARS, required=True),`
- `sources/mnemos/mnemos/integrations/hermes/provider.py:614` `text_arg("kind", _words(args, "kind"), MAX_ID_CHARS),`
- `sources/mnemos/mnemos/integrations/hermes/provider.py:615` `signed_as=text_arg("signed_as", _words(args, "signed_as"), MAX_ID_CHARS),`
- `sources/mnemos/mnemos/integrations/hermes/provider.py:616` `in_reply_to=text_arg("in_reply_to", _words(args, "in_reply_to"), MAX_ID_CHARS),`

`_tool_reflect`, body `sources/mnemos/mnemos/integrations/hermes/provider.py:620`:

- `sources/mnemos/mnemos/integrations/hermes/provider.py:620` `return self._runtime.reflect(`
- `sources/mnemos/mnemos/integrations/hermes/provider.py:621` `target_id=text_arg("target_id", _words(args, "target_id"), MAX_ID_CHARS, required=True),`
- `sources/mnemos/mnemos/integrations/hermes/provider.py:622` `text=text_arg("text", _words(args, "text"), MAX_REFLECTION_CHARS, required=True),`
- `sources/mnemos/mnemos/integrations/hermes/provider.py:624` `signed_as=text_arg("signed_as", _words(args, "signed_as"), MAX_ID_CHARS),`

`_tool_introduce`, body `sources/mnemos/mnemos/integrations/hermes/provider.py:628`:

- `sources/mnemos/mnemos/integrations/hermes/provider.py:628` `return self._runtime.introduce(`

`_tool_maintain`, body `sources/mnemos/mnemos/integrations/hermes/provider.py:634`:

- `sources/mnemos/mnemos/integrations/hermes/provider.py:636` `return result`

`_tool_health`, body `sources/mnemos/mnemos/integrations/hermes/provider.py:639`:

- `sources/mnemos/mnemos/integrations/hermes/provider.py:639` `return format_health_card(self._runtime.health())`

`_tool_page`, body `sources/mnemos/mnemos/integrations/hermes/provider.py:642`:

- `sources/mnemos/mnemos/integrations/hermes/provider.py:656` `return said`

</details>

## 4. Data access boundaries

Direct SQLite access `yes`: default `$HERMES_HOME/mnemos/mnemos.db`, overridable with config/MNEMOS_DB_PATH (`sources/mnemos/mnemos/integrations/hermes/scope.py:302`). Accountless local mode `yes`, optional model keys do not change that basic store path. No mandatory credential file; optional keys use host configuration. Telemetry defaults beyond visible local/runtime code `unknown`; exact env/egress evidence below.

Boundary model: same OS user, terminal, mounted filesystem and allowed network. Direct reachability means another client can use the same file/endpoint and available credentials, not unauthenticated access or encryption bypass. Hermes setup persists declared secrets through `hermes-agent/hermes_cli/memory_setup.py:321` and `hermes-agent/hermes_cli/config.py:2697`. These are possible configured paths; no live credentials were read.

Host credential-file detail: `get_hermes_home()/.env` is resolved at `hermes-agent/hermes_cli/config.py:473`. New files are secured to 0600 at `hermes-agent/hermes_cli/config.py:506`; the writer preserves an existing mode at `hermes-agent/hermes_cli/config.py:2544` and `:2562`. Therefore 0600 is not an unconditional mode claim for every existing profile.

<details>
<summary>Paths, environment, permissions, egress, telemetry and process-launch evidence</summary>

- `sources/mnemos/hermes-plugin/plugin.yaml:6` `homepage: https://github.com/Riley-Coyote/mnemos-mcp`

- `sources/mnemos/mnemos/config/defaults.py:29` `"db_path": "~/.mnemos/memory.db",`

- `sources/mnemos/mnemos/config/loader.py:10` `MNEMOS_STORE_DB_PATH -> config["store"]["db_path"]`
- `sources/mnemos/mnemos/config/loader.py:57` `config_path = Path.home() / ".mnemos" / "config.json"`
- `sources/mnemos/mnemos/config/loader.py:80` `config_path = Path.home() / ".mnemos" / "config.json"`
- `sources/mnemos/mnemos/config/loader.py:84` `private_root = Path.home() / ".mnemos"`
- `sources/mnemos/mnemos/config/loader.py:130` `for key, value in os.environ.items():`

- `sources/mnemos/mnemos/integrations/hermes/installer.py:101` `f"    os.environ.setdefault('MNEMOS_COMMAND', {mnemos_command()!r})\n\n"`
- `sources/mnemos/mnemos/integrations/hermes/installer.py:168` `proc = subprocess.run(`
- `sources/mnemos/mnemos/integrations/hermes/installer.py:171` `except subprocess.TimeoutExpired:`
- `sources/mnemos/mnemos/integrations/hermes/installer.py:173` `except (OSError, subprocess.SubprocessError) as exc:`
- `sources/mnemos/mnemos/integrations/hermes/installer.py:380` `db_path: str | None = None,`
- `sources/mnemos/mnemos/integrations/hermes/installer.py:425` `if _no_scope_chosen(home, db_path, agent_id, person_id, project_scope):`
- `sources/mnemos/mnemos/integrations/hermes/installer.py:426` `db_path, agent_id, person_id, project_scope = _one_memory_scope(home)`
- `sources/mnemos/mnemos/integrations/hermes/installer.py:428` `db_path = db_path or kept.db_path`
- `sources/mnemos/mnemos/integrations/hermes/installer.py:434` `if db_path and older.exists() and Path(db_path).expanduser().resolve() != older.resolve():`
- `sources/mnemos/mnemos/integrations/hermes/installer.py:485` `"db_path": db_path,`
- `sources/mnemos/mnemos/integrations/hermes/installer.py:505` `db_path=db_path,`
- `sources/mnemos/mnemos/integrations/hermes/installer.py:605` `home: Path, db_path: str | None, agent_id: str | None, person_id: str | None, project_scope: str | None,`
- `sources/mnemos/mnemos/integrations/hermes/installer.py:610` `if any((db_path, agent_id, person_id, project_scope)):`
- `sources/mnemos/mnemos/integrations/hermes/installer.py:613` `return not (kept.db_path or kept.agent_id) and not (home / "mnemos" / "mnemos.db").exists()`
- `sources/mnemos/mnemos/integrations/hermes/installer.py:629` `scope = resolve_scope(agent_id=profile_agent_id(home), db_path=profile_store(home))`
- `sources/mnemos/mnemos/integrations/hermes/installer.py:630` `return str(scope.db_path), scope.agent_id, scope.person_id, scope.project_scope`
- `sources/mnemos/mnemos/integrations/hermes/installer.py:645` `if raw.get("db_path") or raw.get("agent_id"):`
- `sources/mnemos/mnemos/integrations/hermes/installer.py:647` `if os.environ.get("MNEMOS_DB_PATH") or os.environ.get("MNEMOS_AGENT_ID"):`
- `sources/mnemos/mnemos/integrations/hermes/installer.py:651` `db_path, agent_id, person_id, project_scope = _one_memory_scope(home)`
- `sources/mnemos/mnemos/integrations/hermes/installer.py:654` `"db_path": db_path,`
- `sources/mnemos/mnemos/integrations/hermes/installer.py:676` `db_path=named("--db-path"),`
- `sources/mnemos/mnemos/integrations/hermes/installer.py:686` `db_path: str | None = None,`
- `sources/mnemos/mnemos/integrations/hermes/installer.py:709` `db_path=db_path,`
- `sources/mnemos/mnemos/integrations/hermes/installer.py:789` `db_path = Path(_sidecar_scope(mcp_server).db_path).expanduser() if mode == "sidecar" else _configured_db_path(home)`
- `sources/mnemos/mnemos/integrations/hermes/installer.py:790` `damage = _store_damage(db_path)`
- `sources/mnemos/mnemos/integrations/hermes/installer.py:794` `tools_store = Path(_sidecar_scope(mcp_server).db_path).expanduser()`
- `sources/mnemos/mnemos/integrations/hermes/installer.py:795` `if tools_store.resolve() != db_path.resolve():`
- `sources/mnemos/mnemos/integrations/hermes/installer.py:818` `"db_path": str(db_path),`
- `sources/mnemos/mnemos/integrations/hermes/installer.py:819` `"db_exists": db_path.exists(),`
- `sources/mnemos/mnemos/integrations/hermes/installer.py:839` `f"Database:    {payload['db_path']} ({_store_state(payload)})",`
- `sources/mnemos/mnemos/integrations/hermes/installer.py:852` `f"Two memories: the provider keeps {payload['db_path']}, and the tools keep "`
- `sources/mnemos/mnemos/integrations/hermes/installer.py:983` `its_memory = Path(_sidecar_scope(server).db_path).expanduser()`
- `sources/mnemos/mnemos/integrations/hermes/installer.py:984` `return None if its_memory.resolve() == _configured_db_path(home).resolve() else its_memory`
- `sources/mnemos/mnemos/integrations/hermes/installer.py:1011` `db_path: str | None = None,`
- `sources/mnemos/mnemos/integrations/hermes/installer.py:1024` `db_path=db_path,`
- `sources/mnemos/mnemos/integrations/hermes/installer.py:1104` `db_path: str | None = None,`
- `sources/mnemos/mnemos/integrations/hermes/installer.py:1110` `if db_path:`
- `sources/mnemos/mnemos/integrations/hermes/installer.py:1111` `args.extend(["--db-path", db_path])`
- `sources/mnemos/mnemos/integrations/hermes/installer.py:1225` `def _configured_db_path(home: Path) -> Path:`
- `sources/mnemos/mnemos/integrations/hermes/installer.py:1226` `raw = HermesMnemosConfig.load(home).db_path`
- `sources/mnemos/mnemos/integrations/hermes/installer.py:1297` `default = Path.home() / ".hermes"`
- `sources/mnemos/mnemos/integrations/hermes/installer.py:1390` `return os.environ.get("HERMES_MEMORY_PROVIDER", "")`

- `sources/mnemos/mnemos/integrations/hermes/provider.py:147` `get_secret = os.environ.get`
- `sources/mnemos/mnemos/integrations/hermes/provider.py:288` `db_path=self._scope.db_path,`
- `sources/mnemos/mnemos/integrations/hermes/provider.py:301` `Path(self._scope.db_path).expanduser().parent.mkdir(parents=True, exist_ok=True)`
- `sources/mnemos/mnemos/integrations/hermes/provider.py:510` `{"key": "db_path", "description": "Mnemos identity-continuity SQLite path", "default": "$HERMES_HOME/mnemos/mnemos.db"},`
- `sources/mnemos/mnemos/integrations/hermes/provider.py:535` `store = Path(scope.db_path).expanduser()`
- `sources/mnemos/mnemos/integrations/hermes/provider.py:646` `"db_path": scope.db_path,`

- `sources/mnemos/mnemos/integrations/hermes/scope.py:58` `raw = os.environ.get("HERMES_HOME", "").strip()`
- `sources/mnemos/mnemos/integrations/hermes/scope.py:59` `return Path(raw).expanduser() if raw else Path.home() / ".hermes"`
- `sources/mnemos/mnemos/integrations/hermes/scope.py:80` `home = Path.home() / ".hermes" / "profiles" / name`
- `sources/mnemos/mnemos/integrations/hermes/scope.py:94` `default = Path.home() / ".hermes"`
- `sources/mnemos/mnemos/integrations/hermes/scope.py:107` `return str(Path.home() / ".mnemos" / f"{profile_agent_id(hermes_home)}.db")`
- `sources/mnemos/mnemos/integrations/hermes/scope.py:132` `updates.update(agent_id=profile_agent_id(home), db_path=profile_store(home))`
- `sources/mnemos/mnemos/integrations/hermes/scope.py:203` `db_path: str | None = None`
- `sources/mnemos/mnemos/integrations/hermes/scope.py:220` `raw = {**raw, "agent_id": profile_agent_id(home), "db_path": profile_store(home)}`
- `sources/mnemos/mnemos/integrations/hermes/scope.py:222` `db_path=str(raw.get("db_path") or "").strip() or None,`
- `sources/mnemos/mnemos/integrations/hermes/scope.py:237` `"db_path": self.db_path,`
- `sources/mnemos/mnemos/integrations/hermes/scope.py:258` `db_path: str`
- `sources/mnemos/mnemos/integrations/hermes/scope.py:281` `or os.environ.get("MNEMOS_AGENT_ID")`
- `sources/mnemos/mnemos/integrations/hermes/scope.py:288` `or os.environ.get("MNEMOS_PERSON_ID")`
- `sources/mnemos/mnemos/integrations/hermes/scope.py:296` `or os.environ.get("MNEMOS_PROJECT_SCOPE")`
- `sources/mnemos/mnemos/integrations/hermes/scope.py:302` `explicit_db = cfg.db_path or os.environ.get("MNEMOS_DB_PATH")`
- `sources/mnemos/mnemos/integrations/hermes/scope.py:303` `db_path = explicit_db or str(home / "mnemos" / "mnemos.db")`
- `sources/mnemos/mnemos/integrations/hermes/scope.py:304` `db_path = db_path.replace("$HERMES_HOME", str(home)).replace("${HERMES_HOME}", str(home))`
- `sources/mnemos/mnemos/integrations/hermes/scope.py:311` `db_path=db_path,`

</details>

## 5. Reproducibility

`range`: mnemos-continuity>=0.6.7,<0.7 (`sources/mnemos/hermes-plugin/pyproject.toml:12`). It is not a fixed engine patch version.

Retained lock/checksum files: none in snapshot. Upstream completeness is unknown: fetch.py:15 excludes .lock files, fetch.py:12 excludes build/vendor directories. Absence here is not absence upstream.

- `sources/mnemos/hermes-plugin/plugin.yaml:2` `version: "0.6.7"`
- `sources/mnemos/hermes-plugin/plugin.yaml:7` `requires_hermes: ">=0.23.0"`

- `sources/mnemos/hermes-plugin/pyproject.toml:3` `version = "0.6.7"`
- `sources/mnemos/hermes-plugin/pyproject.toml:5` `requires-python = ">=3.11"`
- `sources/mnemos/hermes-plugin/pyproject.toml:7` `# Hermes installs these into its own environment, under its own exact pins, when the`
- `sources/mnemos/hermes-plugin/pyproject.toml:8` `# plugin is installed or enabled, and again after 'hermes update'. The range keeps to`
- `sources/mnemos/hermes-plugin/pyproject.toml:10` `# reviewed and what Hermes installs stay close: 0.6.7 hands the upkeep process only what it needs.`
- `sources/mnemos/hermes-plugin/pyproject.toml:11` `dependencies = [`
- `sources/mnemos/hermes-plugin/pyproject.toml:12` `"mnemos-continuity>=0.6.7,<0.7",`

- `sources/mnemos/mnemos/integrations/hermes/installer.py:1` `"""Installer and diagnostics for the Hermes Mnemos provider shim."""`
- `sources/mnemos/mnemos/integrations/hermes/installer.py:31` `PLUGIN_VERSION = "0.3.0"`
- `sources/mnemos/mnemos/integrations/hermes/installer.py:41` `# How hermes-plugin/__init__.py opens: Mnemos as Hermes's plugin catalog installs it.`
- `sources/mnemos/mnemos/integrations/hermes/installer.py:64` `"""Where this Mnemos and what it imports are installed: usually one`
- `sources/mnemos/mnemos/integrations/hermes/installer.py:65` `site-packages, or a checkout and a site-packages for an editable install."""`
- `sources/mnemos/mnemos/integrations/hermes/installer.py:175` `if proc.returncode == 0:`
- `sources/mnemos/mnemos/integrations/hermes/installer.py:198` `if loads == "no":`
- `sources/mnemos/mnemos/integrations/hermes/installer.py:208` `f"version: {PLUGIN_VERSION}\n"`
- `sources/mnemos/mnemos/integrations/hermes/installer.py:221` `"""Return the installed plugin readme."""`
- `sources/mnemos/mnemos/integrations/hermes/installer.py:226` `"implementation lives in the installed 'mnemos-continuity' Python package.\n\n"`
- `sources/mnemos/mnemos/integrations/hermes/installer.py:238` `class HermesInstallResult:`
- `sources/mnemos/mnemos/integrations/hermes/installer.py:257` `mode_label = "Provider Mode" if self.mode == PROVIDER_MODE else "Sidecar Mode"`
- `sources/mnemos/mnemos/integrations/hermes/installer.py:259` `"Mnemos Hermes install",`
- `sources/mnemos/mnemos/integrations/hermes/installer.py:263` `if self.mode == PROVIDER_MODE or self.files_written:`
- `sources/mnemos/mnemos/integrations/hermes/installer.py:278` `elif self.mode == SIDECAR_MODE:`
- `sources/mnemos/mnemos/integrations/hermes/installer.py:280` `if self.mode == PROVIDER_MODE:`
- `sources/mnemos/mnemos/integrations/hermes/installer.py:281` `if self.activated and self.hermes_loads == "no":`
- `sources/mnemos/mnemos/integrations/hermes/installer.py:293` `if self.mode == PROVIDER_MODE and not self.activated:`
- `sources/mnemos/mnemos/integrations/hermes/installer.py:295` `if self.mode == SIDECAR_MODE:`
- `sources/mnemos/mnemos/integrations/hermes/installer.py:305` `install_result: HermesInstallResult | None = None`
- `sources/mnemos/mnemos/integrations/hermes/installer.py:313` `if self.install_result is None:`
- `sources/mnemos/mnemos/integrations/hermes/installer.py:319` `and not self.install_result.mcp_configured`
- `sources/mnemos/mnemos/integrations/hermes/installer.py:320` `and self.install_result.active_provider != PLUGIN_NAME`
- `sources/mnemos/mnemos/integrations/hermes/installer.py:326` `mode_label = "Provider Mode" if self.mode == PROVIDER_MODE else "Sidecar Mode"`
- `sources/mnemos/mnemos/integrations/hermes/installer.py:328` `install = self.install_result`
- `sources/mnemos/mnemos/integrations/hermes/installer.py:334` `if install and install.mcp_configured:`
- `sources/mnemos/mnemos/integrations/hermes/installer.py:347` `if self.mode == SIDECAR_MODE:`
- `sources/mnemos/mnemos/integrations/hermes/installer.py:353` `f"Provider shim: {'installed' if provider_shim_ready else 'not installed'}",`
- `sources/mnemos/mnemos/integrations/hermes/installer.py:356` `if install and install.files_written:`
- `sources/mnemos/mnemos/integrations/hermes/installer.py:357` `lines.append("Changed files: " + ", ".join(str(path) for path in install.files_written))`
- `sources/mnemos/mnemos/integrations/hermes/installer.py:358` `if install and install.mcp_config_path:`
- `sources/mnemos/mnemos/integrations/hermes/installer.py:359` `mcp_status = "updated" if install.mcp_configured else "not changed"`
- `sources/mnemos/mnemos/integrations/hermes/installer.py:360` `lines.append(f"Hermes config: {install.mcp_config_path} ({mcp_status})")`
- `sources/mnemos/mnemos/integrations/hermes/installer.py:370` `elif self.mode == SIDECAR_MODE:`
- `sources/mnemos/mnemos/integrations/hermes/installer.py:377` `def install_hermes_plugin(`
- `sources/mnemos/mnemos/integrations/hermes/installer.py:391` `) -> HermesInstallResult:`
- `sources/mnemos/mnemos/integrations/hermes/installer.py:392` `"""Install the Hermes user memory-provider shim under ''$HERMES_HOME''."""`
- `sources/mnemos/mnemos/integrations/hermes/installer.py:402` `should_configure_mcp = selected_mode == SIDECAR_MODE if configure_mcp is None else bool(configure_mcp)`
- `sources/mnemos/mnemos/integrations/hermes/installer.py:403` `result = HermesInstallResult(`
- `sources/mnemos/mnemos/integrations/hermes/installer.py:412` `if selected_mode == SIDECAR_MODE and agent_safe and result.active_provider == PLUGIN_NAME:`
- `sources/mnemos/mnemos/integrations/hermes/installer.py:439` `if selected_mode == PROVIDER_MODE:`
- `sources/mnemos/mnemos/integrations/hermes/installer.py:444` `"installed it, and Hermes updates it (hermes plugins update mnemos)."`
- `sources/mnemos/mnemos/integrations/hermes/installer.py:476` `if selected_mode == PROVIDER_MODE and not dry_run and not theirs:`
- `sources/mnemos/mnemos/integrations/hermes/installer.py:478` `if result.hermes_loads == "no":`
- `sources/mnemos/mnemos/integrations/hermes/installer.py:521` `if selected_mode == SIDECAR_MODE:`
- `sources/mnemos/mnemos/integrations/hermes/installer.py:524` `if result.active_provider == PLUGIN_NAME:`
- `sources/mnemos/mnemos/integrations/hermes/installer.py:549` `result.activated = result.active_provider == PLUGIN_NAME`
- `sources/mnemos/mnemos/integrations/hermes/installer.py:552` `selected_mode == PROVIDER_MODE`
- `sources/mnemos/mnemos/integrations/hermes/installer.py:553` `and result.active_provider == PLUGIN_NAME`
- `sources/mnemos/mnemos/integrations/hermes/installer.py:558` `if result.hermes_loads == "no":`
- `sources/mnemos/mnemos/integrations/hermes/installer.py:575` `"""Whether a plugin folder holds Mnemos as Hermes's plugin catalog installed`
- `sources/mnemos/mnemos/integrations/hermes/installer.py:636` `memory in every chat. A plugin installed from Hermes's catalog starts this`
- `sources/mnemos/mnemos/integrations/hermes/installer.py:696` `"""Install Mnemos for Hermes with safe defaults and an immediate doctor pass."""`
- `sources/mnemos/mnemos/integrations/hermes/installer.py:703` `activate = mode == PROVIDER_MODE`
- `sources/mnemos/mnemos/integrations/hermes/installer.py:707` `install = install_hermes_plugin(`
- `sources/mnemos/mnemos/integrations/hermes/installer.py:714` `configure_mcp=mode == SIDECAR_MODE,`
- `sources/mnemos/mnemos/integrations/hermes/installer.py:720` `warnings.extend(install.warnings)`
- `sources/mnemos/mnemos/integrations/hermes/installer.py:729` `if not install.mcp_configured and after_provider != PLUGIN_NAME:`
- `sources/mnemos/mnemos/integrations/hermes/installer.py:732` `restart_likely = bool(install.files_written or install.mcp_configured or install.activated)`
- `sources/mnemos/mnemos/integrations/hermes/installer.py:735` `mode=install.mode,`
- `sources/mnemos/mnemos/integrations/hermes/installer.py:737` `install_result=install,`
- `sources/mnemos/mnemos/integrations/hermes/installer.py:779` `loads, load_problem = _hermes_load_check(home) if active == PLUGIN_NAME else ("", "")`
- `sources/mnemos/mnemos/integrations/hermes/installer.py:780` `if active == PLUGIN_NAME:`
- `sources/mnemos/mnemos/integrations/hermes/installer.py:785` `mode = "provider-installed"`
- `sources/mnemos/mnemos/integrations/hermes/installer.py:789` `db_path = Path(_sidecar_scope(mcp_server).db_path).expanduser() if mode == "sidecar" else _configured_db_path(home)`
- `sources/mnemos/mnemos/integrations/hermes/installer.py:793` `if active == PLUGIN_NAME and mcp_server and _is_mnemos_server(mcp_server):`
- `sources/mnemos/mnemos/integrations/hermes/installer.py:808` `"provider_mode_active": active == PLUGIN_NAME,`
- `sources/mnemos/mnemos/integrations/hermes/installer.py:822` `"ready": not damage and not other_memory and (loads != "no" if active == PLUGIN_NAME else mcp_configured),`
- `sources/mnemos/mnemos/integrations/hermes/installer.py:823` `"restart_likely_needed": active == PLUGIN_NAME or mcp_configured or provider_shim_ready,`
- `sources/mnemos/mnemos/integrations/hermes/installer.py:841` `f"Restart:     {'likely after install/config changes' if payload['restart_likely_needed'] else 'not likely'}",`
- `sources/mnemos/mnemos/integrations/hermes/installer.py:855` `if payload["provider_mode_active"] and payload.get("hermes_loads_mnemos") == "no":`
- `sources/mnemos/mnemos/integrations/hermes/installer.py:858` `"Run 'mnemos connect hermes --write' from where Mnemos is installed now."`
- `sources/mnemos/mnemos/integrations/hermes/installer.py:879` `if not path.is_file() or path.stat().st_size == 0:`
- `sources/mnemos/mnemos/integrations/hermes/installer.py:887` `return "" if said == "ok" else str(said)`
- `sources/mnemos/mnemos/integrations/hermes/installer.py:904` `if mode == "provider":`
- `sources/mnemos/mnemos/integrations/hermes/installer.py:906` `if mode == "sidecar":`
- `sources/mnemos/mnemos/integrations/hermes/installer.py:908` `if mode == "provider-installed":`
- `sources/mnemos/mnemos/integrations/hermes/installer.py:909` `return "Provider shim installed, not active"`
- `sources/mnemos/mnemos/integrations/hermes/installer.py:934` `return Path(str(server.get("command") or "")).stem == "mnemos" and "serve" in args`
- `sources/mnemos/mnemos/integrations/hermes/installer.py:953` `def _retire_own_server(home: Path, server_name: str, result: HermesInstallResult, backed_up: set[Path]) -> None:`
- `sources/mnemos/mnemos/integrations/hermes/installer.py:984` `return None if its_memory.resolve() == _configured_db_path(home).resolve() else its_memory`
- `sources/mnemos/mnemos/integrations/hermes/installer.py:1070` `same_server = all(existing_server.get(key) == server[key] for key in ("command", "args"))`
- `sources/mnemos/mnemos/integrations/hermes/installer.py:1086` `if server == existing_server:`
- `sources/mnemos/mnemos/integrations/hermes/installer.py:1190` `if stripped == "mcp_servers:":`
- `sources/mnemos/mnemos/integrations/hermes/installer.py:1196` `in_server = stripped[:-1] == server_name`
- `sources/mnemos/mnemos/integrations/hermes/installer.py:1201` `if stripped == "args:":`
- `sources/mnemos/mnemos/integrations/hermes/installer.py:1217` `if raw == "{}":`
- `sources/mnemos/mnemos/integrations/hermes/installer.py:1258` `if holder == PLUGIN_NAME:`
- `sources/mnemos/mnemos/integrations/hermes/installer.py:1264` `"'mnemos hermes install --activate --force' replaces it."`
- `sources/mnemos/mnemos/integrations/hermes/installer.py:1319` `if _read_active_provider(config_path) == PLUGIN_NAME:`
- `sources/mnemos/mnemos/integrations/hermes/installer.py:1382` `in_memory = line.split("#", 1)[0].strip() == "memory:"`

## 6. Tests

Hermes integration/correction tests and runtime forget tests are inventoried separately. Forget semantics are archive/state changes, so a test called delete is not physical-erasure proof. Hermes integration test covers externally forgotten memory and recall/cache behavior. (`sources/mnemos/tests/test_hermes_integration.py:610`).

Adapter-related declarations: **199**. Python AST test_ functions, fixtures excluded; literal TS/JS it/test, Go Test*, Rust test_* declarations. No parametrization expansion, collection, execution, or coverage measurement. Other rows are engine test candidates selected by mutation names; names alone do not verify forget coverage.

| file | declarations | adapter-related | forget-related declarations |
|---|---:|---|---|
| `sources/mnemos/tests/test_belief_edge_correction.py:1` | 11 | no | `test_forget_retires_an_agent_belief` `sources/mnemos/tests/test_belief_edge_correction.py:106` |
| `sources/mnemos/tests/test_belief_verdicts.py:1` | 26 | no | `test_retire_sets_confidence_to_zero_and_deletes_nothing` `sources/mnemos/tests/test_belief_verdicts.py:396` |
| `sources/mnemos/tests/test_edges_hermes_review.py:1` | 7 | yes | `test_review_note_write_invalidates_prefetch` `sources/mnemos/tests/test_edges_hermes_review.py:89` |
| `sources/mnemos/tests/test_forget_is_complete.py:1` | 13 | no | `test_forgetting_by_engram_id_deactivates_its_revised_continuity_note` `sources/mnemos/tests/test_forget_is_complete.py:147`; `test_the_persons_forget_takes_it_too` `sources/mnemos/tests/test_forget_is_complete.py:281`; `test_forgetting_a_lesson_by_its_id_says_it_was_archived` `sources/mnemos/tests/test_forget_is_complete.py:337` |
| `sources/mnemos/tests/test_forget_needs_a_match.py:1` | 6 | no | `test_forgetting_what_no_note_mentions_archives_nothing` `sources/mnemos/tests/test_forget_needs_a_match.py:48`; `test_forgetting_what_the_words_name_archives_that_note` `sources/mnemos/tests/test_forget_needs_a_match.py:75` |
| `sources/mnemos/tests/test_hermes_catalog_plugin.py:1` | 9 | yes | none |
| `sources/mnemos/tests/test_hermes_config_care.py:1` | 26 | yes | none |
| `sources/mnemos/tests/test_hermes_conversations.py:1` | 56 | yes | none |
| `sources/mnemos/tests/test_hermes_integration.py:1` | 30 | yes | `test_prefetch_sees_another_process_forget` `sources/mnemos/tests/test_hermes_integration.py:610` |
| `sources/mnemos/tests/test_hermes_keys_come_from_hermes.py:1` | 6 | yes | none |
| `sources/mnemos/tests/test_hermes_names_model_and_place.py:1` | 6 | yes | none |
| `sources/mnemos/tests/test_hermes_profiles.py:1` | 20 | yes | none |
| `sources/mnemos/tests/test_hermes_provider_is_read_from_the_memory_block.py:1` | 1 | yes | none |
| `sources/mnemos/tests/test_hermes_round2.py:1` | 3 | yes | `test_external_forget_during_recall_does_not_pin_stale_cache` `sources/mnemos/tests/test_hermes_round2.py:41` |
| `sources/mnemos/tests/test_hermes_same_tools.py:1` | 26 | yes | none |
| `sources/mnemos/tests/test_hermes_writes_nothing_in_its_place.py:1` | 9 | yes | none |
| `sources/mnemos/tests/test_lessons_forget_repros.py:1` | 3 | no | `test_forgetting_a_lesson_id_reports_what_it_actually_did` `sources/mnemos/tests/test_lessons_forget_repros.py:48`; `test_forget_keeps_a_lesson_with_natural_independent_support` `sources/mnemos/tests/test_lessons_forget_repros.py:61` |
| `sources/mnemos/tests/test_one_capture_one_object.py:1` | 26 | no | `test_correcting_then_forgetting_leaves_nothing_live` `sources/mnemos/tests/test_one_capture_one_object.py:300`; `test_forgetting_a_summary_note_by_its_id_leaves_the_memory_it_references` `sources/mnemos/tests/test_one_capture_one_object.py:766` |
| `sources/mnemos/tests/test_parallel_handoffs.py:1` | 27 | no | `test_forgetting_by_query_never_removes_a_handoff` `sources/mnemos/tests/test_parallel_handoffs.py:688` |
| `sources/mnemos/tests/test_simple_runtime.py:1` | 25 | no | `test_query_only_forget_archives_closest_continuity_and_memory` `sources/mnemos/tests/test_simple_runtime.py:161` |
| `sources/mnemos/tests/test_small_follow_ups.py:1` | 24 | no | `test_reports_older_than_the_newest_five_are_retired_never_deleted` `sources/mnemos/tests/test_small_follow_ups.py:704` |
| `sources/mnemos/tests/test_still_true.py:1` | 16 | no | `test_no_longer_true_lets_it_go_archived_never_deleted` `sources/mnemos/tests/test_still_true.py:203` |
| `sources/mnemos/tests/test_store.py:1` | 10 | no | `test_delete_engram` `sources/mnemos/tests/test_store.py:89` |
| `sources/mnemos/tests/test_the_person_corrects_and_forgets.py:1` | 14 | no | `test_the_persons_forget_archives_the_pair_and_writes_nothing_in_its_place` `sources/mnemos/tests/test_the_person_corrects_and_forgets.py:196`; `test_an_agents_forget_never_counts_as_the_persons` `sources/mnemos/tests/test_the_person_corrects_and_forgets.py:268`; `test_in_an_agents_shell_correct_and_forget_stay_the_agents` `sources/mnemos/tests/test_the_person_corrects_and_forgets.py:365` |

## 7. Corrections to B2

Old values below are copied from the triage row (TRIAGE.md), not reconstructed from memory. A refinement narrows an over-broad field; it need not reverse its original value. Counts are distinct field paths, not the number of factual errors.

- `forget_tool.v`: `"update_only"` → yes: mnemos_correct has explicit forget/archive/remove/delete actions, with cache invalidation; these archive rather than erase. Evidence: `sources/mnemos/mnemos/integrations/hermes/provider.py:588`, `sources/mnemos/mnemos/simple_runtime.py:6692`.
- `context.bounds_status`: `"Source expressions include recall and other output bounds. They are not interchangeable: token vs character limits and static block vs tool output must be read at their cited call sites. No single total bound is established unless explicitly stated."` → Recall default is 2200 chars; system wake has no explicit cap by default (max_wake_chars=0). Evidence: `sources/mnemos/mnemos/integrations/hermes/scope.py:211`, `sources/mnemos/mnemos/integrations/hermes/provider.py:318`.
- `engine_delete.scope`: `"Named deletion/retraction endpoint or method, not a guarantee of physical erasure. Its reach may overlap an agent tool; lack of a native tool is counted separately."` → `yes` for logical forgetting, correcting B2 update_only: `mnemos_correct(action='forget'/'archive'/'remove'/'delete')` dispatches runtime.correct and clears prefetch cache (`sources/mnemos/mnemos/integrations/hermes/provider.py:587`; `sources/mnemos/mnemos/integrations/hermes/provider.py:592`). Runtime routes forget actions to _forget_pair, retiring note/engram together and handling linked lessons (`sources/mnemos/mnemos/simple_runtime.py:6647`; `sources/mnemos/mnemos/simple_runtime.py:6692`). CLI/runtime forget_memory uses the same archive path (`sources/mnemos/mnemos/simple_runtime.py:6981`). `partial`: archived records/version history remain; this is retrieval exclusion, not row erasure. Scope and lineage handling do not purge host state. Evidence: `sources/mnemos/mnemos/integrations/hermes/provider.py:587`, `sources/mnemos/mnemos/integrations/hermes/provider.py:592`, `sources/mnemos/mnemos/simple_runtime.py:6647`, `sources/mnemos/mnemos/simple_runtime.py:6692`, `sources/mnemos/mnemos/simple_runtime.py:6981`.
- `context.mechanism.v`: `"Source expressions below identify retrieval, cached/static returns, lifecycle capture, and limits. A hook definition is not a guarantee it emits nonempty context each turn; branches and configuration apply."` → System wake context takes up to four results and truncates only if max_wake_chars is nonzero (`sources/mnemos/mnemos/integrations/hermes/provider.py:315`). Default max_wake_chars=0 means no explicit wake-character bound; recall defaults max_context_chars=2200 (`sources/mnemos/mnemos/integrations/hermes/scope.py:211`). Prefetch builds a cue block and uses that bound; cache is revision-sensitive (`sources/mnemos/mnemos/integrations/hermes/provider.py:339`). Tool results are separately limited JSON envelopes (`sources/mnemos/mnemos/integrations/hermes/provider.py:397`). Evidence: `sources/mnemos/mnemos/integrations/hermes/provider.py:315`, `sources/mnemos/mnemos/integrations/hermes/scope.py:211`, `sources/mnemos/mnemos/integrations/hermes/provider.py:339`, `sources/mnemos/mnemos/integrations/hermes/provider.py:397`.
- `sec.telemetry.v`: `"unknown: no emitter identified in the selected adapter files; external dependencies not cleared"` → Direct SQLite access `yes`: default `$HERMES_HOME/mnemos/mnemos.db`, overridable with config/MNEMOS_DB_PATH (`sources/mnemos/mnemos/integrations/hermes/scope.py:302`). Accountless local mode `yes`, optional model keys do not change that basic store path. No mandatory credential file; optional keys use host configuration. Telemetry defaults beyond visible local/runtime code `unknown`; exact env/egress evidence below. Evidence: `sources/mnemos/mnemos/integrations/hermes/scope.py:302`.
- `repro.decls`: `["hermes-plugin/plugin.yaml:6", "hermes-plugin/plugin.yaml:9", "hermes-plugin/plugin.yaml:10", "hermes-plugin/plugin.yaml:11", "hermes-plugin/plugin.yaml:12", "hermes-plugin/pyproject.toml:3", "hermes-plugin/pyproject.toml:5", "hermes-plugin/pyproject.toml:11", "hermes-plugin/pyproject.toml:12", "pyproject.toml:7", "pyproject.toml:11", "pyproject.toml:27", "pyproject.toml:42", "pyproject.toml:43", "pyproject.toml:46", "pyproject.toml:49", "pyproject.toml:58", "pyproject.toml:64", "pyproject.toml:66", "pyproject.toml:68", "pyproject.toml:69", "pyproject.toml:70", "pyproject.toml:73", "pyproject.toml:74", "pyproject.toml:75", "pyproject.toml:77", "pyproject.toml:106", "pyproject.toml:107", "pyproject.toml:108", "pyproject.toml:109", "pyproject.toml:110"]` → `range`: mnemos-continuity>=0.6.7,<0.7 (`sources/mnemos/hermes-plugin/pyproject.toml:12`). It is not a fixed engine patch version. Evidence: `sources/mnemos/hermes-plugin/pyproject.toml:12`.
- `tests.count`: `1982` → 199 adapter-related declarations using the stated parser/scope; B2 counted a different, repository-wide inventory. This is a scope refinement, not a count of executed or passing tests. Hermes integration/correction tests and runtime forget tests are inventoried separately. Forget semantics are archive/state changes, so a test called delete is not physical-erasure proof. Hermes integration test covers externally forgotten memory and recall/cache behavior. (`sources/mnemos/tests/test_hermes_integration.py:610`). Evidence: `sources/mnemos/tests/test_hermes_integration.py:610`.

MATRIX counts these 7 explicit field corrections/refinements. Sections 1-6 provide the body and test inventories. Test-name candidates are not passing tests or measured coverage.
