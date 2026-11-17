# memanto: B3 static correctness review

Snapshot `21554fe8ac40685a45f222022b08e58459760b7d` (MANIFEST.json row `memanto`). No plugin execution or passing-test claims.

## 1. Architecture

`provider`: in-process Python client with a background warmup/capture thread; _MemantoClient is constructed with MOORCHEH_API_KEY and an agent ID. The storage service is Moorcheh, not a local SQLite memory engine (`sources/memanto/integrations/hermes-agents/hermes_memanto/provider.py:643`; `sources/memanto/integrations/hermes-agents/hermes_memanto/provider.py:667`).

Hook/body inventory (no-op/fallback declarations are not active behavior):

| callable | declaration | first executable body line |
|---|---|---|
| `register` | `sources/memanto/integrations/hermes-agents/hermes_memanto/provider.py:952` | `sources/memanto/integrations/hermes-agents/hermes_memanto/provider.py:953` |
| `is_available` | `sources/memanto/integrations/hermes-agents/hermes_memanto/provider.py:600` | `sources/memanto/integrations/hermes-agents/hermes_memanto/provider.py:601` |
| `get_config_schema` | `sources/memanto/integrations/hermes-agents/hermes_memanto/provider.py:609` | `sources/memanto/integrations/hermes-agents/hermes_memanto/provider.py:610` |
| `save_config` | `sources/memanto/integrations/hermes-agents/hermes_memanto/provider.py:628` | `sources/memanto/integrations/hermes-agents/hermes_memanto/provider.py:629` |
| `initialize` | `sources/memanto/integrations/hermes-agents/hermes_memanto/provider.py:640` | `sources/memanto/integrations/hermes-agents/hermes_memanto/provider.py:641` |
| `system_prompt_block` | `sources/memanto/integrations/hermes-agents/hermes_memanto/provider.py:697` | `sources/memanto/integrations/hermes-agents/hermes_memanto/provider.py:698` |
| `prefetch` | `sources/memanto/integrations/hermes-agents/hermes_memanto/provider.py:708` | `sources/memanto/integrations/hermes-agents/hermes_memanto/provider.py:709` |
| `sync_turn` | `sources/memanto/integrations/hermes-agents/hermes_memanto/provider.py:727` | `sources/memanto/integrations/hermes-agents/hermes_memanto/provider.py:730` |
| `on_memory_write` | `sources/memanto/integrations/hermes-agents/hermes_memanto/provider.py:776` | `sources/memanto/integrations/hermes-agents/hermes_memanto/provider.py:783` |
| `shutdown` | `sources/memanto/integrations/hermes-agents/hermes_memanto/provider.py:822` | `sources/memanto/integrations/hermes-agents/hermes_memanto/provider.py:823` |
| `get_tool_schemas` | `sources/memanto/integrations/hermes-agents/hermes_memanto/provider.py:831` | `sources/memanto/integrations/hermes-agents/hermes_memanto/provider.py:832` |
| `handle_tool_call` | `sources/memanto/integrations/hermes-agents/hermes_memanto/provider.py:940` | `sources/memanto/integrations/hermes-agents/hermes_memanto/provider.py:941` |

## 2. Forget paths and derived copies

`none`: recall, remember and answer are the adapter surface, with no agent-facing forget (`sources/memanto/integrations/hermes-agents/hermes_memanto/provider.py:703`). Engine `yes` at the Memanto service/client layer: DELETE `/{agent_id}/memories/{memory_id}` invokes memory-write service (`sources/memanto/memanto/app/routes/memory.py:899`). Write-service body calls Moorcheh `documents.delete(namespace_name=..., ids=[memory_id])` (`sources/memanto/memanto/app/services/memory_write_service.py:583`). CLI forget and direct/SDK clients, agent deletion, expire and purge-expired are separate operations listed below. Derived erasure `unknown`: the final Moorcheh backend is not present, and one document removal does not establish summary/cache/host cleanup. Expiry is a lifecycle change, not physical deletion.

Host state.db is independent (`hermes-agent/hermes_state.py:181`). No reviewed adapter forget body performs a general purge of that database or built-in MEMORY.md/USER.md. Prompt/tool-result copies and independent source transcripts remain separate; source operation bodies below establish the scope of each plugin mutation.

<details>
<summary>Deletion, invalidation, archive and reset body ledger</summary>

Public routes, SDK/CLI dispatch and internal storage operations are distinguished by their source. Cache invalidation and management-object deletion are not additional fact-forget tools. A stub is not working deletion. Call-through means the downstream body still determines erasure. HTTP prefixes/selectors must be read with route registration.

**edit_memory**, body `sources/memanto/memanto/app/routes/memory.py:598`; implementation/call-through.

- `sources/memanto/memanto/app/routes/memory.py:582` `@router.patch("/{agent_id}/memories/{memory_id}")`
- `sources/memanto/memanto/app/routes/memory.py:583` `async def edit_memory(`
- `sources/memanto/memanto/app/routes/memory.py:598` `enforce_session_scope(session, agent_id)`

**delete_memory**, body `sources/memanto/memanto/app/routes/memory.py:895`; implementation/call-through.

- `sources/memanto/memanto/app/routes/memory.py:880` `@router.delete("/{agent_id}/memories/{memory_id}")`
- `sources/memanto/memanto/app/routes/memory.py:881` `async def delete_memory(`
- `sources/memanto/memanto/app/routes/memory.py:900` `write_service.delete_memory,`
- `sources/memanto/memanto/app/routes/memory.py:915` `"status": "deleted",`

