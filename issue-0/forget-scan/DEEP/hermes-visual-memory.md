# hermes-visual-memory: B3 static correctness review

Snapshot `86598e5c663cb132db2821094291372f1a72de9f` (MANIFEST.json row `hermes-visual-memory`). No plugin execution or passing-test claims.

## 1. Architecture

`tools_hooks` is too coarse: actual native registration is tools plus an auxiliary vision task, with no MemoryProvider or registered lifecycle hook. Runtime connects to/starts a separate local visual-memory service; that service reads camera images and invokes vision (`sources/hermes-visual-memory/extras/visual-memory/vismem/plugin.py:186`; `sources/hermes-visual-memory/extras/visual-memory/vismem/plugin.py:195`).

Hook/body inventory (no-op/fallback declarations are not active behavior):

| callable | declaration | first executable body line |
|---|---|---|
| `register` | `sources/hermes-visual-memory/extras/visual-memory/vismem/plugin.py:185` | `sources/hermes-visual-memory/extras/visual-memory/vismem/plugin.py:186` |
| `shutdown` | `sources/hermes-visual-memory/extras/visual-memory/vismem/service.py:259` | `sources/hermes-visual-memory/extras/visual-memory/vismem/service.py:260` |

## 2. Forget paths and derived copies

`yes`: forget_visual_memory(scope, older_than_days) sends POST `/forget` (`sources/hermes-visual-memory/extras/visual-memory/vismem/tools.py:151`; `sources/hermes-visual-memory/extras/visual-memory/vismem/service.py:360`). Scope all deletes pictures/observations, images preserves text, older deletes before cutoff with baseline exception, session deletes current-session scenes (`sources/hermes-visual-memory/extras/visual-memory/vismem/service.py:183`; `sources/hermes-visual-memory/extras/visual-memory/vismem/service.py:188`; `sources/hermes-visual-memory/extras/visual-memory/vismem/service.py:200`). Store unlinks image files and deletes observations/changes/scenes/items/events by branch; full forget clears sessions/meta and VACUUMs (`sources/hermes-visual-memory/extras/visual-memory/vismem/store.py:174`; `sources/hermes-visual-memory/extras/visual-memory/vismem/store.py:182`; `sources/hermes-visual-memory/extras/visual-memory/vismem/store.py:198`). `partial`: images-only intentionally preserves observations; session path leaves independently logged events/usage and host dialogue. No single-fact delete.

Host state.db is independent (`hermes-agent/hermes_state.py:181`). No reviewed adapter forget body performs a general purge of that database or built-in MEMORY.md/USER.md. Prompt/tool-result copies and independent source transcripts remain separate; source operation bodies below establish the scope of each plugin mutation.

<details>
<summary>Deletion, invalidation, archive and reset body ledger</summary>

Public routes, SDK/CLI dispatch and internal storage operations are distinguished by their source. Cache invalidation and management-object deletion are not additional fact-forget tools. A stub is not working deletion. Call-through means the downstream body still determines erasure. HTTP prefixes/selectors must be read with route registration.

**forget_visual_memory**, body `sources/hermes-visual-memory/extras/visual-memory/vismem/mcp_server.py:78`; implementation/call-through.

- `sources/hermes-visual-memory/extras/visual-memory/vismem/mcp_server.py:76` `@mcp.tool(description=tools.DESCRIPTIONS["forget_visual_memory"])`
- `sources/hermes-visual-memory/extras/visual-memory/vismem/mcp_server.py:77` `def forget_visual_memory(scope: str, older_than_days: float = 0) -> str:`
- `sources/hermes-visual-memory/extras/visual-memory/vismem/mcp_server.py:78` `return tools.forget_visual_memory(call, scope, older_than_days)`

**forget**, body `sources/hermes-visual-memory/extras/visual-memory/vismem/service.py:181`; implementation/call-through.

- `sources/hermes-visual-memory/extras/visual-memory/vismem/service.py:180` `def forget(self, scope: str, days: float | None = None) -> dict[str, Any]:`
- `sources/hermes-visual-memory/extras/visual-memory/vismem/service.py:183` `gone = self.store.forget()`
- `sources/hermes-visual-memory/extras/visual-memory/vismem/service.py:188` `gone = self.store.forget(images_only=True)`
- `sources/hermes-visual-memory/extras/visual-memory/vismem/service.py:192` `gone = self.store.forget(before=time.time() - days * 86400)`
- `sources/hermes-visual-memory/extras/visual-memory/vismem/service.py:200` `gone = self.store.forget_session(sid)`

