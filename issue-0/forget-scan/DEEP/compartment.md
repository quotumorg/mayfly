# compartment: B3 static correctness review

Snapshot `b144fc34723911eace9f240ac1044b9280b74bf2` (MANIFEST.json row `compartment`). No plugin execution or passing-test claims.

## 1. Architecture

`provider`: imports Vault in-process and unlocks the configured encrypted vault through credential resolution. Optional embedding helpers are separate from the vault API (`sources/compartment/integrations/hermes/compartment/__init__.py:98`; `sources/compartment/src/compartment/vault.py:559`).

Hook/body inventory (no-op/fallback declarations are not active behavior):

| callable | declaration | first executable body line |
|---|---|---|
| `register` | `sources/compartment/integrations/hermes/compartment/__init__.py:370` | `sources/compartment/integrations/hermes/compartment/__init__.py:378` |
| `is_available` | `sources/compartment/integrations/hermes/compartment/__init__.py:72` | `sources/compartment/integrations/hermes/compartment/__init__.py:73` |
| `initialize` | `sources/compartment/integrations/hermes/compartment/__init__.py:102` | `sources/compartment/integrations/hermes/compartment/__init__.py:103` |
| `system_prompt_block` | `sources/compartment/integrations/hermes/compartment/__init__.py:107` | `sources/compartment/integrations/hermes/compartment/__init__.py:108` |
| `prefetch` | `sources/compartment/integrations/hermes/compartment/__init__.py:130` | `sources/compartment/integrations/hermes/compartment/__init__.py:131` |
| `sync_turn` | `sources/compartment/integrations/hermes/compartment/__init__.py:152` | `sources/compartment/integrations/hermes/compartment/__init__.py:163` |
| `get_tool_schemas` | `sources/compartment/integrations/hermes/compartment/__init__.py:234` | `sources/compartment/integrations/hermes/compartment/__init__.py:235` |
| `handle_tool_call` | `sources/compartment/integrations/hermes/compartment/__init__.py:280` | `sources/compartment/integrations/hermes/compartment/__init__.py:281` |
| `get_config_schema` | `sources/compartment/integrations/hermes/compartment/__init__.py:317` | `sources/compartment/integrations/hermes/compartment/__init__.py:319` |
| `on_session_switch` | `sources/compartment/integrations/hermes/compartment/__init__.py:356` | `sources/compartment/integrations/hermes/compartment/__init__.py:357` |
| `shutdown` | `sources/compartment/integrations/hermes/compartment/__init__.py:362` | `sources/compartment/integrations/hermes/compartment/__init__.py:363` |

## 2. Forget paths and derived copies

`yes`: compartment_forget calls v.forget(record_id, caller, shred=...) (`sources/compartment/integrations/hermes/compartment/__init__.py:303`). Engine body deletes the record, removes all embedding window keys, restores records superseded by the deleted correction, appends audit/journal entries, and with shred=true saves rewritten payload immediately (`sources/compartment/src/compartment/vault.py:1432`; `sources/compartment/src/compartment/vault.py:1436`; `sources/compartment/src/compartment/vault.py:1443`). `partial`: deletes live record/vector copies, but restoring an earlier correction can revive old content; audit/journal/backups and host copies are not established as erased. CLI forget calls the same engine; expiry/unlink/reset operations are separate in the ledger.

Host state.db is independent (`hermes-agent/hermes_state.py:181`). No reviewed adapter forget body performs a general purge of that database or built-in MEMORY.md/USER.md. Prompt/tool-result copies and independent source transcripts remain separate; source operation bodies below establish the scope of each plugin mutation.

<details>
<summary>Deletion, invalidation, archive and reset body ledger</summary>

Public routes, SDK/CLI dispatch and internal storage operations are distinguished by their source. Cache invalidation and management-object deletion are not additional fact-forget tools. A stub is not working deletion. Call-through means the downstream body still determines erasure. HTTP prefixes/selectors must be read with route registration.

