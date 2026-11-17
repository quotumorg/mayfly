# mindgraph: B3 static correctness review

Snapshot `2abbfd03b73c6e369ab645bcb747cc6804555507` (MANIFEST.json row `mindgraph`). No plugin execution or passing-test claims.

## 1. Architecture

`provider`: Python adapter imports mindgraph-sdk, makes HTTP requests, and maintains local durable capture/outbox queues. MCP-compatible tool schemas do not mean an MCP subprocess is used (`sources/mindgraph/provider.py:250`; `sources/mindgraph/provider.py:271`; `sources/mindgraph/provider.py:840`). Isolated plugin-host mode disables automatic turn/delegation capture because transport lacks turn identity (`sources/mindgraph/provider.py:294`).

Hook/body inventory (no-op/fallback declarations are not active behavior):

| callable | declaration | first executable body line |
|---|---|---|
| `register` | `sources/mindgraph/__init__.py:18` | `sources/mindgraph/__init__.py:20` |
| `is_available` | `sources/mindgraph/provider.py:207` | `sources/mindgraph/provider.py:210` |
| `get_config_schema` | `sources/mindgraph/provider.py:215` | `sources/mindgraph/provider.py:217` |
| `initialize` | `sources/mindgraph/provider.py:242` | `sources/mindgraph/provider.py:245` |
| `system_prompt_block` | `sources/mindgraph/provider.py:304` | `sources/mindgraph/provider.py:305` |
| `on_turn_start` | `sources/mindgraph/provider.py:309` | `sources/mindgraph/provider.py:310` |
| `prefetch` | `sources/mindgraph/provider.py:326` | `sources/mindgraph/provider.py:333` |
| `queue_prefetch` | `sources/mindgraph/provider.py:355` | `sources/mindgraph/provider.py:358` |
| `sync_turn` | `sources/mindgraph/provider.py:478` | `sources/mindgraph/provider.py:482` |
| `on_delegation` | `sources/mindgraph/provider.py:504` | `sources/mindgraph/provider.py:505` |
| `on_pre_compress` | `sources/mindgraph/provider.py:674` | `sources/mindgraph/provider.py:683` |
| `on_memory_write` | `sources/mindgraph/provider.py:706` | `sources/mindgraph/provider.py:710` |
| `on_session_switch` | `sources/mindgraph/provider.py:735` | `sources/mindgraph/provider.py:744` |
| `on_session_end` | `sources/mindgraph/provider.py:793` | `sources/mindgraph/provider.py:794` |
| `shutdown` | `sources/mindgraph/provider.py:825` | `sources/mindgraph/provider.py:826` |
| `get_tool_schemas` | `sources/mindgraph/provider.py:835` | `sources/mindgraph/provider.py:840` |
| `handle_tool_call` | `sources/mindgraph/provider.py:842` | `sources/mindgraph/provider.py:843` |

## 2. Forget paths and derived copies

`yes`, correcting B2: `mindgraph_remember` action forget accepts uid/custom_id. Without confirm=true it calls dry_run; confirmed path invokes client.forget (`sources/mindgraph/handlers_write.py:79`; `sources/mindgraph/handlers_write.py:89`). SDK 0.17.0 implements POST `/memory/forget`; its documented reversible node/edge tombstone semantics are `claimed_only` for the absent server body (`sources/_ext/mindgraph-sdk/mindgraph_sdk-0.17.0/mindgraph/client.py:757`). The supplied SDK also implements delete_series, delete_node, delete_edge, delete_embedding, delete_document and ontology type deletion; exact method/path bodies are inventoried below. Derived reach `unknown`: cascade is sent, server execution not visible. Restore explicitly restores the node and individually requested edges (`sources/mindgraph/handlers_write.py:105`).

Host state.db is independent (`hermes-agent/hermes_state.py:181`). No reviewed adapter forget body performs a general purge of that database or built-in MEMORY.md/USER.md. Prompt/tool-result copies and independent source transcripts remain separate; source operation bodies below establish the scope of each plugin mutation.

<details>
<summary>Deletion, invalidation, archive and reset body ledger</summary>

Public routes, SDK/CLI dispatch and internal storage operations are distinguished by their source. Cache invalidation and management-object deletion are not additional fact-forget tools. A stub is not working deletion. Call-through means the downstream body still determines erasure. HTTP prefixes/selectors must be read with route registration.