**forget**, body `sources/hermes-visual-memory/extras/visual-memory/vismem/store.py:164`; implementation/call-through.

- `sources/hermes-visual-memory/extras/visual-memory/vismem/store.py:162` `def forget(self, before: float | None = None, images_only: bool = False) -> dict[str, int]:`
- `sources/hermes-visual-memory/extras/visual-memory/vismem/store.py:174` `p.unlink()`
- `sources/hermes-visual-memory/extras/visual-memory/vismem/store.py:182` `"DELETE FROM observations WHERE scene IN (SELECT id FROM scenes WHERE ts<? AND id!=?)",`
- `sources/hermes-visual-memory/extras/visual-memory/vismem/store.py:185` `self.db.execute("DELETE FROM changes WHERE ts<?", (cutoff,))`
- `sources/hermes-visual-memory/extras/visual-memory/vismem/store.py:186` `self.db.execute("DELETE FROM scenes WHERE ts<? AND id!=?", (cutoff, keep))`
- `sources/hermes-visual-memory/extras/visual-memory/vismem/store.py:187` `self.db.execute("DELETE FROM items WHERE last_seen<? AND status_ts<?", (cutoff, cutoff))`
- `sources/hermes-visual-memory/extras/visual-memory/vismem/store.py:188` `self.db.execute("DELETE FROM events WHERE ts<?", (cutoff,))`
- `sources/hermes-visual-memory/extras/visual-memory/vismem/store.py:191` `self.db.execute(f"DELETE FROM {table}")`
- `sources/hermes-visual-memory/extras/visual-memory/vismem/store.py:192` `self.db.execute("DELETE FROM meta WHERE key IN ('baseline', 'pending')")`

**forget_session**, body `sources/hermes-visual-memory/extras/visual-memory/vismem/store.py:202`; implementation/call-through.

- `sources/hermes-visual-memory/extras/visual-memory/vismem/store.py:201` `def forget_session(self, session_id: int) -> dict[str, int]:`
- `sources/hermes-visual-memory/extras/visual-memory/vismem/store.py:205` `(self.dir / s["image"]).unlink()`
- `sources/hermes-visual-memory/extras/visual-memory/vismem/store.py:209` `self.db.execute(f"DELETE FROM observations WHERE scene IN ({marks})", ids)`
- `sources/hermes-visual-memory/extras/visual-memory/vismem/store.py:210` `self.db.execute(f"DELETE FROM changes WHERE scene IN ({marks})", ids)`
- `sources/hermes-visual-memory/extras/visual-memory/vismem/store.py:211` `self.db.execute(f"DELETE FROM scenes WHERE id IN ({marks})", ids)`
- `sources/hermes-visual-memory/extras/visual-memory/vismem/store.py:212` `self.db.execute("DELETE FROM sessions WHERE id=?", (session_id,))`
- `sources/hermes-visual-memory/extras/visual-memory/vismem/store.py:215` `"DELETE FROM items WHERE id NOT IN (SELECT DISTINCT item FROM observations "`
- `sources/hermes-visual-memory/extras/visual-memory/vismem/store.py:218` `self.db.execute("DELETE FROM meta WHERE key IN ('baseline', 'pending')")`

**forget_visual_memory**, body `sources/hermes-visual-memory/extras/visual-memory/vismem/tools.py:148`; implementation/call-through.

- `sources/hermes-visual-memory/extras/visual-memory/vismem/tools.py:147` `def forget_visual_memory(call: Call, scope: str, older_than_days: float = 0) -> str:`
- `sources/hermes-visual-memory/extras/visual-memory/vismem/tools.py:148` `body: dict = {"scope": scope}`

</details>

## 3. What reaches the agent

No native MemoryProvider system block or prefetch. Agent receives registered tool schemas and service-derived tool replies; forget returns the service's `say` string (`sources/hermes-visual-memory/extras/visual-memory/vismem/tools.py:151`). Vision auxiliary task is internal service work, not automatic user-turn memory injection (`sources/hermes-visual-memory/extras/visual-memory/vismem/plugin.py:186`). No total tool-result cap established by the forget string wrapper.