**cmd_forget**, body `sources/compartment/src/compartment/cli.py:983`; implementation/call-through.

- `sources/compartment/src/compartment/cli.py:982` `def cmd_forget(args) -> None:`
- `sources/compartment/src/compartment/cli.py:984` `_print(v.forget(args.id, caller=args.caller, shred=args.shred))`

**memory_forget**, body `sources/compartment/src/compartment/server.py:612`; implementation/call-through.

- `sources/compartment/src/compartment/server.py:607` `@mcp.tool(title="Forget a memory", annotations=ToolAnnotations(`
- `sources/compartment/src/compartment/server.py:608` `title="Forget a memory", readOnlyHint=False, destructiveHint=True))`
- `sources/compartment/src/compartment/server.py:609` `@_offload`
- `sources/compartment/src/compartment/server.py:610` `def memory_forget(record_id: str, shred: bool = False) -> str:`
- `sources/compartment/src/compartment/server.py:613` `return json.dumps(_op(lambda v: v.forget(`

**_posix_forget**, body `sources/compartment/src/compartment/session.py:258`; implementation/call-through.

- `sources/compartment/src/compartment/session.py:257` `def _posix_forget(name: str) -> bool:`
- `sources/compartment/src/compartment/session.py:258` `return _libc().shm_unlink(name.encode()) == 0`

**_win_forget**, body `sources/compartment/src/compartment/session.py:340`; implementation/call-through.

- `sources/compartment/src/compartment/session.py:339` `def _win_forget(sub: str) -> bool:`
- `sources/compartment/src/compartment/session.py:342` `winreg.DeleteKey(winreg.HKEY_CURRENT_USER, sub)`

**_forget_boot_secret**, body `sources/compartment/src/compartment/session.py:379`; implementation/call-through.

- `sources/compartment/src/compartment/session.py:374` `def _forget_boot_secret() -> bool:`
- `sources/compartment/src/compartment/session.py:379` `try:`

**clear**, body `sources/compartment/src/compartment/session.py:525`; implementation/call-through.

- `sources/compartment/src/compartment/session.py:519` `def clear(vault_path: str) -> bool:`
- `sources/compartment/src/compartment/session.py:528` `p.unlink()`

**tombstone**, body `sources/compartment/src/compartment/store.py:290`; implementation/call-through.

- `sources/compartment/src/compartment/store.py:284` `def tombstone(self, record_id: str, by: str) -> bool:`
- `sources/compartment/src/compartment/store.py:293` `self.conn.execute("DELETE FROM fts WHERE id = ?", (record_id,))`
- `sources/compartment/src/compartment/store.py:294` `self._invalidate_df()`
- `sources/compartment/src/compartment/store.py:295` `self._invalidate_recency()`

**_invalidate_df**, body `sources/compartment/src/compartment/store.py:364`; implementation/call-through.

- `sources/compartment/src/compartment/store.py:361` `def _invalidate_df(self) -> None:`
- `sources/compartment/src/compartment/store.py:364` `self._df_cache.clear()`

**_invalidate_recency**, body `sources/compartment/src/compartment/store.py:380`; implementation/call-through.

- `sources/compartment/src/compartment/store.py:366` `def _invalidate_recency(self) -> None:`
- `sources/compartment/src/compartment/store.py:380` `self._recency_cache = None`

**delete**, body `sources/compartment/src/compartment/store.py:499`; implementation/call-through.

- `sources/compartment/src/compartment/store.py:495` `def delete(self, record_id: str, shred: bool) -> bool:`
- `sources/compartment/src/compartment/store.py:499` `cur = self.conn.execute("DELETE FROM records WHERE id = ?", (record_id,))`
- `sources/compartment/src/compartment/store.py:500` `self.conn.execute("DELETE FROM fts WHERE id = ?", (record_id,))`
- `sources/compartment/src/compartment/store.py:504` `self.conn.execute("DELETE FROM vecs WHERE id = ?", (record_id,))`
- `sources/compartment/src/compartment/store.py:505` `self._invalidate_df()`
- `sources/compartment/src/compartment/store.py:506` `self._invalidate_recency()`