**forget_injected**, body `sources/mindgraph/recall.py:579`; implementation/call-through.

- `sources/mindgraph/recall.py:577` `def forget_injected(self, session: str) -> None:`
- `sources/mindgraph/recall.py:579` `with self._lock:`

**delete_series**, body `sources/_ext/mindgraph-sdk/mindgraph_sdk-0.17.0/mindgraph/client.py:323`; implementation/call-through.

- `sources/_ext/mindgraph-sdk/mindgraph_sdk-0.17.0/mindgraph/client.py:316` `def delete_series(`
- `sources/_ext/mindgraph-sdk/mindgraph_sdk-0.17.0/mindgraph/client.py:323` `body: dict[str, Any] = {`

**forget**, body `sources/_ext/mindgraph-sdk/mindgraph_sdk-0.17.0/mindgraph/client.py:742`; implementation/call-through.

- `sources/_ext/mindgraph-sdk/mindgraph_sdk-0.17.0/mindgraph/client.py:725` `def forget(`
- `sources/_ext/mindgraph-sdk/mindgraph_sdk-0.17.0/mindgraph/client.py:742` `if not uid and not custom_id:`

**delete_node**, body `sources/_ext/mindgraph-sdk/mindgraph_sdk-0.17.0/mindgraph/client.py:890`; implementation/call-through.

- `sources/_ext/mindgraph-sdk/mindgraph_sdk-0.17.0/mindgraph/client.py:889` `def delete_node(self, uid: str) -> None:`
- `sources/_ext/mindgraph-sdk/mindgraph_sdk-0.17.0/mindgraph/client.py:890` `self._request("DELETE", f"/node/{uid}")`

**batch_delete_nodes**, body `sources/_ext/mindgraph-sdk/mindgraph_sdk-0.17.0/mindgraph/client.py:905`; implementation/call-through.

- `sources/_ext/mindgraph-sdk/mindgraph_sdk-0.17.0/mindgraph/client.py:892` `def batch_delete_nodes(`
- `sources/_ext/mindgraph-sdk/mindgraph_sdk-0.17.0/mindgraph/client.py:905` `body: dict[str, Any] = {"hard_purge": hard_purge}`

**delete_edge**, body `sources/_ext/mindgraph-sdk/mindgraph_sdk-0.17.0/mindgraph/client.py:976`; implementation/call-through.

- `sources/_ext/mindgraph-sdk/mindgraph_sdk-0.17.0/mindgraph/client.py:975` `def delete_edge(self, uid: str) -> None:`
- `sources/_ext/mindgraph-sdk/mindgraph_sdk-0.17.0/mindgraph/client.py:976` `self._request("DELETE", f"/edge/{uid}")`

**delete_embedding**, body `sources/_ext/mindgraph-sdk/mindgraph_sdk-0.17.0/mindgraph/client.py:1136`; implementation/call-through.

- `sources/_ext/mindgraph-sdk/mindgraph_sdk-0.17.0/mindgraph/client.py:1135` `def delete_embedding(self, uid: str) -> None:`
- `sources/_ext/mindgraph-sdk/mindgraph_sdk-0.17.0/mindgraph/client.py:1136` `self._request("DELETE", f"/node/{uid}/embedding")`

**purge**, body `sources/_ext/mindgraph-sdk/mindgraph_sdk-0.17.0/mindgraph/client.py:1204`; implementation/call-through.

- `sources/_ext/mindgraph-sdk/mindgraph_sdk-0.17.0/mindgraph/client.py:1203` `def purge(self, before: float | None = None) -> Any:`
- `sources/_ext/mindgraph-sdk/mindgraph_sdk-0.17.0/mindgraph/client.py:1204` `body: dict[str, Any] = {}`

**tombstone**, body `sources/_ext/mindgraph-sdk/mindgraph_sdk-0.17.0/mindgraph/client.py:1271`; implementation/call-through.

- `sources/_ext/mindgraph-sdk/mindgraph_sdk-0.17.0/mindgraph/client.py:1268` `def tombstone(`
- `sources/_ext/mindgraph-sdk/mindgraph_sdk-0.17.0/mindgraph/client.py:1271` `body: dict[str, Any] = {"action": "tombstone", "uid": uid}`