This is not a native MemoryProvider; provider prefetch rules do not establish its behavior.

<details>
<summary>Context and tool-result assembly quotes</summary>

</details>

## 4. Data access boundaries

Direct store/service access `yes` subject to service authorization; POST handler rejects disallowed callers before dispatch (`sources/hermes-visual-memory/extras/visual-memory/vismem/service.py:347`). EYE_TOKEN is mandatory camera/device credential, not necessarily a hosted-account key (`sources/hermes-visual-memory/extras/visual-memory/plugin.yaml:23`). Accountless local mode `yes` with a local vision provider and camera token. Local SQLite/snapshot paths, service bind/auth and vision egress are quoted below. No dedicated telemetry emitter established; model-provider behavior unknown. Standalone service can read EYE token at ~/.config/hermes-voice/secrets/eye_token; data defaults ~/.local/share/hermes-visual-memory and API port 28950 (`sources/hermes-visual-memory/extras/visual-memory/vismem/config.py:18`; `sources/hermes-visual-memory/extras/visual-memory/vismem/config.py:22`; `sources/hermes-visual-memory/extras/visual-memory/vismem/config.py:26`).

Boundary model: same OS user, terminal, mounted filesystem and allowed network. Direct reachability means another client can use the same file/endpoint and available credentials, not unauthenticated access or encryption bypass. Hermes setup persists declared secrets through `hermes-agent/hermes_cli/memory_setup.py:321` and `hermes-agent/hermes_cli/config.py:2697`. These are possible configured paths; no live credentials were read.

Host credential-file detail: `get_hermes_home()/.env` is resolved at `hermes-agent/hermes_cli/config.py:473`. New files are secured to 0600 at `hermes-agent/hermes_cli/config.py:506`; the writer preserves an existing mode at `hermes-agent/hermes_cli/config.py:2544` and `:2562`. Therefore 0600 is not an unconditional mode claim for every existing profile.

<details>
<summary>Paths, environment, permissions, egress, telemetry and process-launch evidence</summary>

- `sources/hermes-visual-memory/extras/visual-memory/vismem/cli.py:33` `url = f"http://{cfg.api_host}:{cfg.api_port}{path}" + (`
- `sources/hermes-visual-memory/extras/visual-memory/vismem/cli.py:73` `"PATH": os.environ.get("PATH", "/usr/bin:/bin"),`
- `sources/hermes-visual-memory/extras/visual-memory/vismem/cli.py:80` `subprocess.run(["launchctl", "bootstrap", f"gui/{os.getuid()}", str(PLIST)], check=True)`
- `sources/hermes-visual-memory/extras/visual-memory/vismem/cli.py:89` `subprocess.run(["launchctl", "bootout", target], capture_output=True)`
- `sources/hermes-visual-memory/extras/visual-memory/vismem/cli.py:91` `if subprocess.run(["launchctl", "print", target], capture_output=True).returncode != 0:`
- `sources/hermes-visual-memory/extras/visual-memory/vismem/cli.py:99` `path = os.environ.get("PATH", "/usr/bin:/bin")`
- `sources/hermes-visual-memory/extras/visual-memory/vismem/cli.py:106` `subprocess.run(["systemctl", "--user", "daemon-reload"], check=True)`
- `sources/hermes-visual-memory/extras/visual-memory/vismem/cli.py:107` `subprocess.run(["systemctl", "--user", "enable", "--now", UNIT.name], check=True)`
- `sources/hermes-visual-memory/extras/visual-memory/vismem/cli.py:108` `subprocess.run(["systemctl", "--user", "restart", UNIT.name], check=True)`
- `sources/hermes-visual-memory/extras/visual-memory/vismem/cli.py:114` `subprocess.run(["systemctl", "--user", "disable", "--now", UNIT.name], capture_output=True)`
- `sources/hermes-visual-memory/extras/visual-memory/vismem/cli.py:117` `subprocess.run(["systemctl", "--user", "daemon-reload"], capture_output=True)`