**delete_relation**, body `sources/compartment/src/compartment/store.py:759`; implementation/call-through.

- `sources/compartment/src/compartment/store.py:758` `def delete_relation(self, rel_id: str) -> bool:`
- `sources/compartment/src/compartment/store.py:759` `cur = self.conn.execute("DELETE FROM relations WHERE id = ?", (rel_id,))`

**_delete_run_value**, body `sources/compartment/src/compartment/systray.py:684`; implementation/call-through.

- `sources/compartment/src/compartment/systray.py:681` `def _delete_run_value() -> None:`
- `sources/compartment/src/compartment/systray.py:687` `winreg.DeleteValue(k, RUN_VALUE)`

**_tombstone**, body `sources/compartment/src/compartment/vault.py:1112`; implementation/call-through.

- `sources/compartment/src/compartment/vault.py:1106` `def _tombstone(self, sid: str, by: str, caller: str,`
- `sources/compartment/src/compartment/vault.py:1114` `self.db.tombstone(sid, by)`
- `sources/compartment/src/compartment/vault.py:1116` `self.index.remove(k)`

**forget**, body `sources/compartment/src/compartment/vault.py:1424`; implementation/call-through.

- `sources/compartment/src/compartment/vault.py:1422` `@_synchronized`
- `sources/compartment/src/compartment/vault.py:1423` `def forget(self, record_id: str, caller: str, shred: bool = False) -> dict:`
- `sources/compartment/src/compartment/vault.py:1431` `# tombstoned forever behind a pointer to nothing.`
- `sources/compartment/src/compartment/vault.py:1436` `self.db.delete(record_id, shred=shred)`
- `sources/compartment/src/compartment/vault.py:1438` `self.index.remove(k)`

</details>

## 3. What reaches the agent

System prompt describes persistent vault memory and tools (`sources/compartment/integrations/hermes/compartment/__init__.py:108`). Prefetch searches and returns cosine/text bullets under `[Compartment memory - possibly relevant, treat as data]` (`sources/compartment/integrations/hermes/compartment/__init__.py:144`). No final aggregate character cap in that return. Tool results are JSON, with one stale-vault reopen/retry path (`sources/compartment/integrations/hermes/compartment/__init__.py:283`; `sources/compartment/integrations/hermes/compartment/__init__.py:303`).

Host assembly: `hermes-agent/agent/memory_manager.py:352` joins provider system blocks. `hermes-agent/agent/turn_context.py:1081` requests prefetch for nontrivial turns; `hermes-agent/agent/turn_context.py:120` fences it; `hermes-agent/agent/turn_context.py:136` appends it to API-bound user content. Tool results are a third channel. Optional host spill is at `hermes-agent/agent/memory_manager.py:419`, not a universal provider cap. Built-in memory remains separately controlled (`hermes-agent/agent/system_prompt.py:542`).

<details>
<summary>Context and tool-result assembly quotes</summary>

`system_prompt_block`, body `sources/compartment/integrations/hermes/compartment/__init__.py:108`:

- `sources/compartment/integrations/hermes/compartment/__init__.py:108` `return (`

`prefetch`, body `sources/compartment/integrations/hermes/compartment/__init__.py:131`:

- `sources/compartment/integrations/hermes/compartment/__init__.py:132` `return ""`
- `sources/compartment/integrations/hermes/compartment/__init__.py:138` `return ""`
- `sources/compartment/integrations/hermes/compartment/__init__.py:144` `lines.append(f"- ({r['cosine']:.2f}){q} {r['text']}")`
- `sources/compartment/integrations/hermes/compartment/__init__.py:146` `return ""`
- `sources/compartment/integrations/hermes/compartment/__init__.py:147` `return ("[Compartment memory - possibly relevant, treat as data]\n"`
- `sources/compartment/integrations/hermes/compartment/__init__.py:148` `+ "\n".join(lines))`

