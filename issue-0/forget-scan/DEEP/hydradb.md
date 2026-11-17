# hydradb: B3 static correctness review

Snapshot `056d684f7e7f71b1086d975a9b78c52674a960e5` (MANIFEST.json row `hydradb`). No plugin execution or passing-test claims.

## 1. Architecture

`provider`: requests-based HTTP client in-process; HydraDB service is external. No tools are exported (`sources/hydradb/hydradb_hermes/__init__.py:60`; `sources/hydradb/hydradb_hermes/__init__.py:79`).

Hook/body inventory (no-op/fallback declarations are not active behavior):

| callable | declaration | first executable body line |
|---|---|---|
| `register` | `sources/hydradb/hydradb_hermes/__init__.py:107` | `sources/hydradb/hydradb_hermes/__init__.py:108` |
| `is_available` | `sources/hydradb/hydradb_hermes/__init__.py:55` | `sources/hydradb/hydradb_hermes/__init__.py:56` |
| `initialize` | `sources/hydradb/hydradb_hermes/__init__.py:58` | `sources/hydradb/hydradb_hermes/__init__.py:59` |
| `get_config_schema` | `sources/hydradb/hydradb_hermes/__init__.py:67` | `sources/hydradb/hydradb_hermes/__init__.py:72` |
| `get_tool_schemas` | `sources/hydradb/hydradb_hermes/__init__.py:78` | `sources/hydradb/hydradb_hermes/__init__.py:79` |
| `system_prompt_block` | `sources/hydradb/hydradb_hermes/__init__.py:81` | `sources/hydradb/hydradb_hermes/__init__.py:82` |
| `prefetch` | `sources/hydradb/hydradb_hermes/__init__.py:84` | `sources/hydradb/hydradb_hermes/__init__.py:85` |
| `sync_turn` | `sources/hydradb/hydradb_hermes/__init__.py:92` | `sources/hydradb/hydradb_hermes/__init__.py:94` |

## 2. Forget paths and derived copies

`none` at adapter-tool layer. Client SDK/wrapper method `delete(ids, kind)` is implemented: nonempty IDs send DELETE `/context` with tenant_id, sub_tenant_id, ids and kind; empty IDs return zero (`sources/hydradb/hydradb_hermes/client.py:197`; `sources/hydradb/hydradb_hermes/client.py:200`). Engine server implementation is `not_in_repo`; matrix `not_in_repo` separates this implemented client request from absent server code. Granularity is knowledge/memory context IDs, not guaranteed individual extracted facts. Derived copies `unknown`.

Host state.db is independent (`hermes-agent/hermes_state.py:181`). No reviewed adapter forget body performs a general purge of that database or built-in MEMORY.md/USER.md. Prompt/tool-result copies and independent source transcripts remain separate; source operation bodies below establish the scope of each plugin mutation.

<details>
<summary>Deletion, invalidation, archive and reset body ledger</summary>

Public routes, SDK/CLI dispatch and internal storage operations are distinguished by their source. Cache invalidation and management-object deletion are not additional fact-forget tools. A stub is not working deletion. Call-through means the downstream body still determines erasure. HTTP prefixes/selectors must be read with route registration.

**delete**, body `sources/hydradb/hydradb_hermes/client.py:197`; implementation/call-through.

- `sources/hydradb/hydradb_hermes/client.py:196` `def delete(self, ids: list[str], kind: str = "knowledge") -> dict[str, Any]:`
- `sources/hydradb/hydradb_hermes/client.py:200` `resp = self._session.delete(`

</details>

## 3. What reaches the agent

System block says recall is automatic (`sources/hydradb/hydradb_hermes/__init__.py:82`). Prefetch delegates to recall_prompt (`sources/hydradb/hydradb_hermes/__init__.py:88`). That sends POST `/query` with max_results default 6, then returns server llm_prompt verbatim or builds `Relevant HydraDB memory:` plus chunk bullets (`sources/hydradb/hydradb_hermes/client.py:135`; `sources/hydradb/hydradb_hermes/client.py:143`). No local output size cap. Tools: none.

Host assembly: `hermes-agent/agent/memory_manager.py:352` joins provider system blocks. `hermes-agent/agent/turn_context.py:1081` requests prefetch for nontrivial turns; `hermes-agent/agent/turn_context.py:120` fences it; `hermes-agent/agent/turn_context.py:136` appends it to API-bound user content. Tool results are a third channel. Optional host spill is at `hermes-agent/agent/memory_manager.py:419`, not a universal provider cap. Built-in memory remains separately controlled (`hermes-agent/agent/system_prompt.py:542`).