**purge_expired_memories**, body `sources/memanto/memanto/app/routes/memory.py:1715`; implementation/call-through.

- `sources/memanto/memanto/app/routes/memory.py:1699` `@router.post("/{agent_id}/policy/purge")`
- `sources/memanto/memanto/app/routes/memory.py:1700` `async def purge_expired_memories(`
- `sources/memanto/memanto/app/routes/memory.py:1720` `service.purge_expired, agent_id, dry_run=request.dry_run`

**delete_agent**, body `sources/memanto/memanto/app/routes/sessions.py:207`; implementation/call-through.

- `sources/memanto/memanto/app/routes/sessions.py:193` `@router.delete("/agents/{agent_id}", status_code=200)`
- `sources/memanto/memanto/app/routes/sessions.py:194` `def delete_agent(`
- `sources/memanto/memanto/app/routes/sessions.py:218` `moorcheh_client.namespaces.delete(namespace_name=agent.namespace)`
- `sources/memanto/memanto/app/routes/sessions.py:227` `get_session_service().delete_session(agent_id)`
- `sources/memanto/memanto/app/routes/sessions.py:228` `agent_service.delete_agent(agent_id)`

**delete_agent**, body `sources/memanto/memanto/app/services/agent_service.py:234`; implementation/call-through.

- `sources/memanto/memanto/app/services/agent_service.py:224` `def delete_agent(self, agent_id: str) -> None:`
- `sources/memanto/memanto/app/services/agent_service.py:241` `agent_file.unlink()`

**_validate_purge**, body `sources/memanto/memanto/app/services/memory_policy_service.py:187`; implementation/call-through.

- `sources/memanto/memanto/app/services/memory_policy_service.py:183` `@field_validator("purge_expired_after", mode="before")`
- `sources/memanto/memanto/app/services/memory_policy_service.py:184` `@classmethod`
- `sources/memanto/memanto/app/services/memory_policy_service.py:185` `def _validate_purge(cls, value: Any) -> Any:`
- `sources/memanto/memanto/app/services/memory_policy_service.py:187` `parse_duration(value)`

**purge_expired_after_seconds**, body `sources/memanto/memanto/app/services/memory_policy_service.py:204`; implementation/call-through.

- `sources/memanto/memanto/app/services/memory_policy_service.py:201` `@property`
- `sources/memanto/memanto/app/services/memory_policy_service.py:202` `def purge_expired_after_seconds(self) -> int | None:`
- `sources/memanto/memanto/app/services/memory_policy_service.py:204` `return parse_duration(self.purge_expired_after)`

**purge_expired**, body `sources/memanto/memanto/app/services/memory_policy_service.py:449`; implementation/call-through.

- `sources/memanto/memanto/app/services/memory_policy_service.py:437` `def purge_expired(`
- `sources/memanto/memanto/app/services/memory_policy_service.py:450` `window = policy.purge_expired_after_seconds`
- `sources/memanto/memanto/app/services/memory_policy_service.py:493` `write_service.delete_memory(str(memory_id), namespace)`
- `sources/memanto/memanto/app/services/memory_policy_service.py:502` `"purge_expired_after": policy.purge_expired_after,`

**delete_memory**, body `sources/memanto/memanto/app/services/memory_write_service.py:578`; implementation/call-through.

- `sources/memanto/memanto/app/services/memory_write_service.py:575` `def delete_memory(self, memory_id: str, namespace: str) -> bool:`
- `sources/memanto/memanto/app/services/memory_write_service.py:583` `self.client.documents.delete(namespace_name=namespace, ids=[memory_id]),`

**_deletion_succeeded**, body `sources/memanto/memanto/app/services/memory_write_service.py:594`; implementation/call-through.

- `sources/memanto/memanto/app/services/memory_write_service.py:591` `@staticmethod`
- `sources/memanto/memanto/app/services/memory_write_service.py:592` `def _deletion_succeeded(result: dict[str, Any]) -> bool:`
- `sources/memanto/memanto/app/services/memory_write_service.py:594` `raw = result.get("actual_deletions")`

**log_memory_deletion_to_session_summary**, body `sources/memanto/memanto/app/services/session_service.py:804`; implementation/call-through.

- `sources/memanto/memanto/app/services/session_service.py:790` `def log_memory_deletion_to_session_summary(`
- `sources/memanto/memanto/app/services/session_service.py:804` `validate_safe_id(agent_id, "agent_id")`

**try_log_memory_deletion_to_session_summary**, body `sources/memanto/memanto/app/services/session_service.py:859`; implementation/call-through.

- `sources/memanto/memanto/app/services/session_service.py:852` `def try_log_memory_deletion_to_session_summary(`
- `sources/memanto/memanto/app/services/session_service.py:859` `try:`

**delete_session**, body `sources/memanto/memanto/app/services/session_service.py:913`; implementation/call-through.

- `sources/memanto/memanto/app/services/session_service.py:906` `def delete_session(self, agent_id: str) -> bool:`
- `sources/memanto/memanto/app/services/session_service.py:933` `session_file.unlink(missing_ok=True)`

**update_ui_config**, body `sources/memanto/memanto/app/ui/routes/ui_router.py:240`; implementation/call-through.

- `sources/memanto/memanto/app/ui/routes/ui_router.py:232` `@router.patch("/api/ui/config")`
- `sources/memanto/memanto/app/ui/routes/ui_router.py:233` `async def update_ui_config(updates: dict, _: None = Depends(_require_local)):`
- `sources/memanto/memanto/app/ui/routes/ui_router.py:240` `allowed_keys = {"schedule_time", "session", "cli", "server", "answer", "recall"}`