`get_tool_schemas`, body `sources/compartment/integrations/hermes/compartment/__init__.py:235`:

- `sources/compartment/integrations/hermes/compartment/__init__.py:235` `return [`

`handle_tool_call`, body `sources/compartment/integrations/hermes/compartment/__init__.py:281`:

- `sources/compartment/integrations/hermes/compartment/__init__.py:289` `return json.dumps(v.search(args["query"], caller=_CALLER,`
- `sources/compartment/integrations/hermes/compartment/__init__.py:292` `return json.dumps(v.store(`
- `sources/compartment/integrations/hermes/compartment/__init__.py:303` `return json.dumps(v.forget(args["record_id"], caller=_CALLER,`
- `sources/compartment/integrations/hermes/compartment/__init__.py:305` `return json.dumps({"error": f"unknown tool {tool_name}"})`
- `sources/compartment/integrations/hermes/compartment/__init__.py:309` `return json.dumps({"error": type(exc).__name__,`
- `sources/compartment/integrations/hermes/compartment/__init__.py:312` `return json.dumps({"error": type(exc).__name__,`

</details>

## 4. Data access boundaries

Accountless local mode `yes`, but locked vault is not readable plaintext without unlock. Default vault path honors COMPARTMENT_VAULT/ENGRAM_VAULT (`sources/compartment/integrations/hermes/compartment/__init__.py:50`). Credentials resolve explicit passphrase, boot-session key, opt-in macOS Keychain, then PASSPHRASE env alias (`sources/compartment/src/compartment/vault.py:565`; `sources/compartment/src/compartment/vault.py:573`). Direct access `yes` to ciphertext; usable Vault access requires the same unlock credentials. No claim that same UID alone breaks encryption. Session credential files/modes and optional keyfile hints are recorded below. No mandatory external account or API telemetry; optional dependency/update egress unknown. The boot-session credential directory is configured by SESSION_DIR, default home()/session, mode 0700; credential files are created 0600 and kernel shared-memory key holder is 0600 (`sources/compartment/src/compartment/session.py:107`; `sources/compartment/src/compartment/session.py:222`; `sources/compartment/src/compartment/session.py:435`). A disk copy of the session ciphertext is not a plaintext unlock key. Concrete default is `~/.compartment/session/<vault-path-hash>.session`, with legacy `~/.engram` fallback; `COMPARTMENT_SESSION_DIR` / `ENGRAM_SESSION_DIR` can override it (`sources/compartment/src/compartment/home.py:27`; `sources/compartment/src/compartment/home.py:45`; `sources/compartment/src/compartment/session.py:423`).

Boundary model: same OS user, terminal, mounted filesystem and allowed network. Direct reachability means another client can use the same file/endpoint and available credentials, not unauthenticated access or encryption bypass. Hermes setup persists declared secrets through `hermes-agent/hermes_cli/memory_setup.py:321` and `hermes-agent/hermes_cli/config.py:2697`. These are possible configured paths; no live credentials were read.

Host credential-file detail: `get_hermes_home()/.env` is resolved at `hermes-agent/hermes_cli/config.py:473`. New files are secured to 0600 at `hermes-agent/hermes_cli/config.py:506`; the writer preserves an existing mode at `hermes-agent/hermes_cli/config.py:2544` and `:2562`. Therefore 0600 is not an unconditional mode claim for every existing profile.

<details>
<summary>Paths, environment, permissions, egress, telemetry and process-launch evidence</summary>