**delete_document**, body `sources/_ext/mindgraph-sdk/mindgraph_sdk-0.17.0/mindgraph/client.py:1638`; implementation/call-through.

- `sources/_ext/mindgraph-sdk/mindgraph_sdk-0.17.0/mindgraph/client.py:1636` `def delete_document(self, uid: str) -> Any:`
- `sources/_ext/mindgraph-sdk/mindgraph_sdk-0.17.0/mindgraph/client.py:1638` `return self._request("DELETE", f"/ingest/document/{uid}")`

**archive_ontology_series_binding**, body `sources/_ext/mindgraph-sdk/mindgraph_sdk-0.17.0/mindgraph/client.py:1790`; implementation/call-through.

- `sources/_ext/mindgraph-sdk/mindgraph_sdk-0.17.0/mindgraph/client.py:1786` `def archive_ontology_series_binding(`
- `sources/_ext/mindgraph-sdk/mindgraph_sdk-0.17.0/mindgraph/client.py:1793` `"DELETE",`

**archive_ontology_schema**, body `sources/_ext/mindgraph-sdk/mindgraph_sdk-0.17.0/mindgraph/client.py:1820`; implementation/call-through.

- `sources/_ext/mindgraph-sdk/mindgraph_sdk-0.17.0/mindgraph/client.py:1818` `def archive_ontology_schema(self, schema_id: str) -> dict[str, Any]:`
- `sources/_ext/mindgraph-sdk/mindgraph_sdk-0.17.0/mindgraph/client.py:1820` `return self._request("DELETE", f"/v1/ontology/schemas/{schema_id}")`

**delete_ontology_object_type**, body `sources/_ext/mindgraph-sdk/mindgraph_sdk-0.17.0/mindgraph/client.py:1883`; implementation/call-through.

- `sources/_ext/mindgraph-sdk/mindgraph_sdk-0.17.0/mindgraph/client.py:1880` `def delete_ontology_object_type(`
- `sources/_ext/mindgraph-sdk/mindgraph_sdk-0.17.0/mindgraph/client.py:1884` `"DELETE",`

**delete_ontology_relation_type**, body `sources/_ext/mindgraph-sdk/mindgraph_sdk-0.17.0/mindgraph/client.py:1907`; implementation/call-through.

- `sources/_ext/mindgraph-sdk/mindgraph_sdk-0.17.0/mindgraph/client.py:1904` `def delete_ontology_relation_type(`
- `sources/_ext/mindgraph-sdk/mindgraph_sdk-0.17.0/mindgraph/client.py:1908` `"DELETE",`

</details>

## 3. What reaches the agent

System block comes from contract instructions plus Hermes notes (`sources/mindgraph/provider.py:307`). Prefetch runs session brief beside recall, joins both and prepends `## MindGraph context (recalled for this message)` (`sources/mindgraph/provider.py:343`; `sources/mindgraph/provider.py:353`). Recall's individual cap does not include all brief/header text. Tool results come through handle_tool/handlers/results formatting (`sources/mindgraph/provider.py:845`). Exact deadlines, sizes and formatting below.

Host assembly: `hermes-agent/agent/memory_manager.py:352` joins provider system blocks. `hermes-agent/agent/turn_context.py:1081` requests prefetch for nontrivial turns; `hermes-agent/agent/turn_context.py:120` fences it; `hermes-agent/agent/turn_context.py:136` appends it to API-bound user content. Tool results are a third channel. Optional host spill is at `hermes-agent/agent/memory_manager.py:419`, not a universal provider cap. Built-in memory remains separately controlled (`hermes-agent/agent/system_prompt.py:542`).

<details>
<summary>Context and tool-result assembly quotes</summary>

`static_tool_schemas`, body `sources/mindgraph/dispatch.py:49`:

- `sources/mindgraph/dispatch.py:49` `return [dict(tool) for tool in contract()["tools"]]`

`system_prompt_block`, body `sources/mindgraph/provider.py:305`:

- `sources/mindgraph/provider.py:306` `return ""`
- `sources/mindgraph/provider.py:307` `return f"# MindGraph memory\n\n{contract()['instructions']}\n{_HERMES_NOTES}"`

`prefetch`, body `sources/mindgraph/provider.py:333`:

- `sources/mindgraph/provider.py:339` `return ""`
- `sources/mindgraph/provider.py:342` `return ""  # recall off is no automatic injection at all, the brief included`
- `sources/mindgraph/provider.py:349` `return ""`
- `sources/mindgraph/provider.py:353` `return "## MindGraph context (recalled for this message)\n\n" + "\n\n".join(sections)`

`queue_prefetch`, body `sources/mindgraph/provider.py:358`:



`_capture_context`, body `sources/mindgraph/provider.py:470`:

- `sources/mindgraph/provider.py:475` `return 0, {}`
- `sources/mindgraph/provider.py:476` `return bound[1], bound[2]`

`get_tool_schemas`, body `sources/mindgraph/provider.py:840`:

- `sources/mindgraph/provider.py:840` `return static_tool_schemas()`

`handle_tool_call`, body `sources/mindgraph/provider.py:843`:

- `sources/mindgraph/provider.py:844` `return '{"error":"MindGraph provider is not initialized"}'`
- `sources/mindgraph/provider.py:845` `return handle_tool(self._rt, self._gate, self._memory_config, tool_name, args)`

`render_context`, body `sources/mindgraph/render.py:405`:

- `sources/mindgraph/render.py:407` `return None`
- `sources/mindgraph/render.py:410` `# dated items, supersession and conflict markers, a hard token budget.`
- `sources/mindgraph/render.py:415` `budget = render_budget(max_chars)`
- `sources/mindgraph/render.py:416` `if budget is not None and len(server_text) > budget:`
- `sources/mindgraph/render.py:417` `return f"{server_text[:budget]}\n[truncated to {budget} characters]"`
- `sources/mindgraph/render.py:418` `return server_text`
- `sources/mindgraph/render.py:428` `sections.append(`
- `sources/mindgraph/render.py:444` `sections.append(_Section("## Relationships", relationships))`
- `sources/mindgraph/render.py:446` `sections.append(`
- `sources/mindgraph/render.py:453` `sections.append(`
- `sources/mindgraph/render.py:467` `return "No results found for this search."`
- `sources/mindgraph/render.py:471` `return _assemble("# Retrieved context\n\nNo graph results matched this search.", sections, state, max_chars)`
- `sources/mindgraph/render.py:472` `return _assemble("# Retrieved context", sections, state, max_chars)`

`render_resume_context`, body `sources/mindgraph/render.py:521`:

- `sources/mindgraph/render.py:523` `return None`
- `sources/mindgraph/render.py:528` `return _assemble(`
- `sources/mindgraph/render.py:532` `max_chars,`
- `sources/mindgraph/render.py:559` `sections.append(_Section("## Warning", [f"- {warning}"]))`
- `sources/mindgraph/render.py:562` `sections.append(_Section("## Selection", [f"- {selection_reason}"]))`
- `sources/mindgraph/render.py:564` `return None`
- `sources/mindgraph/render.py:565` `return _assemble("# Resume context", sections, state, max_chars)`

</details>

## 4. Data access boundaries

MINDGRAPH_API_KEY/base URL/agent/org IDs use scoped secret access (`sources/mindgraph/provider.py:246`). Local capture queue is opened in the Hermes profile; it can contain transcript data even though the engine is remote (`sources/mindgraph/provider.py:271`). Direct access `yes`, via SDK/HTTP with same key or local outbox files. Accountless local mode `no` in supplied integration. Telemetry default for remote service `unknown`; governance requests are explicit functional traffic, not evidence of telemetry being off.

Boundary model: same OS user, terminal, mounted filesystem and allowed network. Direct reachability means another client can use the same file/endpoint and available credentials, not unauthenticated access or encryption bypass. Hermes setup persists declared secrets through `hermes-agent/hermes_cli/memory_setup.py:321` and `hermes-agent/hermes_cli/config.py:2697`. These are possible configured paths; no live credentials were read.

Host credential-file detail: `get_hermes_home()/.env` is resolved at `hermes-agent/hermes_cli/config.py:473`. New files are secured to 0600 at `hermes-agent/hermes_cli/config.py:506`; the writer preserves an existing mode at `hermes-agent/hermes_cli/config.py:2544` and `:2562`. Therefore 0600 is not an unconditional mode claim for every existing profile.

<details>
<summary>Paths, environment, permissions, egress, telemetry and process-launch evidence</summary>