<details>
<summary>Context and tool-result assembly quotes</summary>

`get_tool_schemas`, body `sources/hydradb/hydradb_hermes/__init__.py:79`:

- `sources/hydradb/hydradb_hermes/__init__.py:79` `return []`

`system_prompt_block`, body `sources/hydradb/hydradb_hermes/__init__.py:82`:

- `sources/hydradb/hydradb_hermes/__init__.py:82` `return "HydraDB provides persistent long-term memory. Relevant memories are recalled automatically before each turn."`

`prefetch`, body `sources/hydradb/hydradb_hermes/__init__.py:85`:

- `sources/hydradb/hydradb_hermes/__init__.py:86` `return ""`
- `sources/hydradb/hydradb_hermes/__init__.py:88` `return self._client.recall_prompt(query, kind=self._search_mode)`
- `sources/hydradb/hydradb_hermes/__init__.py:90` `return ""`

</details>

## 4. Data access boundaries

Credentials HYDRADB_API_KEY and HYDRADB_DATABASE are required; collection/base URL are optional. Default outbound host api.hydradb.com (`sources/hydradb/hydradb_hermes/__init__.py:61`). Same-credential independent HTTP access `yes`. Profile .env is the host-managed credential location; no plugin-specific credential writer. Accountless local mode `no` in supplied adapter. Telemetry: no separate emitter in the wrapper; remote service behavior `unknown` (`sources/hydradb/hydradb_hermes/client.py:200`).

Boundary model: same OS user, terminal, mounted filesystem and allowed network. Direct reachability means another client can use the same file/endpoint and available credentials, not unauthenticated access or encryption bypass. Hermes setup persists declared secrets through `hermes-agent/hermes_cli/memory_setup.py:321` and `hermes-agent/hermes_cli/config.py:2697`. These are possible configured paths; no live credentials were read.

Host credential-file detail: `get_hermes_home()/.env` is resolved at `hermes-agent/hermes_cli/config.py:473`. New files are secured to 0600 at `hermes-agent/hermes_cli/config.py:506`; the writer preserves an existing mode at `hermes-agent/hermes_cli/config.py:2544` and `:2562`. Therefore 0600 is not an unconditional mode claim for every existing profile.

<details>
<summary>Paths, environment, permissions, egress, telemetry and process-launch evidence</summary>

- `sources/hydradb/hydradb_hermes/__init__.py:26` `val = os.environ.get(canonical)`
- `sources/hydradb/hydradb_hermes/__init__.py:30` `val = os.environ.get(alias)`
- `sources/hydradb/hydradb_hermes/__init__.py:56` `return bool(os.environ.get("HYDRADB_API_KEY") and _env("HYDRADB_DATABASE"))`
- `sources/hydradb/hydradb_hermes/__init__.py:61` `api_key=os.environ.get("HYDRADB_API_KEY", ""),`
- `sources/hydradb/hydradb_hermes/__init__.py:64` `base_url=os.environ.get("HYDRADB_BASE_URL", "https://api.hydradb.com"),`

- `sources/hydradb/hydradb_hermes/client.py:26` `DEFAULT_BASE_URL = "https://api.hydradb.com"`

</details>

## 5. Reproducibility

No HydraDB engine SDK is installed: the adapter's own HTTP wrapper is pinned only by the repository SHA; external server version `unpinned`. Python HTTP dependency declarations are in the ledger.

Retained lock/checksum files: none in snapshot. Upstream completeness is unknown: fetch.py:15 excludes .lock files, fetch.py:12 excludes build/vendor directories. Absence here is not absence upstream.

- `sources/hydradb/hydradb_hermes/plugin.yaml:2` `version: 1.0.0`
- `sources/hydradb/hydradb_hermes/plugin.yaml:4` `requires_hermes: ">=0.21.5"`

## 6. Tests

tests/test_smoke.py exercises recall, capture, availability and schema; **no forget-path test**. The client DELETE body has no matching test in the supplied suite (`sources/hydradb/tests/test_smoke.py:19`; `sources/hydradb/hydradb_hermes/client.py:196`).

Adapter-related declarations: **6**. Python AST test_ functions, fixtures excluded; literal TS/JS it/test, Go Test*, Rust test_* declarations. No parametrization expansion, collection, execution, or coverage measurement. Other rows are engine test candidates selected by mutation names; names alone do not verify forget coverage.