**delete_agent**, body `sources/memanto/memanto/cli/client/direct_client.py:525`; implementation/call-through.

- `sources/memanto/memanto/cli/client/direct_client.py:515` `def delete_agent(self, agent_id: str) -> dict[str, Any]:`
- `sources/memanto/memanto/cli/client/direct_client.py:526` `self._get_agent_service().delete_agent(agent_id)`
- `sources/memanto/memanto/cli/client/direct_client.py:527` `self._get_session_service().delete_session(agent_id)`
- `sources/memanto/memanto/cli/client/direct_client.py:532` `return {"status": "deleted", "agent_id": agent_id}`

**delete_memory**, body `sources/memanto/memanto/cli/client/direct_client.py:997`; implementation/call-through.

- `sources/memanto/memanto/cli/client/direct_client.py:982` `def delete_memory(self, agent_id: str, memory_id: str) -> dict[str, Any]:`
- `sources/memanto/memanto/cli/client/direct_client.py:1006` `deleted = self._get_write_service().delete_memory(memory_id, namespace)`
- `sources/memanto/memanto/cli/client/direct_client.py:1021` `"status": "deleted",`

**purge_expired**, body `sources/memanto/memanto/cli/client/direct_client.py:1152`; implementation/call-through.

- `sources/memanto/memanto/cli/client/direct_client.py:1150` `def purge_expired(self, agent_id: str, dry_run: bool = False) -> dict[str, Any]:`
- `sources/memanto/memanto/cli/client/direct_client.py:1153` `return self._get_policy_service().purge_expired(agent_id, dry_run=dry_run)`

**delete**, body `sources/memanto/memanto/cli/client/direct_client.py:128`; implementation/call-through.

- `sources/memanto/memanto/cli/client/direct_client.py:127` `def delete(self, namespace_name, ids):`
- `sources/memanto/memanto/cli/client/direct_client.py:128` `return self.client._request(`

**delete_agent**, body `sources/memanto/memanto/cli/client/sdk_client.py:357`; implementation/call-through.

- `sources/memanto/memanto/cli/client/sdk_client.py:347` `def delete_agent(self, agent_id: str) -> dict[str, Any]:`
- `sources/memanto/memanto/cli/client/sdk_client.py:358` `self._get_agent_service().delete_agent(agent_id)`
- `sources/memanto/memanto/cli/client/sdk_client.py:359` `self._get_session_service().delete_session(agent_id)`
- `sources/memanto/memanto/cli/client/sdk_client.py:364` `return {"status": "deleted", "agent_id": agent_id}`

**delete_memory**, body `sources/memanto/memanto/cli/client/sdk_client.py:823`; implementation/call-through.

- `sources/memanto/memanto/cli/client/sdk_client.py:808` `def delete_memory(self, agent_id: str, memory_id: str) -> dict[str, Any]:`
- `sources/memanto/memanto/cli/client/sdk_client.py:832` `deleted = self._get_write_service().delete_memory(memory_id, namespace)`
- `sources/memanto/memanto/cli/client/sdk_client.py:847` `"status": "deleted",`

**purge_expired**, body `sources/memanto/memanto/cli/client/sdk_client.py:978`; implementation/call-through.

- `sources/memanto/memanto/cli/client/sdk_client.py:976` `def purge_expired(self, agent_id: str, dry_run: bool = False) -> dict[str, Any]:`
- `sources/memanto/memanto/cli/client/sdk_client.py:979` `return self._get_policy_service().purge_expired(agent_id, dry_run=dry_run)`

**agent_delete**, body `sources/memanto/memanto/cli/commands/agent.py:163`; implementation/call-through.

- `sources/memanto/memanto/cli/commands/agent.py:149` `@agent_app.command("delete")`
- `sources/memanto/memanto/cli/commands/agent.py:150` `def agent_delete(`
- `sources/memanto/memanto/cli/commands/agent.py:201` `client.delete_agent(agent_id)`
- `sources/memanto/memanto/cli/commands/agent.py:214` `client._get_moorcheh().namespaces.delete(namespace)`

**forget**, body `sources/memanto/memanto/cli/commands/memory.py:513`; implementation/call-through.

- `sources/memanto/memanto/cli/commands/memory.py:498` `@app.command()`
- `sources/memanto/memanto/cli/commands/memory.py:499` `def forget(`
- `sources/memanto/memanto/cli/commands/memory.py:531` `result = client.delete_memory(`

**policy_purge**, body `sources/memanto/memanto/cli/commands/policy.py:399`; implementation/call-through.

- `sources/memanto/memanto/cli/commands/policy.py:380` `@policy_app.command("purge")`
- `sources/memanto/memanto/cli/commands/policy.py:381` `def policy_purge(`
- `sources/memanto/memanto/cli/commands/policy.py:404` `preview = client.purge_expired(agent_id, dry_run=True)`
- `sources/memanto/memanto/cli/commands/policy.py:449` `report = client.purge_expired(agent_id, dry_run=False)`

</details>

## 3. What reaches the agent

System block names the active memory agent and three tools (`sources/memanto/integrations/hermes-agents/hermes_memanto/provider.py:700`). Prefetch clips query to 2000 characters and requests configured result count/confidence (`sources/memanto/integrations/hermes-agents/hermes_memanto/provider.py:717`). The actual formatting and output cap are in the assembly quotes below; query/input caps are not output caps. Tool results are client recall/remember/answer envelopes. `_format_recall_block` builds type/confidence bullets and `<memanto-memory>` framing with no character cap, only max_results (default 10, configuration clamped to 100) (`sources/memanto/integrations/hermes-agents/hermes_memanto/provider.py:266`; `sources/memanto/integrations/hermes-agents/hermes_memanto/provider.py:274`; `sources/memanto/integrations/hermes-agents/hermes_memanto/provider.py:193`).