- `sources/mindgraph/capture.py:225` `os.chmod(path, 0o600)  # transcript text; the profile owner only`

- `sources/mindgraph/evals/_plugin.py:41` `secrets.get_secret = lambda name, default=None: os.environ.get(name, default)`

- `sources/mindgraph/evals/run.py:27` `DEFAULT_BASE_URL = "https://api.mindgraph.cloud"`
- `sources/mindgraph/evals/run.py:183` `os.environ["MINDGRAPH_API_KEY"] = api_key`
- `sources/mindgraph/evals/run.py:184` `os.environ["MINDGRAPH_BASE_URL"] = base_url`
- `sources/mindgraph/evals/run.py:274` `api_key = os.environ.get("MINDGRAPH_EVAL_API_KEY", "")`
- `sources/mindgraph/evals/run.py:278` `base_url = os.environ.get("MINDGRAPH_EVAL_BASE_URL", DEFAULT_BASE_URL)`

- `sources/mindgraph/plugin.yaml:11` `url: "https://mindgraph.cloud"`

- `sources/mindgraph/provider.py:210` `return bool(get_secret("MINDGRAPH_API_KEY", ""))`
- `sources/mindgraph/provider.py:224` `"url": "https://mindgraph.cloud",`
- `sources/mindgraph/provider.py:245` `self._isolated_host = os.environ.get("HERMES_PLUGIN_HOST_PROCESS") == "1"`
- `sources/mindgraph/provider.py:246` `base_url = (get_secret("MINDGRAPH_BASE_URL", "") or "").strip() or DEFAULT_BASE_URL`
- `sources/mindgraph/provider.py:247` `agent_id = (get_secret("MINDGRAPH_AGENT_ID", "") or "").strip() or DEFAULT_AGENT_ID`
- `sources/mindgraph/provider.py:248` `api_key = get_secret("MINDGRAPH_API_KEY", "") or ""`
- `sources/mindgraph/provider.py:249` `org_id = (get_secret("MINDGRAPH_ORG_ID", "") or "").strip() or None`
- `sources/mindgraph/provider.py:264` `disabled=(get_secret("MINDGRAPH_GOVERNANCE", "") or "").strip().lower() == "off",`
- `sources/mindgraph/provider.py:266` `self._auto_capture = (get_secret("MINDGRAPH_AUTO_CAPTURE", "") or "").strip().lower() not in (`

- `sources/mindgraph/runtime.py:19` `DEFAULT_BASE_URL = "https://api.mindgraph.cloud"`

- `sources/mindgraph/scripts/check_version.py:77` `ref = os.environ.get("GITHUB_REF", "")`

</details>

## 5. Reproducibility

`range`: mindgraph-sdk>=0.17.0,<0.18 (`sources/mindgraph/pyproject.toml:10`). Extra supplied SDK source is 0.17.0; its sdist SHA256 was supplied in TASK-B3.md, not independently rehashed from an archive here. Range upgrades and remote server revisions remain unverified.

Retained lock/checksum files: none in snapshot. Upstream completeness is unknown: fetch.py:15 excludes .lock files, fetch.py:12 excludes build/vendor directories. Absence here is not absence upstream.

- `sources/mindgraph/plugin.yaml:1` `manifest_version: 1`
- `sources/mindgraph/plugin.yaml:3` `version: 1.2.0`
- `sources/mindgraph/plugin.yaml:6` `requires_hermes: ">=0.22.0"`

- `sources/mindgraph/pyproject.toml:3` `version = "1.2.0"`
- `sources/mindgraph/pyproject.toml:5` `requires-python = ">=3.11"`
- `sources/mindgraph/pyproject.toml:7` `# Installed into the Hermes environment by 'hermes plugins install' / 'enable'.`
- `sources/mindgraph/pyproject.toml:9` `dependencies = [`
- `sources/mindgraph/pyproject.toml:10` `"mindgraph-sdk>=0.17.0,<0.18",`
- `sources/mindgraph/pyproject.toml:11` `"httpx>=0.27,<1",`
- `sources/mindgraph/pyproject.toml:16` `target-version = "py311"`

## 6. Tests

Forget tests exist in tests/test_handlers_write.py, including dry-run/target validation and confirmed apply; SDK offline tests are separately inventoried. These test requests and confirmation, not backend tombstone persistence (`sources/mindgraph/tests/test_handlers_write.py:69`). Confirmed forget exercises SDK request dispatch; remote persistence is unverified. (`sources/mindgraph/tests/test_mcp_027.py:20`).