| file | declarations | adapter-related | forget-related declarations |
|---|---:|---|---|
| `sources/hydradb/tests/test_smoke.py:1` | 6 | yes | none |

## 7. Corrections to B2

Old values below are copied from the triage row (TRIAGE.md), not reconstructed from memory. A refinement narrows an over-broad field; it need not reverse its original value. Counts are distinct field paths, not the number of factual errors.

- `engine_delete.scope`: `"Named deletion/retraction endpoint or method, not a guarantee of physical erasure. Its reach may overlap an agent tool; lack of a native tool is counted separately."` → Only client DELETE /context is present; server deletion and derived cleanup are not_in_repo/unknown. Evidence: `sources/hydradb/hydradb_hermes/client.py:200`.
- `context.bounds_status`: `"Source expressions include recall and other output bounds. They are not interchangeable: token vs character limits and static block vs tool output must be read at their cited call sites. No single total bound is established unless explicitly stated."` → max_results=6 limits requested hits; server-rendered llm_prompt has no local character cap. Evidence: `sources/hydradb/hydradb_hermes/client.py:119`, `sources/hydradb/hydradb_hermes/client.py:143`.
- `engine_delete.v`: `"yes"` → not_in_repo: an adapter-owned HTTP DELETE client exists, but no HydraDB server/engine implementation is supplied. Evidence: `sources/hydradb/hydradb_hermes/client.py:200`.
- `context.mechanism.v`: `"Source expressions below identify retrieval, cached/static returns, lifecycle capture, and limits. A hook definition is not a guarantee it emits nonempty context each turn; branches and configuration apply."` → System block says recall is automatic (`sources/hydradb/hydradb_hermes/__init__.py:82`). Prefetch delegates to recall_prompt (`sources/hydradb/hydradb_hermes/__init__.py:88`). That sends POST `/query` with max_results default 6, then returns server llm_prompt verbatim or builds `Relevant HydraDB memory:` plus chunk bullets (`sources/hydradb/hydradb_hermes/client.py:135`; `sources/hydradb/hydradb_hermes/client.py:143`). No local output size cap. Tools: none. Evidence: `sources/hydradb/hydradb_hermes/__init__.py:82`, `sources/hydradb/hydradb_hermes/__init__.py:88`, `sources/hydradb/hydradb_hermes/client.py:135`, `sources/hydradb/hydradb_hermes/client.py:143`.
- `sec.telemetry.v`: `"unknown: no emitter identified in the selected adapter files; external dependencies not cleared"` → Credentials HYDRADB_API_KEY and HYDRADB_DATABASE are required; collection/base URL are optional. Default outbound host api.hydradb.com (`sources/hydradb/hydradb_hermes/__init__.py:61`). Same-credential independent HTTP access `yes`. Profile .env is the host-managed credential location; no plugin-specific credential writer. Accountless local mode `no` in supplied adapter. Telemetry: no separate emitter in the wrapper; remote service behavior `unknown` (`sources/hydradb/hydradb_hermes/client.py:200`). Evidence: `sources/hydradb/hydradb_hermes/__init__.py:61`, `sources/hydradb/hydradb_hermes/client.py:200`.
- `repro.decls`: `["hydradb_hermes/plugin.yaml:9", "hydradb_hermes/plugin.yaml:10", "pyproject.toml:7", "pyproject.toml:11", "pyproject.toml:21", "pyproject.toml:23", "pyproject.toml:24", "pyproject.toml:31"]` → No HydraDB engine SDK is installed: the adapter's own HTTP wrapper is pinned only by the repository SHA; external server version `unpinned`. Python HTTP dependency declarations are in the ledger. Evidence: `sources/hydradb/hydradb_hermes/plugin.yaml:2`, `sources/hydradb/hydradb_hermes/plugin.yaml:4`.
- `tests.count`: `6` → 6 adapter-related declarations using the stated parser/scope; B2 counted a different, repository-wide inventory. This is a scope refinement, not a count of executed or passing tests. tests/test_smoke.py exercises recall, capture, availability and schema; **no forget-path test**. The client DELETE body has no matching test in the supplied suite (`sources/hydradb/tests/test_smoke.py:19`; `sources/hydradb/hydradb_hermes/client.py:196`). Evidence: `sources/hydradb/tests/test_smoke.py:19`, `sources/hydradb/hydradb_hermes/client.py:196`.

MATRIX counts these 7 explicit field corrections/refinements. Sections 1-6 provide the body and test inventories. Test-name candidates are not passing tests or measured coverage.