Host assembly: `hermes-agent/agent/memory_manager.py:352` joins provider system blocks. `hermes-agent/agent/turn_context.py:1081` requests prefetch for nontrivial turns; `hermes-agent/agent/turn_context.py:120` fences it; `hermes-agent/agent/turn_context.py:136` appends it to API-bound user content. Tool results are a third channel. Optional host spill is at `hermes-agent/agent/memory_manager.py:419`, not a universal provider cap. Built-in memory remains separately controlled (`hermes-agent/agent/system_prompt.py:542`).

<details>
<summary>Context and tool-result assembly quotes</summary>

`system_prompt_block`, body `sources/memanto/integrations/hermes-agents/hermes_memanto/provider.py:698`:

- `sources/memanto/integrations/hermes-agents/hermes_memanto/provider.py:699` `return ""`
- `sources/memanto/integrations/hermes-agents/hermes_memanto/provider.py:700` `return (`

`prefetch`, body `sources/memanto/integrations/hermes-agents/hermes_memanto/provider.py:709`:

- `sources/memanto/integrations/hermes-agents/hermes_memanto/provider.py:715` `return ""`
- `sources/memanto/integrations/hermes-agents/hermes_memanto/provider.py:718` `query[:2000],`
- `sources/memanto/integrations/hermes-agents/hermes_memanto/provider.py:719` `limit=self._max_recall_results,`
- `sources/memanto/integrations/hermes-agents/hermes_memanto/provider.py:722` `return _format_recall_block(memories, self._max_recall_results)`
- `sources/memanto/integrations/hermes-agents/hermes_memanto/provider.py:725` `return ""`

`get_tool_schemas`, body `sources/memanto/integrations/hermes-agents/hermes_memanto/provider.py:832`:

- `sources/memanto/integrations/hermes-agents/hermes_memanto/provider.py:832` `return [REMEMBER_SCHEMA, RECALL_SCHEMA, ANSWER_SCHEMA]`

`_tool_remember`, body `sources/memanto/integrations/hermes-agents/hermes_memanto/provider.py:835`:

- `sources/memanto/integrations/hermes-agents/hermes_memanto/provider.py:837` `return tool_error("content is required")`
- `sources/memanto/integrations/hermes-agents/hermes_memanto/provider.py:850` `content[: _MAX_TITLE_LENGTH - 3] + "..."`
- `sources/memanto/integrations/hermes-agents/hermes_memanto/provider.py:851` `if len(content) > _MAX_TITLE_LENGTH`
- `sources/memanto/integrations/hermes-agents/hermes_memanto/provider.py:864` `return json.dumps(`
- `sources/memanto/integrations/hermes-agents/hermes_memanto/provider.py:873` `return tool_error(f"Failed to store memory: {exc}")`

`_tool_recall`, body `sources/memanto/integrations/hermes-agents/hermes_memanto/provider.py:876`:

- `sources/memanto/integrations/hermes-agents/hermes_memanto/provider.py:878` `return tool_error("query is required")`
- `sources/memanto/integrations/hermes-agents/hermes_memanto/provider.py:880` `limit = max(`
- `sources/memanto/integrations/hermes-agents/hermes_memanto/provider.py:885` `args.get("limit", self._max_recall_results)`
- `sources/memanto/integrations/hermes-agents/hermes_memanto/provider.py:886` `or self._max_recall_results`
- `sources/memanto/integrations/hermes-agents/hermes_memanto/provider.py:891` `limit = self._max_recall_results`
- `sources/memanto/integrations/hermes-agents/hermes_memanto/provider.py:898` `limit=limit,`
- `sources/memanto/integrations/hermes-agents/hermes_memanto/provider.py:912` `formatted.append(entry)`
- `sources/memanto/integrations/hermes-agents/hermes_memanto/provider.py:913` `return json.dumps({"results": formatted, "count": len(formatted)})`
- `sources/memanto/integrations/hermes-agents/hermes_memanto/provider.py:915` `return tool_error(f"Recall failed: {exc}")`

`_tool_answer`, body `sources/memanto/integrations/hermes-agents/hermes_memanto/provider.py:918`:

- `sources/memanto/integrations/hermes-agents/hermes_memanto/provider.py:920` `return tool_error("question is required")`
- `sources/memanto/integrations/hermes-agents/hermes_memanto/provider.py:921` `limit = args.get("limit")`
- `sources/memanto/integrations/hermes-agents/hermes_memanto/provider.py:922` `if limit is not None:`
- `sources/memanto/integrations/hermes-agents/hermes_memanto/provider.py:924` `limit = max(1, min(100, int(limit)))`
- `sources/memanto/integrations/hermes-agents/hermes_memanto/provider.py:926` `limit = None`
- `sources/memanto/integrations/hermes-agents/hermes_memanto/provider.py:928` `result = self._client.answer(question, limit=limit)`
- `sources/memanto/integrations/hermes-agents/hermes_memanto/provider.py:930` `return json.dumps(`
- `sources/memanto/integrations/hermes-agents/hermes_memanto/provider.py:938` `return tool_error(f"Answer failed: {exc}")`

`handle_tool_call`, body `sources/memanto/integrations/hermes-agents/hermes_memanto/provider.py:941`:

- `sources/memanto/integrations/hermes-agents/hermes_memanto/provider.py:942` `return tool_error("Memanto is not configured")`
- `sources/memanto/integrations/hermes-agents/hermes_memanto/provider.py:944` `return self._tool_remember(args)`
- `sources/memanto/integrations/hermes-agents/hermes_memanto/provider.py:946` `return self._tool_recall(args)`
- `sources/memanto/integrations/hermes-agents/hermes_memanto/provider.py:948` `return self._tool_answer(args)`
- `sources/memanto/integrations/hermes-agents/hermes_memanto/provider.py:949` `return tool_error(f"Unknown tool: {tool_name}")`

</details>

## 4. Data access boundaries

MOORCHEH_API_KEY is required for activation; empty key returns without becoming active (`sources/memanto/integrations/hermes-agents/hermes_memanto/provider.py:664`). Cloud/default accountless mode `no`; a usable accountless on-prem stack is `unknown`, since MOORCHEH_BASE_URL exists but the external on-prem engine/account semantics are not retained (`sources/memanto/memanto/cli/client/direct_client.py:80`). A required API key alone does not prove a hosted account requirement. Profile directories are set on the client (`sources/memanto/integrations/hermes-agents/hermes_memanto/provider.py:674`). Direct service/client access `yes` with the same key. Telemetry default `unknown` across Moorcheh SDK; local wrapper logging is not proof of telemetry off.

Boundary model: same OS user, terminal, mounted filesystem and allowed network. Direct reachability means another client can use the same file/endpoint and available credentials, not unauthenticated access or encryption bypass. Hermes setup persists declared secrets through `hermes-agent/hermes_cli/memory_setup.py:321` and `hermes-agent/hermes_cli/config.py:2697`. These are possible configured paths; no live credentials were read.

Host credential-file detail: `get_hermes_home()/.env` is resolved at `hermes-agent/hermes_cli/config.py:473`. New files are secured to 0600 at `hermes-agent/hermes_cli/config.py:506`; the writer preserves an existing mode at `hermes-agent/hermes_cli/config.py:2544` and `:2562`. Therefore 0600 is not an unconditional mode claim for every existing profile.

<details>
<summary>Paths, environment, permissions, egress, telemetry and process-launch evidence</summary>

- `sources/memanto/integrations/hermes-agents/hermes_memanto/install.py:35` `env = os.environ.get("HERMES_HOME")`
- `sources/memanto/integrations/hermes-agents/hermes_memanto/install.py:38` `return Path.home() / ".hermes"`
- `sources/memanto/integrations/hermes-agents/hermes_memanto/install.py:89` `print("  2. export MOORCHEH_API_KEY=...   (https://console.moorcheh.ai/api-keys)")`

- `sources/memanto/integrations/hermes-agents/hermes_memanto/provider.py:3` `Memanto (https://memanto.ai) is a *memory agent*: typed long-term memory with`
- `sources/memanto/integrations/hermes-agents/hermes_memanto/provider.py:127` `return os.environ.get("HERMES_HOME") or str(Path.home() / ".hermes")`
- `sources/memanto/integrations/hermes-agents/hermes_memanto/provider.py:333` `0o600,`
- `sources/memanto/integrations/hermes-agents/hermes_memanto/provider.py:338` `os.chmod(self._token_file, 0o600)`
- `sources/memanto/integrations/hermes-agents/hermes_memanto/provider.py:601` `if not os.environ.get("MOORCHEH_API_KEY", "").strip():`
- `sources/memanto/integrations/hermes-agents/hermes_memanto/provider.py:617` `"url": "https://console.moorcheh.ai/api-keys",`
- `sources/memanto/integrations/hermes-agents/hermes_memanto/provider.py:643` `self._api_key = os.environ.get("MOORCHEH_API_KEY", "").strip()`
- `sources/memanto/integrations/hermes-agents/hermes_memanto/provider.py:649` `os.environ.get("MEMANTO_AGENT_ID", "").strip() or self._config["agent_id"]`

- `sources/memanto/memanto/app/config.py:21` `_memanto_env = Path.home() / ".memanto" / ".env"`
- `sources/memanto/memanto/app/config.py:26` `_config_file = Path.home() / ".memanto" / "config.yaml"`
- `sources/memanto/memanto/app/config.py:39` `os.environ["ANSWER_MODEL"] = _ans_model`
- `sources/memanto/memanto/app/config.py:42` `os.environ["ANSWER_TEMPERATURE"] = str(_ans_temp)`
- `sources/memanto/memanto/app/config.py:45` `os.environ["ANSWER_LIMIT"] = str(_ans_limit)`
- `sources/memanto/memanto/app/config.py:51` `os.environ["SUMMARY_MODEL"] = _sum_model`
- `sources/memanto/memanto/app/config.py:57` `os.environ["AUTO_PARSE_ENABLED"] = str(_smart_parse)`
- `sources/memanto/memanto/app/config.py:72` `os.environ.setdefault(_env_key, str(_toggle))`
- `sources/memanto/memanto/app/config.py:77` `os.environ["MEMANTO_BACKEND"] = str(_backend)`
- `sources/memanto/memanto/app/config.py:86` `_state_path = Path.home() / ".memanto" / "on-prem" / "state.json"`
- `sources/memanto/memanto/app/config.py:91` `os.environ["MOORCHEH_ONPREM_URL"] = str(_op_url)`
- `sources/memanto/memanto/app/config.py:94` `os.environ["MOORCHEH_ONPREM_EMBEDDING_PROVIDER"] = str(_op_embed)`
- `sources/memanto/memanto/app/config.py:137` `MOORCHEH_ONPREM_URL: str = "http://localhost:8080"`
- `sources/memanto/memanto/app/config.py:206` `base = Path.home() / ".memanto"`

