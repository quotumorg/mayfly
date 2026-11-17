# cognee: B3 static correctness review

Snapshot `b132740960181e712f15c76ee80519cbd15fdb5d` (MANIFEST.json row `cognee`). No plugin execution or passing-test claims.

## 1. Architecture

`provider`: native adapter supports HTTP remote/local server and SDK embedded mode. Local runtime is installed into a private environment; session finalization may be handed to a detached exit watcher (`sources/cognee/integrations/hermes-agent/cognee_integration_hermes/runtime_env.py:165`; `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/provider.py:1325`). Core Cognee engine source is not retained in this repository snapshot; backend.py is an adapter abstraction, not the engine.

Hook/body inventory (no-op/fallback declarations are not active behavior):

| callable | declaration | first executable body line |
|---|---|---|
| `register` | `sources/cognee/integrations/hermes-agent/__init__.py:15` | `sources/cognee/integrations/hermes-agent/__init__.py:17` |
| `register` | `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/__init__.py:8` | `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/__init__.py:10` |
| `register` | `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/_plugin_root/plugin_init.py:15` | `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/_plugin_root/plugin_init.py:17` |
| `shutdown` | `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/backend.py:290` | `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/backend.py:291` |
| `save_config` | `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/config.py:348` | `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/config.py:350` |
| `is_available` | `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/provider.py:390` | `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/provider.py:393` |
| `get_config_schema` | `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/provider.py:417` | `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/provider.py:418` |
| `save_config` | `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/provider.py:465` | `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/provider.py:466` |
| `initialize` | `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/provider.py:557` | `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/provider.py:558` |
| `system_prompt_block` | `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/provider.py:782` | `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/provider.py:784` |
| `prefetch` | `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/provider.py:804` | `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/provider.py:805` |
| `queue_prefetch` | `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/provider.py:821` | `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/provider.py:822` |
| `sync_turn` | `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/provider.py:841` | `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/provider.py:866` |
| `get_tool_schemas` | `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/provider.py:1148` | `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/provider.py:1149` |
| `handle_tool_call` | `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/provider.py:1160` | `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/provider.py:1161` |
| `on_session_end` | `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/provider.py:1219` | `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/provider.py:1220` |
| `on_pre_compress` | `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/provider.py:1246` | `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/provider.py:1255` |
| `on_session_switch` | `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/provider.py:1370` | `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/provider.py:1378` |
| `on_memory_write` | `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/provider.py:1412` | `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/provider.py:1423` |
| `on_delegation` | `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/provider.py:1455` | `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/provider.py:1463` |
| `shutdown` | `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/provider.py:1468` | `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/provider.py:1469` |

## 2. Forget paths and derived copies

`yes` for HTTP mode: cognee_forget action=find lists candidate documents, action=forget deletes confirmed data_ids (`sources/cognee/integrations/hermes-agent/cognee_integration_hermes/provider.py:1966`; `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/provider.py:1969`). HTTP forget_document POSTs `/api/v1/forget` with datasetId/dataId (`sources/cognee/integrations/hermes-agent/cognee_integration_hermes/http_backend.py:901`). Embedded per-document method is `stub`: raises HTTP-only error (`sources/cognee/integrations/hermes-agent/cognee_integration_hermes/backend.py:224`). Wider SDK `forget(dataset/everything/memory_only)` is implemented as call-through to cognee.forget (`sources/cognee/integrations/hermes-agent/cognee_integration_hermes/backend.py:582`). Engine body `not_in_repo`; matrix not_in_repo refers to absent core; request/call-through is present. Derived cleanup `unknown`: session-cache cleanup is a version-dependent contract, not verified core code here.

Host state.db is independent (`hermes-agent/hermes_state.py:181`). No reviewed adapter forget body performs a general purge of that database or built-in MEMORY.md/USER.md. Prompt/tool-result copies and independent source transcripts remain separate; source operation bodies below establish the scope of each plugin mutation.

<details>
<summary>Deletion, invalidation, archive and reset body ledger</summary>

Public routes, SDK/CLI dispatch and internal storage operations are distinguished by their source. Cache invalidation and management-object deletion are not additional fact-forget tools. A stub is not working deletion. Call-through means the downstream body still determines erasure. HTTP prefixes/selectors must be read with route registration.

**forget**, body `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/backend.py:179`; stub.

- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/backend.py:171` `def forget(`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/backend.py:179` `raise NotImplementedError`

**forget_document**, body `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/backend.py:224`; stub.

- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/backend.py:223` `def forget_document(self, *, dataset_id: str, data_id: str, timeout: float) -> dict[str, Any]:`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/backend.py:224` `raise self._http_only("per-document deletion")`

**forget**, body `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/backend.py:428`; implementation/call-through.

- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/backend.py:427` `def forget(self, *, dataset, everything, memory_only, timeout) -> dict[str, Any]:`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/backend.py:428` `return self._bridge.run(`

**_do_forget**, body `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/backend.py:573`; implementation/call-through.

- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/backend.py:566` `async def _do_forget(`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/backend.py:582` `return await cognee.forget(**kwargs)`

**forget**, body `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/http_backend.py:802`; implementation/call-through.

- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/http_backend.py:801` `def forget(self, *, dataset, everything, memory_only, timeout) -> dict[str, Any]:`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/http_backend.py:802` `body: dict[str, Any] = {"everything": everything, "memory_only": memory_only}`

**forget_document**, body `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/http_backend.py:897`; implementation/call-through.

- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/http_backend.py:890` `def forget_document(self, *, dataset_id: str, data_id: str, timeout: float) -> dict[str, Any]:`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/http_backend.py:897` `body = {`

**_forget_terms**, body `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/provider.py:1962`; implementation/call-through.

- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/provider.py:1960` `@staticmethod`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/provider.py:1961` `def _forget_terms(raw_terms: str) -> list[str]:`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/provider.py:1962` `return [token for token in raw_terms.lower().replace(",", " ").split() if len(token) >= 2]`

**_handle_forget**, body `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/provider.py:1965`; implementation/call-through.

- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/provider.py:1964` `def _handle_forget(self, args: dict[str, Any]) -> str:`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/provider.py:1965` `action = str(args.get("action") or "").strip().lower()`

**_forget_find**, body `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/provider.py:1980`; implementation/call-through.

- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/provider.py:1979` `def _forget_find(self, args: dict[str, Any]) -> str:`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/provider.py:1980` `terms = self._forget_terms(str(args.get("terms") or ""))`

**_forget_delete**, body `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/provider.py:2055`; implementation/call-through.

- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/provider.py:2054` `def _forget_delete(self, args: dict[str, Any]) -> str:`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/provider.py:2064` `result = self._backend.forget(`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/provider.py:2091` `self._backend.forget_document(`

</details>

## 3. What reaches the agent

System instructions describe recall/remember/forget (`sources/cognee/integrations/hermes-agent/cognee_integration_hermes/provider.py:790`). Prefetch waits up to three seconds for cached work, consumes result and wraps it with `## Cognee Memory` and optional steering (`sources/cognee/integrations/hermes-agent/cognee_integration_hermes/provider.py:808`; `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/provider.py:818`). Formatter/config limits are quoted below; context depends on previous background work, not an unconditional new search at return time.

Host assembly: `hermes-agent/agent/memory_manager.py:352` joins provider system blocks. `hermes-agent/agent/turn_context.py:1081` requests prefetch for nontrivial turns; `hermes-agent/agent/turn_context.py:120` fences it; `hermes-agent/agent/turn_context.py:136` appends it to API-bound user content. Tool results are a third channel. Optional host spill is at `hermes-agent/agent/memory_manager.py:419`, not a universal provider cap. Built-in memory remains separately controlled (`hermes-agent/agent/system_prompt.py:542`).

<details>
<summary>Context and tool-result assembly quotes</summary>

`_tool_patterns`, body `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/capture_policy.py:231`:

- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/capture_policy.py:236` `return [_parse_pattern(pattern) for pattern in entries]`

`_context`, body `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/http_backend.py:291`:

- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/http_backend.py:293` `return self._ssl_context`

`_tool_args`, body `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/provider.py:172`:

- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/provider.py:173` `return raw`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/provider.py:178` `return {"arguments": raw}`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/provider.py:179` `return parsed if isinstance(parsed, dict) else {"arguments": parsed}`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/provider.py:180` `return {}`

`_tool_error`, body `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/provider.py:185`:

- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/provider.py:190` `return ""`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/provider.py:192` `return _text_of(data["error"])`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/provider.py:193` `return ""`

`system_prompt_block`, body `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/provider.py:784`:

- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/provider.py:785` `return ""`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/provider.py:801` `lines.append(steer)`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/provider.py:802` `return "\n".join(lines)`

`prefetch`, body `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/provider.py:805`:

- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/provider.py:806` `return ""`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/provider.py:808` `self._prefetch_thread.join(timeout=3.0)`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/provider.py:813` `return ""`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/provider.py:818` `return f"## Cognee Memory\n{_BLOCK_STEER}\n{result}"`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/provider.py:819` `return f"## Cognee Memory\n{result}"`

`queue_prefetch`, body `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/provider.py:822`:



`get_tool_schemas`, body `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/provider.py:1149`:

- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/provider.py:1153` `schemas.append(SWITCH_DATASET_SCHEMA)`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/provider.py:1155` `schemas.append(CODE_SEARCH_SCHEMA)`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/provider.py:1157` `schemas.append(RECAP_SCHEMA)`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/provider.py:1158` `return schemas`

`handle_tool_call`, body `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/provider.py:1161`:

- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/provider.py:1162` `return self._unavailable_envelope()`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/provider.py:1164` `return json.dumps(`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/provider.py:1173` `return self._handle_recall(args)`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/provider.py:1175` `return self._handle_remember(args)`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/provider.py:1177` `return self._handle_forget(args)`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/provider.py:1179` `return self._handle_switch_dataset(args)`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/provider.py:1181` `return self._handle_code_search(args)`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/provider.py:1183` `return self._handle_recap(args)`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/provider.py:1184` `return json.dumps({"error": f"Unknown Cognee tool: {tool_name}"})`

`_run_layered_prefetch`, body `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/provider.py:1656`:

- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/provider.py:1658` `budget = self._config.get("recall_budget")`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/provider.py:1659` `deadline = time.monotonic() + (20.0 if budget is None else float(budget))`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/provider.py:1662` `passage_chars = self._char_limit("recall_passage_chars", recall_text.DEFAULT_PASSAGE_CHARS)`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/provider.py:1663` `context_chars = self._char_limit("recall_context_chars", recall_text.DEFAULT_CONTEXT_CHARS)`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/provider.py:1667` `# instead of spending the whole budget on a server that is not`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/provider.py:1681` `lanes.append(`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/provider.py:1694` `lanes.append(`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/provider.py:1701` `lanes.append(`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/provider.py:1708` `# Same deadline for every lane: min(per-call timeout, budget left).`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/provider.py:1709` `# Below the floor a call cannot return anything useful, so nothing is`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/provider.py:1732` `return None, exc`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/provider.py:1733` `return results, None`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/provider.py:1740` `# with ''lane_timeout'', so the pool joins within the budget.`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/provider.py:1742` `max_workers=len(lanes), thread_name_prefix="cognee-recall-lane"`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/provider.py:1781` `logger.debug("cognee recall_context_%s %s", event, json.dumps(stats))`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/provider.py:1785` `memories.append(context.strip())`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/provider.py:1787` `guidance = "\n\n".join(self._memory_lane_texts(results)).strip()`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/provider.py:1789` `lines = self._format_recall_lines(results, limit=top_k)`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/provider.py:1791` `code_blocks.append(f"<{label}>\n" + "\n".join(lines) + f"\n</{label}>")`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/provider.py:1829` `blocks.append(f"<{_GUIDANCE_LANE}>\n{kept_guidance.strip()}\n</{_GUIDANCE_LANE}>")`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/provider.py:1851` `self._prefetch_result = instruction + header + "\n\n".join(blocks)`

`split_only_context`, body `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/recall_text.py:100`:

- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/recall_text.py:105` `return unparsed`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/recall_text.py:108` `return unparsed`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/recall_text.py:116` `return unparsed`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/recall_text.py:118` `return {`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/recall_text.py:119` `"history": text[:question_at].rstrip(),`

`minimize_context`, body `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/recall_text.py:237`:

- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/recall_text.py:238` `passage_chars = passage_chars_limit()`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/recall_text.py:240` `context_chars = context_chars_limit()`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/recall_text.py:246` `"budget_trimmed": 0,`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/recall_text.py:250` `return (context or "").strip(), stats`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/recall_text.py:255` `out.append(preamble)`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/recall_text.py:268` `passages.append(capped)`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/recall_text.py:280` `blocks.append(PASSAGES_HEADING + "\n" + PASSAGE_SEPARATOR.join(kept_passages))`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/recall_text.py:282` `blocks.append(ENTITIES_HEADING + "\n" + entities)`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/recall_text.py:284` `blocks.append(FACTS_HEADING + "\n" + facts)`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/recall_text.py:285` `return "\n\n".join(blocks).strip()`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/recall_text.py:290` `stats["budget_trimmed"] += 1`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/recall_text.py:292` `return text, stats`

</details>

## 4. Data access boundaries

Direct service access `yes` with the same local/cloud credentials; embedded SDK/store access also bypasses adapter tool schemas. Profile/instance/private-runtime credentials and env exports are listed below. Accountless local mode `yes` with locally configured LLM/embedding services; cloud requires a key. Telemetry default for separately installed core `unknown`, because its implementation is absent.

Boundary model: same OS user, terminal, mounted filesystem and allowed network. Direct reachability means another client can use the same file/endpoint and available credentials, not unauthenticated access or encryption bypass. Hermes setup persists declared secrets through `hermes-agent/hermes_cli/memory_setup.py:321` and `hermes-agent/hermes_cli/config.py:2697`. These are possible configured paths; no live credentials were read.

Host credential-file detail: `get_hermes_home()/.env` is resolved at `hermes-agent/hermes_cli/config.py:473`. New files are secured to 0600 at `hermes-agent/hermes_cli/config.py:506`; the writer preserves an existing mode at `hermes-agent/hermes_cli/config.py:2544` and `:2562`. Therefore 0600 is not an unconditional mode claim for every existing profile.

<details>
<summary>Paths, environment, permissions, egress, telemetry and process-launch evidence</summary>

- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/backend.py:342` `os.environ.setdefault(key, value)`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/backend.py:347` `for key, value in ollama_embedding_pins(os.environ).items():`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/backend.py:348` `os.environ.setdefault(key, value)`

- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/capture_policy.py:7` `mapping instead of ''os.environ'', so a value in ''$HERMES_HOME/cognee.json''`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/capture_policy.py:124` `value = os.environ.get(env_name, "")`

- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/code_graph.py:166` `return isinstance(spec, str) and spec.startswith(("https://", "http://", "git@", "ssh://"))`

- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/config.py:16` `SHARED_PLUGIN_STATE_DIR = Path.home() / ".cognee-plugin"`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/config.py:17` `SHARED_COGNEE_HOME = Path.home() / ".cognee"`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/config.py:68` `''os.environ'' read does not when one gateway serves several profiles (and`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/config.py:73` `value = _hermes_get_secret(name)`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/config.py:75` `value = os.environ.get(name)`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/config.py:235` `"llm_model": os.environ.get("LLM_MODEL", ""),`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/config.py:241` `"embedded": str_to_bool(os.environ.get("COGNEE_EMBEDDED"), False),`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/config.py:244` `"transport": os.environ.get("COGNEE_TRANSPORT", ""),`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/config.py:245` `"local_port": str_to_int(os.environ.get("COGNEE_LOCAL_PORT"), DEFAULT_LOCAL_PORT),`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/config.py:247` `os.environ.get("COGNEE_SERVER_BOOT_TIMEOUT"), DEFAULT_SERVER_BOOT_TIMEOUT`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/config.py:251` `"dataset": os.environ.get("COGNEE_PLUGIN_DATASET")`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/config.py:252` `or os.environ.get("COGNEE_DATASET", DEFAULT_DATASET),`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/config.py:253` `"top_k": str_to_int(os.environ.get("COGNEE_TOP_K"), 5),`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/config.py:254` `"auto_route": str_to_bool(os.environ.get("COGNEE_AUTO_ROUTE"), True),`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/config.py:255` `"improve_on_end": str_to_bool(os.environ.get("COGNEE_IMPROVE_ON_END"), True),`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/config.py:258` `"improve_background": os.environ.get("COGNEE_IMPROVE_BACKGROUND", ""),`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/config.py:259` `"session_prefix": os.environ.get("COGNEE_SESSION_PREFIX", "hermes"),`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/config.py:260` `"data_root": os.environ.get("COGNEE_DATA_ROOT", ""),`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/config.py:261` `"system_root": os.environ.get("COGNEE_SYSTEM_ROOT", ""),`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/config.py:262` `"identity_email": os.environ.get("COGNEE_HERMES_USER_EMAIL", DEFAULT_IDENTITY_EMAIL),`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/config.py:263` `"identity_password": os.environ.get(`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/config.py:267` `"recall_timeout": str_to_int(os.environ.get("COGNEE_RECALL_TIMEOUT"), 120),`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/config.py:268` `"write_timeout": str_to_int(os.environ.get("COGNEE_WRITE_TIMEOUT"), 120),`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/config.py:269` `"improve_timeout": str_to_int(os.environ.get("COGNEE_IMPROVE_TIMEOUT"), 300),`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/config.py:274` `"recall_budget": str_to_int(os.environ.get("COGNEE_RECALL_BUDGET"), 20),`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/config.py:279` `os.environ.get("COGNEE_RECALL_PASSAGE_CHARS"), DEFAULT_RECALL_PASSAGE_CHARS`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/config.py:282` `os.environ.get("COGNEE_RECALL_CONTEXT_CHARS"), DEFAULT_RECALL_CONTEXT_CHARS`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/config.py:285` `"chunk_size": str_to_int(os.environ.get("COGNEE_CHUNK_SIZE"), 0),`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/config.py:288` `"memory_steer": str_to_bool(os.environ.get("COGNEE_MEMORY_STEER"), True),`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/config.py:289` `"memory_steer_text": os.environ.get("COGNEE_MEMORY_STEER_TEXT", ""),`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/config.py:291` `"memory_hits": str_to_bool(os.environ.get("COGNEE_MEMORY_HITS"), True),`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/config.py:293` `"dataset_switch_tool": str_to_bool(os.environ.get("COGNEE_DATASET_SWITCH_TOOL"), True),`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/config.py:294` `"code_search_tool": str_to_bool(os.environ.get("COGNEE_CODE_SEARCH_TOOL"), True),`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/config.py:295` `"recap_tool": str_to_bool(os.environ.get("COGNEE_RECAP_TOOL"), True),`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/config.py:298` `"code_graph_recall": str_to_bool(os.environ.get("COGNEE_CODE_GRAPH_RECALL"), True),`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/config.py:299` `"code_datasets": os.environ.get("COGNEE_CODE_DATASETS", ""),`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/config.py:301` `"update_check": str_to_bool(os.environ.get("COGNEE_UPDATE_CHECK"), True),`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/config.py:302` `"update_check_interval": str_to_int(os.environ.get("COGNEE_UPDATE_CHECK_INTERVAL"), 3600),`

- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/exit_watcher.py:73` `tmp.chmod(0o600)`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/exit_watcher.py:154` `os.close(os.open(str(marker_path(state_path)), os.O_CREAT | os.O_EXCL | os.O_WRONLY, 0o600))`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/exit_watcher.py:164` `env = dict(os.environ)`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/exit_watcher.py:170` `log = subprocess.DEVNULL`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/exit_watcher.py:175` `log = subprocess.DEVNULL`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/exit_watcher.py:177` `subprocess.Popen(`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/exit_watcher.py:186` `stdin=subprocess.DEVNULL,`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/exit_watcher.py:192` `if log is not subprocess.DEVNULL:`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/exit_watcher.py:358` `api_key = os.environ.get("COGNEE_API_KEY", "").strip()`

- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/installer.py:55` `home = Path.home() / ".hermes"`

- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/memory_credit.py:293` `raw = os.environ.get(ENV_CITE_MEMORY, "").strip().lower()`

- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/recall_text.py:69` `raw = os.environ.get(name, "").strip()`

- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/recap.py:303` `return tuple(dict.fromkeys([str(Path.home()), *(e for e in extra if e)]))`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/recap.py:319` `home = PurePath(str(Path.home()))`

- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/runtime_env.py:112` `subprocess.run(command, env=env, check=True, timeout=timeout)`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/runtime_env.py:113` `except subprocess.CalledProcessError as exc:`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/runtime_env.py:115` `except subprocess.TimeoutExpired as exc:`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/runtime_env.py:126` `result = subprocess.run(`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/runtime_env.py:132` `except (OSError, subprocess.TimeoutExpired):`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/runtime_env.py:156` `env = {**os.environ, "UV_PYTHON_DOWNLOADS": "never"}`

- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/server_bootstrap.py:89` `raw = os.environ.get(SERVER_LOG_MAX_BYTES_ENV, "").strip()`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/server_bootstrap.py:183` `env = dict(os.environ)`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/server_bootstrap.py:247` `log = subprocess.DEVNULL`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/server_bootstrap.py:249` `return subprocess.Popen(`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/server_bootstrap.py:269` `if log is not subprocess.DEVNULL:`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/server_bootstrap.py:331` `url = "http://127.0.0.1:%d" % int(port)`

- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/update_check.py:28` `_PYPI_URL = f"https://pypi.org/pypi/{PYPI_PACKAGE}/json"`

- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/write_buffer.py:110` `os.chmod(self.path, 0o600)`

</details>

## 5. Reproducibility

Local engine requirement is exact `cognee==1.6.5` (`sources/cognee/integrations/hermes-agent/cognee_integration_hermes/config.py:52`); pip/uv installer uses it (`sources/cognee/integrations/hermes-agent/cognee_integration_hermes/runtime_env.py:165`). Cloud revision and transitive dependencies are not pinned by that requirement.

Retained lock/checksum files: `sources/cognee/integrations/openclaw/package-lock.json:1`, `sources/cognee/integrations/mastra/package-lock.json:1`, `sources/cognee/integrations/n8n/package-lock.json:1`, `sources/cognee/integrations/openclaw-skills/package-lock.json:1`, `sources/cognee/integrations/vscode/package-lock.json:1`, `sources/cognee/integrations/opencode/package-lock.json:1`. Upstream completeness is unknown: fetch.py:15 excludes .lock files, fetch.py:12 excludes build/vendor directories. Absence here is not absence upstream.

- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/_plugin_root/plugin.yaml:3` `version: 1.4.1`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/_plugin_root/plugin.yaml:4` `description: "Cognee knowledge-graph memory for Hermes: recall, remember, forget and session-end improvement. After installing, run 'hermes memory setup cognee' in a terminal to make it your memory provider, then start a new chat. Remote mode (the default: Cognee Cloud URL + API key) needs nothing else; local mode installs its cognee runtime (cognee==1.6.5, about 1 GB) into a private environment when you choose it."`

- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/installer.py:1` `"""Materialize the Hermes directory plugin from the installed pip package.`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/installer.py:3` `Hermes discovers memory providers through pip entry points and by scanning`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/installer.py:4` `''$HERMES_HOME/plugins/''. The ''cognee-hermes-install'' console script adds`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/installer.py:8` `pip install cognee-integration-hermes-agent`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/installer.py:9` `cognee-hermes-install`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/installer.py:15` `''register_memory_provider'' in its first 8KB). Versions 1.0.0–1.2.0 of this`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/installer.py:16` `installer never laid that file down, so a pip install produced a plugin Hermes`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/installer.py:22` `Because Hermes runs the *copy*, ''pip install -U'' alone changes nothing ,`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/installer.py:23` `re-run ''cognee-hermes-install'' after upgrading. ''hermes cognee status''`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/installer.py:24` `warns when the pip package is newer than the installed copy.`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/installer.py:45` `"after-install.md": "after-install.md",`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/installer.py:50` `def install(hermes_home: str | Path | None = None) -> Path:`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/installer.py:62` `"source-tree install; reinstall the package from PyPI" % ", ".join(missing)`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/installer.py:90` `prog="cognee-hermes-install",`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/installer.py:91` `description="Install the Cognee memory plugin into a Hermes Agent home.",`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/installer.py:96` `help="HERMES_HOME to install into (default: auto-detect, else ~/.hermes)",`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/installer.py:100` `target = install(args.home)`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/installer.py:103` `print(f"Cognee memory plugin installed at {target}")`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/installer.py:105` `print("After a 'pip install -U', re-run 'cognee-hermes-install' to update this copy.")`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/installer.py:109` `if __name__ == "__main__":`

- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/runtime_env.py:5` `whole install is refused. So local mode keeps cognee in a venv of its own and`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/runtime_env.py:9` `The venv is built only on request (''hermes cognee install-runtime'', or the`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/runtime_env.py:11` `starting: it downloads cognee and its dependencies from PyPI. It is built from`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/runtime_env.py:13` `3.10-3.14) with uv when one is on PATH, else stdlib ''venv'' + pip, and uv is`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/runtime_env.py:30` `from .config import LOCAL_COGNEE_REQUIREMENT, SHARED_PLUGIN_STATE_DIR`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/runtime_env.py:32` `try:  # POSIX only; on Windows two concurrent installs are not serialized.`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/runtime_env.py:39` `# Written only after cognee installed and its version was read back, so a build`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/runtime_env.py:40` `# cut short leaves no marker and the next install starts over.`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/runtime_env.py:42` `INSTALL_TIMEOUT_SECS = 1800`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/runtime_env.py:47` `class RuntimeInstallError(RuntimeError):`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/runtime_env.py:51` `def pinned_version() -> str:`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/runtime_env.py:52` `return LOCAL_COGNEE_REQUIREMENT.split("==", 1)[1]`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/runtime_env.py:57` `if os.name == "nt":`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/runtime_env.py:68` `if not isinstance(marker, dict) or marker.get("requirement") != LOCAL_COGNEE_REQUIREMENT:`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/runtime_env.py:78` `version. A cognee installed into Hermes's own environment by hand still`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/runtime_env.py:96` `def _install_lock():`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/runtime_env.py:101` `with open(RUNTIME_DIR / "install.lock", "a") as handle:`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/runtime_env.py:114` `raise RuntimeInstallError(f"{step} failed (exit code {exc.returncode})") from exc`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/runtime_env.py:116` `raise RuntimeInstallError(f"{step} did not finish within {int(timeout)}s") from exc`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/runtime_env.py:118` `raise RuntimeInstallError(f"{step} could not start: {exc}") from exc`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/runtime_env.py:121` `def _installed_version(python: Path, timeout: float = 60) -> str:`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/runtime_env.py:124` `probe = "import importlib.metadata as m; print(m.version('cognee'))"`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/runtime_env.py:134` `return result.stdout.strip() if result.returncode == 0 else ""`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/runtime_env.py:137` `def install_runtime(output: Output = print, timeout: float = INSTALL_TIMEOUT_SECS) -> str:`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/runtime_env.py:140` `Installer output streams straight to the terminal, so the user sees the`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/runtime_env.py:144` `with _install_lock():`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/runtime_env.py:147` `output(f"cognee runtime is ready ({LOCAL_COGNEE_REQUIREMENT}): {VENV_DIR}")`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/runtime_env.py:153` `output(f"Installing {LOCAL_COGNEE_REQUIREMENT} into {VENV_DIR}")`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/runtime_env.py:154` `output("This downloads cognee and its dependencies from PyPI and takes a few minutes.")`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/runtime_env.py:165` `[uv, "pip", "install", "--python", str(python), LOCAL_COGNEE_REQUIREMENT],`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/runtime_env.py:166` `step="Installing cognee with uv",`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/runtime_env.py:178` `[str(python), "-m", "pip", "install", LOCAL_COGNEE_REQUIREMENT],`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/runtime_env.py:179` `step="Installing cognee with pip",`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/runtime_env.py:182` `version = _installed_version(python)`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/runtime_env.py:183` `if version != pinned_version():`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/runtime_env.py:184` `raise RuntimeInstallError(`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/runtime_env.py:185` `f"the venv reports cognee {version or 'missing'}, expected {pinned_version()}"`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/runtime_env.py:188` `json.dumps({"requirement": LOCAL_COGNEE_REQUIREMENT, "python": str(python)}),`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/runtime_env.py:191` `output(f"cognee runtime is ready ({LOCAL_COGNEE_REQUIREMENT}): {VENV_DIR}")`
- `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/runtime_env.py:199` `with _install_lock():`

- `sources/cognee/integrations/hermes-agent/plugin.yaml:3` `version: 1.4.1`
- `sources/cognee/integrations/hermes-agent/plugin.yaml:4` `description: "Cognee knowledge-graph memory for Hermes: recall, remember, forget and session-end improvement. After installing, run 'hermes memory setup cognee' in a terminal to make it your memory provider, then start a new chat. Remote mode (the default: Cognee Cloud URL + API key) needs nothing else; local mode installs its cognee runtime (cognee==1.6.5, about 1 GB) into a private environment when you choose it."`

- `sources/cognee/integrations/hermes-agent/pyproject.toml:2` `requires = ["setuptools>=64", "wheel"]`
- `sources/cognee/integrations/hermes-agent/pyproject.toml:7` `version = "1.4.1"`
- `sources/cognee/integrations/hermes-agent/pyproject.toml:8` `description = "Cognee memory provider plugin for Hermes Agent. After installing, run 'hermes memory setup cognee'; Cognee Cloud needs nothing else, local mode runs cognee from a private environment the plugin installs on request."`
- `sources/cognee/integrations/hermes-agent/pyproject.toml:11` `requires-python = ">=3.10"`
- `sources/cognee/integrations/hermes-agent/pyproject.toml:17` `# install outright , Cloud users included, who never import it. An extra or a`
- `sources/cognee/integrations/hermes-agent/pyproject.toml:18` `# dependency group would not help: Hermes's lock resolves those too. Remote`
- `sources/cognee/integrations/hermes-agent/pyproject.toml:20` `# builds on request (runtime_env.py, LOCAL_COGNEE_REQUIREMENT in config.py).`
- `sources/cognee/integrations/hermes-agent/pyproject.toml:21` `dependencies = []`
- `sources/cognee/integrations/hermes-agent/pyproject.toml:24` `# (plugins/memory ENTRY_POINTS_GROUP): with this, a plain pip install activates`
- `sources/cognee/integrations/hermes-agent/pyproject.toml:25` `# the provider via 'register(ctx)' even before cognee-hermes-install runs.`
- `sources/cognee/integrations/hermes-agent/pyproject.toml:37` `# Copies the installed package into $HERMES_HOME/plugins/cognee, which gets the`
- `sources/cognee/integrations/hermes-agent/pyproject.toml:39` `# (dashboard config panel, 'hermes cognee' subcommands). See installer.py.`
- `sources/cognee/integrations/hermes-agent/pyproject.toml:40` `cognee-hermes-install = "cognee_integration_hermes.installer:main"`
- `sources/cognee/integrations/hermes-agent/pyproject.toml:47` `# The plugin-root files (plugin.yaml, cli.py shim, after-install.md, and`
- `sources/cognee/integrations/hermes-agent/pyproject.toml:48` `# plugin_init.py , installed as the plugin root's __init__.py) that the`
- `sources/cognee/integrations/hermes-agent/pyproject.toml:49` `# installer lays down next to the package. Kept byte-identical to the repo-root`
- `sources/cognee/integrations/hermes-agent/pyproject.toml:50` `# canonical copies by tests/test_installer.py.`
- `sources/cognee/integrations/hermes-agent/pyproject.toml:53` `[dependency-groups]`
- `sources/cognee/integrations/hermes-agent/pyproject.toml:55` `"pytest>=8.0",`
- `sources/cognee/integrations/hermes-agent/pyproject.toml:56` `"pytest-timeout>=2.3",`
- `sources/cognee/integrations/hermes-agent/pyproject.toml:57` `"ruff>=0.14.6",`
- `sources/cognee/integrations/hermes-agent/pyproject.toml:63` `# option pytest-asyncio owns and hermes does not install, so every run, local`

## 6. Tests

Adapter tests include targeted forget find/apply/HTTP behavior and confirmation checks. They mock or call the transport layer; no retained core-engine suite can establish actual derived deletion. Confirmed-ID contract tests exercise targeted HTTP requests, not absent core erasure. (`sources/cognee/integrations/hermes-agent/tests/test_tool_contract.py:268`).

Adapter-related declarations: **718**. Python AST test_ functions, fixtures excluded; literal TS/JS it/test, Go Test*, Rust test_* declarations. No parametrization expansion, collection, execution, or coverage measurement. Other rows are engine test candidates selected by mutation names; names alone do not verify forget coverage.

| file | declarations | adapter-related | forget-related declarations |
|---|---:|---|---|
| `sources/cognee/integrations/chat-memory/tests/test_adapter.py:1` | 14 | no | `test_forget_scope_wipes_everything` `sources/cognee/integrations/chat-memory/tests/test_adapter.py:135`; `test_forget_user_removes_only_that_user` `sources/cognee/integrations/chat-memory/tests/test_adapter.py:145`; `test_forget_me_revokes_consent` `sources/cognee/integrations/chat-memory/tests/test_adapter.py:163`; `test_forget_user_without_conversation_raises` `sources/cognee/integrations/chat-memory/tests/test_adapter.py:173`; `test_forget_with_no_args_raises` `sources/cognee/integrations/chat-memory/tests/test_adapter.py:179` |
| `sources/cognee/integrations/chat-memory/tests/test_cognee_backend.py:1` | 4 | no | `test_forget_user_full_deletes_only_that_user` `sources/cognee/integrations/chat-memory/tests/test_cognee_backend.py:105` |
| `sources/cognee/integrations/chat-memory/tests/test_http_backend.py:1` | 9 | no | `test_forget_scope_posts_forget` `sources/cognee/integrations/chat-memory/tests/test_http_backend.py:114`; `test_forget_user_wipes_dataset_over_http` `sources/cognee/integrations/chat-memory/tests/test_http_backend.py:129`; `test_forget_missing_dataset_4xx_is_noop` `sources/cognee/integrations/chat-memory/tests/test_http_backend.py:166` |
| `sources/cognee/integrations/dify/tests/test_memory_tools.py:1` | 21 | no | `test_forget_body_prefers_dataset_id_and_carries_data_id` `sources/cognee/integrations/dify/tests/test_memory_tools.py:191`; `test_forget_body_memory_only_by_name` `sources/cognee/integrations/dify/tests/test_memory_tools.py:199`; `test_forget_body_everything_needs_an_explicit_true` `sources/cognee/integrations/dify/tests/test_memory_tools.py:207`; `test_forget_invocation_reports_a_malformed_body_as_failure` `sources/cognee/integrations/dify/tests/test_memory_tools.py:315`; `test_forget_invocation_success` `sources/cognee/integrations/dify/tests/test_memory_tools.py:326`; `test_delete_dataset_invocation_with_login_credentials` `sources/cognee/integrations/dify/tests/test_memory_tools.py:341` |
| `sources/cognee/integrations/dify/tests/test_smoke.py:1` | 14 | no | `test_forget_everything_is_form_only` `sources/cognee/integrations/dify/tests/test_smoke.py:95` |
| `sources/cognee/integrations/hermes-agent/tests/_char_helpers.py:1` | 0 | yes | none |
| `sources/cognee/integrations/hermes-agent/tests/conftest.py:1` | 0 | yes | none |
| `sources/cognee/integrations/hermes-agent/tests/test_capture_and_buffer.py:1` | 33 | yes | none |
| `sources/cognee/integrations/hermes-agent/tests/test_capture_policy_conformance.py:1` | 6 | yes | none |
| `sources/cognee/integrations/hermes-agent/tests/test_catalog_install.py:1` | 5 | yes | none |
| `sources/cognee/integrations/hermes-agent/tests/test_cli_keys.py:1` | 6 | yes | none |
| `sources/cognee/integrations/hermes-agent/tests/test_code_graph.py:1` | 17 | yes | none |
| `sources/cognee/integrations/hermes-agent/tests/test_config_contract.py:1` | 46 | yes | none |
| `sources/cognee/integrations/hermes-agent/tests/test_dataset_name_conformance.py:1` | 3 | yes | none |
| `sources/cognee/integrations/hermes-agent/tests/test_dim_mismatch.py:1` | 8 | yes | none |
| `sources/cognee/integrations/hermes-agent/tests/test_exit_watcher.py:1` | 23 | yes | none |
| `sources/cognee/integrations/hermes-agent/tests/test_http_backend.py:1` | 85 | yes | `test_dataset_scoped_delete` `sources/cognee/integrations/hermes-agent/tests/test_http_backend.py:386`; `test_forget_reaches_the_endpoint_and_reports_details` `sources/cognee/integrations/hermes-agent/tests/test_http_backend.py:970` |
| `sources/cognee/integrations/hermes-agent/tests/test_installer.py:1` | 12 | yes | none |
| `sources/cognee/integrations/hermes-agent/tests/test_integration_concurrency.py:1` | 2 | yes | none |
| `sources/cognee/integrations/hermes-agent/tests/test_integration_roundtrip.py:1` | 5 | yes | `test_forget_finds_and_deletes_the_stored_document` `sources/cognee/integrations/hermes-agent/tests/test_integration_roundtrip.py:204` |
| `sources/cognee/integrations/hermes-agent/tests/test_layered_recall.py:1` | 46 | yes | none |
| `sources/cognee/integrations/hermes-agent/tests/test_lifecycle_contract.py:1` | 66 | yes | none |
| `sources/cognee/integrations/hermes-agent/tests/test_local_mode_requirement.py:1` | 13 | yes | none |
| `sources/cognee/integrations/hermes-agent/tests/test_memory_credit.py:1` | 34 | yes | none |
| `sources/cognee/integrations/hermes-agent/tests/test_memory_steer.py:1` | 5 | yes | none |
| `sources/cognee/integrations/hermes-agent/tests/test_openclaw_parity.py:1` | 23 | yes | none |
| `sources/cognee/integrations/hermes-agent/tests/test_overflow_hint.py:1` | 12 | yes | none |
| `sources/cognee/integrations/hermes-agent/tests/test_pre_compress.py:1` | 17 | yes | none |
| `sources/cognee/integrations/hermes-agent/tests/test_recall_result_text.py:1` | 7 | yes | none |
| `sources/cognee/integrations/hermes-agent/tests/test_recall_text_conformance.py:1` | 4 | yes | none |
| `sources/cognee/integrations/hermes-agent/tests/test_recall_wire_contract.py:1` | 9 | yes | none |
| `sources/cognee/integrations/hermes-agent/tests/test_recap.py:1` | 27 | yes | none |
| `sources/cognee/integrations/hermes-agent/tests/test_runtime_env.py:1` | 13 | yes | none |
| `sources/cognee/integrations/hermes-agent/tests/test_sdk_backend.py:1` | 26 | yes | `test_dataset_scoped_delete` `sources/cognee/integrations/hermes-agent/tests/test_sdk_backend.py:159` |
| `sources/cognee/integrations/hermes-agent/tests/test_server_mode.py:1` | 68 | yes | none |
| `sources/cognee/integrations/hermes-agent/tests/test_server_still_starting.py:1` | 7 | yes | none |
| `sources/cognee/integrations/hermes-agent/tests/test_session_id_conformance.py:1` | 1 | yes | none |
| `sources/cognee/integrations/hermes-agent/tests/test_smoke.py:1` | 5 | yes | none |
| `sources/cognee/integrations/hermes-agent/tests/test_switch_and_code_tools.py:1` | 16 | yes | none |
| `sources/cognee/integrations/hermes-agent/tests/test_tool_contract.py:1` | 60 | yes | `test_forget_requires_confirm` `sources/cognee/integrations/hermes-agent/tests/test_tool_contract.py:268`; `test_forget_requires_data_ids` `sources/cognee/integrations/hermes-agent/tests/test_tool_contract.py:275`; `test_forget_deletes_exactly_the_listed_ids` `sources/cognee/integrations/hermes-agent/tests/test_tool_contract.py:282`; `test_a_failed_delete_is_reported_per_id_not_thrown` `sources/cognee/integrations/hermes-agent/tests/test_tool_contract.py:296`; `test_document_delete_resolves_the_dataset_id_by_name` `sources/cognee/integrations/hermes-agent/tests/test_tool_contract.py:547`; `test_an_unknown_dataset_is_an_error_before_any_delete` `sources/cognee/integrations/hermes-agent/tests/test_tool_contract.py:577` |
| `sources/cognee/integrations/hermes-agent/tests/test_update_check.py:1` | 8 | yes | none |
| `sources/cognee/integrations/mastra/__tests__/e2e/tools.test.ts:1` | 18 | no | `calls /forget with memory_only: true forced, and never sends everything: true` `sources/cognee/integrations/mastra/__tests__/e2e/tools.test.ts:261`; `returns cognee_search / cognee_ask / cognee_remember, and NOT cognee_forget, by default` `sources/cognee/integrations/mastra/__tests__/e2e/tools.test.ts:302`; `includes cognee_forget when config.tools.enableForget is true` `sources/cognee/integrations/mastra/__tests__/e2e/tools.test.ts:310`; `includes cognee_forget when COGNEE_ENABLE_FORGET=true is set in the environment` `sources/cognee/integrations/mastra/__tests__/e2e/tools.test.ts:317` |
| `sources/cognee/integrations/mastra/__tests__/live/smoke.live.test.ts:1` | 2 | no | `completes the full ingest -> cognify -> recall -> forget cycle against a real server` `sources/cognee/integrations/mastra/__tests__/live/smoke.live.test.ts:39` |
| `sources/cognee/integrations/mastra/__tests__/unit/config.test.ts:1` | 29 | no | `tools.enableForget / COGNEE_ENABLE_FORGET (boolean parsing)` `sources/cognee/integrations/mastra/__tests__/unit/config.test.ts:212` |
| `sources/cognee/integrations/n8n/tests/Cognee.node.test.ts:1` | 20 | no | `delete.deleteDataset deletes the dataset and replaces the body with {deleted: true}` `sources/cognee/integrations/n8n/tests/Cognee.node.test.ts:77`; `delete.deleteData deletes the data item and replaces the body with {deleted: true}` `sources/cognee/integrations/n8n/tests/Cognee.node.test.ts:84` |
| `sources/cognee/integrations/n8n/tests/memory.test.ts:1` | 18 | no | `blocks Forget Everything without the confirmation toggle` `sources/cognee/integrations/n8n/tests/memory.test.ts:364` |
| `sources/cognee/integrations/n8n/tests/payloads.test.ts:1` | 29 | no | `forgets a dataset by name or by id` `sources/cognee/integrations/n8n/tests/payloads.test.ts:286`; `forgets a single data item and supports memory_only` `sources/cognee/integrations/n8n/tests/payloads.test.ts:295`; `refuses to forget everything without explicit confirmation` `sources/cognee/integrations/n8n/tests/payloads.test.ts:319` |
| `sources/cognee/integrations/openclaw/__tests__/e2e/test_cliCommands.ts:1` | 22 | no | `deletes nothing on a bare invocation and asks for a target` `sources/cognee/integrations/openclaw/__tests__/e2e/test_cliCommands.ts:267` |
| `sources/cognee/integrations/openclaw/__tests__/e2e/test_recallSelfHealing.ts:1` | 12 | no | `invalidates the cache, re-resolves by name, and retries once` `sources/cognee/integrations/openclaw/__tests__/e2e/test_recallSelfHealing.ts:108` |
| `sources/cognee/integrations/openclaw/__tests__/integration/test_clientHttp.ts:1` | 41 | no | `forget by datasetId sends dataset_id + data_id (never the name), one document only` `sources/cognee/integrations/openclaw/__tests__/integration/test_clientHttp.ts:538` |
| `sources/cognee/integrations/openclaw/__tests__/unit/test_forgetTool.ts:1` | 14 | no | `tells the model not to delete on a miss` `sources/cognee/integrations/openclaw/__tests__/unit/test_forgetTool.ts:98`; `refuses without dataIds and without confirm, deleting nothing` `sources/cognee/integrations/openclaw/__tests__/unit/test_forgetTool.ts:139`; `deletes exactly the confirmed ids, one call each, using the dataset from find` `sources/cognee/integrations/openclaw/__tests__/unit/test_forgetTool.ts:149`; `treats a 404 as already deleted and surfaces other errors as failures` `sources/cognee/integrations/openclaw/__tests__/unit/test_forgetTool.ts:169` |
| `sources/cognee/integrations/openclaw/__tests__/unit/test_syncFiles.ts:1` | 44 | no | `deletes removed file with dataId` `sources/cognee/integrations/openclaw/__tests__/unit/test_syncFiles.ts:350`; `handles delete failure` `sources/cognee/integrations/openclaw/__tests__/unit/test_syncFiles.ts:360`; `skips deletion without dataId or datasetId` `sources/cognee/integrations/openclaw/__tests__/unit/test_syncFiles.ts:369`; `handles add, update, skip, delete in one sync` `sources/cognee/integrations/openclaw/__tests__/unit/test_syncFiles.ts:376`; `does not delete unchanged files when called with partial changedFiles` `sources/cognee/integrations/openclaw/__tests__/unit/test_syncFiles.ts:411` |
| `sources/cognee/integrations/second-brain/tests/test_cognee_adapter.py:1` | 7 | no | `test_forget_clears_dataset_and_drops_citations` `sources/cognee/integrations/second-brain/tests/test_cognee_adapter.py:122` |
| `sources/cognee/integrations/second-brain/tests/test_forget.py:1` | 1 | no | `test_forget_me_wipes_across_transports` `sources/cognee/integrations/second-brain/tests/test_forget.py:6` |
| `sources/cognee/integrations/second-brain/tests/test_http_client.py:1` | 10 | no | `test_forget_posts_dataset` `sources/cognee/integrations/second-brain/tests/test_http_client.py:94`; `test_forget_missing_dataset_4xx_is_noop` `sources/cognee/integrations/second-brain/tests/test_http_client.py:102`; `test_forget_5xx_raises` `sources/cognee/integrations/second-brain/tests/test_http_client.py:108` |
| `sources/cognee/integrations/slack/tests/test_cognee_memory.py:1` | 12 | no | `test_forget_deletes_channel_dataset` `sources/cognee/integrations/slack/tests/test_cognee_memory.py:257` |
| `sources/cognee/integrations/slack/tests/test_http_client.py:1` | 8 | no | `test_forget_posts_dataset_and_4xx_is_noop` `sources/cognee/integrations/slack/tests/test_http_client.py:88` |
| `sources/cognee/integrations/slack/tests/test_ingestion_buffer.py:1` | 12 | no | `test_forget_clears_pending_and_delegates_to_adapter` `sources/cognee/integrations/slack/tests/test_ingestion_buffer.py:173` |
| `sources/cognee/integrations/slack/tests/test_slack_commands.py:1` | 8 | no | `test_forget_command_deletes_channel_dataset_and_confirms` `sources/cognee/integrations/slack/tests/test_slack_commands.py:29`; `test_forget_reply_surfaces_channel_level_limitation_honestly` `sources/cognee/integrations/slack/tests/test_slack_commands.py:47` |
| `sources/cognee/integrations/telegram/tests/test_adapter.py:1` | 9 | no | `test_forget_clears_dataset_and_drops_ledger` `sources/cognee/integrations/telegram/tests/test_adapter.py:85` |
| `sources/cognee/integrations/telegram/tests/test_bot.py:1` | 18 | no | `test_forget_command_clears_memory` `sources/cognee/integrations/telegram/tests/test_bot.py:204` |
| `sources/cognee/integrations/telegram/tests/test_http_client.py:1` | 9 | no | `test_forget_posts_dataset` `sources/cognee/integrations/telegram/tests/test_http_client.py:83`; `test_forget_missing_dataset_4xx_is_noop` `sources/cognee/integrations/telegram/tests/test_http_client.py:91`; `test_forget_5xx_raises` `sources/cognee/integrations/telegram/tests/test_http_client.py:97` |
| `sources/cognee/integrations/tests/tests/e2e/test_forget_script.py:1` | 14 | no | `test_forget_posts_dataset_and_data_id` `sources/cognee/integrations/tests/tests/e2e/test_forget_script.py:129`; `test_forget_without_data_id_exits_1` `sources/cognee/integrations/tests/tests/e2e/test_forget_script.py:166`; `test_forget_payload_has_exactly_the_two_expected_fields` `sources/cognee/integrations/tests/tests/e2e/test_forget_script.py:201` |
| `sources/cognee/integrations/tests/tests/unit/test_claude_cloud_worker.py:1` | 36 | no | `test_expiry_archives_and_logs_without_silent_deletion` `sources/cognee/integrations/tests/tests/unit/test_claude_cloud_worker.py:47` |
| `sources/cognee/integrations/tests/tests/unit/test_plugin_keys.py:1` | 8 | no | `test_revoking_this_machines_key_needs_force_and_forgets_it` `sources/cognee/integrations/tests/tests/unit/test_plugin_keys.py:89` |
| `sources/cognee/integrations/tests/tests/unit/test_warmup_drain.py:1` | 21 | no | `test_lock_treats_windows_delete_pending_as_busy` `sources/cognee/integrations/tests/tests/unit/test_warmup_drain.py:163` |
| `sources/cognee/integrations/vscode/src/core/httpClient.test.ts:1` | 6 | no | `posts the dataset and memory_only flag to /api/v1/forget` `sources/cognee/integrations/vscode/src/core/httpClient.test.ts:117` |
| `sources/cognee/integrations/web-widget/tests/test_http_client.py:1` | 10 | no | `test_forget_posts_dataset` `sources/cognee/integrations/web-widget/tests/test_http_client.py:95`; `test_forget_missing_dataset_4xx_is_noop` `sources/cognee/integrations/web-widget/tests/test_http_client.py:102`; `test_forget_is_best_effort_and_swallows_5xx` `sources/cognee/integrations/web-widget/tests/test_http_client.py:108` |
| `sources/cognee/integrations/web-widget/tests/test_web_widget.py:1` | 13 | no | `test_forget_clears_only_this_conversation` `sources/cognee/integrations/web-widget/tests/test_web_widget.py:157`; `test_chat_forget_command_is_not_answered` `sources/cognee/integrations/web-widget/tests/test_web_widget.py:195`; `test_forget_endpoint_clears_conversation` `sources/cognee/integrations/web-widget/tests/test_web_widget.py:206` |

## 7. Corrections to B2

Old values below are copied from the triage row (TRIAGE.md), not reconstructed from memory. A refinement narrows an over-broad field; it need not reverse its original value. Counts are distinct field paths, not the number of factual errors.

- `engine_delete.e`: `["integrations/hermes-agent/cognee_integration_hermes/backend.py:566"]` → This is SDK call-through, not core-engine deletion. HTTP targeted forget exists; embedded targeted deletion raises; core source absent. Evidence: `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/backend.py:582`, `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/backend.py:224`.
- `forget_tool.scope`: `"Dataset-scoped forgetting and raw inputs need separate deep review."` → Actual tool is confirmed per-document HTTP deletion, not unrestricted all-memory purge; separate wider SDK method is not the tool. Evidence: `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/provider.py:1969`, `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/http_backend.py:901`.
- `engine_delete.v`: `"yes"` → not_in_repo for the core engine; implemented adapter call-through and targeted embedded stub are distinguished. Evidence: `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/backend.py:224`, `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/http_backend.py:901`.
- `engine.v`: `"same repo Cognee engine; installed integration dependencies"` → external installed core, not same-repo engine in this retained snapshot; local installer pins cognee==1.6.5. Evidence: `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/config.py:52`, `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/runtime_env.py:165`.
- `engine_delete.scope`: `"Named deletion/retraction endpoint or method, not a guarantee of physical erasure. Its reach may overlap an agent tool; lack of a native tool is counted separately."` → `yes` for HTTP mode: cognee_forget action=find lists candidate documents, action=forget deletes confirmed data_ids (`sources/cognee/integrations/hermes-agent/cognee_integration_hermes/provider.py:1966`; `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/provider.py:1969`). HTTP forget_document POSTs `/api/v1/forget` with datasetId/dataId (`sources/cognee/integrations/hermes-agent/cognee_integration_hermes/http_backend.py:901`). Embedded per-document method is `stub`: raises HTTP-only error (`sources/cognee/integrations/hermes-agent/cognee_integration_hermes/backend.py:224`). Wider SDK `forget(dataset/everything/memory_only)` is implemented as call-through to cognee.forget (`sources/cognee/integrations/hermes-agent/cognee_integration_hermes/backend.py:582`). Engine body `not_in_repo`; matrix not_in_repo refers to absent core; request/call-through is present. Derived cleanup `unknown`: session-cache cleanup is a version-dependent contract, not verified core code here. Evidence: `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/provider.py:1966`, `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/provider.py:1969`, `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/http_backend.py:901`, `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/backend.py:224`, `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/backend.py:582`.
- `context.mechanism.v`: `"Source expressions below identify retrieval, cached/static returns, lifecycle capture, and limits. A hook definition is not a guarantee it emits nonempty context each turn; branches and configuration apply."` → System instructions describe recall/remember/forget (`sources/cognee/integrations/hermes-agent/cognee_integration_hermes/provider.py:790`). Prefetch waits up to three seconds for cached work, consumes result and wraps it with `## Cognee Memory` and optional steering (`sources/cognee/integrations/hermes-agent/cognee_integration_hermes/provider.py:808`; `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/provider.py:818`). Formatter/config limits are quoted below; context depends on previous background work, not an unconditional new search at return time. Evidence: `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/provider.py:790`, `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/provider.py:808`, `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/provider.py:818`.
- `sec.telemetry.v`: `"unknown: no emitter identified in the selected adapter files; external dependencies not cleared"` → Direct service access `yes` with the same local/cloud credentials; embedded SDK/store access also bypasses adapter tool schemas. Profile/instance/private-runtime credentials and env exports are listed below. Accountless local mode `yes` with locally configured LLM/embedding services; cloud requires a key. Telemetry default for separately installed core `unknown`, because its implementation is absent. Evidence: `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/backend.py:342`, `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/backend.py:347`, `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/backend.py:348`, `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/capture_policy.py:7`.
- `repro.decls`: `["integrations/hermes-agent/cognee_integration_hermes/_plugin_root/plugin.yaml:4", "integrations/hermes-agent/cognee_integration_hermes/_plugin_root/plugin.yaml:6", "integrations/hermes-agent/cognee_integration_hermes/_plugin_root/plugin.yaml:7", "integrations/hermes-agent/cognee_integration_hermes/_plugin_root/plugin.yaml:8", "integrations/hermes-agent/cognee_integration_hermes/_plugin_root/plugin.yaml:12", "integrations/hermes-agent/plugin.yaml:4", "integrations/hermes-agent/plugin.yaml:6", "integrations/hermes-agent/plugin.yaml:7", "integrations/hermes-agent/plugin.yaml:8", "integrations/hermes-agent/plugin.yaml:12", "integrations/hermes-agent/pyproject.toml:2", "integrations/hermes-agent/pyproject.toml:7", "integrations/hermes-agent/pyproject.toml:11", "integrations/hermes-agent/pyproject.toml:21", "integrations/hermes-agent/pyproject.toml:55", "integrations/hermes-agent/pyproject.toml:56", "integrations/hermes-agent/pyproject.toml:57", "pyproject.toml:2"]` → Local engine requirement is exact `cognee==1.6.5` (`sources/cognee/integrations/hermes-agent/cognee_integration_hermes/config.py:52`); pip/uv installer uses it (`sources/cognee/integrations/hermes-agent/cognee_integration_hermes/runtime_env.py:165`). Cloud revision and transitive dependencies are not pinned by that requirement. Evidence: `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/config.py:52`, `sources/cognee/integrations/hermes-agent/cognee_integration_hermes/runtime_env.py:165`.
- `tests.count`: `3862` → 718 adapter-related declarations using the stated parser/scope; B2 counted a different, repository-wide inventory. This is a scope refinement, not a count of executed or passing tests. Adapter tests include targeted forget find/apply/HTTP behavior and confirmation checks. They mock or call the transport layer; no retained core-engine suite can establish actual derived deletion. Confirmed-ID contract tests exercise targeted HTTP requests, not absent core erasure. (`sources/cognee/integrations/hermes-agent/tests/test_tool_contract.py:268`). Evidence: `sources/cognee/integrations/hermes-agent/tests/test_tool_contract.py:268`.

MATRIX counts these 9 explicit field corrections/refinements. Sections 1-6 provide the body and test inventories. Test-name candidates are not passing tests or measured coverage.