- `sources/hermes-visual-memory/extras/visual-memory/vismem/config.py:11` `CONFIG_FILE = Path(os.environ.get("VISMEM_CONFIG", "~/.config/hermes-voice/visual-memory.toml")).expanduser()`
- `sources/hermes-visual-memory/extras/visual-memory/vismem/config.py:17` `eye_url: str = "http://[ip]:28940"`
- `sources/hermes-visual-memory/extras/visual-memory/vismem/config.py:49` `local_url: str = "http://127.0.0.1:11434/v1"`
- `sources/hermes-visual-memory/extras/visual-memory/vismem/config.py:156` `message = "display_origin must be the screen's HTTP(S) origin, e.g. http://homeassistant.local:8123"`

- `sources/hermes-visual-memory/extras/visual-memory/vismem/mcp_server.py:4` `VISMEM_URL         default http://127.0.0.1:28950`
- `sources/hermes-visual-memory/extras/visual-memory/vismem/mcp_server.py:21` `URL = os.environ.get("VISMEM_URL", "http://127.0.0.1:28950").rstrip("/")`
- `sources/hermes-visual-memory/extras/visual-memory/vismem/mcp_server.py:23` `os.environ.get("VISMEM_TOKEN_FILE", "~/.local/share/hermes-visual-memory/api_token")`

- `sources/hermes-visual-memory/extras/visual-memory/vismem/plugin.py:84` `"hermes config set plugins.entries.hermes-visual-memory.settings.eye_url http://<echo-ip>:28940"`
- `sources/hermes-visual-memory/extras/visual-memory/vismem/plugin.py:118` `self.url = f"http://127.0.0.1:{cfg.api_port}"`
- `sources/hermes-visual-memory/extras/visual-memory/vismem/plugin.py:167` `return get_secret(name, "") or ""`

- `sources/hermes-visual-memory/extras/visual-memory/vismem/service.py:31` `cfg.data_path.mkdir(parents=True, exist_ok=True, mode=0o700)`
- `sources/hermes-visual-memory/extras/visual-memory/vismem/service.py:33` `path.chmod(0o600)`
- `sources/hermes-visual-memory/extras/visual-memory/vismem/service.py:252` `"visual memory: API http://%s:%s, display feed :%s (allowed: %s)",`
- `sources/hermes-visual-memory/extras/visual-memory/vismem/service.py:312` `return origin == f"http://{local_ip}:8123" or (`
- `sources/hermes-visual-memory/extras/visual-memory/vismem/service.py:313` `local_ip == "127.0.0.1" and origin == "http://localhost:8123"`

- `sources/hermes-visual-memory/extras/visual-memory/vismem/store.py:67` `self.dir.mkdir(parents=True, exist_ok=True, mode=0o700)`
- `sources/hermes-visual-memory/extras/visual-memory/vismem/store.py:68` `self.dir.chmod(0o700)  # pictures of someone's home: this account only`
- `sources/hermes-visual-memory/extras/visual-memory/vismem/store.py:69` `(self.dir / "snapshots").mkdir(exist_ok=True, mode=0o700)`
- `sources/hermes-visual-memory/extras/visual-memory/vismem/store.py:71` `path.touch(mode=0o600, exist_ok=True)`
- `sources/hermes-visual-memory/extras/visual-memory/vismem/store.py:72` `path.chmod(0o600)  # SQLite creates its -wal and -shm files with the same permissions`
- `sources/hermes-visual-memory/extras/visual-memory/vismem/store.py:115` `path.parent.mkdir(exist_ok=True, mode=0o700)`
- `sources/hermes-visual-memory/extras/visual-memory/vismem/store.py:117` `path.chmod(0o600)`

- `sources/hermes-visual-memory/extras/visual-memory/vismem/vision_model.py:5` `- LocalVision: any OpenAI-compatible server with a vision model, e.g. Ollama (http://127.0.0.1:11434/v1)`
- `sources/hermes-visual-memory/extras/visual-memory/vismem/vision_model.py:6` `or LM Studio (http://127.0.0.1:1234/v1). Pictures stay on the user's network.`
- `sources/hermes-visual-memory/extras/visual-memory/vismem/vision_model.py:447` `proc = subprocess.run(`
- `sources/hermes-visual-memory/extras/visual-memory/vismem/vision_model.py:454` `env={**os.environ, "NO_COLOR": "1"},`
- `sources/hermes-visual-memory/extras/visual-memory/vismem/vision_model.py:456` `except subprocess.TimeoutExpired as err:`

</details>

## 5. Reproducibility