- `sources/memanto/memanto/cli/config/manager.py:129` `self.config_dir = config_dir or Path.home() / ".memanto"`
- `sources/memanto/memanto/cli/config/manager.py:145` `key = os.environ.get("MOORCHEH_API_KEY", "").strip()`
- `sources/memanto/memanto/cli/config/manager.py:157` `os.environ.get("SUPERMEMORY_API_KEY")`
- `sources/memanto/memanto/cli/config/manager.py:158` `or os.environ.get("supermemory_api_key")`
- `sources/memanto/memanto/cli/config/manager.py:174` `os.environ.get("MEM0_API_KEY") or os.environ.get("mem0_api_key") or ""`
- `sources/memanto/memanto/cli/config/manager.py:189` `os.environ.get("LETTA_API_KEY") or os.environ.get("letta_api_key") or ""`
- `sources/memanto/memanto/cli/config/manager.py:204` `os.environ.get("ZEP_API_KEY") or os.environ.get("zep_api_key") or ""`
- `sources/memanto/memanto/cli/config/manager.py:219` `os.environ.get("HINDSIGHT_API_KEY")`
- `sources/memanto/memanto/cli/config/manager.py:220` `or os.environ.get("hindsight_api_key")`
- `sources/memanto/memanto/cli/config/manager.py:235` `url = (os.environ.get("HINDSIGHT_BASE_URL") or "").strip()`
- `sources/memanto/memanto/cli/config/manager.py:253` `combined = (os.environ.get("LANGFUSE_API_KEY") or "").strip()`
- `sources/memanto/memanto/cli/config/manager.py:257` `public_key = (os.environ.get("LANGFUSE_PUBLIC_KEY") or "").strip()`
- `sources/memanto/memanto/cli/config/manager.py:258` `secret_key = (os.environ.get("LANGFUSE_SECRET_KEY") or "").strip()`
- `sources/memanto/memanto/cli/config/manager.py:271` `host = (os.environ.get("LANGFUSE_HOST") or "").strip()`
- `sources/memanto/memanto/cli/config/manager.py:279` `"""Write a single variable to ~/.memanto/.env and update os.environ."""`
- `sources/memanto/memanto/cli/config/manager.py:284` `os.environ[name] = value`
- `sources/memanto/memanto/cli/config/manager.py:286` `self.env_file.chmod(0o600)`
- `sources/memanto/memanto/cli/config/manager.py:288` `pass  # Windows may not support chmod`
- `sources/memanto/memanto/cli/config/manager.py:358` `"url": "http://localhost:8080",`
- `sources/memanto/memanto/cli/config/manager.py:426` `self.config_file.chmod(0o600)`
- `sources/memanto/memanto/cli/config/manager.py:465` `if host.startswith(("http://", "https://")):`
- `sources/memanto/memanto/cli/config/manager.py:477` `return f"http://{host}:{port}"`

</details>

## 5. Reproducibility

`range`: memanto>=0.1.0 (`sources/memanto/integrations/hermes-agents/hermes_memanto/plugin.yaml:6`); Moorcheh backend is external and not locked by the adapter.

Retained lock/checksum files: `sources/memanto/sdks/typescript/package-lock.json:1`. Upstream completeness is unknown: fetch.py:15 excludes .lock files, fetch.py:12 excludes build/vendor directories. Absence here is not absence upstream.

- `sources/memanto/integrations/hermes-agents/hermes_memanto/install.py:1` `"""Install the Memanto plugin into a Hermes plugins directory.`
- `sources/memanto/integrations/hermes-agents/hermes_memanto/install.py:8` `''__init__.py'' , so once installed it only needs the ''memanto'' SDK (declared`
- `sources/memanto/integrations/hermes-agents/hermes_memanto/install.py:13` `hermes-memanto-install                 # install into $HERMES_HOME (or ~/.hermes)`
- `sources/memanto/integrations/hermes-agents/hermes_memanto/install.py:14` `hermes-memanto-install --hermes-home /path/to/.hermes`
- `sources/memanto/integrations/hermes-agents/hermes_memanto/install.py:15` `hermes-memanto-install --force         # overwrite an existing install`
- `sources/memanto/integrations/hermes-agents/hermes_memanto/install.py:41` `def install(hermes_home: Path, *, force: bool = False) -> Path:`
- `sources/memanto/integrations/hermes-agents/hermes_memanto/install.py:60` `prog="hermes-memanto-install",`
- `sources/memanto/integrations/hermes-agents/hermes_memanto/install.py:61` `description="Install the Memanto memory-agent plugin into Hermes.",`
- `sources/memanto/integrations/hermes-agents/hermes_memanto/install.py:81` `target = install(hermes_home, force=args.force)`
- `sources/memanto/integrations/hermes-agents/hermes_memanto/install.py:86` `print(f"Installed the Memanto memory plugin at {target}")`
- `sources/memanto/integrations/hermes-agents/hermes_memanto/install.py:88` `print("  1. pip install memanto")`
- `sources/memanto/integrations/hermes-agents/hermes_memanto/install.py:95` `if __name__ == "__main__":  # pragma: no cover`

- `sources/memanto/integrations/hermes-agents/hermes_memanto/plugin.yaml:3` `version: 0.1.1`
- `sources/memanto/integrations/hermes-agents/hermes_memanto/plugin.yaml:5` `pip_dependencies:`
- `sources/memanto/integrations/hermes-agents/hermes_memanto/plugin.yaml:6` `- memanto>=0.1.0`