Adapter-related declarations: **398**. Python AST test_ functions, fixtures excluded; literal TS/JS it/test, Go Test*, Rust test_* declarations. No parametrization expansion, collection, execution, or coverage measurement. Other rows are engine test candidates selected by mutation names; names alone do not verify forget coverage.

| file | declarations | adapter-related | forget-related declarations |
|---|---:|---|---|
| `sources/mindgraph/tests/conftest.py:1` | 0 | yes | none |
| `sources/mindgraph/tests/fixtures/mcp_requests.json:1` | 0 | yes | none |
| `sources/mindgraph/tests/host/test_real_host.py:1` | 33 | yes | none |
| `sources/mindgraph/tests/support/fake_mindgraph.py:1` | 0 | yes | none |
| `sources/mindgraph/tests/test_capture.py:1` | 86 | yes | `test_delivery_keeps_the_receipt_and_forgets_the_text` `sources/mindgraph/tests/test_capture.py:257`; `test_pruning_forgets_old_receipts_but_never_undelivered_text` `sources/mindgraph/tests/test_capture.py:500` |
| `sources/mindgraph/tests/test_contract_parity.py:1` | 4 | yes | none |
| `sources/mindgraph/tests/test_dispatch.py:1` | 9 | yes | none |
| `sources/mindgraph/tests/test_evals.py:1` | 6 | yes | none |
| `sources/mindgraph/tests/test_governance.py:1` | 14 | yes | none |
| `sources/mindgraph/tests/test_handlers_read.py:1` | 44 | yes | none |
| `sources/mindgraph/tests/test_handlers_write.py:1` | 47 | yes | `test_forget_by_custom_id_with_dry_run` `sources/mindgraph/tests/test_handlers_write.py:69`; `test_forget_uid_takes_precedence_over_custom_id` `sources/mindgraph/tests/test_handlers_write.py:75`; `test_forget_without_target_is_an_error` `sources/mindgraph/tests/test_handlers_write.py:83` |
| `sources/mindgraph/tests/test_mcp_027.py:1` | 12 | yes | `test_forget_requires_boolean_confirmation` `sources/mindgraph/tests/test_mcp_027.py:12`; `test_confirmed_forget_has_undo_hint_but_dry_run_still_wins` `sources/mindgraph/tests/test_mcp_027.py:20` |
| `sources/mindgraph/tests/test_mcp_request_parity.py:1` | 2 | yes | none |
| `sources/mindgraph/tests/test_memory_tool.py:1` | 7 | yes | none |
| `sources/mindgraph/tests/test_mirroring.py:1` | 5 | yes | `test_lost_forget_reply_does_not_let_a_later_add_overtake_removal` `sources/mindgraph/tests/test_mirroring.py:49` |
| `sources/mindgraph/tests/test_provider.py:1` | 24 | yes | `test_a_failed_forget_does_not_block_the_replacement` `sources/mindgraph/tests/test_provider.py:222`; `test_replacing_a_memory_with_identical_content_never_forgets_it` `sources/mindgraph/tests/test_provider.py:230` |
| `sources/mindgraph/tests/test_recall.py:1` | 56 | yes | `test_an_undo_forgets_what_the_undone_turns_were_shown` `sources/mindgraph/tests/test_recall.py:378` |
| `sources/mindgraph/tests/test_render.py:1` | 39 | yes | none |
| `sources/mindgraph/tests/test_render_context_text.py:1` | 3 | yes | none |
| `sources/mindgraph/tests/test_wire.py:1` | 7 | yes | none |
| `sources/_ext/mindgraph-sdk/mindgraph_sdk-0.17.0/tests/test_integration.py:1` | 83 | no | `test_forget_dry_run_then_forget` `sources/_ext/mindgraph-sdk/mindgraph_sdk-0.17.0/tests/test_integration.py:449` |

## 7. Corrections to B2

Old values below are copied from the triage row (TRIAGE.md), not reconstructed from memory. A refinement narrows an over-broad field; it need not reverse its original value. Counts are distinct field paths, not the number of factual errors.