Engine is same-repo service, pinned to supplied source SHA; vision/backend dependencies may resolve ranges. Token/camera configuration and an available vision backend remain prerequisites, not validated runtime outcomes.

Retained lock/checksum files: none in snapshot. Upstream completeness is unknown: fetch.py:15 excludes .lock files, fetch.py:12 excludes build/vendor directories. Absence here is not absence upstream.



## 6. Tests

Adapter and service/store tests live under extras/visual-memory/tests, 70 declarations in the retained suite; forget assertions cover explicit scopes. No tests were run. Service/store test covers forget scopes. (`sources/hermes-visual-memory/extras/visual-memory/tests/test_memory.py:292`).

Adapter-related declarations: **70**. Python AST test_ functions, fixtures excluded; literal TS/JS it/test, Go Test*, Rust test_* declarations. No parametrization expansion, collection, execution, or coverage measurement. Other rows are engine test candidates selected by mutation names; names alone do not verify forget coverage.

| file | declarations | adapter-related | forget-related declarations |
|---|---:|---|---|
| `sources/hermes-visual-memory/extras/visual-memory/tests/scenes.py:1` | 0 | yes | none |
| `sources/hermes-visual-memory/extras/visual-memory/tests/test_memory.py:1` | 55 | yes | `test_model_forgets_link_still_moved` `sources/hermes-visual-memory/extras/visual-memory/tests/test_memory.py:182`; `test_forget_everything` `sources/hermes-visual-memory/extras/visual-memory/tests/test_memory.py:292` |
| `sources/hermes-visual-memory/extras/visual-memory/tests/test_plugin.py:1` | 7 | yes | none |
| `sources/hermes-visual-memory/extras/visual-memory/tests/test_service.py:1` | 8 | yes | none |

## 7. Corrections to B2

Old values below are copied from the triage row (TRIAGE.md), not reconstructed from memory. A refinement narrows an over-broad field; it need not reverse its original value. Counts are distinct field paths, not the number of factual errors.