## 6. Tests

Adapter tests are in integrations/hermes-agents/tests/test_provider.py, counted below. No adapter forget path. Engine memory-write/policy/CLI delete tests are separately listed and do not establish server-side derived erasure. API test asserts DELETE returns deleted and calls Moorcheh documents.delete with the exact namespace and ID; Moorcheh is mocked. (`sources/memanto/tests/test_api.py:1416`).

Adapter-related declarations: **41**. Python AST test_ functions, fixtures excluded; literal TS/JS it/test, Go Test*, Rust test_* declarations. No parametrization expansion, collection, execution, or coverage measurement. Other rows are engine test candidates selected by mutation names; names alone do not verify forget coverage.

| file | declarations | adapter-related | forget-related declarations |
|---|---:|---|---|
| `sources/memanto/integrations/hermes-agents/tests/test_provider.py:1` | 41 | yes | none |
| `sources/memanto/integrations/langgraph/tests/test_store.py:1` | 23 | no | `test_do_put_delete_not_supported` `sources/memanto/integrations/langgraph/tests/test_store.py:266` |
| `sources/memanto/sdks/typescript/test/memanto.test.ts:1` | 14 | no | `rebootstraps after deleting the active agent` `sources/memanto/sdks/typescript/test/memanto.test.ts:319` |
| `sources/memanto/tests/test_api.py:1` | 126 | no | `test_delete_agent` `sources/memanto/tests/test_api.py:935`; `test_delete_active_agent_clears_session_state` `sources/memanto/tests/test_api.py:945`; `test_delete_agent_with_backup_delete` `sources/memanto/tests/test_api.py:976`; `test_delete_memory_with_session` `sources/memanto/tests/test_api.py:1416`; `test_delete_memory_not_found` `sources/memanto/tests/test_api.py:1446`; `test_delete_memory_rejects_session_agent_mismatch` `sources/memanto/tests/test_api.py:1469` |
| `sources/memanto/tests/test_chatgpt_claude_okf.py:1` | 62 | no | `test_refuses_to_delete_an_unrelated_directory` `sources/memanto/tests/test_chatgpt_claude_okf.py:252` |
| `sources/memanto/tests/test_cli.py:1` | 68 | no | `test_forget_force` `sources/memanto/tests/test_cli.py:974`; `test_forget_cancelled` `sources/memanto/tests/test_cli.py:991`; `test_forget_not_found` `sources/memanto/tests/test_cli.py:999`; `test_agent_delete_keep_cloud` `sources/memanto/tests/test_cli.py:1065`; `test_agent_delete_purge_cloud` `sources/memanto/tests/test_cli.py:1081`; `test_agent_delete_not_found` `sources/memanto/tests/test_cli.py:1100`; `test_resolving_by_provided_index_deletes_the_correct_memory` `sources/memanto/tests/test_cli.py:1321`; `test_expire_actions_retire_without_deleting` `sources/memanto/tests/test_cli.py:1387` |
| `sources/memanto/tests/test_connect_engine.py:1` | 12 | no | `test_remove_dedicated_instruction_deletes_memanto_managed_file` `sources/memanto/tests/test_connect_engine.py:38`; `test_pi_remove_deletes_extension` `sources/memanto/tests/test_connect_engine.py:202` |
| `sources/memanto/tests/test_e2e.py:1` | 24 | no | `test_19_delete_agent_local_only` `sources/memanto/tests/test_e2e.py:458` |
| `sources/memanto/tests/test_memory_policy.py:1` | 49 | no | `test_purges_only_past_the_window` `sources/memanto/tests/test_memory_policy.py:461`; `test_dry_run_deletes_nothing` `sources/memanto/tests/test_memory_policy.py:484`; `test_expired_memory_without_a_stamp_is_never_purged` `sources/memanto/tests/test_memory_policy.py:502` |
| `sources/memanto/tests/test_migrate_zep_hindsight.py:1` | 12 | no | `test_map_hindsight_maps_fact_types_and_skips_invalidated` `sources/memanto/tests/test_migrate_zep_hindsight.py:107` |
| `sources/memanto/tests/test_postcommit_summary_resilience.py:1` | 3 | no | `test_delete_returns_committed_result_when_summary_logging_fails` `sources/memanto/tests/test_postcommit_summary_resilience.py:94` |
| `sources/memanto/tests/test_unit.py:1` | 125 | no | `test_client_delete_agent_clears_session_state` `sources/memanto/tests/test_unit.py:2904`; `test_check_and_auto_recreate_invalidates_old_token` `sources/memanto/tests/test_unit.py:427`; `test_delete_session_tolerates_external_marker_removal` `sources/memanto/tests/test_unit.py:731`; `test_delete_agent` `sources/memanto/tests/test_unit.py:1095`; `test_delete_memory_handles_backend_shapes` `sources/memanto/tests/test_unit.py:1166`; `test_update_memory_accepts_onprem_delete_response` `sources/memanto/tests/test_unit.py:1174`; `test_update_memory_rejects_non_success_status_without_delete` `sources/memanto/tests/test_unit.py:1371`; `test_forget_succeeds_on_onprem_response_shape` `sources/memanto/tests/test_unit.py:1948`; `test_forget_reports_not_found_when_truly_missing` `sources/memanto/tests/test_unit.py:1966`; `test_forget_succeeds_on_cloud_response_shape` `sources/memanto/tests/test_unit.py:1977` |

## 7. Corrections to B2