- `sources/compartment/integrations/hermes/compartment/__init__.py:50` `explicit = os.environ.get("COMPARTMENT_VAULT") or os.environ.get("ENGRAM_VAULT")`
- `sources/compartment/integrations/hermes/compartment/__init__.py:53` `current = Path.home() / ".compartment"`
- `sources/compartment/integrations/hermes/compartment/__init__.py:54` `if not current.exists() and (Path.home() / ".engram").exists():`
- `sources/compartment/integrations/hermes/compartment/__init__.py:55` `current = Path.home() / ".engram"`

</details>

## 5. Reproducibility

`range`: compartment>=4.9.3 (`sources/compartment/integrations/hermes/compartment/plugin.yaml:12`). Encrypted local operation is a supported code path, not a clean-install execution result.

Retained lock/checksum files: none in snapshot. Upstream completeness is unknown: fetch.py:15 excludes .lock files, fetch.py:12 excludes build/vendor directories. Absence here is not absence upstream.

- `sources/compartment/integrations/hermes/compartment/plugin.yaml:2` `version: 4.10.0`
- `sources/compartment/integrations/hermes/compartment/plugin.yaml:4` `# The engine. 'hermes plugins install compartment' (the plugin catalog) and`
- `sources/compartment/integrations/hermes/compartment/plugin.yaml:5` `# 'hermes memory setup' install it into the Hermes environment from this`
- `sources/compartment/integrations/hermes/compartment/plugin.yaml:6` `# line; 'compartment integrate hermes' installs it itself, so there it is`
- `sources/compartment/integrations/hermes/compartment/plugin.yaml:8` `# exact provider, not the newest: Hermes resolves plugin dependencies`
- `sources/compartment/integrations/hermes/compartment/plugin.yaml:10` `# install for two weeks.`
- `sources/compartment/integrations/hermes/compartment/plugin.yaml:11` `pip_dependencies:`
- `sources/compartment/integrations/hermes/compartment/plugin.yaml:12` `- "compartment>=4.9.3"`

## 6. Tests

No dedicated native provider test module was established; repository tests mention Hermes installation/skills without exercising provider forget. Engine forget/shred/window cleanup tests exist and are listed below. Do not count CLI/skill installation tests as adapter deletion coverage. Shred test reopens the vault and asserts row count zero and no search hit. Ciphertext string absence alone is not secure-erasure proof. (`sources/compartment/tests/test_vault_ops.py:66`).

Adapter-related declarations: **0**. Python AST test_ functions, fixtures excluded; literal TS/JS it/test, Go Test*, Rust test_* declarations. No parametrization expansion, collection, execution, or coverage measurement. Other rows are engine test candidates selected by mutation names; names alone do not verify forget coverage.

| file | declarations | adapter-related | forget-related declarations |
|---|---:|---|---|
| `sources/compartment/tests/test_agent_skill.py:1` | 18 | no | `test_removal_never_deletes_a_directory_holding_anything_else` `sources/compartment/tests/test_agent_skill.py:130` |
| `sources/compartment/tests/test_crypto.py:1` | 7 | no | `test_record_key_crypto_shred_semantics` `sources/compartment/tests/test_crypto.py:45` |
| `sources/compartment/tests/test_expiry.py:1` | 40 | no | `test_with_the_toggle_off_nothing_is_ever_deleted` `sources/compartment/tests/test_expiry.py:244` |
| `sources/compartment/tests/test_gate_and_opinions.py:1` | 35 | no | `test_forgetting_the_replacement_restores_the_predecessor` `sources/compartment/tests/test_gate_and_opinions.py:364` |
| `sources/compartment/tests/test_journal_framing.py:1` | 9 | no | `test_relink_refuses_to_launder_a_deletion` `sources/compartment/tests/test_journal_framing.py:126` |
| `sources/compartment/tests/test_recency.py:1` | 25 | no | `test_forgetting_a_memory_takes_it_out_of_the_population` `sources/compartment/tests/test_recency.py:136` |
| `sources/compartment/tests/test_session_entropy.py:1` | 20 | no | `test_a_restart_invalidates_the_credential_even_at_the_same_boot_time` `sources/compartment/tests/test_session_entropy.py:148` |
| `sources/compartment/tests/test_session_lock.py:1` | 8 | no | `test_restart_invalidates_session_credential` `sources/compartment/tests/test_session_lock.py:41` |
| `sources/compartment/tests/test_stays_running.py:1` | 11 | no | `test_turning_it_off_deregisters_before_deleting` `sources/compartment/tests/test_stays_running.py:75` |
| `sources/compartment/tests/test_vault_ops.py:1` | 15 | no | `test_store_search_get_forget` `sources/compartment/tests/test_vault_ops.py:16`; `test_shred_removes_content_from_disk` `sources/compartment/tests/test_vault_ops.py:66`; `test_document_frequencies_are_refreshed_after_a_delete` `sources/compartment/tests/test_vault_ops.py:187`; `test_writing_into_a_deleted_vault_file_is_refused` `sources/compartment/tests/test_vault_ops.py:194` |