- `forget_tool.v`: `"update_only"` → yes: confirmed mindgraph_remember action=forget dispatches to SDK forget; unconfirmed calls preview. Evidence: `sources/mindgraph/handlers_write.py:79`, `sources/mindgraph/handlers_write.py:89`.
- `engine_delete.v`: `"not_in_repo"` → yes at supplied SDK layer: POST /memory/forget plus node/edge/embedding/document deletes; server implementation remains not_in_repo. Evidence: `sources/mindgraph/handlers_write.py:89`.
- `context.mechanism.v`: `"Source expressions below identify retrieval, cached/static returns, lifecycle capture, and limits. A hook definition is not a guarantee it emits nonempty context each turn; branches and configuration apply."` → Automatic context combines a session brief and bounded recall; isolated host disables automatic turn/delegation capture. Evidence: `sources/mindgraph/provider.py:294`, `sources/mindgraph/provider.py:353`.
- `engine_delete.scope`: `"Not established beyond the selected snapshot; none means no content-deletion operation identified in inspected implementation, not proof that arbitrary terminal deletion is impossible."` → `yes`, correcting B2: `mindgraph_remember` action forget accepts uid/custom_id. Without confirm=true it calls dry_run; confirmed path invokes client.forget (`sources/mindgraph/handlers_write.py:79`; `sources/mindgraph/handlers_write.py:89`). SDK 0.17.0 implements POST `/memory/forget`; its documented reversible node/edge tombstone semantics are `claimed_only` for the absent server body (`sources/_ext/mindgraph-sdk/mindgraph_sdk-0.17.0/mindgraph/client.py:757`). The supplied SDK also implements delete_series, delete_node, delete_edge, delete_embedding, delete_document and ontology type deletion; exact method/path bodies are inventoried below. Derived reach `unknown`: cascade is sent, server execution not visible. Restore explicitly restores the node and individually requested edges (`sources/mindgraph/handlers_write.py:105`). Evidence: `sources/mindgraph/handlers_write.py:79`, `sources/mindgraph/handlers_write.py:89`, `sources/mindgraph/handlers_write.py:105`.
- `sec.telemetry.v`: `"inspect listed emitters/configuration"` → MINDGRAPH_API_KEY/base URL/agent/org IDs use scoped secret access (`sources/mindgraph/provider.py:246`). Local capture queue is opened in the Hermes profile; it can contain transcript data even though the engine is remote (`sources/mindgraph/provider.py:271`). Direct access `yes`, via SDK/HTTP with same key or local outbox files. Accountless local mode `no` in supplied integration. Telemetry default for remote service `unknown`; governance requests are explicit functional traffic, not evidence of telemetry being off. Evidence: `sources/mindgraph/provider.py:246`, `sources/mindgraph/provider.py:271`.
- `repro.decls`: `["plugin.yaml:9", "plugin.yaml:15", "plugin.yaml:16", "plugin.yaml:17", "plugin.yaml:18", "plugin.yaml:19", "plugin.yaml:20", "pyproject.toml:3", "pyproject.toml:5", "pyproject.toml:9", "pyproject.toml:10", "pyproject.toml:11", "pyproject.toml:16"]` → `range`: mindgraph-sdk>=0.17.0,<0.18 (`sources/mindgraph/pyproject.toml:10`). Extra supplied SDK source is 0.17.0; its sdist SHA256 was supplied in TASK-B3.md, not independently rehashed from an archive here. Range upgrades and remote server revisions remain unverified. Evidence: `sources/mindgraph/pyproject.toml:10`.
- `tests.count`: `398` → 398 adapter-related declarations using the stated parser/scope; B2 counted a different, repository-wide inventory. This is a scope refinement, not a count of executed or passing tests. Forget tests exist in tests/test_handlers_write.py, including dry-run/target validation and confirmed apply; SDK offline tests are separately inventoried. These test requests and confirmation, not backend tombstone persistence (`sources/mindgraph/tests/test_handlers_write.py:69`). Confirmed forget exercises SDK request dispatch; remote persistence is unverified. (`sources/mindgraph/tests/test_mcp_027.py:20`). Evidence: `sources/mindgraph/tests/test_handlers_write.py:69`, `sources/mindgraph/tests/test_mcp_027.py:20`.

MATRIX counts these 7 explicit field corrections/refinements. Sections 1-6 provide the body and test inventories. Test-name candidates are not passing tests or measured coverage.