Old values below are copied from the triage row (TRIAGE.md), not reconstructed from memory. A refinement narrows an over-broad field; it need not reverse its original value. Counts are distinct field paths, not the number of factual errors.

- `engine_delete.scope`: `"Named deletion/retraction endpoint or method, not a guarantee of physical erasure. Its reach may overlap an agent tool; lack of a native tool is counted separately."` → Memanto handler delegates document deletion to an external Moorcheh client; expiry and purge are separate; backend erasure unknown. Evidence: `sources/memanto/memanto/app/services/memory_write_service.py:583`.
- `context.mechanism.v`: `"Source expressions below identify retrieval, cached/static returns, lifecycle capture, and limits. A hook definition is not a guarantee it emits nonempty context each turn; branches and configuration apply."` → System block names the active memory agent and three tools (`sources/memanto/integrations/hermes-agents/hermes_memanto/provider.py:700`). Prefetch clips query to 2000 characters and requests configured result count/confidence (`sources/memanto/integrations/hermes-agents/hermes_memanto/provider.py:717`). The actual formatting and output cap are in the assembly quotes below; query/input caps are not output caps. Tool results are client recall/remember/answer envelopes. `_format_recall_block` builds type/confidence bullets and `<memanto-memory>` framing with no character cap, only max_results (default 10, configuration clamped to 100) (`sources/memanto/integrations/hermes-agents/hermes_memanto/provider.py:266`; `sources/memanto/integrations/hermes-agents/hermes_memanto/provider.py:274`; `sources/memanto/integrations/hermes-agents/hermes_memanto/provider.py:193`). Evidence: `sources/memanto/integrations/hermes-agents/hermes_memanto/provider.py:700`, `sources/memanto/integrations/hermes-agents/hermes_memanto/provider.py:717`, `sources/memanto/integrations/hermes-agents/hermes_memanto/provider.py:266`, `sources/memanto/integrations/hermes-agents/hermes_memanto/provider.py:274`, `sources/memanto/integrations/hermes-agents/hermes_memanto/provider.py:193`.
- `sec.telemetry.v`: `"unknown: no emitter identified in the selected adapter files; external dependencies not cleared"` → MOORCHEH_API_KEY is required for activation; empty key returns without becoming active (`sources/memanto/integrations/hermes-agents/hermes_memanto/provider.py:664`). Cloud/default accountless mode `no`; a usable accountless on-prem stack is `unknown`, since MOORCHEH_BASE_URL exists but the external on-prem engine/account semantics are not retained (`sources/memanto/memanto/cli/client/direct_client.py:80`). A required API key alone does not prove a hosted account requirement. Profile directories are set on the client (`sources/memanto/integrations/hermes-agents/hermes_memanto/provider.py:674`). Direct service/client access `yes` with the same key. Telemetry default `unknown` across Moorcheh SDK; local wrapper logging is not proof of telemetry off. Evidence: `sources/memanto/integrations/hermes-agents/hermes_memanto/provider.py:664`, `sources/memanto/memanto/cli/client/direct_client.py:80`, `sources/memanto/integrations/hermes-agents/hermes_memanto/provider.py:674`.
- `repro.decls`: `["integrations/hermes-agents/hermes_memanto/plugin.yaml:5", "integrations/hermes-agents/hermes_memanto/plugin.yaml:6", "integrations/hermes-agents/hermes_memanto/plugin.yaml:8", "pyproject.toml:3", "pyproject.toml:6", "pyproject.toml:36", "pyproject.toml:37", "pyproject.toml:38", "pyproject.toml:39", "pyproject.toml:40", "pyproject.toml:41", "pyproject.toml:42", "pyproject.toml:43", "pyproject.toml:44", "pyproject.toml:45", "pyproject.toml:46", "pyproject.toml:47", "pyproject.toml:48", "pyproject.toml:49", "pyproject.toml:50", "pyproject.toml:56", "pyproject.toml:57", "pyproject.toml:59", "pyproject.toml:61", "pyproject.toml:62", "pyproject.toml:63", "pyproject.toml:64", "pyproject.toml:65", "pyproject.toml:66", "pyproject.toml:67", "pyproject.toml:68", "pyproject.toml:69", "pyproject.toml:81", "pyproject.toml:108", "pyproject.toml:109", "pyproject.toml:110", "pyproject.toml:111", "pyproject.toml:112", "pyproject.toml:113", "pyproject.toml:114", "pyproject.toml:115", "pyproject.toml:116", "pyproject.toml:117", "pyproject.toml:122", "pyproject.toml:133", "pyproject.toml:144"]` → `range`: memanto>=0.1.0 (`sources/memanto/integrations/hermes-agents/hermes_memanto/plugin.yaml:6`); Moorcheh backend is external and not locked by the adapter. Evidence: `sources/memanto/integrations/hermes-agents/hermes_memanto/plugin.yaml:6`.
- `tests.count`: `1461` → 41 adapter-related declarations using the stated parser/scope; B2 counted a different, repository-wide inventory. This is a scope refinement, not a count of executed or passing tests. Adapter tests are in integrations/hermes-agents/tests/test_provider.py, counted below. No adapter forget path. Engine memory-write/policy/CLI delete tests are separately listed and do not establish server-side derived erasure. API test asserts DELETE returns deleted and calls Moorcheh documents.delete with the exact namespace and ID; Moorcheh is mocked. (`sources/memanto/tests/test_api.py:1416`). Evidence: `sources/memanto/tests/test_api.py:1416`.

MATRIX counts these 5 explicit field corrections/refinements. Sections 1-6 provide the body and test inventories. Test-name candidates are not passing tests or measured coverage.