## 7. Corrections to B2

Old values below are copied from the triage row (TRIAGE.md), not reconstructed from memory. A refinement narrows an over-broad field; it need not reverse its original value. Counts are distinct field paths, not the number of factual errors.

- `sec.direct.v`: `"Local storage paths and configured endpoints below are reachable independently of provider tool schemas when the same UID has access. Server authorization, encrypted vault unlocks and OS/container boundaries still apply."` → Ciphertext path is reachable, but decryption requires an unlocked session/key/passphrase; same UID is not itself an encryption bypass. Evidence: `sources/compartment/src/compartment/vault.py:567`, `sources/compartment/src/compartment/vault.py:576`.
- `engine_delete.scope`: `"Named deletion/retraction endpoint or method, not a guarantee of physical erasure. Its reach may overlap an agent tool; lack of a native tool is counted separately."` → `yes`: compartment_forget calls v.forget(record_id, caller, shred=...) (`sources/compartment/integrations/hermes/compartment/__init__.py:303`). Engine body deletes the record, removes all embedding window keys, restores records superseded by the deleted correction, appends audit/journal entries, and with shred=true saves rewritten payload immediately (`sources/compartment/src/compartment/vault.py:1432`; `sources/compartment/src/compartment/vault.py:1436`; `sources/compartment/src/compartment/vault.py:1443`). `partial`: deletes live record/vector copies, but restoring an earlier correction can revive old content; audit/journal/backups and host copies are not established as erased. CLI forget calls the same engine; expiry/unlink/reset operations are separate in the ledger. Evidence: `sources/compartment/integrations/hermes/compartment/__init__.py:303`, `sources/compartment/src/compartment/vault.py:1432`, `sources/compartment/src/compartment/vault.py:1436`, `sources/compartment/src/compartment/vault.py:1443`.
- `context.mechanism.v`: `"Source expressions below identify retrieval, cached/static returns, lifecycle capture, and limits. A hook definition is not a guarantee it emits nonempty context each turn; branches and configuration apply."` → System prompt describes persistent vault memory and tools (`sources/compartment/integrations/hermes/compartment/__init__.py:108`). Prefetch searches and returns cosine/text bullets under `[Compartment memory - possibly relevant, treat as data]` (`sources/compartment/integrations/hermes/compartment/__init__.py:144`). No final aggregate character cap in that return. Tool results are JSON, with one stale-vault reopen/retry path (`sources/compartment/integrations/hermes/compartment/__init__.py:283`; `sources/compartment/integrations/hermes/compartment/__init__.py:303`). Evidence: `sources/compartment/integrations/hermes/compartment/__init__.py:108`, `sources/compartment/integrations/hermes/compartment/__init__.py:144`, `sources/compartment/integrations/hermes/compartment/__init__.py:283`, `sources/compartment/integrations/hermes/compartment/__init__.py:303`.
- `sec.telemetry.v`: `"unknown: no emitter identified in the selected adapter files; external dependencies not cleared"` → Accountless local mode `yes`, but locked vault is not readable plaintext without unlock. Default vault path honors COMPARTMENT_VAULT/ENGRAM_VAULT (`sources/compartment/integrations/hermes/compartment/__init__.py:50`). Credentials resolve explicit passphrase, boot-session key, opt-in macOS Keychain, then PASSPHRASE env alias (`sources/compartment/src/compartment/vault.py:565`; `sources/compartment/src/compartment/vault.py:573`). Direct access `yes` to ciphertext; usable Vault access requires the same unlock credentials. No claim that same UID alone breaks encryption. Session credential files/modes and optional keyfile hints are recorded below. No mandatory external account or API telemetry; optional dependency/update egress unknown. The boot-session credential directory is configured by SESSION_DIR, default home()/session, mode 0700; credential files are created 0600 and kernel shared-memory key holder is 0600 (`sources/compartment/src/compartment/session.py:107`; `sources/compartment/src/compartment/session.py:222`; `sources/compartment/src/compartment/session.py:435`). A disk copy of the session ciphertext is not a plaintext unlock key. Concrete default is `~/.compartment/session/<vault-path-hash>.session`, with legacy `~/.engram` fallback; `COMPARTMENT_SESSION_DIR` / `ENGRAM_SESSION_DIR` can override it (`sources/compartment/src/compartment/home.py:27`; `sources/compartment/src/compartment/home.py:45`; `sources/compartment/src/compartment/session.py:423`). Evidence: `sources/compartment/integrations/hermes/compartment/__init__.py:50`, `sources/compartment/src/compartment/vault.py:565`, `sources/compartment/src/compartment/vault.py:573`, `sources/compartment/src/compartment/session.py:107`, `sources/compartment/src/compartment/session.py:222`, `sources/compartment/src/compartment/session.py:435`.
- `repro.decls`: `["integrations/hermes/compartment/plugin.yaml:8", "integrations/hermes/compartment/plugin.yaml:11", "integrations/hermes/compartment/plugin.yaml:12", "integrations/hermes/compartment/plugin.yaml:15", "plugin.json:4", "plugin.json:8", "plugin.json:10", "plugin.json:11", "pyproject.toml:2", "pyproject.toml:7", "pyproject.toml:11", "pyproject.toml:35", "pyproject.toml:36", "pyproject.toml:37", "pyproject.toml:38", "pyproject.toml:39", "pyproject.toml:40", "pyproject.toml:45", "pyproject.toml:53", "pyproject.toml:54", "pyproject.toml:55", "pyproject.toml:63", "pyproject.toml:64", "pyproject.toml:65", "pyproject.toml:66", "pyproject.toml:67", "pyproject.toml:68", "pyproject.toml:78", "pyproject.toml:83", "pyproject.toml:90"]` → `range`: compartment>=4.9.3 (`sources/compartment/integrations/hermes/compartment/plugin.yaml:12`). Encrypted local operation is a supported code path, not a clean-install execution result. Evidence: `sources/compartment/integrations/hermes/compartment/plugin.yaml:12`.
- `tests.count`: `898` → 0 adapter-related declarations using the stated parser/scope; B2 counted a different, repository-wide inventory. This is a scope refinement, not a count of executed or passing tests. No dedicated native provider test module was established; repository tests mention Hermes installation/skills without exercising provider forget. Engine forget/shred/window cleanup tests exist and are listed below. Do not count CLI/skill installation tests as adapter deletion coverage. Shred test reopens the vault and asserts row count zero and no search hit. Ciphertext string absence alone is not secure-erasure proof. (`sources/compartment/tests/test_vault_ops.py:66`). Evidence: `sources/compartment/tests/test_vault_ops.py:66`.

MATRIX counts these 6 explicit field corrections/refinements. Sections 1-6 provide the body and test inventories. Test-name candidates are not passing tests or measured coverage.