- `kind.v`: `"tools_hooks"` → tools_hooks vocabulary retained for table, but actual registration is tools plus auxiliary task, no lifecycle hook or MemoryProvider. Evidence: `sources/hermes-visual-memory/extras/visual-memory/vismem/plugin.py:186`, `sources/hermes-visual-memory/extras/visual-memory/vismem/plugin.py:195`.
- `engine_delete.scope`: `"Named deletion/retraction endpoint or method, not a guarantee of physical erasure. Its reach may overlap an agent tool; lack of a native tool is counted separately."` → `yes`: forget_visual_memory(scope, older_than_days) sends POST `/forget` (`sources/hermes-visual-memory/extras/visual-memory/vismem/tools.py:151`; `sources/hermes-visual-memory/extras/visual-memory/vismem/service.py:360`). Scope all deletes pictures/observations, images preserves text, older deletes before cutoff with baseline exception, session deletes current-session scenes (`sources/hermes-visual-memory/extras/visual-memory/vismem/service.py:183`; `sources/hermes-visual-memory/extras/visual-memory/vismem/service.py:188`; `sources/hermes-visual-memory/extras/visual-memory/vismem/service.py:200`). Store unlinks image files and deletes observations/changes/scenes/items/events by branch; full forget clears sessions/meta and VACUUMs (`sources/hermes-visual-memory/extras/visual-memory/vismem/store.py:174`; `sources/hermes-visual-memory/extras/visual-memory/vismem/store.py:182`; `sources/hermes-visual-memory/extras/visual-memory/vismem/store.py:198`). `partial`: images-only intentionally preserves observations; session path leaves independently logged events/usage and host dialogue. No single-fact delete. Evidence: `sources/hermes-visual-memory/extras/visual-memory/vismem/tools.py:151`, `sources/hermes-visual-memory/extras/visual-memory/vismem/service.py:360`, `sources/hermes-visual-memory/extras/visual-memory/vismem/service.py:183`, `sources/hermes-visual-memory/extras/visual-memory/vismem/service.py:188`, `sources/hermes-visual-memory/extras/visual-memory/vismem/service.py:200`, `sources/hermes-visual-memory/extras/visual-memory/vismem/store.py:174`, `sources/hermes-visual-memory/extras/visual-memory/vismem/store.py:182`, `sources/hermes-visual-memory/extras/visual-memory/vismem/store.py:198`.
- `context.mechanism.v`: `"Uses registered hooks rather than a MemoryProvider prefetch; inspect hook registration and handler source. Exact injected size is unknown unless a bound is listed."` → No native MemoryProvider system block or prefetch. Agent receives registered tool schemas and service-derived tool replies; forget returns the service's `say` string (`sources/hermes-visual-memory/extras/visual-memory/vismem/tools.py:151`). Vision auxiliary task is internal service work, not automatic user-turn memory injection (`sources/hermes-visual-memory/extras/visual-memory/vismem/plugin.py:186`). No total tool-result cap established by the forget string wrapper. Evidence: `sources/hermes-visual-memory/extras/visual-memory/vismem/tools.py:151`, `sources/hermes-visual-memory/extras/visual-memory/vismem/plugin.py:186`.
- `sec.telemetry.v`: `"unknown: no emitter identified in the selected adapter files; external dependencies not cleared"` → Direct store/service access `yes` subject to service authorization; POST handler rejects disallowed callers before dispatch (`sources/hermes-visual-memory/extras/visual-memory/vismem/service.py:347`). EYE_TOKEN is mandatory camera/device credential, not necessarily a hosted-account key (`sources/hermes-visual-memory/extras/visual-memory/plugin.yaml:23`). Accountless local mode `yes` with a local vision provider and camera token. Local SQLite/snapshot paths, service bind/auth and vision egress are quoted below. No dedicated telemetry emitter established; model-provider behavior unknown. Standalone service can read EYE token at ~/.config/hermes-voice/secrets/eye_token; data defaults ~/.local/share/hermes-visual-memory and API port 28950 (`sources/hermes-visual-memory/extras/visual-memory/vismem/config.py:18`; `sources/hermes-visual-memory/extras/visual-memory/vismem/config.py:22`; `sources/hermes-visual-memory/extras/visual-memory/vismem/config.py:26`). Evidence: `sources/hermes-visual-memory/extras/visual-memory/vismem/service.py:347`, `sources/hermes-visual-memory/extras/visual-memory/plugin.yaml:23`, `sources/hermes-visual-memory/extras/visual-memory/vismem/config.py:18`, `sources/hermes-visual-memory/extras/visual-memory/vismem/config.py:22`, `sources/hermes-visual-memory/extras/visual-memory/vismem/config.py:26`.
- `repro.decls`: `["extras/visual-memory/plugin.yaml:11", "extras/visual-memory/plugin.yaml:15", "extras/visual-memory/plugin.yaml:16", "extras/visual-memory/plugin.yaml:17", "extras/visual-memory/plugin.yaml:18", "extras/visual-memory/plugin.yaml:19", "extras/visual-memory/plugin.yaml:20", "extras/visual-memory/plugin.yaml:21", "extras/visual-memory/plugin.yaml:23", "extras/visual-memory/plugin.yaml:25", "extras/visual-memory/pyproject.toml:3", "extras/visual-memory/pyproject.toml:6", "extras/visual-memory/pyproject.toml:7", "extras/visual-memory/pyproject.toml:8", "extras/visual-memory/pyproject.toml:9", "extras/visual-memory/pyproject.toml:12", "extras/visual-memory/pyproject.toml:14", "extras/visual-memory/pyproject.toml:15", "extras/visual-memory/pyproject.toml:21", "pyproject.toml:3", "pyproject.toml:6", "pyproject.toml:15", "pyproject.toml:28"]` → Engine is same-repo service, pinned to supplied source SHA; vision/backend dependencies may resolve ranges. Token/camera configuration and an available vision backend remain prerequisites, not validated runtime outcomes. Evidence: .
- `tests.count`: `150` → 70 adapter-related declarations using the stated parser/scope; B2 counted a different, repository-wide inventory. This is a scope refinement, not a count of executed or passing tests. Adapter and service/store tests live under extras/visual-memory/tests, 70 declarations in the retained suite; forget assertions cover explicit scopes. No tests were run. Service/store test covers forget scopes. (`sources/hermes-visual-memory/extras/visual-memory/tests/test_memory.py:292`). Evidence: `sources/hermes-visual-memory/extras/visual-memory/tests/test_memory.py:292`.

MATRIX counts these 6 explicit field corrections/refinements. Sections 1-6 provide the body and test inventories. Test-name candidates are not passing tests or measured coverage.
